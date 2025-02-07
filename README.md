zcat 00-All.vcf.gz | sed 's/^\([1-9]\|1[0-9]\|2[0-2]\|X\|Y]\)\t/chr\1\t/' > modified_00-All.vcf
