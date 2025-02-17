import itertools
import pandas as pd

def generate_mutant_primers(reference_primer, mutation_positions, mutation_options, exon_fixed_range):
    """
    Generate mutant primer sequences where only intronic regions are mutated, keeping exonic regions fixed.
    
    :param reference_primer: The original reference primer sequence (string)
    :param mutation_positions: List of positions (0-based index) where mutations should occur
    :param mutation_options: List of lists, where each sublist contains possible nucleotides at a given position
    :param exon_fixed_range: Tuple (start, end) defining the range of exonic sequence that should not be mutated
    :return: List of mutant primer sequences
    """
    mutant_primers = []
    
    # Generate all possible mutant combinations
    for mutations in itertools.product(*mutation_options):
        primer = list(reference_primer)
        for pos, mut in zip(mutation_positions, mutations):
            # Only mutate intronic positions, not exonic regions
            if not (exon_fixed_range[0] <= pos < exon_fixed_range[1]):
                primer[pos] = mut
        mutant_primers.append("".join(primer))
    
    return mutant_primers

# Example input for int8-ex9
reference_primer_int8_ex9 = "AGTGAGCCAAGATTGTGCCACTGCACTCCAGCCTAGGCGACAGAGCAAGACTCTGTCTCAGGCATGTGTTGTTCCTCTG"
mutation_positions_int8_ex9 = [0, 2, 5, 15, 25, 30]  # Example positions in intron
mutation_options_int8_ex9 = [['T', 'C', 'G'], ['A', 'C'], ['G', 'T'], ['A', 'G'], ['C', 'T'], ['G', 'A']]  # Possible mutations
exon_fixed_range_int8_ex9 = (50, 70)  # Exon range (do not mutate these positions)

# Generate mutant primers for int8-ex9
mutant_primers_int8_ex9 = generate_mutant_primers(reference_primer_int8_ex9, mutation_positions_int8_ex9, mutation_options_int8_ex9, exon_fixed_range_int8_ex9)

# Example input for ex9-int9
reference_primer_ex9_int9 = "ACGTTTGCAGAAGATGGAGGGTAAGAAAAGCATTGATTGATTTTTAACTATTAGATGAAGAATGAT"
mutation_positions_ex9_int9 = [0, 3, 6, 12, 20, 25]  # Example positions in intron
mutation_options_ex9_int9 = [['G', 'C', 'T'], ['T', 'A'], ['A', 'C'], ['G', 'T'], ['A', 'C'], ['G', 'T']]  # Possible mutations
exon_fixed_range_ex9_int9 = (0, 20)  # Exon range (do not mutate these positions)

# Generate mutant primers for ex9-int9
mutant_primers_ex9_int9 = generate_mutant_primers(reference_primer_ex9_int9, mutation_positions_ex9_int9, mutation_options_ex9_int9, exon_fixed_range_ex9_int9)

# Save to DataFrame
df = pd.DataFrame({
    "Mutant_Primers_int8_ex9": mutant_primers_int8_ex9,
    "Mutant_Primers_ex9_int9": mutant_primers_ex9_int9
})

# Save to CSV
output_file = "mutant_primers.csv"
df.to_csv(output_file, index=False)

print(f"Generated {len(mutant_primers_int8_ex9)} mutant primers for int8-ex9 and {len(mutant_primers_ex9_int9)} mutant primers for ex9-int9. Saved to {output_file}")
