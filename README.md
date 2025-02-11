#!/bin/bash

# 에러 발생 시 즉시 종료
set -e

# SVTopo 실행 파일 경로 설정
SVTOPO_PATH="/mnt/e/CSW/Pacbio/sawfish/svtopo/HIFI-SVTopo"

# 환경 변수 설정
export PATH=$SVTOPO_PATH:$PATH

# 입력 및 출력 파일 경로 설정
INPUT_BAM="/mnt/u/Results_Revio/WGS/DEDD/DEDD_G1704337_78807608/G1704337_78807608.pbmm2.sorted.haplotagged.snp.vaf30.bam"
VCF_FILE="/mnt/e/CSW/Pacbio/sawfish/output/DEDD/G1704337_78807608_joint_call_dir/genotyped.sv.vcf.gz"
SUPPORTING_READS_JSON="/mnt/e/CSW/Pacbio/sawfish/output/DEDD/G1704337_78807608_joint_call_dir/supporting_reads.json.gz"
OUTPUT_PATH="/mnt/e/CSW/Pacbio/sawfish/svtopo/HIFI-SVTopo/output/test"
EXCLUDE_REGIONS="/mnt/e/CSW/Pacbio/sawfish/svtopo/HIFI-SVTopo/cnv.excluded_regions.hg38.bed.gz"

# SVTopo 실행 후 생성될 JSON 파일 경로
SVTOPO_JSON="$OUTPUT_PATH/svtopo_G1704337_78807608_output.json.gz"
IMAGE_OUTPUT="$OUTPUT_PATH/images/G1704337_78807608_svtopo"

# 파일 존재 여부 확인
if [[ ! -f "$INPUT_BAM" ]]; then
    echo "Error: BAM file not found: $INPUT_BAM"
    exit 1
fi

if [[ ! -f "$VCF_FILE" ]]; then
    echo "Error: VCF file not found: $VCF_FILE"
    exit 1
fi

if [[ ! -f "$SUPPORTING_READS_JSON" ]]; then
    echo "Error: Supporting Reads JSON file not found: $SUPPORTING_READS_JSON"
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
    --bam "$INPUT_BAM" \
    --json-out "$SVTOPO_JSON" \
    --variant-readnames "$SUPPORTING_READS_JSON" \  # ✅ Sawfish JSON 적용됨
    --vcf "$VCF_FILE" \
    --exclude-regions "$EXCLUDE_REGIONS"

# SVTOPOVZ 실행 (시각화)
echo "Running svtopovz..."
svtopovz \
    --json "$SVTOPO_JSON" \
    --out-prefix "$IMAGE_OUTPUT"

echo "✅ SVTopo analysis completed successfully!"
