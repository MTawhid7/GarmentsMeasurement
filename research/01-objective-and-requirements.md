# 01 — What must be measured, and to what standard

*Consolidated research on automated garment measurement, v2 (September 2026). This document answers **Question 1: what exactly needs to be measured, and how well?** Evidence labels:*
- ***[E]*** *— established from a source that was opened;*
- ***[E-sec]*** *— established via a secondary source;*
- ***[E-vendor]*** *— a manufacturer's claim;*
- ***[I]*** *— engineering inference, with the derivation shown;*
- ***[H]*** *— hypothesis that needs an experiment.*

*Source keys resolve in [references.md](references.md).*

---

## 1. The industrial objective

### 1.1 The business problem

- **Manual measurement is slow.** Measuring a garment's points of measure (POMs) with a tape takes roughly **45–60 s per garment** (figure provided by the project). That is **60–80 garments per inspector-hour** [I, arithmetic].
- **So factories sample.** Buyers' lot acceptance follows ANSI/ASQ Z1.4 / ISO 2859-1 sampling. In one worked example, a 4,000-unit lot at general inspection level II gives code letter L, which at AQL 2.5 means a sample of 200 units, with the lot passing if 10 or fewer fail [E-sec, QIMA-AQL]. The US Army's coat purchase descriptions make finished measurements part of Z1.4-sampled conformance inspection [E, GLPD-07-13A].
- **Many factories also run 100 % end-of-line inspection** after sewing, which includes checks against size specifications [E-sec, EndLine]. How much of that is actual *measurement* rather than visual checking varies by factory, and is unknown for the target factory.
- **Inspectors disagree.** Positioning, tension, landmark interpretation and technique all vary. No published measurement-system study of *trained* garment inspectors was found. The available evidence is informal:
  - 20 untrained people measuring the same jackets differed by up to 3.02 cm [E, Paulauskaite2022];
  - one LiDAR study quotes a 3.25 % "relative error" for manual measurement without defining it [E, Kim2022].
- **The goal:** fast, consistent, traceable measurement, so that inspection coverage can grow beyond sampling and measurement data can drive process control.

### 1.2 What the factory actually needs from a measurement system

The output that matters is **decisions and data**, not keypoints:

1. **Per-POM conformity decisions,** rolled up into a **garment decision**: a garment is typically non-conforming if *any* POM is out of tolerance. Garment decisions then roll up into a **lot decision**.
2. **Measurement data with known uncertainty,** so that trends can be detected: a drifting cutting table, a line sewing sleeves short, a wash that shrinks more than planned.
3. **Speed and coverage** that beat the 45–60 s manual benchmark. Otherwise the system adds cost without adding coverage (document 05).

Segmentation masks, keypoints and point clouds are intermediate machinery. They matter only insofar as they deliver these outputs at known risk.

### 1.3 Two deployment scenarios

| | **S-AQL — sampling / audit** | **S-Inline — 100 % of output** |
|---|---|---|
| Purpose | Lot acceptance; replace tape and paperwork | Screen every garment; statistical process control (SPC) |
| Volume | A sample per size per lot | Every unit (possibly only a few "sentinel" POMs) |
| Time budget | Must beat 45–60 s for the full POM set | Must fit an existing handling step or the line's takt time |
| Handling | Trained operator, written lay protocol | Minimal extra handling. Feasible only where garments are *already* laid flat, or with a fast feeder (document 05) |
| Fallback | Inspector re-measures by hand | Abstentions must be rare and routed cheaply |

---

## 2. What is being measured

### 2.1 Standards define *where and how* to measure, not the tolerances

**ISO 18890:2018** (*Clothing — Standard method of garment measurement*) defines the main measurement points and method. Its intended uses include "production control" and "garment inspection and quality control" [E, ISO18890]. Key clauses:

- The garment is laid on a flat surface; points are identified and measured "in a straight line between two points" unless otherwise specified.
- **Apparatus:** a flat surface "suitable for the garment to be laid flat with no overhang" and a "flexible and non-stretchable measurement tape, graduated in millimetres". Crucially: **"Any measurement device which leads to the same results may be used. The use of such measurement device should be reported."** (§4.2)
- **State:** the garment is "flat, in an ambient environment and without mechanical tension"; knitted garments should be in a "relaxed state"; fasteners are closed; "unattended folds, wrinkles and/or creases" are avoided. ISO 139 conditioning (20 °C, 65 % RH) applies only when the parties require it (§4.3.1).
- **Under tension:** "stretched to the maximum without seam or stitch breakage, or tear of the fabrics" (§4.3.2, normative Annex A).
- **Definitions:**
  - The side neck point (SNP, alias HPS) is "where the garment body intersects with the side of the neck", *excluding* neck trim, collar or hood.
  - D2 chest is measured "at 2,5 cm below base of armhole … from edge to edge".
  - Several POMs are "seam to seam or edge to edge as agreed by interested parties".
  - "Required positions" must be supplied, or "the selected measurements cannot be carried out" (§3.5).

Related documents:

- **ISO 3759:2011** (dimensional-change testing) measures "between specific points, preferably at seams or between points where seams meet". It allows the points to be *marked*, measures "without unnecessarily tensioning the garment" to 1 mm, and **for flat garments says to "measure the width between seams"** (§7.1.6) [E, ISO3759].
- **GB/T 31907** (China) requires ≥ 600 lx lighting and 1 mm reading, and specifies stretching "pulling sizes" to the maximum [E-sec, GBT31907].
- **Purchase descriptions define their own rules.** The US Army combat-uniform coat is measured "in line with arm pit of armhole from outside folded edge to opposite outside folded edge" (half chest) and "along underarm seam from sleeve joining seam to bottom of sleeve (including cuff)" (sleeve) [E, GLPD-07-13A; GLPD-14-04A].

**No standard sets finished-garment tolerances.** In every source examined, tolerances are contractual (§6) [E for the documents examined].

**The standards disagree with each other.** ISO 3759 says "between seams"; ISO 18890 and the Army say edge-to-edge or fold-to-fold [E]. On a side-seamed body where both side seams have rolled r mm toward the camera, the two definitions differ by **2r** — for example **6 mm for r = 3 mm** — on the *same garment in the same state* [I]. On a tubular knit body with no side seams, "between seams" is undefined.

### 2.2 POM classes by what defines their endpoints

The useful engineering classification is by the **physical feature that defines each endpoint** and whether it can be observed from above in a flat lay. Physical detail is in document 03 §3.

| Class | Endpoint definition | Examples | Observability (flat lay, from above) |
|---|---|---|---|
| **P1 Edge-to-edge at a constructed position** | Two silhouette edges (fold lines on tubular bodies) at a position derived from a landmark | Chest 2.5 cm below armhole (D2); waist at a required position; hem width; thigh, knee, leg opening | Edges good. The construction inherits the anchor landmark's uncertainty, and the *direction* of "below" is often unstated |
| **P2 Seam-anchored lengths** | A seam junction at one end, an edge at the other | SNP→hem; CB neck seam→hem; sleeve along the underarm seam; inseam; rises "excluding the waistband" | The junction is often visible, but may lie under a collar or hood, or inside a fold |
| **P3 Seam-to-seam *or* edge-to-edge (declared ambiguous)** | Seams or edges "as agreed" | Across shoulder, back neck width, across front/back, armhole | Depends on the agreement; seams need topographic or texture cues |
| **P4 Curves and diagonals** | Landmarks joined by a straight line or a curve | Armhole straight, raglan, neck drop, shoulder slope | Landmarks are the difficulty; evaluation is easy once they are fixed |
| **P5 Force-defined** | Dimension under a specified extension | Waist or neck "extended" | **Not observable passively.** Needs actuation with a specified force |
| **P6 Small details** | Short distances between construction features | Pocket size or placement (−0/+1/8 in in the Army coat), label position, collar points | Visible, but tolerances are tiny (§8) |
| **P7 Hidden or multi-layer** | Features inside or under other layers | Neckband of a closed collar, linings, pocket bags | Poor. Needs opening, a former or another modality |

---

## 3. The measurand is "garment + state + operator"

### 3.1 Definitional uncertainty sets a floor no instrument can beat

The VIM defines the **measurand** as the "quantity intended to be measured". Specifying it requires "describing the state of the phenomenon, body, or substance carrying the quantity" [E, VIM-2.3]. **Definitional uncertainty** — the component "resulting from the finite amount of detail in the definition of a measurand" — is "the practical minimum measurement uncertainty achievable in any measurement of a given measurand" [E, VIM-2.27].

If a POM's wording allows several physically different distances on the same garment, **no sensor, algorithm or inspector can do better than that spread.** The only remedy is a more complete definition.

### 3.2 Non-rigid parts: the state belongs to the specification

**ISO 10579:2010** covers parts that deform beyond tolerance in the free state:
- the **free state** is "subjected only to the force of gravity";
- since distortion depends on orientation and support, the drawing must state "the direction of gravity, conditions in which it is to be supported, etc.";
- alternatively it specifies a **restrained condition** with defined forces or fixtures [E, ISO10579].

Garments are extreme non-rigid parts. ISO 18890's "without mechanical tension" and "avoid … wrinkles" are qualitative. They specify no support, force limit or flattening method in the ISO 10579 sense [I]. Document 03 §6 shows why the resulting state is history-dependent.

### 3.3 Complete specification operators

GPS standards model a specification as an ordered set of operations — partition, extraction, filtration, association, collection, construction, evaluation. An **incomplete** operator "is ambiguous and therefore introduces ambiguity of specification"; the textbook example is "the step dimension 30 ± 0,1, which does not specify the association to be used" [E, ISO17450-2]. Nielsen lists the typical causes of ambiguity: poorly defined **filtering**, **direction** and **feature boundaries** [E, Nielsen-CIRP]. All three occur in garment POMs:
- does fibre fuzz or a wavy hem belong to the "edge"?
- is "2.5 cm below" measured along the centre-front line or vertically in the image?
- where exactly is the "base of armhole" on a rounded underarm?

A **complete operator** for "chest 2.5 cm below armhole, edge to edge" might read as follows. The parameters are examples and would have to be agreed with the buyer.

```
1 Partition     left/right underarm regions and side-edge regions (from the semantic layer)
2 Extraction    sample each side edge at full sensor resolution (sub-pixel)
3 Filtration    suppress edge features shorter than an agreed cutoff (e.g. fibre fuzz ≲ 1–2 mm)
4 Association   fit a line to each side edge over an agreed segment; fit the armhole and side seams
5 Construction  datum A = fitted centre-front axis (placket or symmetry axis)
                anchor = intersection of the fitted armhole seam with the side edge
                measurement line ⟂ datum A, 25.0 mm below the anchor, measured along datum A
6 Evaluation    distance between the line's intersections with the two filtered edges
```

**This is the single most transferable idea from precision metrology** [I]. It converts a vague instruction into a machine-executable recipe and makes the remaining ambiguity explicit, so it can be quantified (document 07, stage S4).

### 3.4 Three consequences

1. **An automated system is "another measurement device"** under ISO 18890 §4.2. It must *lead to the same results*, which is an equivalence claim that has to be demonstrated by a method-comparison study (document 07) [E + I].
2. **Standard capability frameworks assume the definition is sharp.** ISO 22514-7 (the basis of VDA 5) is "only useful if it is known that the method uncertainty and the specification uncertainty are small compared to the implementation uncertainty". It is not suited to measurements "that rely on several measurement points" [E, ISO22514-7]. For garments this assumption is unverified [H], so the definitional component must be measured separately.
3. **Tolerances were calibrated against manual practice.** When tolerances come from experience with a particular measurement method, the method details must be part of the specification or "the tolerance is meaningless" [E, Nielsen-CIRP]. An automated system that realises a *different* operator can be more repeatable yet **biased relative to contractual practice**. That bias must be measured and either corrected or agreed [I].

---

## 4. Garment state

This section states the requirement; the physics is in document 03 §6.
- The dimensions of the *same* knit depend on its relaxation history. Munden linked relaxed plain-knit dimensions to loop length, confirmed after dry and wet relaxation [E, Munden1959 abstract].
- Handling leaves strain; in low-friction and low-stiffness regimes this is fabric-specific and can be millimetre-level for compliant knits [I/H, document 03 §6.2].
- Wrinkles always shorten projected lengths [I, document 03 §6.3].
- ISO 3759 conditions garments to constant mass before dimensional testing [E]. ISO 18890 makes conditioning optional [E].

**Requirement:** the measurement state is defined **operationally**, by a written lay or restraint protocol per garment family and scenario, and its reproducibility is **measured**. This is ISO 10579 applied to textiles.

---

## 5. Ground truth

The design pattern and the finished garment differ systematically:
- seam allowances and seam-gauge changes (the Army PD requires pattern adjustment when the seam construction changes);
- sewing take-up;
- pressing;
- garment treatments — manufacturers must "compensate for actual fabric shrinkage or growth after the application of garment treatment" [E, GLPD-07-13A; GLPD-14-04A].

They also differ randomly, through production variation. **The CAD pattern is therefore the nominal — what QC compares against — and cannot be the truth for the measured value.** It is useful as a *prior* (topology, approximate locations). But any algorithm that pulls measurements toward it hides the deviations QC exists to find (document 03 §8).

| Kind of "truth" | Definition | Role |
|---|---|---|
| **Conventional value** | What competent inspectors obtain with the contractual method (ISO 18890 or the buyer's How-to-Measure) | What contracts and disputes refer to. The system must be equivalent to it, or its bias must be agreed |
| **Reference value** | Result of a reference procedure with stated, smaller uncertainty: marked endpoints, a controlled lay, a calibrated instrument, the *same lay* as the system under test | What validation uses to estimate bias and precision (document 07) |
| **Design nominal** | CAD / spec-sheet value | What conformity is judged against. Not a measurement |

Every accuracy statement must say **which truth** it was measured against and **how uncertain** that truth was. A system with u = 1.0 mm validated against an uncharacterised tape with u = 3.0 mm shows an apparent disagreement of √(1² + 3²) ≈ 3.2 mm, and the system gets blamed for the reference's scatter [I].

---

## 6. Metrology vocabulary for this project

VIM definitions [E, VIM] and JCGM 106 tolerance terms [E, JCGM106], with the operational meaning adopted here:

| Term | Formal definition (abridged) | Operational meaning in this project |
|---|---|---|
| **Accuracy** | "closeness of agreement between a measured quantity value and a true quantity value" — *not a quantity; not given a numerical value* | Used only qualitatively. **"X % accurate" is not a valid specification** |
| **Trueness / bias** | Trueness: closeness of the *average* of replicates to a reference value. Bias: "estimate of a systematic measurement error" | Mean error vs the reference per POM, garment type and fabric; including bias vs the *conventional* (manual) method |
| **Precision** | Closeness of agreement between replicate measurements under *specified conditions*; expressed as SD | Always reported with its conditions (next three rows) |
| **Repeatability** | Precision under the same procedure, operator, system, conditions and location, over a short time | Repeated captures of **the same lay without touching the garment** (sensor + algorithm noise) |
| **Intermediate precision** | Same procedure and location over an extended time; may include new calibrations and operators | **Re-lays** of the same garment by the same operator over a day (adds garment-state variability) |
| **Reproducibility** | Different locations, operators and measuring systems | Different operators, stations, days, factories |
| **Resolution** | "smallest change in a quantity being measured that causes a perceptible change in the corresponding indication" | Effective resolution of the evaluated dimension. Not the pixel size, although bounded by it |
| **Measurement uncertainty** | Non-negative parameter characterising the dispersion of values attributed to the measurand; **includes definitional uncertainty** | Combined standard uncertainty u_c from all components (document 03), and expanded U = 2u_c |
| **Tolerance** | Difference between upper and lower tolerance limits; tolerance interval = permissible values | Written ±t with full width **T = 2t** |
| **Conformity risk** | Specific and global consumer's risk (false accept) and producer's risk (false reject) [E, JCGM106] | Evaluated per POM **and per garment** (§8) |
| **Gross-error rate** | — | Share of measurements with \|error\| > t/2. Essential for learned components, whose errors are heavy-tailed |
| **Coverage / abstention** | — | Share of POMs or garments the system declines to measure and routes to a person |

---

## 7. Tolerances in practice (public sources only — generic)

| Source | Tolerances |
|---|---|
| US Army ACU coat, GL/PD 07-13A and 14-04A [E] | **±1/2 in (±12.7 mm)** on half chest, back length, sleeve length, shoulder, half sweep and half sleeve opening. Chest pocket **−0/+1/8 in** (a 3.2 mm window). Hook-and-loop widths ±1/32 in. Half-chest **grade step 2 in** |
| QIMA (inspection company) [E-sec] | Tolerances "vary for different parts of the garment"; measurement checks are done "for every clothing size" |
| Practitioner guides [weak] | Commonly ±0.5 cm for details up to ±1–1.5 cm for body dimensions |

Tables of "typical" tolerance tiers (±3/8 in, ±3/4 in and so on) circulate in industry but were not verified against primary sources here. **Real programmes must use the buyer's tech packs.** The grade step bounds the problem from outside: a system that cannot tell adjacent sizes apart is useless, while the tolerance sets the operative bound.

---

## 8. From tolerance to required uncertainty

Common criteria:
- **AIAG %GRR** = 6σ/T: ≤ 10 % acceptable, 10–30 % conditional, > 30 % unacceptable [E-sec, AIAG-MSA].
- **VDA 5 / ISO 22514-7:** Q = 2U/T, with Q_MS ≤ 15 % for the measuring system, Q_MP ≤ 30 % for the whole measurement process, and resolution ≤ 5 % of T [E framework; limits E-sec, VDA5-QDAS].
- **JCGM 106:** C_m = T/(4u); "C_m ≥ 4 means that u ≤ T/16" [E, JCGM106].

Largest allowable σ or u, in mm [I, arithmetic from the definitions]:

| Tolerance | T | σ for %GRR 10 % | σ for %GRR 30 % | u for Q_MS 15 % | u for Q_MP 30 % | u for C_m ≥ 4 | Resolution 5 % |
|---|---|---|---|---|---|---|---|
| −0/+1/8 in (pocket) | 3.2 | 0.05 | 0.16 | 0.12 | 0.24 | 0.20 | 0.16 |
| ±1/8 in | 6.4 | 0.11 | 0.32 | 0.24 | 0.48 | 0.40 | 0.32 |
| ±1/4 in | 12.7 | 0.21 | 0.63 | 0.48 | 0.95 | 0.79 | 0.64 |
| ±10 mm | 20.0 | 0.33 | 1.00 | 0.75 | 1.50 | 1.25 | 1.00 |
| ±1/2 in | 25.4 | 0.42 | 1.27 | 0.95 | 1.90 | 1.59 | 1.27 |

What follows from the table:
- **Resolution is never the bottleneck.** Every camera considered samples finer than 0.5 mm per pixel (document 03 §4.1) [I].
- **A 10 % %GRR for body dimensions is almost certainly unattainable by any method, including the tape.** It requires σ ≈ 0.2–0.4 mm across re-lays and operators on a material whose state varies by millimetres (document 03) [H, strongly indicated].
- **The realistic target zone for body POMs is the 30 % band: u ≈ 0.6–1.9 mm** for ±1/4 in to ±1/2 in [I]. Whether it can be met is the central experimental question [H].
- **Small-detail POMs (±1/8 in, −0/+1/8 in) need u ≲ 0.2–0.5 mm.** Treat them separately — dedicated close-range optics, go/no-go templates or manual checks [I].
- Wheeler cautions that %GRR "percentages" are not proportions. At %GRR = 30 %, 95.4 % of the production signal still comes through [E, Wheeler189]. %GRR is therefore not used here as the sole criterion. Requirements are set from **decision risk** (§9) and **comparison with the manual method**.

---

## 9. Decision risk — per POM and per garment

### 9.1 Per POM

JCGM 106 defines:
- **global consumer's risk** — the probability that a non-conforming item is accepted;
- **global producer's risk** — the probability that a conforming item is rejected.

The default guarded rule (ISO 14253-1) shrinks the acceptance zone by U = 2u on each side, capping the false-accept probability at 2.3 % for any accepted item [E, JCGM106].

Illustrative calculations [I]. The true value is normal with process capability C_p = t/(3σ_p); measurement error is normal with standard uncertainty u.

| Process | u/t | Rule | Truly out of tolerance | Falsely rejected | Share of bad items accepted |
|---|---|---|---|---|---|
| C_p = 1.33, centred | 0.20 | simple | 0.01 % | 0.18 % | — (too few bad items) |
| C_p = 1.0, centred | 0.10 | simple | 0.27 % | 0.20 % | 24 % |
| C_p = 1.0, centred | 0.20 | simple | 0.27 % | 0.83 % | 34 % |
| C_p = 1.0, centred | 0.20 | guarded (w = U) | 0.27 % | 12 % | 1 % |
| C_p = 1.0, mean at +t/2 | 0.20 | simple | 6.7 % | 5.1 % | 27 % |

Guard bands trade false accepts for false rejects very steeply once u/t > 0.1. Most accepted bad items are only marginally out, which may or may not matter commercially.

### 9.2 Per garment — the multi-POM effect

A garment is rejected if **any** of its m POMs measures out of tolerance. With independent per-POM false-reject rates r, the garment false-reject rate is 1 − (1 − r)^m, so small per-POM rates compound [I]. The general point was raised in the independent Gemini review. **How large the effect is depends on process capability and correlation.** Monte Carlo results for **m = 15 POMs** [I]:

| Process (every POM) | u/t | Per-POM false reject | Garment false reject (independent) | Garment false reject (ρ = 0.5 in true values and in errors) | Garments truly non-conforming (indep.) |
|---|---|---|---|---|---|
| C_p = 1.33, centred | 0.10 | 0.02 % | 0.2 % | 0.2 % | 0.1 % |
| C_p = 1.33, centred | 0.20 | 0.18 % | 2.6 % | 1.8 % | 0.1 % |
| C_p = 1.0, centred | 0.10 | 0.20 % | 3.0 % | 1.8 % | 4 % |
| C_p = 1.0, centred | 0.20 | 0.83 % | 11.9 % | 7.7 % | 4 % |
| C_p = 1.0, mean at +t/2 | 0.20 | 5.4 % | 57 % | 25 % | 64 % |

- **Positive correlation** (shared scale error, shared lay state) **reduces** the garment-level rate compared with independence.
- **Poorly centred processes** make any measurement system look bad: garments cluster near a limit.
- Gemini's headline "45.8 % of good garments rejected" is the arithmetic for r = 4 % with 15 independent POMs. That per-POM rate belongs to poorly capable or off-centre processes, **not to a typical u/t = 0.2 system on a capable process.**

**Requirements consequences:**
1. Specify acceptable **garment-level** false-reject and false-accept rates with the buyer. Derive per-POM uncertainty targets from them **using the measured production distribution and correlation** (document 07, S0/S7). Do not use a fixed rule of thumb.
2. On capable, centred processes, u/t ≈ 0.1–0.2 gives garment-level false rejects of ≲ 0.2–3 % for 15 POMs. On marginal processes, even u/t = 0.1 produces several percent [I].
3. Near-limit garments are exactly where both humans and machines disagree. A **re-measure-before-reject** policy (a second capture or lay for garments flagged within ~U of a limit) reduces false rejects at modest time cost [I]. Its effect should be simulated with the measured components.

---

## 10. What could the previous team's "80 %" mean?

Assuming normal errors [I]:

| Interpretation | Implied error SD | For ±1/2 in |
|---|---|---|
| 80 % of measurements within ±t of the reference | ≈ 0.78t | ≈ 9.9 mm |
| 80 % within ±t/2 | ≈ 0.39t | ≈ 5.0 mm |
| 80 % of garments with all 10 POMs within ±t (independent) | ≈ 0.44t | ≈ 5.6 mm |
| 80 % agreement with inspectors' pass/fail | Not determinable without the failure prevalence | — |

Under the interpretations that can be computed, the implied error is several times the ≈ 1–2 mm target zone. But the figure's meaning, reference and conditions are unknown, so **it supports no firm conclusion about their system** (document 06). What must be reported instead, per POM and stratum:
- bias;
- repeatability, intermediate precision and reproducibility SDs;
- the 95th/99th percentiles of |error|;
- the gross-error rate;
- the abstention rate;
- garment-level false-reject and false-accept rates under the measured production distribution.

---

## 11. Requirements (provisional; confirmed or revised by the validation programme)

| ID | Requirement | Basis | Status |
|---|---|---|---|
| R1 | Every in-scope POM has a **complete operator** (§3.3), agreed with at least one buyer and cross-referenced to ISO 18890 | §3 | Design task |
| R2 | A written **state protocol** (lay, restraint or conditioning) per garment family and scenario, with its reproducibility measured | §4; document 03 §6 | Experiment (S3) |
| R3 | Performance reported in the §6 metrics against a **reference** with stated uncertainty, on the same lay | §5–6 | Experiment (S3–S7) |
| R4 | **Body POMs (±1/4–±1/2 in):** combined u ≲ 0.15T (Q_MP ≤ 30 %), i.e. about 0.9–1.9 mm, **and** garment-level false-reject and false-accept rates within buyer-agreed limits at the *measured* production distribution | §8–9 | Provisional |
| R5 | **Small-detail POMs:** separate method or explicitly excluded from automatic measurement | §8 | Scope decision |
| R6 | **Force-defined POMs:** actuated fixture with a specified force, or excluded | §2.2 | Scope decision |
| R7 | **Equivalence with the contractual method** (ISO 18890 §4.2), shown by a TOST against a margin agreed with the buyer, or a documented agreed bias | §3.4 | Experiment (S7) |
| R8 | **Reliability:** gross-error rate on accepted measurements and abstention rate within agreed limits; every abstention routed to manual measurement | §6 | To be agreed |
| R9 | **Calibration** traceable to a certified length artefact, verified in service, with drift monitored | document 07 | Design |
| R10 | **Speed:** time per garment (including handling, confirmation and abstentions) below the 45–60 s manual benchmark by a margin that justifies the investment (document 05) | §1 | Measured in the pilot |

---

## Sources cited in this document

See [references.md](references.md): ISO18890, ISO3759, GBT31907, GLPD-07-13A, GLPD-14-04A, QIMA-AQL, EndLine, Paulauskaite2022, Kim2022, VIM-2.3, VIM-2.27, VIM, ISO10579, ISO17450-2, Nielsen-CIRP, ISO22514-7, Munden1959, JCGM106, AIAG-MSA, VDA5-QDAS, Wheeler189.

*The formulas behind §8–10 are %GRR = 6σ/T; Q = 2U/T with U = 2u; C_m = T/(4u); and the JCGM 106 risk integrals with normal priors. Garment-level rates come from a Monte Carlo over m correlated normal POMs. The scripts are throwaway scratchpad analyses, and every number can be reproduced from the formulas.*
