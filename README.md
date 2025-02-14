import pandas as pd
import matplotlib.pyplot as plt

# 파일 경로
control1_file = "/mnt/e/CSW/ICLR/SV/MED13L/control1_GL00178P_LRS_depth.txt"
control2_file = "/mnt/e/CSW/ICLR/SV/MED13L/control2_GL00873P_LRS_depth.txt"
patient_file = "/mnt/e/CSW/ICLR/SV/MED13L/patient_GL00143P_LRS_depth.txt"
output_file = "/mnt/e/CSW/ICLR/SV/MED13L/chr12_ratio_patient_vs_control.txt"

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

# Show plot
plt.show()
