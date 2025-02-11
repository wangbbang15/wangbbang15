#!/bin/bash

# 에러 발생 시 즉시 종료
set -e

# SVTOPO 실행 파일 경로 설정
SVTOPO_PATH="/mnt/e/CSW/Pacbio/sawfish/svtopo/HIFI-SVTopo"

# 환경 변수 설정
export PATH=$SVTOPO_PATH:$PATH

# 입력 및 출력 파일 경로 설정
INPUT_FILE="/mnt/u/Results_Revio/WGS/DEDD/DEDD_G1704337_78807608/G1704337_78807608.pbmm2.sorted.haplotagged.snp.vaf30.bam"
VCF_FILE="/mnt/e/CSW/Pacbio/sawfish/output/DEDD/G1704337_78807608_joint_call_dir/genotyped.sv.vcf.gz"
OUTPUT_PATH="/mnt/e/CSW/Pacbio/sawfish/svtopo/HIFI-SVTopo/output/test"
EXCLUDE_REGIONS="/mnt/e/CSW/Pacbio/sawfish/svtopo/HIFI-SVTopo/cnv.excluded_regions.hg38.bed.gz"

# 결과 파일 경로
JSON_OUTPUT="$OUTPUT_PATH/svtopo_G1704337_78807608_output.json.gz"
READMAPS_OUTPUT="$OUTPUT_PATH/sawfish_G1704337_78807608_supporting_reads.json"
IMAGE_OUTPUT="$OUTPUT_PATH/images/G1704337_78807608_svtopo"

# 파일 존재 여부 확인
if [[ ! -f "$INPUT_FILE" ]]; then
    echo "Error: BAM file not found: $INPUT_FILE"
    exit 1
fi

if [[ ! -f "$VCF_FILE" ]]; then
    echo "Error: VCF file not found: $VCF_FILE"
    exit 1
fi

if [[ ! -f "$EXCLUDE_REGIONS" ]]; then
    echo "Error: Exclude regions file not found: $EXCLUDE_REGIONS"
    exit 1
fi

# SVTOPO 실행 파일 확인
if ! command -v svtopo &>/dev/null; then
    echo "Error: svtopo command not found in PATH."
    exit 1
fi

if ! command -v svtopovz &>/dev/null; then
    echo "Error: svtopovz command not found in PATH."
    exit 1
fi

# 출력 디렉토리 생성 (존재하지 않으면 생성)
mkdir -p "$OUTPUT_PATH/images"

# SVTOPO 실행
echo "Running svtopo..."
svtopo \
    --bam "$INPUT_FILE" \
    --json-out "$JSON_OUTPUT" \
    --variant-readmaps "$READMAPS_OUTPUT" \
    --vcf "$VCF_FILE" \
    --exclude-regions "$EXCLUDE_REGIONS"

# SVTOPOVZ 실행
echo "Running svtopovz..."
svtopovz \
    --json "$JSON_OUTPUT" \
    --out-prefix "$IMAGE_OUTPUT"

echo "✅ SVTopo analysis completed successfully!"

에서
(svtopo) rare1@DESKTOP-Q8E0MN3:/mnt/e/CSW/script/sawfish/svtopo$ ./2_svtopo.sh
Running svtopo...
error: unexpected argument '--variant-readmaps' found

  tip: a similar argument exists: '--variant-readnames'

Usage: svtopo <--bam <BAM>|--json-out <PATH>|--vcf <VCF>|--variant-readnames <JSON>|--exclude-regions <BED>|--no-filter|--allow-unphased|--max-coverage <INT>|--target-region <TARGET_REGION>|--verbose>

For more information, try '--help'.
(svtopo) rare1@DESKTOP-Q8E0MN3:/mnt/e/CSW/script/sawfish/svtopo$ 에러

