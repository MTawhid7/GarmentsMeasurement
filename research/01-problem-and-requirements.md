# 01 — The problem and its measurement requirements

*Part of the garment-measurement feasibility study. Read [README](README.md) first for the evidence labels and the map of the documents.*

**Evidence labels.** **[E]** = established, backed by a cited primary or authoritative source that was opened and checked. **[I]** = engineering inference; the derivation or assumptions are shown. **[H]** = hypothesis that needs an experiment. Source keys in brackets (e.g. `[ISO18890]`) are listed at the end and resolved in [references.md](references.md).

---

## 1. The question, restated as a metrology problem

The factory does not need "keypoints". It needs **decisions** and **data**:

1. For each point of measure (POM) on each measured garment, a decision: *conforms / does not conform* to the specified tolerance. For a lot, this rolls up into accept or reject.
2. Measurement data with known uncertainty that can be trended, e.g. to spot a drifting cutting table, a line that is systematically sewing sleeves short, or a wash recipe that shrinks more than expected.

Everything else — segmentation masks, keypoints, 3D point clouds, neural networks — is intermediate machinery. It is only useful to the extent that it helps produce decisions with known, acceptable risk.

That makes this a **measurement and conformity-assessment problem**. The questions that matter are:

- What exactly is the quantity being measured (the *measurand*)?
- How well is it defined?
- How well can any instrument realise that definition on a floppy, three-dimensional, sewn textile?
- How does the remaining uncertainty turn into false accepts and false rejects?

The measurement chain we are evaluating is:

```
garment (as produced) ──► garment state (how it is laid / held) ──► sensing (optics, depth, calibration)
     ──► feature extraction (edges, seams, folds) ──► semantic identification (which feature is "the armhole point")
     ──► geometric evaluation (distance along a defined path) ──► decision (conform / not conform, with a decision rule)
```

Each link adds uncertainty. The previous team's experience — points "slightly displaced from their intended physical locations" — is a symptom that could come from at least four different links: definition, state, sensing, or the semantic model. The purpose of this document set is to separate those links.

Two deployment scenarios are analysed in parallel throughout (confirmed scope):

| | **S-AQL** — sampling inspection | **S-Inline** — 100 % measurement |
|---|---|---|
| Where | End-of-line, final or pre-shipment inspection | On the line, every garment |
| Volume | A sample per size per lot (ANSI/ASQ Z1.4 / ISO 2859-1 style sampling) | Every unit |
| Cycle-time budget | Minutes per garment are acceptable | Must fit the line's takt time, typically seconds |
| Handling | A trained operator lays out and flattens the garment by protocol | Minimal handling; ideally conveyor-fed |
| Fallback | The inspector can re-measure by hand | Abstentions must be rare and routed cheaply |
| Main value | Consistency, traceability, faster reporting, removing transcription errors | Process control and 100 % screening |

---

## 2. What is being measured: points of measure

### 2.1 There is now an international standard for *where and how* to measure — but not for tolerances

**ISO 18890:2018, *Clothing — Standard method of garment measurement*** (ISO/TC 133) "defines the main measurement points and describes the method used to measure garment dimensions". It lists "garment inspection and quality control" and "production control" among its intended uses [E, ISO18890]. Its general procedure is:

- The garment is "laid on a flat surface, its measurement points identified and the measurements carried out between the relevant measurement points using a measurement device".
- The apparatus is a clean, flat surface "suitable for the garment to be laid flat with no overhang" and a "flexible and non-stretchable measurement tape, graduated in millimetres".
- "Any measurement device which leads to the same results may be used. The use of such measurement device should be reported." [E, ISO18890 §4.2]
- Preparation: each garment is "flat, in an ambient environment and without mechanical tension", which is "especially important for knitted garments which should be in a 'relaxed state'". Fasteners are closed, and the garment is laid "as flat as possible", avoiding "any unattended folds, wrinkles and/or creases". Conditioning to the ISO 139 atmosphere (20 °C, 65 % RH) applies only "when it is required by interested parties" [E, ISO18890 §4.3.1].
- "Unless otherwise specified, measure in a straight line between two points". For measurement under tension, stretch "to the maximum without seam or stitch breakage, or tear of the fabrics" (normative Annex A) [E, ISO18890 §4.3.2].

Older and adjacent documents define POMs in the same spirit:

- **ISO 3759:2011** (measuring garments for dimensional-change tests) says to "measure between specific points, preferably at seams or between points where seams meet". It allows the positions to be marked, and specifies measurement "without unnecessarily tensioning the garment" to at least 1 mm [E, ISO3759 §7.1].
- **GB/T 31907** (China, *Method of garment measurement*; 2015, revised 2025) requires ≥ 600 lx lighting and 1 mm reading. For "pulling size" measurements it requires stretching to the maximum without damaging stitches [E, secondary summary, GBT31907].
- **Buyer and government purchase descriptions** define their own POMs. For example, the US Army Combat Uniform coat purchase descriptions measure half chest "from outside folded edge to opposite outside folded edge" and sleeve length "along underarm seam from sleeve joining seam to bottom of sleeve (including cuff)" [E, GLPD-07-13A; GLPD-14-04A].

**No international standard sets finished-garment tolerances.** ISO 18890 standardises the method, not the acceptance limits. In every source examined, tolerances are contractual, set in buyer tech packs or purchase descriptions (§6). [E for the documents examined; a general negative cannot be proven.]

### 2.2 A taxonomy of POMs by geometry and observability

For measurement engineering the useful classification is not "tops vs bottoms". It is **what physical feature defines each end of the measurement**, and whether that feature can be seen from above when the garment lies flat. The table uses ISO 18890 codes where they were verified (upper-body tables) and ISO 3759 / generic industry definitions for bottoms.

| Class | What defines the endpoints | Examples (knit tops, woven tops, bottoms) | Observability in a flat lay, from above |
|---|---|---|---|
| **P1 Edge-to-edge at a constructed position** | Two silhouette edges, which are *fold lines* on tubular bodies, at a position defined relative to another landmark | Chest "at 2,5 cm below base of armhole, measure from edge to edge" (D2); waist at a "required position" (E1); hem width (I1); trouser thigh, knee ("halfway between crotch and bottom of leg"), leg opening [ISO18890, ISO3759] | Edges: good, if contrast is good. The *constructed position* inherits the uncertainty of the anchor landmark (the armhole base, the crotch point) and of the direction in which "below" is taken |
| **P2 Seam-anchored lengths** | A seam junction at one end, an edge at the other | Front length SNP (side neck point, alias HPS) → hem (A1); CB neck seam → hem (A2); sleeve along underarm seam; inside leg crotch → hem; front/back rise "excluding the waistband" [ISO18890, ISO3759] | Seam junctions are often visible, but may be covered by a collar, hood, neck trim or placket. SNP explicitly "does not include any neck trim, collar or hood", so the defining point can sit *underneath* a collar |
| **P3 Seam-to-seam *or* edge-to-edge (declared ambiguous)** | Either seams or edges, "as agreed by interested parties" | Shoulder length (J1), back neck width (Q1), across front/back (C1/C2), armhole (M1) [ISO18890] | Depends on the agreement. Seams are raised, textured lines; edges are silhouettes |
| **P4 Curves and diagonals** | Two landmarks joined by a straight line or by the curve | Armhole (straight "shoulder point to base of armhole" by default), raglan length (M2/M3), neck drop, shoulder slope measured "vertically" along an imaginary line (J2) [ISO18890] | The landmarks are the problem; a straight-line evaluation is easy once they are fixed |
| **P5 Force-defined** | The dimension under a specified extension | Waist extended, neck opening extended (ISO 18890 Annex A; GB/T 31907 "pulling size") | **Not observable without actuation.** A passive camera cannot measure these at all |
| **P6 Small details** | Short distances between construction features | Pocket width/depth (−0/+1/8 in in the Army coat), label and pocket placement, collar point, hem depth (V1) | Features are visible, but the tolerances are tiny (§8) |
| **P7 Hidden or multi-layer** | Features inside the garment or needing a former | Neckband length of a closed collar ("a suitable former can be used"), linings measured separately, inside-leg crotch point on some constructions [ISO3759] | Poor. Requires opening or re-arranging the garment, or a special fixture |

Two observations follow directly from the standards themselves [E + I]:

1. **Two ISO standards disagree on chest/body width.** ISO 3759 says that for flat garments one should "measure the width between seams" [ISO3759 §7.1.6]. ISO 18890 measures chest "from edge to edge" [ISO18890, D2]. The Army coat uses "outside folded edge to … outside folded edge" [GLPD-07-13A]. On a side-seamed T-shirt where the side seam has rolled 3–5 mm onto the front, these give different numbers for the *same garment in the same state*. On a tubular (seamless-side) knit body, "between seams" is undefined.
2. **The standard leaves some operations to agreement.** ISO 18890 says "seam to seam or edge to edge as agreed by interested parties" for shoulder, neck width and armhole. "Required positions" must be supplied by the parties ("If a required position is not specified, the selected measurements cannot be carried out") [E, ISO18890 §3.5]. So the international standard itself admits that it is not a complete definition.

---

## 3. The measurand is "garment + state + operator", not "the garment"

### 3.1 Vocabulary from general metrology

The *International Vocabulary of Metrology* defines the **measurand** as the "quantity intended to be measured". It notes that specifying it requires "describing the state of the phenomenon, body, or substance carrying the quantity" [E, VIM-2.3]. It also defines **definitional uncertainty**: the "component of measurement uncertainty resulting from the finite amount of detail in the definition of a measurand". This is "the practical minimum measurement uncertainty achievable in any measurement of a given measurand" [E, VIM-2.27].

For garments this is the crux. **If the definition of "chest width" allows several physically different answers for the same garment, no sensor and no neural network can do better than that spread.** Definitional uncertainty is not an algorithmic problem. It can only be reduced by writing a more complete definition.

### 3.2 Non-rigid parts: the state is part of the specification

Mechanical engineering met this problem with sheet-metal, rubber and plastic parts. **ISO 10579:2010** covers the dimensioning and tolerancing of *non-rigid parts*:

- A non-rigid part is one "which deforms to an extent that in the free state is beyond the dimensional and/or geometrical tolerances on the drawing".
- The free state is the "condition of a part subjected only to the force of gravity".
- Because "the extent of distortion may depend upon the orientation of the part and condition of the part in the free state", the drawing must state the conditions: "the direction of gravity, conditions in which it is to be supported, etc.".
- Alternatively the drawing specifies a **restrained condition**, i.e. the exact forces and fixtures under which the dimension applies (e.g. "mounted with 64 bolts M6 tightened to a torque of 9 N·m to 15 N·m") [E, ISO10579].

A garment is an extreme non-rigid part. Its free-state shape on a table depends on how it was placed, and on how much in-plane strain the operator put in while smoothing it. ISO 18890's "without mechanical tension" and "avoid any unattended folds, wrinkles and/or creases" are qualitative. They do not specify a support, a force limit or a flattening method in the ISO 10579 sense [I].

### 3.3 Specification operators: why "chest width" is an incomplete definition

The GPS (geometrical product specification) system treats a specification as an ordered set of operations — an **operator**. It formally distinguishes:

- **Complete operators**, which are unambiguous.
- **Incomplete operators**, which are "ambiguous and therefore introduce ambiguity of specification".

The textbook example of an incomplete specification is "the step dimension 30 ± 0,1, which does not specify the association to be used" [E, ISO17450-2]. Nielsen summarises the concept: uncertainty becomes "the universal currency for quantifying ambiguity in requirements, specifications and verifications". He lists three typical causes of specification ambiguity: "undefined or poorly defined filtering requirements", "poor definition of the direction in which the requirement applies" and "poor definition of the boundaries of the feature to which the requirement applies" [E, Nielsen-CIRP].

All three appear in garment POMs [I]:

| GPS ambiguity type | Garment instance |
|---|---|
| **Filtering** (which small-scale deviations to ignore) | Do surface fuzz, pile or a slightly wavy hem count toward the "edge"? Are small wrinkles to be smoothed out (filtered) or measured over? How far may a rib hem flare before it is "stretched"? |
| **Direction** | "2,5 cm below base of armhole": below along the centre-front line, along the side seam, or vertically in the image? "Straight across": perpendicular to which datum? "Shoulder slope measured vertically": vertical relative to what, when the garment is not aligned? |
| **Feature boundaries** | Where exactly is the "base of armhole" on a rounded underarm? Where does the SNP lie under a rib collar? Is the edge the outer fold of the fabric or the seam line? Where is the "top" of a curved waistband? |

The GPS system also offers a vocabulary for writing complete garment definitions. A POM can be defined as a sequence of the seven standard operations — **partition, extraction, filtration, association, collection, construction, evaluation** [E, Nielsen-CIRP, ISO17450-2]. For example, a complete operator for "chest width 2.5 cm below armhole" could read:

- **partition** — the left and right side edge regions
- **extraction** — sample the edge contour at ≥ N points
- **filtration** — suppress fibre fuzz shorter than 1 mm
- **association** — fit a line to each side edge over a stated segment
- **construction** — the datum is the fitted centre-front axis; the armhole base is the intersection of the fitted armhole seam curve and the fitted side edge; the measurement line is perpendicular to the datum at 25.0 mm from the armhole base
- **evaluation** — the distance between the two edge intersections along that line

**This is the single most transferable idea from precision metrology to garments** [I]. It turns a vague instruction into a machine-executable measurement recipe. It also makes the remaining ambiguity explicit, so it can be quantified.

### 3.4 Consequences for automation

Three consequences follow, each of them requirements in their own right:

1. **An automated system is "another measurement device".** ISO 18890 allows it only if it "leads to the same results". That is a claim of *equivalence* with the reference method, and it has to be demonstrated by a method-comparison study (document 05) [E, ISO18890 §4.2; I for the consequence].
2. **Standard capability frameworks assume the definition is not the problem.** ISO 22514-7 (the basis of VDA 5) is "only useful if it is known that the method uncertainty and the specification uncertainty are small compared to the implementation uncertainty". It is "not suitable for complex geometrical measurement processes … that rely on several measurement points" [E, ISO22514-7]. For garments that assumption is unverified and plausibly false [H]. So the feasibility study must measure the definitional component *separately* (document 05, stage S4) rather than fold it into "gauge R&R".
3. **Tolerances were set with a particular (manual) measurement method.** Nielsen warns that when tolerances come from experience with a particular measurement practice, "it is imperative that the details of the measurement process … are communicated as part of the product specification, otherwise … the tolerance is meaningless" [E, Nielsen-CIRP]. Garment tolerances were implicitly calibrated against hand-and-tape practice. If an automated system realises a *different* operator (e.g. a true edge-to-edge distance instead of a tape laid over a slight bulge), it can be more repeatable and still **biased relative to contractual practice**. That bias must be measured and either corrected or agreed with the buyer [I].

---

## 4. Garment state: "flat, relaxed, without tension" is not a unique state

The physics is analysed in document 03. The requirement-level facts are these:

- **Knit dimensions depend on relaxation history.** Munden showed that the relaxed dimensions of plain-knit fabric are governed by the loop (stitch) length, with the relationships confirmed after both dry and wet relaxation [E, Munden1959, abstract level]. The knitting literature treats "dry-relaxed", "wet-relaxed" and "fully relaxed" as distinct reference states, i.e. the dimensions of the *same* knit depend on which relaxation it has undergone [E secondary]. ISO 3759 prescribes conditioning "for at least 4 h, or until a constant mass is achieved" (≤ 0.25 % mass change per hour), with garments hung on a hanger [E, ISO3759 §6.3, §7.1.4]. In ISO 18890 conditioning is optional [E, ISO18890 §4.3.1]. Freshly pressed, folded, warm or humid production garments are in none of these states [I].
- **Fabric–table friction and fabric hysteresis lock in whatever strain the operator introduced while smoothing.** "Without tension" is therefore history-dependent, not a unique physical state [I; magnitude is [H], see document 03].
- **Flat-lay silhouettes of tubular garments are fold lines.** Where the fold falls relative to the side seam depends on how the garment was laid (§2.2) [I].

**Requirement consequence.** The measurement state must be defined *operationally* by a written lay protocol: support surface, orientation, smoothing method, and optionally a flattening device or tension. The reproducibility of that state must be measured as its own uncertainty component. This is the ISO 10579 lesson applied to textiles.

---

## 5. Ground truth: design pattern vs sewn garment vs measured garment

### 5.1 The pattern is the design intent, not the truth

The design geometry (2D pattern pieces, grading rules, spec sheet) and the physical garment differ systematically and randomly:

- Pattern pieces carry seam allowances, so finished dimensions come from *sewn seam lines*, not cut edges. The Army coat's purchase description, for instance, notes that if a 5-thread safety stitch replaces the double-needle seam, the contractor "must adjust patterns for 3/8-inch gage in lieu of 1/2-inch" [E, GLPD-07-13A].
- Sewing introduces take-up, ease and puckering. Pressing and garment treatments (washing, dyeing, finishing) introduce shrinkage or growth. Purchase descriptions therefore require manufacturers to "compensate for actual fabric shrinkage or growth after the application of garment treatment" [E, GLPD-14-04A §3.4.1.2]. Shrinkage allowances are routinely engineered into patterns, e.g. for washed denim [E practitioner-level only; magnitudes vary by fabric and wash and are not generalised here].
- Random production variation (ply shift in cutting, operator handling at the machine) adds scatter.

**Conclusion [I].** The design (CAD) geometry is the **nominal** — exactly what QC compares against. It cannot also be the ground truth for the measured value. CAD is still very useful as a *prior*: it says roughly where features should be, which features exist, and what the garment topology is. But any algorithm that pulls measurements towards the CAD geometry biases results towards "in tolerance" and hides the very defects QC is meant to find (document 03 §6, document 04, layer A8).

### 5.2 Three kinds of "truth" — the requirement must say which one

| Kind | Definition | Role |
|---|---|---|
| **Conventional value** | What a competent inspector following the contractual procedure (ISO 18890 or the buyer's How-to-Measure) would obtain, averaged over inspectors | What contracts and disputes refer to. The automated system must be **equivalent** to it or agree its bias |
| **Reference value** | Value from a reference measurement procedure with stated, smaller uncertainty: marked endpoints (as ISO 3759 already allows), a controlled lay, a calibrated instrument, the same lay as the system under test | What validation uses to estimate the system's bias and precision (document 05) |
| **Design nominal** | CAD / spec-sheet value | What conformity is judged against (nominal ± tolerance). It is *not* a measurement |

The previous team reportedly struggled with reference points. A central requirement of this study is therefore that **every accuracy number states which of these it was measured against, and how uncertain that reference was.**

---

## 6. Tolerances and QC practice (published, generic)

No internal buyer data were available; the values below are from public documents and are generic.

| Source | Type | Tolerances |
|---|---|---|
| US Army ACU coat, GL/PD 07-13A and 14-04A [E] | Government purchase description | **±1/2 in (±12.7 mm)** for half chest, back length, sleeve length, shoulder, half sweep, half sleeve opening. Chest pocket width and depth **−0/+1/8 in** (a 3.2 mm window). Hook-and-loop widths ±1/32 in. Half-chest grade step **2 in** between sizes |
| QIMA inspection guidance [E, practitioner] | Third-party inspection company | Tolerances "vary for different parts of the garment, depending on their significance"; example of ¼ in on chest width. Measurement checks are "done for every clothing size in your order" |
| Practitioner summaries [weak] | Blogs, tech-pack guides | Typically ±0.5 cm for small details up to ±1–1.5 cm for body dimensions. One academic paper asserts "~2 cm" is acceptable in industry, citing another paper [Paulauskaite2022] — treat as anecdotal |

The Army documents also show the **sampling context**: "Sampling for inspection shall be performed in accordance with ANSI/ASQ Z1.4", and finished measurements are part of conformance inspection [E, GLPD-07-13A §4.3].

**Tolerance vs grade step.** The Army coat uses a half-chest tolerance of ±1/2 in against a 2 in grade: the tolerance band (1 in wide) is half the grade, so adjacent sizes cannot overlap. A common heuristic is "tolerance = half the grade"; practitioners criticise it because of stack-up [E practitioner, FashionIncubator]. For measurement design, the grade step sets an *outer* bound: a system that cannot reliably tell adjacent sizes apart is useless. The tolerance sets the *operative* bound [I].

---

## 7. A measurement vocabulary for this problem

"95 % accurate" has no meaning until the following are specified [E for the general definitions: VIM-2.3, VIM-2.27, JCGM106, ISO 5725 as referenced by ISO22514-7; I for the garment-specific operationalisation].

| Term | Operational meaning in this project |
|---|---|
| **Measurand** | A specific POM defined by a complete operator (§3.3), on a garment in a specified state (§4) |
| **Reference value** | Result of the reference procedure (§5.2) on the *same* garment in the *same* lay |
| **Error / bias (trueness)** | Measured − reference; bias is the mean error per POM, garment type and fabric |
| **Repeatability** (static) | Spread of repeated captures of the *same lay*, without touching the garment. Isolates the sensor, calibration and algorithm noise |
| **Intermediate precision** (re-lay) | Spread when the *same operator* lifts and re-lays the same garment. Adds garment-state variability |
| **Reproducibility** | Spread across operators, stations, days, and ideally factories |
| **Definitional uncertainty** | Spread of expert placements of the POM endpoints on a *fixed* lay (physical marking and on-image). The floor for any system |
| **Measurement uncertainty (combined, expanded)** | GUM-style combination of all components; expanded uncertainty U = k·u (k = 2) [JCGM106] |
| **Linearity** | Bias as a function of size (XS → XXL) and garment dimension |
| **Stability / drift** | Change of bias and precision over days and weeks, after re-calibration and hardware change |
| **Gross-error rate** | Fraction of measurements with \|error\| above a stated bound (e.g. half the tolerance). Essential for learned components, whose errors are heavy-tailed [I] |
| **Coverage / abstention** | Fraction of garments or POMs the system declines to measure (routes to a human). A system that abstains on hard cases and is accurate on the rest may be industrially excellent |
| **Decision performance** | False-accept and false-reject rates, both per item ("specific") and averaged over production ("global") [E, JCGM106] |

---

## 8. From tolerance to required uncertainty

### 8.1 What the standard criteria demand

Common industrial criteria:

- **AIAG MSA %GRR** = 6·σ_GRR / T: ≤ 10 % acceptable, 10–30 % may be acceptable depending on application, > 30 % unacceptable; plus ndc (number of distinct categories) ≥ 5 [E secondary, AIAG-MSA].
- **VDA 5 / ISO 22514-7** capability ratios Q = 2U/T: Q_MS (measuring system) ≤ 15 % and Q_MP (whole measurement process) ≤ 30 %; resolution ≤ 5 % of T [E for the ISO 22514-7 framework; limit values E secondary, VDA5-QDAS].
- **JCGM 106 measurement capability index** C_m = T/(4u) = T/(2U). "Cm ≥ 4 means that u_m ≤ T/16"; rules such as a 4:1 test uncertainty ratio "are sometimes ambiguously or incompletely defined" [E, JCGM106 §7.6].

Here T is the full tolerance width (2t for ±t). Converted to the largest allowable standard deviation or standard uncertainty, in millimetres [I; arithmetic from the definitions above]:

| Tolerance | T (mm) | σ for %GRR 10 % | σ for %GRR 30 % | u for Q_MS 15 % | u for Q_MP 30 % | u for C_m ≥ 4 | Resolution 5 % |
|---|---|---|---|---|---|---|---|
| ±1/8 in (and a −0/+1/8 in pocket has T = 3.2 mm, i.e. half these values) | 6.4 | 0.11 | 0.32 | 0.24 | 0.48 | 0.40 | 0.32 |
| ±5 mm | 10.0 | 0.17 | 0.50 | 0.38 | 0.75 | 0.62 | 0.50 |
| ±1/4 in | 12.7 | 0.21 | 0.63 | 0.48 | 0.95 | 0.79 | 0.64 |
| ±10 mm | 20.0 | 0.33 | 1.00 | 0.75 | 1.50 | 1.25 | 1.00 |
| ±1/2 in | 25.4 | 0.42 | 1.27 | 0.95 | 1.90 | 1.59 | 1.27 |
| ±15 mm | 30.0 | 0.50 | 1.50 | 1.12 | 2.25 | 1.88 | 1.50 |

How to read the table [I]:

- **Resolution is not the bottleneck.** Every camera considered in document 04 samples finer than 0.5 mm per pixel.
- **The 10 % %GRR criterion is almost certainly unreachable for textile body dimensions by *any* method.** It would require the total spread across repeat lays and operators to be below about 0.2–0.4 mm for typical tolerances, on a material that deforms by millimetres under hand smoothing (document 03). This is a hypothesis to be confirmed [H]. Even the *manual* reference method has never been shown, in the sources found, to meet it.
- **The 30 % %GRR / Q_MP 30 % band corresponds to σ or u of roughly 0.6–1.9 mm** for ±1/4 in to ±1/2 in tolerances. This is the plausible target zone for body POMs, and whether it can be met is the central experimental question [H].
- **Small-detail POMs with ±1/8 in or −0/+1/8 in tolerances need sub-0.3 mm uncertainty.** They should be treated as a separate class. Some may be better handled as attribute (go/no-go) checks or by fixtures than by a general measurement system [I].

### 8.2 Why the classical thresholds are the wrong primary criterion

The %GRR thresholds were developed for rigid parts, where the definition is sharp and implementation uncertainty dominates. ISO 22514-7 itself restricts its scope to that situation [E, ISO22514-7]. Wheeler shows that the "percentages" of %GRR are not proportions. He argues that a 30 % %GRR still passes about 95 % of the production signal ("when the %GRR value is 30%, any signal coming from the production process will show up at 95.4 percent of full strength") [E, Wheeler189].

For garments, the requirement should therefore rest on three benchmarks rather than one threshold [I]:

1. **Relative to current practice.** The automated system should not be worse than the manual method as it is actually practised, on the same garments in the same states. That means the manual method must be measured too; no public data on garment-inspector reproducibility with an MSA design were found. The only direct evidence located is an informal experiment with 20 untrained people: errors up to 3.02 cm on jacket length, even with instruction [E, Paulauskaite2022]. That says little about trained inspectors [H].
2. **Relative to decision risk** (next section). This requires knowing how production actually varies relative to tolerance.
3. **Relative to process-control usefulness.** Can the system resolve real production variation (ndc, or the intraclass correlation coefficient)?

---

## 9. Decision risk: how uncertainty becomes wrong pass/fail decisions

JCGM 106 defines:
- **global consumer's risk** as the "probability that a non-conforming item will be accepted based on a future measurement result";
- **global producer's risk** as the "probability that a conforming item will be rejected based on a future measurement result".

The default guarded-acceptance rule of ISO 14253-1 shrinks the acceptance zone by U = 2u on each side. This caps the probability of accepting a nonconforming item at 2.3 % for any accepted measurement [E, JCGM106 §8.3].

The table below is an illustrative calculation, not data [I]. The true value of a POM is normally distributed with process capability C_p = t/(3σ_p), either centred or offset by half the tolerance. Measurement error is normal with standard uncertainty u. Rates are per measured item.

| Process | u / t | Decision rule | Truly out of tolerance | Falsely accepted | Falsely rejected | Share of bad items accepted |
|---|---|---|---|---|---|---|
| C_p = 1.0, centred | 0.10 | simple (accept if within ±t) | 0.27 % | 0.07 % | 0.20 % | 24 % |
| C_p = 1.0, centred | 0.20 | simple | 0.27 % | 0.09 % | 0.83 % | 34 % |
| C_p = 1.0, centred | 0.20 | guarded (w = U) | 0.27 % | 0.003 % | 12 % | 1 % |
| C_p = 1.0, mean at +t/2 | 0.10 | simple | 6.7 % | 1.2 % | 2.0 % | 18 % |
| C_p = 1.0, mean at +t/2 | 0.20 | simple | 6.7 % | 1.8 % | 5.1 % | 27 % |
| C_p = 1.0, mean at +t/2 | 0.20 | guarded (w = U) | 6.7 % | 0.05 % | 33 % | 0.7 % |
| C_p = 0.67, centred | 0.20 | simple | 4.4 % | 1.1 % | 2.9 % | 24 % |
| C_p = 0.67, centred | 0.30 | simple | 4.4 % | 1.3 % | 5.4 % | 30 % |

For a ±1/2 in POM, u/t = 0.2 means u ≈ 2.5 mm. For a ±1/4 in POM it means u ≈ 1.3 mm.

What the calculation shows [I]:

- **With simple acceptance, a large share of out-of-tolerance garments are accepted even at modest uncertainty.** Most bad garments are only slightly out, and slightly-out items are hard to separate from slightly-in ones. Whether that matters depends on how critical a "slightly out" garment is. That is a commercial question, not a metrology one.
- **Guard bands trade consumer's risk for producer's risk very steeply** once u/t exceeds about 0.1. At u/t = 0.2 a guarded rule rejects 12–33 % of good garments. That is unacceptable in production, so guard-banding is only viable if u/t is small.
- **The required uncertainty depends on the unknown production distribution.** No public data on within-style production variation relative to tolerance was found. (Kinley's widely cited study measured 1,011 women's pants, but across brands and price points, not within a style [E, Kinley2003].) Measuring the production distribution is part of the feasibility study.

---

## 10. What could "80 % accuracy" mean?

The previous team's figure is undefined. Assuming normally distributed errors, some plausible readings imply very different error levels [I]:

| Interpretation | Implied error SD per measurement | e.g. for ±1/2 in (t = 12.7 mm) |
|---|---|---|
| 80 % of individual measurements within ±t of the reference | σ ≈ 0.78 t | ≈ 9.9 mm |
| 80 % within ±t/2 | σ ≈ 0.39 t | ≈ 5.0 mm |
| 80 % of garments have *all* 10 POMs within ±t (independent errors) | σ ≈ 0.44 t | ≈ 5.6 mm |
| 80 % agreement of pass/fail decisions with manual inspection | Not determinable without the prevalence of failures | — |

Under every interpretation, the implied error is several times larger than the u ≈ 1–2 mm zone of §8. Heavy-tailed errors from a learned localiser (mostly small errors, occasionally a wrong landmark) could also produce "80 %" with quite different operational consequences. **The figure cannot be compared with anything until it is restated in the metrics of §7.** A deliverable of the feasibility study is to report, per POM and per stratum:

- bias and repeatability, intermediate-precision and reproducibility SDs;
- the 95th percentile of |error|;
- the gross-error rate and the abstention rate;
- false-accept and false-reject rates under the measured production distribution.

---

## 11. Scenario-specific requirements

| Requirement | S-AQL | S-Inline |
|---|---|---|
| Cycle time | Must beat the manual method (tape plus transcription) for the full POM set. Minutes are acceptable | Must be at or below the line takt. Takt (s) = 3600 / (garments per hour at that station). This is an input to be gathered from the factory [H] |
| Handling | A trained operator follows a written lay protocol. Guides, flattening aids and a second smoothing pass are acceptable | Minimal operator involvement. The garment may arrive folded, warm or unpressed |
| Garment state | Can be conditioned (e.g. hung for a defined time) if the buyer requires ISO 139 conditions | Uncontrolled: fresh from sewing or pressing. State variability is an explicit risk |
| POM coverage | Full spec sheet, including force-defined POMs if a stretch fixture is added | A subset: the POMs most predictive of process drift |
| Abstention | Acceptable at a moderate rate; the inspector measures by hand | Must be low, and routed cheaply |
| Output | Per-garment report plus lot decision per Z1.4 / ISO 2859-1 | SPC charts, alarms, 100 % screening decisions |
| Evidence needed | Equivalence with the contractual manual method (ISO 18890 §4.2) | Plus stability under line conditions and handling |

---

## 12. Draft requirements (to be confirmed by the feasibility study)

| ID | Requirement | Status |
|---|---|---|
| R1 | Every POM in scope has a **complete operational definition** (§3.3), agreed with at least one buyer and cross-referenced to ISO 18890 codes | Design task |
| R2 | A written **lay protocol** per garment family and scenario, with its reproducibility measured | Design + experiment |
| R3 | Performance reported in the §7 metrics, per POM × garment family × fabric class × colour class, against a **reference** with stated uncertainty | Experiment |
| R4 | **Provisional numeric targets** for body POMs with ±1/4 in to ±1/2 in tolerances: combined standard uncertainty u ≤ 0.15 T (Q_MP ≤ 30 % in VDA 5 terms), i.e. about 1–2 mm, **and** not worse than the manual method's measured reproducibility. Small-detail POMs are specified separately | Provisional; revise after S4–S7 of document 05 |
| R5 | Gross-error rate and abstention rate within agreed limits, with every abstention routed to manual measurement | To be agreed |
| R6 | Calibration traceable to a length standard (certified scale or artefact), re-verified on a schedule, with drift monitoring | Design |
| R7 | Documented **equivalence** (or agreed bias) with the contractual measurement method | Experiment (method comparison) |
| R8 | Force-defined POMs (P5) are either out of scope for passive sensing or handled by an actuated fixture with a specified force | Scope decision |

---

## Sources cited in this document

Full annotated entries in [references.md](references.md).

- `[ISO18890]` ISO 18890:2018 *Clothing — Standard method of garment measurement* (iTeh preview of §1–5.3). https://cdn.standards.iteh.ai/samples/63693/131110d2f792493aabb7200cbd9136ba/ISO-18890-2018.pdf
- `[ISO3759]` ISO 3759:2011 *Textiles — Preparation, marking and measuring of fabric specimens and garments in tests for determination of dimensional change* (iTeh preview). https://cdn.standards.iteh.ai/samples/57309/1af372753bf84b3cb0288a274429c73d/ISO-3759-2011.pdf
- `[GBT31907]` GB/T 31907-2015 *Method of garment measurement* (English summary). https://www.codeofchina.com/standard/GBT31907-2015.html
- `[GLPD-07-13A]` US Army, Purchase Description GL/PD 07-13A, *Coat, Army Combat Uniform*. https://ciehub.info/spec/PD/GL-PD-07-13A.pdf
- `[GLPD-14-04A]` US Army, Purchase Description GL/PD 14-04A, *Coat, Army Combat Uniform* (2015). https://imlive.s3.amazonaws.com/Federal%20Government/ID339162058590411927735464482663559176426/Spec%20GL-PD-14-04A.pdf
- `[VIM-2.3]`, `[VIM-2.27]` JCGM 200:2012 *International Vocabulary of Metrology*, entries 2.3 and 2.27. https://jcgm.bipm.org/vim/en/2.3.html · https://jcgm.bipm.org/vim/en/2.27.html
- `[ISO10579]` ISO 10579:2010 *GPS — Dimensioning and tolerancing — Non-rigid parts* (iTeh preview). https://cdn.standards.iteh.ai/samples/54707/4d70c7544c894dce8a99e89b862cdf09/ISO-10579-2010.pdf
- `[ISO17450-2]` ISO 17450-2:2012 *GPS — General concepts — Part 2* (iTeh preview). https://cdn.standards.iteh.ai/samples/53629/2ee7b9b5e346412f87828f9adfe5e4be/ISO-17450-2-2012.pdf
- `[Nielsen-CIRP]` H. S. Nielsen, *Specifications, operators and uncertainties*. https://www.hn-metrology.com/papers/CIRP%20Specifications%20operators%20and%20uncertainties.pdf
- `[ISO22514-7]` ISO 22514-7:2021 *Capability of measurement processes* (iTeh preview). https://cdn.standards.iteh.ai/samples/iso/iso-22514-7-2021/087b2771a1074737a409ba3501bb3329/iso-22514-7-2021.pdf
- `[JCGM106]` JCGM 106:2012 *The role of measurement uncertainty in conformity assessment*. https://www.bipm.org/documents/20126/2071204/JCGM_106_2012_E.pdf
- `[AIAG-MSA]` AIAG *Measurement Systems Analysis*, 4th ed. (criteria via secondary source). https://www.spcforexcel.com/knowledge/measurement-systems-analysis-gage-rr/acceptance-criteria-for-msa/
- `[VDA5-QDAS]` Q-DAS/Hexagon knowledge base, *Capability analysis according to VDA 5*. https://nexus.hexagon.com/community/public/q-das/w/knowledge-base/16452/capability-analysis-according-to-vda-5
- `[Wheeler189]` D. J. Wheeler, *An Honest Gauge R&R Study*. https://www.spcpress.com/pdf/DJW189.pdf
- `[Munden1959]` D. L. Munden, "The geometry and dimensional properties of plain-knit fabrics", *J. Textile Institute Transactions* 50(7), 1959. https://www.tandfonline.com/doi/abs/10.1080/19447025908659923
- `[QIMA-blog]` QIMA, *Garment quality control & inspection procedures*. https://www.qima.com/blog/inspection/garment-quality-inspection-procedures
- `[FashionIncubator]` K. Fasanella, *How to develop sewing tolerances*. https://fashion-incubator.com/how-to-develop-sewing-tolerances/
- `[Paulauskaite2022]` A. Paulauskaite-Taraseviciene et al., *Applied Sciences* 12(9):4470, 2022. https://doi.org/10.3390/app12094470
- `[Kinley2003]` T. R. Kinley, "Size variation in women's pants", *Clothing and Textiles Research Journal* 21(1), 2003. https://journals.sagepub.com/doi/10.1177/0887302X0302100103

*Calculations in §8–§10 are reproducible from the formulas given: %GRR = 6σ/T; Q = 2U/T with U = 2u; C_m = T/(4u); and risk integrals as in JCGM 106 §9 with normal priors.*
