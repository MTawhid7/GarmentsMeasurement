# 02 — Metrology principles and the state of the art

*Part of the garment-measurement feasibility study. Evidence labels: **[E]** established (a cited source that was opened and checked), **[I]** engineering inference, **[H]** hypothesis. Source keys resolve in [references.md](references.md).*

This document covers four questions:
1. How do fields that measure to micrometres or nanometres actually achieve it, and which of their principles transfer to a textile garment? (§1)
2. What have other domains that measure *soft or ambiguous* objects learned? (§2)
3. What exists today for garment measurement specifically, and how good is the evidence? (§3)
4. Which computer-vision and CAD building blocks are relevant, and where do they break down for metrology? (§4–§5)

---

## 1. How precision metrology achieves accuracy — and what transfers to garments

The ability to measure a silicon wafer to nanometres or an aircraft part to micrometres comes mainly from *how the problem is posed*, not from better cameras. Nine principles recur. Each is listed below with its evidence and a judgement on whether it transfers to garments, and why.

| # | Principle | How precision fields use it | Transfer to garments | Why |
|---|---|---|---|---|
| **P1** | **Define the measurand completely** | GPS specification operators: partition → extraction → filtration → association → construction → evaluation. A "complete specification operator is unambiguous" [E, ISO17450-2] | **Fully** | This is about writing the specification, not about the material. It is also the cheapest lever available (document 01 §3.3) |
| **P2** | **Specify and control the part's state** | Non-rigid parts: dimensions apply in the free state (gravity only, with orientation and support stated) or in a specified restrained condition (forces, fixtures) [E, ISO10579]. Rigid-part metrology fixes the temperature at 20 °C | **Partly** | Garments can be supported, flattened, clamped or tensioned. But every restraint changes the garment's geometry, so the restraint *becomes part of the measurand* and must be agreed with the buyer [I] |
| **P3** | **Traceable calibration with artefacts** | Calibrated scale bars and test fields for optical 3D systems (VDI/VDE 2634 Part 1); ISO 10360-based verification for coordinate systems, which InspecVision uses for its 2D gauges [E, InspecVision] | **Fully, for the sensing chain** | Artefacts are rigid, so the camera, lens and algorithm chain can be verified independently of any garment [I] |
| **P4** | **Uncertainty budgets and iteration** | GUM; the ISO 14253-2 budget-driven procedure; ISO 22514-7 capability indices [E, JCGM106, ISO22514-7] | **Fully** | The budget must include definitional and garment-state components that rigid-part budgets normally omit (ISO 22514-7 explicitly assumes they are small) [E, ISO22514-7] |
| **P5** | **Average many points by fitting features** | Diameters, planes and datums are fitted to many surface points (least squares, minimum zone) instead of trusting single points | **Partly** | Fitting a line to hundreds of edge pixels suppresses pixel noise roughly as 1/√N. But fabric edges are not ideal geometry (waviness, fuzz, rolled seams), so what the fit "filters" must itself be specified (P1) [I] |
| **P6** | **Reference measurement systems** | Semiconductor CD metrology validates the fast production tool against a slower reference system (e.g. CD-AFM). Mandel regression accounts for the reference's own error; "total measurement uncertainty" (TMU) is what remains after the reference uncertainty is removed [E secondary, TMU] | **Fully, as a validation method** | This is exactly the structure the feasibility study needs: a slow, careful reference (marked points, controlled lay, calibrated instrument) and a fast system under test (document 05) [I] |
| **P7** | **Optics chosen for the measurement, not for pictures** | Backlit silhouette gauging: InspecVision Planar measures flat parts (including flexible gaskets) on an LED backlit table with a 50 MP camera. The P220.35 has a 1830 × 1220 mm area, ±35 µm over most of the table and ±70 µm at the edges, with <0.25 s measurement [E vendor, InspecVision]. Telecentric lenses remove perspective error, but their field is limited by the front optic (≈ 417 mm front lens for the largest standard lens, and "up to 600 mm" with Fresnel designs) [E vendor, secondary, Telecentric]. Targeted photogrammetry reaches relative precisions of roughly 1:50,000–1:100,000 [E secondary, Luhmann2010] | **Mostly** | Garment-sized fields (≈ 1.2 × 1.6 m) are within reach of backlit single-camera gauges, but not of telecentric optics. The achievable precision is set by the *textile edge*, not the optics (§4.2, document 03) [I] |
| **P8** | **Design for metrology: add features that make the measurand observable** | Semiconductor overlay targets; retroreflective photogrammetric targets; textile dimensional-change testing marks reference points (indelible ink with a template, sewn contrasting threads, holes, staples) because fabric edges are too poorly defined [E, ISO3759 §4.2] | **Conditionally** | Very powerful for *validation* (marked garments) and possibly for production (marks applied at cutting). Production use needs buyer acceptance (document 04, A9) [I/H] |
| **P9** | **Reversal and redundancy to expose instrument bias** | Overlay metrology estimates tool-induced shift by measuring at 0° and 180° wafer rotation: TIS = ½[OVL(0°) + OVL(180°)] [E, TIS] | **Fully** | Measure the same garment lay rotated 180° on the table, or mirrored left/right, to expose lighting-direction, perspective or model biases. It is cheap and diagnostic [I] |

### Why nanometre and micrometre performance does *not* transfer

The principles transfer; the numbers do not. Precision fields measure objects whose geometry is:
- **stable** — stiff enough that the fixturing force and gravity change it by less than the tolerance;
- **well-defined** — sharp, smooth surfaces and edges, so a "point on the surface" is a meaningful notion at µm scale;
- **thermally and mechanically predictable**, so corrections work.

A garment fails all three at the millimetre scale:
- hand smoothing changes a knit's in-plane dimensions by millimetres (document 03 §3);
- a fabric edge is a rounded fold, possibly with fibre fuzz, not a surface boundary;
- moisture content and relaxation history change dimensions over hours to days [E, ISO3759 conditioning rules; magnitude H].

The InspecVision figures above (tens of micrometres over a metre) show what the *sensing* can do on a flat, stiff part. They are a **ceiling** that says sensing is not the fundamental limit, not a prediction for textiles [I].

---

## 2. What other soft-object and ambiguous-landmark domains have learned

| Domain | Object and measurand | Reported performance | Lesson for garments |
|---|---|---|---|
| **Anthropometry and 3D body scanning** | Human body; landmark-defined lengths and girths | ISO 20685-1 sets maximum allowable differences between scan-derived and traditional measurements: 4 mm for heights, widths and small girths; 5 mm for depths and lengths; 9 mm for large girths [E secondary, ISO20685-1]. Kouchi & Mochimaru marked 35 landmarks on 40 subjects five times. Landmarking errors exceeded the resulting body-dimension errors. For 15 of 34 dimensions, marks by two different markers were "not comparable according to the ISO 20685 criterion". The remedy proposed was to "explicitly define landmarks with large errors in more detail" [E, Kouchi2011] | 1. Point-placement disagreement between trained experts is a primary error source, and is hidden if only dimensions are compared. 2. The fix is a better definition, not a better instrument. 3. **Method detail worth copying:** the first marks were made with invisible ink under black light, so the second marker could not see them — a clean way to measure independent expert placement on garments (document 05, S4) |
| **Medical landmark detection** (cephalometric X-rays) | 19 skull landmarks | Inter-observer variability between two clinicians in the ISBI 2015 benchmark: 1.38 ± 1.55 mm. Automated methods: mean radial error ≈ 2 mm; 2 mm success rate ~75 % for the 2015 challenge winner and ~78 % for a 2025 CNN baseline on a newer dataset. A 2025 baseline reports MRE 1.79 ± 6.55 mm — a mean error of under 2 mm with a standard deviation of over 6 mm, i.e. a heavy tail [E, Wang2016-ISBI via Khalid 2025] | **Learned localisers converge to roughly the human inter-observer variability and then stop**, because they are trained on labels that carry it. Their error distribution is heavy-tailed, so mean error hides the occasional gross miss. Both effects are expected for garment keypoints [I] |
| **Fish length on a conveyor** (CatchMeter) | Whole fish, tip-to-tail length | Length SD 1.2 mm; species sorted with 99.8 % reliability [E, CatchMeter] | Deformable objects *can* be measured to ~1 mm on a conveyor when the endpoints are unambiguous (snout, tail tip) and the object lies flat. Ambiguity of endpoints, not deformability per se, is the harder problem [I] |
| **Non-rigid part inspection** (automotive and aerospace sheet parts) | Free-state vs restrained geometry | Standardised by ISO 10579. Research on "fixtureless" inspection numerically restrains a free-state 3D scan (FE-based virtual fixtures) before comparing it with CAD [E, ISO10579; Abenhaim2012 review] | A computational analogue of flattening a garment is conceivable: measure the relaxed, wrinkled 3D shape and compute the flat-state dimension. It depends on material models that garments lack (§5), so it is a research direction, not a near-term tool [H] |
| **Composite ply-boundary inspection** | Carbon-fibre plies (a textile) laid on a tool | A Boeing patent describes vision combined with laser-line discontinuities, laser-tracker registration and comparison with the CAD ply boundary [E, Boeing-ply]. Published accuracy figures could not be verified from primary sources and are not quoted | Industrial textile boundary measurement works when the textile is *held on a known surface* and compared with *known CAD geometry*: strong state control plus strong prior [I] |
| **Flat-pattern photo-digitising** (paper garment patterns) | Paper pattern outlines on a calibrated board | Vendor claims ≈ 1 mm on boards under 2 m and ≈ 2 mm on XL boards [E vendor, VShoot] | A calibrated single-camera board system resolves a flat, thin, stiff-ish sheet to about a millimetre. Garments add thickness, softness and multiple layers, so treat this as an *optimistic* reference point [I] |
| **Robotic cloth perception** | Keypoints on almost-flattened towels, shorts and T-shirts | At 512 × 256 px (≈ 5 mm/px), average keypoint distance was 16.8 px when trained on real data, 18.1 px on synthetic data only ("longer tail"), and 8.7 px with synthetic pre-training plus real fine-tuning. That is ≈ 4–9 cm [E, Lips2024]. Other work analyses garment surfaces with high-resolution active stereo for wrinkle detection and flattening [E bibliographic, Sun2015] | Semantic cloth keypoints for *manipulation* live at the centimetre level. That suffices for grasping and is an order of magnitude too coarse for QC. The domain gap from synthetic data shows up as a heavier error tail |

**Cross-domain synthesis [I]:**
- Wherever the endpoints are unambiguous and the object's state is controlled (fish on a conveyor, flexible gaskets on a backlit table, plies on a tool), millimetre-level or better measurement of deformable objects is routine.
- Wherever the measurand relies on **landmarks that trained humans disagree about** (anthropometry, cephalometry), both humans and algorithms plateau at the inter-observer spread — typically about 1–2 mm on bone or skin landmarks.
- Garment POMs sit between the two. Some endpoints are sharp (the hem edge, a seam intersection); others are soft (the "base of armhole", the SNP under a collar). The distribution of POMs between these classes, and how large the soft-landmark spread is on garments, is not in the literature and must be measured (document 05, S4) [H].

---

## 3. Garment measurement: the state of the art

### 3.1 Standards

ISO 18890:2018 is the first international standard for garment measurement points and method. It explicitly allows "any measurement device which leads to the same results" [E, ISO18890]. Together with ISO 3759 and China's GB/T 31907, it defines the manual reference practice discussed in document 01. **None of these standards contains a precision statement** (repeatability or reproducibility) for the manual method, in the portions examined [E for the portions examined].

### 3.2 Academic work

| Study | Setup | Algorithm | Reported error | Reference ("truth") | Notes on evidence quality |
|---|---|---|---|---|---|
| Cao, Jiang & Jiang 2010 (ICCASM) — via [Paulauskaite2022] | Lying garment | Fuzzy edge detection + Freeman chain-code corners | 0.73–2.84 % relative on T-shirts; length < 1 % | Manual | Secondary report only |
| Li et al. 2017 (ICGSP) — via [Paulauskaite2022] | Camera, LED light, stand, workbench | Garment template + feature points | ≈ 2 % average relative error | Manual | The claim that "~2 cm" is tolerable in industry originates here — not a tolerance source |
| **Paulauskaite-Taraseviciene et al. 2022** [E] | Mannequins, hangers, lying; heterogeneous photos | UNet segmentation (best at **128 × 128**), images resized to **224 × 336**; rule-based keypoints on the mask | MAE: blazers 1.27 cm, dresses 0.75 cm, skirts 1.01 cm | Manual tape | Also measured untrained human error: up to 3.02 cm across 20 people. Resolutions are far too coarse for mm work (≈ 9 mm per mask pixel at 1.2 m; document 03) |
| **Kim et al. 2022** [E] | iPad Pro camera + LiDAR, garments on the floor, 33 designs × 5 types, **10 independent re-lays each** | HRNet-W48 keypoints + LiDAR depth for metric distance | Average relative error 1.59 % and 2.08 % in two environments (≈ 0.8–1.0 cm at 50 cm) | Manual | One of the few studies with repeated re-lays. Compares with a "manual measurement relative error" of 3.25 % (definition unclear) |
| DeepFashion2 (Ge et al. 2019) [E] | 801K clothing items from consumer and commercial photos, dense landmarks and masks | Match R-CNN baseline | Not a measurement study | Human-annotated landmarks | Widely used for *semantic* garment landmarks; says nothing about metric accuracy |

**Common pattern [I]:**
- Errors of about 1–2 % (≈ 0.7–1.3 cm) against a manual tape of unknown uncertainty.
- Small sample sizes.
- No separation of repeatability, re-lay variability, definitional spread and algorithm error.
- No pass/fail risk analysis.
- Input resolutions chosen for semantic recognition, not metrology.

This is exactly the "80 %-style" evidence that cannot answer the industrial question.

### 3.3 Patents and commercial systems

| Item | What it is | Claims | Validation publicly available? |
|---|---|---|---|
| **US5530652A** — Levi Strauss & Co, filed 1993, expired [E] | Flattened garments on an entry conveyor pass under a CCD camera and are compared with stored patterns and tolerances. A 3D variant inflates trousers "to fully expand" them on a rotating platform | No accuracy figures | No |
| **Yunji Zhihui** (Dongguan), reported 2021 [E trade press] | AI measuring machine for T-shirts, 15 dimensions | "± 1mm", "99.5 per cent", beds up to 900 × 900 mm, 6 s/piece | No. A 900 mm bed cannot hold most trousers or long-sleeve tops flat [I] |
| **Size AI** (2026) [E vendor] | iPhone LiDAR flat-lay app | "~5 mm … typical with LiDAR, under optimal conditions". It calls itself "a screening layer, not a certificate" and recommends the tape for "final verification against tight tech-pack tolerances"; "tolerance-grade capture … is on our roadmap". The same vendor notes that "major wrinkles can shift measurements by 2–3 cm" | No methodology published |
| Other flat-lay AI tools (e.g. seamline, Tailored) [trade/vendor pages] | Photo-based POM detection | Claims of ~0.3 cm | Not found |

**Assessment [I].** In 30 years since the Levi Strauss patent, no public source was found that reports a garment measurement system validated to industrial MSA standards. The most candid current vendor positions its product as a screening tool. That is consistent with — but does not prove — the hypothesis that the unsolved part is not the camera but the measurand definition and the garment state.

---

## 4. Computer-vision foundations and where they break down for metrology

### 4.1 Calibration and planar measurement

- **Calibration is a solved problem for rigid scenes.** Zhang's method — a planar target seen at several orientations, closed-form estimation, then maximum-likelihood refinement — underlies the standard checkerboard calibration [E, Zhang2000].
- **On rigid parts, monocular non-telecentric vision reaches the micrometre range.** One 2023 study (100 × 100 mm field, 0.8 m working distance, logistic-model sub-pixel edge fitting) reports a gauge-block mean error of 0.008 mm, a repeatability SD of 0.0063 mm, and a maximum deviation of 0.013 mm from a CMM [E, Nogueira2023]. That is on the order of 1/10,000 of the field.
- **Sub-pixel edge and line extraction is mature** when the edge profile is modelled explicitly, and its bias can be removed [E, Steger].

**Implication [I]:** a calibrated 2D camera over a 1.2 m flat field is not intrinsically limited to millimetres. Scaled crudely, the rigid-part result suggests ~0.1 mm is achievable on ideal edges. The limits for garments come from what the edge *is* (document 03), from height/perspective, and from the garment state.

### 4.2 Keypoint detectors were designed for semantic localisation, not measurement

- **HRNet,** a widely used keypoint backbone, crops each subject and resizes it to **256 × 192 or 384 × 288**. It uses a stem of "two strided convolutions decreasing the resolution" (heatmaps at ¼ of input resolution). It decodes positions by "adjusting the highest heatvalue location with a quarter offset" toward the second-highest response [E, HRNet].
- **DARK** showed that the heatmap-to-coordinate decoding step is "surprisingly significant" and biased in standard pipelines [E, DARK].
- Applied to a whole garment spanning about 1.2 m, a 64-cell heatmap row corresponds to **≈ 19 mm per cell**, and a 96-cell row to ≈ 12.5 mm (document 03 §5). Sub-cell refinement helps. But the architecture is designed to find a wrist to within a few percent of a person's size, not an armhole point to within a millimetre [I].
- Robotic cloth keypoint work at ≈ 5 mm/px reports average errors of several centimetres (§2) [E, Lips2024].

**Implication [I]:** a keypoint network should be treated as a *semantic proposal generator* that says roughly where the armhole is. Metric localisation should then be done by a geometric stage operating at full sensor resolution on physically defined features (edges, seam lines, intersections of fitted curves). This is standard practice in industrial vision (coarse localisation, then fine metrology on the original image). The previous team's "algorithms intended to correct inaccurate keypoints" suggest a pipeline that lacked this geometric stage (document 03 §8) [H].

### 4.3 Segmentation models for background removal

- RMBG-1.4 (IS-Net-based) and RMBG-2.0 (BiRefNet-based) both resize input to **1024 × 1024** [E, RMBG-1.4, RMBG-2.0]. Over a 1.2 m field, that is ≈ 1.2 mm per mask pixel before upsampling. The soft alpha edge is a learned estimate of where "foreground" ends, not a metrological edge definition. Both are licensed non-commercial by default; commercial use needs an agreement with BRIA [E].
- With a *controlled background*, background removal is a physics problem, not a learning problem. A backlit table gives a silhouette by transmission; a uniform, colour-controlled or retro-reflective background gives a threshold. The metrological edge can then be fitted at full resolution. A green sheet enables chroma keying but creates two known failure modes: green garments, and green spill or reflection onto fabric edges [I].

### 4.4 Synthetic training data: generated images vs. simulated physics

The previous team trained keypoint models on thousands of text-to-image generated flat-lays (Gemini 2.5 Flash Image, "Nano Banana" [E, NanoBanana]) on white backgrounds, then deployed them on real garments on a green sheet. The literature supports several specific concerns:

1. **Generated images are not geometrically faithful.** Classifiers that see only derived geometric features — perspective fields, line structure, object–shadow relations — reliably separate generated images from real ones, even images pre-selected to fool pixel-based detectors. Current generators "cannot reliably reproduce geometric properties of real images" [E, Sarkar2024]. For metrology the consequence is that seam placement, left–right symmetry, proportions between POMs and the location of fold lines in a generated flat-lay need not correspond to *any* physically sewable garment [I].
2. **Labels on generated images inherit the annotator's interpretation, with no physical check.** There is no physical garment to measure. Keypoint labels therefore encode the annotator's visual prior of where "the armhole point" is, which is precisely the definitional ambiguity the system must resolve [I].
3. **Domain gap.** Background (white vs green), illumination, the camera's optics and noise, fabric textures and real wrinkles all differ between generated training images and real captures. Physically based synthetic cloth data *with exact labels* still shows a residual sim-to-real gap and a heavier error tail. In the Ghent study, synthetic-only training gave mAP 64.3 vs 59.8 for real-only but a worse average distance (18.1 vs 16.8 px) because of "a longer tail". Fine-tuning on real data was needed (8.7 px), and the authors conclude that "further improvements to the fidelity of cloth assets will be required" [E, Lips2024].
4. **Physically based alternative.** Pattern-based cloth simulation (e.g. CLO3D) produces labels that are *exact by construction*: seam lines, pattern corners and POM endpoints are known on the mesh. Its geometric fidelity is limited but measurable: one 2025 study reports average differences between simulated and real prototype deformation "up to 6 % with an 8 % standard deviation" [E, MahnicNaglic2025]. That is good enough for training *semantic* proposal models with correct label definitions, but not as a metric ground truth [I].

**Bottom line [I].** Text-to-image synthetic data is a plausible way to bootstrap a coarse garment-part detector. It is structurally unsuited to teaching a model *where a metrological reference point is to within a millimetre*. The previous team's reference-point displacement is what one would predict from this training regime combined with coarse keypoint architectures. Whether it was the dominant error is unknown and testable (document 05, S5–S6).

---

## 5. Garment CAD, patterns and simulation

- **Design data is a strong prior.** It tells the system which POMs exist, their nominal values and grading, the garment topology (which seams exist, where the sleeves join) and approximate feature locations. Datasets with sewing patterns and simulated 3D garments exist for research (e.g. GarmentCodeData, 2024, found in searches but not examined in detail) [E bibliographic].
- **Design data is not the truth (document 01 §5).** A prior that pulls estimates toward nominal geometry is dangerous in QC: it hides deviations. Any use of CAD (template registration, shape priors, constrained optimisation) must be *audited for shrinkage toward nominal*, e.g. by measuring seeded out-of-tolerance garments (document 05) [I].
- **Simulated flat-lays** can predict where fold lines fall, how the side seam rolls, and how much a sleeve foreshortens when laid at an angle. This is useful for designing lay protocols and for synthetic data. Simulation error (percent-level deformation differences [E, MahnicNaglic2025]) means it cannot replace physical validation [I].

---

## 6. What the state of the art does and does not establish

| Claim | Status |
|---|---|
| Calibrated optical systems can measure large *flat, stiff* parts to tens of micrometres | **Established** [E, InspecVision; Nogueira2023] |
| Deformable objects with unambiguous endpoints can be measured on conveyors to ~1 mm | **Established** for fish [E, CatchMeter]; plausible for garments [H] |
| Where the measurand depends on ambiguous landmarks, both humans and learned models plateau near the inter-observer spread, with heavy-tailed algorithm errors | **Established** in anthropometry and cephalometry [E, Kouchi2011; Wang2016-ISBI]; expected for garments [H] |
| Published garment measurement systems achieve ~1–2 % (≈ 1 cm) against a manual tape | **Established** for the studies found [E, Paulauskaite2022; Kim2022] |
| Any garment measurement system has demonstrated industrial MSA-grade capability (%GRR, uncertainty budget, pass/fail risk) | **Not found** in any public source |
| Semantic cloth keypoint detectors reach metrological precision | **Contradicted** by available evidence (centimetre-level) [E, Lips2024] |
| Text-to-image synthetic data preserves the geometry needed for metrological labels | **Contradicted** in general terms [E, Sarkar2024]; untested for garments specifically [H] |
| The binding constraint for garment measurement is the measurand definition and the garment state, not the sensor | **Hypothesis**, strongly suggested by the evidence above; to be tested (document 05) [H] |

---

## Sources cited in this document

Full annotated entries in [references.md](references.md).

- `[ISO17450-2]` ISO 17450-2:2012 (iTeh preview). https://cdn.standards.iteh.ai/samples/53629/2ee7b9b5e346412f87828f9adfe5e4be/ISO-17450-2-2012.pdf
- `[ISO10579]` ISO 10579:2010 (iTeh preview). https://cdn.standards.iteh.ai/samples/54707/4d70c7544c894dce8a99e89b862cdf09/ISO-10579-2010.pdf
- `[ISO3759]` ISO 3759:2011 (iTeh preview). https://cdn.standards.iteh.ai/samples/57309/1af372753bf84b3cb0288a274429c73d/ISO-3759-2011.pdf
- `[ISO18890]` ISO 18890:2018 (iTeh preview). https://cdn.standards.iteh.ai/samples/63693/131110d2f792493aabb7200cbd9136ba/ISO-18890-2018.pdf
- `[ISO22514-7]` ISO 22514-7:2021 (iTeh preview). https://cdn.standards.iteh.ai/samples/iso/iso-22514-7-2021/087b2771a1074737a409ba3501bb3329/iso-22514-7-2021.pdf
- `[JCGM106]` JCGM 106:2012. https://www.bipm.org/documents/20126/2071204/JCGM_106_2012_E.pdf
- `[TMU]` Banke & Archie (1999) and later SPIE work on total measurement uncertainty; e.g. SPIE 7272, 727203 (2009). https://www.spiedigitallibrary.org/conference-proceedings-of-spie/7272/727203/The-measurement-uncertainty-challenge-of-advanced-patterning-development/10.1117/12.814170.pdf
- `[TIS]` US 8,860,941 B2, *Tool induced shift reduction determination for overlay metrology*. https://patents.google.com/patent/US8860941B2/en
- `[InspecVision]` InspecVision Planar P220.35 specifications. https://www.inspecvision.com/planar2d-models/planar-p220.35 ; overview https://www.inspecvision.com/products-planar2d
- `[Telecentric]` Edmund Optics, *Large field of view telecentrics*; Opto Engineering; Vision & Control. https://www.edmundoptics.com/knowledge-center/trending-in-optics/large-fov-telecentrics/ · https://www.opto-e.com/en/news/Order-now-the-biggest-telecentric-lenses-and-lights-ever-made-by-Opto-Engineering · https://www.vision-control.com/en/press-release/new-telecentric-wide-field-lenses-for-inspecting-large-objects/
- `[Luhmann2010]` T. Luhmann, "Close range photogrammetry for industrial applications", *ISPRS J. Photogramm. Remote Sens.* 65(6), 2010. https://doi.org/10.1016/j.isprsjprs.2010.06.003
- `[ISO20685-1]` ISO 20685-1:2018. https://www.iso.org/standard/63260.html
- `[Kouchi2011]` M. Kouchi, M. Mochimaru, *Applied Ergonomics* 42(3):518–527, 2011. https://pubmed.ncbi.nlm.nih.gov/20947062/
- `[Wang2016-ISBI]` C.-W. Wang et al., *Medical Image Analysis* 31:63–76, 2016; figures as reported in Khalid et al., *Scientific Data* 2025. https://pmc.ncbi.nlm.nih.gov/articles/PMC12313948/
- `[CatchMeter]` D. J. White, C. Svellingen, N. J. C. Strachan, *Fisheries Research* 80(2–3):203–210, 2006. https://www.sciencedirect.com/science/article/abs/pii/S0165783606001512
- `[Abenhaim2012]` G. N. Abenhaim, A. Desrochers, S. A. Tahan, "Nonrigid parts' specification and inspection methods: notions, challenges, and recent advancements", *Int. J. Adv. Manuf. Technol.* 63:741–752, 2012. https://link.springer.com/article/10.1007/s00170-012-3929-2
- `[Boeing-ply]` US 2009/0043533 A1, *Methods and systems for automated ply boundary and orientation inspection*. https://patents.google.com/patent/US20090043533
- `[VShoot]` Velocity Plotters, V-Shoot camera digitizing system. https://velocityplotters.com/camera-digitizing-system-apparel-garment-manufacturing/
- `[Lips2024]` T. Lips, V.-L. De Gusseme, F. wyffels, "Learning keypoints for robotic cloth manipulation using synthetic data", IEEE RA-L 2024. https://arxiv.org/abs/2401.01734
- `[Sun2015]` L. Sun, G. Aragon-Camarasa, S. Rogers, J. P. Siebert, ICRA 2015, pp. 185–192.
- `[Paulauskaite2022]` *Applied Sciences* 12(9):4470, 2022. https://doi.org/10.3390/app12094470
- `[Kim2022]` S. Kim et al., *Applied Sciences* 12(10):5286, 2022. https://doi.org/10.3390/app12105286
- `[DeepFashion2]` Y. Ge et al., CVPR 2019. https://arxiv.org/abs/1901.07973
- `[US5530652A]` Levi Strauss & Co, *Automatic garment inspection and measurement system*. https://patents.google.com/patent/US5530652A/en
- `[YunjiZhihui2021]` Apparel Resources, 23 Jan 2021. https://apparelresources.com/technology-news/manufacturing-tech/finally-ai-manufacturing-garment-measurement-inspection-flash/
- `[SizeAI-QC]`, `[SizeAI-cheatsheet]` https://sizeai.co/qc · https://sizeai.co/blog/flat-lay-cheat-sheet
- `[Zhang2000]` Z. Zhang, *IEEE TPAMI* 22(11):1330–1334, 2000. https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/tr98-71.pdf
- `[Nogueira2023]` V. V. E. Nogueira, L. F. Barca, T. C. Pimenta, *Sensors* 2023. https://pmc.ncbi.nlm.nih.gov/articles/PMC10346874/
- `[Steger]` C. Steger, "Subpixel-precise extraction of lines and edges", IAPRS 33, 2000; "An unbiased detector of curvilinear structures", *IEEE TPAMI* 20(2), 1998.
- `[HRNet]` K. Sun, B. Xiao, D. Liu, J. Wang, CVPR 2019. https://arxiv.org/abs/1902.09212
- `[DARK]` F. Zhang et al., CVPR 2020. https://arxiv.org/abs/1910.06278
- `[RMBG-1.4]`, `[RMBG-2.0]` BRIA model cards. https://huggingface.co/briaai/RMBG-1.4 · https://huggingface.co/briaai/RMBG-2.0
- `[NanoBanana]` Google Developers Blog, *Introducing Gemini 2.5 Flash Image*. https://developers.googleblog.com/en/introducing-gemini-2-5-flash-image/
- `[Sarkar2024]` A. Sarkar et al., CVPR 2024. https://arxiv.org/abs/2311.17138
- `[MahnicNaglic2025]` M. Mahnić Naglić, S. Petrak, A. Tomljenović, *Polymers* 17(15):2013, 2025. https://doi.org/10.3390/polym17152013
