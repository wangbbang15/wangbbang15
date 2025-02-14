#!/bin/bash

# Define filenames (adjust to match user's depth filenames)
CONTROL1="/mnt/e/CSW/ICLR/SV/MED13L/control1_GL00178P_LRS_depth.txt"
CONTROL2="/mnt/e/CSW/ICLR/SV/MED13L/control2_GL00873P_LRS_depth.txt"
PATIENT="/mnt/e/CSW/ICLR/SV/MED13L/patient_GL00143P_LRS_depth.txt"
OUTPUT="/mnt/e/CSW/ICLR/SV/MED13L/chr12_ratio_patient_vs_control.txt"

# Check if files exist
if [[ ! -f "$CONTROL1" || ! -f "$CONTROL2" || ! -f "$PATIENT" ]]; then
    echo "Error: One or more depth files are missing!"
    exit 1
fi

# Calculate the average coverage of two controls and compare with patient coverage to calculate the ratio
paste "$CONTROL1" "$CONTROL2" "$PATIENT" | awk '{
    mean_control = ($3 + $6) / 2;  # Average coverage of Control 1 and Control 2
    ratio = ($9 / mean_control);   # Coverage ratio of Case (Patient)
    print $1, $2, ratio;
}' > "$OUTPUT"

echo "CNV analysis complete: Results saved in $OUTPUT file!"
