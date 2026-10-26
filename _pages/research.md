---
layout: page
title: research
permalink: /research/
description: Academic research in continuous generative flows, high-dimensional representation geometry, and computational biology.
nav: true
nav_order: 1
display_categories: [research]
horizontal: false
---

My research focuses on foundational and applied questions across three primary areas:

1. **Continuous-Latent Language Flows**: Formulating manifold trajectory retraction operators (Encoder-Cycle Projection) and causal wave-clock schedules for high-throughput non-autoregressive text generation.
2. **High-Dimensional Geometry & Vector Quantization**: Developing data-oblivious quantization codecs that address spectral coordinate anisotropy in protein language model embeddings (ESM-2), outperforming trained FAISS OPQ with zero calibration data.
3. **Computational Biology & Connectomics**: Engineering out-of-core pipelines to analyze 16.8M synapses from the adult *Drosophila* whole-brain electron microscopy connectome (FlyWire FAFB v783), uncovering chemospecific feedback architecture and steering dynamics.

---

<!-- Display research project cards -->
<div class="projects">
{% assign sorted_projects = site.projects | where: "category", "research" | sort: "importance" %}
  <div class="row row-cols-1 row-cols-md-3">
    {% for project in sorted_projects %}
      {% include projects.liquid %}
    {% endfor %}
  </div>
</div>
