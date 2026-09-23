# 05 — How to find out, before investing heavily: a feasibility and validation methodology

*Part of the garment-measurement feasibility study. Evidence labels: **[E]** established (a cited source that was opened and checked), **[I]** engineering inference, **[H]** hypothesis. Source keys resolve in [references.md](references.md).*

The earlier documents lead to one practical conclusion: whether garment measurement can meet industrial requirements depends mainly on **quantities nobody has measured**:
- the spread in where experts put POM endpoints on a fixed garment (definitional, A);
- the reproducibility of the garment's state from lay to lay (B);
- how much of the endpoint information is actually visible to a sensor (observability).

This document designs a staged experiment that measures them first and cheaply. It then measures the sensing chain, the automated localisation, and the full system against the manual method. **Each stage isolates one error component, so a failure can be attributed to sensing, to garment physics, to the definition, or to the AI model.**

---

## 1. Decision questions and gates

| # | Question the study must answer | Stage(s) | Why it matters |
|---|---|---|---|
| Q1 | How large are the definitional spread (A) and the state reproducibility (B), per POM class, fabric class and scenario? | S3, S4 | They set the floor for *any* architecture (document 03 §8) |
| Q2 | Can the sensing and edge chain (C+D) be made small relative to the target? | S1, S2 | Tells us whether hardware or optics are the problem |
| Q3 | Is the information needed to locate each endpoint present in the image? | S5 | Separates *sensing/observability* failure from *AI* failure |
| Q4 | How close does automated localisation get to that image ceiling, and how often does it fail grossly? | S6 | Isolates the AI/model component and the abstention trade-off |
| Q5 | End to end, how does the system compare with the manual method and the reference on real garments, and what decision risks result? | S7 | The industrial answer |
| Q6 | What does each architectural option buy (backlight, flatness sensing, vacuum or platen, 3D camera, human-in-the-loop)? | S1–S7 arms | Guides investment (document 04) |
| Q7 | What are the economic inputs (manual time, automated time, abstention rate, production spread)? | S3, S7 | Document 04 §4.3 |

**Gates.** The study stops or redirects at the first gate that fails, so money is not spent on AI for a problem that physics or definitions have already decided (§7).

---

## 2. Design principles

1. **A measurement ladder.** Each stage holds everything fixed except the component under study:
   - rigid artefacts → sensing only;
   - marked textiles → edges;
   - marked garments with re-lays → garment state;
   - independent expert marking on a fixed lay → definition;
   - experts clicking on images → information content;
   - models versus marks → the AI.
2. **Oracle substitution.** Replace one component with a perfect or physical version and see how much error disappears:
   - physical markers instead of learned keypoints;
   - a backlit silhouette instead of a learned mask;
   - a frozen lay instead of re-lays.

   This is how error attribution is done without guessing [I].
3. **Same lay for system and reference.** When comparing a system with the reference, both must measure the *same physical configuration*. Otherwise garment-state variability contaminates the accuracy estimate [I].
4. **Independent, blind marking.** Borrow the anthropometry technique: mark with **invisible (UV-fluorescent) ink under black light** so a second marker cannot see the first mark [E, Kouchi2011]. The same trick gives an image that is *clean* under white light (for humans and models) and *marked* under UV (for the reference) of the **same lay** [H — ink invisibility in the capture spectrum must be verified in S2].
5. **Pre-registration.** Fix before data collection:
   - the POM operators;
   - the lay protocols;
   - the metrics, strata, hold-out sets and go/no-go criteria.

   This prevents the "80 %" problem of undefined, post-hoc metrics (document 01 §10).
6. **A reference better than the system.** The reference's standard uncertainty for the quantity compared should be ≤ ¼ of the target uncertainty (a TUR-like 4:1 rule [E, JCGM106 §7.6]). Where that is not achievable, the reference uncertainty is modelled explicitly (errors-in-variables regression, TMU-style [E secondary, TMU]) rather than ignored.
7. **Hold-outs for learning.** Garments, styles and colourways used to evaluate a model must never appear in its training data. Report results separately for seen and unseen styles.

---

## 3. Reference instrumentation

| Ref | What | Role | Uncertainty considerations |
|---|---|---|---|
| **R1 Rigid artefacts** | Certified grid or dot plate(s) covering the field, or calibrated scale bars placed at several positions and orientations (the approach of VDI/VDE 2634 for optical systems [E secondary, VDI2634; Luhmann2010]); step blocks of known thickness (e.g. 1–10 mm) for height tests | S1 sensing verification; routine in-service checks | Calibration certificates (ideally from an ISO/IEC 17025 lab). Artefact uncertainty is negligible at the garment scale [I] |
| **R2 Certified rules and tapes** | Steel rules and tapes to OIML R 35 class I or II. The MPE is ±(a + bL) mm with L rounded up to whole metres: class I ±(0.1 + 0.1L), class II ±(0.3 + 0.2L), i.e. **±0.2 mm and ±0.5 mm up to 1 m** [E, OIML-R35] | Manual reference readings between *marked* points | The instrument is not the problem. Reading, alignment and handling dominate [I] |
| **R3 Marked-point optical reference** | UV-ink or micro-dot marks at POM endpoints; a high-resolution calibrated camera (the S1-verified system can serve) with scale bars in the garment plane; optionally two viewpoints or known heights to resolve mark heights | S3–S7 reference for marker-to-marker distances on the *same lay* as the system under test | Target: marker position uncertainty ≲ 0.2–0.3 mm [H — to be verified in S1/S2]. The marks turn a soft landmark into a hard one, so the *definition* is fixed when the expert places the mark |
| **R4 Conventional manual method** | Trained inspectors following ISO 18890 / buyer How-to-Measure with a tape [E, ISO18890] | S7 comparator: the method contracts refer to | Its reproducibility is measured, not assumed |

**What the reference does and does not certify [I].** R3 certifies distances between *marks*. The marks certify the *expert's interpretation* of the POM definition, whose spread S4 measures. The complete reference uncertainty for a POM is therefore u_ref² ≈ u_R3² + u_A²/n_experts when marks are consensus positions (e.g. the mean of several experts' marks).

---

## 4. Sampling frame (confirmed scope: knit tops, woven tops, bottoms)

| Factor | Levels (minimum proposal) | Purpose |
|---|---|---|
| Garment family and style | **Knit tops:** jersey T-shirt, piqué polo, fleece sweatshirt/hoodie. **Woven tops:** poplin shirt, soft blouse. **Bottoms:** rigid denim jeans, stretch denim, twill chino, knit shorts | Coverage of construction and fabric mechanics |
| Colour and surface | White/light, black/dark (dark-on-dark seams), mid-tone, **green** (trap for chroma-key pipelines), printed or striped, heathered | Edge and seam contrast; background interaction |
| Size | Smallest and largest graded size of each style | Linearity across size (bias vs dimension) |
| Pieces | ≥ 3–5 production pieces per style × size | Real production variation (needed for decision-risk estimates) |
| **Seeded defects** | Pieces deliberately altered by the sample room by known amounts (hem shortened, side seam taken in, sleeve shortened), with the true change measured by R2/R3 | Sensitivity; detects shrinkage toward nominal (slope < 1) and symmetry bias |
| POMs | 8–12 per family covering classes P1–P7 (document 01 §2.2). E.g. front length SNP→hem; CB length; chest 2.5 cm below armhole; hem sweep; across shoulder; sleeve length; sleeve opening; neck width; armhole (straight); waist relaxed; front and back rise; inseam; thigh; knee; leg opening; one pocket placement; one force-defined POM (only if a fixture is built) | Separates easy and hard POM classes |
| Scenario protocol | **S-AQL careful lay** vs **S-Inline quick lay** | State reproducibility per scenario |
| Environment | Record T and RH; time since pressing; one arm conditioned (hung ≥ 4 h, ISO 3759-style [E, ISO3759]) vs unconditioned | Moisture and relaxation effects |

The design is intentionally *factorial in the physics-relevant factors* (fabric mechanics, colour/contrast, construction) and *not* in style novelty. Style novelty matters for the AI component and is handled by hold-outs in S6.

---

## 5. The stages

### S0 — Measurand and protocol definition (desk work, before any measurement)

- Write **complete operators** for every POM in the frame (document 01 §3.3), cross-referenced to ISO 18890 codes [E, ISO18890]. For each ambiguous POM, record the alternative definitions (seam vs edge, direction datum) so S4 can quantify their difference.
- Write **lay protocols** for S-AQL and S-Inline. Specify orientation, alignment lines, smoothing strokes, side-seam placement rule, sleeve and leg angles, closure state and wait time.
- Assemble an **expert panel** (≥ 5 people from technical design and QC) and an **operator panel** (≥ 5 inspectors if possible; see §6.3 on why 3 is too few).
- Pre-register metrics, strata, hold-outs and go/no-go criteria (§7).

### S1 — Sensing chain on rigid artefacts (no garments)

- For each candidate sensor configuration (e.g. 45–61 MP 2D with front light; the same with backlight; the existing Zivid; optionally line-scan):
  - measure R1 scale bars and grids across the field and in several orientations;
  - repeat captures without touching anything (**static repeatability**);
  - repeat over a working day (**thermal drift**);
  - rotate artefacts 180° (**reversal, TIS-style** [E, TIS]);
  - place step blocks at the centre and the corners to verify the height/perspective model Δr = r·h/(H−h) (document 03 §2.2).
- **Outputs:** length bias and SD across the field; distortion residuals; drift; height sensitivity; u_C per configuration. A **head-to-head comparison of the Zivid and a high-resolution 2D camera** on the same artefacts answers the €15k question at the sensing level (document 04 §4.2).

### S2 — Textile artefacts with marks: edges, colours, lighting

- Flat specimens of each fabric and colour class, with printed or UV-ink marks and with both cut edges and folded edges. Where needed, mount on a stiff backing so the reference distances are stable.
- Capture under front, raking and back light. Measure:
  - mark-detection precision;
  - edge localisation bias between lighting modes (fold vs cut edge; fuzzy vs clean fabrics);
  - UV-ink visibility (it must be invisible in the white-light capture).
- **Outputs:** u_D per fabric and colour class and lighting mode; choice of edge operator (e.g. "outermost backlit silhouette"); marking method validated.

### S3 — Garment state: re-lays with physical markers, no AI

- Experts place **POM endpoint markers** on real garments according to the S0 operators. The markers stay on for this stage.
- Each operator lays each garment repeatedly (≥ 3 re-lays; randomised order; blind to earlier results) under each scenario protocol. Each lay is captured by all sensor configurations at once, plus the R3 reference.
- Distances are computed from **classically detected markers**. There is **no machine learning anywhere in this stage**.
- **Arms** (Q6):
  - vacuum table vs plain table;
  - glass platen vs none;
  - conditioned vs unconditioned;
  - re-measurement of the first lay at the end of the session (handling fatigue);
  - re-measurement after 24 h (stability).
- **Outputs:**
  - variance components for garment, operator, operator × garment and re-lay (§6);
  - u_B per POM class, fabric class and scenario;
  - the **bias of each restrained state relative to the free state** (document 04, A6);
  - the time per lay (economic input).

### S4 — Definitional spread: independent expert marking on a fixed lay

- A garment is laid once and held (e.g. lightly weighted edges, verified by repeat captures to be motionless).
- Each expert independently marks every POM endpoint twice:
  - **physically**, with UV ink under black light, so earlier marks are invisible during marking [E method, Kouchi2011];
  - **on the image**, by clicking on the full-resolution capture, blind to the others.
- Ambiguous POMs are marked under each alternative definition.
- **Outputs:**
  - u_A per endpoint (inter-expert and intra-expert SD of positions) and per POM (SD of the resulting distances);
  - the effect of definition choice (e.g. seam-to-seam vs edge-to-edge);
  - a list of POMs whose operators must be rewritten.

  Kouchi & Mochimaru found that dimension-level agreement understates landmark-level disagreement [E, Kouchi2011], so **both levels are reported**.

### S5 — Image-information ceiling: experts on images vs physical marks

- Compare the experts' on-image clicks (S4) and their clicks on the *clean* images of the S3 lays with the UV-marked physical positions on the same lays. Do this for each sensing configuration: front light, raking light, backlight, and a depth rendering.
- **Interpretation:**
  - If on-image clicks match the physical marks about as well as experts match each other physically, the **information is in the image**. Any remaining automated error is a *model* problem.
  - If on-image clicks are systematically off or much more scattered than physical marking, the **feature is not observable in that modality**. That is a *sensing/observability* problem, and the remedy is another modality or physical marks, not more training data.

### S6 — Automated localisation vs marks and vs the image ceiling

- Run each candidate pipeline on the clean images:
  - the previous team's model, if available (it gives a baseline and tests the §9 hypotheses of document 03);
  - a hybrid semantic-proposal + geometric-metric pipeline (document 04, layers 2–3);
  - optionally a human-in-the-loop confirmation workflow (A7).

  The **reference** is the UV-marked position. The **ceiling** is the S5 expert-on-image spread.
- **Report per POM:**
  - bias and SD;
  - median, 95th and 99th percentile of |error|;
  - **gross-error rate** (e.g. |error| > t/2) with confidence intervals;
  - a **risk–coverage curve**, i.e. error as a function of the fraction of POMs the system chooses to measure, using its own quality score [E concept, GeifmanElYaniv2017];
  - calibration of per-POM uncertainty estimates. Normality assumptions tend to understate landmark uncertainty; conformal methods give finite-sample guarantees [E, Jonkers2025].
- **Stratify** by seen vs unseen style, fabric class, colour class and background. The white-vs-green comparison tests the domain-gap hypothesis H3 of document 03.
- **Systematic per-POM bias** (not just scatter) is the signature of labels that encode a different definition, e.g. labels drawn on generated images (hypothesis H2).

### S7 — End-to-end measurement-system study and decision risk, compared with the manual method

- **Crossed design:**
  - garments: ≥ 10–20 per family, spanning production variation and including seeded defects;
  - operators: ≥ 3 (preferably ≥ 5);
  - replicates: ≥ 2–3 independent re-lays;
  - both methods: (a) the automated station as it would be deployed, including its abstention logic; (b) the manual tape method per ISO 18890 (R4);
  - reference: R3 marked-point values on the same lays wherever the markers do not interfere with the method under test.
- **Outputs:**
  - repeatability, reproducibility, %GRR and ndc (with confidence intervals);
  - ICC, bias vs reference, equivalence with manual (TOST), Bland–Altman limits of agreement;
  - slope on seeded defects;
  - pass/fail agreement with the reference;
  - **global consumer's and producer's risk under the measured production spread**, with and without guard bands [E, JCGM106];
  - times per garment and abstention rates for both methods (economic inputs).

---

## 6. Statistical analysis plan

### 6.1 Models

For a POM measured on garment g by operator o in lay r, capture c:

y_{gorc} = μ + G_g + O_o + (GO)_{go} + L_{gor} + ε_{gorc}

- G_g is the true garment-to-garment variation.
- O_o and (GO)_{go} are the operator and interaction components.
- L_{gor} is the re-lay (garment-state) component.
- ε is capture repeatability.

The model is fitted with **REML linear mixed models**, per POM class and fabric stratum, or hierarchically with stratum-specific variances. Confidence intervals for variance components and derived ratios use the methods in Burdick, Borror & Montgomery [E, Burdick2005] or parametric bootstrap. Point estimates of %GRR without intervals are not reported.

For comparisons with the reference:
- **error** e = y − y_ref, with bias estimated per stratum;
- **Deming / errors-in-variables regression** of y on y_ref (with known reference variance), testing slope = 1 (shrinkage) and intercept = 0;
- **Bland–Altman** limits of agreement for automated vs manual [E, BlandAltman1986];
- **ICC** of the appropriate form [E, ShroutFleiss1979; KooLi2016], reported alongside Wheeler's signal-attenuation interpretation [E, Wheeler189].

### 6.2 Assembling the uncertainty budget and checking it

From the stage estimates:

u_c² ≈ u_A² (S4) + u_B² (S3) + u_{C+D}² (S1–S2) + u_E² (S6, core errors) [+ systematic bias terms where not corrected]

The **consistency check** is that the end-to-end dispersion observed in S7 must be explained by the budget. A large unexplained remainder means a component is missing, e.g. interactions between fabric and lighting. This is the PUMA-style iteration of ISO 14253-2 in practice [I].

### 6.3 How many operators, garments and repeats?

The precision of an estimated standard deviation depends on its degrees of freedom. Approximate 95 % confidence limits for σ, as multiples of the estimate s [I; chi-square based]:

| Degrees of freedom | 95 % CI for σ |
|---|---|
| 5 | 0.62 s – 2.50 s |
| 10 | 0.70 s – 1.76 s |
| 20 | 0.77 s – 1.45 s |
| 50 | 0.84 s – 1.24 s |
| 100 | 0.88 s – 1.16 s |

Consequences:
- **Re-lay and repeatability components** accumulate degrees of freedom quickly (garments × operators × (lays − 1)). Aim for ≥ 50 df per stratum.
- **The operator component has only (number of operators − 1) df.** With 3 operators the reproducibility estimate is almost uninformative. Use ≥ 5–6 operators in S3/S7, or accept and report very wide intervals [I, consistent with Burdick2005].
- **Gross-error rates** need many opportunities. To show a rate below 1 % with reasonable confidence, several hundred POM measurements per stratum are needed (a rule of three: zero failures in n trials gives a 95 % upper bound ≈ 3/n) [I].

### 6.4 Heavy tails and abstention

Learned components produce heavy-tailed errors (document 02 §2). Therefore:
- report percentiles and gross-error rates, not only SDs;
- never trim outliers silently; classify them (wrong feature, occlusion, lay failure);
- evaluate the system at its operating point on the risk–coverage curve: the uncertainty that matters is that of *accepted* measurements, together with the abstention rate that the workflow must absorb.

### 6.5 Equivalence and decisions

- **Equivalence with the manual method** is tested with two one-sided tests (TOST) against a pre-registered margin δ, e.g. a stated fraction of T [E, Schuirmann1987]. Failing to find a significant difference is *not* evidence of equivalence.
- **Decision performance** is estimated two ways: directly, by comparing system and reference decisions on the sampled garments (including seeded defects); and by the JCGM 106 model using the measured uncertainty and production distribution [E, JCGM106]. The two should agree. Report both global risks, with and without guard bands.

---

## 7. Attribution: from observed pattern to root cause

| Observation | Diagnosis | Remedy class |
|---|---|---|
| S1 fails: artefact lengths biased or noisy beyond ~0.2–0.3 mm, drift, height sensitivity uncorrected | **Sensing / calibration** | Optics, calibration, camera height, orthographic scan, height correction |
| S1 passes; S2 shows large edge bias for certain fabrics or lighting | **Edge physics / lighting** | Backlight, per-fabric edge operator, definition of which edge counts |
| S3 re-lay SD large relative to target, while static repeatability is small | **Garment state** | Lay protocol, flattening fixtures (A6), conditioning. **No algorithm will fix this** |
| S4 inter-expert SD large | **Definition** | Rewrite the POM operator. If it stays large: inherently soft POM → negotiate the definition with the buyer or drop it from automation |
| S5 expert-on-image ≫ expert-physical | **Observability** (information not in the image) | Add a modality (raking light, backlight, depth) or physical marks |
| S5 ≈ physical, but S6 ≫ S5 | **AI / model** | Better model and data (physically grounded labels), geometric metric stage, or human-in-the-loop (A7) |
| S6 shows systematic per-POM offsets | **Label definition mismatch** (e.g. synthetic-image labels) | Relabel against physical definitions |
| Seeded defects: slope < 1 | **Prior shrinkage** | Weaken or remove priors (document 03 §6) |
| S7: automated ≈ manual, both ≫ target | **Requirement or definition problem, not an automation problem** | Revisit tolerances or definitions with buyers |

---

## 8. Go / no-go criteria (provisional — to be finalised in S0)

| Gate | After | Pass condition (proposal) | If it fails |
|---|---|---|---|
| **G1 Sensing** | S1–S2 | u_{C+D} ≤ ~0.3 mm for body-POM edges in the chosen configuration; drift controlled by an in-service check | Redesign optics or lighting; do not proceed to AI work |
| **G2 Physics & definition** | S3–S4 | For a POM to be feasible, √(u_A² + u_B²) ≤ ~0.8 × the target u (document 01 §8). The **programme** proceeds if the buyer-critical POMs pass under at least one acceptable protocol or fixture | Redefine POMs, add fixtures (A6), or restrict scope. **Stop the AI work** for POMs that fail |
| **G3 Automation** | S5–S6 | Automated localisation within ~1.2× the expert-on-image spread for accepted measurements; gross-error rate and coverage within agreed limits (e.g. coverage ≥ 90–95 % for S-AQL) | Deploy A7 (human-in-the-loop) for those POMs; keep collecting labelled data |
| **G4 System** | S7 | End-to-end uncertainty and decision risks within buyer-agreed limits **and** not worse than the manual method; equivalence or agreed bias with the contractual method; slope on seeded defects consistent with 1 | Iterate on the failing component identified by §7 |

---

## 9. Practical notes

- **Equipment** (most of it reusable in production):
  - a table ≥ 1.2 × 1.6 m with a backlight option;
  - a rigid camera stand at ≥ 1.5–2 m;
  - a 45–61 MP camera with a fixed lens, all settings locked;
  - raking lights (several azimuths);
  - a UV lamp, UV-fluorescent ink and adhesive micro-dots;
  - certified scale bars or grid and step blocks;
  - OIML R 35 class I/II steel rules;
  - the existing Zivid, as a comparison instrument and a garment-state probe;
  - a thermo-hygrometer;
  - optionally a vacuum-table prototype and a glass platen.
- **Garments:** bought from real production runs (not samples), plus seeded defects prepared by the sample room.
- **People:**
  - an expert panel (≥ 5);
  - an operator panel (≥ 5 preferred);
  - an analyst with measurement-systems / mixed-model statistics skills.
- **Pitfalls to design out:**
  - markers changing the fabric's behaviour (use the smallest marks; prefer ink);
  - experts seeing each other's marks (UV method);
  - garments changing with repeated handling (randomise order, re-measure the first lay at the end);
  - camera auto-settings (lock them);
  - lighting drift;
  - leakage of test garments or styles into training data.
- **Effort (planning estimate, [I]):** S0–S2 are desk and bench work measured in weeks. S3–S5 need a few hundred lays and a few thousand marked endpoints — also weeks, not months. S6–S7 depend on whether an automated pipeline already exists. **The first two gates (G1, G2) can be reached with no machine learning at all**, and they decide most of the feasibility question.

---

## Sources cited in this document

Full annotated entries in [references.md](references.md).

- `[Kouchi2011]` https://pubmed.ncbi.nlm.nih.gov/20947062/
- `[JCGM106]` https://www.bipm.org/documents/20126/2071204/JCGM_106_2012_E.pdf
- `[TMU]` https://www.spiedigitallibrary.org/conference-proceedings-of-spie/7272/727203/The-measurement-uncertainty-challenge-of-advanced-patterning-development/10.1117/12.814170.pdf
- `[TIS]` https://patents.google.com/patent/US8860941B2/en
- `[VDI2634]`, `[Luhmann2010]` VDI/VDE 2634 Part 1 as described in T. Luhmann, *ISPRS J. Photogramm. Remote Sens.* 65(6), 2010. https://doi.org/10.1016/j.isprsjprs.2010.06.003
- `[OIML-R35]` OIML R 35-1:2007. https://www.oiml.org/en/files/pdf_r/r035-1-e07.pdf
- `[ISO18890]` https://cdn.standards.iteh.ai/samples/63693/131110d2f792493aabb7200cbd9136ba/ISO-18890-2018.pdf
- `[ISO3759]` https://cdn.standards.iteh.ai/samples/57309/1af372753bf84b3cb0288a274429c73d/ISO-3759-2011.pdf
- `[Burdick2005]` R. K. Burdick, C. M. Borror, D. C. Montgomery, *Design and Analysis of Gauge R&R Studies*, ASA-SIAM, 2005. https://doi.org/10.1137/1.9780898718379
- `[BlandAltman1986]` *The Lancet* 327(8476):307–310. https://doi.org/10.1016/S0140-6736(86)90837-8
- `[ShroutFleiss1979]` *Psychological Bulletin* 86(2):420–428. https://doi.org/10.1037/0033-2909.86.2.420
- `[KooLi2016]` *J. Chiropractic Medicine* 15(2):155–163. https://doi.org/10.1016/j.jcm.2016.02.012
- `[Wheeler189]` https://www.spcpress.com/pdf/DJW189.pdf
- `[Schuirmann1987]` *J. Pharmacokinetics and Biopharmaceutics* 15:657–680. https://doi.org/10.1007/BF01068419
- `[GeifmanElYaniv2017]` NeurIPS 2017. https://arxiv.org/abs/1705.08500
- `[Jonkers2025]` https://arxiv.org/abs/2503.14106
