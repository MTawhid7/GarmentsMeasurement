# Automated Industrial Garment Measurement: Greenfield Feasibility & Metrology Investigation

## Executive Summary & Navigation Guide

This repository contains an exhaustive, first-principles metrological investigation into the feasibility of automatically measuring factory-produced garments for industrial quality control (QC). 

Unlike conventional computer-vision explorations that treat apparel inspection as an off-the-shelf "keypoint detection" or "background segmentation" task, this study approaches the challenge fundamentally as a **dimensional metrology and conformity assessment problem on non-rigid, viscoelastic bodies**.

### The Core Inquiry
> **Can an automated system measure a broad spectrum of production garments (knit tops, woven tops, bottoms) with the accuracy, repeatability, and reliability required for industrial quality control; and if so, what sensing, physical, and algorithmic architecture makes this achievable?**

---

### Key Findings at a Glance

1. **The Binding Bottleneck is Physical and Definitional, Not Sensor Resolution:**
   The fundamental limitation in garment measurement is **not** optical resolution or camera sensor noise. Modern 2D industrial sensors easily achieve $<0.2\text{ mm/pixel}$ across a $1.2\text{ m} \times 1.6\text{ m}$ field. The true limiting factors are:
   - **Definitional Uncertainty ($u_A$):** Vague, text-based specifications in tech packs (e.g., "chest width 2.5 cm below armhole" or "SNP to hem") leave ambiguous datum axes and feature boundaries, causing $1\text{ mm to }3\text{ mm}$ of inter-expert disagreement on a static garment.
   - **Garment State Variability ($u_B$):** Viscoelastic fabric relaxation, handling-induced in-plane shear, table friction hysteresis (where friction cannot sustain elastic recovery tension), seam roll, and out-of-plane wrinkles (where even a $1\text{ mm}$ ridge introduces a $2\text{ mm}$ projected shortening error) induce $1\text{ mm to }>5\text{ mm}$ of variation between independent lays of the same garment.
   - **Conclusion:** No algorithm or 3D sensor can overcome an under-defined measurand or an uncontrolled physical state.

2. **Autopsy of the Previous Approach (€15k Zivid 3D + Gemini/Nano Banana Synthetic Data):**
   The previous team's ~80% accuracy and difficulty in obtaining reference points stemmed from predictable structural mismatches:
   - **Zivid 2+ 3D Camera:** While an exceptional industrial sensor, its spatial resolution ($0.44\text{ mm/pixel}$ at $1.1\text{ m}$) is coarser than a €500 24–61 MP 2D sensor. Its datasheet dimensional trueness ($\pm 0.35\%\text{ to }0.60\%$) introduces $2.5\text{ mm to }4.2\text{ mm}$ of systematic scale uncertainty over a $700\text{ mm}$ garment span unless calibrated in-situ. Furthermore, summing 3D surface point clouds along geodesics severely inflates length due to depth noise ($\mathbb{E}[\sqrt{a^2 + 2\sigma_z^2}] > a$).
   - **Synthetic Text-to-Image Data (Gemini / Nano Banana):** Generative text-to-image models lack projective and topological consistency (Sarkar et al., CVPR 2024). They hallucinate physically impossible seam geometry, introduce arbitrary folds, and carry zero metric truth.
   - **Domain Gap & Green Sheet:** Moving from synthetic white backgrounds to a physical green sheet introduced chroma reflections/spill onto garment edges, edge blur in RMBG segmentation (which resizes to $1024 \times 1024$, i.e., $1.2\text{ mm/pixel}$), and failure on greenish fabrics.
   - **Keypoint Detector Abstraction Failure:** Top-down keypoint networks (e.g., HRNet) downsample via strided convolutions into $64 \times 48$ heatmaps ($12\text{ mm to }19\text{ mm}$ cell quantization). Post-hoc "correction algorithms" act as strong geometric priors that cause **shrinkage toward nominal**, masking the very manufacturing defects QC is tasked with catching.

3. **Technically Credible Architectural Solution:**
   Industrial accuracy cannot be achieved with an unconstrained flat-lay and a single deep neural network. A credible architecture requires a **strict three-layer decoupling**:
   - **Layer 1: State Control Layer:** Standardized ergonomic lay protocol combined with active physical constraints (such as a perforated vacuum table or electrostatic platen, adapting ISO 10579 non-rigid part metrology) and photometric flatness/wrinkle gating.
   - **Layer 2: Semantic Proposal Layer:** A lightweight neural network whose *only* job is rough region-of-interest localization and topological classification (e.g., distinguishing the front hem from the back hem). It *never* outputs the final measurement coordinate.
   - **Layer 3: Metrology & Geometric Operator Layer:** Operates exclusively at full sensor resolution ($<0.15\text{ mm/pixel}$). Executes formal GPS (Geometrical Product Specification, ISO 17450) operators: sub-pixel edge extraction (Steger curvilinear detector), multi-angle photometric stereo seam ridge fitting, datum construction, and intersection calculation. Every measurement generates an explicit confidence metric; ambiguous cases trigger **selective abstention** rather than guessing.

4. **Economics & Equipment:**
   An expensive €15,000 structured-light setup is **neither necessary nor optimal** for flat-lay quality control. A metrology-grade system based on a calibrated 45–61 MP 2D industrial camera, telecentric/low-distortion optics, multi-spectral backlighting (transmission silhouette gauging), and multi-angle raking illumination can be constructed for **€2,500 to €5,000 in hardware**, achieving standard measurement uncertainty $u_c \le 0.5\text{ mm}$ on calibrated features—surpassing the 3D camera's planar accuracy.

---

### Structure of the Investigation Documents

The research is organized into six core analytical chapters and a comprehensive bibliography:

| Document | Focus & Key Questions Answered |
|---|---|
| **[01 — Problem and Requirements](01-problem-and-measurement-requirements.md)** | • Formal metrological formulation of garment QC.<br>• Taxonomy of Points of Measure (POMs) under ISO 18890:2018 and ISO 3759:2011.<br>• GPS specification operator framework (ISO 17450 / ISO 10579).<br>• Definitional uncertainty, tolerance budgeting, and deconstructing "80% accuracy".<br>• Operational contrast: End-of-line sampling (S-AQL) vs 100% Inline screening (S-Inline). |
| **[02 — Metrology Principles & State of the Art](02-metrology-principles-and-state-of-the-art.md)** | • Transferable principles from semiconductor, aerospace, and precision optical metrology.<br>• Why micro/nanometre precision does not scale directly to viscoelastic textiles.<br>• Lessons from analogous soft-tissue and deformable object inspection (cephalometrics, anthropometrics, fish conveyor sorting).<br>• Critical review of published apparel measurement literature, patents, and commercial tools.<br>• Deep-dive post-mortem on the previous team's €15k Zivid and synthetic data pipeline. |
| **[03 — Fundamental Technical Limitations](03-fundamental-technical-limitations.md)** | • First-principles error propagation model ($u_A$ through $u_G$).<br>• Physical limits of fabrics: hysteresis, table friction limits, wrinkle-induced foreshortening, seam roll.<br>• Optical/sensor physics: perspective height distortion ($h/H$), scale trueness, depth noise inflation.<br>• Observability of reference landmarks (cut edges vs folds vs hidden seam junctions).<br>• Semantic model resolution bottlenecks (heatmap quantization) and the danger of prior shrinkage.<br>• Comprehensive classification matrix: Algorithmic vs Sensing vs Physical vs Definitional. |
| **[04 — Candidate Measurement Architectures](04-candidate-measurement-architectures.md)** | • Detailed architectural templates for candidate approaches (A0 through A10):<br>  - *A0/A0+:* Manual baseline and connected digital metrology tools.<br>  - *A1:* High-resolution calibrated 2D flat-lay with multi-angle raking illumination.<br>  - *A2:* Backlit transmission silhouette metrology table.<br>  - *A3:* Continuous line-scan / CIS orthographic conveyor system.<br>  - *A4:* Hybrid 2D metrology with active low-cost photometric flatness gating.<br>  - *A5:* Dedicated structured-light 3D camera system (Zivid class).<br>  - *A6:* Restrained/tension-controlled fixtures (vacuum hold-down, platen compression).<br>  - *A7:* Human-in-the-loop collaborative semantic verification.<br>  - *A8:* CAD/Pattern-constrained non-rigid registration layer.<br>  - *A9:* Design-for-measurability (in-situ invisible UV fiducials).<br>• Comparative evaluation matrix across 12 dimensions.<br>• Cycle time, throughput, and rigorous economic value model. |
| **[05 — Feasibility & Validation Methodology](05-feasibility-and-validation-methodology.md)** | • Staged empirical validation ladder (Stages S0 to S7) designed to isolate variance components without premature AI investments.<br>• Traceable reference instrumentation (OIML R 35 Class I rules, certified grid plates, blind UV marking).<br>• Factorial sampling frame (knit tops, woven tops, denim/chinos, sizes, finishes, colors, seeded defects).<br>• Statistical analysis plan: REML mixed models, Gage R&R, Deming regression, Bland-Altman agreement, JCGM 106 decision risk integration.<br>• Formal Go/No-Go Decision Gates (G1 to G4) and attribution diagnostics. |
| **[06 — Synthesis & Strategic Roadmap](06-synthesis-and-strategic-roadmap.md)** | • Definitive engineering verdict: What is feasible, under what physical conditions, and what is structurally unachievable.<br>• Ranked bottleneck hierarchy.<br>• Step-by-step phased pilot experimental protocol to establish feasibility before capital allocation.<br>• Final recommendations for industrial deployment. |
| **[References & Annotated Bibliography](references.md)** | • Fully keyed, annotated citations across international standards, metrology treatises, peer-reviewed computer-vision literature, patents, and vendor datasheets.<br>• Epistemic verification and access status for every cited source. |

---

### Epistemic Conventions and Evidence Classification

Every substantive claim throughout this research is explicitly tagged with an epistemic label to distinguish empirical facts from engineering models and unproven conjectures:

- **`[E]` Established Fact:** Supported by an authoritative, peer-reviewed primary source, an official international standard (ISO, OIML, ASTM, VDA, AIAG), or manufacturer hardware specification that was directly inspected.
  - `[E secondary]`: A claim reported reliably by a secondary peer-reviewed survey or standard summary.
  - `[E vendor]`: A manufacturer specification or claim that requires independent calibration verification.
- **`[I]` Engineering Inference:** Deduced from physical first principles, projective geometry, classical mechanics, or rigorous mathematical error propagation. Assumptions and derivations are fully laid out.
- **`[H]` Hypothesis:** A plausible physical or algorithmic proposition that is currently unverified in published textile metrology literature and requires empirical validation via the Stage S0–S7 experimental protocol.

---

### Units and Mathematical Conventions

- **Units:** SI units are used exclusively ($mm, \mu m, m, N, kPa$). Garment industry tolerance values originating in imperial units are converted with exact conversions ($1/2'' = 12.7\text{ mm}$, $1/4'' = 6.35\text{ mm}$, $1/8'' = 3.175\text{ mm}$).
- **Uncertainty Nomenclature (per JCGM 100:2008 / GUM):**
  - $u$: Standard uncertainty (corresponding to one standard deviation $\sigma$).
  - $U = k \cdot u$: Expanded uncertainty with coverage factor $k=2$ (approximating a 95.45% confidence interval under normality).
  - $u_c$: Combined standard uncertainty calculated via root-sum-square quadrature.
  - $T$: Total tolerance band width ($T = 2t$ for a symmetric tolerance of $\pm t$).
