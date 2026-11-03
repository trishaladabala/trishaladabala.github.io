---
layout: page
title: "AbDev-X: Therapeutic Antibody Developability with Paired-Sequence Gaussian Processes"
description: Predicting clinical antibody developability across 5 biophysical assays using paired-sequence Gaussian processes, multiplicative Kolmogorov-Arnold networks, and OOD uncertainty triage.
importance: 2
category: research
related_publications: false
---

### Problem & Biological Motivation

Therapeutic monoclonal antibodies (mAbs) must exhibit high antigen affinity alongside favorable **developability properties**—including low self-association, minimal hydrophobic aggregation, thermal stability, low polyreactivity, and high recombinant expression yields. Failures in developability account for substantial attrition in clinical development pipelines.

In the **AbDev-X** research project, I investigated machine learning architectures for predicting five key experimental developability assays from paired variable heavy ($\text{VH}$) and light ($\text{VL}$) chain sequences:
1. **`HIC`**: Hydrophobic Interaction Chromatography retention time (measuring surface hydrophobicity; lower is better).
2. **`Tm2`**: Second thermal unfolding transition temperature ($^\circ\text{C}$; higher is better).
3. **`AC-SINS (pH 7.4)`**: Affinity-Capture Self-Interaction Nanoparticle Spectroscopy plasmon shift (measuring colloidal self-association; lower is better).
4. **`PR_CHO`**: Polyreactivity score against solubilized CHO cell membrane proteins (lower is better).
5. **`Titer`**: Recombinant expression yield in CHO cells (mg/L; higher is better).

---

### Benchmark Cohorts & Cross-Validation Protocols

- **`GDPa1` Clinical Cohort ($N=246$)**: Clinical-stage therapeutic antibodies evaluated under three strict 5-fold cross-validation protocols:
  1. **Random 5-Fold CV**: Uniform random partition.
  2. **Hierarchical Sequence-Cluster 5-Fold CV**: Agglomerative clustering at $80\%$ sequence identity threshold across concatenated $\text{VH}+\text{VL}$ sequences, ensuring whole sequence families do not leak across folds.
  3. **Cluster + IgG Isotype Stratified 5-Fold CV**: Sequence clusters jointly stratified by heavy-chain IgG subclass ($\text{IgG1}, \text{IgG2}, \text{IgG4}$) to prevent both family and isotype distribution shifts across folds.
- **`GDPa3` External Benchmark ($N=80$)**: Held-out cohort of human repertoire IgG1 antibodies used for out-of-distribution (OOD) generalization testing.

---

### Core Research Contributions & Methodological Breakthroughs

```
                      +------------------------------------------+
                      |   Paired Antibody Sequences (VH + VL)    |
                      +------------------------------------------+
                                           |
                   +-----------------------+-----------------------+
                   |                                               |
                   v                                               v
     [ Paired Sequence Kernels ]                      [ Biophysical Feature Extraction ]
     • 298-position AHo alignment                     • Physicochemical (E3, 30-D / 165-D)
     • E5: Paired Linear Sequence GP                  • Structural Surfaces (E4b, 60-D SAP/A3D)
     • E6: Paired Tanimoto Sequence GP                • Germline & Subclass (E4a, 19-D)
     • E7: Paired LOCK-GP (BLOSUM50)                               |
                   |                                               |
                   +-----------------------+-----------------------+
                                           |
                                           v
       +==================== Advanced Modeling & Fusion ====================+
       |                                                                    |
       |  1. Hybrid Product Kernel GP:                                      |
       |     k_hybrid = k_seq(VH, VL) * k_physchem(X_phys)                  |
       |                                                                    |
       |  2. MultKAN (Multiplicative KAN 2.0):                              |
       |     Cubic B-splines with biophysical product interaction nodes     |
       |                                                                    |
       |  3. Random Neighbor Score (RNS):                                   |
       |     Unsupervised k-NN OOD abstention in ESM-2/AbLang2 space        |
       |                                                                    |
       +====================================================================+
                                           |
                                           v
                       +---------------------------------------+
                       |    Calibrated Developability Assays   |
                       |    (HIC, Tm2, AC-SINS, PR_CHO, Titer) |
                       +---------------------------------------+
```

#### 1. Forensic Bug Resolution & Metric Reconciliation
- **The Upstream Subtype Swap**: Auditing baseline discrepancies revealed a silent column swap in the processed dataset: `hc_subtype` held light-chain classes (`Kappa`, `Lambda`) while `lc_subtype` held heavy-chain subclasses (`IgG1`, `IgG2`, `IgG4`).
- **Correction Impact**: Canonicalizing subtype assignments dramatically resolved negative thermal stability predictions (`Tm2` correlation shifted from **$-0.0587 \to +0.3247$**) and lifted overall `E8_AbDevX` 5-assay mean Spearman correlation on Cluster-Isotype CV from **$\bar{\rho}_{\text{fold}} = 0.3621 \to \mathbf{0.4242}$** ($\rho_{\text{pooled}} = 0.3207 \to \mathbf{0.3951}$).

#### 2. Paired Sequence LOCK-GP (`E7_LOCK_GP`)
- Designed sequence kernels over AHo-aligned variable heavy ($L_H = 149$) and light ($L_L = 149$) chains ($L = 298$ aligned positions).
- Built positive-semidefinite BLOSUM50 embeddings $\boldsymbol{\phi}(a) \in \mathbb{R}^{21}$ via eigenvalue thresholding ($\mathbf{B} = \mathbf{V}\boldsymbol{\Lambda}^+\mathbf{V}^\top$).
- Implemented **Locally Linear Correlation Kernel GP (`LOCK-GP`)**, computing per-position correlation $c_\ell(S_i, S_j) = \frac{1 + C(S_{i,\ell}, S_{j,\ell})}{2}$ modulated by learnable local ($\gamma_{\text{loc}}$) and global ($\gamma_{\text{glob}}$) exponents and separate chain weights ($\sigma_{\text{VH}}^2, \sigma_{\text{VL}}^2$).

#### 3. Multiplicative Kolmogorov–Arnold Networks (`MultKAN` / KAN 2.0)
- Formulated `MultKAN` architectures with cubic B-spline bases ($k=3, G=3$) paired with multiplicative interaction nodes ($u = z_1 \times z_2$) operating on 8–12 domain-curated biophysical features per assay (e.g., Spatial Aggregation Propensity, Eisenberg hydrophobic moment, structural Fv charge symmetry $\text{SFvCSP}$, Aggrescan3D).
- `MultKAN` achieved the **all-time best standalone performance on the held-out `GDPa3` benchmark**: **$\bar{\rho} = \mathbf{0.2556}$** (compared to the baseline `0.1918`), with exceptional precision on polyreactivity (`PR_CHO` $\rho = 0.4108$, Precision@5 $= 1.00$).

#### 4. Random Neighbor Score (RNS) for Selective Abstention
- Engineered an unsupervised out-of-distribution (OOD) score measuring $k$-NN overlap between query antibodies and synthetic randomized sequences in ESM-2 and AbLang2 latent spaces.
- When applied as a selective abstention filter on the held-out `GDPa3` benchmark, discarding the most uncertain $30\%$ of antibodies raised baseline correlation from **$0.1918 \to \mathbf{0.2412}$** ($+25.7\%$ relative improvement).

#### 5. Comprehensive 36-Family Gaussian Process Regression Study
- Evaluated 36 GP kernel formulations across conventional kernels, Automatic Relevance Determination (ARD), 5-group additive kernels (`GP_FeatureGroup_Additive`), CDR/Framework variance partitioning, and hybrid sequence $\times$ physicochemical product kernels (`GP_Ab_Hybrid_Product`).
- Hybrid sequence-physicochemical product GPs achieved the **highest overall 5-assay correlation on `GDPa1` Cluster-Isotype CV**: **$\bar{\rho}_{\text{fold}} = \mathbf{0.4449}$ / $\rho_{\text{pooled}} = \mathbf{0.4315}$**, demonstrating that continuous Gaussian Process priors effectively regularize sequence representations against out-of-distribution cluster shifts.

---

### Grand Benchmark Summary: 5-Assay Mean Spearman $\rho$

| Evaluation Protocol | Baseline `E8_AbDevX` *(Corrected)* | Best Standalone Model | Best Fused Stacking | Best Under Selective Abstention |
|:---|:---:|:---:|:---:|:---:|
| **`GDPa1` Cluster-Isotype 5-Fold CV** | `0.4242` fold / `0.3951` pool | **`0.4351` fold / `0.4056` pool** (`Hybrid_Product_StrictPSD` GP) | **`0.4449` fold / `0.4315` pool** (`Fuse_HybridSeqPhys_Weighted`) | — |
| **`GDPa1` Hierarchical Cluster 5-Fold CV** | `0.4148` fold / `0.3554` pool | **`0.4143` fold / `0.3925` pool** (`GP_Ab_Hybrid_Product`) | **`0.4284` fold / `0.3727` pool** (`Fuse_HybridSeqPhys_Stacking`) | — |
| **`GDPa1` Random 5-Fold CV** | `0.4287` fold / `0.3992` pool | **`0.4379` fold / `0.4315` pool** (`GP_Ab_Hybrid_Product`) | **`0.4517` fold / `0.4328` pool** (`Fuse_FeatureGroupAdd_Weighted`) | — |
| **Held-Out `GDPa3` External Benchmark ($N=80$)** | `0.1918` | **`0.2556`** (`MultKAN`, KAN 2.0) | **`0.2093`** (`AbDevX + MultKAN`) | **`0.2412`** (`RNS` @ 70% coverage)<br>**`0.2249`** (`GP Posterior` @ 90% coverage) |

---

### Best Per-Assay Performance Highlights

- **Hydrophobicity (`HIC`)**: `0.5196` fold / `0.5128` pool on Cluster-Isotype CV (`GP_Ab_Hybrid_Additive`), reaching **`0.5444`** on held-out `GDPa3` ($0.5582$ under 90% confidence retention).
- **Thermal Stability (`Tm2`)**: `0.4188` fold / `0.3297` pool on Cluster-Isotype CV (`Subtype_5feat_RidgeCV`), reaching **`0.3567`** on held-out `GDPa3`.
- **Self-Association (`AC-SINS pH 7.4`)**: `0.4755` pool on Cluster-Isotype CV (`GP_TabPCA20_Matern32_plus_RQ`) and **`0.4959`** on Cluster CV.
- **Polyreactivity (`PR_CHO`)**: **`0.6198` fold / `0.6143` pool** on Cluster-Isotype CV (`Fuse_HybridSeqPhys_Weighted`), with **`0.4108`** (Precision@5 $= 1.00$) on held-out `GDPa3`.
- **Expression Yield (`Titer`)**: `0.3063` fold on Cluster-Isotype CV (`AbDevX + MultKAN`) and **`0.4030`** on Random CV.

---

### Technical Stack & Verification
- **Languages & Libraries**: Python 3.12, GPyTorch, PyTorch, scikit-learn, SciPy, NumPy, LightGBM, Pandas.
- **Reproducibility**: Complete self-contained specification (`ABDEVX_COMPLETE_PROJECT_REPLICATION_AND_RESULTS_DOSSIER.md`), deterministic cross-validation splits, exact Holm–Bonferroni / Benjamini–Hochberg FDR multiple-testing corrections, and cross-conformal uncertainty calibration.
