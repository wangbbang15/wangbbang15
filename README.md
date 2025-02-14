paste control1_depth.txt control2_depth.txt case_depth.txt | awk '{
    mean_control = ($3 + $6) / 2;   # Control 1과 Control 2의 평균 Coverage
    ratio = ($9 / mean_control);    # Case의 Coverage 비율
    print $1, $2, ratio;
}' > chr14_ratio_case_vs_control.txt
