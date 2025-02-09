#!/usr/bin/env python3

import sys
import csv
import os

def parse_snpEff_eff_field(eff_string):
    """
    EFF= 값(문자열)에서 쉼표(',')로 구분된 각 sub-annotation을 파싱하여
    아래 키를 갖는 dict 리스트 형태로 반환.

    포맷(헤더 참조):
      Effect( Effect_Impact|Functional_Class|Codon_Change|Amino_Acid_Change|
              Amino_Acid_length|Gene_Name|Transcript_BioType|Gene_Coding|
              Transcript_ID|Exon_Rank|Genotype_Number [| ERRORS | WARNINGS ] )
    예) downstream_gene_variant(MODIFIER||111||685|GALC|protein_coding|CODING|NM_000153.3||1)
    """
    # EFF=... 에서 '=' 기준 오른쪽만 추출
    # 예) "downstream_gene_variant(...),intergenic_region(...)" 등
    eff_string_value = eff_string.split('=', 1)[1].strip()

    # 쉼표로 나누어 각각 파싱
    eff_items = [x.strip() for x in eff_string_value.split(',') if x.strip()]

    parsed_list = []
    for item in eff_items:
        # item 예: "downstream_gene_variant(MODIFIER||111||685|GALC|protein_coding|CODING|NM_000153.3||1)"
        # 괄호 전후로 분리
        if '(' not in item:
            continue
        effect_name, inside = item.split('(', 1)
        effect_name = effect_name.strip()
        # inside 맨 끝의 ')' 제거
        if inside.endswith(')'):
            inside = inside[:-1]

        parts = inside.split('|')
        # parts[0] = Effect_Impact
        # parts[1] = Functional_Class
        # parts[2] = Codon_Change
        # parts[3] = Amino_Acid_Change
        # parts[4] = Amino_Acid_length
        # parts[5] = Gene_Name
        # parts[6] = Transcript_BioType
        # parts[7] = Gene_Coding
        # parts[8] = Transcript_ID
        # parts[9] = Exon_Rank
        # parts[10] = Genotype_Number
        # 그 뒤 [11], [12] ...에 에러/워닝이 있을 수 있음.

        # 길이가 부족할 경우를 대비해 빈 문자열로 채움
        while len(parts) < 11:
            parts.append('')

        eff_dict = {
            'Effect': effect_name,
            'Effect_Impact': parts[0],
            'Functional_Class': parts[1],
            'Codon_Change': parts[2],
            'Amino_Acid_Change': parts[3],
            'Amino_Acid_length': parts[4],
            'Gene_Name': parts[5],
            'Transcript_BioType': parts[6],
            'Gene_Coding': parts[7],
            'Transcript_ID': parts[8],
            'Exon_Rank': parts[9],
            'Genotype_Number': parts[10],
        }
        parsed_list.append(eff_dict)

    return parsed_list


def parse_snpEff_lof_nmd_field(lof_nmd_string):
    """
    LOF= 혹은 NMD= 값(문자열)을 받아, 쉼표(',')로 여러 개가 있을 수 있으므로
    각각 다음 4개 항목으로 분할:
      Gene_Name | Gene_ID | Number_of_transcripts_in_gene | Percent_of_transcripts_affected

    여러 개가 있으면, 각 항목별로 세미콜론(';')을 구분자로 합쳐서 반환(dict).
    즉, 최종적으로 하나의 variant에 대해서는
    "Gene_Name" -> "GN1;GN2", "Gene_ID" -> "ID1;ID2", ...
    이런 식으로 한 덩어리로 묶어 넘김.
    """
    # "LOF=GENE1|ENSG00001|2|50.0,GENE2|ENSG00002|1|100.0" 등
    # '=' 오른쪽 잘라낸 뒤에 쉼표로 split
    value_str = lof_nmd_string.split('=', 1)[1].strip()
    blocks = [b.strip() for b in value_str.split(',') if b.strip()]

    # 각 블록을 파싱해 4개 항목 리스트를 만든다
    gene_names = []
    gene_ids = []
    num_transcripts = []
    perc_affected = []

    for block in blocks:
        # block 예: "GeneName|GeneID|NumTx|Percent"
        parts = [x.strip() for x in block.split('|')]
        # 4개 보다 적을 때 대비
        while len(parts) < 4:
            parts.append('')
        gene_names.append(parts[0])
        gene_ids.append(parts[1])
        num_transcripts.append(parts[2])
        perc_affected.append(parts[3])

    # 여러 개면 ;로 합쳐서 문자열화
    return {
        'Gene_Name': ';'.join(gene_names),
        'Gene_ID': ';'.join(gene_ids),
        'Number_of_transcripts_in_gene': ';'.join(num_transcripts),
        'Percent_of_transcripts_affected': ';'.join(perc_affected)
    }


def get_info_value(info_str, key):
    """
    INFO 필드 전체 문자열에서 'key='로 시작하는 항목을 찾아 반환 (없으면 None).
    예) key='EFF' -> 'EFF=...' 항목을 찾아 그 오른쪽만 반환
    """
    for entry in info_str.split(';'):
        entry = entry.strip()
        if entry.startswith(key + '='):
            return entry  # "EFF=..." 통째로
    return None


def vcf_to_csv(vcf_dir, csv_dir):
    """
    VCF 파일에서 #CHROM, POS, ID, REF, ALT, QUAL, FILTER 및
    INFO의 EFF, LOF, NMD를 '풀어서' CSV로 저장.
    
    - EFF: 여러 sub-annotation이 있을 수 있으므로 -> 여러 행(row)을 만든다.
    - LOF, NMD: 여러 개 있으면 항목별로 세미콜론으로 묶어서 한 행에 저장.
    """
    # VCF 파일을 디렉토리에서 가져오기
    vcf_files = [f for f in os.listdir(vcf_dir) if f.endswith('.vcf')]

    fieldnames = [
        'CHROM', 'POS', 'ID', 'REF', 'ALT', 'QUAL', 'FILTER',
        # EFF details
        'EFF_Effect',
        'EFF_Effect_Impact',
        'EFF_Functional_Class',
        'EFF_Codon_Change',
        'EFF_Amino_Acid_Change',
        'EFF_Amino_Acid_length',
        'EFF_Gene_Name',
        'EFF_Transcript_BioType',
        'EFF_Gene_Coding',
        'EFF_Transcript_ID',
        'EFF_Exon_Rank',
        'EFF_Genotype_Number',
        # LOF details
        'LOF_Gene_Name',
        'LOF_Gene_ID',
        'LOF_Number_of_transcripts_in_gene',
        'LOF_Percent_of_transcripts_affected',
        # NMD details
        'NMD_Gene_Name',
        'NMD_Gene_ID',
        'NMD_Number_of_transcripts_in_gene',
        'NMD_Percent_of_transcripts_affected'
    ]
    
    if not os.path.exists(csv_dir):
        os.makedirs(csv_dir)

    for vcf_file in vcf_files:
        input_path = os.path.join(vcf_dir, vcf_file)
        output_path = os.path.join(csv_dir, vcf_file.replace('.vcf', '.csv'))

        with open(input_path, 'r') as vcf_in, open(output_path, 'w', newline='') as csv_out:
            writer = csv.DictWriter(csv_out, fieldnames=fieldnames)
            writer.writeheader()

            for line in vcf_in:
                line = line.strip()
                # 헤더/주석(#)은 스킵
                if not line or line.startswith('#'):
                    continue

                cols = line.split('\t')
                if len(cols) < 8:
                    continue

                chrom, pos, vid, ref, alt, qual, flt, info = cols[0:8]

                # EFF, LOF, NMD 부분 추출
                eff_raw = get_info_value(info, 'EFF')  # "EFF=..." 또는 None
                lof_raw = get_info_value(info, 'LOF')  # "LOF=..." 또는 None
                nmd_raw = get_info_value(info, 'NMD')  # "NMD=..." 또는 None

                # 1) EFF 파싱: 여러 sub-annotation이 있을 수 있으므로 list 반환
                eff_list = []
                if eff_raw:
                    eff_list = parse_snpEff_eff_field(eff_raw)
                else:
                    # EFF가 없는 경우에도, 최소 1개(빈)로 처리해서 행 생성하도록
                    eff_list = [{
                        'Effect': '',
                        'Effect_Impact': '',
                        'Functional_Class': '',
                        'Codon_Change': '',
                        'Amino_Acid_Change': '',
                        'Amino_Acid_length': '',
                        'Gene_Name': '',
                        'Transcript_BioType': '',
                        'Gene_Coding': '',
                        'Transcript_ID': '',
                        'Exon_Rank': '',
                        'Genotype_Number': '',
                    }]

                # 2) LOF 파싱: 여러 개일 경우 세미콜론으로 합쳐서 하나의 dict 반환
                lof_dict = {
                    'Gene_Name': '',
                    'Gene_ID': '',
                    'Number_of_transcripts_in_gene': '',
                    'Percent_of_transcripts_affected': ''
                }
                if lof_raw:
                    lof_dict = parse_snpEff_lof_nmd_field(lof_raw)

                # 3) NMD 파싱: 여러 개일 경우 세미콜론으로 합쳐서 하나의 dict 반환
                nmd_dict = {
                    'Gene_Name': '',
                    'Gene_ID': '',
                    'Number_of_transcripts_in_gene': '',
                    'Percent_of_transcripts_affected': ''
                }
                if nmd_raw:
                    nmd_dict = parse_snpEff_lof_nmd_field(nmd_raw)

                # EFF sub-annotation 하나당 1개의 CSV row를 생성
                for eff_item in eff_list:
                    row = {
                        'CHROM': chrom,
                        'POS': pos,
                        'ID': vid,
                        'REF': ref,
                        'ALT': alt,
                        'QUAL': qual,
                        'FILTER': flt,
                        # EFF 상세
                        'EFF_Effect': eff_item['Effect'],
                        'EFF_Effect_Impact': eff_item['Effect_Impact'],
                        'EFF_Functional_Class': eff_item['Functional_Class'],
                        'EFF_Codon_Change': eff_item['Codon_Change'],
                        'EFF_Amino_Acid_Change': eff_item['Amino_Acid_Change'],
                        'EFF_Amino_Acid_length': eff_item['Amino_Acid_length'],
                        'EFF_Gene_Name': eff_item['Gene_Name'],
                        'EFF_Transcript_BioType': eff_item['Transcript_BioType'],
                        'EFF_Gene_Coding': eff_item['Gene_Coding'],
                        'EFF_Transcript_ID': eff_item['Transcript_ID'],
                        'EFF_Exon_Rank': eff_item['Exon_Rank'],
                        'EFF_Genotype_Number': eff_item['Genotype_Number'],
                        # LOF 상세 (이미 여러 항목을 ';'로 합쳐놓음)
                        'LOF_Gene_Name': lof_dict['Gene_Name'],
                        'LOF_Gene_ID': lof_dict['Gene_ID'],
                        'LOF_Number_of_transcripts_in_gene': lof_dict['Number_of_transcripts_in_gene'],
                        'LOF_Percent_of_transcripts_affected': lof_dict['Percent_of_transcripts_affected'],
                        # NMD 상세
                        'NMD_Gene_Name': nmd_dict['Gene_Name'],
                        'NMD_Gene_ID': nmd_dict['Gene_ID'],
                        'NMD_Number_of_transcripts_in_gene': nmd_dict['Number_of_transcripts_in_gene'],
                        'NMD_Percent_of_transcripts_affected': nmd_dict['Percent_of_transcripts_affected'],
                    }
                    writer.writerow(row)

if __name__ == "__main__":
    """
    VCF 디렉토리 경로와 출력 CSV 디렉토리 경로 설정
    """
    vcf_dir = "/mnt/e/CSW"
    csv_dir = "/mnt/e/CSW/output"

    vcf_to_csv(vcf_dir, csv_dir)
