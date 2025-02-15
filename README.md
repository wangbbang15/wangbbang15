import pandas as pd
import matplotlib.pyplot as plt


# Define file paths
control1_file = r"E:\CSW\ICLR\SV\MED13L\bam\MED13L\control1_GL00178P_SRS_depth.txt"
control2_file = r"E:\CSW\ICLR\SV\MED13L\bam\MED13L\control2_GL00873P_SRS_depth.txt"
patient_file = r"E:\CSW\ICLR\SV\MED13L\bam\MED13L\patient_GL00143P_SRS_depth.txt"
output_file = r"E:\CSW\ICLR\SV\MED13L\bam\MED13L\chr12_ratio_patient_vs_control_SRS.txt"

# Load depth data
control1 = pd.read_csv(control1_file, sep="\t", header=None, names=["chr", "pos", "depth1"])
control2 = pd.read_csv(control2_file, sep="\t", header=None, names=["chr", "pos", "depth2"])
patient = pd.read_csv(patient_file, sep="\t", header=None, names=["chr", "pos", "depth3"])

# Merge data on chromosome and position
df = control1.merge(control2, on=["chr", "pos"]).merge(patient, on=["chr", "pos"])

# Calculate control average and ratio for patient
df["mean_control"] = (df["depth1"] + df["depth2"]) / 2
df["ratio"] = df["depth3"] / df["mean_control"]

# Plotting
plt.figure(figsize=(12, 6))

# Plot Control 1, Control 2, and Patient with different colors
plt.scatter(df["pos"], df["depth1"], color="blue", s=1, alpha=0.5, label="Control 1")
plt.scatter(df["pos"], df["depth2"], color="green", s=1, alpha=0.5, label="Control 2")
plt.scatter(df["pos"], df["depth3"], color="red", s=1, alpha=0.5, label="Patient")

# Add labels and title
plt.xlabel("Genomic Position (chr12)")
plt.ylabel("Coverage Depth")
plt.title("Coverage Depth Comparison: Control vs Patient on chr12")
plt.legend()

# Save plot to file
output_image_path = r"E:\CSW\ICLR\SV\MED13L\bam\MED13L\coverage_depth_comparison_SRS.png"
plt.savefig(output_image_path)

# Remove the plt.show() to not display the plot
# plt.show()
