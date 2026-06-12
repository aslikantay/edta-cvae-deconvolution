# EDTA-CVAE: Brain Cell Type Deconvolution

A deep learning pipeline for deconvolving bulk RNA-seq data into cell type proportions using a Conditional VAE with MMD regularization and supervised contrastive loss for bulk RNA-seq deconvolution.

---

## Overview

This project predicts brain cell type composition from bulk RNA-seq data by training on single-cell RNA-seq reference data (Allen Human MTG), augmenting rare cell types with a CVAE, and validating against IHC ground truth and a real-world AD dataset (GSE153873).

**Cell types modelled:** Exc_L2_3, Exc_Deep, Inh_PVALB, Inh_SST, Inh_VIP, Astrocyte, Oligodendrocyte, Endothelial, Microglia

---

## Pipeline Structure

```
Cell 2  → AnnData construction + subclass mapping + filtering
Cell 3  → Stratified donor split (train / val / test)
Cell 4  → Marker gene selection (Wilcoxon + specificity + donor CV)
Cell 5  → WAE-MMD CVAE architecture
Cell 6  → CVAE training with contrastive regularisation (ablation: 32D vs 48D latent)
Cell 7  → Synthetic cell generation + biological quality validation
Cell 8  → Pseudo-bulk construction with IHC-target alpha blending
Cell 9  → DeconvNet training (Baseline + Augmented) + isotonic calibration
Cell 9b → Aug-guarantee per-class blend
Cell 10 → Evaluation: CCC / R² / MAE + stress tests + TTA
Cell 11 → 8-panel visualisation report
Cell 12 → Latent space UMAP
Cell 13 → Summary report
Cell 14 → GSE153873 download and loading
Cell 15 → Metadata + gene alignment + normalisation
Cell 16 → Model predictions on real bulk data
Cell 17 → Statistical testing (AD vs Old vs Young, Cohen's d, Bonferroni)
Cell 18 → GSE153873 visualisation
Cell 19 → Baseline vs Augmented difference analysis
Cell 20 → GSE153873 summary report
Cell 21 → Bootstrap CCC confidence intervals + Wilcoxon test
Cell 22 → IHC validation bootstrap test
Cell 23 → IHC-level per-class adaptive blend
Cell 24 → Bootstrap test + visualisation (Blend vs Base)
```

---

## Key Methods

| Component | Detail |
|---|---|
| Reference data | Allen Human MTG (scRNA-seq, ~3000 HVG) |
| Validation data | GSE153873 (bulk RNA-seq, lateral temporal lobe, AD/Old/Young) |
| IHC ground truth | CortexCellDeconv (Neuronal, Astrocyte, Oligodendrocyte, Endothelial, Microglia) |
| Augmentation | WAE-MMD CVAE with NT-Xent contrastive loss, prior orthogonalisation |
| Deconvolution model | ResNet-style DeconvNet, weighted MSE + KL loss |
| Calibration | Per-class isotonic regression |
| Inference | 4-view domain alignment, TTA (n=8), adaptive blend |
| Evaluation metric | CCC (concordance correlation coefficient), R², MAE |

---

## Reproducing the Results

### 1. Open in Google Colab

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1A4V1bJSVKSs5zlmBmWcWtqtLMpGNyWFo)

### 2. Install dependencies

Run Cell 1 (all installs are handled in the first cell).

### 3. Run all cells in order

Cells are numbered and self-contained. Each cell prints its own status output. Total runtime on a T4 GPU: ~45–60 minutes.

### 4. Data

Data is downloaded automatically by the notebook from:
- Allen Brain Map MTG (Cell 1)
- NCBI GEO: GSE123496 (Cell 1)
- NCBI GEO: GSE153873 (Cell 14)
- CortexCellDeconv IHC files must be placed manually in `/content/data/` (see below)

---

## IHC Validation Files

Place the following files in `/content/data/` before running Cell 14 onwards:

```
geneExpr.txt
IHC.neuro.txt
IHC.astro.txt
IHC.oligo.txt
IHC.endo.txt
IHC.microglia.txt
```

These files are from the [CortexCellDeconv dataset](https://github.com/ellispatrick/CortexCellDeconv).

---

## Output Files

| File | Description |
|---|---|
| `results_v9.csv` | Per-class CCC/R²/MAE for all models |
| `full_report_v8.png` | 8-panel training and evaluation figure |
| `umap_v7.png` | Latent space UMAP (real vs synthetic) |
| `gse153873_validation.png` | Real bulk RNA-seq validation figure |
| `gse153873_stats.csv` | Statistical test results (AD vs Old vs Young) |
| `gse153873_predictions.csv` | Per-sample cell type predictions |
| `gse153873_summary.json` | Summary metrics |
| `realworld_validation_v3.png` | IHC CCC comparison figure |
| `ihc_adaptive_blend_analysis.png` | Adaptive blend analysis figure |

---

## Reproducibility

- Random seed fixed at `SEED = 42` throughout
- Donor-stratified splits ensure no data leakage across train/val/test
- All synthetic data is generated within the notebook; no external files needed beyond the above

---

## Dependencies

```
anndata
scanpy
torch
torchvision
scipy
statsmodels
matplotlib
seaborn
umap-learn
pandas
numpy
h5py
scikit-learn
```

All installed via Cell 1.

---

## Biological Hypothesis

Alzheimer's disease is associated with neuronal loss (particularly Exc_L2_3 and Exc_Deep) and relative preservation or increase of glial populations. This pipeline tests that hypothesis using Cohen's d effect sizes and Mann-Whitney U tests on predicted cell type proportions in lateral temporal lobe bulk RNA-seq (GSE153873).

---

## Citation / Acknowledgements

- Allen Brain Map: [celltypes.brain-map.org](https://celltypes.brain-map.org)
- GSE153873: Bulk RNA-seq of lateral temporal lobe, AD/Old/Young donors
- CortexCellDeconv IHC reference: Patrick et al.
