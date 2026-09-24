# 08 — Feasibility assessment

*Consolidated research, v2. This is the conclusion. The argument and the evidence are in documents 01–07. Labels: [E] established · [I] inference · [H] hypothesis.*

---

## The verdict

**This is an industrial metrology problem, not an AI keypoint-detection problem.** Machine learning has a real but bounded role: recognising *which* feature is which, and reducing the operator's work. The dimensional result must come from:
- a **specified garment state**;
- **completely defined** points of measure;
- **calibrated geometry** on physically observable features, validated against a reference that is better than the system.

**Reliable automated measurement looks technically credible for a defined subset of POMs, under an engineered measurement station.**
- **Body lengths and widths** defined by edges, folds and visible seams are the core of that subset.
- **Knitted garments and tight-tolerance POMs are uncertain.** The deciding quantity — how reproducibly a garment can be brought into the measured state — has never been measured.
- **Force-defined POMs cannot be measured passively.**
- **The business case** for a stand-alone audit station rests mainly on consistency and traceability, with roughly a 1.2–2× labour gain. A large gain in coverage needs capture embedded in a step where garments are already laid flat, and whether the target factory has one is unknown.

[I/H — to be confirmed by the gated programme in document 07]

The original formulation made the problem harder than necessary: inferring millimetre geometry from unconstrained images with keypoints trained on generated pictures. **Engineering the measurement environment and the measurand definitions converts most of it into a tractable measurement task** [I].

---

## 1. The four questions

### Q1 — What exactly needs to be measured?

- Each POM as a **complete measurement operator**: feature, datum, direction, filtering, construction. It is cross-referenced to ISO 18890 and agreed with the buyer, and applies in a **specified garment state**.
- The output is conformity decisions **per garment** (any POM out of tolerance fails the garment) plus measurement data for process control.
- **Targets (provisional):**
  - body POMs at ±1/4–±1/2 in: combined uncertainty ≈ 0.9–1.9 mm (Q_MP ≤ 30 %), **and** garment-level false-reject and false-accept rates within buyer limits at the real production distribution;
  - small details (±1/8 in, −0/+1/8 in): a separate method;
  - force-defined POMs: a fixture with a specified force.
- **The classical automotive 10 % %GRR criterion is unrealistic for textiles by any method, including the tape** [H, strongly indicated]. (Document 01.)

### Q2 — What physical information is required?

For each endpoint, one of the following:
- a **silhouette edge** — cut edge or fold, best seen by backlight;
- a **seam line** — best seen by multi-azimuth raking light and height relief;
- an **intersection of fitted seams** — for junctions such as the side neck point or underarm point;
- a **construction** from a defined datum.

In addition:
- **Height** at thick features (collars, waistbands, hems), to about 1 mm, to remove perspective errors of 1–4 mm.
- **Evidence of the garment's state**: flatness, and possibly stitch-density strain on knits [H].
- **Definitions** for endpoints hidden under layers. The side neck point is defined *excluding* the collar, so it sits under it.

(Document 03.)

### Q3 — Which architecture can acquire it with low enough uncertainty?

A **three-layer engineered station** (document 04, configuration C1 → C2):
1. **State layer:** standard surface with a fiducial frame; guides and lay rules; a **float → settle → hold** relaxation surface to test; tension fixtures only where the specification calls for a restrained state.
2. **Semantic layer:** ML proposals trained on real, physically labelled images. The **operator confirms** initially (an industrial precedent exists), and confirmation becomes automatic, with abstention, as validation allows.
3. **Metric layer:** a high-resolution 2D camera with backlight, front and raking light; **the existing Zivid reused as the height channel**; complete operators executed at full resolution; audited priors.

This architecture is supported by physics and by what commercial stations already do [E-vendor]. **Its uncertainty on real garments is not known.** Gates G1–G3 establish it.

### Q4 — Can it operate economically and rapidly enough?

- **Handling time, not sensing, sets throughput.** A dedicated station where the operator lays each garment plausibly gives about **1.2–2× the labour productivity** of the 45–60 s manual method; one vendor claims "more than half" the work time saved [E-vendor]. Its value is mainly consistency, digital records and the removal of transcription.
- **Order-of-magnitude coverage (Tier A sentinels on 100 % of output)** requires capture embedded in an existing flat-handling step, or solved automatic feeding. The first is unknown for the target factory. The second is unsolved.
- **A €15k 3D camera is not needed as the metric sensor.** Spending on definitions, state control, calibration artefacts and a validation reference is more likely to reduce uncertainty.

(Document 05.)

---

## 2. Feasibility by POM class and scenario

| POM class (document 01 §2.2) | S-AQL, wovens (engineered station) | S-AQL, knits | S-Inline (quick lay at an existing step) |
|---|---|---|---|
| **P1 Edge-to-edge widths** (hem, chest at a defined datum, leg opening) | **Likely feasible** [H] | **Uncertain** — depends on u_B under a relaxation protocol (S3) | **Plausible for 1–2 sentinel widths** if the quick lay passes S3 [H] |
| **P2 Seam-anchored lengths** (body length, sleeve, inseam) | **Likely feasible** where the junction is visible or defined [H]; the side neck point under a collar depends on definition (S4) | Uncertain (state + definition) | **Plausible for body length** as a sentinel [H] |
| **P3 Seam-to-seam / edge-to-edge "as agreed"** | Feasible **after** the operator is agreed; S4 decides | Uncertain | Not a sentinel candidate |
| **P4 Curves and diagonals** (armhole, raglan) | Uncertain (landmark definition; bias-direction strain) | Uncertain | No |
| **P5 Force-defined** (extended waist or neck) | **Not feasible passively.** Feasible only with a force-controlled fixture and an agreed force | Same | No |
| **P6 Small details** (−0/+1/8 in pockets, label placement) | **Separate method** (close-range optics, templates) or manual | Same | No |
| **P7 Hidden / multi-layer** (closed-collar neckband, linings) | **Not from a top view.** Manual or special fixtures | Same | No |

"Likely feasible" means that physics and analogues give no reason to expect failure **and** the uncertainty budget closes with plausible values. **It is not a measured result.**

---

## 3. What remains uncertain — and the experiment that settles each

| Rank | Unknown | Why it decides the outcome | Experiment |
|---|---|---|---|
| 1 | **Garment-state reproducibility u_B**, especially for knits, per protocol (careful, quick, float/hold, restraint) | Friction lock-in and hysteresis could be ~0.1–0.2 mm (wovens) or several mm (compliant knits); buckles bias lengths short | S3 (no ML) |
| 2 | **Definitional spread u_A** per POM class | A floor for any method; decides which POMs need rewriting | S4 (no ML) |
| 3 | **Existence and lay quality of an embedding point** in the factory | Decides whether S-Inline coverage is achievable | S0 process mapping + S3 quick-lay arm |
| 4 | **Production distributions** (C_p, centring, correlations) | Decide garment-level false-reject and false-accept rates for a given uncertainty | S0/S7 |
| 5 | **Observability** of junctions under layers, and of dark-on-dark seams | Decides whether modality changes or marks are needed | S5 |
| 6 | **Automation performance on unseen styles** (tails, coverage) | Decides C2 vs C1 | S6 |
| 7 | **Manual-method reproducibility** | The realistic bar, and the equivalence target | S7 (manual MSA) |
| 8 | **Moisture and relaxation drift** under factory conditions | Possible hidden bias between measurement times | S3 time/RH arm |

---

## 4. Proceed / do-not-proceed

**Proceed to full engineering** only if all of the following hold (document 07 §6):
- **G1** sensing: u_{C+D} ≲ 0.3 mm on body-POM edges; scale verified in the measurement plane.
- **G2** physics and definition: √(u_A² + u_B²) ≤ 0.8 × u_target for the buyer-critical POMs under an acceptable protocol — **the decisive gate**.
- **G3** automation: acceptable errors, gross-error rate and coverage, or operator confirmation that is economically acceptable.
- **G4** system: garment-level decision risks within buyer limits, not worse than manual, equivalence or agreed bias, and a positive value model at measured handling times.

**Stop or restrict scope** if:
- G2 fails for core body POMs under every practicable protocol. Then redefine POMs or restraint standards with buyers, or position the system as a consistency and traceability tool;
- or handling cannot be brought well below 45–60 s and no embedding point exists;
- or key endpoints are unobservable and production marking is unacceptable.

**Phases 0–1 (weeks to a few months, no machine learning)** settle G1 and G2, and so most of the feasibility question, at small cost.

---

## 5. What would change these conclusions

- **More optimistic.** If S3/S4 show u_A and u_B well below 0.5 mm for most body POMs, including knits under a simple relaxation protocol, near-classical capability becomes plausible. S-Inline then becomes a handling-engineering problem.
- **More pessimistic.** If knits vary by several millimetres under every practicable protocol, or experts disagree by several millimetres on core POMs after the operators are rewritten, then ±1/4 in automation is out of reach and ±1/2 in becomes marginal for those garments.
- **A different bottleneck.** If S5 shows key features are unobservable, design-for-measurability (production marks) becomes the main research direction.

---

*Sources: see documents 01–07 and [references.md](references.md). Differences from both earlier research bodies are recorded in [09-reconciliation-record.md](09-reconciliation-record.md).*
