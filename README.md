import itertools
import pandas as pd

def generate_sequential_mutants(reference_primer, mutation_positions, nucleotides, exon_fixed_range):
    """
    Generate mutant primer sequences by sequentially mutating only the intronic regions while keeping exonic regions fixed.
    
    :param reference_primer: The original reference primer sequence (string)
    :param mutation_positions: List of positions (0-based index) where mutations should occur
    :param nucleotides: List of nucleotides (A, T, C, G) for mutations
    :param exon_fixed_range: Tuple (start, end) defining the range of exonic sequence that should not be mutated
    :return: List of mutant primer sequences
    """
    mutant_primers = []
    
    for pos in mutation_positions:
        if exon_fixed_range[0] <= pos < exon_fixed_range[1]:
            continue  # Skip exon positions
        
        for nt in nucleotides:
            primer = list(reference_primer)
            primer[pos] = nt  # Mutate one intronic position at a time
            mutant_primers.append("".join(primer))
    
    return mutant_primers

# Example input for int8-ex9
reference_primer_int8_ex9 = "AGTGAGCCAAGATTGTGCCACTGCACTCCAGCCTAGGCGACAGAGCAAGACTCTGTCTCAGGCATGTGTTGTTCCTCTG"
mutation_positions_int8_ex9 = [i for i in range(len(reference_primer_int8_ex9)) if not (50 <= i < 70)]  # Exclude exon range
nucleotides = ['A', 'T', 'C', 'G']  # Possible nucleotides for mutation
exon_fixed_range_int8_ex9 = (50, 70)  # Exon range (do not mutate these positions)

# Generate sequential mutant primers for int8-ex9
mutant_primers_int8_ex9 = generate_sequential_mutants(reference_primer_int8_ex9, mutation_positions_int8_ex9, nucleotides, exon_fixed_range_int8_ex9)

# Example input for ex9-int9
reference_primer_ex9_int9 = "ACGTTTGCAGAAGATGGAGGGTAAGAAAAGCATTGATTGATTTTTAACTATTAGATGAAGAATGAT"
mutation_positions_ex9_int9 = [i for i in range(len(reference_primer_ex9_int9)) if not (0 <= i < 20)]  # Exclude exon range
exon_fixed_range_ex9_int9 = (0, 20)  # Exon range (do not mutate these positions)

# Generate sequential mutant primers for ex9-int9
mutant_primers_ex9_int9 = generate_sequential_mutants(reference_primer_ex9_int9, mutation_positions_ex9_int9, nucleotides, exon_fixed_range_ex9_int9)

# Save to DataFrame
df = pd.DataFrame({
    "Mutant_Primers_int8_ex9": mutant_primers_int8_ex9,
    "Mutant_Primers_ex9_int9": mutant_primers_ex9_int9
})

# Save to CSV
output_file = "mutant_primers_sequential_fixed_exon.csv"
df.to_csv(output_file, index=False)

print(f"Generated {len(mutant_primers_int8_ex9)} sequential mutant primers for int8-ex9 and {len(mutant_primers_ex9_int9)} for ex9-int9. Saved to {output_file}")
