#!/bin/bash

# Methbat 실행을 위한 환경 설정
SAWFISH_PATH="/mnt/e/CSW/Pacbio/sawfish-v0.12.9-x86_64-unknown-linux-gnu/bin"  # methbat 경로
REFERENCE_PATH="/mnt/e/CSW/reference/hg38.fa" 
INPUT_FILE="/mnt/u/Results_Revio/WGS/DEDD/DEDD_G1704337_78807608"  #   
OUTPUT_PATH="/mnt/e/CSW/Pacbio/sawfish/output/DEDD"  # 결과 출력 파일 경로

# 환경 변수 설정
export PATH=$SAWFISH_PATH:$PATH

# 명령어 실행
sawfish discover --threads 10 --ref $REFERENCE_PATH --bam $INPUT_FILE/G1704337_78807608.pbmm2.sorted.haplotagged.snp.vaf30.bam --output-dir $OUTPUT_PATH/G1704337_78807608_discover_dir
sawfish joint-call --threads 10 --sample $OUTPUT_PATH/G1704337_78807608_discover_dir --output-dir $OUTPUT_PATH/G1704337_78807608_joint_call_dir



sawfish 에서

Optional variant read support output
To help show which reads support each SV allele, the optional --report-supporting-reads argument can be added to the joint-call command line. When this is used a compressed json output file is provided in ${OUTPUT_DIR}/supporting_reads.json.gz. 

이렇게 되어있는데 적절하게 수정해줘
