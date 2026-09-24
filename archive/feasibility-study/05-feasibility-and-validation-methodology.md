# 05 — Empirical Feasibility and Validation Methodology

## 1. Research Philosophy: The Phased Validation Ladder

To determine whether an automated garment measurement system is technically viable *before* committing capital to full-scale software and hardware engineering, an empirical research program must **isolate and quantify each variance component individually**.

The previous team spent two years iterating on models because all error sources were conflated into a single undefined "80% accuracy" metric. When an algorithm failed, they could not determine whether the fault lay in:
- The camera calibration;
- The optical visibility of the seam;
- The physical wrinkle state of the fabric;
- An ambiguous tech pack definition; or
- The neural network's weights.

```
┌────────────────────────────────────────────────────────────────────────┐
│                   THE PHASED METROLOGICAL LADDER                       │
├────────────────────────────────────────────────────────────────────────┤
│ S0: Measurand Pre-Registration & GPS Operators       [Desk / Specs]    │
│ S1: Sensing & Optical Chain on Rigid Standards       [Zero Textiles]   │
│ S2: Boundary Physics on Rigid Textile Swatches       [Zero Handling]   │
│ S3: Garment State Variability on Marked Garments     [ZERO AI]         │
│ S4: Definitional Uncertainty of Certified Experts    [Fixed Lay]       │
│ S5: Observability & Information Ceiling              [Modality Test]   │
│ S6: Model Semantic Localization vs UV Ground Truth   [AI Isolation]    │
│ S7: End-to-End System Study vs Manual Method         [Conformity Risk] │
└────────────────────────────────────────────────────────────────────────┘
```

### Core Methodological Principles:
1. **The Principle of Oracle Substitution:** Replace one subsystem at a time with a "perfect" physical realization:
   - To test sensing ($u_C$), substitute rigid quartz plates for garments.
   - To test garment state ($u_B$), substitute physical high-contrast markers for learned neural keypoints.
   - To test feature extraction ($u_D$), substitute a backlit transmission silhouette for deep segmentation models.
2. **Strict Identity of the Physical Lay:** When comparing an automated system against human inspectors or reference sensors, **both systems must measure the identical physical lay without disturbing the fabric**. If an operator lifts and re-lays the garment between measurements, garment-state variability ($u_B$) contaminates the instrument evaluation `[I]`.
3. **Independent Blind Marking via Invisible UV-Fluorescent Ink:** Adapting the validated anthropometric methodology of Kouchi & Mochimaru `[E, Kouchi2011]`, expert landmarkers place microscopic dots using ink that is completely invisible under standard illumination ($400\text{–}700\text{ nm}$) but fluoresces vividly under $365\text{ nm}$ ultraviolet light. This enables:
   - Multiple experts to mark the identical garment without seeing previous placements;
   - Capturing pristine, unmarked images for testing AI models under white light, followed immediately by a UV strobe capturing exact physical ground truth on the **same physical state** `[H]`.

---

## 2. Traceable Reference Instrumentation

Every stage of validation requires reference standards whose expanded uncertainty $U_{\text{ref}}$ is strictly characterized under **ISO/IEC 17025** `[E, JCGM106 §7.6]`:

| Ref ID | Instrument / Artefact | Traceability & Standards | Uncertainty Capability ($k=2$) | Operational Role |
|---|---|---|---|---|
| **REF-1** | **Certified Glass/Invar Grid Plate** | Calibrated on a coordinate measuring machine (CMM) traceable to national standards (NIST/PTB). Laser-etched chrome dot array ($1000 \times 1000\text{ mm}$, $20\text{ mm}$ pitch). | $U \le 0.015\text{ mm}$ ($15\ \mu\text{m}$) across entire plate | Stage S1: Calibration, field distortion verification, and camera stability. |
| **REF-2** | **Step-Height Gauge Blocks** | Hardened tool steel / ceramic gauge blocks ($1.0, 3.0, 5.0, 10.0, 15.0\text{ mm}$ height) traceable to ISO 3650. | $U \le 0.002\text{ mm}$ ($2\ \mu\text{m}$) | Stage S1: Direct verification of perspective parallax height models ($\Delta r = r \frac{h}{H-h}$). |
| **REF-3** | **Precision Rigid Steel Scale Rules** | Class I rigid stainless steel rules certified to **OIML R 35-1:2007** `[E, OIML-R35]`. MPE formula: $\pm (0.1 + 0.1 L)\text{ mm}$. | $U \le 0.20\text{ mm}$ up to $1000\text{ mm}$; $U \le 0.30\text{ mm}$ up to $2000\text{ mm}$ | Stage S2, S3, S7: Physical manual verification between marked fiducials. |
| **REF-4** | **Optical Photogrammetric Reference (R3-Opt)** | Calibrated multi-camera stereo photogrammetry or high-resolution vertical camera viewing retroreflective/UV micro-dots in-situ. | $U \le 0.15\text{ mm}$ in-plane | Primary ground-truth metric reference for marked garments on the inspection table. |
| **REF-5** | **Certified QC Inspector Panel** | Five professional, certified apparel QC inspectors following ISO 18890 with calibrated OIML Class II tapes. | Measured empirically in Stage S4 and S7 | Evaluates the contractual conventional baseline ($u_G$). |

---

## 3. Factorial Sampling Frame

To ensure statistical validity, test garments must represent the full mechanical, optical, and structural diversity of commercial apparel manufacturing:

```
                          SAMPLING FACTORIAL DESIGN
                                     │
         ┌───────────────────────────┼───────────────────────────┐
         ▼                           ▼                           ▼
   KNIT TOPS (3 Styles)        WOVEN TOPS (2 Styles)         BOTTOMS (3 Styles)
   • 100% Cotton Single Jersey • 100% Cotton Poplin Shirt    • Rigid 14oz Denim Jeans
   • Piqué Cotton Polo Shirt   • Fluid Poly/Viscose Blouse   • 98/2 Stretch Denim Jeans
   • Brushed Poly/Cotton Fleece                              • Cotton Twill Chinos
```

### Strata and Experimental Variables:
1. **Garment Structural Families:**
   - *Single Jersey T-Shirt:* High in-plane shear, curl-prone edges, knitting spirality (seam twist).
   - *Piqué Polo Shirt:* Heavy knitted collar stand ($10\text{ mm}$ relief), front button placket, ribbed cuffs.
   - *Brushed Fleece Hoodie:* Heavy thickness ($3\text{–}5\text{ mm}$), surface fuzz, Kangaroo pocket relief.
   - *Poplin Woven Shirt:* Low extensibility, crisp pressed seams, multi-ply collar and yoke.
   - *Denim Jeans (Rigid vs Stretch):* Heavy flat-felled seams ($4\text{–}6\text{ plies}$), thick waistband ($8\text{ mm}$ relief), elastic recovery hysteresis in stretch denim.
2. **Color and Optical Reflectance Strata:**
   - Optical White (High brightness, low contrast against white tables).
   - Pitch Black (Zero internal seam contrast under diffuse light).
   - Chroma Green (Directly tests the chroma-key / green-sheet failure mode).
   - Yarn-Dyed Heather Grey & Striped Prints (Confuses edge filters with internal texture).
3. **Size Gradient:** Smallest graded size (XS) and largest graded size (XXL / 3XL) per style to verify geometric linearity.
4. **Seeded Defect Population:** In addition to normal production garments, the factory sample room must prepare **seeded out-of-tolerance units** with deliberate, verified manufacturing defects:
   - Hem shortened by $-15\text{ mm}$ and $-25\text{ mm}$.
   - Sleeve shortened by $-20\text{ mm}$ on the left arm only (asymmetry defect).
   - Chest width altered by $+15\text{ mm}$.
   - *Purpose:* Audits candidate algorithms against **shrinkage toward nominal** (Chapter 03 §5.2).

---

## 4. Detailed Experimental Protocols (Stages S0 to S7)

### Stage S0: Measurand Pre-Registration and GPS Operator Specification
- **Objective:** Eliminate definitional ambiguity ($u_A$) prior to physical measurement.
- **Protocol:** Technical design and QC teams draft formal GPS specification operators (Partition, Extraction, Filtration, Association, Construction, Evaluation) for every POM in scope, cross-referenced to ISO 18890 codes. For ambiguous POMs, alternative interpretations (e.g., seam-to-seam vs edge-to-edge) are explicitly pre-registered.
- **Deliverables:** Machine-executable measurement recipes and standardized lay protocol documents.

---

### Stage S1: Sensing Chain Verification on Rigid Standards (Zero Textiles)
- **Objective:** Quantify optical and calibration uncertainty ($u_C$) and directly answer whether an expensive €15k structured-light camera is necessary.
- **Apparatus:** Precision gantry mounting the Candidate 2D camera (45–61 MP) and the Zivid 2+ L110 3D camera side-by-side over the inspection table.
- **Measurements:**
  1. *Spatial Calibration & Trueness:* Measure REF-1 glass grid plate across 9 table positions. Compute root-mean-square residual error across the entire $1200 \times 1600\text{ mm}$ volume.
  2. *Perspective Parallax Verification:* Place step gauge blocks ($h = 1, 3, 5, 10\text{ mm}$) at center and corners ($r = 0, 300, 600\text{ mm}$). Measure observed radial shift $\Delta r$ and compare against theoretical pinhole model $\Delta r = r \frac{h}{H-h}$.
  3. *Thermal & Temporal Drift:* Capture static grid continuously over an 8-hour factory operational shift. Track thermal scale drift versus ambient temperature.
  4. *Tool Induced Shift (TIS Reversal):* Rotate the grid plate $180^\circ$ to isolate optical asymmetry.
- **Pass Gate G1:** $u_C \le 0.25\text{ mm}$ across the entire working volume; scale trueness error $\le 0.05\%$.

---

### Stage S2: Boundary Physics on Rigid Textile Swatches (Zero Handling)
- **Objective:** Quantify edge and seam extraction uncertainty ($u_D$) free from garment handling effects.
- **Apparatus:** Textile swatches representing all fabric classes, featuring cut edges, folded edges, and stitched seams mounted rigidly on flat invar backing plates.
- **Measurements:**
  1. Measure physical boundary locations under three illumination modes:
     - Pure Transmission Backlight (Architecture A2);
     - Overhead Diffuse Dome Light (Architecture A1);
     - Multi-Azimuth Raking LED Grazing Light (Architecture A1/A4).
  2. Compare sub-pixel optical edge coordinates against reference micrometer stages.
  3. Verify UV-fluorescent ink invisibility under white light ($400\text{–}700\text{ nm}$) and optical signal-to-noise ratio under $365\text{ nm}$ UV strobe.
- **Outputs:** Standard edge localization bias and variance per fabric class; determination of optimal illumination per POM.

---

### Stage S3: Garment State Variability on Marked Garments (Zero AI)
- **Objective:** Quantify physical garment state variability ($u_B$) and evaluate the efficacy of physical restraint fixtures.
- **Crucial Feature:** **Zero machine learning is used in this stage.**
- **Protocol:**
  1. Certified technicians physically mark all POM endpoints on real production garments with microscopic UV-fluorescent dots according to S0 definitions.
  2. Operators lay each garment repeatedly ($n \ge 5$ independent re-lays per garment, randomized order) under three physical conditions:
     - *Arm 1 (Free State):* Standard hand-smoothed flat lay per ISO 18890 protocol.
     - *Arm 2 (Vacuum Restraint - A6-Vac):* Garment laid on perforated vacuum table; $2.0\text{ kPa}$ suction activated.
     - *Arm 3 (Compression Platen - A6-Platen):* Optical acrylic platen lowered onto garment.
  3. Capture UV strobe image on every lay. Locate physical dots using classical sub-pixel centroid algorithms ($u_{\text{algorithm}} < 0.02\text{ mm}$).
  4. Compute all POM distances strictly from physical dot coordinates.
- **Statistical Outputs:**
  - True physical re-lay variance ($\sigma_{\text{lay}}^2 = u_B^2$) per fabric class and POM.
  - Direct measurement of the **bias introduced by vacuum and platen restraints relative to the free state**.
  - Cycle time per lay across protocols.

---

### Stage S4: Definitional Uncertainty of Certified Experts on a Fixed Lay
- **Objective:** Quantify definitional uncertainty ($u_A$) among qualified human inspectors and establish the theoretical accuracy ceiling.
- **Protocol:**
  1. A production garment is laid on the table and **clamped immovably** at its edges. Repeat camera captures verify zero physical motion ($\Delta x < 0.02\text{ mm}$).
  2. Five certified QC inspectors independently mark every POM endpoint using invisible UV ink pens under black light. Because ink is invisible under ambient light, each inspector works completely blind to the markings of the others `[E method, Kouchi2011]`.
  3. The same inspectors subsequently view high-resolution white-light digital images of the fixed garment on calibrated 4K monitors and click the perceived POM coordinates.
- **Outputs:**
  - Physical inter-expert standard deviation ($u_{A,\text{physical}}$) per landmark.
  - Digital on-screen inter-expert standard deviation ($u_{A,\text{digital}}$).
  - Absolute divergence between competing standard definitions (e.g., ISO 18890 edge-to-edge vs ISO 3759 seam-to-seam).
- **Pass Gate G2:** For a POM to be feasible for automation, its definitional uncertainty must satisfy $u_A \le 0.8\text{ mm}$ for major POMs and $u_A \le 0.4\text{ mm}$ for tight POMs.

---

### Stage S5: Observability and Optical Information Ceiling
- **Objective:** Determine whether the information required to locate POM endpoints is physically present in the optical image, separating sensing failure from AI model failure.
- **Protocol:** Compare the digital on-screen expert clicks (from S4) against the true physical UV consensus marks (from S4) across imaging modalities:
  - Modality 1: High-res RGB diffuse illumination;
  - Modality 2: Multi-spectral raking illumination;
  - Modality 3: Backlit transmission illumination;
  - Modality 4: 3D structured-light depth map.
- **Metrological Deduction:**
  - If expert digital clicks closely match physical UV marks ($\Delta \le u_A$), **the optical information is present**. Any downstream automated failure is an **AI / model failure**.
  - If expert digital clicks are scattered or systematically biased away from physical marks, **the feature is optically unobservable from above**. No neural network can locate it; the system requires physical fiducials (A9) or a redefined POM.

---

### Stage S6: Automated Model Semantic Localization vs Ground Truth
- **Objective:** Evaluate candidate machine learning and geometric algorithms strictly against physical UV ground truth on held-out garments.
- **Candidate Algorithms Evaluated:**
  - *Pipeline Baseline (Previous System):* HRNet keypoint detector trained on synthetic Gemini images + post-hoc snapping.
  - *Pipeline Alpha (Three-Layer Decoupled):* Lightweight YOLO/RT-DETR semantic proposal + full-resolution Steger sub-pixel contour fitting + GPS operator construction.
  - *Pipeline Beta (Collaborative A7):* Human confirms proposal ROI on touch display + full-resolution sub-pixel contour snapping.
- **Data Partitions:** Strictly held-out factory production garments; stratified across seen styles, unseen styles, novel fabrics, and green backgrounds.
- **Outputs & Metrics:**
  - Coordinate error distribution ($e_x, e_y$) against UV ground truth;
  - 95th and 99th percentile errors;
  - **Gross-Error Rate ($P(|e| > t/2)$);**
  - **Selective Classification Risk-Coverage Curve `[E, GeifmanElYaniv2017]`:** Error rate as a function of coverage when low-confidence POMs are abstained.

---

### Stage S7: End-to-End Measurement System Analysis (MSA) and Decision Risk Study
- **Objective:** Execute an industrial-grade conformity assessment study benchmarking the fully integrated automated station against contractual manual inspection and traceable ground truth.
- **Experimental Design:** Fully crossed Gage R&R factorial experiment:
  - $N = 30$ garments per family (spanning production tolerance distribution, including seeded defects);
  - $p = 5$ certified QC operators;
  - $r = 3$ independent re-lays per garment;
  - 2 Parallel Methods: (1) Candidate Automated Station; (2) Contractual Manual Tape Method per ISO 18890.
  - Ground Truth: In-situ UV-marked photogrammetric reference (REF-4).
- **Statistical Deliverables:**
  - Total $\%GRR$, $ndc$, and VDA 5 $Q_{MP}$ capability ratios;
  - Two One-Sided Equivalence Tests (TOST) against the manual baseline;
  - Deming regression slope on seeded defects ($\beta \in [0.95, 1.05]$);
  - JCGM 106 Consumer's Risk ($R_C$) and Producer's Risk ($R_P$) under actual factory production variance.

---

## 5. Statistical Analysis Plan

### 5.1 Linear Mixed-Effects Model (REML)
For a given POM measured on garment $g$ by operator $i$ in re-lay $j$, replicate capture $k$:

$$y_{gijk} = \mu + G_g + O_i + (GO)_{gi} + L_{gij} + \epsilon_{gijk}$$

where:
- $\mu$: Grand mean dimension.
- $G_g \sim \mathcal{N}(0, \sigma_G^2)$: Random effect of true garment-to-garment production variance.
- $O_i \sim \mathcal{N}(0, \sigma_O^2)$: Random effect of operator bias.
- $(GO)_{gi} \sim \mathcal{N}(0, \sigma_{GO}^2)$: Operator-by-garment interaction.
- $L_{gij} \sim \mathcal{N}(0, \sigma_L^2)$: Garment state re-lay variance ($u_B^2$).
- $\epsilon_{gijk} \sim \mathcal{N}(0, \sigma_\epsilon^2)$: Static optical/sensor repeatability ($u_C^2 + u_D^2$).

Parameters are estimated via **Restricted Maximum Likelihood (REML)**. Confidence intervals for variance components are derived via Satterthwaite approximation or parametric bootstrap `[E, Burdick2005]`.

### 5.2 Deming Regression on Seeded Defects
To audit candidate systems against shrinkage toward nominal, measurements of seeded defects ($y$) are regressed against traceable reference values ($x$) via Deming regression, which accounts for reference measurement error $\sigma_x^2$:

$$y = \beta x + \alpha$$

$$\hat{\beta} = \frac{s_{yy} - \lambda s_{xx} + \sqrt{(s_{yy} - \lambda s_{xx})^2 + 4\lambda s_{xy}^2}}{2 s_{xy}}, \quad \lambda = \frac{\sigma_\epsilon^2}{\sigma_x^2}$$

- **Hypothesis Test:** $H_0: \beta = 1.0$. If the $95\%$ confidence interval for $\beta$ falls strictly below $0.95$, the system is rejected for **prior shrinkage bias**.

---

## 6. Diagnostic Root-Cause Attribution Flowchart

When a candidate system exhibits measurement discrepancies during validation, the flowchart below deterministically isolates the responsible subsystem `[I]`:

```
                           OBSERVED MEASUREMENT ERROR
                                       │
            Is S1 (Rigid Artefacts) Error > 0.25 mm or Scaling Drifting?
                          ┌────────────┴────────────┐
                        YES                         NO
                         ▼                          │
            [SENSING / OPTICS FAILURE]              │
            • Lens distortion model invalid         ▼
            • Camera mount rigidity / vibration     Is S2 Edge Bias Large Across
            • Perspective parallax uncorrected        Different Fabric Colors/Plies?
                                                   ┌────────────┴────────────┐
                                                 YES                         NO
                                                  ▼                          │
                                     [FEATURE EXTRACTION FAILURE]            │
                                     • Switch to Backlight Silhouette        ▼
                                     • Implement Raking Illumination   Is S3 Re-lay Spread >> Static
                                     • Model fabric thickness fold       Repeatability on Marked Units?
                                                                        ┌────┴────┐
                                                                      YES         NO
                                                                       ▼          │
                                                          [GARMENT STATE FAILURE] │
                                                          • In-plane shear locked ▼
                                                          • Wrinkles shortening   Is S4 Expert Marking Spread
                                                          • MUST USE VACUUM /      Large on Fixed Lay?
                                                            PLATEN FIXTURE (A6)  ┌────┴────┐
                                                                               YES         NO
                                                                                ▼          │
                                                                   [DEFINITIONAL FAILURE]  ▼
                                                                   • Ambiguous tech pack  Did S5 Pass but S6 Fails
                                                                   • Incomplete GPS spec    on Automated Neural Model?
                                                                   • REWRITE OPERATOR    ┌────┴────┐
                                                                                       YES         NO
                                                                                        ▼          ▼
                                                                                   [AI FAILURE] [SYSTEM
                                                                                   • Downsampling PASSES]
                                                                                   • Domain gap
                                                                                   • USE A7
```

---

## 7. Formal Go/No-Go Decision Gates

The feasibility program incorporates explicit, quantitative stopping gates. If a gate fails, engineering activity on downstream machine learning must halt immediately `[I]`:

| Gate | Evaluation Point | Quantitative Pass Criteria | Consequence / Action if Gate Fails |
|---|---|---|---|
| **GATE G1: Sensing Feasibility** | End of Stage S1 | Static repeatability $\sigma \le 0.10\text{ mm}$; Field-wide scale trueness error $\le 0.05\%$ ($<0.35\text{ mm}$ over $700\text{ mm}$); Height parallax corrected to $<0.20\text{ mm}$. | **STOP.** Redesign optical gantry, increase mount height to $H \ge 2.2\text{ m}$, replace lens, or implement in-situ fiducial grid frame. Do not proceed to textiles. |
| **GATE G2: Physical & Definitional Viability** | End of Stage S4 | For core body POMs ($\pm 1/2''$): $\sqrt{u_A^2 + u_B^2} \le 1.20\text{ mm}$. For tight POMs ($\pm 1/4''$): $\sqrt{u_A^2 + u_B^2} \le 0.65\text{ mm}$ under at least one physical protocol (Free or Vacuum). | **STOP AI DEVELOPMENT.** Software cannot solve this. If $u_A$ fails, negotiate rewritten GPS operators with buyer. If $u_B$ fails, mandate perforated vacuum table (A6-Vac). If still failing, drop tight POMs from automated scope. |
| **GATE G3: Algorithmic Localization & Coverage** | End of Stage S6 | For accepted measurements: RMSE $\le 1.2 \times$ human on-screen expert spread ($u_A$); Gross error rate ($|e| > t/2$) $\le 0.5\%$; System abstention rate $\le 10\%$ in S-AQL. Seeded defect slope $\beta \ge 0.95$. | Deploy **Human-in-the-Loop Architecture (A7)**. Do not deploy full autonomous keypoint model to factory floor. Retrain models strictly on real production images marked with UV ground truth. |
| **GATE G4: Industrial System Acceptance** | End of Stage S7 | Total process capability $Q_{MP} \le 30\%$ (VDA 5); Equivalence to manual baseline proven via TOST ($\delta = 0.20 \cdot t$); Global Consumer Risk $R_C \le 0.5\%$; Cycle time $\le 45\text{ s}$ per garment. | System is certified for factory production deployment (S-AQL). Authorize pilot line roll-out. |
