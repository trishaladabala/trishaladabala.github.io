---
layout: about
title: about
permalink: /
subtitle: B.Tech in CSE (AI/ML Specialization) · Minor in Mathematics · Shiv Nadar University

profile:
  align: right
  image: prof_pic.jpg
  image_circular: false # crops the image to make it circular
  more_info: >
    <p>Shiv Nadar Institute of Eminence</p>
    <p>Greater Noida, Delhi NCR, India</p>
    <p><a href="mailto:adabalatrishal@gmail.com">adabalatrishal@gmail.com</a></p>

selected_papers: false # includes a list of papers marked as "selected={true}"
social: true # includes social icons at the bottom of the page

announcements:
  enabled: false # news section removed per user preference

latest_posts:
  enabled: false
---

I am an undergraduate student at [Shiv Nadar Institute of Eminence](https://snu.edu.in/), pursuing a Bachelor of Technology in **Computer Science and Engineering** with a specialization in **Artificial Intelligence & Machine Learning** and a **Minor in Mathematics** (cumulative GPA: **9.14 / 10.0**).

As an undergraduate student researcher, I work at the intersection of **Machine Learning Theory**, **Generative Models**, **High-Dimensional Geometry**, and **Computational Biology**. I am especially motivated by using mathematical and geometric principles to build scalable, observable, and efficient learning systems for biological and foundation model representations.

---

### Research Interests

- **Continuous-Latent Flow Matching & Sampler Control Geometry**: Investigating probability flow ODEs/SDEs, proving operating-point non-equivalence between guidance and stochastic temperature, manifold trajectory retraction (Encoder-Cycle Projection), causal wave-clock schedules, and trajectory distillation for non-autoregressive text generation.
- **Therapeutic Antibody Developability & Biophysical Priors**: Formulating paired heavy-light chain Gaussian Process models (LOCK-GP), Kolmogorov-Arnold Networks (MultKAN 2.0), and conformal uncertainty estimation (Residual Non-Conformity Score selective abstention) across rigorous clinical developability assays (`GDPa1`, `GDPa3`).
- **High-Dimensional Geometry & Vector Quantization**: Uncovering spectral coordinate anisotropy in foundation model embeddings (e.g., ESM-2), developing data-oblivious quantization codecs with closed-form Bussgang de-attenuation, spherical $E_8$ lattices, and engineering SIMD-accelerated asymmetric distance search.
- **Computational Neurobiology & Whole-Brain Connectomics**: Analyzing whole-brain synaptic connectivity using the FlyWire FAFB v783 electron microscopy connectome (139,000+ neurons, 16.8M chemical synapses), formulating degree-stratified null models, and dissecting chemospecific recurrent microcircuits in the insect steering system.

---

### Featured Research

Please visit the dedicated **[Research](/research/)** section or explore each case study below:

- **[Continuous Language Flows & Sampler Control Geometry](/projects/continuous-flows/)**: Proved that operating points $(\text{Gen-PPL}, H_{\text{ps}}, \text{NFE})$ are non-identifiable across guidance ($\text{SC-CFG}$) vs. stochastic temperature ($\gamma$) across a 280-run factorial grid on ELF-B/M/L and MDLM; two-way ANOVA demonstrates $\text{SC-CFG}$ controls local 4-gram repetition (66.1%–98.3% variance) while $\gamma$ controls cross-sample corpus diversity (85.2%–91.1% variance) and MAUVE (92.5%); formulated Encoder-Cycle Projection ($\Pi = E \circ D$) reducing 16-step ODE perplexity by 10.7% (+53.9% MAUVE).
- **[AbDev-X: Therapeutic Antibody Developability with Paired-Sequence GPs](/projects/abdevx/)**: Built paired-sequence Gaussian Process models (LOCK-GP) and Kolmogorov-Arnold Networks (MultKAN 2.0) across 5 clinical assays (`HIC`, `Tm2`, `AC-SINS`, `PR_CHO`, `Titer`) on `GDPa1` ($N=246$) and `GDPa3` ($N=80$); resolved upstream heavy-light subtype feature swap recovering `Tm2` from $-0.0587 \to +0.3247$; introduced Residual Non-Conformity Score (RNS) selective abstention achieving $\rho = 0.2412$ at 70% coverage.
- **[QuantPPI: Data-Oblivious Vector Quantization for Protein Embeddings](/projects/quantppi/)**: Identified extreme coordinate anisotropy ($\mathrm{PR}/D = 0.0012$) in protein language model embeddings (ESM-2); proved Support-Orthogonal Bleed Elimination and Analytical Bussgang De-Attenuation theorems; engineered data-oblivious multi-radix codecs (168 B–488 B) and spherical $E_8$ lattices that surpass trained FAISS OPQ ($0.9993$ vs. $0.9968$ Recall@20@30) with zero training time; built an interactive 3D Mol* web studio.
- **[VectorWing: Whole-Brain Drosophila Connectomics & Chemospecific Feedback](/projects/connectomics/)**: Developed an out-of-core pipeline analyzing 16.8M synapses from the FlyWire whole-brain connectome under $<380\text{ MB}$ RAM; identified topological edge-swap artifacts; discovered a $2.53\times$–$5.92\times$ heterotypic $\text{ACh} \leftrightarrow \text{GABA}$ reciprocity inversion with 100% strict directional chemical polarity in the fly steering matrix.

---

### Internship Projects (Hewlett Packard Enterprise)

- **[Decentralized Swarm Learning with Scikit-Learn](/projects/#internship%20projects)** (*Software Development Intern, CPP-3*): Extended HPE Swarm Learning beyond deep neural networks to support classical Scikit-Learn models, implementing framework-independent weight handling and decentralized parameter aggregation for privacy-preserving machine learning.
- **[Dolos Plagiarism Detection & Assessment Infrastructure](/projects/#internship%20projects)** (*Software Development Intern, CPP-2*): Integrated AST-based Dolos plagiarism detection into the DMOJ programming contest platform and engineered an automated assessment backend with penalty algorithms for proctoring violations.

---

### Leadership & Honors

- **PULSE (Programmers United for Learning and Skill Enhancement)** — *Organizing Committee Member*: Organized weekly competitive programming contests, led technical workshops, and conducted interview-preparation sessions on algorithms and data structures.
- **Academic Honors**: Named to the **Dean's List for Academic Excellence** across four semesters (Monsoon 2023, Spring 2024, Monsoon 2025, Spring 2026).

---

### Prospective Graduate Study

I am actively preparing to apply for **PhD and Master's programs in Computer Science and Artificial Intelligence**. I look forward to joining a research group focused on generative modeling, scalable machine learning systems, high-dimensional representation geometry, or computational biology.

Feel free to explore my **[Research](/research/)**, browse my **[Projects](/projects/)**, view my **[CV](/cv/)**, or reach out via [email](mailto:adabalatrishal@gmail.com).
