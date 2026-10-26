---
layout: page
title: "Continuous Language Flows: Manifold Geometry & Causal Clocks"
description: Investigating manifold drift retraction (ECP), causal wave-clock schedules, and trajectory distillation in continuous-latent language flow matching.
importance: 1
category: research projects
related_publications: false
---

### Overview & Motivation

Standard autoregressive language models (such as GPT architectures) generate text token-by-token. For a document of length $L$, this requires $L$ sequential forward passes, creating a fundamental inference latency bottleneck.

**Continuous-latent language flow matching** offers an alternative non-autoregressive paradigm: text generation is cast as integrating an Ordinary Differential Equation (ODE) that transports probability mass from an isotropic Gaussian prior in latent embedding space ($\mathbb{R}^d$) to structured contextual text embeddings in $N$ parallel steps ($N \ll L$).

In this independent student research project, I conducted an empirical and theoretical investigation of continuous flow matching across **105M (`ELF-B`), 342M (`ELF-M`), and 652M (`ELF-L`)** models, addressing three critical bottlenecks in the continuous diffusion literature:
1. **Evaluation confounds** arising from post-EOS repetition mode collapse and Jensen's inequality misinterpretations.
2. **Off-manifold trajectory drift** during intermediate ODE integration steps.
3. **Contextual noise leakage** in asynchronous token clock schedules.

---

### Key Research Contributions

#### 1. Deconstructing the Evaluation Anomaly & Establishing the $PPL@H=5.15$ Pareto Frontier
- **The Literature Dilemma**: Prior work reported an apparent perplexity minimum under aggressive stochastic back-noising ($\gamma = 3.0, \text{PPL} = 5.68$), attributing discrepancies with corpus-level perplexity to Jensen's inequality.
- **Mathematical Clarification**: Because $e^u$ is strictly convex, Jensen's inequality proves $\frac{1}{M}\sum e^{\bar{L}_m} \ge \exp\left(\frac{1}{M}\sum \bar{L}_m\right)$. Arithmetic sample averaging *inflates* perplexity; it cannot introduce downward bias.
- **Underlying Mechanism**: Auditing raw token trajectories revealed that high back-noising caused the denoiser to collapse into repetitive 4-gram loops ($\text{rep-4} = 78.5\%$) in unmasked post-EOS padding. Because autoregressive evaluation models easily predict repeating patterns, sequence loss collapsed ($\bar{L}_m \to 1.31$) while true unigram entropy tanked ($H \approx 3.1 \ll 5.15\text{ nats}$).
- **Calibrated Benchmarking**: Formulated an 82-run grid evaluating models on the true Pareto frontier at natural unigram entropy ($H = 5.15\text{ nats}$). Under calibrated conditions at $N=4$ steps, continuous flow matching outperforms 130M discrete masked diffusion (`MDLM`) by **$13.5\times$ in perplexity** ($13.49$ vs. $181.86$ dedup-PPL) while running **$3.2\times$ faster** in wall-clock time ($4.2\text{ s}$ vs. $13.7\text{ s}$).

```
                      +---------------------------------------+
                      |       Input Prompt / Source Text      |
                      +---------------------------------------+
                                          |
                                          v
                      +---------------------------------------+
                      |       Frozen T5-Small Encoder         |
                      +---------------------------------------+
                                          |
                                          v
                           Initial Latents z_0 ~ N(0, I)
                                          |
                                          v
   +==================== Iterative Probability Flow ODE ====================+
   |                                                                        |
   |   Local Clocks:  tau_i(t) = clamp(W * t - w_i, 0, 1)                   |
   |   CME Mask:      M_ij = 1[ q(tau_j) >= q(tau_i) ]                      |
   |                                                                        |
   |   [ Denoiser Network v_theta(z_t, tau, cond) ]                         |
   |                     |                                                  |
   |                     v                                                  |
   |   Self-Cond Channel: Retraction Operator Pi = E o Decoder(z_t)         |
   |                     |                                                  |
   |                     v                                                  |
   |   Euler / Heun Step: z_{t + dt} = z_t + dt * v_theta                   |
   |                                                                        |
   +========================================================================+
                                          |
                                          v
                      +---------------------------------------+
                      |     Frozen Projection Decoder         |
                      +---------------------------------------+
                                          |
                                          v
                      +---------------------------------------+
                      |   Final Tokens (Argmax + EOS Mask)    |
                      +---------------------------------------+
```

#### 2. Manifold Geometry & Encoder-Cycle Projection (ECP)
- **Measuring Off-Manifold Drift**: Defined the encodability gap $g_t$ between the continuous velocity prediction $\hat{x}_1(z_t)$ and the discrete text manifold $\mathcal{M} = \{E(w)\}$:
  $$g_t = \frac{1}{D}\|\hat{x}_1(z_t) - E(\arg\max D_{\text{dec}}(\hat{x}_1(z_t)))\|_2^2$$
  Intermediate trajectories stray **$40\times\text{--}120\times$ off-manifold** ($g_t \in [0.14, 0.59]$ vs. text floor $g_{\text{ref}} = 0.0045$).
- **Idempotent Retraction Operator**: Formulated $\Pi = E \circ \arg\max D_{\text{dec}}$. Since $D(E(w)) = w$, the operator is idempotent ($\Pi \circ \Pi = \Pi$), projecting continuous latents back onto the valid text representation manifold.
- **Noise-Preserving Formulation (ECP-Z)**: Derived an update that avoids corrupting the implied velocity direction:
  $$z_{t'} \leftarrow z_{t'} + \lambda t'(\Pi(\hat{x}) - \hat{x})$$
- **Empirical Results**: Reduced 16-step ODE perplexity by **10.7%** ($98.61 \to 84.25$) and improved **MAUVE distributional coherence by +53.9%** ($0.445 \to 0.860$). A matched-norm Gaussian perturbation control confirmed that gains stem strictly from discrete manifold retraction.

#### 3. The Wave-Step Resolution Theorem & Causal Clocks
- **The Wave-Step Resolution Theorem**: For a sequence partitioned into $W$ waves with piecewise-linear local times $\tau_i(t) = \text{clamp}(W \cdot t - w_i, 0, 1)$, the effective number of integration steps per active wave interval is:
  $$N_{\text{eff}} = \frac{N}{W}$$
  *Significance*: When $N < W$, $N_{\text{eff}} < 1$, causing local time to jump from noise ($\tau_i = 0$) to text ($\tau_i = 1$) in a single step and inducing unigram entropy collapse ($3.97 \to 0.65\text{ nats}$). Stable integration strictly requires **$N \ge 2W$ global integration steps**.
- **Clock-Modulated Encoder (CME) Masks**: Linear probing revealed that bidirectional encoders leak uncommitted noisy tokens to clean committed tokens ($43.2\%$ neighbor decodability at $\delta = -1$). CME masking enforces $M_{ij} = \mathbb{1}[q(\tau_j) \ge q(\tau_i)]$, reducing neighbor leakage by $>60\%$ and cutting early-commit token flip rates from $20.5\%$ to $0.44\%$.
- **Task Verification**: Quantized wave clocks ($W=8, K=4$) improved hierarchical key-value copy accuracy from $16.8\%$ to **$100.0\%$** and sequence sorting monotonicity from $10.9\%$ to **$62.5\%$**.

#### 4. Few-Step Trajectory Distillation
- Established strictly held-out WMT14 De-En splits (`DISTILL` $N=2000$, `SELECT` $N=500$, `TEST` $N=500$).
- **Progressive Halving**: Achieved **$22.13\text{ chrF}$ in a single $2.2\text{ ms}$ step**—a **$5.1\times$ leap** over the 1-step teacher ($4.32\text{ chrF}$) and outperforming the 32-step teacher baseline ($20.60\text{ chrF}$).
- **Consistency Distillation**: Reached $0.38\text{ BLEU}$ at 4 steps ($2.6\times$ higher than baseline).

---

### Summary of Empirical Benchmarks

| Metric / Task | Baseline | Flow Matching + Our Methods | Impact |
|:---|:---:|:---:|:---:|
| **Few-Step Perplexity ($N=4$)** | $181.86$ dedup-PPL (MDLM-130M) | **$13.49$ dedup-PPL** (ELF-B) | **$13.5\times$ better perplexity** |
| **Generation Latency ($N=4$)** | $13.7\text{ s}$ (MDLM-130M) | **$4.2\text{ s}$** (ELF-B) | **$3.2\times$ wall-clock speedup** |
| **16-Step MAUVE Coherence** | $0.445$ (ODE-16) | **$0.860$** (ECP-ODE-16) | **+53.9% distributional coherence** |
| **16-Step ODE Perplexity** | $98.61$ | **$84.25$** | **10.7% perplexity reduction** |
| **Hierarchical KV Copy Accuracy** | $16.8\%$ (Global time) | **$100.0\%$** (Wave Clocks $W=8, K=4$) | **$5.95\times$ accuracy gain** |
| **1-Step Distillation Quality** | $4.32\text{ chrF}$ / $57.3\text{ ms}$ | **$22.13\text{ chrF}$ / $2.2\text{ ms}$** | **$5.1\times$ quality leap, $26\times$ faster** |

---

### Technical Stack & Rigor
- **Frameworks**: PyTorch, Hugging Face Transformers (T5, GPT-2), SafeTensors, SciPy, NumPy.
- **Verification**: Built an automated suite of 19 CPU unit and stress tests (testing boundary interpolations, water-filling clock algorithms, and operator shapes).
- **Manuscript**: Authored a 23-page research manuscript targeting top-tier conferences (NeurIPS/ICML), including complete mathematical proofs, vector figures, and audit ledgers across >250 evaluation runs.
