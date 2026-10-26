---
layout: page
title: Dolos Plagiarism Detection & Assessment Engine
description: Integration of AST-based Dolos plagiarism detection into DMOJ with anti-cheating penalty mechanics.
importance: 2
category: software engineering & systems
related_publications: false
---

### Project Context

Automated coding assessment platforms must balance two competing challenges: delivering seamless developer experiences during programming contests while upholding rigorous evaluation integrity against sophisticated plagiarism and cheating techniques.

During my software development internship at **Hewlett Packard Enterprise (HPE)**, I designed and integrated core evaluation infrastructure into the **DMOJ** (Don Mills Online Judge) programming contest platform.

---

### Key Technical Contributions

#### 1. AST-Based Plagiarism Detection via Dolos
- Traditional diff-based or token-string matching tools struggle to catch refactored code (variable renaming, dead code insertion, statement reordering).
- Integrated **Dolos**, an advanced plagiarism detection engine that parses source submissions into language-specific Abstract Syntax Trees (ASTs).
- Implemented k-gram fingerprinting and winnowing algorithms to detect structural similarity invariants across submitted source files, providing instructors with visual cluster graphs and similarity matrices.

#### 2. MCQ Assessment & Automated Evaluation Module
- Architected the backend service for multiple-choice assessment management, supporting categorized question banks, weighted question scoring, and deterministic automated grading.
- Implemented state-machine session handling to manage test timers, autosave candidate progress, and safely conclude tests upon network interruptions.

#### 3. Proctoring Telemetry & Anti-Cheating Penalty System
- Engineered proctoring listeners capturing tab-switching events, window defocus incidents, and video stream interruptions.
- Implemented a configurable progressive penalty algorithm that computes deduction weights and marks flagged sessions for manual human audit based on anomaly severity.
