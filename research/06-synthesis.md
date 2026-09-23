# 06 — Synthesis: what is feasible, under what conditions, and what to test first

*Part of the garment-measurement feasibility study. This is the concise conclusion. The supporting argument and sources are in documents 01–05; evidence labels are as defined there ([E] established, [I] inference, [H] hypothesis).*

---

## The short answer

**Reliable automatic garment measurement for industrial QC looks technically credible — but only as a metrology system with a specified garment state and completely specified points of measure, not as a computer-vision system that "finds keypoints".**

The most likely limits are not in the camera or the neural network. They are in:
1. how precisely each point of measure is **defined**;
2. how **reproducibly** a garment can be brought into the measured state.

Both are unmeasured today, and both can be measured cheaply — without any machine learning — before a large engineering commitment. [I/H]

---

## 1. What appears technically feasible

| Capability | Confidence | Basis |
|---|---|---|
| Making the **sensing chain** (camera, optics, calibration, edges) accurate to a few tenths of a millimetre over a garment-sized flat field | **High** | Backlit 2D gauges reach ±35 µm over a 1.8 × 1.2 m table on flat stiff parts. Monocular sub-pixel vision reaches ~10 µm on rigid parts. Pixel sampling of ≈ 0.13–0.2 mm is cheap. [E, InspecVision; Nogueira2023; I, document 03 §2] |
| Measuring distances between **physically well-defined features** (hem and cuff edges, fold edges, clearly visible seams, marked points) to about a millimetre on a carefully laid garment | **Moderate — plausible, unproven** | Deformable objects with unambiguous endpoints are measured to ~1 mm in other industries (fish on conveyors, SD 1.2 mm); the physics of edges and folds is sub-millimetre to millimetre [E, CatchMeter; I, document 03 §3–4; H for garments] |
| **Identifying** which feature is which POM endpoint (semantics) automatically, with abstention on hard cases | **Moderate** | Learned models are good at semantics but not at millimetre metrology. Split the roles: ML proposes, geometry measures, low-quality cases abstain [I, documents 02 §4, 03 §5] |
| A deployable **S-AQL station** in which a human confirms machine-proposed endpoints and the machine does the geometry | **Moderate–high** | Removes tape handling and transcription error, keeps human semantic judgement, and generates physically grounded training data [I/H, document 04 A7] |
| Meeting classical automotive MSA thresholds (%GRR ≤ 10 %) for body dimensions | **Low for any method, including the manual tape** | That would need σ ≈ 0.2–0.4 mm on a material that deforms by millimetres under handling [I, document 01 §8]. The frameworks' own scope assumes definitional uncertainty is negligible, which is unlikely here [E, ISO22514-7] |
| Measuring **force-defined** POMs (waist or neck "extended") passively | **Not feasible without actuation** | Not observable in an image; needs a fixture with a specified force [E, ISO18890; I] |
| Full **S-Inline** automation (100 % measurement with automated handling) | **Low near term** | Handling must deliver a reproducible, specified state at line speed. Robotic cloth flattening research is at "coverage" level, with centimetre keypoints [E, FlingBot; Lips2024] |

---

## 2. Under what conditions

Feasibility, where it exists, is conditional on all of the following:

1. **Complete POM definitions.** Each POM is written as a machine-executable operator in the GPS sense: which feature (seam vs fold), which direction datum, which filtering, which construction [E concept, ISO17450-2; Nielsen-CIRP]. Definitions are agreed with buyers and cross-referenced to ISO 18890. ISO 18890 itself leaves several choices "as agreed by interested parties" [E, ISO18890].
2. **A specified, reproducible garment state.** A written lay protocol per scenario, a flatness check with re-lay or abstention, and — where the protocol alone is insufficient — a restraining device such as a vacuum table or platen, with its bias relative to the free state measured and agreed. This is the ISO 10579 principle for non-rigid parts [E, ISO10579].
3. **Metrology-grade sensing,** verified with traceable artefacts in the measurement plane. Camera height and height correction handle perspective (garment thickness and seams cause millimetre-level errors at typical heights). Lighting is chosen for edge and seam definition — backlight for silhouettes, raking light for seams [I, document 03 §2].
4. **A three-layer pipeline:** state layer → semantic layer → metric layer. The learned component never supplies the final coordinate. Every measurement carries a quality metric that drives abstention [I, document 04 §0].
5. **Validation against a reference that is better than the system,** on the same lay, with marked points and a measured definitional spread. The report covers bias, repeatability, reproducibility, percentiles, the gross-error rate, abstention and decision risk — never a single "accuracy %" [E framework: JCGM106, VIM-2.27; I].
6. **Priors audited for shrinkage toward nominal** with seeded out-of-tolerance garments [I, document 03 §6].

---

## 3. What remains uncertain (and determines the answer)

| Unknown | Why it matters | Where it gets measured |
|---|---|---|
| **Definitional spread** of POM endpoints among trained experts on a fixed lay, per POM class | Sets the floor for *any* method; decides which POMs need rewriting or excluding | S4 |
| **Garment-state reproducibility** per fabric class (knits vs wovens vs denim) and per protocol or fixture | Most likely to break ±1/4 in POMs; decides whether fixtures are necessary | S3 |
| **Observability** of seam junctions (SNP under collars, underarm point at the fold, crotch point) in each sensing modality | Decides whether a modality change or physical marks are needed | S5 |
| **Reproducibility of the current manual method** | Sets the realistic bar; buyers' tolerances were calibrated against it | S7 |
| **Production spread relative to tolerance** | Determines the actual false-accept and false-reject rates for a given uncertainty | S3/S7 |
| Automated localisation performance on **unseen styles**, and its heavy tail | Decides full automation vs human-in-the-loop | S6 |
| Moisture and relaxation effects under factory conditions | Possible hidden drift | S3 arms |

---

## 4. Where the true bottlenecks lie (ranked hypotheses)

1. **Definition of soft landmarks** — seam junctions, constructed "x cm below" points, points under collars. This is a specification problem; analogous fields show experts disagree by 1–2 mm and algorithms plateau there [E analogues: Kouchi2011; Wang2016-ISBI].
2. **Garment state** — handling strain retained by fabric hysteresis (the table can hold only ≲ 1 gf/cm by friction), wrinkles that always bias lengths *short*, seam roll (a 3 mm roll on both sides changes a width by 6 mm). This is a physical process problem [I, document 03 §3].
3. **Semantic localisation on hard cases** — dark-on-dark seams, fleece, prints, occlusion. An algorithmic problem, largely containable by abstention [I].
4. **Sensing** — solved in adjacent fields; its traps are perspective/height, calibration verification and lighting, not resolution [E/I].

The order of 1 and 2 may differ between knits and wovens; that is what S3/S4 will show.

---

## 5. What this means for the previous system

- **"80 % accuracy" is not comparable with anything.** Plausible readings imply per-measurement error SDs of roughly 0.4–0.8 of the half-tolerance, i.e. ~5–10 mm for ±1/2 in POMs (document 01 §10).
- The "slightly displaced reference points" are what one would predict from coarse-resolution keypoint architectures, text-to-image training labels without geometric truth, a white-to-green domain gap, and uncontrolled lay and reference error. Each is **a testable hypothesis, not a finding** (document 03 §9).
- **Their result is not a ceiling.** Their architecture attacked the component least likely to be limiting (sensing, with an expensive 3D camera) and left the likely limiting components (definition, state, reference) unmeasured [I/H].

---

## 6. Is a €15k-class setup necessary?

**Probably not for flat-lay spec measurement — but price is not the real issue.** [H, to be confirmed in S1]

- The structured-light camera's colour image is 5 MP (≈ 0.44 mm/px over its 1.09 m field). Its datasheet dimension trueness is < 0.35–0.60 % at the 70th percentile (≈ 2.5–4 mm over 700 mm) unless re-verified. Naive surface-path lengths from its depth data are strongly inflated by noise [E, Zivid2+L110; I, document 03 §2].
- A calibrated 45–61 MP 2D camera with backlight and raking light (hardware in the low thousands of euros) plausibly matches or beats it for planar dimensions. The 3D camera's genuinely useful functions — height correction and flatness detection — can likely be obtained more cheaply [H].
- Money is better spent first on **definitions, a state protocol and fixtures, calibration artefacts and a validation reference**. The existing Zivid should be kept as a **research instrument** for measuring garment state in the feasibility study.
- The dominant cost is development risk, not station hardware. That is why the physical feasibility envelope (gates G1–G2) should be established before any further AI investment.

---

## 7. Experiments to run before committing to full-scale engineering (in order)

| Step | What | Uses ML? | Answers |
|---|---|---|---|
| 1 | **S0** Write complete POM operators and lay protocols; pre-register metrics and gates; recruit ≥ 5 experts and ≥ 5 operators | No | Makes every later number interpretable |
| 2 | **S1–S2** Rigid artefacts and marked textiles: head-to-head of a high-resolution 2D camera (front, raking, backlight) vs the existing Zivid; perspective and step-block tests; UV-ink visibility check | No | Gate G1 (sensing); the €15k question at sensor level |
| 3 | **S3** Re-lay study on UV/micro-dot-marked production garments across the three families, S-AQL vs S-Inline protocols, plain vs vacuum/platen arms, conditioned vs not | No | u_B per fabric and POM class; value of fixtures; time per lay |
| 4 | **S4** Independent blind expert marking (UV ink, black light) and on-image clicking on fixed lays; alternative definitions for ambiguous POMs | No | u_A per POM; which POMs to rewrite or exclude → **Gate G2, the decisive gate** |
| 5 | **S5** Experts on clean images vs physical marks, per modality | No | Observability: sensing vs information limits |
| 6 | **S6** Automated pipelines (the previous model as a baseline, a hybrid semantic + geometric pipeline, human-in-the-loop) vs marks, on held-out styles | Yes | Gate G3: AI error, tails, coverage |
| 7 | **S7** End-to-end crossed study vs the manual tape method and the reference, including seeded defects and decision-risk analysis | Yes | Gate G4: the industrial answer and the economic inputs |

**Steps 1–5 need no machine learning** and mostly reuse inexpensive equipment. They are measured in weeks, not years [I]. They will reveal whether ±1/2 in and ±1/4 in POMs are physically attainable, and under which protocol. That is the information that was missing when the previous two-year effort started.

---

## 8. What would change these conclusions

- **More optimistic:** if S3/S4 show definitional and state components well below 0.5 mm for most body POMs without fixtures. Then a straightforward A1/A2 station with modest ML could approach classical MSA criteria, and S-Inline becomes a handling-engineering problem.
- **More pessimistic:** if experts disagree by several millimetres on core POMs even after the operators are rewritten, or re-lays of knits vary by several millimetres under every practicable protocol. Then no sensing architecture can meet ±1/4 in tolerances. The path forward would be redefining POMs with buyers (e.g. seam-anchored definitions, restrained states), restricting automation to robust POMs, and positioning the system as a consistency and traceability tool rather than a tolerance authority. The most candid current vendor already positions its product this way [E, SizeAI-QC].
- **Different bottleneck:** if S5 shows that key features are not observable in any practical modality. Then design-for-measurability (marks applied in production, A9-P) becomes the main research direction.

---

*Sources: see documents 01–05 and [references.md](references.md).*
