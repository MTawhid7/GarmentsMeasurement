# 03 — What physical information is required, and what limits it

*Consolidated research, v2. This document answers **Question 2: what physical information is needed to measure each POM, and which physical, optical, definitional or algorithmic effects limit it?** Evidence labels: [E] source opened · [E-sec] secondary · [E-vendor] manufacturer claim · [I] inference (derivation shown) · [H] hypothesis. The numbers are first-principles estimates under stated assumptions, **not** measured garment performance.*

---

## 1. Identifying a feature is not measuring it

Three different operations are routinely conflated:

| Operation | Question it answers | Required precision | Suitable tools |
|---|---|---|---|
| **Semantic identification** | *Which* feature is this — the armhole seam, the side-neck region, the front hem? | Coarse: a region of roughly 1–2 cm, plus correct topology | Learned models (detection, segmentation, keypoint proposals); human judgement |
| **Metrological localisation** | *Where exactly* is the endpoint defined by the POM's complete operator (document 01 §3.3)? | Sub-millimetre to millimetre | Geometric fitting at full sensor resolution on physical features (edges, seam lines, intersections); physical marks; validated dense feature detectors |
| **Evaluation** | What is the distance, in the specified state, per the operator? | Set by the uncertainty budget | Calibrated geometry; height correction; decision rule |

A model can **identify** the sleeve-end region, the shoulder seam or the collar boundary reliably and still be unfit to **measure** them, for four reasons:

1. **Resolution.** Common keypoint architectures work on heavily downsampled crops. Over a 1.2 m garment one output cell spans ≈ 12–19 mm (§5) [I from E, HRNet].
2. **Label definition.** A model learns whatever definition its annotators used, including their disagreement. Nothing in training forces the label onto the physical feature the POM operator names [I].
3. **Heavy tails.** Learned localisers are usually close and occasionally badly wrong (the wrong seam, a crease) [E analogue, Wang2016-ISBI; Lips2024].
4. **No uncertainty per output** unless one is explicitly built, calibrated and validated [E, Jonkers2025].

**Architectural consequence [I].** Machine learning should do semantic identification and may propose regions. The **final coordinate must come from a procedure whose bias, precision and failure behaviour have been characterised against a reference** (document 07). In practice that favours deterministic geometric fitting at full resolution — analysable, and averaging over many pixels. A learned *dense* feature detector at full resolution (e.g. a seam-line probability map) is acceptable as an extraction step **if** it is characterised like any other instrument. The rule is not "no ML in the metric layer". It is **"no uncharacterised component produces the number."**

---

## 2. Error model

A reported dimension is the output of a chain. To first order (GUM style, independent components):

u_c² ≈ u_A² + u_B² + u_C² + u_D² + u_E² + u_F²

| Code | Component | Physical origin |
|---|---|---|
| **A** | Definitional | Incomplete POM operator (feature boundary, direction, filtering) |
| **B** | Garment state | Handling strain, wrinkles and buckles, seam roll, fold placement, relaxation, moisture |
| **C** | Sensing | Sampling, noise, distortion residuals, calibration scale, perspective/height, depth noise |
| **D** | Feature extraction | Edge definition (fold radius, fuzz), seam visibility, lighting-dependent edge position, background spill |
| **E** | Semantic localisation | Choosing the wrong feature (gross errors); coarse learned localisation if used as the final coordinate |
| **F** | Prior / constraint bias | Shrinkage toward nominal (CAD templates, symmetry rules, "correction" heuristics) |
| **G** | Reference error | Uncertainty of the "truth" used in validation (it inflates *apparent* error only) |

**What follows [I]:**
- **A and B are properties of the garment and its specification, not of the instrument.** No camera or network reduces them. They also limit the manual tape.
- **E is heavy-tailed.** A Gaussian budget describes its core. The tail must be handled by detection and abstention and reported as a gross-error rate.
- **G misleads validation.** A system with u = 1.0 mm judged against a tape reference with u = 3.0 mm shows ≈ 3.2 mm of apparent disagreement.

---

## 3. Physical information required: observability of endpoints

| Endpoint type | Examples | Physical cue | Revealed by | Typical failure |
|---|---|---|---|---|
| **Cut or finished edge** | Hem, cuff, leg opening | Material boundary | Backlit silhouette (independent of colour); front-light contrast | Fuzz or pile; curled or flared rib; shaped hems (which layer?) |
| **Fold edge** | Chest or waist "edge to edge"; sleeve and thigh widths | Bend of a flattened tube | Backlight; front light | Position depends on how the tube was flattened; rounded fold; seam roll near the fold |
| **Visible seam line** | Shoulder, armhole, yoke, waistband seams | Stitch row + seam ridge or groove | **Raking light** (topography), texture, height map | Dark-on-dark thread, fleece or pile, fine gauge, prints |
| **Seam junction** | SNP (shoulder ∩ neck seam), underarm point, crotch point | Intersection of two seams | Intersect *fitted* seam curves geometrically | **Often under a layer or at a fold:** the SNP is defined excluding collar and trim [E, ISO18890], so it can lie *under* the collar; the underarm point is a multi-ply junction that lands on a fold; the crotch point sits in a fold between the legs |
| **Constructed point** | "2.5 cm below armhole", a waist "required position", "halfway" knee | Arithmetic from another point plus a direction | Computation | Inherits the anchor's uncertainty; the **direction** is often unspecified |
| **Centre line** | CF, CB | Placket, buttons, symmetry | Features; fitted symmetry axis | Unmarked backs; real asymmetry must not be symmetrised away |
| **Layer boundary** | Front hem over a longer back tail | Change in the number of layers | Front light (a step edge, often low contrast); **transmission** shows a layer-count change in thin fabrics | Opaque fabrics give only the outer silhouette (the union of layers) |
| **Force-defined** | Waist or neck extended | Dimension under a specified extension | **Nothing passive.** Actuation with force control | Maximum-extension definitions are force-ambiguous |

**Physical conclusion [I].**
- Edges, folds and clearly visible seams are physically well defined once the operator states which feature counts. Their residual uncertainty is sub-millimetre to about a millimetre and fabric-dependent.
- Seam junctions under layers and constructed points carry most of the definitional risk.
- The anthropometry analogue predicts an expert spread of about 1–2 mm that dimension-level comparisons will hide [E analogue, Kouchi2011; H for garments].
- Force-defined POMs are outside passive sensing.

---

## 4. Sensing physics (component C)

### 4.1 Sampling

| Sensor example | Field (long axis) | mm per pixel |
|---|---|---|
| Zivid 2+ L110 colour image, 2448 px [E, Zivid2+L110] | 1090 mm at 1.1 m | **0.44** (datasheet) |
| Zivid 2+ M130 colour image, 2448 px [E, Zivid2+M130] | 790 mm at 1.3 m | **0.32** (datasheet) |
| 12 MP (4000 px) | 1200 mm | 0.30 |
| 24 MP (6000 px) | 1200 mm | 0.20 |
| 45 MP (8192 px) | 1200 mm | 0.15 |
| 61 MP (9504 px) | 1200 mm | 0.13 |
| 16k line-scan | 1200 mm | 0.07 |

Every option passes the "resolution ≤ 5 % of T" criterion for body POMs (document 01 §8). Raw pixel density is not the bottleneck [I].

### 4.2 Perspective and the need for a height channel

For a camera at height H, a point raised by h appears displaced radially by **Δr = r·h/(H − h)**, where r is its distance from the principal point. A length whose *both* ends are raised by h is scaled by H/(H − h) [I, pinhole geometry].

| H | h | Radial shift at r = 600 mm | Scale error on 700 mm (both ends raised) |
|---|---|---|---|
| 1.0 m | 3 mm (hem) | 1.8 mm | 2.1 mm |
| 1.5 m | 3 mm | 1.2 mm | 1.4 mm |
| 2.0 m | 3 mm | 0.9 mm | 1.05 mm |
| 2.0 m | 12 mm (collar band) | 3.6 mm (2.1 mm at r = 350 mm) | 4.2 mm |
| 1.0 m | 12 mm | 7.3 mm (4.3 mm at r = 350 mm) | 8.5 mm |

The independent review quoted "8.5 mm at H = 2.0 m" for a 12 mm collar. That is the H = **1.0 m** value, and it treats a raised point as a raised span. The correct figures at 2.0 m are about 2–4 mm, still far too large [I].

**Consequence.** A flat-laid garment is 1–6 layers thick, with collars, waistbands and pocket flaps standing higher. Its top surface is **not** the calibration plane, and the offset varies across the garment. For thick features the height must be **measured**, or tightly **bounded by the state protocol**; otherwise millimetre-level systematic errors remain.

**How good must the height channel be?** The residual after correction is r·δh/H. At H = 2 m and r ≤ 0.6–0.8 m, a height error δh = 1 mm leaves 0.3–0.4 mm [I]. **About 1 mm of height accuracy is sufficient**, which is modest. Options:

- **The existing Zivid as a height channel.** Point precision 260 µm is more than adequate for this role [E, Zivid2+L110; I]. It lets the previous investment be reused while a high-resolution 2D camera does the planar metrology.
- **Stereo.** σ_z ≈ Z²·σ_d/(f·B). With Z = 2 m, a 6000-px camera over 1.6 m (f ≈ 7500 px), baseline 0.4–0.8 m and disparity noise 0.25 px, σ_z ≈ 0.2–0.35 mm — **if matching works** [I]. Passive stereo degrades on textureless surfaces [E-sec, Hirschmuller2008; Konolige2010], and plain dark or white fabrics are exactly that. **Projected texture (active stereo) or structured light is needed.** Passive stereo is not a universal fix.
- **Laser light-section or photometric stereo** from the raking lights already needed for seams (for relief and flatness rather than absolute height) [H].

### 4.3 Calibration scale

Zivid 2+ specifies a **dimension trueness error < 0.35 % (70th percentile) at the focus distance, < 0.40 % in the optimal range and < 0.60 % over the full temperature range**, measured with 5–50 cm reference distances after warm-up [E, Zivid2+L110]. Over 700 mm that is **≈ 2.5–4.2 mm** [I].

Any sensor used for dimensions must be **verified in the measurement plane with certified artefacts** (document 07, S1) rather than trusted from its datasheet. Rigid-part planar vision shows what in-situ calibration can reach [E, Nogueira2023].

### 4.4 Depth noise inflates surface-path lengths

Summing point-to-point 3D distances over noisy depth overestimates length. With lateral step a and independent depth noise σ_z, each step averages √(a² + Δz²) with Δz ~ N(0, 2σ_z²). To leading order that gives a **fractional inflation ≈ (σ_z/a)²** — a derivation given in both research bodies [I]. Monte Carlo:

| Step a | σ_z | Inflation | On 700 mm |
|---|---|---|---|
| 0.44 mm | 0.26 mm (Zivid L110 point precision) | **+26.6 %** | +186 mm |
| 1.0 mm | 0.26 mm | +6.2 % | +44 mm |
| 2.0 mm | 0.26 mm | +1.7 % | +12 mm |
| 0.44 mm | 0.05 mm | +1.3 % | +9 mm |

Smoothing removes the inflation but also the real wrinkles the surface length was meant to capture. **Use depth to correct planar coordinates for height and to detect non-flatness — not to integrate lengths along the fabric** [I].

### 4.5 Scanning sensors

- **Line-scan cameras** give orthographic geometry along the scan direction and very fine sampling. Across the line, ordinary lenses still have perspective [I].
- **Contact image sensors (CIS) have a depth of field of only about 0.2–0.3 mm** [E-sec, CIS-DOF]. They are unsuitable for garments with millimetres of relief unless the garment is pressed against glass (a flatbed arrangement), which changes the state. The independent review's CIS conveyor proposal does not account for this [I].

---

## 5. Learned localisation resolution (component E if misused)

Object-space cell size of common architectures applied to a whole garment, before sub-cell refinement [I from E, HRNet; RMBG-1.4; Paulauskaite2022]:

| Model / input | Span | Output cell | Quantisation SD (cell/√12) |
|---|---|---|---|
| Heatmap, 256-px side, stride 4 (64 cells) | 1200 mm | 18.8 mm | 5.4 mm |
| Heatmap, 384-px side, stride 4 (96 cells) | 1200 mm | 12.5 mm | 3.6 mm |
| Same, crop of 600 mm | 600 mm | 6.3 mm | 1.8 mm |
| Segmentation at 1024 × 1024 (RMBG class) | 1200 mm | 1.2 mm | 0.34 mm |
| UNet 128 × 128 (Paulauskaite 2022 best model) | 1200 mm | 9.4 mm | 2.7 mm |

Sub-cell decoding helps, but only down to the limit set by label quality [E, DARK]. A keypoint also discards the continuous contour information that lets geometric fitting average over hundreds of pixels [I].

---

## 6. Garment-state physics (component B)

### 6.1 Low-load compliance and anisotropy

- **Knits.** Stockinette has "almost vanishing rigidity" at low load, a response with "large variability over the cycles", and "large hysteresis" from inter-yarn friction [E, Poincloux2018].
- **Wovens.** Wovens are stiff along the yarns but compliant on the bias. Bias tension builds only through yarn–yarn friction [E, PanBias2015]. Fabric objective-measurement testing derives shear rigidity from the bias extension at 5 gf/cm (G = 123/EB5) [E-sec, FAST-shear].
- **Consequence [I].** Handling strain is **direction-dependent**: diagonal smoothing strokes distort body rectangularity. It is also **fabric-dependent**: knits ≫ wovens at low load. POMs that run diagonally or across the course direction of knits are the most exposed. The orientation of the lay relative to smoothing strokes is a factor to control and test (document 07, S3).

### 6.2 How much strain can survive smoothing? (corrected analysis)

Once the hands leave, table friction can hold an in-plane tension of at most **T(x) ≤ μ·w·g·x** at distance x from a free edge (μ friction coefficient, w areal mass). The elastic strain this can lock in is ε(x) = T(x)/K₀, where K₀ is the fabric's low-load tensile stiffness. Integrating over a half-span L from each free edge gives a locked extension of about **2 × μ·w·g·L²/(2K₀)** [I].

Estimating K₀ from the extension at 5 gf/cm (4.9 N/m), with w = 180 g/m², μ = 0.5 and L = 0.35 m [I]:

| Fabric behaviour (extension at 5 gf/cm) | K₀ (N/m) | Elastic lock-in by friction |
|---|---|---|
| Firm woven (0.5 %) | 980 | ≈ 0.1 mm |
| Woven (1 %) | 490 | ≈ 0.2 mm |
| Compliant woven / firm knit (3 %) | 163 | ≈ 0.7 mm |
| Knit (10 %) | 49 | ≈ 2.2 mm |
| Very compliant knit (20 %) | 25 | ≈ 4.4 mm |

What this shows:
- **Both earlier analyses were too simple.**
  - The first-pass analysis concluded that friction cannot hold meaningful elastic strain because it can only sustain ≲ 1 gf/cm. That is true for firm wovens and false for compliant knits, whose stiffness near zero load is tiny.
  - The independent review concluded that "any in-plane strain … is permanently locked into the garment by friction". That inverts the logic: a *low* friction limit means large strains relax. It also mis-converted 0.294 N/m to 0.030 gf/cm (the correct value is 0.30 gf/cm).
- **Hysteresis** (non-recovered strain) adds to elastic lock-in, and for knits it is large [E, Poincloux2018].
- **Knits are therefore the likely dominant source of re-lay variability**, possibly at the millimetre level on body dimensions. Wovens may sit at a few tenths of a millimetre. The magnitudes depend on low-load stiffness values not measured here [H → document 07, S3].

### 6.3 Wrinkles, buckles and why pressing them flat does not help

A camera measures **projected** length. For a sinusoidal wrinkle field (amplitude A, wavelength λ) along a POM, the projected length is short by a fraction of about (πA/λ)² [I]:

| A | λ | Length lost | On 500 mm |
|---|---|---|---|
| 0.5 mm | 50 mm | 0.10 % | 0.5 mm |
| 1 mm | 50 mm | 0.39 % | 2.0 mm |
| 1 mm | 30 mm | 1.08 % | 5.4 mm |
| 2 mm | 40 mm | 2.37 % | 11.8 mm |

A single ridge of height a over base b hides 2√((b/2)² + a²) − b: **0.9 mm** for 3 × 20 mm, **2.4 mm** for 5 × 20 mm [I].
- **Wrinkles always bias lengths short.** Good garments near the lower limit get rejected, and long garments get masked.
- A vendor reports "major wrinkles can shift measurements by 2–3 cm" [E-vendor, SizeAI-cheatsheet].

**A physical point both earlier analyses missed [I].** A wrinkle on a table is usually a **buckle**: excess fabric length trapped because friction stops it from sliding back. Normal pressure — from a vacuum, a platen or a hand pressing down — can flatten the buckle, but the excess length cannot escape, because pressure *increases* friction. **The buckle becomes a pleat, and the projected length is still short by the same amount.** Only **in-plane relaxation** removes trapped length: letting the fabric slide on a low-friction surface, or gently tensioning it. That is why document 04 proposes testing a **float → settle → hold** surface (air-flotation, as used in cutting rooms [E-vendor, AirFlotation], then gentle hold-down), rather than vacuum alone.

### 6.4 Seams, folds and layers

- **Seam roll:** r mm of roll on both sides changes a seam-to-seam width by 2r relative to fold-to-fold. A 3 mm roll changes it by 6 mm [I].
- **Tubes flatten along arbitrary fold lines.** The silhouette of a sleeve or a tubular body can shift by rotating the tube [I].
- **Fold radius and fuzz** make the optical edge lighting-dependent at the sub-millimetre to millimetre scale [H — measured in S2].
- **Multi-layer silhouettes are the union of layers.** The front hem of a shirt with a longer back is not on the silhouette [I].

### 6.5 Moisture and relaxation

ISO 3759 conditions to constant mass before dimensional testing [E]. For wool, hygral expansion limits appear in fabric-testing charts [E-sec]. **No primary source was found quantifying the dimensional change of finished cotton or blended garments under factory humidity swings.** The figure of "0.5–1.5 % per 30 % RH" in the independent review is unsupported, and it misattributes ISO 3759 and Munden as its sources. This is an **unmeasured effect**, to be tested by repeat measurements across a day with RH logged (document 07, S3) [H].

---

## 7. Sensing the state instead of assuming it

Controlling the state (document 04 §2) can be complemented by **measuring** it in every capture:

- **Flatness.** A height map (§4.2) or raking-light shading detects buckles, lifted edges and curled hems, and triggers re-lay or abstention [I].
- **Fabric strain from the fabric's own structure [H].** Knit course and wale densities can be measured automatically from images, including transmitted-light images, with reported agreement "above 98 %" with manual counts [E abstract, ZhangKnit2023]. Comparing *local* stitch density in the lay with the fabric's relaxed density (from a conditioned reference swatch, or statistics across the garment) could reveal over-stretched regions and flag a bad lay. This is untested at garment scale.
  - Stitch pitch is typically sub-millimetre to about a millimetre, so this needs ≲ 0.1–0.2 mm per pixel locally — within reach of a 45–61 MP camera or a small auxiliary close-up camera.
  - It would work for knits (where strain matters most); fine wovens are too dense.
- **Why not "strain from the CAD pattern"?** The independent review proposed computing a Green–Lagrange strain tensor between the CAD pattern and the laid garment, and invalidating lays above 4 % strain. The difference between pattern and garment **also contains the legitimate production deviation that QC measures**, plus sewing take-up and shrinkage. The rule would therefore flag good garments and could excuse bad ones. It also needs dense material correspondences that a sewn garment does not provide [I]. The fabric-texture approach measures the fabric's state directly.

---

## 8. Priors and constraints (component F)

Priors that pull estimates toward nominal geometry hide the deviations QC must find:
- CAD or template registration with strong regularisation;
- symmetry constraints, which hide asymmetric defects;
- "keypoint correction" heuristics that snap points to where they "should" be.

**Safeguard:** test on **seeded out-of-tolerance garments** whose true deviations are known from the reference. A slope below 1 when measured deviation is regressed on reference deviation is the signature of shrinkage toward nominal (document 07) [I].

---

## 9. Which errors can be fixed, and how?

| Error source | Better algorithms | Better sensors or optics | Physical control (surface, protocol, fixture, environment) | Only by redefining the measurand | Likely residual |
|---|---|---|---|---|---|
| Sampling and noise (C) | Sub-pixel fitting | Resolution, SNR | Lighting | — | Negligible if designed properly |
| Calibration and distortion (C) | Models | Stable optics | Artefacts, verification | — | Small, verifiable |
| Perspective / height (C) | Height correction | Height channel, greater H | Lower relief (flattening) | — | Small with correction |
| Surface-path noise (C) | Regularisation (a trade-off) | Better depth precision | Flatten instead | Define POMs as *flat projected* lengths | — |
| Edge definition: fold, fuzz (D) | Per-fabric edge models | Backlight, resolution | — | Specify the edge operator | Fraction of fabric thickness [H] |
| Seam visibility (D) | Seam detectors, cue fusion | Raking light, height | — | Seam vs fold per POM | Some combinations stay ambiguous → abstain [H] |
| Wrong feature chosen (E, gross) | Models, consistency checks, abstention | More cues | Standard orientation | — | Rare, detectable |
| Soft-landmark placement (A/E) | Geometric construction | — | — | **Complete operators** | Residual definitional spread |
| Handling strain (B) | Texture-based strain check [H] | — | **Low-friction relaxation, protocol, tension fixture** | Define the state | Fabric hysteresis spread; knits ≫ wovens [H] |
| Wrinkles and buckles (B) | Detect → re-lay | Height and raking cues | **In-plane relaxation** (not pressure alone) | Define allowable non-flatness | Small if detection plus re-lay works [H] |
| Seam roll, fold placement (A/B) | Detect seam vs fold | Seam cues | Lay rule (e.g. roll seams to the fold) | Declare seam vs edge | — |
| Moisture and relaxation (B) | — | — | Conditioning, timing rules | Define the state | Unknown [H] |
| Force-defined POMs | — | — | Actuated fixture with specified force | Specify the force | — |
| Priors (F) | Weak priors plus audit | — | — | — | Zero if audited |
| Reference (G) | — | Better reference | Marked points, same lay | — | Must be ≲ ¼ of the target, or modelled |

**The central point [I].** A more sophisticated network, a larger dataset or more GPU mainly reduces **gross semantic errors** and some **edge/seam extraction** cases. The components most likely to dominate — **definitional spread, handling strain on knits, buckles, seam roll** — yield only to specification and physical control. That is why this is a metrology and process-design problem first.

---

## 10. Budget allocation: what would have to be true

With a provisional target of u_c ≈ 1.5 mm for ±1/2 in POMs (Q_MP ≈ 24 %), sensing at 0.2 mm and edge extraction at 0.3 mm leave **≈ 1.46 mm** for definition, state and localisation combined [I]:
- 0.5 / 0.5 / 0.5 mm → u_c ≈ 0.94 mm ✔
- 1.0 / 0.8 / 0.5 mm → u_c ≈ 1.42 mm ✔ (just)
- 1.5 / 1.0 / 0.5 mm → u_c ≈ 1.91 mm ✘
- 0.5 / 1.5 / 0.3 mm → u_c ≈ 1.65 mm ✘ (e.g. knits with poor state control)

For ±1/4 in POMs (u_c ≲ 0.8–0.95 mm), definition and state must each be well below 0.5 mm. **Feasibility therefore reduces to two measurable quantities, u_A and u_B, per POM class, fabric class and state protocol** (document 07, S3–S4).

---

## 11. Where the bottlenecks probably are (ranked hypotheses)

1. **Garment state on knits** — low-load compliance, hysteresis, friction lock-in, anisotropy — and **buckles on all fabrics**. Physical process problems [H, indicated by §6].
2. **Definition of soft landmarks** — seam junctions under layers, constructed points. A specification problem [H, indicated by analogues].
3. **Semantic localisation on hard cases** — dark-on-dark seams, fleece, prints, occlusion. Algorithmic, largely contained by abstention [I].
4. **Sensing** — solved in adjacent fields. The traps are **height/perspective on thick features**, **calibration verification** and **lighting**, not resolution [E/I].

The order of 1 and 2 may differ between knits and wovens, which is what the first stages of document 07 will show.

---

## Sources cited in this document

See [references.md](references.md): HRNet, Wang2016-ISBI, Lips2024, Jonkers2025, ISO18890, Kouchi2011, Zivid2+L110, Hirschmuller2008, Konolige2010, Nogueira2023, CIS-DOF, RMBG-1.4, Paulauskaite2022, DARK, Poincloux2018, PanBias2015, FAST-shear, SizeAI-cheatsheet, AirFlotation, ISO3759, ZhangKnit2023.

*The formulas are shown next to each table: pinhole perspective; stereo σ_z ≈ Z²σ_d/(fB); E[√(a² + Δz²)] (Monte Carlo) and its small-noise form (σ_z/a)²; friction lock-in μwgL²/K₀; sinusoid arc length; ridge length; RSS combination. They were tabulated with throwaway scripts.*
