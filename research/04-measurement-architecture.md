# 04 — Which architecture can acquire the information with low enough uncertainty?

*Consolidated research, v2. This document answers **Question 3**. Evidence labels: [E] source opened · [E-sec] secondary · [E-vendor] manufacturer claim · [I] inference · [H] hypothesis.*

**Ground rules**
- No uncertainty, cycle-time or cost figure appears here unless it is cited or derived.
- Earlier drafts, in both research bodies, attached performance ranges to architectures. The independent review, for instance, gave "0.7–1.2 mm" for stereo and "0.6–1.0 mm" for tension fixtures. No experiment supports those figures, so they have been removed. Unknowns are listed as unknowns and routed to document 07.

---

## 1. Should the factory engineer the measurement environment? Yes

Document 03 showed that the likely dominant error components — garment state, definitional spread and observability — are **not reduced by better inference from unconstrained images**. They are reduced by:
- **specifying** the measurand (complete operators);
- **controlling** the state (surface, lay protocol, relaxation, restraint);
- **choosing the physics of observation** (backlight for silhouettes, raking light for seams, a height channel for relief).

This is the ISO 10579 idea applied to textiles. Commercial stations already work this way: a dedicated table, a fixed camera, sometimes a backlight, and operator adjustment of points [E-vendor, Toyoshima; Seamline].

**The problem is substantially easier if garments are placed on a standardised surface, in a defined orientation, under consistent illumination, with a defined relaxation or restraint procedure.** Inferring "perfect geometry" from unconstrained garment photographs attacks the problem at its hardest point [I].

The limit is practical. Every constraint costs handling time (document 05), and restraints change the measurand, so they must be agreed with buyers (document 01 §3.4).

---

## 2. Layer 1 — Physical state control: options and physics

| Option | What it does physically | What it should fix | What it can break | Evidence | Verdict |
|---|---|---|---|---|---|
| **Standard surface + fiducial frame** (matte contrasting or backlit top; certified targets around the field) | Fixed geometry and scale reference in every image | Calibration drift, background ambiguity | — | Standard in optical gauging [E-vendor, InspecVision] | **Baseline — always** |
| **Orientation guides / registration** (printed or projected alignment lines: CF on a line, shoulder line on a mark, sleeves at a set angle) | Reduces variability in pose and orientation | Pose-dependent perspective, sleeve angle, datum alignment | Operator time | Practice-level [I] | **Baseline** |
| **Lay rules** (e.g. "roll side seams to the fold", "smooth along the grain, not diagonally", "no stroke across the bias") | Reduces seam roll and directional strain | Seam roll (2r width error), anisotropic strain (document 03 §6.1) | Time | [I] | **Baseline; effect measured in S3** |
| **Float → settle → hold surface** (air-flotation, then low hold-down) | Air through the table lets trapped buckles slide out; the garment then settles and is held | Buckles (trapped excess length), friction lock-in on knits | Light fabrics may drift; exact settle behaviour unknown | Air-flotation tables are standard for moving lays in cutting rooms [E-vendor, AirFlotation]. The physics is in document 03 §6.3 [I] | **Promising; prototype and test [H]** |
| **Vacuum hold-down alone** | Pulls the garment onto the table | Out-of-plane relief, curled edges, stability during capture | Cannot remove buckles (they become pleats); raises friction, which locks in any strain; can dimple over coarse holes; uncovered holes leak | Cutting rooms use air-impermeable overlay film to reach and keep *deep* vacuum on multi-ply stacks [E-sec, OverlayFilm]. A single garment needs far less. At face velocities of 0.01–0.05 m/s, estimated ΔP is 20–330 Pa on poplin or denim — **8–136× the fabric's self-weight** — but only ~2–10 Pa on open knits [I, Darcy scaling from ISO 9237 values, E-sec AirPerm] | **Not rejected, not proven.** Useful for hold-down after relaxation. Needs zoning or masking and fine perforation |
| **Glass / acrylic platen** | Presses the garment flat | Relief (perspective), curled edges | Traps buckles as pleats; friction on landing; reflections; handling time | [I] | **Test in S3 as an arm; likely niche** |
| **Tension fixtures** (spreader bars or clamps with a specified force) | Creates a **specified restrained state** | Force-defined POMs (waist or neck extended); possibly knit body widths in a defined restrained state | **Changes the measurand.** It must be written into the specification and agreed; needs fixture design per garment type | ISO 10579 restrained-condition concept [E]; the independent review proposed 0.5–1.0 N spreaders (no evidence of performance) | **Required for P5 POMs; optional elsewhere by agreement** |
| **Air-knife edge smoothing** | Air flow along the surface flattens hems and ripples | Curled hems | Fluttering, lifting light fabrics | None found | **Speculative [H]** |
| **Conditioning / timing rules** | Hang or rest for a defined time; measure at a defined time after pressing | Relaxation and moisture drift | Floor space, time | ISO 3759 conditioning [E]; magnitude for garments unknown [H] | **Decide after S3 measures drift** |

**Key physical insight [I].** Removing wrinkles needs **in-plane relaxation** (low friction or gentle tension). Pressing down does not remove trapped length. The candidate physical-control stack is therefore **guides + lay rules + (float → settle → hold)**, with tension fixtures only where the specification calls for a restrained state.

---

## 3. Layers 2 and 3 — Sensing

### 3.1 Metric camera (planar measurement)

- **A high-resolution 2D camera** (24–61 MP, fixed focus and aperture, locked settings). At 1.2 m this gives ≈ 0.13–0.2 mm per pixel [I].
  - Mount it high enough to limit perspective: at H ≈ 2 m, a 3 mm hem shifts by ≈ 0.9–1.2 mm at 0.6–0.8 m from the image centre, and more at the far corners [I, document 03 §4.2]. A height channel corrects the rest.
  - Verify scale and distortion **in every image** against the fiducial frame.
- **Multi-mode illumination, captured sequentially:**
  - **Backlight** gives a colour- and print-independent silhouette. A commercial garment workbench uses a backlit table [E-vendor, Seamline].
    - Limits: it shows the *union* of layers, so shaped hems need front light; seam allowances show through only in thin, light fabrics; edge position on translucent or hairy edges must be characterised (S2).
    - The independent review's claims that backlighting "fails on > 60 % of styles" and shifts edges by 0.3–0.8 mm have no supporting source. Opaque fabrics give the *best* silhouettes; what they don't give is internal seams.
  - **Diffuse front light** — cross-polarised for glossy synthetics [I] — for colour and texture.
  - **Multi-azimuth raking light** (low-angle, several directions) makes seam ridges and grooves visible regardless of thread colour [I], and supports photometric flatness cues.

### 3.2 Height channel (for perspective correction and flatness detection)

About **1 mm height accuracy** keeps the parallax residual to ~0.3–0.4 mm at H = 2 m (document 03 §4.2) [I]. Options:

| Option | Notes |
|---|---|
| **Reuse the existing Zivid as the height channel** | Point precision 260 µm [E, Zivid2+L110] is more than adequate for height. Registered to the metric camera, it salvages the earlier investment. Its trueness spec does not matter for height *correction* at this level [I] |
| **Active stereo** (two cameras + projected texture) | σ_z ≈ 0.2–0.35 mm in plausible geometries [I]. **Passive** stereo fails on textureless fabrics [E-sec, Hirschmuller2008; Konolige2010] |
| **Laser light-section or photometric stereo** | Cheap relief and flatness cues [H] |

**Structured-light 3D as the *primary metric* sensor is not recommended** for flat-lay POMs:
- its 2D sampling is coarse (0.44 mm/px);
- its dimension trueness spec is ≈ 2.5–4 mm over 700 mm unless re-verified;
- surface-path lengths inflate with depth noise;
- it does not address components A or B [E, Zivid2+L110; I].

### 3.3 Scanning acquisition (conveyor)

A **line-scan camera** gives orthographic geometry along the conveyor and fine sampling. Scale along the belt depends on encoder and speed accuracy [I]. **CIS arrays are unsuitable:** their ≈ 0.2–0.3 mm depth of field cannot hold a garment with millimetres of relief in focus [E-sec, CIS-DOF]. The unsolved part of conveyor measurement is not the sensor but **getting garments onto the belt flat and in a defined state** (document 05).

---

## 4. Layer 2 — Semantic identification

- **ML proposals.** Garment type, part identity, approximate regions for each POM's defining features, and topology checks.
  - Train on **real production images labelled against physical marks** (document 07). The human-in-the-loop station below produces such labels as a by-product.
  - Use **physically based** synthetic data with exact labels (CLO3D or Blender with pattern correspondences) for pre-training, never as the metric truth [E, Lips2024; MahnicNaglic2025; I].
- **Human confirmation.** The operator confirms or adjusts proposals on a touch display; the machine snaps to the nearest physical feature at full resolution. A commercial precedent exists [E-vendor, Toyoshima].
- **Quality and abstention.** Per POM:
  - edge contrast, fit residuals, agreement between cues (backlight vs front light vs height), topology plausibility → a quality score;
  - calibrated uncertainty, e.g. conformal prediction for localisers [E, Jonkers2025];
  - a **risk–coverage** operating point [E, GeifmanElYaniv2017].

## 5. Layer 3 — Metric evaluation

1. Execute the **complete POM operator** (document 01 §3.3): partition from the proposals → sub-pixel extraction at full resolution → filtration → association → construction → evaluation.
2. **Height correction** of every endpoint from the height channel.
3. **State checks:**
   - flatness (height and raking cues) → re-lay or abstain;
   - optionally, stitch-density strain maps on knits [H, document 03 §7].
4. **Prior audit.** No template or symmetry "correction" moves a measured endpoint. Templates are used only to identify features and check plausibility.
5. **Decision rule** agreed with the buyer (simple or guarded; re-measure-before-reject near the limits; document 01 §9).

---

## 6. Candidate system configurations

| ID | Configuration | Scenario | Components attacked | Main unknowns | Hardware cost anchor |
|---|---|---|---|---|---|
| **C0** | Manual tape (A0) / digital Bluetooth tape (A0+) [E-vendor, DigitalTapes] | Baseline | Transcription only (A0+) | Its own reproducibility — never measured with an MSA design | Negligible |
| **C1** | **Collaborative audit station:** metric camera + backlight/front/raking light + height channel + guides + lay rules; ML proposes, **operator confirms**, geometry measures | S-AQL | C, D, E (gross errors near zero), partly B (protocol), A (only if operators are rewritten) | Operator confirmation time; u_B under the protocol; u_A of confirmed points | Commercial analogue quoted at US$5–7k per station [E-vendor, Seamline]. Camera example: 61 MP body at US$3,900 MSRP [E, Prices]. Height channel: reuse the Zivid, or active stereo [I] |
| **C2** | **Autonomous audit station:** C1 with ML proposals accepted automatically when quality is high; operator only for abstentions | S-AQL (faster) | As C1, plus fewer human touches | Automated gross-error rate at useful coverage (G3) | As C1 |
| **C3** | **Embedded sentinel:** metric camera (+ height channel) over an **existing** step where each garment is already laid flat (end-line inspection, folding); a few robust POMs (edge-to-edge widths and lengths); aggressive abstention | S-Inline | C, D, E; B only as well as the existing step allows | **Whether such a step exists and how flat and consistent its lays are** (unknown for the target factory); u_B under a quick lay | As C1, per embedding point [I] |
| **C4** | **Restrained-state station:** C1 on a float → settle → hold surface, plus tension fixtures for force-defined POMs | S-AQL (knits, tight POMs, P5) | B directly; A for P5 | Restraint bias vs the free state; buyer acceptance; cycle time | C1 + surface + fixtures (to be quoted) [I] |
| **C5** | **Conveyor line-scan:** flat-feeding conveyor under a line-scan camera + height channel | S-Inline (long-term) | C, D | **A flat, defined-state feeding method** (unsolved); encoder scale | Integration-dominated; not estimated |

**Not recommended [I]:**
- unconstrained photo measurement as a tolerance authority (vendors themselves position it as screening [E-vendor, SizeAI-QC]);
- raw 3D surface-path integration;
- CIS conveyors;
- passive stereo on plain fabrics;
- CAD or template "snapping" of measured points;
- keypoint heatmaps as final coordinates.

---

## 7. Which configurations to prototype first

1. **C1**, built so it can run the **C4 arms** (float/hold surface, tension fixture) and the **C3 quick-lay protocol**. It is the instrument for the validation programme (document 07), a plausible S-AQL product, and a labelled-data engine for C2.
2. **The height channel reuses the Zivid** in the pilot. That turns a sunk cost into a useful instrument and directly tests whether a cheaper height channel would suffice.
3. **C3 in the factory only after S0 process mapping** shows a suitable existing flat-handling step, **and** S3 shows that quick lays meet the sentinel POMs' uncertainty targets.
4. **C5** only if S-Inline value is proven with C3 and no embedding point exists.

---

## Sources cited in this document

See [references.md](references.md): Toyoshima, Seamline, InspecVision, AirFlotation, OverlayFilm, AirPerm, ISO10579, ISO3759, Zivid2+L110, Hirschmuller2008, Konolige2010, CIS-DOF, Lips2024, MahnicNaglic2025, Jonkers2025, GeifmanElYaniv2017, DigitalTapes, Prices, SizeAI-QC.
