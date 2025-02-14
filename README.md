(base) rare1@DESKTOP-Q8E0MN3:/mnt/e/CSW/ICLR/SV/MED13L/bam$ paste control1_GL00178P_LRS_depth.txt control2_GL00873P_LRS_depth.txt patient_GL00143P_LRS_depth.txt | awk '{ mean_control = ($3 + $6) / 2; # Control 1과 Control 2의 평균 Coverage ratio = ($9 / mean_control); # Case의 Coverage 비율 print $1, $2, ratio; }' > chr12_ratio_patient_vs_control.txt
awk: cmd. line:1: (END OF FILE)
awk: cmd. line:1:                                 ^ source files / command-line arguments must contain complete functions or rules
(base) rare1@DESKTOP-Q8E0MN3:/mnt/e/CSW/ICLR/SV/MED13L/bam$
