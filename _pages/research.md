---
layout: page
title: research
permalink: /research/
description: Academic research in continuous generative flows, antibody developability, high-dimensional representation geometry, and whole-brain connectomics.
nav: true
nav_order: 1
display_categories: [research]
horizontal: false
---

My research focuses on foundational and applied questions across four primary areas:

1. **[Continuous Language Flows & Sampler Control Geometry](/projects/continuous-flows/)**: Investigating the geometry of generation in continuous-latent flow models, proving the operating-point non-equivalence of classifier-free guidance ($\text{SC-CFG}$) and stochastic temperature ($\gamma$) across identical $(\text{Gen-PPL}, H_{\text{ps}})$, and formulating manifold trajectory retraction operators (ECP, CMLC) for high-throughput text generation.
2. **[AbDev-X: Therapeutic Antibody Developability with Paired-Sequence GPs](/projects/abdevx/)**: Developing paired heavy-light chain Gaussian Process models with biologically grounded covariance kernels (LOCK-GP), Kolmogorov-Arnold Networks (MultKAN), and residual non-conformity score (RNS) selective abstention across clinical developability assays (`GDPa1` $N=246$, `GDPa3` $N=80$).
3. **[QuantPPI: Data-Oblivious Vector Quantization for Protein Embeddings](/projects/quantppi/)**: Addressing spectral coordinate anisotropy ($\mathrm{PR}/D = 0.0012$) in protein language model embeddings (ESM-2) through data-oblivious Multi-Radix Lloyd-Max codecs, spherical $E_8$ lattices, and C/ARM NEON SIMD kernels, outperforming trained FAISS OPQ with zero calibration data and half the RAM.
4. **[VectorWing: Whole-Brain Drosophila Connectomics & Chemospecific Feedback](/projects/connectomics/)**: Engineering memory-bounded streaming pipelines to map 16.8M synapses from the adult *Drosophila* whole-brain electron microscopy connectome (FlyWire FAFB v783), discovering topological null-model artifacts, chemospecific feedback asymmetry ($2.53\times$–$5.92\times$ heterotypic $\text{ACh} \leftrightarrow \text{GABA}$ loops), and a 16-column dynamical compass model.

---

<!-- Display research project cards -->
<div class="projects">
{% assign sorted_projects = site.projects | where: "category", "research" | sort: "importance" %}
  <div class="row row-cols-1 row-cols-md-2">
    {% for project in sorted_projects %}
      {% include projects.liquid %}
    {% endfor %}
  </div>
</div>
