# 05 — Is it fast and economical enough to matter?

*Consolidated research, v2. This document answers **Question 4: can a feasible architecture run quickly and cheaply enough to give a real QC advantage over manual measurement?** Evidence labels: [E] source opened · [E-sec] secondary · [E-vendor] manufacturer claim · [I] inference · [H] hypothesis. **All handling and confirmation times below are stated assumptions for illustration.** Measuring them is a pilot task.*

---

## 1. The benchmark and what "better" means

- **Manual measurement** takes **45–60 s per garment** (project figure), i.e. **60–80 garments per inspector-hour** [I].
- **Sampling** is the consequence. In one worked AQL example, a 4,000-unit lot at inspection level II → code L → **200 units** at AQL 2.5 [E-sec, QIMA-AQL]. If every sampled unit were fully measured, that alone would take **2.5–3.3 inspector-hours per lot** [I]. How many sampled units are actually *measured*, as opposed to visually inspected, varies by buyer and was not found in a primary source.
- **Inconsistency** between inspectors adds disputes and noise to process control (document 01 §1.1).

An automated system creates value in four ways:
1. **Labour time per measured garment** — time saved relative to 45–60 s;
2. **Coverage** — more garments or more POMs measured for the same labour, up to 100 % for a few POMs;
3. **Consistency and traceability** — the same answer regardless of inspector, digitally recorded;
4. **Earlier detection of process drift** (SPC).

(1) and (2) depend almost entirely on **handling time**, not on sensing or computing.

---

## 2. Where the time goes

**t_garment = t_handle + t_capture + t_compute + t_confirm + p_abst · t_manual + p_re · t_re**

where:
- **t_handle** — unfold, lay, align, smooth or relax, and flip if back POMs need it. **The dominant term.**
- **t_capture** — well under 1 s for a 2D exposure sequence; 0.05–1.5 s for a structured-light capture [E, Zivid2+L110].
- **t_compute** — seconds on a workstation, and can overlap the next handling step [I]. **Heavy GPU is not a structural requirement for inference** [I].
- **t_confirm** — operator confirmation of points (C1 only).
- **p_abst · t_manual** — abstentions measured by hand.
- **p_re · t_re** — re-measurement of garments flagged near a limit (document 01 §9.2).

### 2.1 Illustrative scenarios (assumed times — to be measured)

| Scenario | Assumed components | t per garment | Garments per operator-hour | vs manual |
|---|---|---|---|---|
| Manual tape (baseline) | midpoint of 45–60 s | 52.5 s | 69 | 1× |
| Dedicated station, careful lay, automatic points (C2) | lay 25 s, capture 2 s, 10 % abstention | 32 s | 112 | ~1.6× |
| Dedicated station, careful lay, operator confirms points (C1) | lay 25 s, capture 2 s, confirm 15 s, 2 % abstention | 43 s | 84 | ~1.2× |
| Capture embedded at an existing flat step, quick adjust, automatic (C3) | adjust 5 s, capture 1 s, 10 % abstention; the step's own labour not counted | 11 s | ~320 | ~4.7× |
| Same, three sentinel POMs only | adjust 5 s, capture 1 s, 5 % abstention | 8.6 s | ~420 | ~6× |

[I; the arithmetic is exact, the inputs are assumptions]

**External anchor.** Toyoshima claims its camera station reduces measurement "work time by more than half" [E-vendor, Toyoshima] — about 2×, consistent with the dedicated-station range above.

**What this shows [I]:**
- A **dedicated station** in which an operator still lays and smooths each garment saves perhaps **1.2–2×** in labour. Its main benefits are **consistency, digital records and no transcription**, not a step change in coverage. The independent review argued this about the first-pass architecture; the concern is valid, though its stated cycle times were assumptions presented as findings.
- **An order-of-magnitude gain in coverage requires one of two things.** Either (a) the capture is **embedded in a handling step that already exists**, where each garment is laid flat anyway (end-line inspection is commonly 100 % [E-sec, EndLine]; folding or packing is another candidate), or (b) garments are **fed automatically in a defined state**, which is unsolved: robotic cloth-unfolding research reports *coverage*, and cloth keypoints for manipulation are centimetre-level [E, FlingBot; Lips2024] (document 04, C5).
- Whether (a) exists in the target factory is **unknown** (user confirmation). The roadmap makes it an S0 task: map the process and time each candidate step (document 07).

---

## 3. A two-tier deployment concept (conditional)

This adapts the independent review's two-tier idea, removing its unsupported cycle times, capex and "HIGH feasibility" ratings:

| Tier | What | Scenario | Conditions for it to work | Status |
|---|---|---|---|---|
| **Tier A — embedded sentinel (C3)** | Capture at an existing flat-handling step on 100 % of garments; measure 3–5 **robust** POMs (e.g. body length, chest or hem width, sleeve length — edge- and fold-defined) for SPC and screening; abstain liberally | S-Inline | (1) such a step exists; (2) the quick lay there meets the sentinel POMs' uncertainty targets (S3 quick-lay arm); (3) garment-level false rejects are acceptable at the measured production distribution; (4) abstentions and flags are routed cheaply | **[H]** — decided by S0 + S3 + S7 |
| **Tier B — audit station (C1 → C2)** | Full-spec measurement of AQL samples and Tier-A escalations, operator-confirmed at first, then automatic where G3 is met | S-AQL | Beats 45–60 s with equal or better reproducibility and equivalence to the contractual method | **[H]** — most likely to succeed first |

**Why Tier A matters even with few POMs [I].** Catching a cutting-table or wash drift on the tenth garment rather than at final audit has value far beyond the labour saved. But it depends entirely on a physically adequate lay at a step that already exists. Without that, 100 % measurement re-creates the handling bottleneck.

---

## 4. The cost of wrong decisions and abstentions

- **False rejects compound across POMs** (document 01 §9.2). With 15 POMs, u/t = 0.2 on a capable process (C_p ≈ 1.33) gives ≈ 2–3 % of good garments rejected. On a marginal process (C_p ≈ 1.0) it gives ≈ 8–12 % [I]. Every false reject costs re-measurement or rework time. A **re-measure-before-reject** rule for near-limit garments trades a little time for many fewer false rejects [I].
- **False accepts** are mostly marginal garments (document 01 §9.1). Whether that is commercially acceptable is a buyer question.
- **Abstentions** cost t_manual each. An automated station with a high abstention rate saves little. Coverage and gross-error rate must be optimised together (risk–coverage curve, document 07).

---

## 5. Value model (to be populated from the pilot)

Per station and year:

```
Value =  N · (t_manual − t_auto) · c_labour                                  labour saved per measured garment
       + ΔN_covered · v_coverage                                             extra garments/POMs measured (Tier A)
       + N · (Δp_escape · c_escape − Δp_false_reject · c_false_reject)      decision quality vs manual
       + V_SPC + V_disputes                                                  earlier drift detection; fewer measurement disputes

Cost  =  C_hw / L_hw + C_maint + C_calib + C_dev / (N_stations · L_dev) + p_abst · N · t_manual · c_labour
```

- **t_auto** includes handling, confirmation and re-measurement.
- **p_abst, Δp_escape, Δp_false_reject** depend on the measured uncertainty **and** the production distribution.
- **v_coverage, V_SPC, c_escape** are buyer- and factory-specific.

No monetary values are inserted.

**Sensitivities [I]:**
1. **Handling time** dominates labour value.
2. **Abstention rate** directly erodes it.
3. **The decision rule** (guard bands vs re-measure) trades escapes against false rejects.
4. **Development cost amortisation** favours standard components (camera, backlight, fixtures) plus simple, auditable software over bespoke 3D or robotic stacks.

---

## 6. Hardware and development cost

| Item | Anchor | Status |
|---|---|---|
| Commercial camera + backlit-table garment station | US$5–7k per station | [E-vendor, Seamline] (vendor blog, via search summary) |
| 61 MP camera body | US$3,900 MSRP | [E, Prices] |
| Industrial 12 MP camera kit | ≈ US$4.8k | [E-sec, Prices] |
| Structured-light 3D camera (Zivid 2+) | Quote-based. The project's previous setup including GPU was ≈ €15k | [E, Prices; project information] |
| Backlight table, raking lights, guides, fiducial frame, height channel, compute | Order of magnitude: hundreds to low thousands of euros each | [I] — to be quoted |
| Float → hold surface, tension fixtures | Not estimated (design-dependent) | [H] |
| **Development** (definitions, protocols, software, validation) | The previous team spent about two years | Dominant cost and risk |

The independent review listed detailed capex ranges (e.g. "€5,500–€8,400" for a stereo station, "€12.5–16k" for the Zivid). No sources were given, so they are not reproduced here.

---

## 7. Is a €15k-class setup necessary?

**No — not as the metric sensor. And the price was never the main issue** [I, to be confirmed in S1]:
- For flat-lay POMs the metric sensor should be a calibrated high-resolution 2D camera with backlight and raking light. This is cheaper, samples 2–3× finer, and is verifiable in-plane (documents 03–04).
- A 3D sensor's genuine value here is as a **height channel** (perspective correction, flatness detection). For that role the **existing Zivid is well suited and already paid for**.
- The spending that most plausibly reduces uncertainty is on **definitions, state control (surface, protocol, fixtures), calibration artefacts and a validation reference** — not on more sensing or GPU [I].
- The decisive economic risk is **development effort spent before the physical feasibility envelope is known.** Gates G1–G2 (document 07) are designed to remove that risk with little spending and no machine learning.

---

## 8. What the pilot must measure for the business case

1. **t_manual** per garment family for the buyer's actual POM set, with an MSA of the manual method (verifies the 45–60 s figure and inspector reproducibility).
2. **Existing flat-handling steps** — where each garment is laid flat today, their durations, and lay quality (S0 process mapping).
3. **t_handle** under the careful (S-AQL) and quick (S-Inline) protocols, and with each physical-control option (S3).
4. **t_confirm** and operator fatigue in C1 over a shift (S7).
5. **p_abst** vs gross-error rate (risk–coverage curve) for C2/C3 (S6–S7).
6. **Production distributions** (C_p, centring, correlation between POMs) for the pilot styles, to compute garment-level false-reject and false-accept rates (S0/S7).
7. **Costs** of false rejects, escapes and disputes, from factory and buyer records.

---

## Sources cited in this document

See [references.md](references.md): QIMA-AQL, EndLine, Zivid2+L110, FlingBot, Lips2024, Toyoshima, Seamline, Prices.
