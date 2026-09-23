# 01 — Problem Formulation and Metrological Requirements

## 1. Reframing Garment QC: From Computer Vision to Dimensional Metrology

In industrial manufacturing, quality control (QC) is not a visual recognition task; it is a **conformity assessment process** governed by mathematical decision rules. 

A garment factory does not need an algorithm that estimates pixel coordinates of "keypoints". It requires:
1. **Conformity Decisions:** A binary determination for each Point of Measure (POM) on an individual garment: *Conforming* ($y \in [LSL, USL]$) or *Non-Conforming* ($y \notin [LSL, USL]$), where $LSL$ and $USL$ are Lower and Upper Specification Limits, rolled up into an acceptance/rejection decision for an inspection lot.
2. **Statistically Valid Measurement Data:** Continuous dimensional data with well-characterized, traceable measurement uncertainty ($u_c$) to drive Statistical Process Control (SPC)—detecting cutting-table drift, sewing line tension anomalies, or laundering shrinkage variations before entire production runs fall out of specification.

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│ Produced Garment│ ──► │  Physical State │ ──► │ Transduced Image│
│  (Viscoelastic) │     │ (Lay / Restraint│     │  / Point Cloud  │
└─────────────────┘     └─────────────────┘     └─────────────────┘
                                                         │
                                                         ▼
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   Conformity    │ ◄── │    Evaluated    │ ◄── │ Feature & Datum │
│ Decision (Risk) │     │ Dimension (GPS) │     │  Construction   │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

The measuring chain consists of five distinct physical and mathematical transformations:
- **Physical realization:** The garment is transformed from an unconstrained 3D shape into a 2D or semi-constrained measurement state.
- **Sensing:** Photons reflected or transmitted from the textile are focused by optics onto an array sensor and digitized.
- **Feature extraction:** Discontinuities in radiance or depth are localized as edges, ridges, seams, and folds.
- **Semantic identification:** Extracted geometric features are associated with functional garment entities (e.g., distinguishing the front neck drop from the back collar edge).
- **Geometric construction & evaluation:** A formal metrological operator evaluates distances along defined vectors between constructed datums.
- **Conformity decision:** The evaluated measurand is compared against tolerance intervals under an established decision rule (JCGM 106:2012 / ISO 14253-1).

Every link in this chain contributes variance. The reported failure of previous computer-vision systems—where detected points were "slightly displaced from intended locations"—is the inevitable result of treating this multi-stage physical chain as a direct image-to-coordinate regression problem.

---

## 2. What is Being Measured: Points of Measure (POMs) and Standards

### 2.1 Authoritative Garment Measurement Standards

To determine whether an automated measurement system is viable, the measurand must be defined against international textile standards:

- **ISO 18890:2018 (*Clothing — Standard method of garment measurement*):** Standardizes reference measurement points and methodologies for apparel QC. Key procedural stipulations include:
  - *Apparatus:* A flat, rigid table large enough to support the garment without overhang, and a flexible, non-stretch steel or reinforced tape graduated in millimetres `[E, ISO18890 §4.1]`.
  - *State Preparation:* Garments must be laid "flat, in an ambient environment and without mechanical tension", avoiding "any unattended folds, wrinkles and/or creases" `[E, ISO18890 §4.3.1]`. Knitted garments must be in a "relaxed state".
  - *Alternative Devices:* Crucially, Clause 4.2 states: *"Any measurement device which leads to the same results may be used. The use of such measurement device should be reported."* `[E, ISO18890 §4.2]`. This establishes that automated vision systems are legally and contractually classified as **alternative measurement devices requiring proof of equivalence**.
  - *Tolerances:* ISO 18890 specifies *how* to measure, but explicitly does **not** specify manufacturing tolerances. Acceptance tolerances remain purely contractual.

- **ISO 3759:2011 (*Textiles — Preparation, marking and measuring of fabric specimens and garments in tests for determination of dimensional change*):** Stipulates measurement between defined points, "preferably at seams or between points where seams meet", with garments conditioned to ISO 139 atmosphere ($20^\circ\text{C}, 65\%\text{ RH}$) until mass equilibrium is reached ($\le 0.25\%/\text{hour}$) `[E, ISO3759 §6.3, §7.1]`.

- **National and Military Purchase Specifications (e.g., US Army GL/PD 07-13A, GL/PD 14-04A):** Provide explicit contractually binding measurement rules. For example, half-chest is measured "from outside folded edge to opposite outside folded edge, 1 inch below armhole seam", with strict tolerances ($\pm 1/2'' \approx \pm 12.7\text{ mm}$) and tight detail tolerances (e.g., pocket dimensions $-0 / +1/8'' \approx -0 / +3.18\text{ mm}$) `[E, GLPD-07-13A]`.

### 2.2 Formal Conflict Between Standards

A critical finding from analyzing these standards is that **international standards disagree on fundamental definitions**:
- For chest width on a flat garment, **ISO 3759 Clause 7.1.6** mandates measuring *"between seams"*.
- In direct contrast, **ISO 18890 (Code D2)** specifies chest width *"at 2.5 cm below base of armhole, measure from edge to edge"*.
- On a circular (tubular) knit T-shirt without side seams, "between seams" is physically undefined. On a side-seamed garment where handling or knitting spirality causes the side seams to twist or roll $3\text{ mm to }5\text{ mm}$ forward onto the front panel, "between seams" and "edge to edge" diverge by **$6\text{ mm to }10\text{ mm}$ on the exact same garment in the exact same state** `[I]`.

An automated inspection system cannot be developed without first specifying which standard governs the contract.

### 2.3 Metrological Taxonomy of Garment Features

For sensing system design, apparel dimensions cannot be categorized merely as "tops vs bottoms". They must be categorized by the **physical observability of their boundary features**:

| Class | Geometric Definition | Representative POMs | Observability in Planar Flat-Lay | Sensing & Algorithmic Challenge |
|---|---|---|---|---|
| **P1: Constructed Edge-to-Edge** | Distance between two silhouette boundaries along an axis constructed from an anchor landmark | Chest width ($2.5\text{ cm}$ below armhole), Waist width, Thigh width ($2.5\text{ cm}$ below crotch) | Silhouette edges: Excellent contrast. Constructed position: Ambiguous | Inherits the localization uncertainty of the anchor landmark and the angular error of the datum axis. |
| **P2: Seam-Anchored Lengths** | Distance from a seam intersection to a silhouette edge or second seam | Front Length (SNP to hem), Inseam (crotch seam to leg hem), Back Rise | Seam intersection: Moderate to Poor. Edge: Excellent | Side Neck Point (SNP) often sits beneath collars or rib trims. Inseam crotch sits inside overlapping fabric folds. |
| **P3: Seam-to-Seam (Declared Ambiguous)** | Distance between two internal seam stitch lines | Across Shoulder, Across Back, Back Neck Width | Seams visible as surface ridges or thread contrast | Extremely sensitive to camera angle, lighting azimuth, and thread-to-fabric contrast. Ambiguous whether edge or seam governs. |
| **P4: Curved & Diagonal POMs** | Path length along a seam trajectory or straight diagonal between landmarks | Armhole circumference / straight, Raglan sleeve length, Neck drop | Curved trajectory: Moderate. Endpoints: Variable | Requires continuous contour tracking along stitch lines; sensitive to out-of-plane puckering. |
| **P5: Force-Defined (Extended)** | Distance under a specified tensile load or "maximum extension without breakage" | Waist extended (elastic), Neck stretch, Leg opening stretch | **Zero passive observability** | Cannot be evaluated by passive optical sensing alone; requires mechanical actuation and force sensing. |
| **P6: Small Construction Details** | Dimensions of patches, pockets, plackets, and button spacing | Pocket width/depth, cuff height, label placement | High visual contrast | Absolute tolerances are minute ($\pm 1.6\text{ mm}$ to $\pm 3.2\text{ mm}$); requires high optical resolution and low distortion. |
| **P7: Multi-Layer & Occluded** | Internal structural components | Pocket bag depth, internal lining length, waistband canvas width | **Completely occluded** | Invisible to top-down optical sensors; requires multi-spectral, transmission, or physical inversion. |

---

## 3. The Measurand is "Garment + State + Operator", Not "The Garment"

### 3.1 Definitional Uncertainty (VIM 2.27)

In classical metrology, the *measurand* is the quantity intended to be measured `[E, VIM-2.3]`. VIM explicitly notes that defining a measurand requires stating the physical state of the entity carrying the quantity.

Furthermore, VIM entry 2.27 defines **definitional uncertainty** ($u_{\text{def}}$):
> *"Component of measurement uncertainty resulting from the finite amount of detail in the definition of a measurand."* `[E, VIM-2.27]`

Definitional uncertainty represents the **theoretical minimum uncertainty** that *any* instrument (human, optical CMM, or AI) can achieve. If a garment tech pack defines chest width as *"measure straight across chest 1 inch below armhole"*, but does not specify:
- Whether the line is orthogonal to the center-front placket, orthogonal to the folded edge, or connecting two points individually offset by $1''$;
- Whether the "armhole point" is the needle-hole intersection of the underarm seam, the outer fabric fold under the sleeve, or the inflection point of the sleeve silhouette;
- Whether the fabric must be smoothed with a specific contact pressure;

Then that definition permits multiple physically distinct numerical distances to satisfy the written spec. If that ambiguity span is $2\text{ mm}$, **no neural network or sensor on earth can reduce measurement uncertainty below $2\text{ mm}$**. The error is in the contract, not the camera.

### 3.2 GPS Principles: Complete vs Incomplete Specification Operators

The Geometrical Product Specification (GPS) standards (ISO 17450-2, ISO 10579) establish a formal framework for eliminating specification ambiguity `[E, ISO17450-2; Nielsen-CIRP]`.

A measurement specification is modeled as an ordered set of operations known as an **operator**:
$$\text{Specification Operator} = \text{Partition} \to \text{Extraction} \to \text{Filtration} \to \text{Association} \to \text{Collection} \to \text{Construction} \to \text{Evaluation}$$

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  Partition   │ ──► │  Extraction  │ ──► │  Filtration  │
│ (ROI Select) │     │ (Edge Sample)│     │(Fuzz Removal)│
└──────────────┘     └──────────────┘     └──────────────┘
                                                 │
                                                 ▼
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  Evaluation  │ ◄── │ Construction │ ◄── │ Association  │
│(Distance Calc│     │(Datums/Offset│     │ (Line Fit)   │
└──────────────┘     └──────────────┘     └──────────────┘
```

The GPS framework classifies specifications into:
- **Incomplete Operators:** Vague specifications that leave operations to the discretion of the inspector, introducing specification uncertainty.
- **Complete Operators:** Fully defined, deterministic sequences of operations that leave zero ambiguity.

Applied to garment metrology, an incomplete tech-pack POM must be translated into a complete, machine-executable GPS operator:

```
[Incomplete POM]: "Chest width 2.5 cm below armhole, edge to edge"

[Complete GPS Operator]:
1. Partition: Select bounding ROIs encompassing the left/right underarm junctions and side edges.
2. Extraction: Extract continuous sub-pixel edge points along the left and right body silhouettes.
3. Filtration: Apply a spatial low-pass filter (cutoff wavelength λc = 2.0 mm) to suppress fabric fuzz and knitting loop serrations.
4. Association: Fit reference lines (orthogonal distance regression) to the upper side body silhouettes.
5. Construction:
   a. Establish Primary Datum A: The fitted line of the center-front placket (or center symmetry axis).
   b. Construct Anchor Points: Locate the intersection of the underarm sleeve-body seam curve with the side edge.
   c. Construct Measurement Axis: A line oriented at exactly 90.0° to Datum A, translated vertically downward by 25.0 mm along Datum A from the left anchor point.
6. Evaluation: Compute the Euclidean distance between the intersection points of the measurement axis with the left and right filtered edge profiles.
```

Transforming garment POMs into complete GPS operators is the single most powerful and cost-effective lever in this entire research study: **it converts an undefined AI estimation problem into a deterministic geometric computation**.

### 3.3 Non-Rigid Parts Metrology: Free State vs Restrained Condition (ISO 10579)

Mechanical metrology long ago confronted the problem of inspecting flexible components (sheet metal, polymer fascias, rubber seals). **ISO 10579:2010 (*Dimensioning and tolerancing — Non-rigid parts*)** defines:
- A **non-rigid part** as a component that deforms under its own weight or low handling forces beyond its dimensional tolerances `[E, ISO10579]`.
- The **free state** as the condition where the part is subjected solely to the force of gravity. ISO 10579 notes that because distortion depends on orientation and support conditions, the drawing *must explicitly state the support conditions and orientation*.
- A **restrained condition** where specific clamping forces and fixtures are defined to bring the part into a repeatable state prior to measurement.

Textile garments are extreme non-rigid parts. A garment laid on a flat table is in a friction-dominated, hysteretic semi-free state. Because the fabric possesses low bending stiffness and low in-plane shear stiffness, hand smoothing by an inspector introduces non-uniform strain fields that are "frozen" into place by fabric-table friction. **Therefore, the garment state is an intrinsic component of the measurand.**

---

## 4. Ground Truth: Design Geometry vs Sewn Garment vs Measured State

A persistent point of confusion in automated inspection is what constitutes "ground truth". There are three fundamentally distinct geometric entities:

```
┌────────────────────────────────┐
│      1. CAD Pattern Spec       │  Nominal Design Intent (2D DXF/AAMA)
└────────────────────────────────┘
                │
                ▼ [Cutting, Sewing, Take-up, Washing, Thermal Shrinkage]
┌────────────────────────────────┐
│      2. Physical Garment       │  True Sewn Topology (3D, Viscoelastic)
└────────────────────────────────┘
                │
                ▼ [Handling, Table Friction, Lay Technique, Wrinkles]
┌────────────────────────────────┐
│      3. Measured State         │  Observable Realization (Image / Point Cloud)
└────────────────────────────────┘
```

1. **Design Nominal (CAD Pattern / Spec Sheet):** The 2D CAD pattern pieces (Gerber, Lectra, Optitex) represent design intent. They do *not* represent finished garment dimensions. Sewing induces stitch take-up, fabric feed distortion, and differential tension between needles. Washing, garment dyeing, and pressing introduce substantial, non-uniform shrinkage or growth ($2\%\text{ to }>8\%$). **The CAD spec is the benchmark for pass/fail classification, never the ground truth for sensor measurement.**
2. **Conventional Value (Contractual Ground Truth):** What a certified QC inspector produces using a calibrated steel tape following contractual buyer guidelines. Contracts, commercial dispute arbitrations, and lot chargebacks are legally bound to this value. An automated system must prove **equivalence** to this conventional value `[E, ISO18890 §4.2]`.
3. **Reference Value (Metrological Ground Truth):** The value obtained using a primary reference method possessing an expanded uncertainty substantially smaller than the system under test (e.g., $U_{\text{ref}} \le \frac{1}{4} T$). In textiles, this can only be achieved by physically marking landmark intersections on the fabric with sub-millimetre targets and measuring them in-situ.

---

## 5. Apparel Tolerances and Quality Control Reality

Apparel manufacturing operates on contractual tolerance tables negotiated between fashion brands and OEM factories. Drawing from authoritative military purchase descriptions (GL/PD 07-13A, GL/PD 14-04A), third-party inspection manuals (QIMA), and industrial tech packs, standard tolerances fall into discrete tiers:

| Dimension Category | Representative POMs | Typical Manufacturing Tolerance ($\pm t$) | Total Tolerance Band ($T = 2t$) | Typical Grade Step (Size-to-Size Delta) |
|---|---|---|---|---|
| **Major Body Lengths** | Front Length (SNP to hem), Center Back Length, Outseam | $\pm 1/2'' \ (\pm 12.7\text{ mm})$ to $\pm 3/4'' \ (\pm 19.0\text{ mm})$ | $25.4\text{ mm to }38.1\text{ mm}$ | $25.4\text{ mm to }50.8\text{ mm}$ |
| **Major Body Widths** | Chest Width, Hem Sweep, Relaxed Waist | $\pm 1/2'' \ (\pm 12.7\text{ mm})$ | $25.4\text{ mm}$ | $50.8\text{ mm}$ ($2''$ grade) |
| **Secondary Circumferences** | Sleeve Length (CB or underarm), Thigh Width, Inseam | $\pm 3/8'' \ (\pm 9.5\text{ mm})$ to $\pm 1/2'' \ (\pm 12.7\text{ mm})$ | $19.0\text{ mm to }25.4\text{ mm}$ | $25.4\text{ mm}$ |
| **Critical Fit Contours** | Neck Width, Armhole (straight), Front/Back Rise | $\pm 1/4'' \ (\pm 6.35\text{ mm})$ | $12.7\text{ mm}$ | $12.7\text{ mm}$ |
| **Construction Details** | Collar Point, Cuff Width, Pocket Placement, Flap Width | $\pm 1/8'' \ (\pm 3.18\text{ mm})$ | $6.35\text{ mm}$ | Fixed across sizes |
| **Asymmetric Tight Tolerances** | Military ACU Coat Chest Pocket Depth/Width `[GLPD-07-13A]` | $-0 / +1/8'' \ (-0 / +3.18\text{ mm})$ | **$3.18\text{ mm}$ total window** | Fixed |

### 5.1 The Tolerance vs Grade Step Principle
In garment grading, the tolerance band $T$ must always be smaller than the grade step between adjacent sizes (e.g., Size M to Size L). In standard menswear grading, the half-chest grade step is $2''$ ($50.8\text{ mm}$), while tolerance is $\pm 1/2''$ ($T = 1'' = 25.4\text{ mm}$). 
- If an automated measurement system exhibits an expanded uncertainty $U \ge 12.7\text{ mm}$, its measurement uncertainty encompasses half the grade step. 
- Under such uncertainty, a large Size S garment and a small Size M garment become statistically indistinguishable, destroying process capability.

---

## 6. Formal Metrology Framework: Precision, Trueness, and Uncertainty

Vague statements like "95% accuracy" have zero engineering validity. The metrological performance of the measurement system must be defined under **JCGM 200:2012 (VIM)**, **JCGM 100:2008 (GUM)**, and **ISO 22514-7 / VDA 5**:

```
                       TOTAL MEASUREMENT ERROR
                                  │
         ┌────────────────────────┴────────────────────────┐
         ▼                                                 ▼
   SYSTEMATIC ERROR                                  RANDOM ERROR
    (Bias / Trueness)                            (Precision / Spread)
         │                                                 │
 ┌───────┴───────┐                         ┌───────────────┴───────────────┐
 ▼               ▼                         ▼                               ▼
Scale /      Optical /               Static Repeatability        Intermediate Precision
Parallax     Algorithm Bias          (Sensor / Algorithm)            (Re-lay / Handling)
```

1. **Trueness (Bias):** The expectation of measurement errors over repeated measurements against a traceable reference standard:
   $$\text{Bias} = \bar{y} - y_{\text{ref}}$$
2. **Static Repeatability ($\sigma_{\text{rep}}$):** Standard deviation of consecutive measurements of the **identical garment lay** without touching the fabric. Isolates optical, sensor, and algorithmic noise.
3. **Intermediate Precision / Re-lay Precision ($\sigma_{\text{lay}}$):** Standard deviation of measurements when the **same operator lifts, shakes, and re-lays the same garment** following the lay protocol. Quantifies the physical garment state variability ($u_B$).
4. **Reproducibility ($\sigma_{\text{repro}}$):** Standard deviation across different operators, different inspection stations, and different environmental conditions.
5. **Combined Standard Uncertainty ($u_c$):** Quadrature sum of all independent standard uncertainty components (GUM Clause 5.1):
   $$u_c = \sqrt{u_{\text{sensing}}^2 + u_{\text{extraction}}^2 + u_{\text{state}}^2 + u_{\text{def}}^2}$$
6. **Expanded Uncertainty ($U$):**
   $$U = k \cdot u_c \quad (k = 2 \text{ for } 95.45\% \text{ coverage under normality})$$

### 6.1 Industrial Capability Thresholds: MSA, VDA 5, and JCGM 106

How small must $u_c$ or $\sigma$ be relative to the tolerance band $T$? Three primary industrial frameworks define the requirements:

- **AIAG Measurement Systems Analysis (MSA 4th Ed.):**
  $$\%GRR = \frac{6 \cdot \sigma_{MS}}{T} \times 100\%$$
  - $\%GRR < 10\%$: System is acceptable (standard for rigid automotive components).
  - $10\% \le \%GRR \le 30\%$: Conditionally acceptable based on application criticality.
  - $\%GRR > 30\%$: Strictly unacceptable.
  - Number of Distinct Categories ($ndc = 1.41 \frac{\sigma_{\text{part}}}{\sigma_{MS}} \ge 5$).

- **VDA 5 / ISO 22514-7 (*Capability of Measurement Processes*):**
  Defines capability ratios for the measuring system ($Q_{MS}$) and the complete measurement process ($Q_{MP}$):
  $$Q_{MS} = \frac{2 \cdot U_{MS}}{T} \le 15\% \implies u_{MS} \le 0.0375 \cdot T$$
  $$Q_{MP} = \frac{2 \cdot U_{MP}}{T} \le 30\% \implies u_{MP} \le 0.075 \cdot T$$
  $$Resolution \le 5\% \text{ of } T$$

- **JCGM 106:2012 Capability Index ($C_m$):**
  $$C_m = \frac{T}{4 \cdot u_c} \ge 4 \implies u_c \le \frac{T}{16} = 0.0625 \cdot T$$

### 6.2 Required Uncertainty Budget Mapped to Garment Tolerances

The table below calculates the exact maximum permissible standard uncertainty ($u$) and standard deviation ($\sigma$) required across garment tolerance classes `[I]`:

| Tolerance Spec | Total Tolerance $T$ | Required $\sigma$ for $\%GRR \le 10\%$ | Required $\sigma$ for $\%GRR \le 30\%$ | Required $u$ for $Q_{MS} \le 15\%$ | Required $u$ for $Q_{MP} \le 30\%$ | Permissible Sensor Resolution ($5\% T$) |
|---|---|---|---|---|---|---|
| **$\pm 1/8'' \ (\pm 3.18\text{ mm})$** | $6.35\text{ mm}$ | **$0.11\text{ mm}$** | **$0.32\text{ mm}$** | **$0.24\text{ mm}$** | **$0.48\text{ mm}$** | $0.32\text{ mm}$ |
| **$-0 / +1/8''$ (Pocket)** | $3.18\text{ mm}$ | **$0.05\text{ mm}$** | **$0.16\text{ mm}$** | **$0.12\text{ mm}$** | **$0.24\text{ mm}$** | $0.16\text{ mm}$ |
| **$\pm 1/4'' \ (\pm 6.35\text{ mm})$** | $12.70\text{ mm}$ | **$0.21\text{ mm}$** | **$0.64\text{ mm}$** | **$0.48\text{ mm}$** | **$0.95\text{ mm}$** | $0.64\text{ mm}$ |
| **$\pm 3/8'' \ (\pm 9.53\text{ mm})$** | $19.05\text{ mm}$ | **$0.32\text{ mm}$** | **$0.95\text{ mm}$** | **$0.71\text{ mm}$** | **$1.43\text{ mm}$** | $0.95\text{ mm}$ |
| **$\pm 1/2'' \ (\pm 12.70\text{ mm})$** | $25.40\text{ mm}$ | **$0.42\text{ mm}$** | **$1.27\text{ mm}$** | **$0.95\text{ mm}$** | **$1.90\text{ mm}$** | $1.27\text{ mm}$ |
| **$\pm 3/4'' \ (\pm 19.05\text{ mm})$** | $38.10\text{ mm}$ | **$0.64\text{ mm}$** | **$1.91\text{ mm}$** | **$1.43\text{ mm}$** | **$2.86\text{ mm}$** | $1.91\text{ mm}$ |

#### Critical Engineering Deductions from the Budget:
1. **Sensor Resolution is Never the Limiting Factor:** Every 12–61 MP modern camera satisfies the $5\%$ resolution threshold ($<0.3\text{ mm/pixel}$ across a $1.2\text{ m}$ field) `[I]`.
2. **The 10% %GRR Criterion is Physically Impossible on Fabric Bodies:** Achieving $\%GRR \le 10\%$ on a $\pm 1/2''$ dimension requires total process dispersion across operators and re-lays $\sigma \le 0.42\text{ mm}$. On a flexible, knitted textile structure whose loops shift by $1\text{ mm}$ under a light touch, this threshold cannot be achieved by *any* non-destructive measurement method, including manual tape measurement `[I/H]`. ISO 22514-7 explicitly warns that its capability thresholds assume "specification uncertainty is negligible" `[E, ISO22514-7]`.
3. **The Realistic Industrial Operational Envelope:** For body dimensions ($\pm 1/4''\text{ to }\pm 1/2''$), the achievable target envelope corresponds to the VDA 5 $Q_{MP} \le 30\%$ / AIAG $30\%$ band: **combined standard uncertainty $u_c \approx 0.8\text{ mm to }1.8\text{ mm}$** `[I]`.
4. **Detail Features Require Dedicated Gauging:** Pockets and labels with $-0/+1/8''$ limits require $u_c \le 0.24\text{ mm}$. They cannot be measured as part of a whole-garment global view; they require localized high-magnification optics or mechanical go/no-go templates `[I]`.

---

## 7. Conformity Assessment and Decision Risk (JCGM 106)

When measurement uncertainty is non-negligible, every measurement decision entails financial risk. Under **JCGM 106:2012**, conformity assessment defines two forms of risk:
- **Consumer's Risk ($R_C$ / False Accept):** The probability that a non-conforming garment is accepted by the measurement system and shipped to the client, risking brand chargebacks or returns.
- **Producer's Risk ($R_P$ / False Reject):** The probability that a conforming garment is incorrectly rejected by the measurement system, causing unnecessary rework or scrap.

```
       CONFORMITY ASSESSMENT RISK ZONES (JCGM 106)

                 LSL                           USL
                  │                             │
Non-Conforming    │         Conforming          │    Non-Conforming
(Defect Zone)     │      (Acceptance Zone)      │     (Defect Zone)
──────────────────┼─────────────────────────────┼──────────────────
           ▲      │                             │      ▲
           │◄────►│                             │◄────►│
           │  U   │                             │  U   │
           │      │                             │      │
      False Accept│                             │False Accept
         Zone     │                             │    Zone
```

Let production variance be $\sigma_p$ and process capability index be $C_p = \frac{T}{6\sigma_p}$. Let measurement error be normally distributed with standard uncertainty $u$. Under a simple acceptance rule (accept if $y \in [-t, +t]$), the joint distribution yields the global risks:

$$R_C = \int_{|x| > t} \left[ \int_{-t}^{t} \frac{1}{\sqrt{2\pi}u} e^{-\frac{(y-x)^2}{2u^2}} dy \right] \frac{1}{\sqrt{2\pi}\sigma_p} e^{-\frac{(x-\mu_p)^2}{2\sigma_p^2}} dx$$

$$R_P = \int_{-t}^{t} \left[ \int_{|y| > t} \frac{1}{\sqrt{2\pi}u} e^{-\frac{(y-x)^2}{2u^2}} dy \right] \frac{1}{\sqrt{2\pi}\sigma_p} e^{-\frac{(x-\mu_p)^2}{2\sigma_p^2}} dx$$

### 7.1 Quantitative Impact of Uncertainty on Decision Errors
Monte Carlo numerical integration of the JCGM 106 risk equations demonstrates the severe operational penalty of measurement uncertainty `[I]`:

| Process State | Relative Uncertainty ($u/t$) | Decision Rule | True Manufacturing Scrap Rate | Falsely Accepted Rate ($R_C$) | Falsely Rejected Rate ($R_P$) | Fraction of Out-of-Spec Shipped |
|---|---|---|---|---|---|---|
| $C_p = 1.0$, Centered | $0.10$ ($u = 1.27\text{ mm}$ for $\pm 1/2''$) | Simple ($\pm t$) | $0.27\%$ | $0.07\%$ | $0.20\%$ | $25.9\%$ |
| $C_p = 1.0$, Centered | $0.20$ ($u = 2.54\text{ mm}$ for $\pm 1/2''$) | Simple ($\pm t$) | $0.27\%$ | $0.09\%$ | $0.83\%$ | $33.3\%$ |
| $C_p = 1.0$, Centered | $0.20$ | Guarded ($w = 2u$) | $0.27\%$ | **$0.003\%$** | **$12.4\%$** | **$1.1\%$** |
| $C_p = 1.0$, Off-center ($\mu = +t/2$) | $0.10$ | Simple ($\pm t$) | $6.68\%$ | $1.18\%$ | $2.01\%$ | $17.7\%$ |
| $C_p = 1.0$, Off-center ($\mu = +t/2$) | $0.20$ | Simple ($\pm t$) | $6.68\%$ | $1.78\%$ | $5.12\%$ | $26.6\%$ |
| $C_p = 1.0$, Off-center ($\mu = +t/2$) | $0.20$ | Guarded ($w = 2u$) | $6.68\%$ | **$0.05\%$** | **$33.2\%$** | **$0.7\%$** |
| $C_p = 0.67$ (Marginal), Centered | $0.20$ | Simple ($\pm t$) | $4.55\%$ | $1.12\%$ | $2.94\%$ | $24.6\%$ |

#### Critical Insights for Garment QC:
- **Guard Banding Destroys Production Throughput at High Uncertainty:** ISO 14253-1 recommends guard bands ($w = U = 2u$) to reduce consumer risk below $2.3\%$. However, if $u/t = 0.20$ ($u = 2.54\text{ mm}$ on $\pm 1/2''$), applying a guard band causes the system to **falsely reject $12\%\text{ to }33\%$ of perfectly good production garments**! In apparel manufacturing, where gross profit margins are $5\%\text{ to }12\%$, such false rejection rates are economically catastrophic.
- **Uncertainty Must Be Driven Below $0.15 \cdot t$:** Guard bands are only commercially viable if $u/t \le 0.10\text{ to }0.15$ ($u \le 1.3\text{ mm to }1.9\text{ mm}$ for $\pm 1/2''$).

---

## 8. Deconstructing "80% Accuracy": What Did the Previous Team Experience?

The previous team reported achieving "~80% accuracy" before stalling. In metrology, "accuracy" without a formal definition is meaningless. We can mathematically model four plausible interpretations of what "80% accuracy" actually meant in practice `[I]`:

```
┌────────────────────────────────────────────────────────────────────────┐
│                   FOUR INTERPRETATIONS OF "80% ACCURACY"               │
├────────────────────────────────────────────────────────────────────────┤
│ 1. 80% of individual measurements fall within tolerance band (±t)       │
│    ⟹ Error standard deviation: σ ≈ 0.78 · t (9.9 mm for ±1/2")        │
├────────────────────────────────────────────────────────────────────────┤
│ 2. 80% of garments pass all 10 POMs independently                      │
│    ⟹ Per-POM pass rate: (0.80)^(1/10) ≈ 97.8%                          │
│    ⟹ Error standard deviation: σ ≈ 0.44 · t (5.6 mm for ±1/2")        │
├────────────────────────────────────────────────────────────────────────┤
│ 3. 80% binary agreement with human inspector pass/fail decisions       │
│    ⟹ Dominated by lot failure prevalence and decision boundary noise   │
├────────────────────────────────────────────────────────────────────────┤
│ 4. Mean Absolute Error (MAE) relative to tolerance                     │
│    ⟹ In a heavy-tailed error regime, hides catastrophic gross misses    │
└────────────────────────────────────────────────────────────────────────┘
```

Under every single statistical interpretation, an "80% accurate" system exhibits standard measurement errors of **$\sigma \approx 5.6\text{ mm to }10\text{ mm}$**. 

Against a $\pm 12.7\text{ mm}$ tolerance, this means the previous system's measurement dispersion was **3 to 5 times larger** than the permissible threshold ($u_c \le 1.9\text{ mm}$). It explains why their keypoints appeared "slightly displaced": the displacement was not a minor sub-pixel artifact, but a macroscopic localization error on the order of centimeters.

---

## 9. Operational Scenarios: S-AQL vs S-Inline

An automated measurement system must be evaluated within its intended operational deployment scenario:

| Operational Dimension | S-AQL (End-of-Line Quality Audit) | S-Inline (100% Process Screening) |
|---|---|---|
| **Inspection Sampling Basis** | ISO 2859-1 / ANSI/ASQ Z1.4 Lot Sampling (e.g., Level II, Normal, AQL 2.5/4.0). 20 to 125 garments inspected per batch. | 100% inspection of every garment exiting sewing, washing, or pressing. |
| **Cycle Time (Takt Time) Budget** | **2 to 5 minutes per garment** is acceptable. Focus is on metrological precision and automated digital reporting. | **5 to 15 seconds per garment** maximum. Must match the conveyor or sewing line takt time. |
| **Physical Garment Handling** | A trained technician follows an ergonomic, repeatable lay protocol (smoothing passes, alignment lines, edge flattening). | Minimal or automated handling. Garment may arrive warm, crumpled, or unpressed on a conveyor. |
| **Acceptable System Abstention Rate** | **5% to 15% abstention is acceptable.** If the system flags an ambiguous edge, the inspector measures that POM with a tape. | **< 1% abstention required.** Rejected garments must be automatically diverted without jamming the line. |
| **POM Scope** | Complete tech-pack spec sheet (12 to 25 POMs, including internal and back POMs). | 3 to 5 critical process-control POMs (e.g., chest width, total length, sleeve length) predictive of cutting/sewing drift. |
| **Primary Economic Value** | Elimination of manual transcription errors, instant buyer compliance audits, reduction of inspection labor by 70%. | Immediate detection of cutting stack errors or wash shrinkage drift before thousands of units are ruined. |

---

## 10. Quantitative Technical Requirements Specification

Based on this metrological analysis, the automated garment measurement system must satisfy the following explicit engineering requirements:

| ID | Requirement Statement | Target Metric & Method |
|---|---|---|
| **REQ-1: Complete Operational Definitions** | Every POM in scope must possess a complete, unambiguous GPS operator covering Partition, Extraction, Filtration, Association, Construction, and Evaluation. | Zero subjective human interpretation in geometric execution. Cross-referenced to ISO 18890. |
| **REQ-2: Combined Standard Uncertainty ($u_c$)** | For major body dimensions ($\pm 1/2'' \approx \pm 12.7\text{ mm}$), system combined uncertainty must not exceed: | $u_c \le 1.50\text{ mm}$ ($U \le 3.0\text{ mm}, k=2$). |
| **REQ-3: Tight-Tolerance POM Uncertainty** | For fit-critical contours ($\pm 1/4'' \approx \pm 6.35\text{ mm}$), system combined uncertainty must not exceed: | $u_c \le 0.80\text{ mm}$ ($U \le 1.6\text{ mm}, k=2$). |
| **REQ-4: Equivalence to Manual Baseline** | Measurement trueness and reproducibility must be non-inferior to certified manual QC inspectors on the same physical lay. | Two One-Sided Equivalence Test (TOST) against margin $\delta = 0.20 \cdot t$. |
| **REQ-5: Selective Abstention & Gross Error Capping** | The algorithm must output an epistemic confidence/quality score. Gross error rate ($|e| > t/2$) on accepted measurements must be strictly bounded. | Gross error rate $P(|e| > t/2) \le 0.5\%$; Abstention rate $\le 10\%$ in S-AQL. |
| **REQ-6: Traceable Scale & Calibration** | System spatial calibration must be verified against physical reference standards traceable to national length standards. | Verified via in-situ calibrated grid/scale bars certified to ISO/IEC 17025. |
| **REQ-7: Audit of Prior Shrinkage** | Algorithmic priors (CAD templates, shape constraints) must not attenuate true out-of-tolerance manufacturing defects. | Regression slope $\beta \in [0.95, 1.05]$ when evaluating seeded defects. |
| **REQ-8: Cycle Time** | End-to-end capture, feature extraction, GPS evaluation, and decision output. | $\le 10\text{ seconds}$ computation time; handling $\le 45\text{ seconds}$ in S-AQL. |
