import pandas as pd

# Define file paths
control1_file = "/mnt/e/CSW/ICLR/SV/MED13L/control1_GL00178P_LRS_depth.txt"
control2_file = "/mnt/e/CSW/ICLR/SV/MED13L/control2_GL00873P_LRS_depth.txt"
patient_file = "/mnt/e/CSW/ICLR/SV/MED13L/patient_GL00143P_LRS_depth.txt"
output_file = "/mnt/e/CSW/ICLR/SV/MED13L/chr12_ratio_patient_vs_control.txt"

# Check if files exist
for file in [control1_file, control2_file, patient_file]:
    try:
        with open(file, "r") as f:
            pass
    except FileNotFoundError:
        print(f"Error: {file} is missing!")
        exit(1)

# Load depth data
control1 = pd.read_csv(control1_file, sep="\t", header=None, names=["chr", "pos", "depth1"])
control2 = pd.read_csv(control2_file, sep="\t", header=None, names=["chr", "pos", "depth2"])
patient = pd.read_csv(patient_file, sep="\t", header=None, names=["chr", "pos", "depth3"])

# Merge data on chromosome and position
df = control1.merge(control2, on=["chr", "pos"]).merge(patient, on=["chr", "pos"])

# Calculate control average and ratio for patient
df["mean_control"] = (df["depth1"] + df["depth2"]) / 2
df["ratio"] = df["depth3"] / df["mean_control"]

# Save output
df[["chr", "pos", "ratio"]].to_csv(output_file, sep="\t", index=False, header=False)

print(f"✅ CNV analysis complete: Results saved in {output_file}")
