# Variant-Prioritization-miniproject-with-AlphaMissense-cBioPortal-and-ESM-models

One-day mini-project demonstrating how to connect cancer genomics with protein structural biology. I'm taking here an example of TP53 gene, one of the most mutated genes in cancer. 

## Workflow:

Start with public mutation data (here I've chosen a dataset from cbio portal)

Apply ML-based pathogenicity predictions from AlphaMissense (here I've focused only on SNPs in Alphamissense)

Generate protein embeddings with ESM-2 (for the chosen mutants)

Project embeddings with UMAP (clustering and visualization)

Map residue-level effects by placing hotspot variants in 3D context on p53's AlphaFold models

A super coarse grained objective of this min-project:
This project bridges ML + cancer genomics giving an interpretable view of somatic-mutants from a given database.

---------------------------------------------------------------------------------------------------------------------------------------------
### Summary of notebooks in this pipeline.

- Generate all possible TP53 missense variants and attach AlphaMissense scores
- Pull real somatic TP53 variants from cBioPortal and merge the scores
- Compute ESM-2 embeddings for WT + pathogenic variants
- UMAP the embeddings, QC with cosine distance to WT, and compute site-specific Δlogit at the mutated residue
- Map variants on the AlphaFold TP53 structure and color by AlphaMissense or Δlogit

Notebooks (in order to be run)
1) Tp53_all_Missense.ipynb

What it does:
-Generates all possible TP53 missense variants (HGVS p.XnY).
-Parses the AlphaMissense TSV (AlphaMissense_aa_substitutions.tsv.gz) and filters to TP53 (P04637).
-Merges to produce TP53_AlphaMissense_Annotations.csv.

Key outputs:

/AlphaMissense_ex/TP53_Missense_Variants.csv
/AlphaMissense_ex/TP53_AlphaMissense_Annotations.csv

Quick plots: class distribution & position vs. count.

2) Fetch_cbioportal_variants.ipynb

What it does:
- uses bravado + cBioPortal API to fetch MSK-IMPACT 2017 mutations (example dataset from their API page).
- filters to TP53 missense only; keeps proteinChange (p.R273H, etc).
- merges with TP53_AlphaMissense_Annotations.csv.

Key outputs:

/AlphaMissense_ex/TP53_cbioportal_mutations_annotated_with_AlphaMissense.csv

Simple EDA: AlphaMissense score histogram, top recurrent proteinChange, scatter of AlphaMissense vs. position with DNA-binding domain shaded.

3) ESM_StructuralFeatureAugmentation.ipynb

What it does:
- loads the merged cBioPortal+AlphaMissense file.
- filters to AlphaMissense “pathogenic” variants (scope to edit this filter and explore others)
- builds single-mutant sequences from the TP53 canonical reference.
- runs ESM-2 (650M by default; t6_8M works too) to get full-sequence embeddings (mean over residues).

Key outputs:

/AlphaMissense_ex/TP53_ESM2_embeddings.csv

4) UMAPs_dist_logit_analysis.ipynb

What it does:
-loads EMS embeddings and L2-normalizes them.
-computes cosine distance to WT (QC).
-runs UMAP -> 2D embedding (UMAP1/UMAP2).
-merges back AlphaMissense scores and metadata.
-computes site-specific delta(logit) at the mutated residue with ESM (mut vs WT at the exact position; a more informative local effect than whole-sequence distance).

Produces three figures:
- UMAP colored by AlphaMissense
- UMAP colored by cosine distance to WT
- UMAP colored by del(logit) at site

Key outputs:

/AlphaMissense_ex/TP53_ESM2_umap_augmented.csv
/AlphaMissense_ex/TP53_UMAP_color_by_AlphaMissense.png
/AlphaMissense_ex/TP53_UMAP_color_by_cosine_to_WT.png
/AlphaMissense_ex/TP53_UMAP_color_by_delta_logit.png

5) Visualize_sign_residues_on_p53_structure.ipynb

What it does:
- downloads AlphaFold TP53 (P04637) model.
- loads TP53_ESM2_umap_augmented.csv.
- highlights variant residues on the 3D structure with spheres colored by either del logit or AlphaMissense.
