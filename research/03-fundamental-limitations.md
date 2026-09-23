# 03 — What actually limits garment measurement accuracy

*Part of the garment-measurement feasibility study. Evidence labels: **[E]** established (a cited source that was opened and checked), **[I]** engineering inference (derivation shown), **[H]** hypothesis. Source keys resolve in [references.md](references.md). The calculations here are first-principles estimates under stated assumptions; they are **not** measured performance.*

The central question of this document: when a measured garment dimension is wrong by a few millimetres, **which part of the chain is responsible, and can that part be fixed by software, by hardware, by physical control, or only by redefining what is measured?**

---

## 1. Error model

Every reported dimension is the output of a chain. Its error decomposes into components that can be separated experimentally (document 05):

| Code | Component | Physical origin | Varies with |
|---|---|---|---|
| **A** | **Definitional** | Incomplete POM definition. Several physically different distances satisfy the words ("edge to edge", "base of armhole", "SNP under the collar") | Garment style, POM type, the person interpreting |
| **B** | **Garment state** | The same garment laid twice has a different geometry: in-plane strain left by smoothing, wrinkles and folds, seam roll, sleeve angle, fold-line placement, relaxation since the last handling, moisture | Operator, lay protocol, fabric, time |
| **C** | **Sensing** | Pixel sampling, sensor noise, lens distortion residuals, calibration scale error, perspective (height above the calibration plane), depth noise and trueness | Camera, optics, geometry, calibration, temperature |
| **D** | **Feature extraction** | Where an edge, seam line or intersection is located in the image: fibre fuzz, rounded folds, low contrast, prints, shadows, background spill | Fabric, colour, lighting, algorithm |
| **E** | **Semantic localisation** | Which of the extracted features is "the" POM endpoint; the choice of the wrong seam, a crease mistaken for a seam, occlusion. Includes rare gross errors | Model, training data, garment novelty |
| **F** | **Prior or constraint bias** | Shrinkage toward the expected (CAD or template) geometry, symmetry assumptions, post-hoc "keypoint correction" | Algorithm design |
| **G** | **Reference error** | The uncertainty of whatever "truth" the system was compared with (manual tape, expert marks) | Reference procedure |

To first order (GUM-style, independent components):

u_c² ≈ u_A² + u_B² + u_C² + u_D² + u_E² + u_F²

Component G does not affect the system's real performance. It does affect how well performance can be *estimated*, and it inflates apparent error if ignored [E for the framework, JCGM106; I for the decomposition].

**Two features of this model matter more than any single number [I]:**
1. **A and B are properties of the garment and its definition, not of the measuring system.** No camera or neural network reduces them. They also limit the manual reference method.
2. **E is heavy-tailed.** Most localisations are close, a few are completely wrong (the wrong seam, a crease). A Gaussian budget describes the core. The tail must be handled by detection and abstention, and reported as a gross-error rate (document 01 §7).

---

## 2. Sensing (component C): not the bottleneck if designed properly

### 2.1 Sampling resolution

Object-space pixel size over a garment-sized field [I; arithmetic from sensor formats]:

| Sensor (example format) | Field across long axis | mm per pixel |
|---|---|---|
| Zivid 2+ L110 colour image, 2448 px, vendor FOV at 1.1 m [E, Zivid2+L110] | 1090 mm | **0.44** (vendor: "0.44 at 1100") |
| Zivid 2+ M130, 2448 px [E, Zivid2+M130] | 790 mm | **0.32** |
| 12 MP (4000 px) | 1200 mm | 0.30 |
| 24 MP (6000 px) | 1200 mm | 0.20 |
| 45 MP (8192 px) | 1200 mm | 0.15 |
| 61 MP (9504 px) | 1200 mm | 0.13 |
| 16k-pixel line-scan | 1200 mm | 0.07 |

Against the resolution criterion of ≤ 5 % of the tolerance (≥ 0.64 mm for ±1/4 in, document 01 §8), **every option passes**. Sub-pixel methods localise high-contrast edges to a small fraction of a pixel on rigid parts [E, Nogueira2023; Steger]. Raw resolution is not the limiting factor.

### 2.2 Perspective: height above the calibration plane

For a camera at height H looking down at a table plane calibrated at height 0, a point raised by h appears displaced radially by Δr = r·h/(H−h). Here r is its distance from the principal point, and lengths at height h scale by H/(H−h) [I; pinhole geometry]:

| Camera height H | Raised by h | Length scale error | Error on a 700 mm length | Edge shift at r = 600 mm |
|---|---|---|---|---|
| 1.0 m | 3 mm | 0.30 % | 2.1 mm | 1.8 mm |
| 1.5 m | 3 mm | 0.20 % | 1.4 mm | 1.2 mm |
| 1.5 m | 5 mm | 0.33 % | 2.3 mm | 2.0 mm |
| 2.0 m | 3 mm | 0.15 % | 1.05 mm | 0.9 mm |
| 3.0 m | 3 mm | 0.10 % | 0.70 mm | 0.6 mm |

A flat-laid garment is 2–6 fabric layers thick at seams, hems and waistbands. Puffed areas, ribs, collars and pockets stand higher. **The garment's top surface is therefore not the calibration plane, and the offset varies across the garment.**

At typical camera heights this produces **millimetre-level, position-dependent errors** — the same order as the uncertainty target (document 01 §8). The error is systematic, so it does not average out over repeats. Remedies [I]:
- a greater camera height (scale error ∝ 1/H);
- calibrating at the garment surface height;
- measuring the height map (3D, or a light-section line) and correcting for it;
- orthographic acquisition (line-scan or CIS scanning, telecentric optics for small fields);
- placing critical edges near the principal point.

This is one place where depth sensing earns its keep: not to measure along the surface, but to *correct* planar measurements for height.

### 2.3 Calibration scale and trueness

A factory-calibrated 3D camera is not automatically a metrology instrument at garment scale. Zivid 2+ L110 and M130 specify a **dimension trueness error of < 0.35 % (70th percentile) at the focus distance, < 0.40 % within the optimal working distance, and < 0.60 % over the full temperature range**. This is measured on calibrated reference distances of 5–50 cm, under specified conditions (10 min warm-up, 0 lux ambient light, in-field correction applied) [E, Zivid2+L110; Zivid2+M130].

On a 700 mm dimension, 0.35–0.60 % is **≈ 2.5–4.2 mm at the 70th percentile** [I, arithmetic]. That is far more than the ≈ 1–2 mm uncertainty target. The spec covers the whole field and working volume. A system that re-verifies scale in the field against a certified artefact in the measurement plane can do much better, as rigid-part planar vision shows [E, Nogueira2023]. **The key point is that scale trueness must be established by the system's own calibration and verification (document 05, S1), not inherited from a datasheet** [I].

### 2.4 Depth noise when measuring *along* the surface

A tempting use of 3D data is to measure lengths along the fabric surface (geodesic length) to "undo" wrinkles. Summing point-to-point 3D distances over noisy depth **inflates length systematically**. With lateral step a and independent depth noise σ_z per point, each step length averages ≈ √(a² + 2σ_z²) > a. Monte Carlo [I]:

| Lateral step | Depth noise σ_z | Length inflation | On 700 mm |
|---|---|---|---|
| 0.44 mm | 0.26 mm (≈ Zivid L110 point precision [E]) | **+26.6 %** | +186 mm |
| 1.0 mm | 0.26 mm | +6.2 % | +44 mm |
| 2.0 mm | 0.26 mm | +1.7 % | +12 mm |
| 0.44 mm | 0.05 mm | +1.3 % | +9 mm |

Heavy smoothing removes the inflation, but it also removes the real wrinkles the geodesic length was supposed to capture. **Surface-path measurement with a structured-light camera is therefore a regularisation problem whose answer depends on filter choices.** That is itself a "filtration" operation that must be specified (document 01 §3.3). It is not a free lunch [I].

### 2.5 Sensing summary

The sensing component can be pushed to a few tenths of a millimetre with careful but conventional engineering: resolution, calibration in the measurement plane, height control or correction, and verification with artefacts. The textbook example is the backlit metrology table: ±35 µm over most of a 1.8 × 1.2 m field on flat stiff parts [E, InspecVision]. **Sensing is the most controllable component, and the only one that can be fully verified without garments** [I].

---

## 3. Garment state (component B): the physics of "flat, relaxed, without tension"

### 3.1 In-plane strain from handling

Fabrics extend measurably under very small loads. Fabric objective-measurement systems test low-stress extensibility at **5, 20 and 100 gf/cm** (SiroFAST E5, E20, E100) or up to 500 gf/cm (KES) [E secondary, FAST/KES]. For tailoring-grade woven wool, SiroFAST control charts treat E100 of roughly 2–6 % as the normal range [E secondary]. Knits are generally far more extensible at low load (magnitudes not quantified here) [H].

**How much tension can a laid garment "hold"?** Table friction can sustain an in-plane tension of at most T ≈ μ·w·g·L per unit width (μ friction coefficient, w areal mass, L contact length) [I]:

| Areal mass | μ | L | Max tension held by friction |
|---|---|---|---|
| 150 g/m² | 0.3 | 0.2 m | 0.09 gf/cm |
| 250 g/m² | 0.6 | 0.4 m | 0.60 gf/cm |
| 400 g/m² | 0.6 | 0.4 m | 0.96 gf/cm |

These are one to two orders of magnitude below the *lowest* standard test load (5 gf/cm). The consequence is subtle but important [I]:
- Once the operator's hands leave the fabric, **elastic** strain from smoothing cannot be held by the table and largely recovers.
- The strain that *remains* is the **hysteretic (non-recovered)** part of the fabric's tensile behaviour, which fabric testing quantifies as tensile resilience. It is fabric-specific and history-dependent.
- In the other direction, fabric pushed together stays pushed together: in-plane compression buckles into wrinkles, and gravity plus friction hold them.

So "without tension" is not a unique state. It is the endpoint of a handling history. Its reproducibility depends on the fabric's hysteresis and on how consistently the operator smooths. **This is the physical basis for treating component B as a first-class uncertainty that must be measured per fabric class (document 05, S3) [H for magnitudes].**

### 3.2 Wrinkles and folds: projected length vs fabric length

A camera from above measures *projected* length. A wrinkle hides fabric. For a sinusoidal wrinkle field of amplitude A and wavelength λ spanning the measured line, the fraction of length lost is f ≈ (πA/λ)² for small slopes (exact values by numerical integration) [I]:

| Amplitude A | Wavelength λ | Length lost | On a 500 mm POM |
|---|---|---|---|
| 0.5 mm | 50 mm | 0.10 % | 0.5 mm |
| 1 mm | 50 mm | 0.39 % | 2.0 mm |
| 1 mm | 30 mm | 1.08 % | 5.4 mm |
| 2 mm | 40 mm | 2.37 % | 11.8 mm |

A single isolated ridge of height a over a base width b hides 2·√((b/2)² + a²) − b [I]:
- 3 mm high × 20 mm wide: **0.9 mm**
- 5 mm × 20 mm: **2.4 mm**
- 10 mm × 40 mm: **4.7 mm**

Implications [I]:
- **Barely visible, low wrinkles spread along a POM can shift it by millimetres.** A few distinct ridges cost about a millimetre each.
- This matches the vendor warning that "major wrinkles can shift measurements by 2–3 cm" [E vendor, SizeAI-cheatsheet].
- **Wrinkles bias measurements short, never long.** That is a systematic error that would cause false *rejects* for "too short" and mask "too long" defects. It does not average out.
- Detecting non-flatness (depth, raking light, photometric cues) and rejecting or re-laying is cheaper and more reliable than trying to "unwrinkle" computationally (§2.4).

### 3.3 Seams, folds and layers

- **Seam roll.** On a flat-laid, side-seamed body, if both side seams roll r mm toward the camera, the visible seam-to-seam width is 2r shorter than fold-to-fold. With r = 3 mm that is **6 mm** — the full ±1/4 in tolerance band's half-width [I]. ISO 3759 measures "between seams" [E, ISO3759 §7.1.6], ISO 18890 "edge to edge" [E, ISO18890]. The two definitions diverge exactly when seams roll.
- **The silhouette of a tube is a fold.** For tubular knits, and for sleeves and legs of all garments, the outline is where the fabric bends back on itself. Its lateral position depends on how the tube was flattened (it can be rotated around the tube) [I].
- **Thickness at the edge.** A fold has a finite bend radius, roughly the fabric thickness or more. Fibre hairiness extends the apparent edge outward. The optical "edge" therefore depends on lighting: backlight silhouette, front-light contrast or shading on the rounded fold. The differences are at the sub-millimetre to millimetre scale, and fabric-dependent (fleece and pile more than poplin) [I; magnitude H].
- **Multiple layers.** From above, a flat lay shows the union of front and back layers. On shaped hems (e.g. a shirt with a longer back), "hem edge" in the silhouette may belong to the back layer while the POM refers to the front [I].

### 3.4 Time and environment

- ISO 3759 requires conditioning to constant mass (≤ 0.25 % mass change per hour), taking at least 4 h, in the ISO 139 atmosphere before dimensional testing [E, ISO3759].
- Moisture-driven dimensional change (hygral expansion) is a recognised fabric property, prominent in wool (SiroFAST HE limits exist) [E secondary].
- Garments measured straight off the pressing table, or after hours in a humid store, may differ from their conditioned state. The magnitude for typical cotton and poly-cotton garments within factory humidity ranges was not found and should be measured (repeat measurements of the same garments over a day) [H].

**Summary of component B [I/H].** Garment state is governed by fabric physics that no sensor can change: low-load extensibility, hysteresis, friction, bending, moisture. It can only be controlled by *procedure* (lay protocol, flattening method, conditioning) or *fixtures* (vacuum, platen, tension). Its size relative to tolerance is the most important unknown in this study.

---

## 4. Definitional uncertainty (component A): is the reference point physically observable?

A measurement is only as good as the physical reality of its endpoints. Classifying POM endpoints by what physically defines them and which sensing modality reveals them [I, building on ISO 18890 definitions [E]]:

| Endpoint type | Examples | Physical cue | Revealed by | Typical failure conditions |
|---|---|---|---|---|
| **Cut or finished edge** | Hem edge, cuff edge, leg opening | Material boundary | Backlight silhouette (best), front-light contrast | Fuzzy or pile edges; rolled or flared rib; shaped hems (which layer?) |
| **Fold edge** | Chest "edge to edge", sleeve width, thigh | Bend of a folded tube | Backlight, front light | Position depends on lay (tube rotation); rounded fold; seam roll near the fold |
| **Visible seam line** | Shoulder seam, armhole seam, yoke, waistband seam | Stitch row plus seam ridge or groove | Raking light (topography), texture, depth | Same-colour thread on dark fabric, fleece/pile hiding stitches, fine-gauge knits, prints |
| **Seam junction** | SNP (shoulder ∩ neck seam), underarm point (armhole ∩ side seam), crotch point (inseam ∩ crotch seam) | Intersection of two seams | Fitted seam curves intersected geometrically | **Often at a fold or under a layer:** the SNP is defined excluding trim, collar or hood [E, ISO18890], so it may be *under* the collar; the underarm point sits where the flat lay folds; the crotch point is inside the fold between the legs |
| **Constructed point** | "2.5 cm below base of armhole", waist at a "required position", knee "halfway" | Arithmetic from another point plus a direction | Computation | Inherits all upstream uncertainty. The **direction** ("below") is usually unspecified |
| **Centre line** | CF, CB | Placket, buttons, symmetry | Features or fitted symmetry axis | Unmarked backs; asymmetric garments (a real defect must not be "symmetrised away") |
| **Force-defined** | Waist extended, neck opening extended | Dimension at maximum extension "without seam or stitch breakage" [E, ISO18890] | **Not observable passively.** Needs actuation with defined force | Maximum-extension definitions are force-ambiguous in themselves |

**Interpretation [I/H]:**
- Endpoints of the first three types are physically well defined *once the definition says which one to use* (seam vs fold, which layer). Their residual uncertainty is sub-millimetre to about a millimetre and fabric-dependent.
- Seam junctions and constructed points carry most of the definitional risk. The anthropometry analogue (document 02 §2) predicts that trained experts will disagree about them by roughly 1–2 mm even on a fixed lay, and that dimension-level agreement will *hide* this [E, Kouchi2011 for the analogue; H for garments].
- Force-defined POMs are outside the reach of any passive vision system. They need a fixture, and even then the force must be specified.

The remedy is to rewrite each POM as a complete operator (document 01 §3.3), for example:
- "SNP := intersection of the fitted shoulder-seam stitch line with the fitted neck-seam stitch line, extended under the collar if covered";
- "2.5 cm below := along the perpendicular to the fitted CF axis".

This **moves ambiguity out of the measurement and into the specification**, where it can be agreed once.

---

## 5. Semantic localisation (component E) and the resolution of learned models

Learned models identify *which* feature is meant. Whether they can also place it *precisely* depends on the resolution at which they operate. Object-space grid cell for common architectures applied to a whole garment, before any sub-cell refinement [I; architectures [E, HRNet; RMBG-1.4; Paulauskaite2022]]:

| Model and input | Object span mapped | Output cell | Uniform quantisation SD (cell/√12) |
|---|---|---|---|
| HRNet-style heatmap, 256-px side, stride 4 (64 cells) | 1200 mm | **18.8 mm** | 5.4 mm |
| HRNet-style heatmap, 384-px side, stride 4 (96 cells) | 1200 mm | 12.5 mm | 3.6 mm |
| Same, cropped to a 600 mm region | 600 mm | 6.3 mm | 1.8 mm |
| Segmentation at 1024 × 1024 (RMBG class) | 1200 mm | 1.2 mm | 0.34 mm |
| UNet 128 × 128 mask (best model in Paulauskaite 2022) | 1200 mm | 9.4 mm | 2.7 mm |

Sub-cell decoding (DARK-type, or HRNet's quarter-offset rule) reduces quantisation substantially, but only toward the limit set by the *training labels*. Those labels carry the definitional spread of whoever drew them [E, DARK; I]. In the closest published analogue, keypoints on almost-flat cloth, errors were at the centimetre level [E, Lips2024].

**Architectural conclusion [I]:** use learning for what it is good at — robust semantic identification, occlusion reasoning, and garment-type and style variation. Hand the metric localisation to a geometric stage that:
1. works at full sensor resolution in a window around the semantic proposal;
2. fits physically defined primitives (edge segments, seam curves) with sub-pixel methods;
3. constructs the POM endpoint by the complete operator (intersections, perpendiculars, offsets);
4. reports a per-POM quality metric (fit residuals, contrast, disagreement between cues) that drives **abstention**.

In this design the learned component's error enters only through *gross* misidentification (choosing the wrong seam), which is detectable. Its fine error does not enter.

---

## 6. Priors and constraints (component F): the QC-specific danger

In most computer vision, priors are pure benefit. In conformity assessment they can be harmful:
- **Template or CAD registration** with strong regularisation pulls the estimated geometry toward the nominal. Out-of-tolerance garments then look closer to tolerance than they are, which increases false accepts precisely where it matters [I].
- **Symmetry constraints** — enforcing equal left and right sleeves, or a symmetric chest — hide real asymmetry defects that QC should catch [I].
- **Post-hoc keypoint "correction" rules** (e.g. snapping a point toward where it "should" be relative to others) are priors in disguise [I].

**Required safeguard.** Every prior or constraint must be tested on **seeded out-of-tolerance garments**: deliberately altered garments whose true deviations are known from the reference. The test is whether the reported deviation is attenuated. A slope below 1 in a regression of measured on reference deviation is the signature of shrinkage toward nominal (document 05) [I].

---

## 7. Which errors are algorithmic, sensing, physical or definitional?

| Error source | Reduced by better **algorithms** | Reduced by better **sensors or optics** | Reduced by **physical control** (lay protocol, fixtures, environment) | Reduced only by **redefining the measurand** | Irreducible remainder |
|---|---|---|---|---|---|
| Pixel sampling and noise (C) | Sub-pixel fitting, averaging | Resolution, SNR | Lighting | — | Negligible if designed properly |
| Calibration scale and distortion (C) | Calibration models | Stable optics | Artefacts, verification schedule | — | Small, verifiable |
| Perspective and height (C) | Height correction (if height known) | Height map, orthographic scan, greater height | Flattening (less height variation) | — | Small with correction |
| Depth noise for surface lengths (C) | Regularisation (which trades accuracy) | Better depth precision | Flattening (makes surface length unnecessary) | Define the POM as a *flat* projected length | — |
| Edge definition: fuzz, fold radius (D) | Edge models per fabric class | Backlight, resolution | — | Specify the edge operator (e.g. outermost backlight silhouette) | ≈ fraction of fabric thickness; fabric-dependent [H] |
| Seam visibility (D) | Seam detectors, multi-cue fusion | Raking light, multi-angle lighting, depth | — | Define seam-based vs fold-based per POM | Some fabric/thread combinations will stay ambiguous → abstain [H] |
| Wrong feature chosen (E, gross) | Better models, consistency checks, abstention | More cues (depth, backlight) | Standard lay orientation | — | Rare, detectable if checks exist |
| Fine localisation of soft landmarks (E) | Geometric construction instead of learned points | Resolution (secondary) | — | **Yes**: complete operators | Residual definitional spread (A) |
| Strain from handling (B) | — (not observable in one image, except by marks) | — | **Lay protocol, flattening method, fixtures** | Define the state (e.g. "after vacuum-flattening at X kPa for Y s") | Fabric hysteresis spread [H] |
| Wrinkles and folds (B) | Detect and reject | Depth, raking light to *detect* | Flattening devices, re-lay | Define allowable non-flatness | Small if detection plus re-lay works [H] |
| Seam roll and fold placement (B/A) | Detect seam position relative to fold | Seam visibility cues | Lay protocol (roll seams to the fold) | Declare seam-to-seam vs edge-to-edge per POM | — |
| Moisture and relaxation (B) | — | — | Conditioning time, measurement timing | Define the state (as ISO 3759/139 do) | Fabric-dependent [H] |
| Force-defined POMs (A/B) | — | — | Actuated fixture with specified force | Specify the force | — |
| Priors and constraints (F) | Weak priors, audit on seeded defects | — | — | — | Zero if audited |
| Reference error (G) | — | Better reference instrument | Marked points, same lay | — | Must be ≤ ~¼ of the target uncertainty, or be modelled (document 05) |

**Reading the table [I].** Of the error sources, the only ones that "a more sophisticated neural network, a larger dataset, more GPU power or a better keypoint detector" can reduce are **gross semantic errors** and some **edge/seam detection** cases. The dominant candidates — **definitional spread, handling strain, wrinkles, seam roll** — are reduced only by specification and physical control. That is the core argument for treating this as a metrology and process-design problem first.

---

## 8. Budget allocation: what would have to be true

Rather than predicting performance (which requires the experiments in document 05), the budget can be inverted. Given a target, what must each component achieve? Take the provisional target of u_c ≈ 1.5 mm for a ±1/2 in POM (Q_MP ≈ 24 %; document 01 §8). Combining components in quadrature [I; arithmetic]:
- With sensing at 0.2 mm and edge definition at 0.3 mm, **the remaining budget for garment state, definition and localisation together is ≈ 1.46 mm.**
- Example allocations:
  - definition 0.5 + state 0.5 + localisation 0.5 mm (plus 0.2 sensing and 0.3 edge) → u_c ≈ 0.94 mm ✔
  - definition 1.0 + state 0.8 + localisation 0.5 mm → u_c ≈ 1.42 mm ✔ (just)
  - definition 1.5 + state 1.0 + localisation 0.5 mm → u_c ≈ 1.91 mm ✘
  - definition 0.5 + state 1.5 + localisation 0.3 mm → u_c ≈ 1.65 mm ✘

For ±1/4 in POMs (target u_c ≈ 0.8–0.95 mm), everything must be roughly halved. Definitional and state components would each need to be well below 0.5 mm, which is plausible only for POMs anchored on sharp, well-defined features under a controlled lay [H].

**The feasibility question therefore reduces to two measurable quantities:** the definitional spread (A) and the state reproducibility (B), per POM class and fabric class. If they are around 1 mm or less for the main body POMs, the rest of the chain can be engineered. If they are several millimetres, no sensing architecture will meet ±1/4 in tolerances, and the only levers are specification changes and physical fixtures.

---

## 9. Diagnosis of the previous system (inference, not fact)

The previous team's implementation details are unknown. The analysis below applies the error model to what *is* known and states each point as a hypothesis with a test.

| # | Hypothesis | Basis | How to test (document 05) |
|---|---|---|---|
| H1 | **Coarse learned localisation** was a major contributor to the "slightly displaced" reference points | Standard keypoint architectures map a garment onto 64–96 heatmap cells (≈ 12–19 mm per cell over 1.2 m) [I from E, HRNet]. Sub-cell refinement is limited by label quality [E, DARK]. Cloth keypoint detectors report cm-level errors [E, Lips2024]. Post-hoc "correction algorithms" are consistent with the absence of a full-resolution geometric stage | S5 vs S6: compare model points with expert clicks on the same full-resolution images |
| H2 | **Synthetic text-to-image labels encoded an interpretation, not a physical definition** | Generated images lack faithful projective geometry [E, Sarkar2024]. No physical garment exists to anchor the labels [I] | Evaluate the model against *physically marked* POMs on real garments. Look for systematic offsets per POM (bias), not just scatter |
| H3 | **Domain gap** (white generated backgrounds → real green sheet; generated textures → real fabrics) inflated error and gross failures | The sim-to-real gap persists even with physically based synthetic data, with heavier error tails [E, Lips2024] | Stratify errors by background, colour and fabric; measure the gross-error rate |
| H4 | **Segmentation edge error** from a 1024-px background-removal model plus green spill on edges | RMBG input is 1024 × 1024 [E]; ≈ 1.2 mm cell over 1.2 m [I]; chroma spill is a known physical effect [I] | Compare RMBG edges with backlit or thresholded silhouettes of the same lay |
| H5 | **The 3D camera contributed less than its cost suggests.** Its 2D resolution (≈ 0.44 mm/px) is lower than cheap high-resolution 2D cameras; its dimension trueness spec (< 0.35–0.60 %, 70th percentile) is ≈ 2.5–4 mm over 700 mm unless re-verified; naive surface-length use of its depth would inflate lengths (§2.4) | [E, Zivid2+L110] + arithmetic [I] | S1: artefact tests of the actual camera versus a high-resolution 2D camera in the same fixture |
| H6 | **Garment state was not controlled or measured**, so re-lay variability was mixed into "accuracy" | No lay protocol reported; physics of §3 | S3: re-lay study with physical markers |
| H7 | **The reference (manual tape) was not characterised**, so part of the "error" was reference error | No public data on inspector reproducibility; informal evidence of cm-level human error [E, Paulauskaite2022; Kim2022] | S7: manual-tape MSA on the same garments |
| H8 | **"80 %" mixed several of these effects into one undefined number** | Document 01 §10 | Report the metrics of document 01 §7 |

**What this does *not* imply [I].** None of this shows that the previous team's approach could not work with fixes, or that the problem is unsolvable. It suggests that their difficulty is the predictable result of three things:
- using a semantic-recognition toolchain as a measuring instrument;
- training on data with no geometric truth;
- not separating state, definition and reference errors.

---

## 10. Where the true bottlenecks probably lie (ranked hypotheses)

1. **Definitional ambiguity of soft landmarks** (seam junctions, constructed points, SNP under collars): a specification problem [H, strongly suggested by analogues].
2. **Garment state reproducibility** (handling strain, wrinkles, seam roll, fold placement), which is fabric-dependent: a physical process problem [H].
3. **Semantic localisation on hard cases** (low-contrast seams, occlusions, novel styles): an algorithmic problem, mostly addressable with abstention [H].
4. **Sensing**: an engineering problem that is solved in adjacent fields. Its main traps are perspective/height, calibration verification and edge lighting, not resolution [E/I].

The order of 1 and 2 may well be reversed for knits versus wovens. That is exactly what the first stages of the feasibility study are designed to find out.

---

## Sources cited in this document

Full annotated entries in [references.md](references.md).

- `[Zivid2+L110]`, `[Zivid2+M130]` Zivid 2+ technical specifications (rev. 7/2025). https://www.zivid.com/hubfs/User%20guides%20and%20datasheets/Zivid%202+%20L110%20Datasheet.pdf · https://www.zivid.com/hubfs/User%20guides%20and%20datasheets/Zivid%202+%20M130%20Datasheet.pdf
- `[Nogueira2023]` *Sensors* 2023. https://pmc.ncbi.nlm.nih.gov/articles/PMC10346874/
- `[Steger]` C. Steger, IAPRS 2000; *IEEE TPAMI* 1998.
- `[InspecVision]` https://www.inspecvision.com/planar2d-models/planar-p220.35
- `[FAST/KES]` SiroFAST and Kawabata low-stress test loads and control limits (secondary: ScienceDirect topic overview and journal articles citing SiroFAST charts). https://www.sciencedirect.com/topics/engineering/kawabata-evaluation-system
- `[SizeAI-cheatsheet]` https://sizeai.co/blog/flat-lay-cheat-sheet
- `[ISO3759]` https://cdn.standards.iteh.ai/samples/57309/1af372753bf84b3cb0288a274429c73d/ISO-3759-2011.pdf
- `[ISO18890]` https://cdn.standards.iteh.ai/samples/63693/131110d2f792493aabb7200cbd9136ba/ISO-18890-2018.pdf
- `[Kouchi2011]` https://pubmed.ncbi.nlm.nih.gov/20947062/
- `[HRNet]` https://arxiv.org/abs/1902.09212 · `[DARK]` https://arxiv.org/abs/1910.06278
- `[RMBG-1.4]` https://huggingface.co/briaai/RMBG-1.4
- `[Paulauskaite2022]` https://doi.org/10.3390/app12094470 · `[Kim2022]` https://doi.org/10.3390/app12105286
- `[Lips2024]` https://arxiv.org/abs/2401.01734
- `[Sarkar2024]` https://arxiv.org/abs/2311.17138
- `[JCGM106]` https://www.bipm.org/documents/20126/2071204/JCGM_106_2012_E.pdf

*Calculation scripts (standard-library Python, throwaway) were used only to tabulate the formulas shown: pinhole perspective Δr = r·h/(H−h); sinusoid arc length by numerical integration; ridge length 2√((b/2)²+a²)−b; Monte Carlo of E[√(a²+Δz²)]; friction tension μ·w·g·L; RSS combination. Every number can be reproduced from these formulas.*
