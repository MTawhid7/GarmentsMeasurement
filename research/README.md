# Automated garment measurement for industrial QC — consolidated feasibility research (v2)

This is the single, consolidated research body on whether factory garments can be measured automatically with the accuracy, repeatability and reliability that industrial quality control needs. It covers knit tops, woven tops and bottoms, in two deployment scenarios:
- **S-AQL** — sampling and audit;
- **S-Inline** — 100 % of output.

It reconciles two independent research contributions — a first pass by Claude and an independent investigation and critique by Gemini — through evidence rather than merging them. Gemini's original documents are preserved unchanged in [`../archive/feasibility-study/`](../archive/feasibility-study/). The first pass is in git history ("Research phase 1"). **The conclusions are in [08-feasibility-assessment.md](08-feasibility-assessment.md).**

## The four questions and where they are answered

| Question | Document |
|---|---|
| **1. What exactly needs to be measured, and how well?** | [01-objective-and-requirements.md](01-objective-and-requirements.md) |
| **2. What physical information is required to measure it, and what limits it?** | [03-physics-and-error-sources.md](03-physics-and-error-sources.md) |
| **3. Which architecture can acquire that information with low enough uncertainty?** | [04-measurement-architecture.md](04-measurement-architecture.md) |
| **4. Can it run fast and cheaply enough to beat 45–60 s manual measurement?** | [05-operations-and-economics.md](05-operations-and-economics.md) |

## Headline conclusions (details and evidence in document 08)

1. **This is an industrial metrology problem, not an AI keypoint problem.** Machine learning identifies features. Calibrated geometry on physically defined features, in a specified garment state, measures them. The rule is that **no uncharacterised component produces the number.**
2. **Engineering the measurement environment and the measurand definitions is the main lever:**
   - a standard surface with a fiducial frame;
   - guides and lay rules;
   - a relaxation surface ("float → settle → hold") to test;
   - backlight, raking light and a height channel;
   - complete operators for every POM.

   More model capacity mainly reduces gross semantic errors.
3. **Likely feasible:** body lengths and widths defined by edges, folds and visible seams, on an engineered audit station.
   **Uncertain:** knits (low-load compliance, friction lock-in, buckles) and ±1/4 in POMs.
   **Not feasible passively:** force-defined POMs. Small details need a separate method.
4. **Throughput is set by handling, not sensing.**
   - A stand-alone station plausibly gives about 1.2–2× the productivity of manual measurement, plus consistency and records.
   - Order-of-magnitude coverage needs capture embedded in a step where garments are already laid flat. Whether the target factory has one is unknown.
5. **A €15k 3D camera is not needed as the metric sensor.** The existing Zivid is well suited as the **height channel**. Money is better spent on definitions, state control, calibration and a validation reference.
6. **The decisive unknowns** — garment-state reproducibility and definitional spread per POM class and fabric — **can be measured in weeks to a few months with no machine learning** (document 07, gates G1–G2).

## Reading order

| # | Document | Purpose |
|---|---|---|
| 01 | [Objective and requirements](01-objective-and-requirements.md) | Business objective; POMs and standards; measurand = garment + state + operator; ground truth; the VIM vocabulary (accuracy, trueness, bias, precision, repeatability, reproducibility, resolution, uncertainty, tolerance); tolerance → uncertainty; **garment-level multi-POM decision risk**; "80 %"; requirements R1–R10 |
| 02 | [Evidence and state of the art](02-evidence-and-state-of-the-art.md) | Transferable metrology principles; analogous domains; academic and **commercial** garment systems (Toyoshima, Seamline); vision and synthetic-data evidence; evidence summary |
| 03 | [Physics and error sources](03-physics-and-error-sources.md) | Identifying vs measuring; the error model A–G; observability; perspective, height channel, depth noise, CIS; **knit compliance, anisotropy, friction lock-in, buckles vs pressure**; state sensing; priors; what is fixable by what; budget; bottlenecks |
| 04 | [Measurement architecture](04-measurement-architecture.md) | Why the environment should be engineered; physical-control options with their physics; sensing; semantic and metric layers; configurations C0–C5; what to prototype first |
| 05 | [Operations and economics](05-operations-and-economics.md) | Time model vs 45–60 s; embedding points; two-tier concept (conditional); cost of wrong decisions; value model; cost anchors; the €15k question; pilot measurements |
| 06 | [Previous approach reassessed](06-previous-approach-reassessed.md) | Known vs inferable vs unknowable; Zivid, RMBG and the green sheet, keypoints and correction heuristics, **Nano Banana synthetic data**; discriminating tests; what to salvage |
| 07 | [Validation and roadmap](07-validation-and-roadmap.md) | Reference methodology (same lay, UV marks, TUR, errors-in-variables, equivalence); stages S0–S7; statistics; attribution; gates G1–G4 with derivations; phased roadmap; proceed/stop conditions |
| 08 | [Feasibility assessment](08-feasibility-assessment.md) | The verdict; answers to the four questions; feasibility per POM class and scenario; ranked unknowns; proceed/stop; what would change the conclusions |
| 09 | [Reconciliation record](09-reconciliation-record.md) | What was kept, adopted, corrected or removed from both contributions, with evidence |
| — | [References](references.md) | Consolidated bibliography with honest access status |

## Evidence labels

- **[E]** established: a primary or authoritative source that was opened. Standards read only in preview are cited only for what the preview shows.
- **[E-sec]** established via a secondary source (including search-engine summaries of pages that could not be retrieved).
- **[E-vendor]** a manufacturer's claim: a precedent, not performance evidence.
- **[I]** engineering inference: the derivation and assumptions are shown.
- **[H]** hypothesis: document 07 specifies the experiment.

**No performance figures were invented.** Throughput times in document 05 are explicitly labelled assumptions.

## Conventions

- **Units:** SI (mm). ±t is a tolerance of full width T = 2t (1/4 in = 6.35 mm; 1/2 in = 12.7 mm). u is standard uncertainty; U = 2u.
- **Error components:** A definitional · B garment state · C sensing · D feature extraction · E semantic localisation · F prior bias · G reference.
- **POM classes:** P1–P7 (document 01 §2.2).
- **Configurations:** C0–C5 (document 04 §6).
- **Stages and gates:** S0–S7 and G1–G4 (document 07).

## How the brief maps to this research

| Item in the brief | Where |
|---|---|
| AI keypoint problem vs industrial metrology problem | 08 (verdict); 03 §1; 04 §1 |
| Critical evaluation of Gemini's claims, architectures, citations and conclusions | 09 §3–4; corrections applied in 01–07 |
| Retrospective evaluation of the first pass; weak reasoning removed | 09 §5 |
| The previous team: Zivid, RMBG, keypoints, correction algorithms, Nano Banana data, green sheet; inferable vs not; the synthetic-to-real gap | 06 |
| Identifying vs measuring; ML for semantics, deterministic metrology for dimensions | 03 §1; 04 §4–5 |
| Physical control: standard surface, registration, tension control, illumination, engineered environment | 04 §1–3; 03 §6 |
| Metrology framework: accuracy, precision, repeatability, reproducibility, resolution, bias, uncertainty, tolerance | 01 §6 |
| Validation against ground truth; relation to tolerances and QC decisions | 01 §5, §8–9; 07 §1, §4, §6 |
| Business objective: 45–60 s, AQL, inspector variability, production-scale consistency | 01 §1; 05 |
| The four questions | 01, 03, 04, 05; summarised in 08 §1 |
| Feasibility assessment; unresolved uncertainties; efficient experiments; reference methodology; architectures to prototype; proceed/stop conditions | 08; 07 §6–8; 04 §7 |

## Scope and limits

- **Desk research.** No garment experiments were run. Every garment-specific performance statement is cited, derived or marked [H].
- **No internal buyer data** (tech packs, tolerances, production distributions). Tolerances come from public purchase descriptions and inspection-company material.
- **Paywalled standards** were not purchased. The AIAG MSA and VDA 5 limits and the ISO 20685 values are cited via secondary sources.
- **The embedding point** (an existing step where each garment is laid flat) is **unknown for the target factory**. It is an S0 task.
