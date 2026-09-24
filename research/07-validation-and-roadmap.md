# 07 — Validation methodology and research roadmap

*Consolidated research, v2. Evidence labels: [E] source opened · [E-sec] secondary · [E-vendor] manufacturer claim · [I] inference · [H] hypothesis.*

Whether garment measurement can meet industrial requirements depends mainly on quantities no one has measured:
- the **definitional spread** of POM endpoints (A);
- the **reproducibility of garment state** under a practical protocol (B);
- the **observability** of endpoints in each sensing modality;
- for the business case, **handling time** and the **production distribution**.

This programme measures them first and cheaply, mostly **without machine learning**. Each stage isolates one error component, so any failure can be attributed to sensing, physics, definition or AI.

---

## 1. Reference methodology: how "ground truth" is established

### 1.1 Principles

1. **Say which truth.** The *conventional* value (the contractual manual method) and the *reference* value (a reference procedure with smaller uncertainty) are both needed. The CAD nominal is **not** truth (document 01 §5).
2. **Same lay.** System and reference measure the **same physical configuration**. Otherwise garment-state variability contaminates the accuracy estimate [I].
3. **Mark the endpoints physically.** Marked reference points are the standard textile practice when edges are ambiguous (ISO 3759 allows ink with a template, sewn threads, holes, staples) [E, ISO3759]. Marks turn soft landmarks into hard ones. The *definition* is then fixed by the expert who places the mark, and the spread among experts is measured separately (S4).
4. **Blind, invisible marking.** Marks made with **UV-fluorescent ink under black light** cannot be seen by later markers [E method, Kouchi2011]. The same lay can then be imaged **clean** under white light (for humans and models) and **marked** under UV (for the reference) [H — invisibility in the capture spectrum verified in S2].
5. **Reference better than the system.** Aim for a reference standard uncertainty ≤ ¼ of the target, a TUR-style 4:1 [E, JCGM106 §7.6]. Where that is not achieved, **model** the reference error rather than ignore it: Deming / errors-in-variables regression, and the semiconductor "total measurement uncertainty" practice of separating reference-system error [E-sec, TMU].
6. **Equivalence with the contractual method** (ISO 18890 §4.2) is demonstrated statistically (TOST) or replaced by a documented, agreed bias [E, ISO18890; Schuirmann1987].

### 1.2 Reference instrumentation

| Ref | What | Role | Uncertainty notes |
|---|---|---|---|
| **R1 Rigid planar artefacts** | Certified dot/grid plate(s) or calibrated scale bars placed across the field in several orientations (VDI/VDE 2634-style testing [E-sec, VDI2634; Luhmann2010]) | S1 sensing verification; in-service checks | From the calibration certificate (preferably an ISO/IEC 17025 lab). Negligible at garment scale [I] |
| **R2 Step artefacts** | Blocks of known thickness (e.g. 1–15 mm) | S1 perspective / height-channel verification | Certificate |
| **R3 Certified rules and tapes** | OIML R 35 class I or II: MPE ±(0.1 + 0.1L) or ±(0.3 + 0.2L) mm, L in whole metres — **±0.2 or ±0.5 mm up to 1 m** [E, OIML-R35] | Manual readings between *marked* points | Handling and reading dominate, not the instrument [I] |
| **R4 Marked-point optical reference** | UV-ink or micro-dot marks + the S1-verified camera with in-plane scale bars (and height correction for raised marks) | Reference distances on the same lay (S3–S7) | Target mark-position uncertainty ≲ 0.2–0.3 mm [H — verified in S1/S2] |
| **R5 Conventional manual method** | Trained inspectors, ISO 18890 / buyer How-to-Measure, tape | S7 comparator | Its reproducibility is **measured**, not assumed |

For a POM whose reference is the consensus of n experts' marks: **u_ref² ≈ u_R4² + u_A²/n** [I].

---

## 2. Sampling frame (scope: knit tops, woven tops, bottoms)

| Factor | Minimum proposal | Purpose |
|---|---|---|
| Styles | Knit tops: jersey T-shirt, piqué polo, fleece sweatshirt/hoodie. Woven tops: poplin shirt, soft blouse. Bottoms: rigid denim, stretch denim, twill chino, knit shorts | Fabric mechanics and construction |
| Colour / surface | White, black (dark-on-dark seams), mid-tone, **green** (tests the green-sheet failure mode), printed or striped, heathered | Edge and seam contrast; background interaction |
| Sizes | Smallest and largest graded size | Linearity with size |
| Production pieces | ≥ 3–5 per style × size | Production variation — needed for decision risk |
| **Seeded defects** | Sample room alters pieces by known amounts (hem shortened; one sleeve shortened; side seam taken in), measured by R3/R4 | Sensitivity; shrinkage-toward-nominal and symmetry-bias audit |
| POMs | 8–12 per family covering P1–P7 (document 01 §2.2), including one force-defined POM only if a fixture is built | Easy vs hard POM classes |
| Protocols | S-AQL careful lay; S-Inline quick lay | State reproducibility per scenario |
| Orientation | Smoothing along the grain vs diagonal; garment rotated 0°/180° | Anisotropy; reversal diagnostics |
| Environment | Temperature and RH logged; time since pressing; conditioned vs not | Moisture and relaxation drift |

---

## 3. The measurement ladder

| Stage | Isolates | Machine learning? | Core protocol | Key outputs |
|---|---|---|---|---|
| **S0 Definitions, process and production** | — | No | Write **complete operators** for every POM (ISO 18890-referenced), with alternatives for ambiguous POMs. Write careful and quick **lay protocols**. **Map the factory's handling steps** (end-line inspection, folding, packing), time them, and assess where garments are already laid flat. **Sample production distributions** of the pilot POMs with the manual method. Pre-register metrics, strata, hold-outs and gates | Operator catalogue; candidate embedding points; manual cycle times; initial C_p per POM |
| **S1 Sensing chain** | C | No | Rigid artefacts across the field and orientations; static repeats; day-long drift; 180° reversal (TIS-style [E-sec, TIS]); step blocks at centre and corners to verify Δr = r·h/(H−h) and the height-corrected result; the **same tests for the Zivid** as metric sensor and as height channel | u_C per configuration; the sensing half of the €15k question |
| **S2 Edges and marks** | D | No | Fabric specimens with cut and folded edges and marks under backlight, diffuse front light and raking light; UV-ink visibility and detection | u_D per fabric/colour/lighting; choice of edge operator; marking method validated |
| **S3 Garment state** | B | No | Marked production garments; ≥ 5 operators × ≥ 3 re-lays each. **Arms:** careful vs quick lay; plain vs **float → settle → hold** surface vs platen; tension fixture for P5 and selected knit widths; along-grain vs diagonal smoothing; conditioned vs not; re-measure after 24 h with RH logged; *exploratory* stitch-density strain maps on knits [H]. Distances from classically detected marks only | u_B per POM class × fabric × protocol; restraint bias vs the free state; drift; time per lay |
| **S4 Definition** | A | No | A garment is laid once and held (repeat captures confirm no movement). ≥ 5 experts mark every endpoint twice — physically with UV ink under black light (blind [E method, Kouchi2011]) and by clicking on the full-resolution image. Ambiguous POMs are marked under each alternative definition | u_A per endpoint and per POM (**both levels reported**, since dimension agreement hides point disagreement [E, Kouchi2011]); POMs to rewrite |
| **S5 Observability** | Information content | No | Experts' clicks on **clean** images of S3/S4 lays vs the UV-marked positions, for each modality (front, raking, backlight, height rendering) | Whether each endpoint's information is in the image — the ceiling for any automated method |
| **S6 Automated localisation** | E | Yes | Pipelines: (a) the previous team's model if available (baseline, document 06); (b) hybrid semantic proposals + full-resolution geometry; (c) C1 operator confirmation. Reference = UV marks; ceiling = S5. Held-out styles; stratified by background (white/green/backlit), fabric and colour | Per-POM bias, SD, 95th/99th percentiles, **gross-error rate**, **risk–coverage curve** [E, GeifmanElYaniv2017], calibrated per-POM uncertainty [E, Jonkers2025] |
| **S7 End-to-end and decisions** | All | Yes | Crossed design: ≥ 10–20 garments per family (incl. seeded defects) × ≥ 5 operators × ≥ 2–3 re-lays, for the candidate station **and** the manual method (R5), with R4 references on the same lays where marks do not interfere | Repeatability, reproducibility, %GRR/ndc and Q_MP with CIs; bias; TOST equivalence; Bland–Altman; seeded-defect slope; **garment-level false-reject and false-accept rates** at the measured production distribution; cycle times and abstention (document 05) |

---

## 4. Statistical analysis plan

**Model.** For a POM on garment g, operator o, lay r, capture c:

y_gorc = μ + G_g + O_o + (GO)_go + L_gor + ε_gorc

- estimated by REML per POM class and fabric stratum, or hierarchically with stratum-specific variances;
- confidence intervals for components and ratios via Burdick–Borror–Montgomery methods or parametric bootstrap [E, Burdick2005];
- **no %GRR point estimates without intervals.**

**Budget check.** u_c² ≈ u_A²(S4) + u_B²(S3) + u_{C+D}²(S1–S2) + u_E²(S6, core). The S7 dispersion must be explained by this budget. An unexplained remainder means a missing component, e.g. a fabric × lighting interaction (ISO 14253-2-style iteration) [I].

**Method comparison:**
- errors vs reference;
- Deming / errors-in-variables regression (slope = 1 tests shrinkage; intercept = 0);
- Bland–Altman limits for automated vs manual [E, BlandAltman1986];
- ICC of the appropriate form [E, ShroutFleiss1979; KooLi2016], interpreted alongside Wheeler's attenuation view [E, Wheeler189].

**Heavy tails and abstention:**
- report percentiles and gross-error rates, never only SDs;
- classify outliers (wrong feature, occlusion, lay failure) — never trim them silently;
- evaluate at the chosen **risk–coverage** operating point.

**Garment-level decisions.** Combine the measured uncertainty components with the measured production distributions and **between-POM correlations** to compute garment-level false-reject and false-accept rates (as in document 01 §9.2). Cross-check against the direct confusion matrix on sampled garments (including seeded defects) [I].

**Sample sizes.** Approximate 95 % confidence intervals for σ, as multiples of the estimate s [I, chi-square]:

| df | 5 | 10 | 20 | 50 | 100 |
|---|---|---|---|---|---|
| 95 % CI for σ | 0.62–2.50 s | 0.70–1.76 s | 0.77–1.45 s | 0.84–1.24 s | 0.88–1.16 s |

- Re-lay components accumulate degrees of freedom quickly. Aim for ≥ 50 df per stratum.
- **The operator component has only (operators − 1) df.** Three operators are nearly uninformative, so use ≥ 5–6.
- To show a gross-error rate below 1 % with zero observed failures needs ~300 independent opportunities (rule of three, 3/n); 0.5 % needs ~600 [I].

**Equivalence.** TOST against a margin δ agreed with the buyer before the study. The independent review proposed δ = 0.2t, which is a reasonable starting point for negotiation, not a derived value. Failing to find a difference is *not* equivalence [E, Schuirmann1987].

---

## 5. Attributing failures

| Observation | Diagnosis | Remedy class |
|---|---|---|
| S1: artefact lengths biased or noisy, drift, uncorrected height effects | Sensing / calibration | Optics, calibration, camera height, height channel |
| S1 passes; S2 shows large fabric- or lighting-dependent edge bias | Edge physics / lighting | Backlight, per-fabric edge operator, specify which edge counts |
| S3 re-lay SD large while static repeatability is small | **Garment state** | Protocol, float/hold surface, restraint by agreement, conditioning. **No algorithm will fix this** |
| S4 inter-expert spread large | **Definition** | Rewrite the operator. If it stays large, negotiate the definition or drop the POM from automation |
| S5 on-image ≫ physical | **Observability** | Another modality, or physical marks |
| S5 ≈ physical but S6 ≫ S5 | **AI / model** | Better model or data (physically grounded labels), geometric metric stage, or operator confirmation (C1) |
| S6 systematic per-POM offsets | **Label definition mismatch** | Relabel against physical definitions |
| Seeded-defect slope < 1 | **Prior shrinkage** | Remove or weaken priors |
| S7: automated ≈ manual, both ≫ target | **Requirement or definition problem** | Revisit tolerances or definitions with buyers |
| S7 accurate but slow, or abstaining a lot | **Operations** | Handling design; embedding point; human-in-the-loop design (document 05) |

---

## 6. Gates (thresholds derived from documents 01 and 03 — provisional until agreed in S0)

| Gate | After | Pass condition | Derivation | If it fails |
|---|---|---|---|---|
| **G1 Sensing** | S1–S2 | u_{C+D} ≤ ~0.3 mm for body-POM edges. In-plane scale residual ≤ ~0.2 mm over the longest POM. Height-corrected step-block errors ≤ ~0.2 mm | Leaves ≥ 95 % of a 1.5 mm budget for A, B and E (document 03 §10) | Redesign optics, lighting or height channel. **Do not proceed to AI work** |
| **G2 Physics & definition** (decisive) | S3–S4 | Per POM class and protocol: √(u_A² + u_B²) ≤ 0.8 × u_target, with u_target = 0.075T (Q_MP 30 %) → **≤ 1.5 mm for ±1/2 in, ≤ 0.76 mm for ±1/4 in** | Leaves room for sensing and localisation within the Q_MP ≤ 30 % target (document 01 §8; requirement R4 in §11) | Rewrite operators; adopt a relaxation or restraint protocol; restrict scope to passing POMs. **Stop AI work for failing POMs** |
| **G3 Automation** | S5–S6 | Accepted-measurement error ≤ ~1.2× the expert-on-image spread; gross-error rate and coverage within agreed limits (e.g. coverage ≥ 90–95 % for S-AQL); seeded-defect slope consistent with 1 | Benchmarks the model against the information actually in the image | Deploy C1 (operator confirmation) for those POMs; keep collecting labelled data |
| **G4 System & business** | S7 | Garment-level false-reject and false-accept rates within buyer limits at the measured production distribution; not worse than the manual method's reproducibility; equivalence (TOST) or agreed bias; time per garment and abstention consistent with a positive value model (document 05) | Industrial objective (document 01 §1) | Iterate on the failing component (§5), or confine to the scenario that passes |

---

## 7. Roadmap

Durations are planning estimates [I]. They depend on garment availability and staff.

| Phase | Stages | Duration | ML? | Decision at end |
|---|---|---|---|---|
| **0 — Definition & process** | S0 (operators, protocols, process mapping and time study, production sampling, panels, pre-registration) | ~2–4 weeks | No | Scope: which POMs, which scenarios, which embedding points |
| **1 — Physical feasibility** | S1–S4 on a bench built as the C1 station (metric camera, backlight, raking light, Zivid height channel, guides; float/hold surface and tension-fixture arms) | ~6–10 weeks | **No** | **G1, G2 — the go/no-go decision for everything that follows** |
| **2 — Observability & semantics** | S5–S6; the C1 station runs in the sample room as a data engine (operator-confirmed points on real garments) | ~2–3 months | Yes | G3: which POMs can be automatic (C2) and which need confirmation (C1) |
| **3 — System validation & pilot** | S7 on the factory floor (S-AQL); Tier-A trial at an embedding point if S0/S3 support it | ~2–3 months | Yes | G4: pilot deployment decision |
| **4 — Scale** | Roll-out of the audit station; Tier-A sentinels where justified; in-service calibration and drift monitoring | — | Yes | Based on the measured value model |

**Equipment for phases 0–1** (mostly reusable):
- a table ≥ 1.2 × 1.6 m with a backlight;
- a rigid mount at ≥ 2 m;
- a 45–61 MP camera with fixed optics;
- multi-azimuth raking lights;
- the existing Zivid;
- a UV lamp, UV ink and micro-dots;
- certified grid or scale bars and step blocks;
- OIML class I/II rules;
- a thermo-hygrometer;
- a float/hold surface prototype and a tension fixture.

**People:**
- an expert panel (≥ 5, from technical design and QC);
- an operator panel (≥ 5);
- an analyst skilled in mixed models and measurement-system analysis.

**Pitfalls to design out:**
- marks altering fabric behaviour (use the smallest; prefer ink);
- experts seeing each other's marks;
- garments changing through repeated handling (randomise; repeat the first lay at the end);
- camera auto-settings;
- lighting drift;
- test garments leaking into training data.

---

## 8. Proceed / do-not-proceed conditions

**Proceed to full engineering if:**
- G1 and G2 pass for the buyer-critical POMs under a protocol the factory will accept (including any restraint agreed with the buyer);
- and G3 passes for enough POMs, or C1 operator confirmation is economically acceptable;
- and G4 shows garment-level decision risks within buyer limits **and** a positive value model at the measured handling times.

**Do not proceed (or restrict scope) if:**
- **G2 fails for core body POMs under every practicable protocol.** Then no sensing or AI investment will meet the tolerances. The options are redefining POMs with buyers, restraint standards, or positioning the system as a consistency and traceability tool rather than a tolerance authority.
- **Handling time cannot be brought well below the 45–60 s benchmark** and no embedding point exists. Then the business case rests only on consistency and data value, which must be judged on its own.
- **S5 shows key endpoints are unobservable in any practical modality** and physical marking in production is unacceptable.

---

## Sources cited in this document

See [references.md](references.md): ISO3759, Kouchi2011, JCGM106, TMU, ISO18890, Schuirmann1987, VDI2634, Luhmann2010, OIML-R35, TIS, GeifmanElYaniv2017, Jonkers2025, Burdick2005, BlandAltman1986, ShroutFleiss1979, KooLi2016, Wheeler189.
