import os
import pandas as pd

# Input and output directory
input_dir = "/mnt/e/CSW/ICLR/sv"
output_file = os.path.join(input_dir, "merged_filtered.csv")

# List all TSV files in the directory
tsv_files = [f for f in os.listdir(input_dir) if f.endswith(".tsv")]

# Merge all TSV files
df_list = []
for file in tsv_files:
    file_path = os.path.join(input_dir, file)
    df = pd.read_csv(file_path, sep="\t", low_memory=False)  # Read TSV
    df_list.append(df)

# Combine all dataframes
merged_df = pd.concat(df_list, ignore_index=True)

# Filter the dataframe where ACMG_class is 4 or 5, or full is 4 or 5
filtered_df = merged_df[
    (merged_df["ACMG_class"].astype(str).isin(["4", "5"])) |
    (merged_df["full"].astype(str).isin(["4", "5"]))
]

# Save the filtered dataframe as CSV
filtered_df.to_csv(output_file, index=False)

print(f"Filtered CSV saved to: {output_file}")
