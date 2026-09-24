# 02 — Evidence base: transferable metrology, analogous domains and the state of the art

*Consolidated research, v2. Evidence labels:*
- ***[E]*** *— established from a source that was opened;*
- ***[E-sec]*** *— via a secondary source;*
- ***[E-vendor]*** *— a manufacturer's claim;*
- ***[I]*** *— inference;*
- ***[H]*** *— hypothesis.*

*Keys resolve in [references.md](references.md).*

This document collects the evidence the argument rests on:
- what precision metrology does that transfers to garments (§1);
- what fields measuring soft or ambiguous objects have learned (§2);
- what exists for garment measurement today, academic and commercial (§3);
- which vision and simulation building blocks matter, and where they fail for metrology (§4).

---

## 1. How precision metrology achieves accuracy, and what transfers

Accuracy in semiconductor, aerospace and precision engineering comes mainly from **how the measurement problem is posed**, not from better cameras. The table shows each principle and whether it transfers to garments.

| # | Principle | How precision fields use it | Transfer to garments |
|---|---|---|---|
| P1 | **Define the measurand completely** | GPS specification operators; "a complete specification operator is unambiguous" [E, ISO17450-2] | **Fully.** The cheapest lever available (document 01 §3.3) |
| P2 | **Specify and control the part's state** | Non-rigid parts are measured in a stated free state (gravity, orientation, support) or a specified restrained condition [E, ISO10579] | **Partly.** Garments can be supported, floated, held or tensioned, but every restraint changes the geometry, so the restraint becomes part of the measurand and must be agreed (document 04 §2) |
| P3 | **Traceable calibration with artefacts** | Scale bars and test fields for optical systems (VDI/VDE 2634); ISO 10360-based verification, e.g. for backlit 2D gauges [E-sec, VDI2634; E-vendor, InspecVision] | **Fully, for the sensing chain.** Rigid artefacts verify camera, lens and algorithm independently of any garment |
| P4 | **Uncertainty budgets** | GUM; ISO 14253-2; ISO 22514-7 [E, JCGM106, ISO22514-7] | **Fully,** provided the budget includes definitional and garment-state components, which rigid-part practice assumes are small [E, ISO22514-7] |
| P5 | **Fit features over many points** | Diameters, planes and datums fitted to many points rather than trusting single points | **Partly.** Fitting over hundreds of edge pixels suppresses random noise (~1/√N), but fabric waviness, fuzz and seam roll are not random, so the filtering must itself be specified (P1) |
| P6 | **Reference measurement systems** | A fast production tool is validated against a slower reference (e.g. CD-AFM for CD-SEM). Mandel/errors-in-variables regression separates the reference's own error, giving "total measurement uncertainty" [E-sec, TMU] | **Fully, as the validation method** (document 07) |
| P7 | **Optics chosen for the measurand** | Backlit silhouette gauges: a 1830 × 1220 mm backlit table with ±35 µm over most of the field and ±70 µm at the edges on flat parts, including flexible gaskets [E-vendor, InspecVision]. Telecentric lenses remove perspective, but their field is limited by the front optic (≈ 0.4–0.6 m at most) [E-vendor-sec, Telecentric] | **Mostly.** Garment-sized backlit tables are practical, and one commercial garment station uses one (§3.2). Telecentric optics cannot cover a garment. The textile edge, not the optics, limits precision |
| P8 | **Design for metrology — add observable features** | Semiconductor overlay targets; photogrammetric targets; textile dimensional-change tests *mark* reference points because edges are poorly defined [E, ISO3759 §4.2] | **Conditionally.** Essential for validation (marked garments). Speculative for production |
| P9 | **Reversal to expose instrument bias** | Tool-induced shift in overlay metrology: TIS = ½[OVL(0°) + OVL(180°)] [E-sec, TIS] | **Fully.** Measure the same lay rotated 180° to expose lighting, perspective and model asymmetries |

**Why the *numbers* do not transfer [I].** Precision fields measure objects that are stiff, sharply bounded and predictable. Garments are none of these at the millimetre scale:
- Knits have "almost vanishing rigidity" at low load, **large variability over repeated load cycles and large hysteresis** from inter-yarn friction [E, Poincloux2018].
- Woven fabrics are strongly **anisotropic**. On the 45° bias no yarn is held at both ends, so tension builds only through yarn–yarn friction [E, PanBias2015]. Fabric objective-measurement testing derives shear rigidity from the bias extension at just 5 gf/cm (G = 123/EB5, N/m) [E-sec, FAST-shear], so bias directions extend far more under light handling than the yarn directions.
- Edges are rounded folds with fibre fuzz, not surfaces.
- Moisture and relaxation change dimensions: ISO 3759 conditions garments to constant mass before dimensional testing [E]. The size of this effect under factory humidity swings was **not** found in primary sources [H].

---

## 2. Analogous soft-object and ambiguous-landmark domains

| Domain | Evidence | Lesson |
|---|---|---|
| **Anthropometry / 3D body scanning** | ISO 20685-1 allowable scan-vs-manual differences: 4 mm (heights, widths, small girths), 5 mm (lengths), 9 mm (large girths) [E-sec, ISO20685-1]. Kouchi & Mochimaru marked 35 landmarks on 40 subjects five times. Landmark errors *exceeded* the resulting dimension errors, and for 15 of 34 dimensions two markers were "not comparable according to the ISO 20685 criterion". The remedy was to "explicitly define landmarks … in more detail" [E, Kouchi2011] | Expert disagreement about *points* is a primary error source that dimension-level agreement hides. The fix is definition, not instrumentation. **Their method — first marks made with invisible ink under black light — is directly reusable** (document 07, S4) |
| **Medical landmark detection** (cephalometric X-rays) | ISBI 2015: inter-observer 1.38 ± 1.55 mm; algorithms ≈ 2 mm mean radial error; ~75 % within 2 mm. A 2025 CNN baseline on a newer dataset: MRE 1.79 ± 6.55 mm, 78 % within 2 mm [E-sec, Wang2016-ISBI via Khalid 2025] | Learned localisers approach the human inter-observer spread and stop there. Their errors are **heavy-tailed** (an SD far above the mean) |
| **Fish length on a conveyor** | Length SD 1.2 mm; species 99.8 % [E abstract, CatchMeter] | Deformable objects with **unambiguous endpoints** can be measured to ~1 mm while moving. Ambiguity, not deformability, is the harder problem |
| **Non-rigid part inspection** | ISO 10579 restrained states; research on "fixtureless" inspection that numerically restrains a free-state scan before comparing with CAD [E, ISO10579; E abstract, Abenhaim2012] | Computational "virtual flattening" of a wrinkled garment would need material models garments lack. A research direction, not a near-term tool [H] |
| **Composite ply inspection** (a textile on a tool) | A Boeing patent combines vision, laser-line discontinuities and laser-tracker registration against CAD ply boundaries. **The patent states no accuracy figure** [E, Boeing-ply] | Textile boundary inspection works when the textile is **held on a known surface** and compared with **known CAD**: strong state control plus a strong prior. No numerical transfer |
| **Pattern photo-digitising** | ≈ 1 mm on boards under 2 m, ≈ 2 mm on XL boards [E-vendor, VShoot] | A calibrated single-camera board resolves thin, flat sheets to about a millimetre. Garments are harder |
| **Robotic cloth perception** | Keypoints on almost-flattened towels, shorts and T-shirts at 512 × 256 px (≈ 5 mm/px): average error 16.8 px trained on real data, 18.1 px on synthetic only ("longer tail"), 8.7 px on synthetic + real — i.e. **≈ 4–9 cm** [E, Lips2024] | Semantic cloth keypoints for *manipulation* live at the centimetre scale. Synthetic-only training fattens the error tail |

**Synthesis [I].** Deformable objects are measured routinely to about a millimetre when endpoints are unambiguous and the state is controlled. Where the measurand depends on landmarks that trained people disagree about, both humans and algorithms plateau at the inter-observer spread — about 1–2 mm on anatomical landmarks. Garment POMs contain both kinds of endpoint. **How large the expert spread is for garments has not been measured** [H].

---

## 3. Garment measurement: the state of the art

### 3.1 Academic work

| Study | Setup | Method | Reported error | Reference | Evidence quality |
|---|---|---|---|---|---|
| Cao, Jiang & Jiang 2010 (via Paulauskaite2022) | Lying garment | Fuzzy edges + chain-code corners | 0.73–2.84 % relative | Manual | Secondary |
| Li et al. 2017 (via Paulauskaite2022) | Camera, LED, stand, workbench | Template + feature points | ≈ 2 % relative | Manual | Secondary. The source of the "~2 cm tolerable" remark, which is **not** a tolerance source |
| **Paulauskaite-Taraseviciene et al. 2022** [E] | Mannequin, hanger or lying; mixed photos | UNet masks (best at 128 × 128) on 224 × 336 images; rule-based points | MAE 0.75–1.27 cm | Manual tape | ≈ 9 mm per mask pixel at 1.2 m — semantic-scale resolution |
| **Kim et al. 2022** [E] | iPad Pro camera + LiDAR; 33 designs, **10 independent re-lays** each | HRNet-W48 keypoints + LiDAR depth | 1.59 % / 2.08 % relative (≈ 0.8–1.0 cm at 50 cm) | Manual (reported as 3.25 % "relative error", undefined) | One of the few studies with re-lays; components not separated |
| DeepFashion2 (Ge et al. 2019) [E abstract] | 801K consumer/commercial clothing items | Dense *semantic* landmarks and masks; Match R-CNN baseline | — (not a measurement study) | Human-annotated landmarks | Widely used for garment landmarks; no metric calibration or dimensional ground truth [E abstract, DeepFashion2] |
| Unverified lead: "Measurement system of garment dimension based on machine vision" (ResearchGate listing) | CCD camera; SIFT/Förstner corners | Search-snippet report: "standard error < 0.25 mm", "repeatability ≈ 5 mm" | — | — | **Not opened (access denied).** If confirmed, that contrast — sub-mm image error vs ~5 mm repeatability — would be exactly the signature of garment-state variability dominating. Worth obtaining |

**Common pattern [I]:**
- errors of about 1–2 % against a manual tape of unknown uncertainty;
- small samples;
- no separation of repeatability, re-lay, definitional and algorithm components;
- no decision-risk analysis;
- input resolutions chosen for recognition, not metrology.

### 3.2 Commercial systems and patents

| System | What it is | Claims | Validation public? | Relevance |
|---|---|---|---|---|
| **Toyoshima VIRTUAL STANDARD AI MEASURE** (Japan; 2nd generation announced 10 Jul 2025) | Dedicated camera station. "A single picture with a dedicated camera"; operators "adjust the measurement points on the monitor screen according to the design". The 2nd generation enlarges the base (long sleeves, large sizes), adds **product weighing to 1 g**, and adds web operation | "Work time by more than half"; "more accurate … than conventional manual work"; press coverage headlined a "2 mm margin of error" [E-vendor, Toyoshima] | No (the interview detail is paywalled) | **An industrial precedent for human-in-the-loop semantics + machine measurement** (document 04, C1). Also a benchmark for speed: at best about 2× faster than manual |
| **Seamline Workbench** (Tailored) | Vendor description: "a purpose-built photo station consisting of a **backlit table**, a high-resolution machine vision camera, and a mini-computing unit", US$5–7k per station | "~0.3 cm" for "the best systems in this category"; "hundreds of garments per shift" [E-vendor, via search summary; page not directly retrievable] | No | A commercial precedent for **backlit-table** garment measurement |
| **Yunji Zhihui** (China, 2021) | AI measuring machine for T-shirts | "± 1mm", "99.5 per cent", 6 s/piece, beds up to 900 × 900 mm [E-vendor, YunjiZhihui2021] | No | A 900 mm bed cannot hold most trousers or long-sleeve tops flat |
| **Size AI** (2026) | iPhone LiDAR flat-lay app | "~5 mm … under optimal conditions"; describes itself as **"a screening layer, not a certificate"** and keeps the tape for "tight tech-pack tolerances"; "major wrinkles can shift measurements by 2–3 cm" [E-vendor, SizeAI-QC; SizeAI-cheatsheet] | No | A candid vendor boundary: phone capture ≠ tolerance-grade |
| **US 5,530,652 A** (Levi Strauss, filed 1993, expired) | Flattened garments on a conveyor under a CCD camera, compared with stored patterns; 3D variant inflates trousers | No figures [E, US5530652A] | — | Conveyor measurement of flat garments has been proposed for three decades |
| **CN 106174830 A** | "Garment dimension automatic measurement system based on machine vision" | Bibliographic only [E-sec, CN106174830A] | — | Prior art exists in China |

**Assessment [I]:**
- Industry has converged on a **flat table with an overhead camera**, sometimes **backlit**, with the **operator confirming or adjusting points**. That is the architecture this study reaches independently from metrology principles (document 04).
- **No public source reports an MSA-grade validation** (uncertainty budget, re-lay variance, pass/fail risk) for any garment system.
- The best-documented speed claim (Toyoshima) is **about 2× faster than manual**, not an order of magnitude — relevant to the throughput question (document 05).

---

## 4. Vision and simulation building blocks: where they break for metrology

### 4.1 Calibration and planar measurement

- Planar-target calibration (Zhang) is standard [E, Zhang2000].
- On rigid parts, monocular non-telecentric vision reaches the micrometre range. One study over a ~100 mm field reports a gauge-block mean error of 0.008 mm, repeatability SD 0.0063 mm, and ≤ 0.013 mm deviation from a CMM [E, Nogueira2023].
- Sub-pixel edge extraction with explicit models can be made unbiased [E, Steger].

**Implication [I]:** sensing is not intrinsically limited to millimetres. The limits come from what the textile edge *is*, from garment height (perspective), and from the garment state (document 03).

### 4.2 Keypoint networks are semantic localisers

- HRNet crops a subject to **256 × 192 or 384 × 288**. Its stem of "two strided convolutions" produces ¼-resolution heatmaps, and it decodes by a "quarter offset" toward the second-highest response [E, HRNet].
- DARK showed that heatmap decoding is "surprisingly significant" and biased in standard practice [E, DARK].
- Mapped onto a 1.2 m garment, one heatmap cell spans ≈ 12–19 mm (document 03 §5) [I].

These architectures are built to find a wrist to within a few percent of a person's size, not a seam junction to within a millimetre.

### 4.3 Background removal

RMBG-1.4 (IS-Net) and RMBG-2.0 (BiRefNet) resize input to **1024 × 1024** [E, RMBG-1.4; RMBG-2.0]. That is ≈ 1.2 mm per mask pixel over 1.2 m, with a *learned* soft edge. Both are non-commercial by default. With a controlled background — backlight, contrasting or retro-reflective surface — segmentation becomes a physical threshold, and the metrological edge can be fitted at full resolution [I].

### 4.4 Synthetic training data

- **Text-to-image generators and geometry.** Classifiers that see only geometric cues (perspective fields, lines, object–shadow relations) reliably detect generated images. The authors conclude that "current generators cannot reliably reproduce geometric properties of real images" [E, Sarkar2024]. **Scope caveat:** they tested SDXL, Kandinsky-v3, DeepFloyd IF, PixArt-α, DALL·E 3 and Firefly on indoor and outdoor scenes — **not** Gemini's image model and not garment flat-lays. The finding is a strong warning, not a direct measurement of Nano Banana flat-lays [E for scope].
- **Resolution of generated images.** Gemini 2.5 Flash Image ("Nano Banana") generates natively at about **1024 × 1024** [E-sec, NanoBanana-res]. Newer Gemini image models offer 2K/4K.
- **Physically based synthetic data still leaves a gap.** Procedural meshes deformed in Nvidia Flex and rendered in Blender, with exact labels, still gave centimetre-level keypoint errors and a heavier error tail than real data. The authors conclude "further improvements to the fidelity of cloth assets will be required" [E, Lips2024].
- **Simulation fidelity.** CLO3D simulations parameterised from measured fabric properties differed from real prototypes (ARAMIS optical measurement) in deformation by "up to 6 % with an 8 % standard deviation" [E abstract, MahnicNaglic2025]. Good enough to pre-train *semantic* models with exact label definitions; not good enough for metric truth [I].

### 4.5 Fabric texture as a measurement signal

Automatic image-based measurement of **knit course and wale density** is established. One method uses **transmitted-light (backlight) images**, skew correction and projection curves, reporting "accuracy above 98 % when compared with the manual mode" [E abstract, ZhangKnit2023]. Density-measurement methods also exist for tight woven fabrics.

This enables a new hypothesis. **The fabric's own structure can act as a strain gauge:** local stitch density in the lay, compared with the fabric's relaxed density, reveals handling strain (document 03 §7) [H].

---

## 5. What the evidence does and does not establish

| Claim | Status |
|---|---|
| Calibrated optical systems measure large flat, stiff parts to tens of micrometres | **Established** [E-vendor, InspecVision; E, Nogueira2023] |
| Deformable objects with unambiguous endpoints can be measured to ~1 mm | **Established** for fish [E]; plausible for well-defined garment POMs [H] |
| With ambiguous landmarks, humans and learned models plateau near the inter-observer spread, with heavy-tailed model errors | **Established** in anthropometry and cephalometry [E]; expected for garments [H] |
| Published garment systems achieve ~1–2 % (≈ 1 cm) against an uncharacterised tape | **Established** for the studies found [E] |
| Commercial garment stations exist: flat table, sometimes backlit, with operator adjustment of points | **Established** as vendor descriptions [E-vendor] |
| Any garment system has demonstrated MSA-grade capability | **Not found** |
| Cloth keypoint detectors reach millimetre precision | **Contradicted** by the available evidence [E, Lips2024] |
| Text-to-image data preserves the geometry needed for metrological labels | **Contradicted** for the generators tested [E, Sarkar2024]; untested for Nano Banana flat-lays [H] |
| The binding constraints for garments are the measurand definition and the garment state, not the sensor | **Hypothesis,** strongly indicated; tested by document 07 |

---

## Sources cited in this document

See [references.md](references.md): ISO17450-2, ISO10579, VDI2634, InspecVision, JCGM106, ISO22514-7, TMU, Telecentric, ISO3759, TIS, Poincloux2018, PanBias2015, FAST-shear, ISO20685-1, Kouchi2011, Wang2016-ISBI, CatchMeter, Abenhaim2012, Boeing-ply, VShoot, Lips2024, Paulauskaite2022, Kim2022, Toyoshima, Seamline, YunjiZhihui2021, SizeAI-QC, SizeAI-cheatsheet, US5530652A, CN106174830A, DeepFashion2, Zhang2000, Nogueira2023, Steger, HRNet, DARK, RMBG-1.4, RMBG-2.0, Sarkar2024, NanoBanana-res, MahnicNaglic2025, ZhangKnit2023.
