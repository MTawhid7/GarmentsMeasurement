# 02 — Metrology Principles and the State of the Art

## 1. Transferable Principles from Precision Industrial Metrology

Precision dimensional measurement in semiconductor manufacturing, aerospace, and precision optical inspection achieves sub-micrometre to sub-nanometre reproducibility not by employing "better cameras", but through **rigorous experimental design, traceable calibration, complete geometric definitions, and physical state constraint**.

Nine fundamental metrological principles govern precision industries. Below, we analyze their transferability to deformable textile garments:

```
┌────────────────────────────────────────────────────────────────────────┐
│             NINE METROLOGY PRINCIPLES TRANSFERRED TO APPAREL           │
├────────────────────────────────────────────────────────────────────────┤
│ P1. Complete Measurand Specification (GPS ISO 17450)        ──► FULLY  │
│ P2. Physical State Control & Restraint (ISO 10579)          ──► FULLY  │
│ P3. Traceable Calibration via Rigid Standards (VDI 2634)    ──► FULLY  │
│ P4. Explicit Uncertainty Budgeting (GUM / VDA 5)            ──► FULLY  │
│ P5. Sub-Pixel Continuous Feature Fitting (Steger)           ──► FULLY  │
│ P6. Reversal Techniques to Cancel Systematic Bias (TIS)     ──► FULLY  │
│ P7. Reference Measurement Systems & TMU (Semiconductor)     ──► FULLY  │
│ P8. Optical Gauging Designed for Metrology (Transmission)   ──► FULLY  │
│ P9. Design-for-Metrology (In-Situ Fiducials)                ──► PARTLY │
└────────────────────────────────────────────────────────────────────────┘
```

### 1.1 Detailed Analysis of Transferable Principles

1. **Principle P1: Complete Measurand Specification (ISO 17450 / GPS):**
   - *How Precision Fields Use It:* Dimensional tolerances on rigid mechanical drawings are defined via complete mathematical operations: datum references, filtering cutoffs, and geometric associations (least-squares, Chebyshev minimum zone) `[E, ISO17450-2]`.
   - *Transfer to Apparel:* **Fully Transferable.** Rewriting ambiguous tech-pack phrases into deterministic GPS operators (Section 3 of Document 01) requires zero physical hardware and eliminates definitional uncertainty ($u_A$) at the source `[I]`.

2. **Principle P2: Physical State Control and Restraint (ISO 10579):**
   - *How Precision Fields Use It:* In aerospace sheet metal and automotive polymer inspection, non-rigid parts are measured in a *specified restrained condition* using dedicated clamping nests and vacuum platens that overcome gravity and part distortion `[E, ISO10579; Abenhaim2012]`.
   - *Transfer to Apparel:* **Fully Transferable, with Contractual Agreement.** The textile can be held flat on a perforated vacuum table or under a transparent compression platen. However, because restraining a garment introduces local friction and contact forces that alter its free-state dimensions, the restrained condition *becomes part of the specification* and must be agreed with buyers `[I]`.

3. **Principle P3: Traceable Calibration via Physical Artefacts (VDI/VDE 2634 / ISO 10360):**
   - *How Precision Fields Use It:* Optical coordinate measuring systems never rely on factory datasheets; they are calibrated in-situ using certified grid plates (invar or quartz glass) and calibrated ball bars tested across the entire measuring volume `[E, VDI2634; Luhmann2010]`.
   - *Transfer to Apparel:* **Fully Transferable for the Optical Sensing Chain.** The camera, lens distortion model, and planar coordinate frame can be calibrated and verified to $<0.05\text{ mm}$ using certified glass dot plates or calibrated scale bars placed on the measuring table `[I]`.

4. **Principle P4: Explicit Uncertainty Budgeting (GUM / ISO 14253-2 / VDA 5):**
   - *How Precision Fields Use It:* Every measurement is accompanied by an uncertainty budget ($u_c$) combining Type A (statistical dispersion) and Type B (systematic, thermal, calibration) components in quadrature `[E, JCGM106; ISO22514-7]`.
   - *Transfer to Apparel:* **Fully Transferable.** A garment measurement system must report standard uncertainty $u_c$, epistemic confidence, and gross-error probability for every single measured POM `[I]`.

5. **Principle P5: Sub-Pixel Continuous Feature Fitting over Point Probing:**
   - *How Precision Fields Use It:* Precision vision systems never locate an edge by a single thresholded pixel. They fit continuous mathematical models (error functions, polynomials, B-splines) to hundreds or thousands of sampled intensity gradients, suppressing random image noise by $\sim 1/\sqrt{N}$ `[E, Steger]`.
   - *Transfer to Apparel:* **Fully Transferable to Silhouette Edges and Continuous Seams.** Fitting an analytical line or curve across an entire sleeve fold or hemline reduces sensor pixel jitter to $<0.02\text{ mm}$ `[I]`.

6. **Principle P6: Reversal Techniques to Cancel Tool Bias (Tool Induced Shift - TIS):**
   - *How Precision Fields Use It:* In semiconductor lithography overlay metrology, Tool Induced Shift (TIS) is isolated and eliminated by measuring the wafer at $0^\circ$ and $180^\circ$ rotation:
     $$\text{TIS} = \frac{1}{2} \left[ \text{Measurement}(0^\circ) + \text{Measurement}(180^\circ) \right]$$
     This separates true wafer asymmetry from illumination telecentricity errors or optical coma `[E, TIS]`.
   - *Transfer to Apparel:* **Fully Transferable as a Diagnostic.** Capturing the identical garment lay rotated $180^\circ$ on the inspection table immediately isolates lighting-direction bias, camera perspective tilt, and algorithmic asymmetry `[I]`.

7. **Principle P7: Reference Measurement Systems and Total Measurement Uncertainty (TMU):**
   - *How Precision Fields Use It:* Semiconductor CD metrology validates fast production CD-SEMs against slow, highly accurate reference systems (e.g., CD-AFM). Banke & Archie's Total Measurement Uncertainty (TMU) framework uses Mandel errors-in-variables regression to remove the reference system's own variance from the evaluation `[E, TMU]`.
   - *Transfer to Apparel:* **Fully Transferable to Pilot Validation.** The system under test cannot be evaluated against an uncharacterized human inspector with a flexible tape. It must be benchmarked against an optical reference measuring physical targets placed by consensus `[I]`.

8. **Principle P8: Optical Metrology Designed for Gauging (Transmission Silhouette):**
   - *How Precision Fields Use It:* 2D optical gauges (e.g., InspecVision Planar) achieve $\pm 35\ \mu\text{m}$ accuracy across large fields ($1830 \times 1220\text{ mm}$) by placing parts on an ultra-uniform LED backlight diffusing table, capturing a pure transmission silhouette with a high-resolution 50 MP camera `[E, InspecVision]`.
   - *Transfer to Apparel:* **Directly Transferable.** Transmission backlighting eliminates fabric surface color, print patterns, and room lighting reflections, converting outer boundary measurement into a pure optical contrast problem `[I]`.

9. **Principle P9: Design-for-Metrology (In-Situ Fiducial Marking):**
   - *How Precision Fields Use It:* Silicon wafers carry dedicated optical alignment targets in scribe lines; aerospace composite plies carry laser-etched fiducials `[E, Boeing-ply]`.
   - *Transfer to Apparel:* **Partially Transferable.** In production, visible marks are unacceptable to fashion brands. However, for validation studies, blind marking with invisible UV-fluorescent inks is viable. In cutting rooms, automated inkjet heads could place UV micro-dots inside seam allowances `[H]`.

---

### 1.2 Why Nanometre / Micrometre Precision Does NOT Transfer to Textiles

While the *principles* transfer completely, the *numerical accuracy* ($\mu\text{m}$ to $\text{nm}$) **physically cannot transfer to garments**. The physical reasons are structural and unavoidable `[I]`:

```
┌─────────────────────────────────┬─────────────────────────────────┐
│     RIGID METROLOGY DOMAIN      │    TEXTILE APPAREL METROLOGY    │
├─────────────────────────────────┼─────────────────────────────────┤
│ • Linear elastic solids         │ • Highly non-linear viscoelastic│
│   (E ~ 70 to 210 GPa)           │   (E ~ 0.001 to 0.1 GPa)        │
│ • Deterministic thermal drift   │ • Hygral expansion + moisture   │
│   (α ~ 10 to 25 ppm/K)          │   swelling (up to 3% length)    │
│ • Macroscopically sharp edges   │ • Rounded fabric folds, fuzz,   │
│   (step function at sub-µm)     │   knitting loop serrations      │
│ • Geometry independent of lay   │ • Friction-dominated hysteresis;│
│   history                       │   handling strain frozen in     │
└─────────────────────────────────┴─────────────────────────────────┘
```

1. **Viscoelastic Compliance & Hysteresis:** Unlike metals or silicon, fabrics undergo significant in-plane deformation under forces as low as $5\text{ gf/cm}$ ($0.05\text{ N/cm}$) `[E, FAST/KES]`. The friction between fabric and table prevents elastic recovery, freezing in arbitrary shear strains.
2. **Boundary Raggedness (Fibre Fuzz & Fold Curvature):** A textile edge is not a mathematical plane. It is either a cut yarn boundary with projecting fiber hairs ($0.2\text{ mm to }1.5\text{ mm}$ length) or a folded fabric tube with a finite bend radius ($r \approx 0.5\text{ mm to }2.0\text{ mm}$). Optical edges shift based on lighting angle.
3. **Hygral Expansion and Relaxation:** Natural fibres (cotton, wool) swell with ambient relative humidity. A change of $30\%\text{ RH}$ can alter garment dimensions by $0.5\%\text{ to }1.5\%$ ($3.5\text{ mm to }10\text{ mm}$ over a $700\text{ mm}$ length), requiring strict ISO 139 climate conditioning to stabilize `[E, ISO3759; Munden1959]`.

---

## 2. Lessons from Soft-Tissue and Deformable Object Measurement

Examining how other disciplines measure flexible, ambiguous, or organic structures yields critical empirical insights:

| Domain | Object & Measurand | Achieved Precision | Transferable Lesson for Garments |
|---|---|---|---|
| **3D Anthropometry (ISO 20685-1)** | Human body; landmark-defined girths, widths, and lengths | Landmark localization error: **$3\text{ mm to }15\text{ mm}$**; Body dimensions: $\pm 4\text{ mm to }\pm 9\text{ mm}$ `[E, ISO20685-1; Kouchi2011]` | **1. Inter-observer landmark placement error exceeds the resulting dimension error.** Kouchi & Mochimaru showed that for 15 of 34 dimensions, expert landmarkers disagreed significantly.<br>**2. Blind Marking Methodology:** Experts must mark landmarks using invisible UV-fluorescent ink so subsequent evaluators cannot see previous placements `[E, Kouchi2011]`. |
| **Cephalometric Radiography (Medical AI)** | Lateral skull X-rays; 19 anatomical landmarks | Inter-clinician spread: **$1.38 \pm 1.55\text{ mm}$**. Best AI models (ISBI benchmark): Mean Radial Error (MRE) $\approx 1.8\text{ mm}$, but **$\sigma = 6.55\text{ mm}$** `[E, Wang2016-ISBI; Khalid2025]` | **Learned models hit an asymptotic ceiling set by human inter-observer variability.** Crucially, AI error distributions are **heavy-tailed**: average error looks acceptable, but rare gross errors ($\sim 10\text{ mm}$) persist. System must include an abstention mechanism `[I]`. |
| **Marine Conveyor Inspection (CatchMeter)** | Unconstrained fish on high-speed conveyor belt | Tip-to-tail length standard deviation: **$\sigma = 1.2\text{ mm}$** `[E, CatchMeter]` | **Deformable bodies CAN be measured with millimetre accuracy on conveyors IF the boundary features are physically unambiguous** (snout tip and tail fork). Deformability is solvable; ambiguous landmark definition is the true bottleneck `[I]`. |
| **Aerospace Composite Ply Inspection** | Carbon-fibre prepreg fabric plies placed on tooling | Boundary localization: Sub-millimetre using laser-line projection and CAD boundary matching `[E, Boeing-ply]` | Fabric plies can be measured accurately when held against a rigid tool surface and compared to explicit CAD boundaries under constrained geometry `[I]`. |
| **Robotic Cloth Manipulation** | Garment keypoint detection for robotic folding | Mean Keypoint Distance: **$8.7\text{ px to }18.1\text{ px}$** at $5\text{ mm/px}$ = **$4.3\text{ cm to }9.0\text{ cm}$** `[E, Lips2024]` | **Computer-vision cloth keypoint models operate at CENTIMETRE-scale accuracy.** They are engineered for robotic grasping affordances, not dimensional metrology. Treating manipulation keypoints as metrology tools is a category error `[I]`. |

---

## 3. State of the Art in Garment Measurement

### 3.1 Academic Literature Review

A thorough evaluation of peer-reviewed computer vision literature on garment measurement reveals a pervasive methodology gap:

```
┌────────────────────────────────────────────────────────────────────────┐
│              COMMON METHODOLOGICAL FLAWS IN ACADEMIC PAPERS            │
├────────────────────────────────────────────────────────────────────────┤
│ 1. Small sample sizes (n = 10 to 50 garments)                          │
│ 2. Uncalibrated manual tape used as unvalidated "ground truth"         │
│ 3. Severe image downsampling (e.g. 128x128 or 224x336 pixels)          │
│ 4. Single static lay capture; zero separation of re-lay variance ($u_B$)│
│ 5. Conflating relative error (% of length) with industrial tolerance   │
└────────────────────────────────────────────────────────────────────────┘
```

- **Paulauskaite-Taraseviciene et al. (2022) `[E]`:**
  - Evaluated automatic garment measurement using UNet segmentation and heuristic mask corner detection.
  - Images were downsampled to **$224 \times 336$ pixels**; UNet mask resolution was **$128 \times 128$ pixels**.
  - Across a $1.2\text{ m}$ garment field, a $128 \times 128$ mask imposes an intrinsic spatial quantization of **$9.4\text{ mm per pixel}$** `[I]`.
  - Reported Mean Absolute Error (MAE) of $0.75\text{ cm to }1.27\text{ cm}$ ($7.5\text{ mm to }12.7\text{ mm}$) against a manual tape.
  - Crucially, the authors conducted an informal trial with 20 untrained human participants measuring a jacket, observing human measurement discrepancies up to **$3.02\text{ cm}$**! This confirmed that human manual tape measurement has massive uncharacterized variance.

- **Kim et al. (2022) `[E]`:**
  - Employed an iPad Pro LiDAR and RGB camera to measure 33 flat-laid garments across 5 styles.
  - Applied an **HRNet-W48 keypoint detector** to RGB images and sampled LiDAR 3D mesh points for metric distance.
  - Conducted **10 repeated re-lays per garment**—one of the few studies to test garment state variability.
  - Reported average relative errors of $1.59\%\text{ to }2.08\%$ (equivalent to **$8\text{ mm to }15\text{ mm}$ on body dimensions**).
  - Concluded that manual measurement had an average error of $3.25\%$, but failed to separate optical calibration error from fabric handling strain.

- **DeepFashion2 (Ge et al., 2019) `[E]`:**
  - The standard benchmark for clothing landmark estimation (801K items, Match R-CNN).
  - Landmarks are purely semantic (e.g., "approximate sleeve cuff point"). Label annotations exhibit human annotator variance of $5\text{ to }15\text{ pixels}$. It has zero metric calibration or dimensional ground truth.

### 3.2 Patent and Commercial Landscape

- **US Patent 5,530,652 (Levi Strauss & Co., 1993, Expired) `[E]`:**
  - Discloses an automatic garment inspection system where garments pass on an entry conveyor under CCD line-scan/area cameras.
  - Outlines are extracted, segmented, and compared against stored CAD cutting pattern templates.
  - A 3D embodiment inflates trousers with pneumatic air jets on a rotating mannequin to measure seams in tension. Contained no empirical uncertainty data.
- **Yunji Zhihui (Dongguan, China, 2021) `[E trade press]`:**
  - Marketed a commercial flat-bed AI measuring machine for T-shirts, claiming "$\pm 1\text{ mm}$ accuracy" in $6\text{ seconds}$ on a $900 \times 900\text{ mm}$ glass bed `[YunjiZhihui2021]`.
  - Independent technical validation is unavailable. Note that a $900\text{ mm}$ bed cannot accommodate adult trousers ($1050\text{ mm}$ outseam) or outstretched long-sleeve shirts ($1600\text{ mm}$ wingspan) `[I]`.
- **Size AI (2026 Commercial App) `[E vendor]`:**
  - Uses mobile LiDAR/RGB for flat-lay apparel measurements.
  - The vendor explicitly notes in their technical documentation:
    > *"~5mm accuracy is typical with LiDAR under optimal conditions... Size AI is designed as a screening layer, not a replacement for certified tech pack measurement... major wrinkles can shift measurements by 2–3 cm."* `[SizeAI-QC; SizeAI-cheatsheet]`
  - This candid vendor admission confirms that unconstrained optical flat-lay is fundamentally limited by wrinkle physics and depth resolution.

---

## 4. Deep Autopsy of the Previous Team's Architecture

The previous engineering team spent two years developing an apparel measurement system, reaching an asymptotic ceiling of ~80% accuracy. Their stack consisted of:
- **Hardware:** Zivid 3D structured-light camera ($\sim €15,000$ class) and high-end GPU workstations.
- **Computer Vision Pipeline:** RMBG background removal / segmentation, deep keypoint detection, and heuristic post-hoc "keypoint correction algorithms".
- **Training Data:** Thousands of synthetic flat-lay images generated via Gemini / Nano Banana on a white background.
- **Testing Environment:** Physical flat-lay on a green cloth backdrop.

By evaluating this architecture from first principles of metrology and computer vision, we can diagnose the exact physical and mathematical root causes of their failure `[I]`:

```
┌────────────────────────────────────────────────────────────────────────┐
│                   WHY THE PREVIOUS PIPELINE FAILED                     │
├────────────────────────────────────────────────────────────────────────┤
│ 1. Zivid 3D Camera: Coarse 2D resolution (0.44 mm/px); scale trueness  │
│    spec (±0.35-0.60%) introduces 2.5-4.2 mm error; depth noise causes  │
│    surface geodesic length inflation (+26% at raw resolution).         │
├────────────────────────────────────────────────────────────────────────┤
│ 2. Gemini/Nano Banana Synthetic Images: Text-to-image diffusion lacks  │
│    projective geometry; seam topologies and proportions violate CAD    │
│    rules; annotations represent visual guesses with zero metric truth. │
├────────────────────────────────────────────────────────────────────────┤
│ 3. Domain Gap & Green Sheet: White synthetic training vs real green    │
│    sheet caused chroma bleed onto fabric edges; RMBG downsampled to    │
│    1024x1024 (1.2 mm/px), blurring physical silhouette boundaries.     │
├────────────────────────────────────────────────────────────────────────┤
│ 4. Keypoint Abstraction: HRNet-style backbones downsample into coarse  │
│    heatmaps (12-19 mm cells). Keypoints are semantic, not metrological.│
├────────────────────────────────────────────────────────────────────────┤
│ 5. Post-Hoc Correction Algorithms: Snapped points to expected shapes,  │
│    acting as strong priors that masked true manufacturing defects      │
│    (shrinkage toward nominal).                                         │
└────────────────────────────────────────────────────────────────────────┘
```

### 4.1 Root Cause 1: Structural Limitations of the Zivid 3D Camera for Flat-Lay Metrology
The Zivid 2+ (L110 / M130) is an exceptional structured-light sensor designed for robotic bin-picking and discrete parts assembly. However, for 2D flat-lay garment inspection, it introduces severe metrological penalties:
- **Spatial Sampling Coarseness:** The Zivid 2+ L110 color sensor provides $2448 \times 2048$ pixels covering a $1090 \times 850\text{ mm}$ field at its recommended working distance ($1.1\text{ m}$). This corresponds to **$0.44\text{ mm per pixel}$** `[E, Zivid2+L110]`. In contrast, a standard €800 24 MP industrial 2D camera provides $0.20\text{ mm/pixel}$, and a 61 MP sensor provides $0.13\text{ mm/pixel}$.
- **Manufacturer Trueness Specification:** The Zivid 2+ datasheet specifies a **dimension trueness error of $< 0.35\%$ (70th percentile) at focus distance, $< 0.40\%$ in optimal range, and $< 0.60\%$ over temperature** `[E, Zivid2+L110]`. Over a $700\text{ mm}$ body dimension, a $0.35\%\text{ to }0.60\%$ trueness error represents an inherent systematic scale bias of **$2.45\text{ mm to }4.20\text{ mm}$**! This error alone consumes the entire uncertainty budget.
- **Geodesic Path Length Inflation:** If the team attempted to compute surface path lengths across the 3D point cloud to "measure along wrinkles", depth sensor noise ($\sigma_z \approx 0.26\text{ mm}$) systematically inflates Euclidean arc length. As proven in Chapter 03, summing noisy 3D point steps inflates calculated length by **$+26.6\%$** at raw resolution `[I]`.
- **Field of View Deficit:** The $1090 \times 850\text{ mm}$ FOV cannot capture adult trousers ($1100\text{ mm}$ long) or outstretched long-sleeve shirts ($1600\text{ mm}$ wingspan) without mounting the sensor far outside its optimal working volume, which degrades point precision.

### 4.2 Root Cause 2: Flaws of Text-to-Image (Gemini / Nano Banana) Synthetic Data
The use of generative diffusion models (Gemini 2.5 Flash Image / "Nano Banana") to create training images introduced fatal geometric distortions:
- **Violation of Projective and Sewing Geometry:** As demonstrated by Sarkar et al. (CVPR 2024, *Shadows Don't Lie and Lines Can't Bend!*), generative diffusion models **do not understand 3D projective geometry or physical constraints** `[E, Sarkar2024]`. In generated flat-lays, seam lines bend arbitrarily, left and right sleeves exhibit asymmetrical foreshortening, and fold lines do not obey the topological rules of stitched 2D pattern assemblies.
- **Absence of Ground Truth:** Generated images have no underlying physical garment, no CAD pattern, and no physical measurement. Labels placed on synthetic images are purely visual guesses drawn by human annotators. Training a keypoint detector on these annotations trains the model to replicate **annotator hallucination**, not true physical seam junctions `[I]`.
- **Severe Sim-to-Real Domain Gap:** Lips et al. (RA-L 2024) demonstrated that even using physically accurate FEM cloth simulators (Blender/MuJoCo), synthetic training produces heavy error tails when tested on real fabric `[E, Lips2024]`. With unconstrained diffusion generation, this domain gap is drastically wider.

### 4.3 Root Cause 3: Domain Shift and the Green-Sheet Chroma Trap
- **Chroma Bleed and Edge Spill:** Spreading real garments on a green background to enable chroma-key segmentation created destructive optical interactions. Light reflecting off the green sheet casts diffuse green light onto the rounded fabric fold edges. On light, white, or textured fabrics, this chroma spill shifts the apparent intensity gradient, inducing sub-millimetre to millimetre edge extraction errors `[I]`.
- **RMBG Segmentation Bottleneck:** RMBG-1.4 (IS-Net) and RMBG-2.0 (BiRefNet) downsample input images to **$1024 \times 1024$ pixels** `[E, RMBG-1.4; RMBG-2.0]`. Over a $1.2\text{ m}$ table, this yields an effective resolution of **$1.17\text{ mm per mask pixel}$**. The resulting soft alpha boundary represents a neural network's visual probability of foreground presence, not a metrologically calibrated physical boundary.

### 4.4 Root Cause 4: The Keypoint Abstraction Bottleneck
- Standard keypoint architectures (HRNet, SimpleBaseline) were designed for human pose estimation.
- HRNet downsamples input images through two strided convolutions into heatmaps at **$1/4$ input resolution** (e.g., $64 \times 48$ cells for a $256 \times 192$ input) `[E, HRNet]`.
- Mapped across a $1200\text{ mm}$ garment, each heatmap cell spans **$18.8\text{ mm}$**! Even with DARK (Distribution-Aware Coordinate Representation) sub-pixel decoding `[E, DARK]`, heatmap quantization and label noise limit localization accuracy to several millimetres.
- A keypoint is a 0D point abstraction. In contrast, garment POMs are defined by 1D contours and seam line intersections. By discarding the continuous contour information, the keypoint network throws away the sub-pixel averaging benefits of classical metrology `[I]`.

### 4.5 Root Cause 5: Post-Hoc "Correction" Algorithms Masked Real Defects
- To counter displaced keypoints, the previous team implemented post-hoc correction algorithms that snapped detected points toward expected geometric templates.
- **Metrological Consequence:** In quality control, an algorithm must never pull measurements toward the nominal design! A correction rule that snaps an anomalous armhole point back to its "expected" location acts as an aggressive Bayesian prior. It artificially forces bad garments to appear conforming (**shrinkage toward nominal**), masking true manufacturing defects and generating unacceptable Consumer's Risk `[I]`.

---

## 5. Synthesis: The State of the Art vs Metrology Reality

| Dimension | Previous AI Approach | Rigorous Metrology Reality |
|---|---|---|
| **Sensor Philosophy** | Expensive 3D sensor (€15k Zivid) assumed to solve geometry | High-resolution calibrated 2D sensor with transmission backlighting & multi-angle raking lights (€3k–€5k) |
| **Training Pipeline** | Synthetic diffusion images (Gemini) on white background | Real production garments marked with invisible UV fiducials; exact pattern CAD priors |
| **Edge Segmentation** | RMBG neural background removal on green cloth ($1024\times 1024$) | Physical transmission silhouette (backlight) at full optical resolution ($6000\times 4000$) |
| **Feature Extraction** | 0D Keypoint heatmaps (HRNet, $18.8\text{ mm}$ cells) | Continuous sub-pixel curvilinear seam and edge tracking (Steger operator) |
| **Geometric Evaluation** | Direct Euclidean distance between predicted keypoints | Formal GPS operators: datums, perpendicular construction, intersection evaluation |
| **Anomaly Handling** | Heuristic post-hoc "correction" (snapping to template) | Per-POM epistemic uncertainty output with **selective abstention** |
| **Physical Garment State**| Uncontrolled manual lay; assumed flat | Standardized lay protocol + active vacuum/platen restraint + flatness gating |
