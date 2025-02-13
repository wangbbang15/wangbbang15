import os
import pandas as pd

# Input and output directory
input_dir = r"E:\CSW\ICLR\SV\AnnotSV"
output_file = os.path.join(input_dir, "merged_SV_filtered.csv")

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


[Running] python -u "e:\CSW\ICLR\SV\py\sv_test.py"
Traceback (most recent call last):
  File "C:\Users\Rare_1\AppData\Local\Programs\Python\Python312\Lib\site-packages\pandas\core\indexes\base.py", line 3805, in get_loc
    return self._engine.get_loc(casted_key)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "index.pyx", line 167, in pandas._libs.index.IndexEngine.get_loc
  File "index.pyx", line 196, in pandas._libs.index.IndexEngine.get_loc
  File "pandas\\_libs\\hashtable_class_helper.pxi", line 7081, in pandas._libs.hashtable.PyObjectHashTable.get_item
  File "pandas\\_libs\\hashtable_class_helper.pxi", line 7089, in pandas._libs.hashtable.PyObjectHashTable.get_item
KeyError: 'full'

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
  File "e:\CSW\ICLR\SV\py\sv_test.py", line 24, in <module>
    (merged_df["full"].astype(str).isin(["4", "5"]))
     ~~~~~~~~~^^^^^^^^
  File "C:\Users\Rare_1\AppData\Local\Programs\Python\Python312\Lib\site-packages\pandas\core\frame.py", line 4090, in __getitem__
    indexer = self.columns.get_loc(key)
              ^^^^^^^^^^^^^^^^^^^^^^^^^
  File "C:\Users\Rare_1\AppData\Local\Programs\Python\Python312\Lib\site-packages\pandas\core\indexes\base.py", line 3812, in get_loc
    raise KeyError(key) from err
KeyError: 'full'

[Done] exited with code=1 in 26.227 seconds
