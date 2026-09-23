# Automatic garment measurement for industrial QC — feasibility study

A first-principles, metrology-led investigation of whether factory garments (knit tops, woven tops, bottoms) can be measured automatically with the accuracy, repeatability and reliability that industrial quality control needs — and, if so, under what conditions and with which architecture. It covers end-of-line sampling inspection (**S-AQL**) and 100 % inline measurement (**S-Inline**).

**Start with [06-synthesis.md](06-synthesis.md)** for the conclusions. Documents 01–05 contain the argument and the evidence.

## Reading order

| # | Document | What it answers |
|---|---|---|
| 01 | [Problem and requirements](01-problem-and-requirements.md) | What exactly is measured (points of measure, garment state, ground truth); what tolerances apply in practice; how accuracy, precision, repeatability, reproducibility, uncertainty and pass/fail risk are defined; which uncertainty the tolerances actually require; what "80 % accuracy" could mean |
| 02 | [Metrology and state of the art](02-metrology-and-state-of-the-art.md) | How precision industries achieve accuracy and which principles transfer to textiles; lessons from analogous soft-object and landmark domains; the state of garment measurement in research, patents and products; computer-vision and CAD building blocks and their limits |
| 03 | [Fundamental limitations](03-fundamental-limitations.md) | The error model; first-principles estimates for sensing, perspective, depth noise, wrinkles, handling strain, seams and model resolution; observability of reference points; algorithmic vs sensing vs physical vs definitional errors; a diagnosis of the previous system (as hypotheses) |
| 04 | [Candidate architectures](04-candidate-architectures.md) | Twelve architecture options (A0–A11) on a common template; a comparison matrix; throughput per scenario; economics and whether a €15k-class setup is necessary |
| 05 | [Feasibility and validation methodology](05-feasibility-and-validation-methodology.md) | A staged experiment (S0–S7) that isolates sensing, garment-state, definitional and AI errors; reference instrumentation; sampling frame; statistical analysis; go/no-go gates |
| 06 | [Synthesis](06-synthesis.md) | What is feasible, under what conditions, what remains uncertain, and which experiments to run first |
| — | [References](references.md) | Annotated bibliography, with access status for every source |

## Evidence labels

Every substantive claim carries one of:

- **[E] Established** — backed by a cited primary or authoritative source that was opened and checked. Standards that could only be read in preview are cited only for what the preview shows. "[E secondary]" or "[E vendor]" marks weaker grades: a claim reported by another source, or a manufacturer's claim.
- **[I] Engineering inference** — derived from first principles or arithmetic; the formula and assumptions are shown.
- **[H] Hypothesis** — plausible but unproven; document 05 says how to test it.

No performance numbers were invented. Where the answer depends on unmeasured quantities (above all, definitional spread and garment-state reproducibility), the documents say so and define the measurement.

## Conventions

- Units are SI (mm). Tolerances are written ±t with full width T = 2t; inch values appear where the source uses them (1/4 in = 6.35 mm, 1/2 in = 12.7 mm).
- u = standard uncertainty; U = 2u, the expanded uncertainty (k = 2); σ = standard deviation.
- Error components: **A** definitional, **B** garment state, **C** sensing, **D** feature extraction, **E** semantic localisation, **F** prior or constraint bias, **G** reference (document 03 §1).
- Architectures A0–A11 (document 04); experimental stages S0–S7 and gates G1–G4 (document 05).

## How the brief maps to the documents

| Item in the research brief | Where it is answered |
|---|---|
| Central question: can reliable industrial garment measurement be built, and with what architecture? | 06; 03 §8; 04 §5 |
| Treat it as metrology; how semiconductor, aerospace, precision engineering, inspection, optical and coordinate metrology achieve accuracy, and what transfers | 02 §1 (principles P1–P9 and why µm/nm does not transfer) |
| What limits accuracy: sensor, calibration, segmentation, keypoints, depth, perspective, occlusion, deformation, wrinkles, folds, fabric properties, seams, ambiguous definitions, observability | 03 §2–§5 |
| Which problems are algorithmic vs sensing vs physical | 03 §7 (matrix) |
| Alternatives to keypoints: contours, primitives, seam detection, CAD correspondence, registration, 3D reconstruction, constrained optimisation, template matching, hybrids | 02 §4–§5; 03 §5–§6; 04 (A1, A2, A5, A8) |
| Controlling the physical environment: positioning, flat-lay, tension, fixtures, fiducials, lighting, camera geometry, multiple views; acceptable standardisation | 04 (A1, A1-D, A2, A4, A6); 03 §2–§3; 01 §11 |
| Framework for accuracy, precision, repeatability, reproducibility, bias, uncertainty, pass/fail; MSA and standards; relation to tolerances | 01 §7–§9; 05 §6 |
| State of the art across computer vision, optical/3D metrology, inspection, deformable objects, textiles, garment CAD, QC | 02 §2–§5 |
| Several credible architectures compared on uncertainty, physics, robustness, calibration, hardware, compute, throughput, complexity, scalability, cost, failure modes | 04 §1–§3 |
| Ground truth: physical POM definitions; pattern vs sewn garment; reference measurements more accurate than the system | 01 §2–§5; 05 §3 |
| Feasibility experiment: garment types, measurement types, reference instruments, repeats, repositioning, fabrics and colours, variation, statistics; separating sensing failure from AI failure | 05 (S0–S7, §6–§8) |
| Economics; is a €15k setup necessary? | 04 §4; 06 §6 |
| The previous approach: RMBG, keypoints, correction algorithms, Zivid, synthetic Gemini/Nano Banana images on white vs a real green sheet, domain gap | 02 §4.3–§4.4; 03 §9; 06 §5 |
| Established evidence vs inference vs hypotheses | Labels throughout; summary table 02 §6 |
| Concise synthesis and experiments to run before full-scale engineering | 06 §7 |

## Scope and limits of this study

- **Scope:** confirmed with the requester. Both QC scenarios; detailed treatment of knit tops, woven tops and bottoms; dresses and outerwear only by extension.
- **No internal data:** no buyer tech packs or tolerance tables were available. Tolerances come from public documents (e.g. US Army purchase descriptions, inspection-company guidance) and are generic.
- **Paywalled standards:** they were not purchased. AIAG MSA and VDA 5 acceptance limits are cited via secondary sources and flagged as such.
- **No experiments were run.** This is a desk study. Every garment-specific performance statement is either cited or marked as a hypothesis to be tested by the programme in document 05.
