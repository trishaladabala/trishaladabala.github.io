---
layout: page
title: "QuantPPI: Data-Oblivious Vector Quantization for Protein Embeddings"
description: Beating trained FAISS OPQ on 1280-D protein language model embeddings with high-dimensional geometry and zero calibration data.
importance: 2
category: research projects
related_publications: false
---

### Problem: The 512 GB Embedding Wall in Structural Biology

Modern Protein Language Models (such as Meta's ESM-2 650M) represent biological sequences as dense 1280-dimensional continuous embeddings ($D = 1280$). At standard FP32 precision, storing a single embedding consumes **5,120 bytes**.
- A standard database of 100 million proteins (such as UniProt) requires **512 GB of raw RAM** before secondary index structures.
- Industry-standard solutions like **FAISS OPQ (Optimized Product Quantization)** suffer from heavy drawbacks:
  1. **Expensive Offline Training**: Requires several minutes of iterative $k$-means clustering across thousands of sample vectors.
  2. **Codebook Overhead**: A shared rotation and centroid header of **6.88 MB to 8.26 MB**, which inflates per-vector memory by $+540\text{ B/vec}$ at moderate scales.
  3. **Out-of-Distribution Fragility**: Severe recall degradation ($0.996 \to 0.963$) when deployed on evolving biological sequence databases.
- Meanwhile, prior **data-oblivious** quantization techniques (LSH, RaBitQ, TurboQuant) hit an empirical accuracy ceiling of **$\sim 0.76$ Recall@20@30**.

---

### Geometric Root Cause: Rogue Dimensions & Orthogonal Smearing

By conducting a geometric audit across multiple PLM families (ESM-2, ProtT5, SaProt), I identified a severe geometric pathology:
- **Extreme Coordinate Anisotropy**: The normalized Participation Ratio is $\mathrm{PR}/D = 0.0012$, with a Peak-to-RMS crest factor of $31.9$.
- **Energy Concentration**: Just **2 to 4 "rogue coordinates" contain over 83% of the vector's total squared Euclidean energy**.
- **Orthogonal Smearing Penalty**: Standard data-oblivious methods apply a random orthogonal rotation $R \in O(D)$ to balance coordinate variances. However, rotating these rogue coordinates **smears their massive energy uniformly across all 1280 dimensions**, inflating background quantization noise by:
  $$\frac{\sigma_{\text{smeared}}}{\sigma_{\text{isolated}}} = \sqrt{\frac{1.0}{0.15}} \approx \mathbf{2.58\times}$$
This $2.6\times$ noise inflation drowns out fine-grained biological signals separating true structural homologs from background noise.

---

### The Theoretical Solution: The Three Pillars

```
   Input Vector x (||x|| = 1)
          |
          +---> Dynamic Top-k Outliers (k=2 or 4) ------> [Indices: uint16] + [Values: FP16]
          |                                                         |
          +---> Disjoint Trunk z (zeros on outliers)                |  Norm recovered for free:
                   |                                                |  ||z|| = sqrt(1 - sum v_m^2)
                   v                                                |
          [Circulant FFT Rotation: y = R(z / ||z||)]                |
                   |                                                |
                   v                                                |
          [Spherical Lloyd-Max Quantization: u_hat]                 |
                   |                                                |
                   v                                                v
   Fast ADC Scoring Engine: <Rq, beta* ||z|| u_hat> + sum(q_m * v_tilde_m)
   --------------------------------------------------------------------------------------
   • Theorem 1 (Bleed Elimination): Eliminates cross-support noise leakage.
   • Theorem 2 (Bussgang De-Attenuation): Restores unbiased inner products via beta* = 1 / rho_Q.
```

#### Pillar 1: Disjoint Split & Zero-Overhead Norm Recovery
We dynamically isolate the top-$k$ ($k \in \{2, 4\}$) outlier coordinates per vector ($x_\parallel$), leaving a quiet residual trunk ($z$). Because protein embeddings are unit-normalized ($\|x\|_2^2 = \|x_\parallel\|_2^2 + \|z\|_2^2 = 1$), the decoder recovers the Euclidean norm of the trunk with **zero wire bytes**:
$$\|z\|_2 = \sqrt{\max\left(1 - \sum_{m \in S} v_m^2, \; 0\right)}$$

#### Pillar 2: Theorem 1 (Support-Orthogonal Bleed Elimination)
Even though the true trunk $z$ is strictly zero on outlier indices $S$, rotating, quantizing, and back-rotating ($R^\top \hat{u}$) injects random quantization noise back onto $S$. Multiplying these noisy coordinates by the query's own spikes degrades ranking.
- **Theorem 1** establishes an algebraic identity that projects out this noise at index-build time:
  $$\tilde{v}_{i,m} \triangleq v_{i,m} - \|z_i\|_2 (R^\top \tilde{u}_i)_m$$
This precomputes the correction directly into stored outlier values, enabling query-time search in $O(ND + Nk)$ time with zero bleed noise.

#### Pillar 3: Theorem 2 (Analytical Bussgang De-Attenuation)
Quantizing Gaussian coordinates shrinks their dot-product correlation with the query by a factor $\rho_Q < 1$. Leaving this uncorrected under-weights the biological trunk relative to the FP16 outliers.
- **Theorem 2** derives the exact closed-form scalar multiplier $\beta_Q^* = 1/\rho_Q$ from Bussgang's Theorem for Gaussian processes (e.g., $\beta_1^* = 1.2533$ for 1-bit; $\beta_5^* = 1.0425$ for quinary), restoring an exact, unbiased dot-product estimator without requiring any training data.

---

### Codec Engineering & Multi-Radix Bit-Packing

To eliminate padding bits without wasting storage, I implemented custom lossless multi-radix bit-packing formats:
- **Radix-5 Quinary (`382 B`)**: Every 3 quinary symbols pack bijectively into 7 bits ($5^3 = 125 < 128 = 2^7$). 1280 dimensions pack into $374\text{ B}$ trunk $+ 8\text{ B}$ outliers = **382 B/vec**.
- **Radix-6 Senary (`432 B`)**: Every 5 senary symbols pack into 13 bits ($6^5 = 7{,}776 < 8{,}192 = 2^{13}$). 1280 dimensions divide into 256 exact quintuplets $\implies 416\text{ B}$ trunk $+ 16\text{ B}$ outliers = **432 B/vec** (zero padding).
- **3-Bit Lloyd-Max (`488 B`)**: 8 symbols in 3 bytes ($480\text{ B}$ trunk $+ 8\text{ B}$ outliers) = **488 B/vec**.

---

### Empirical Benchmarks: Beating Trained OPQ

Evaluated on canonical **SCOPe ASTRAL-40** ($N=15{,}166$, $Q=500$, ESM-2 650M, $D=1280$) and validated on **Swiss-Prot Human** ($N=13{,}003$, $Q=1{,}000$):

| Method | Type | Fit / Train Time | Total RAM / Vec ($N=15\text{k}$) | Recall@20@30 | Top-1 Recall (R@1) | Compression Ratio |
|:---|:---:|:---:|:---:|:---:|:---:|:---:|
| **Exact Dense FP32** | Reference | 0.0 s | 5,120 B | 1.0000 | 1.0000 | $1.0\times$ |
| **QuantPPI 3-bit Hybrid** | **Data-Oblivious** | **0.0 s** | **488 B** | **0.9993** | **0.9600** | **$10.5\times$** |
| **QuantPPI Senary Hybrid** | **Data-Oblivious** | **0.0 s** | **432 B** | **0.9981** | **0.9200** | **$11.9\times$** |
| **QuantPPI Quinary Hybrid** | **Data-Oblivious** | **0.0 s** | **382 B** | **0.9962** | **0.9100** | **$13.4\times$** |
| **QuantPPI Top-4 1-bit** | **Data-Oblivious** | **0.0 s** | **176 B** | **0.9437** | **0.8000** | **$29.1\times$** |
| *FAISS Learned OPQ (M=224)* | Trained SOTA | 283.0 s | 787.3 B | 0.9968 | 0.8980 | $6.5\times$ |
| *FAISS Unrotated PQ (M=224)* | Trained Baseline | 14.0 s | 314.7 B | 0.9628 | 0.8100 | $16.3\times$ |
| *RaBitQ 1-bit* | Prior Data-Oblivious | 0.0 s | 164 B | 0.7645 | 0.6400 | $31.2\times$ |

#### Key Takeaways:
1. **Surpasses Trained FAISS OPQ**: The 3-bit Hybrid achieves **`0.9993` Recall@20@30** (vs. OPQ's `0.9968`, paired bootstrap $p = 4.0 \times 10^{-8}$) and **`0.9600` Top-1 Recall** (+4.2% to +6.2% over OPQ).
2. **Half the Amortized Memory**: At $N=15\text{k}$, OPQ consumes `787.3 B/vec` due to its 8.26 MB codebook. Our Quinary codec consumes only **`382 B/vec` ($2.06\times$ reduction)**.
3. **Zero Training Latency**: Encodes streaming vectors on the fly with **0.0 seconds of training**, eliminating the 4.7-minute $k$-means calibration phase of OPQ.
4. **SIMD Kernel Acceleration**: Implemented Asymmetric Distance Computation in **C with ARM NEON SIMD vectorization**, achieving $0.35\text{ ms/query}$ latency and $234{,}000\text{ vectors/sec}$ encoding throughput.
