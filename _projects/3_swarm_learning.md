---
layout: page
title: Decentralized Swarm Learning with Scikit-Learn
description: Extending HPE Swarm Learning to classical estimators for privacy-preserving decentralized model training.
importance: 3
category: machine learning & ai
related_publications: false
---

### Context & Motivation

**Swarm Learning** is a decentralized, privacy-preserving machine learning framework that enables multiple collaborative nodes to train shared models collaboratively without sharing private local data. Unlike conventional federated learning, which relies on a central aggregator node that can become a single point of failure or privacy vulnerability, Swarm Learning uses peer-to-peer coordination (often orchestrated via smart contracts and distributed ledger technology) to dynamically elect aggregators.

Historically, Swarm Learning was designed predominantly for deep learning architectures implemented in **TensorFlow** and **PyTorch**, where model state consists cleanly of structured tensor weight arrays and gradients. However, a significant fraction of enterprise data (especially in healthcare, finance, and fraud detection) consists of tabular records best modeled by classical estimators in **Scikit-Learn**.

---

### Key Contributions & Technical Highlights

During my internship at **Hewlett Packard Enterprise (HPE)**, I worked on extending HPE Swarm Learning to natively support **Scikit-Learn** models:

#### 1. Framework-Independent Weight Abstraction
- Designed a unified model adapter interface that decouples framework-specific model internals from Swarm Learning's peer synchronization protocol.
- Engineered specialized serialization and extraction routines for classical estimators (such as Logistic Regression, Linear Models, SGDClassifiers, and Support Vector Machines) to represent coefficient vectors, intercept terms, and convergence states as normalized weight payloads.

#### 2. Decentralized Parameter Aggregation
- Implemented weighted FedAvg-style aggregation mechanics across disparate nodes holding partitioned non-IID datasets.
- Handled edge cases such as sparse representations, differing regularization parameters, and varying iteration convergence states among peers.

#### 3. Validation & Resiliency Testing
- Created end-to-end integration test suites across multi-node distributed setups simulating network latency, node dropouts, and non-IID data distributions.
- Demonstrated that decentralized Scikit-Learn training matches the accuracy of centralized baselines while strictly preserving data locality and regulatory privacy requirements.
