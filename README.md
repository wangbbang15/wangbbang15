#!/bin/bash

# 에러 발생 시 즉시 종료
set -e

# Sawfish 실행을 위한 환경 설정
SAWFISH_PATH="/mnt/e/CSW/Pacbio/sawfish-v0.12.9-x86_64-unknown-linux-gnu/bin"
REFERENCE_PATH="/mnt/e/CSW/reference/hg38.fa"
INPUT_BAM="/mnt/u/Results_Revio/WGS/DEDD/DEDD_G1704337_78807608/G1704337_78807608.pbmm2.sorted.haplotagged.snp.vaf30.bam"
OUTPUT_PATH="/mnt/e/CSW/Pacbio/sawfish/output/DEDD"

# 환경 변수 설정
export PATH=$SAWFISH_PATH:$PATH

# 발견(Discover) 단계 실행
echo "Running Sawfish Discover..."
sawfish discover --threads 10 \
    --ref "$REFERENCE_PATH" \
    --bam "$INPUT_BAM" \
    --output-dir "$OUTPUT_PATH/G1704337_78807608_discover_dir"

# Joint-Call 단계 실행 (supporting_reads.json.gz 생성)
echo "Running Sawfish Joint-Call..."
sawfish joint-call --threads 10 \
    --sample "$OUTPUT_PATH/G1704337_78807608_discover_dir" \
    --output-dir "$OUTPUT_PATH/G1704337_78807608_joint_call_dir" \
    --report-supporting-reads

# Supporting Reads JSON 파일 확인
SUPPORTING_READS_JSON="$OUTPUT_PATH/G1704337_78807608_joint_call_dir/supporting_reads.json.gz"

if [[ -f "$SUPPORTING_READS_JSON" ]]; then
    echo "✅ Supporting Reads JSON generated: $SUPPORTING_READS_JSON"
else
    echo "❌ Error: Supporting Reads JSON file not found!"
    exit 1
fi

echo "✅ Sawfish pipeline completed successfully!"
