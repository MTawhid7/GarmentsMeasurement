# 06 — The previous team's approach, reassessed

*Consolidated research, v2. Evidence labels: [E] source opened · [E-sec] secondary · [E-vendor] manufacturer claim · [I] inference · [H] hypothesis.*

*The implementation details of the previous system are not available. This document separates **what is known**, **what can reasonably be inferred** and **what cannot be known without their artefacts**. It then turns each inference into a test. Where the independent review stated failure causes as facts ("exact root causes", "caused chroma bleed", "snapped points … masked defects"), those have been downgraded to hypotheses.*

---

## 1. What is known

| Item | Information provided |
|---|---|
| Effort | About two years of development |
| Hardware | Zivid 3D camera plus substantial GPU; about €15k in total |
| Segmentation | RMBG-class background removal / segmentation |
| Localisation | Keypoint detection, plus algorithms "intended to correct inaccurate keypoints" before measurements were computed |
| Training data | Thousands of synthetic garment images generated with Gemini / "Nano Banana", mainly spread-out garments on plain **white** backgrounds, used to train keypoint detection |
| Real acquisition | Garments spread on a **green** sheet |
| Result | "Around 80 % accuracy" (undefined) |
| Main difficulty | Detected points "slightly displaced from their intended physical locations", causing measurement errors |

## 2. What cannot be known without their code, data or results

- What "80 %" measured, against which reference, and with what reference uncertainty.
- The keypoint architecture, input resolution and decoding.
- How the labels were drawn, by whom, and with which definitions.
- Whether and how the Zivid depth data was used: height correction, surface lengths, or nothing.
- How the camera was calibrated to real-world units, and whether that was verified.
- What the "correction algorithms" did.
- Whether any real, measured garments were used for training or testing.
- Whether errors were dominated by particular POMs, fabrics or colours.

**Consequence.** Any statement that one component "caused" their result is a hypothesis. The useful output of a retrospective is a set of **discriminating tests**, each able to implicate or clear a component (§5).

---

## 3. Component by component

### 3.1 Zivid 3D camera

- **Known [E, Zivid2+L110]:**
  - colour image 2448 × 2048 (≈ 0.44 mm/px over its 1090 × 850 mm field at 1.1 m);
  - point precision 260 µm;
  - dimension trueness error < 0.35 % / 0.40 % / 0.60 % (70th percentile);
  - capture 50–1500 ms.
- **Inferences [I]:**
  - For planar dimensions it samples 2–3× coarser than an inexpensive 45–61 MP 2D camera.
  - Its datasheet trueness corresponds to ≈ 2.5–4.2 mm over 700 mm unless re-verified in the measurement plane.
  - If depth was integrated along the surface, noise would have inflated lengths strongly (+27 % at native spacing, document 03 §4.4).
  - Its field of view at the optimal distance is tight for trousers and outstretched long sleeves.
- **What it is good for:** a **height channel**. Its depth precision is ample for correcting perspective and detecting non-flatness (document 04 §3.2).
- **Status:** a poor choice as the *metric* sensor, a good one for *height* [I]. Whether it limited their results depends on how it was used — **unknown**.

### 3.2 RMBG segmentation and the green sheet

- **Known [E, RMBG-1.4; RMBG-2.0]:** both models resize input to 1024 × 1024 and output a learned soft mask. Both are non-commercial by default.
- **Inferences [I]:**
  - ≈ 1.2 mm per mask pixel over 1.2 m, before upsampling, with a *learned* edge rather than a metrological one.
  - A **green sheet** suggests colour-based (chroma) separation was intended, perhaps alongside RMBG; the reason is unknown. Chroma separation fails on **green garments**, and green light reflected onto rounded fabric edges can shift apparent edge gradients.
  - The training images had **white** backgrounds, so the real green background was a domain shift for anything the model learned about garment boundaries.
- **Physics alternative:** a **backlit** table produces a colour-independent silhouette with a physical threshold (document 04 §3.1). A commercial garment station uses one [E-vendor, Seamline].
- **Status:** plausible contributor to edge error at the ~1 mm level **[H]**. It cannot explain centimetre-level errors on its own [I].

### 3.3 Keypoint detection and "correction algorithms"

- **Known about typical architectures [E, HRNet; DARK]:** fixed input crops (e.g. 256 × 192 or 384 × 288), ¼-resolution heatmaps, and decoding choices that materially affect accuracy.
- **Inferences [I]:**
  - Over a whole garment, one heatmap cell spans ≈ 12–19 mm.
  - Keypoints discard the continuous contour information that geometric fitting uses.
  - Learned localisers plateau at the label spread and have heavy error tails [E analogues, Wang2016-ISBI; Lips2024].
  - "Correction algorithms" that move points toward expected positions act as **priors**. That can hide real defects (shrinkage toward nominal), **if** they worked that way.
- **Status:** the most likely *mechanism* for "slightly displaced points" given the stated pipeline **[H]**. The size of the correction heuristics' effect cannot be judged without their code.

### 3.4 Synthetic Gemini / Nano Banana training images — can plausible images train a metrology model?

**Short answer: no, not for the metric role. Possibly, in part, for the semantic role [I].**

1. **No physical referent.** A generated image depicts no real garment. There is nothing to measure, no pattern, no seam geometry. Any keypoint label on it records **an annotator's visual interpretation**. The model therefore learns to reproduce that interpretation — including its drift from the contractual POM definition — not a physical feature defined by an operator [I].
2. **Resolution limits label precision.** Gemini 2.5 Flash Image generates at ~1024 × 1024 natively [E-sec, NanoBanana-res]. If a garment spans ~1000 px and is taken to represent 0.8–1.2 m, one pixel ≈ 0.8–1.2 mm. A 2–5 px annotation error is **≈ 1.6–6 mm**, with no way of checking it [I].
3. **Geometric fidelity is not guaranteed.** Current generators "cannot reliably reproduce geometric properties of real images" [E, Sarkar2024]. The generators tested were SDXL, Kandinsky, DeepFloyd, PixArt, DALL·E 3 and Firefly on indoor and outdoor scenes — *not* Gemini, and not garment flat-lays. The finding is a warning, not a measurement for this model. For garments the relevant geometry is sewing geometry (seam topology, proportions, fold placement), which no generator is constrained to respect [I].
4. **Domain gap.** White generated backgrounds vs a green physical sheet; generated textures and lighting vs real fabrics, sensor noise and optics; generated "flat" garments vs real residual wrinkles and seam roll [I].
5. **Even physically based synthetic data falls short.** Cloth keypoint detectors trained on procedurally generated, physics-deformed meshes **with exact labels** still reached only centimetre-level average errors and had a heavier tail than real-data training. Real-data fine-tuning roughly halved the error [E, Lips2024].
6. **What synthetic data *can* do:**
   - pre-train **semantic** detectors (garment type, part regions);
   - provide augmentation diversity;
   - if **physically based**, with labels defined on pattern geometry (CLO3D, Blender) — label definitions become exact by construction, even though geometry fidelity is limited (deformation differences up to ~6 % vs real prototypes [E abstract, MahnicNaglic2025]).

   The **final coordinate must come from full-resolution geometry on real images, validated against physical marks** (document 07) [I].

### 3.5 "80 % accuracy" and "slightly displaced"

- Under the interpretations that can be computed (80 % within ±t, 80 % within ±t/2, 80 % of garments with all POMs within ±t), the implied per-measurement error SD is ≈ 0.4–0.8 of the half-tolerance: ~5–10 mm for ±1/2 in POMs [I, document 01 §10].
- If the reference was a manual tape, part of that is **reference error**: a tape with u ≈ 3 mm alone produces ≈ 3 mm of apparent disagreement [I].
- The independent review's conclusions that the error was "on the order of centimeters", or that 80 % was an "asymptotic ceiling", **cannot be supported** from the information available.

---

## 4. The central retrospective lesson [I]

The previous pipeline, as described, did three things:

1. **Treated a semantic-recognition toolchain as a measuring instrument.** The keypoint *was* the endpoint.
2. **Trained on labels with no physical truth**, in a different visual domain from deployment.
3. **Left garment state, POM definitions and the reference unmeasured**, so no error could be attributed.

The difficulty they reported is the predictable outcome of that combination. It does not show that automated garment measurement is infeasible, nor that their ~80 % is a ceiling.

---

## 5. Hypotheses and discriminating tests

| # | Hypothesis about the previous system | Test (document 07 stage) | What would clear the component |
|---|---|---|---|
| H1 | Coarse learned localisation dominated the error | S5 vs S6: their model's points vs expert clicks on the same full-resolution images and vs UV marks | Model error ≈ expert-on-image spread |
| H2 | Labels encoded a definition different from the physical POM | S6: per-POM **mean offset** of their model vs physical marks (bias, not scatter) | No systematic per-POM offsets |
| H3 | Domain gap (white synthetic → green real) inflated errors and gross failures | S6 stratified by background (white / green / backlit) and fabric colour | Similar error across strata |
| H4 | Segmentation edges (1024 px, green spill) biased silhouette-based POMs | S2/S6: RMBG edges vs backlit silhouettes and marked edges on the same lay | RMBG edge bias ≪ 0.5 mm |
| H5 | Zivid scale or geometry use contributed | S1: Zivid vs a 2D camera on certified artefacts; check their distance computation (planar vs surface path) | In-plane scale error ≪ 0.5 mm; no surface-path summation |
| H6 | Garment state variability was large and uncontrolled | S3: re-lay study with physical marks, no AI | Re-lay SD ≪ target for their fabrics |
| H7 | The reference (manual tape) was noisy | S7: manual-method MSA on the same garments | Manual reproducibility ≪ system error |
| H8 | Correction heuristics pulled points toward nominal | S6/S7 with seeded defects: slope of measured vs reference deviation, with and without correction | Slope ≈ 1 |

---

## 6. What to salvage

- **The Zivid** → the height channel of the new pilot station (document 04 §3.2).
- **Their trained model** → a baseline in S6. Its errors against physical marks quantify H1–H3 directly.
- **Their real captures, if any exist with garments available** → re-measure the garments with marks and re-label the images against physical definitions. This could become the first real training and validation data.
- **Their synthetic images** → at most, semantic pre-training.

---

## Sources cited in this document

See [references.md](references.md): Zivid2+L110, RMBG-1.4, RMBG-2.0, Seamline, HRNet, DARK, Wang2016-ISBI, Lips2024, NanoBanana-res, Sarkar2024, MahnicNaglic2025.
