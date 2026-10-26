---
layout: page
title: "VectorWing: Whole-Brain Connectomics & Chemospecific Feedback in the Drosophila Central Complex"
description: Computational biology pipeline analyzing 16.8M synapses from the FlyWire whole-brain connectome, revealing chemospecific recurrent inhibition and steering dynamics.
importance: 3
category: research
related_publications: false
---

### Biological Motivation & Overview

How do nervous systems navigate 3D environments using compact, energy-efficient biological neural circuits? In the fruit fly (*Drosophila melanogaster*), navigation, spatial memory, and heading control are computed within the **Central Complex (CX)**.

In this student research project, I built an out-of-core computational biology pipeline to analyze whole-brain synaptic connectivity using the **FlyWire FAFB v783** electron microscopy connectome (**139,000+ neurons, 50M+ chemical synapses, and 16.8M connection records**), paired with version-pinned **FlyWire v2.1.0** cell-type annotations and deep-learning neurotransmitter predictions.

The investigation centered on dissecting the chemospecific microcircuit architecture across two primary navigation neuropils:
1. **The Ellipsoid Body (`EB`)**: The biological internal heading compass that maintains a bump attractor representing current body orientation.
2. **The Fan-Shaped Body (`FB`)**: The allocentric vector steering matrix responsible for integrating goal directions and modulating motor output.

---

### High-Throughput Connectomics Data Engineering

Processing whole-brain electron microscopy connectomes typically requires high-memory computing clusters. I engineered a streaming pipeline operating under a **strict process memory ceiling of $< 380\text{ MB}$**:
- **Streaming Connectomic Batches**: Ingested 16.8M connection records using chunked **Apache Arrow (Feather IPC)** record batches of 65,536 rows.
- **64-Bit Biological ID Integrity**: Enforced native 64-bit unsigned root identifier preservation across all synaptic mappings, eliminating silent ID truncation errors caused by standard floating-point conversions.
- **Automated Verification**: Built a suite of **64 automated unit tests** validating synaptic invariants, cell-type annotation alignments, autapse exclusions, and Benjamini–Hochberg False Discovery Rate (FDR) control.

---

### Part 1: Ellipsoid Body (EB) — The 1D Heading Compass

#### Biological Reciprocity
At a confidence threshold of $w \ge 5$ synapses, the Ellipsoid Body is the **most reciprocal neuropil in the fly brain** ($R_{\text{edge}} = 44.50\%$, $483$ active neurons, $15,812$ directed synapses, $3,518$ mutual dyads). Strikingly, $74.33\%$ ($2,615$ dyads) connect GABAergic (inhibitory) neurons.

#### Dissecting Null-Model Artifacts in Connectomics
Standard network configuration models (degree-preserving edge-swapping via MCMC) indicated an apparent **$6.78\times$ enrichment ($Z = +142.37$)** of GABA–GABA mutual dyads.
- **The Finding**: Edge-swapping destroyed spatial neuropil clustering, causing physical edge reciprocity to collapse by **$4.61\times$** (from $44.50\%$ down to $9.66\%$). The unconditioned model was conflating the physical density of reciprocal loops with genuine neurotransmitter-level chemical preference.
- **The Controlled Solution**: Formulated a **conditional label-permutation null model** holding the empirical mutual dyad architecture ($M = 3,518$) invariant while permuting transmitter labels within discrete and continuous joint degree strata.
- **Calibrated Biological Association**: Under degree-stratified permutation, the association was calibrated to a true **$+20.6\%$ to $+31.8\%$ conditional excess** ($Z > +13$, empirical $p = 0.0005$, Benjamini-Hochberg FDR $q < 0.002$).
- **Cell-Type Mapping**: Single-cell annotation mapping confirmed that **$88.18\%$ of GABA–GABA mutual loops connect ring neurons to ring neurons**, with $77.48\%$ connecting neurons of the exact same ER base subtype (ER2, ER3d, ER4d), forming the physical substrate of bump-attractor inhibition.

---

### Part 2: Fan-Shaped Body (FB) — The Reciprocity Inversion

Does the 2D vector steering matrix share the homotypic recurrent inhibition of the heading compass?

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
- In the Fan-Shaped Body ($N = 2,465$ neurons, $E = 46,903$ synapses, $M = 5,435$ mutual dyads, $R_{\text{edge}} = 23.18\%$), homotypic GABA–GABA recurrence is almost entirely absent ($0.11\%$, only 6 dyads).
- Instead, the circuit displays a massive **$2.53\times$ to $5.92\times$ enrichment of heterotypic $\text{ACh} \leftrightarrow \text{GABA}$ mutual feedback loops ($554$ dyads, $p = 0.0005$, FDR $q < 0.003$)**.
- Under continuous $k$-nearest-neighbor degree micro-stratification ($k \in \{8, 16\}$, up to 308 strata), the enrichment remains statistically robust at **$2.53\times$ to $2.75\times$ ($Z = +6.19$ to $+6.74$, $p = 0.000999$, 95% CI: $[2.49\times, 2.57\times]$)**.

#### 2. 100% Strict Directional Chemical Polarity
Single-cell FlyWire annotation matching across all 2,465 nodes demonstrated:
- **$99.46\%$ ($551 / 554$)** of $\text{ACh} \leftrightarrow \text{GABA}$ mutual loops connect columnar steering neurons to tangential gating neurons.
- Directional chemical polarity is **$100.00\%$ strict**: every columnar partner is Cholinergic (ACh, excitatory) and every tangential partner is GABAergic (GABA, inhibitory). Reverse polarity is zero.

#### 3. Premotor Layer Localization (Layers 4 & 5)
- **$99.10\%$ ($549 / 554$ dyads)** concentrate in premotor Layers 4 and 5 (Fisher's exact test odds ratio **$OR = 215.60, p = 3.36 \times 10^{-226}$**), dominated by `FB4A <-> vDeltaA` (150 dyads) and `FB5A <-> FC2` (272 dyads).

#### 4. Dynamical Firing-Rate Modeling
I constructed a 16-column continuous neural rate simulation demonstrating that this tangential inhibitory feedback provides linear subtractive attenuation:
- **Bounds Columnar Firing**: Caps firing rates at **$30.6\text{ Hz}$**, preventing runaway excitation up to $297.4\text{ Hz}$ when tangential inhibition is ablated.
- **Preserves Spatial Tuning**: Maintains sharp heading bump tuning ($90.0^\circ$ full width at half maximum) across varying navigational inputs.

---

### Comparative Neuropil Summary

| Circuit Feature | Ellipsoid Body (`EB`) | Fan-Shaped Body (`FB`) |
|:---|:---:|:---:|
| **Biological Function** | 1D Internal Compass | 2D Allocentric Vector Steering |
| **Neurons ($N$)** | $483$ | $2,465$ |
| **Directed Synapses ($w \ge 5$)** | $15,812$ | $46,903$ |
| **Mutual Dyads ($M$)** | $3,518$ | $5,435$ |
| **Edge Reciprocity ($R_{\text{edge}}$)** | **$44.50\%$** (Rank #1 in whole brain) | **$23.18\%$** (Rank #6 in whole brain) |
| **Dominant Motif** | Homotypic $\text{GABA} \leftrightarrow \text{GABA}$ ($74.33\%$) | Heterotypic $\text{ACh} \leftrightarrow \text{GABA}$ ($10.19\%$) |
| **Statistical Enrichment** | $+20.6\%$ to $+31.8\%$ excess ($Z > +13, p = 0.0005$) | $2.53\times$ to $5.92\times$ enrichment ($Z > +6.19, p = 0.0005$) |
| **Chemical Polarity** | Homotypic ring-to-ring | $100.0\%$ Columnar=ACh, Tangential=GABA |
| **Neuropil Localization** | Toroidal ring layers (ER2, ER3d, ER4d) | Premotor Layers 4 & 5 ($99.10\%, OR = 215.6$) |
| **Dynamical Mechanism** | Heading bump stabilization | Subtractive attenuation; caps firing at $\le 30.6\text{ Hz}$ |
