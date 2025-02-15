import pandas as pd
import matplotlib.pyplot as plt

# Define file paths
control1_file = r"E:\CSW\ICLR\SV\MED13L\bam\MED13L\control1_GL00178P_SRS_depth.txt"
control2_file = r"E:\CSW\ICLR\SV\MED13L\bam\MED13L\control2_GL00873P_SRS_depth.txt"
patient_file = r"E:\CSW\ICLR\SV\MED13L\bam\MED13L\patient_GL00143P_SRS_depth.txt"
output_dir = r"E:\CSW\ICLR\SV\MED13L\bam\MED13L"  # 이미지 저장 경로

# Load depth data
control1 = pd.read_csv(control1_file, sep="\t", header=None, names=["chr", "pos", "depth"])
control2 = pd.read_csv(control2_file, sep="\t", header=None, names=["chr", "pos", "depth"])
patient = pd.read_csv(patient_file, sep="\t", header=None, names=["chr", "pos", "depth"])

# 파일별 플롯 생성 및 저장
def plot_coverage(data, title, filename, color):
    plt.figure(figsize=(12, 6))
    plt.scatter(data["pos"], data["depth"], color=color, s=1, alpha=0.5)
    plt.xlabel("Genomic Position (chr12)")
    plt.ylabel("Coverage Depth")
    plt.title(title)
    plt.savefig(f"{output_dir}/{filename}", dpi=300)  # 이미지 저장
    plt.close()

# 개별 파일별 그래프 저장
plot_coverage(control1, "Control 1 Coverage Depth on chr12", "control1_coverage_SRS.png", "blue")
plot_coverage(control2, "Control 2 Coverage Depth on chr12", "control2_coverage_SRS.png", "green")
plot_coverage(patient, "Patient Coverage Depth on chr12", "patient_coverage_SRS.png", "red")

print("✅ Individual coverage depth images saved in:", output_dir)
