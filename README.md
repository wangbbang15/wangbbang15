#!/bin/bash

# 파일명 정의 (사용자의 depth 파일명을 맞춰서 수정)
CONTROL1="control1_depth.txt"
CONTROL2="control2_depth.txt"
PATIENT="patient_depth.txt"
OUTPUT="chr12_ratio_patient_vs_control.txt"

# 파일 존재 여부 확인
if [[ ! -f "$CONTROL1" || ! -f "$CONTROL2" || ! -f "$PATIENT" ]]; then
    echo "Error: One or more depth files are missing!"
    exit 1
fi

# Control 두 명의 평균 Coverage 계산 후 Patient Coverage와 비교하여 Ratio 계산
paste "$CONTROL1" "$CONTROL2" "$PATIENT" | awk '{
    mean_control = ($3 + $6) / 2;  # Control 1과 Control 2의 평균 Coverage
    ratio = ($9 / mean_control);   # Case(Patient)의 Coverage 비율
    print $1, $2, ratio;
}' > "$OUTPUT"

echo "✅ CNV 분석 완료: 결과가 $OUTPUT 파일에 저장됨!"
