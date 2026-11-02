# BLAST search and hit interpretation: human hemoglobin subunit alpha

Search a public database with a query sequence and interpret the hits.

## What I did
- **Query:** hemoglobin subunit alpha, *Homo sapiens*, RefSeq `NP_000508.1`, 142 aa (`query.fasta`).
- **Search:** NCBI BLAST web, `blastp` against `swissprot` (curated UniProtKB/Swiss-Prot).
- **Date:** 2 November 2026. **RID:** `EH27Z1N3014`.
- Parsed the NCBI text download into CSV tables and wrote an annotation table of the best matches with interpretation.

## Search parameters
| Parameter | Value |
|---|---|
| Program | blastp |
| Database | swissprot (Non-redundant UniProtKB/SwissProt sequences), posted 25 Oct 2026 |
| Organism filter | none |
| Expect threshold | 0.05 |
| Word size | 5 |
| Max target sequences (hitlist size) | 100 |
| Matrix / gap costs | BLOSUM62 / 11,1 |
| Low-complexity filter | off (filter string F) |
| Composition-based statistics | 2 (compositional matrix adjust.) |

The parameter values come from the BLAST "Search Parameters" panel of the result page. The RID, program, database and query come from the downloaded file.

## How to run / reproduce
1. Go to https://blast.ncbi.nlm.nih.gov/Blast.cgi, choose **Protein BLAST**.
2. Paste `query.fasta` (or enter `NP_000508.1`), database **Swiss-Prot**, algorithm **blastp**, leave other settings default, click **BLAST**.
3. Compare with `results/blast_alignment_EH27Z1N3014.txt` (raw NCBI output). Swiss-Prot is updated regularly, so small differences are possible.
4. Rebuild the tables from the raw file (Python 3, no extra packages):
   ```
   python scripts/make_tables.py       # results/all_hits.csv + query.fasta
   python scripts/make_annotation.py   # results/annotation_table.csv
   ```

## Files
```
query.fasta
results/blast_alignment_EH27Z1N3014.txt   raw NCBI output (alignment text download)
results/all_hits.csv                      all 100 hits parsed from the raw output
results/annotation_table.csv              best matches + interpretation
scripts/                                  parsing scripts
```

## Interpretation
- **All 100 hits are hemoglobin alpha chains** from 90 species, all mammals. E-values range from 2e-100 to about 1e-84, far below the 0.05 threshold, and query coverage is 99-100%. These are true homologs, not chance matches.
- **Top hit is the query itself** (P69905, 100% identity). The bonobo (P69906) and chimpanzee (P69907) alpha chains are also 100% identical to human; they are merged with the human hit in the alignment file.
- **Identity falls with evolutionary distance:** other apes and Old World monkeys 97-99%, New World monkeys about 96%, non-primate mammals about 82-90% (cow 88%, mouse 86%, rabbit 82%). This is the pattern expected for **orthologs** (same gene in different species).
- **Coverage of 99% is not a gap:** those subjects are 141 aa instead of 142, and alignments show 0 gaps.
- **Chimp and gorilla "alpha-3"** (P01935, P01934) are only about 89% identical, much lower than the 100% of the main chimp alpha chain. This suggests a diverged duplicate copy (**paralog**) rather than the true ortholog. This is my inference from the identity pattern, not something the output states.
- **Limitation:** with the default cap of 100 hits and hits ranked by score, the list only reached about 82% identity. More distant relatives (birds, fish, beta/delta/gamma globins, myoglobin) are not shown. Raising "Max target sequences" would reveal them.

## Decisions
- **Query:** human hemoglobin alpha is short (142 aa), well studied, and has many orthologs, which makes interpretation clear.
- **Database:** Swiss-Prot instead of nr, because it is curated and non-redundant, so each hit is a reviewed entry rather than many duplicate records.
- **Defaults kept** for all other parameters to keep the search simple and reproducible.
- **Annotation table content:** top 10 hits, the two tied identical hits, and four hits picked on purpose (chimp alpha-3, mouse, cow, rabbit) to show how identity and interpretation change with distance. The full list is in `all_hits.csv`.
