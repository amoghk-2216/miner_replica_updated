# gbmMINER Replication

Replication of the gbmMINER transcriptional regulatory network for glioblastoma multiforme (GBM), as described in:
'An atlas of causal and mechanistic drivers of interpatient heterogeneity in glioma'
https://www.medrxiv.org/content/10.1101/2024.04.05.24305380v1.full

---

## Overview

gbmMINER applies the MINER (Mechanistic Inference of Node-Edge Relationships) algorithm to multi-omics data from 547 TCGA GBM patients to construct a causal and mechanistic TRN. 

The network maps somatic mutations via TFs and miRNAs to disease-relevant co-expressed gene modules (regulons), transcriptional programs, and patient states.

This repository contains the full replication pipeline including data preprocessing, MINER execution, downstream analysis, and independent validation.

---

## Key Results

| Metric                    | Paper     | This Replication |
|------------------------   |-------    |----------------- |
| Co-expression clusters    | ~500 est. | 502              |
| Regulon modules           | 3,797     | 4,504            |
| Transcriptional states    | 23        | 23               |
| Transcriptional programs  | 179       | 175              |
| Disease-relevant regulons | 1,083     | 836 (refining)   |
| Causal mutations          | 30        | 28               |
| Causal TFs                | 306       | 254              |

---

## Data

### Training Cohort
- **TCGA GBM**: 547 patients, 10,383 genes
- Expression: Combined microarray (266 patients) + RNA-seq (284 patients), z-scored per gene, per-sample mean centered
- Mutations: 35 genes × 259 patients (binary, 5% frequency filter)
- Clinical: 545 patients, median OS 12.2 months, 461 deaths

### Validation Cohort
- **Gravendeel (GSE16011)**: 284 CEL files, HGU133 Plus 2.0, RMA normalized with ENTREZG CDF, z-scored, per-sample mean centered

Large data files are not included in this repository due to size constraints. Raw data is available from:
- TCGA: [GDC Data Portal](https://portal.gdc.cancer.gov/)
- Gravendeel: [GEO GSE16011](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE16011)

---

## Repository Structure

```
gbmMINER-replication/
├── notebooks/
│   ├── 01_preprocess_expr.ipynb       # TCGA expression preprocessing
│   ├── 02_preprocess_mirna.ipynb      # miRNA preprocessing
│   ├── 03_run_miner.ipynb             # MINER pipeline execution
│   └── 04_downstream_analysis.ipynb   # Cox regression, validation (TO BE UPLOADED)
|
├── scripts/ (TO BE UPLOADED)
│   ├── gravendeel_preprocessing.R     # RMA normalization with ENTREZG CDF
│   ├── topgo_enrichment.R             # GO BP enrichment per regulon
│   └── hallmark_similarity.R          # Jiang-Conrath hallmark enrichment
├── data/
│   ├── miner_input/
│   │   ├── clinical_data.csv
│   │   └── mutation_matrix.csv
│   └── miner_output_results/
│       ├── output_diagrams/
|       └── cluster_preview.pdf
|       └── gene_variance_dist.pdf
|       └── mosaic_all.pdf
│       └── mosaic_activity_heatmap.pdf
│       └── programs_vs_states.pdf
│       └── cluster_preview.pdf
│       └── regulon_activity_heatmap.pdf
│       └── sample_means_before_fix.pdf
│       └── transcriptional_programs.pdf
│    └── miner_output_jsons/
│       └── coexpressionDictionary.pdf
│       ├── coexpressionModules.json
│       └── coregulationModules.json
│       └── mechanisticOutput.json
│       └── regulons.json
│       └── transcriptional_programs.json
│       └── transcriptional_states.json
└── README.md
```

---

## Pipeline

### 1. Expression Preprocessing
- Microarray: RMA normalized, Ensembl ID mapping, z-scored
- RNA-seq: TMM normalized, z-scored
- Combined: per-gene z-score, per-sample mean centering (critical for batch artifact removal)

### 2. MINER Pipeline
```
cluster() → reviseInitialClusters() → mechanisticInference() → 
networkQuantization() → transcriptionalPrograms() → transcriptionalStates() → 
causalInference() → Cox regression
```

### 3. Independent Validation
- Co-expression validation in Gravendeel: PC1 variance ≥ 0.3 and empirical p ≤ 0.05 (500 permutations)
- Survival validation: Cox HR p ≤ 0.05 in TCGA
- Hallmark enrichment: GO BP enrichment against Hanahan & Weinberg 2011 hallmarks

### Requirements

### Python
See requirements.txt

### R
```
affy
affyio
makecdfenv
topGO
org.Hs.eg.db
GOSemSim
```



## References

1. Warsow et al. (2024). gbmMINER. *medRxiv*. https://doi.org/10.1101/2024.04.05.24305380
2. Plaisier et al. (2016). gbmSYGNAL. *Cell Systems*. https://doi.org/10.1016/j.cels.2016.06.006
3. Hanahan & Weinberg (2011). Hallmarks of Cancer. *Cell*. https://doi.org/10.1016/j.cell.2011.02.013
4. Gravendeel et al. (2009). *Cancer Research*. https://doi.org/10.1158/0008-5472.CAN-09-2307
