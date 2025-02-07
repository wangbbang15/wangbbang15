(base) rare1@DESKTOP-Q8E0MN3:/mnt/e/CSW/reference$ zcat 00-All.vcf.gz | sed 's/^([1-9]|1[0-9]|2[0-2]|X|Y])\t/chr\1\t/' > hg38_00-All.vcf
sed: -e expression #1, char 40: invalid reference \1 on `s' command's RHS
(base) rare1@DESKTOP-Q8E0MN3:/mnt/e/CSW/reference$
