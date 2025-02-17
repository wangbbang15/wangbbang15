import itertools
import pandas as pd

def generate_mutant_primers(reference_primer, mutation_positions, mutation_options):
    """
    Generate mutant primer sequences based on specified mutation positions and options.
    
    :param reference_primer: The original reference primer sequence (string)
    :param mutation_positions: List of positions (0-based index) where mutations should occur
    :param mutation_options: List of lists, where each sublist contains possible nucleotides at a given position
    :return: List of mutant primer sequences
    """
    mutant_primers = []
    
    # Generate all possible mutant combinations
    for mutations in itertools.product(*mutation_options):
        primer = list(reference_primer)
        for pos, mut in zip(mutation_positions, mutations):
            primer[pos] = mut
        mutant_primers.append("".join(primer))
    
    return mutant_primers

# Example input for int8-ex9
reference_primer_int8_ex9 = "AGTGAGCCAAGATTGTGCCACTGCACTCCAGCCTAGGCGACAGAGCAAGACTCTGTCTCA"
mutation_positions_int8_ex9 = [0, 2, 5]  # Example positions to mutate
mutation_options_int8_ex9 = [['T', 'C', 'G'], ['A', 'C'], ['G', 'T']]  # Possible mutations at each position

# Generate mutant primers for int8-ex9
mutant_primers_int8_ex9 = generate_mutant_primers(reference_primer_int8_ex9, mutation_positions_int8_ex9, mutation_options_int8_ex9)

# Example input for ex9-int9
reference_primer_ex9_int9 = "ACGTTTGCAGAAGATGGAGGGTAAGAAAAGCATTGATTGATTTTTAACTATTAGATGAAG"
mutation_positions_ex9_int9 = [1, 3, 6]  # Example positions to mutate
mutation_options_ex9_int9 = [['G', 'C', 'T'], ['T', 'A'], ['A', 'C']]  # Possible mutations at each position

# Generate mutant primers for ex9-int9
mutant_primers_ex9_int9 = generate_mutant_primers(reference_primer_ex9_int9, mutation_positions_ex9_int9, mutation_options_ex9_int9)

# Save to DataFrame
df = pd.DataFrame({
    "Mutant_Primers_int8_ex9": mutant_primers_int8_ex9,
    "Mutant_Primers_ex9_int9": mutant_primers_ex9_int9
})

# Save to CSV
output_file = "mutant_primers.csv"
df.to_csv(output_file, index=False)

print(f"Generated {len(mutant_primers_int8_ex9)} mutant primers for int8-ex9 and {len(mutant_primers_ex9_int9)} mutant primers for ex9-int9. Saved to {output_file}")
