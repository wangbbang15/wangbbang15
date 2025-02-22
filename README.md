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

    # NaN 값 제거
    merged_df = merged_df.dropna(subset=["SRS_VAF", "LRS_VAF", "PacBio_VAF"])

    # **VAF 값 기준으로 정렬 (Y축 정렬)**
    merged_df = merged_df.sort_values(by=["SRS_VAF", "LRS_VAF", "PacBio_VAF"])

    # Scatter Plot 개선 (Y축을 VAF 값 기준으로 정렬)
    plt.figure(figsize=(12, 6))

    # SRS, LRS, PacBio VAF를 같은 Y축에 올바르게 표시
    plt.scatter(merged_df["SRS_Cov"], merged_df["SRS_VAF"], s=50, alpha=0.8, label="SRS (Short-read)", color="red", marker="o")
    plt.scatter(merged_df["LRS_Cov"], merged_df["LRS_VAF"], s=50, alpha=0.8, label="LRS (Illumina Long-read)", color="blue", marker="s")
    plt.scatter(merged_df["PacBio_Cov"], merged_df["PacBio_VAF"], s=50, alpha=0.8, label="PacBio (HiFi)", color="green", marker="^")

    # 레이블 및 제목 추가
    plt.xlabel("Average Coverage")
    plt.ylabel("Variant Allele Frequency (VAF)")
    plt.title(f"Coverage vs. Variant Allele Frequency (VAF) - {chr_name}")

    # Y축을 VAF 값 순서대로 정렬 (작은 값이 아래, 큰 값이 위)
    plt.ylim(merged_df["SRS_VAF"].min(), merged_df["PacBio_VAF"].max())

    plt.legend()
    plt.grid()

    # 이미지 저장
    output_path = os.path.join(output_dir, f"coverage_vs_vaf_comparison_{chr_name}.png")
    plt.savefig(output_path, dpi=300)
    plt.close()

print("All plots have been saved.")
