---
layout: page
title: "Operating-Point Non-Equivalence in Continuous Language Flows (ACL 2027)"
description: "Same Perplexity, Same Entropy, Different Text: Investigating whether standard (Gen-PPL, H_ps, NFE) operating points uniquely determine continuous flow generation across 280 factorial runs."
importance: 1
category: research
related_publications: false
---

### Publication Context

- **Paper Title**: *Same Perplexity, Same Entropy, Different Text: Operating-Point Non-Equivalence of Sampler Controls in a Continuous-Latent Flow Language Model*
- **Target Venue**: **ACL 2027 Main Conference** (ACL Rolling Review / ARR Submission)
- **Model Architectures Evaluated**: Pretrained **Embedded Language Flow (ELF)** family (**ELF-B 105M**, **ELF-M 342M**, **ELF-L 652M**) alongside discrete **Masked Diffusion Language Models (MDLM 170M)** and autoregressive anchors (**GPT-2 Large**).
- **Scale**: 280 raw experiment run directories, including a balanced $4 \times 7 \times 2$ factorial grid on ELF-B, targeted intermediate-$\gamma$ evaluations, and multi-seed factorials on ELF-M.

---

### The Core Problem: Is the $(\text{Gen-PPL}, H_{\text{ps}})$ Frontier a Sufficient Statistic?

Unconditional text generation from continuous diffusion and flow-matching language models is standardly evaluated along a two-dimensional frontier:
1. **Generative Perplexity ($\text{Gen-PPL}$)** under an external autoregressive evaluator (`gpt2-large`).
2. **Mean within-sample unigram entropy ($H_{\text{ps}}$)** measuring token-level lexical diversity.

Unlike autoregressive decoders controlled by a single temperature or top-$p$ threshold, continuous-latent samplers expose multiple distinct inference-time mechanisms. In the ELF framework, generation is governed jointly by:
- **Stochastic Langevin back-noising ($\gamma$)**: Re-injects Gaussian noise and rescales the drift at each step.
- **Self-conditioning classifier-free guidance ($\text{SC-CFG}$)**: Extrapolates the velocity field away from an unconditional pass.

This research addresses a fundamental identifiability question in modern generative modeling:
> **If two inference configurations of the same continuous-latent model are matched on generative perplexity ($\text{Gen-PPL}$), average within-sample unigram entropy ($H_{\text{ps}}$), and exact per-sample function evaluations ($\text{NFE}_{\text{sample}} = K + 1$), do they produce equivalent text distributions?**

---

### Key Scientific Findings

```
                       [ Operating-Point Comparison ]
   Low Guidance / High Back-Noising     vs.     High Guidance / Low Back-Noising
      (SC-CFG = 1, High gamma)                     (SC-CFG in {3, 4}, Low gamma)
   --------------------------------             --------------------------------
   • Low within-sample repetition               • Elevated within-sample repetition (rep-4)
   • Depleted corpus-level diversity            • Elevated corpus entropy (H_pool) & distinct-2
   • Suppressed MAUVE distribution score        • Elevated MAUVE & reduced Self-BLEU
   -----------------------------------------------------------------------------
   Conclusion: The (Gen-PPL, H_ps, NFE) coordinate is NOT a sufficient statistic.
```

#### 1. Operating-Point Non-Equivalence of $(\text{SC-CFG}, \gamma)$
- Matching $\text{Gen-PPL}$, $H_{\text{ps}}$, and exact $\text{NFE}$ between low-guidance/high-back-noising ($\text{SC-CFG}=1$, high $\gamma$) and high-guidance/low-back-noising ($\text{SC-CFG}\in\{3,4\}$, low $\gamma$) leaves a highly reproducible trade-off between local sequence repetition and cross-sequence diversity.
- Across all 6 strictly matched pairs (`M1`, `S1`–`S3`, `Refined-1/2`), trading $\gamma$ for higher $\text{SC-CFG}$ systematically increases within-sample 4-gram repetition (`rep-4`; 5-seed Satterthwaite Welch $p_{\text{Holm}} \le 5.55 \times 10^{-7}$ across 5-seed pairs, $p_{\text{Holm}} \le 0.0193$ across all six pairs).
- Simultaneously, this trade-off increases pooled corpus entropy ($H_{\text{pool}}$), token-weighted inter-sample Jensen–Shannon divergence ($\mathrm{JSD}_{\pi} = H_{\text{pool}} - H_{\text{ps,weighted}}$), and corpus `distinct-2`, while reducing 4-gram `Self-BLEU` and Context-Tree Weighting Jeffreys Coding Divergence (`CTW JCD`).

#### 2. Mechanistic Decomposition via Two-Way Factorial ANOVA
To isolate the individual contributions of each control knob, we performed design-based two-way factorial ANOVA:
- **On the balanced non-collapsed ELF-B grid ($\gamma \le 2.0$)**:
  - $\text{SC-CFG}$ accounts for **$66.1\%$** of total variance in within-sample `rep-4` ($F_{3,24} = 2455.7$).
  - $\gamma$ accounts for **$85.2\%$** of total variance in corpus `distinct-2` (**$91.1\%$** across all 56 runs, $F_{6,28} = 5701.4$) and **$92.5\%$** in `MAUVE` ($F_{6,28} = 403.3$).
- **On the balanced $2 \times 2 \times 5$ ELF-M (342M) grid ($N=20$ runs)**:
  - $\text{SC-CFG}$ accounts for **$98.3\%$** of variance in `rep-4` ($F_{1,16} = 7060.3$), whereas $\gamma$ accounts for only $0.3\%$.
  - Conversely, $\gamma$ accounts for **$95.0\%$** of variance in $H_{\text{pool}}$, **$90.1\%$** in Self-BLEU, **$82.1\%$** in MAUVE, **$78.2\%$** in CTW JCD, and **$76.8\%$** in corpus `distinct-2`.
- **Spatial Scale Resolution**: $\text{SC-CFG}$ operates primarily at the local token-transition scale (driving trajectories into low-dimensional phrase attractors), whereas $\gamma$ governs cross-sequence global mode coverage.

#### 3. Deconstruction of the $\gamma^* = 3.0$ Illusion
- Previous claims in the literature suggested an anomalous generative perplexity optimum at high back-noising ($\gamma = 3.0, \text{PPL} = 5.68$), attributing divergences with corpus metrics to Jensen's inequality.
- We proved from first principles that because $e^u$ is strictly convex, Jensen's inequality dictates $\frac{1}{M}\sum e^{\bar{L}_m} \ge \exp\left(\frac{1}{M}\sum \bar{L}_m\right)$. Arithmetic sample averaging strictly *inflates* perplexity; it cannot introduce downward bias.
- Raw trajectory audits proved the true mechanism: high back-noising causes severe mode collapse into repetitive 4-gram loops ($\text{rep-4} = 78.5\%$) in unmasked post-EOS padding. Autoregressive models assign near-zero loss to repetitive loops, artificially deflating sequence loss ($\bar{L}_m \to 1.31$) while true unigram entropy collapses ($H \approx 3.42 \ll 5.15\text{ nats}$).

#### 4. Diagnostic Audit of Inference-Time Interventions (ECP & CMLC)
- Evaluated **Encoder-Cycle Projection (ECP)**—an idempotent manifold retraction operator ($\Pi = E \circ D$) that maps off-manifold flow predictions ($g_t \in [0.14, 0.59]$) back onto the discrete encoder manifold $\mathcal{M}_E$.
- In preliminary uncalibrated sweeps, ECP showed raw 16-step ODE perplexity reductions of **10.7%** ($98.61 \to 84.25$) and **+53.9% MAUVE coherence gains** ($0.445 \to 0.860$).
- Under strict entropy-matched evaluation, we demonstrated that raw Gen-PPL gains from ECP and Confidence-Modulated Latent Caching (CMLC) are predominantly driven by entropy shifts ($\le 0.2\%$ residual at matched $H_{\text{ps}}$ in 3-seed finals), providing a rigorous benchmark for future inference-time intervention studies.

---

### Empirical Summary Across Model Scales

| Model Family | Parameters | Optimal Operating Point | Gen-PPL @ $H=5.15$ | MAUVE | rep-4 | Distinct-2 |
|:---|:---:|:---:|:---:|:---:|:---:|:---:|
| **ELF-B (Ours)** | 105M | $\gamma=1.5, \text{SC-CFG}=3.0$ | **23.42** | **0.941** | 0.092 | 0.301 |
| **ELF-M (Ours)** | 342M | $\gamma=1.0, \text{SC-CFG}=3.0$ | **19.54** | **0.450** | 0.058 | 0.312 |
| **ELF-L (Ours)** | 652M | $\gamma=1.0, \text{SC-CFG}=3.0$ | **21.59** | 0.303 | 0.050 | 0.318 |
| *MDLM (Baseline)* | 170M | Discrete Diffusion (32 steps) | 65.97 | 0.412 | 0.142 | 0.245 |
| *MDLM Few-Step* | 170M | Discrete Diffusion (4 steps) | 181.86 | 0.120 | 0.994 | 0.052 |
| *ELF-B Few-Step* | 105M | Continuous Flow (4 steps) | **13.49** | **0.620** | 0.084 | 0.288 |

---

### Reproducibility & Research Rigor

- **Claim-to-Artifact Traceability**: 67 verified claims mapped in `RESULT_MAP.csv` across 280 raw experiment run directories in `run_manifest.json`.
- **Double-Blind Human Evaluation Protocol**: Executed with blinded pair exports and exact binomial / McNemar statistical tests.
- **Statistical Testing**: Two-way ANOVA, Satterthwaite Welch $t$-tests with Holm–Bonferroni correction, and Two One-Sided Tests (TOST) equivalence margins.
- **Artifacts**: 8-page main body camera-ready LaTeX manuscript (`main.tex`, `acl.sty`, `references.bib`, compiled to `main.pdf`) targeting ACL 2027 Main Conference.
