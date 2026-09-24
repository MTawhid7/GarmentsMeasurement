# 04 — Candidate Measurement Architectures and Comparative Evaluation

## 0. The Universal Three-Layer Architectural Decoupling

A primary conclusion from the error decomposition in Document 03 and the critique of Claude's work in Chapter 00 is that **different error sources require fundamentally different engineering remedies**:
- Definitional ambiguity ($u_A$) and garment state ($u_B$) are governed by **contracts, protocols, and physical fixturing**.
- Sensing ($u_C$) and feature boundary extraction ($u_D$) are governed by **optics, sensor geometry, and illumination physics**.
- Semantic identification ($u_E$) is governed by **machine learning topology or human visual cognition**.

Any architecture that attempts to collapse these disparate domains into a single deep neural network (e.g., feeding a raw image into a model that directly outputs measurement coordinates) is fundamentally flawed. 

Every viable industrial architecture must decouple the measurement pipeline into **three distinct physical and computational layers**:

```
┌────────────────────────────────────────────────────────────────────────┐
│ 1. PHYSICAL STATE CONTROL LAYER                                        │
│    • Standardized lay protocol / tension-calibrated spreader fixtures  │
│    • In-situ flatness check (photometric/raking) ──► Re-lay or Abstain │
├────────────────────────────────────────────────────────────────────────┤
│ 2. SEMANTIC PROPOSAL LAYER (Neural / Human-in-the-Loop)                │
│    • Coarse region-of-interest (ROI) classification (~10-20 mm)        │
│    • Garment topology, seam type identity, orientation detection       │
│    • NEVER outputs the final measurement coordinate                    │
├────────────────────────────────────────────────────────────────────────┤
│ 3. METROLOGICAL GEOMETRIC EVALUATION LAYER                             │
│    • Operates at full native sensor resolution (<0.15 mm/px)           │
│    • Sub-pixel continuous edge/ridge extraction (Steger operator)      │
│    • Deterministic GPS operator construction (datums, perpendiculars)  │
│    • Epistemic quality metric evaluation ──► Accept / Abstain          │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 1. Candidate Architecture Descriptions

### Architecture A0 / A0+: Manual Baseline & Connected Digital Metrology

```
   [Inspector] ──► [Manual Steel Tape] ──► [Bluetooth Link] ──► [Digital Tech Pack]
```

- **Concept:** A certified QC inspector places the garment on a flat wooden/metal table and measures POMs using a calibrated non-stretch tape (graduated in mm, OIML R 35 Class II) `[E, ISO18890; OIML-R35]`. 
  - **A0 (Standard Manual):** Inspector writes readings by hand on paper or re-types into an ERP/Excel terminal ($45\text{–}60\text{ s}$ per garment).
  - **A0+ (Connected Smart Tape):** Uses a Bluetooth-enabled optical linear encoder tape (e.g., Fujitsu, Fingertip, Bagel Labs ALT) `[E vendor, DigitalTapes]`. As the inspector pulls the tape between landmarks and clicks a button, the dimension transmits directly to the digital tech pack.
- **Error Components Attacked:** A0+ eliminates transcription errors and human reading parallax. It does **not** reduce definitional ambiguity ($u_A$) or fabric handling strain ($u_B$).
- **Throughput:** $45\text{ to }60\text{ seconds}$ per garment for a full 15-POM specification.
- **Hardware Cost:** Tier 0 (< €500).
- **Metrological Role:** **A0 is the contractual benchmark.** Any automated system must prove equivalence or superiority against A0 on identical production lots (Stage S7).

---

### Architecture A1: High-Resolution Calibrated 2D Station with Polarized Multi-Angle Raking Illumination

```
                  ┌──────────────────────┐
                  │ 45-61 MP Area Camera │  (H ≥ 2.2 m, Low-Distortion Optics)
                  └──────────────────────┘
                             │
     ┌───────────────────────┼───────────────────────┐
     ▼                       ▼                       ▼
[Cross-Polarized Dome] [4-Quadrant Raking LEDs]  [In-Situ Glass Grid]
(Eliminates Glare)     (Seam Ridge Relief)     (Traceable Scale Frame)
     │                       │                       │
═════╧═══════════════════════╧═══════════════════════╧═════ Table Surface
```

- **Concept:** A rigid gantry suspends a high-resolution industrial 2D camera (45–61 MP, pixel pitch $\approx 0.13\text{ mm/px}$ across $1200 \times 1600\text{ mm}$) at height $H \ge 2.2\text{ m}$ above a non-reflective matte table.
- **Illumination Architecture:**
  - *Cross-Polarized Diffuse Overhead Illumination:* Eliminates specular glare and highlights from synthetic fabrics (polyester, nylon, satin finishes).
  - *Multi-Azimuth Raking Illumination:* Four directional low-angle LED line bars ($10^\circ\text{ to }15^\circ$ grazing angle) mounted around the table perimeter. By sequentially strobing azimuths (North, South, East, West), surface seams, stitch grooves, and folded ridges cast sharp micro-shadows, making seams visible regardless of thread color.
- **In-Situ Calibration:** The table perimeter features permanent, laser-etched ceramic/invar fiducial targets certified to ISO/IEC 17025. Every image frame mathematically verifies lens distortion parameters and spatial scale in-situ.
- **Hardware Cost:** Tier 1 (€2,500 to €4,500).

---

### Architecture A2: Dual-Mode Transmission Backlight & Polarized Front-Light Station

```
                  ┌──────────────────────┐
                  │ 45-61 MP Area Camera │
                  └──────────────────────┘
                             │
     ┌───────────────────────┴───────────────────────┐
     ▼                                               ▼
[Multi-Angle Raking LEDs]               [Diffuse LED Backlight Table]
(Top Surface Seams on Opaque)           (Transmission Silhouette on Knits)
     │                                               │
═════╧═══════════════════════════════════════════════╧═════ Glass Tabletop
```

- **Concept:** Incorporates an ultra-uniform LED edge-lit diffuser panel into the table surface, but with explicit operational limits recognized from Chapter 00:
  - *Mode 1 (Lightweight / Translucent Knits):* Backlight strobe generates a pure transmission silhouette, bypassing surface fabric prints and yarn colors. Translucent single-ply fabrics reveal internal seam allowance bands.
  - *Mode 2 (Opaque / Heavy Fabrics & Dropped Hems):* For heavy denim, black fleece, or curved/dropped hems, backlighting is **deactivated** to avoid front/back hem conflation. The system switches automatically to cross-polarized multi-angle raking illumination.
- **Hardware Cost:** Tier 1–2 (€4,000 to €7,500).

---

### Architecture A3: Continuous Line-Scan / CIS Orthographic Conveyor System (S-Inline Sentinel)

```
       [16k Telecentric Line-Scan / CIS Array]
                         │
      ───────────────────▼─────────────────── Optical Scan Line
      ═══════════════════════════════════════ Moving Tension-Free Conveyor Belt
                       ◄─── Motion (v)
```

- **Concept:** Designed specifically for **S-Inline 100% production screening**, breaking the 45–60s manual bottleneck. The garment travels on a continuous conveyor belt underneath a 16,384-pixel line-scan camera or a Contact Image Sensor (CIS) array spanning $1200\text{ mm}$ width.
- **Metrological Advantages:**
  - **Zero Perspective Distortion along the Scan Axis:** The optical projection along the motion axis is completely telecentric/orthographic. Height relief ($h$) introduces zero parallax error in the direction of belt travel `[I]`.
  - **Ultra-High Resolution:** Achieves $0.07\text{ mm/pixel}$ spatial resolution across the entire garment span.
  - **Rapid Cycle Time:** Garments pass continuously at $0.5\text{ m/s}$, executing full scan in **$3\text{ to }5\text{ seconds}$**!
- **Technical Demands:** Longitudinal scale accuracy depends strictly on conveyor belt velocity stability and rotary optical encoder synchronization.
- **Hardware Cost:** Tier 2–3 (€12,000 to €22,000).

---

### Architecture A4-Stereo: Calibrated Multi-Camera Stereo Photogrammetry Station

```
         ┌───────────────────┐               ┌───────────────────┐
         │ Left 24 MP Camera │               │ Right 24 MP Camera│
         └───────────────────┘               └───────────────────┘
                   │                                   │
                   └─────────────────┬─────────────────┘
                                     ▼
                   [Calibrated Epipolar Stereo Pair]
                                     │
         ┌───────────────────────────┴───────────────────────────┐
         ▼                                                       ▼
  High-Res Orthophoto                                     Dense Topographic Map
  (Sub-pixel 2D Metrology)                                h(x,y) for Parallax Orthorect.
```

- **Concept:** Overcomes Claude's premature dismissal of 3D sensing. Instead of an expensive €15k Zivid camera, this architecture mounts **two synchronized 24 MP industrial cameras** (€1,200 total) in a calibrated stereo baseline.
- **Metrological Advantages:**
  - **Simultaneous 2D and 3D:** Generates full-resolution 24 MP color orthophotos while computing a dense surface height map $h(x, y)$ via stereo semi-global matching.
  - **Zero Structured-Light Geodesic Inflation:** Because no active laser/fringe projector is used, depth data is used *strictly* to mathematically orthorectify 2D planar coordinates ($\mathbf{x}_{\text{ortho}} = \mathbf{x}_{\text{obs}}(1 - h/H)$), driving perspective error below **$0.2\text{ mm}$**!
  - **Robust Flatness Gating:** Directly detects out-of-plane wrinkles exceeding $1.5\text{ mm}$ height, rejecting bad lays automatically.
- **Hardware Cost:** Tier 1 (€3,500 to €5,500).

---

### Architecture A5: Dedicated Structured-Light 3D Camera System (Previous Team's Baseline)

- **Concept:** A commercial structured-light 3D camera (Zivid 2+ L110 class, costing $\sim €15,000$ with GPU) projects phase-shifted fringe patterns to reconstruct full 3D point clouds `[E, Zivid2+L110]`.
- **Verdict:** **Unfavorable.** Coarser 2D spatial sampling ($0.44\text{ mm/px}$), factory scale trueness specification ($\pm 0.35\%\text{ to }0.60\%$) introduces $2.5\text{ mm to }4.2\text{ mm}$ of scale uncertainty, and depth noise systematically inflates geodesic arc length by $+26.6\%$. Overpriced and technically inferior to Architecture A4-Stereo for flat-lay metrology.
- **Hardware Cost:** Tier 2 (€15,000 to €20,000).

---

### Architecture A6-Mech: Mechanical Tension-Calibrated Fixtures & Boundary Air-Knives

```
   [Tension Spreader Bar] ──► Holds Waistband / Shoulders at Calibrated 0.5-1.0 N
                                       AND
   [Boundary Air-Knives]  ──► Low-Pressure Laminar Air Sweep Flattens Hem Edges
```

- **Concept:** Completely replaces the failed perforated vacuum table concept. Implements true **ISO 10579 restrained metrology** using textile-appropriate physical fixtures:
  - *Tension-Calibrated Spreader Bars:* For waists, neck openings, and shoulders, lightweight anodized aluminum spreader bars with calibrated internal springs or load cells apply an exact, repeatable tensile force ($0.50\text{ N to }1.00\text{ N}$). This eliminates handling shear variance while standardizing measurement tension.
  - *Boundary Air-Knives:* Low-pressure laminar air blades ($<0.5\text{ bar}$) sweep horizontally across the table surface, gently smoothing out fabric edge ripples without applying normal clamping friction.
- **Hardware Cost:** Tier 1–2 (€2,000 to €5,000).

---

### Architecture A7: Human-in-the-Loop Collaborative Architecture (Human Semantics + Machine Metrology)

```
    [High-Res Optical Capture] ──► [AI Proposes Candidate Landmarks on Touch Display]
                                                    │
                                                    ▼
    [Operator Taps / Confirms Landmark] ◄── [Visual Feedback on Screen]
                                                    │
                                                    ▼
    [Full-Resolution Sub-Pixel Snapping to Nearest True Physical Edge/Seam]
                                                    │
                                                    ▼
    [Deterministic Metric Evaluation & Instant QC Pass/Fail Output]
```

- **Concept:** Designed specifically for **S-AQL batch inspection** (proven commercially by Toyoshima AI Measure 2nd Gen `[E, Toyoshima]`). 
- **Division of Cognitive and Metrological Labor:**
  - *The Human Does Semantics:* The inspector confirms or adjusts landmark proposals on a touch monitor within $5\text{ mm}$ of the true feature ($15\text{–}20\text{ s}$).
  - *The Machine Does Metrology:* The algorithm samples the full-resolution gradient, fits a sub-pixel continuous contour (Steger edge operator), executes the formal GPS operator, and calculates the true distance.
- **Active Learning Data Engine:** Every confirmed or corrected point is automatically logged as a verified, metrologically grounded ground-truth training pair on real factory production garments.
- **Hardware Cost:** Tier 1 (€3,000 to €5,500).

---

## 2. Comprehensive Multi-Attribute Comparison Matrix

The table below evaluates all candidate architectures across 12 decisive engineering, metrological, and economic dimensions `[I]`:

| Metric / Dimension | A0 / A0+ Manual / Digital Tape | A1 Calibrated 2D + Raking Light | A2 Dual Backlight / Front Raking | A3 Line-Scan / CIS Conveyor | A4-Stereo Photogrammetry | A5 Structured-Light 3D (Zivid) | A6-Mech Spreader Fixtures | A7 Human-in-the-Loop (Collaborative) |
|---|---|---|---|---|---|---|---|
| **Achievable Standard Uncertainty ($u_c$)** | $1.5\text{–}3.5\text{ mm}$ (Uncontrolled) | $1.0\text{–}1.8\text{ mm}$ (Good lay) | **$0.8\text{–}1.4\text{ mm}$** (High contrast) | **$0.8\text{–}1.4\text{ mm}$** (Belt synced) | **$0.7\text{–}1.2\text{ mm}$** (Parallax free) | $2.0\text{–}4.5\text{ mm}$ (Scale/noise lim) | **$0.6\text{–}1.0\text{ mm}$** (Restrained) | **$0.8\text{–}1.4\text{ mm}$** (Human semantic) |
| **Attacks Definitional Error ($u_A$)** | No (Inspector bias) | Partial (via GPS ops) | Partial (Seam translucency) | Partial (GPS ops) | Partial (GPS ops) | No | **Yes** (Standardized tension) | **Yes** (Interactive consensus) |
| **Attacks Garment State ($u_B$)** | No | No (Protocol only) | No (Protocol only) | Partial (Belt draw) | **Yes** (Rejects wrinkles) | Partial (Detects relief) | **Directly eliminates handling shear** | No (Protocol only) |
| **Attacks Optical/Sensing ($u_C$)** | N/A (Manual) | **Yes** (Calibrated) | **Yes** (Calibrated) | **Yes** (Orthographic) | **Yes** (Full 3D orthorect.) | Partial (Coarse scale) | Yes (Flat plane) | **Yes** (Full-res sub-px) |
| **Attacks Feature Extraction ($u_D$)** | Human eye | **Yes** (Raking shadows) | **Directly** (Backlight silhouette) | Yes (High-res) | **Yes** (Sub-px stereo) | Poor on dark fabrics | Yes (Flat edges) | **Yes** (Sub-px snapping) |
| **Attacks Semantic Error ($u_E$)** | Human brain | Partial (Learned AI) | Partial (Learned AI) | Partial (Learned AI) | Partial (Learned AI) | Partial (Learned AI) | N/A | **Directly eliminates gross errors** |
| **Scenario Suitability: S-AQL** | Baseline | **Excellent** | **Excellent** | Moderate | **Optimal** | Moderate | **Excellent** | **Optimal (Best fit)** |
| **Scenario Suitability: S-Inline**| Strictly Unviable | With fast lay | With fast lay | **Optimal (Best fit)**| With fast lay | Marginal | Slows handling | Unviable (Too slow) |
| **Cycle Time per Garment** | $45\text{–}60\text{ s}$ | $20\text{–}35\text{ s}$ | $20\text{–}35\text{ s}$ | **$5\text{–}8\text{ s}$** | $20\text{–}35\text{ s}$ | $30\text{–}60\text{ s}$ | $25\text{–}40\text{ s}$ | $25\text{–}35\text{ s}$ |
| **Hardware Capex Tier** | Tier 0 (<€500) | **Tier 1 (€3k–€4.5k)** | **Tier 1 (€4k–€7.5k)** | Tier 2–3 (€12k–€22k)| **Tier 1 (€3.5k–€5.5k)**| Tier 2 (€15k–€20k) | Tier 1–2 (€2k–€5k) | **Tier 1 (€3k–€5.5k)** |
| **Primary Failure Modes** | Transcription, tape stretch, human fatigue | Perspective parallax, low seam contrast | Union silhouette on multi-ply hems | Belt speed jitter, fabric skew on belt | Stereo matching on textureless white | Scale bias, FOV truncation, depth noise | Restraint bias vs free state | Operator click fatigue on high volume |

---

## 3. Factory Throughput and the Two-Tier Strategy

To genuinely solve the business problem (breaking the 45–60s manual inspection bottleneck), automation cannot rely on an operator manually smoothing a static garment on a table for full-spec inspection on every unit.

We establish the **Two-Tier Industrial QC Architecture**:

```
 ┌────────────────────────────────────────────────────────────────────────┐
 │ TIER 1: S-INLINE SENTINEL (Conveyor Architecture A3)                   │
 │ • 100% Inline screening of ALL production units                        │
 │ • Cycle Time: 5 to 8 seconds per garment (Continuous feed)             │
 │ • Scope: 3 to 5 critical process-control POMs (Length, Chest, Sleeve)  │
 │ • Value: Instant drift alarm (catches cutting/wash drift in real time) │
 └────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼ (Sampling / Outlier Escalation)
 ┌────────────────────────────────────────────────────────────────────────┐
 │ TIER 2: S-AQL COLLABORATIVE (Stereo / Touch Architecture A4 + A7)      │
 │ • Statistical lot sampling (ANSI/ASQ Z1.4 Level II)                    │
 │ • Cycle Time: 25 to 35 seconds per garment (Dual-sided station)        │
 │ • Scope: Full 15 to 25 POM tech pack audit                             │
 │ • Value: Eliminates manual transcription; generates digital audit cert │
 └────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Economic Model: Is a €15,000 Setup Necessary?

### 4.1 Hardware Capex Comparison
The table below contrasts the actual market hardware costs `[E prices; I]`:

| Hardware Subsystem | High-Res Stereo 2D Station (A4-Stereo) | Structured-Light 3D Station (A5 - Zivid Class) |
|---|---|---|
| **Primary Optical Sensor** | €1,800 – €2,400 (Dual 24 MP Industrial Sony/Basler) | €12,500 – €16,000 (Zivid 2+ L110 3D Structured Light) |
| **Optical Lenses** | €600 – €900 (Dual low-distortion C-mount lenses) | Integrated factory optics |
| **Illumination System** | €800 – €1,500 (Cross-polarized dome + 4 raking LED bars) | Integrated structured-light projector |
| **Calibration Standards** | €500 – €1,000 (Laser-etched quartz/ceramic grid frame) | Factory datasheet calibration (Unverified in-situ) |
| **Physical Table & Gantry** | €800 – €1,200 (Extruded aluminum gantry + optical table) | €1,000 – €2,000 (Heavy vibration-damped gantry) |
| **Compute Hardware** | €1,000 – €1,400 (Standard industrial PC / RTX 4060) | €2,500 – €4,000 (High-end GPU workstation for point clouds) |
| **TOTAL HARDWARE CAPEX** | **€5,500 – €8,400** (Single-camera A1 is **< €4,500**) | **€16,000 – €22,000** |

#### Economic Verdict:
A €15,000+ structured-light 3D camera is **neither necessary nor technically superior**. A dual-camera calibrated stereo photogrammetry system (A4-Stereo) achieves higher planar resolution, eliminates depth-noise length inflation, and costs **less than half the price**.
