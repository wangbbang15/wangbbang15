import pandas as pd
import matplotlib.pyplot as plt
import glob
import os

# 파일 로드
coverage_df = pd.read_csv(r"E:\CSW\ICLR\blacklist\analysis\coverage_comparison\dark_gene_coverage_comparison.txt", sep="\t")

# 열 이름 변경
coverage_df.rename(columns={"Chr": "Chrom"}, inplace=True)

# 모든 chr 파일 읽기
vaf_files = glob.glob(r"E:\CSW\ICLR\blacklist\analysis\variant_comparison\chr*_vaf_comparison.txt")

# 파일 리스트 출력 (디버깅용)
print("Found VAF files:", vaf_files)

# 파일이 존재하는지 확인
if not vaf_files:
    raise FileNotFoundError("No VAF comparison files found in the specified directory.")

# 이미지 저장 폴더 생성
output_dir = r"E:\CSW\ICLR\blacklist\analysis\image\comparison"
os.makedirs(output_dir, exist_ok=True)

# 각 chr 파일에 대해 처리
for file in vaf_files:
    # 파일 이름에서 chr 정보 추출
    chr_name = os.path.basename(file).split('_')[0]

    # VAF 파일 읽기
    vaf_df = pd.read_csv(file, sep="\s+", header=None, names=["Chrom", "Pos", "Ref", "Alt", "SRS_VAF", "LRS_VAF", "PacBio_VAF"])

    # 데이터 병합 (Coverage + VAF)
    merged_df = pd.merge(coverage_df, vaf_df, on=["Chrom"])

    # 병합된 데이터프레임 출력 (디버깅용)
    print(f"Merged DataFrame for {chr_name}:")
    print(merged_df.head())

    # 각 데이터프레임의 Chrom 열 값 출력 (디버깅용)
    print(f"Coverage DataFrame Chrom values for {chr_name}:")
    print(coverage_df["Chrom"].unique())
    print(f"VAF DataFrame Chrom values for {chr_name}:")
    print(vaf_df["Chrom"].unique())

    # NaN 값 제거
    merged_df = merged_df.dropna(subset=["SRS_VAF", "LRS_VAF", "PacBio_VAF"])

    # Scatter Plot
    plt.figure(figsize=(12, 48))
    plt.scatter(merged_df["SRS_Cov"], merged_df["SRS_VAF"], alpha=0.5, label="SRS", color="red")
    plt.scatter(merged_df["LRS_Cov"], merged_df["LRS_VAF"], alpha=0.5, label="LRS", color="blue")
    plt.scatter(merged_df["PacBio_Cov"], merged_df["PacBio_VAF"], alpha=0.5, label="PacBio", color="green")

    plt.xlabel("Average Coverage")
    plt.ylabel("Variant Allele Frequency (VAF)")
    plt.title(f"Coverage vs. Variant Allele Frequency (VAF) - {chr_name}")
    plt.legend()
    plt.grid()


    # 이미지 저장
    output_path = os.path.join(output_dir, f"coverage_vs_vaf_comparison_{chr_name}.png")
    plt.savefig(output_path)
    plt.close()

print("All plots have been saved.")
