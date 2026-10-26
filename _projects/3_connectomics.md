---
layout: page
title: "VectorWing: Quantitative Connectomics in the Drosophila Central Complex"
description: Out-of-core connectomic graph analysis of 16.8M synapses, identifying topological edge-swap artifacts and a reciprocity inversion in the insect steering system.
importance: 3
category: research projects
related_publications: false
---

### Project Overview

How do biological brains navigate 3D environments using compact, energy-efficient recurrent circuits? In the fruit fly (*Drosophila melanogaster*), navigation and heading control are computed within the **Central Complex (CX)**.

In this student research project, I engineered an out-of-core computational connectomics pipeline to analyze whole-brain synaptic connectivity using the **FlyWire FAFB v783** electron microscopy dataset (**139,000+ neurons, 50M+ chemical synapses, and 16.8M connection records**). The study investigated the topological and chemospecific organization of reciprocal feedback in two core neuropils:
1. **The Ellipsoid Body (`EB`)**: The 1D internal heading compass.
2. **The Fan-Shaped Body (`FB`)**: The 2D allocentric vector steering matrix.

---

### Systems Engineering: Out-of-Core Processing under 380 MB RAM

Analyzing whole-brain connectomes typically demands high-memory computing clusters. I engineered this pipeline to operate strictly on commodity consumer hardware under a **peak process memory ceiling of $< 380\text{ MB}$**:
- **Streaming Ingestion**: Streamed 16.8M connection records in chunked **Apache Arrow (Feather IPC)** record batches of 65,536 rows.
- **64-Bit Integer Integrity**: Enforced native 64-bit unsigned root identifier preservation across all graph operations, eliminating silent ID truncation errors caused by double-precision conversions.
- **Automated Verification**: Built a suite of **64 automated unit tests** validating graph invariants, MCMC detailed balance, autapse exclusions, and Benjamini–Hochberg False Discovery Rate (FDR) monotonicity.

---

### Part 1: Ellipsoid Body (EB) & Deconstructing the Edge-Swap Artifact

#### Topological Discovery
At a confidence threshold of $w \ge 5$ synapses, the Ellipsoid Body is the **#1 most reciprocal neuropil in the entire fly brain** ($R_{\text{edge}} = 44.50\%$, $483$ active neurons, $15,812$ directed edges, $3,518$ mutual dyads). $74.33\%$ ($2,615$ dyads) connect GABAergic neurons.

#### The Edge-Swap Confound
Standard network configuration models (degree-preserving edge-swapping via MCMC) indicated a massive **$6.78\times$ enrichment ($Z = +142.37$)** of GABA–GABA mutual dyads.
- **The Forensic Discovery**: Edge swapping preserves in- and out-degrees but scrambles spatial clustering, causing physical edge reciprocity to collapse by **$4.61\times$** (from $44.50\%$ down to $9.66\%$). The unconditioned null model was conflating the physical abundance of reciprocal loops with transmitter-level chemical affinity.
- **The Protected Solution**: I engineered a **conditional label-permutation null model** that holds the empirical mutual dyad graph ($M = 3,518$) strictly invariant while permuting transmitter labels within discrete and continuous joint degree strata.
- **Calibrated Outcome**: Under degree-stratified permutation, the true biological association was calibrated to a **$+20.6\%$ to $+31.8\%$ conditional excess** ($Z > +13$, empirical $p = 0.0005$, Benjamini-Hochberg FDR $q < 0.002$).
- **Anatomical Substrate**: $88.18\%$ of GABA–GABA mutual dyads connect ring neurons to ring neurons, with $77.48\%$ connecting neurons of the exact same ER base subtype (ER2, ER3d, ER4d).

---

### Part 2: Fan-Shaped Body (FB) & The Reciprocity Inversion

Does the 2D vector steering matrix share the homotypic recurrent inhibition of the 1D compass?

```
   1D Heading Compass (EB)                  2D Vector Steering (FB)
   -----------------------                  -----------------------
   • 44.50% Edge Reciprocity (Rank #1)     • 23.18% Edge Reciprocity (Rank #6)
   • Dominated by Homotypic Inhibition      • GABA-GABA suppressed (0.11%, 6 dyads)
   • 74.33% GABA <-> GABA (2,615 dyads)     • Dominated by Heterotypic Feedback:
   • Ring-to-ring mutual inhibition         • ACh <-> GABA: 554 dyads (4.96x-5.92x enrichment)
   • Stabilizes compass heading bump        • 100% Strict Directional Polarity:
                                              Columnar = ACh (excitatory)
                                              Tangential = GABA (inhibitory)
```

#### 1. Discovery of the Reciprocity Inversion
- In the Fan-Shaped Body ($N = 2,465$ neurons, $E = 46,903$ edges, $M = 5,435$ mutual dyads, $R_{\text{edge}} = 23.18\%$), homotypic GABA–GABA recurrence is virtually absent ($0.11\%$, only 6 dyads).
- Instead, the circuit displays a massive **$2.53\times$ to $5.92\times$ enrichment of heterotypic $\text{ACh} \leftrightarrow \text{GABA}$ mutual feedback loops ($554$ dyads, $p = 0.0005$, FDR $q < 0.003$)**.
- Under continuous $k$-nearest-neighbor degree micro-stratification ($k \in \{8, 16\}$, up to 308 strata), the enrichment remains statistically robust at **$2.53\times$ to $2.75\times$ ($Z = +6.19$ to $+6.74$, $p = 0.000999$, 95% CI: $[2.49\times, 2.57\times]$)**.

#### 2. 100% Strict Directional Chemical Polarity
Single-cell FlyWire v2.1.0 mapping across all 2,465 nodes demonstrated:
- **$99.46\%$ ($551 / 554$)** of $\text{ACh} \leftrightarrow \text{GABA}$ mutual loops connect columnar steering neurons to tangential gating neurons.
- Directional polarity is **$100.00\%$ strict**: every columnar partner is Cholinergic (ACh, excitatory) and every tangential partner is GABAergic (GABA, inhibitory). Reverse polarity is zero.

#### 3. Premotor Concentration (Layers 4 & 5)
- **$99.10\%$ ($549 / 554$ dyads)** concentrate in premotor Layers 4 and 5 (Fisher's exact test odds ratio **$OR = 215.60, p = 3.36 \times 10^{-226}$**), dominated by `FB4A <-> vDeltaA` (150 dyads) and `FB5A <-> FC2` (272 dyads).

#### 4. Dynamical Firing-Rate Modeling
I implemented a 16-column recurrent rate simulation demonstrating that this tangential inhibitory feedback provides linear subtractive attenuation:
- **Bounds Columnar Firing**: Capping firing rates at **$30.6\text{ Hz}$** and preventing runaway excitation up to $297.4\text{ Hz}$ when tangential inhibition is ablated.
- **Preserves Spatial Contrast**: Maintains sharp heading bump tuning ($90.0^\circ$ full width at half maximum).

---

### Key Metrics Summary

| Property | Ellipsoid Body (`EB`) | Fan-Shaped Body (`FB`) |
|:---|:---:|:---:|
| **Functional Role** | 1D Heading Compass | 2D Allocentric Vector Steering |
| **Active Neurons ($N$)** | $483$ | $2,465$ |
| **Directed Synapses ($w \ge 5$)** | $15,812$ | $46,903$ |
| **Mutual Dyads ($M$)** | $3,518$ | $5,435$ |
| **Edge Reciprocity ($R_{\text{edge}}$)** | **$44.50\%$** (Rank #1 in whole brain) | **$23.18\%$** (Rank #6 in whole brain) |
| **Dominant Motif** | Homotypic $\text{GABA} \leftrightarrow \text{GABA}$ ($74.33\%$) | Heterotypic $\text{ACh} \leftrightarrow \text{GABA}$ ($10.19\%$) |
| **Statistical Enrichment** | $+20.6\%$ to $+31.8\%$ excess ($Z > +13, p = 0.0005$) | $2.53\times$ to $5.92\times$ enrichment ($Z > +6.19, p = 0.0005$) |
| **Directional Polarity** | Homotypic ring-to-ring | $100.0\%$ Columnar=ACh, Tangential=GABA |
| **Localization** | Toroidal ring layers (ER2, ER3d, ER4d) | Premotor Layers 4 & 5 ($99.10\%, OR = 215.6$) |
