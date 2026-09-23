# 04 — Candidate Measurement Architectures and Comparative Evaluation

## 0. The Universal Three-Layer Architectural Decoupling

A primary conclusion from the error decomposition in Document 03 is that **different error sources require fundamentally different engineering remedies**:
- Definitional ambiguity ($u_A$) and garment state ($u_B$) are governed by **contracts, protocols, and physical fixturing**.
- Sensing ($u_C$) and feature boundary extraction ($u_D$) are governed by **optics, sensor geometry, and illumination physics**.
- Semantic identification ($u_E$) is governed by **machine learning topology or human visual cognition**.

Any architecture that attempts to collapse these disparate domains into a single deep neural network (e.g., feeding a raw image into a model that directly outputs measurement coordinates) is fundamentally flawed. 

Every viable industrial architecture must decouple the measurement pipeline into **three distinct physical and computational layers**:

```
┌────────────────────────────────────────────────────────────────────────┐
│ 1. PHYSICAL STATE CONTROL LAYER                                        │
│    • Standardized lay protocol / active vacuum restraint / platen      │
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

The candidate architectures evaluated below differ primarily in how they implement **Layer 1 (State Control)** and **Layer 3 (Metrological Sensing)**.

---

## 1. Candidate Architecture Descriptions

### Architecture A0 / A0+: Manual Baseline & Connected Digital Metrology

```
   [Inspector] ──► [Manual Steel Tape] ──► [Bluetooth Link] ──► [Digital Tech Pack]
```

- **Concept:** A certified QC inspector places the garment on a flat wooden/metal table and measures POMs using a calibrated non-stretch tape (graduated in mm, OIML R 35 Class II) `[E, ISO18890; OIML-R35]`. 
  - **A0 (Standard Manual):** Inspector writes readings by hand on paper or re-types into an ERP/Excel terminal.
  - **A0+ (Connected Smart Tape):** Uses a Bluetooth-enabled optical linear encoder tape (e.g., Fujitsu, Fingertip, Bagel Labs ALT) `[E vendor, DigitalTapes]`. As the inspector pulls the tape between landmarks and clicks a button, the dimension transmits directly to the digital tech pack.
- **Error Components Attacked:** A0+ eliminates transcription errors and human reading parallax. It does **not** reduce definitional ambiguity ($u_A$) or fabric handling strain ($u_B$).
- **Throughput:** $2\text{ to }5\text{ minutes}$ per garment for a full 15-POM specification.
- **Hardware Cost:** Tier 0 (< €500).
- **Metrological Role:** **A0 is the contractual benchmark.** Any automated system must prove equivalence or superiority against A0 on identical production lots (Stage S7).

---

### Architecture A1: High-Resolution Calibrated 2D Station with Multi-Angle Raking Illumination

```
                  ┌──────────────────────┐
                  │ 45-61 MP Area Camera │  (H ≥ 2.2 m, Fixed Focal Length)
                  └──────────────────────┘
                             │
     ┌───────────────────────┼───────────────────────┐
     ▼                       ▼                       ▼
[Diffuse Dome]     [4-Quadrant Raking LEDs]  [In-Situ Glass Grid]
(General Texture)    (Seam Ridge Relief)     (Traceable Scale Frame)
     │                       │                       │
═════╧═══════════════════════╧═══════════════════════╧═════ Table Surface
```

- **Concept:** A rigid gantry suspends a high-resolution industrial 2D camera (45–61 MP, pixel pitch $\approx 0.13\text{ mm/px}$ across $1200 \times 1600\text{ mm}$) at height $H \ge 2.2\text{ m}$ above a non-reflective matte table.
- **Illumination Architecture:**
  - *Primary Illumination:* High-CRI diffuse overhead LED dome light for global segmentation and color capture.
  - *Multi-Azimuth Raking Illumination:* Four directional low-angle LED line bars ($10^\circ\text{ to }15^\circ$ grazing angle) mounted around the table perimeter. By sequentially strobing azimuths (North, South, East, West), surface seams, stitch grooves, and folded ridges cast sharp micro-shadows, making seams visible regardless of thread color.
- **In-Situ Calibration:** The table perimeter features permanent, laser-etched ceramic/invar fiducial targets certified to ISO/IEC 17025. Every image frame mathematically verifies lens distortion parameters and spatial scale in-situ.
- **Variant A1-D (Dual-Sided Synchronous Capture):** The garment rests on an anti-reflective, low-iron optical glass table. A secondary high-resolution camera mounted beneath the table captures the underside simultaneously. This enables measuring front and back POMs (e.g., front chest vs center back length) in a single lay without flipping the garment `[H]`.
- **Hardware Cost:** Tier 1 (€2,500 to €5,000).

---

### Architecture A2: Backlit Transmission Metrology Table (Silhouette Gauging) + Raking Front Light

```
                  ┌──────────────────────┐
                  │ 45-61 MP Area Camera │
                  └──────────────────────┘
                             │
     ┌───────────────────────┴───────────────────────┐
     ▼                                               ▼
[Multi-Angle Raking LEDs]               [Diffuse LED Backlight Table]
(Top Surface Seams)                     (Pure Transmission Silhouette)
     │                                               │
═════╧═══════════════════════════════════════════════╧═════ Glass Tabletop
```

- **Concept:** Adapts the proven metrological architecture of industrial optical gauges (e.g., InspecVision Planar, which achieves $\pm 35\ \mu\text{m}$ across $1.8\text{ m}$ `[E, InspecVision]`). The table surface consists of a high-power, ultra-uniform LED edge-lit diffuser panel.
- **Dual-Strobe Optical Sequence:**
  - *Frame 1 (Backlight ON, Front Light OFF):* Produces a pure transmission silhouette. The garment edge is converted into a high-contrast binary shadow profile, completely immune to fabric color, graphic prints, embroidery, or ambient room light.
  - *Frame 2 (Front Raking Lights ON, Backlight OFF):* Captures surface seams, pockets, and collars.
- **Multi-Layer Translucency Benefit:** In lightweight, medium, or white fabrics (T-shirts, blouses, shirts), transmission light penetrates single-ply fabric but is absorbed by multi-ply structures. **Internal seam allowances, hem hems, and pocket bags appear as dark bands**, revealing the true sewn seam line beneath the outer fold `[H]`.
- **Hardware Cost:** Tier 1–2 (€4,000 to €8,000).

---

### Architecture A3: Continuous Line-Scan / CIS Orthographic Conveyor System

```
       [16k Telecentric Line-Scan / CIS Array]
                         │
      ───────────────────▼─────────────────── Optical Scan Line
      ═══════════════════════════════════════ Moving Vacuum Conveyor Belt
                       ◄─── Motion (v)
```

- **Concept:** Designed specifically for **S-Inline 100% production screening**. The garment travels on a continuous conveyor belt underneath a 16,384-pixel line-scan camera or a Contact Image Sensor (CIS) array spanning the full $1200\text{ mm}$ width.
- **Metrological Advantages:**
  - **Zero Perspective Distortion along the Scan Axis:** The optical projection along the motion axis is completely telecentric/orthographic. Height relief ($h$) introduces zero parallax error in the direction of belt travel `[I]`.
  - **Ultra-High Resolution:** Achieves $0.07\text{ mm/pixel}$ spatial resolution across the entire garment span.
- **Technical Limitations & Risks:**
  - Longitudinal scale accuracy depends strictly on conveyor belt velocity stability and rotary optical encoder synchronization. Any belt slippage, motor cogging, or stretch introduces longitudinal measurement distortion.
  - Garment must be smoothed onto the moving belt; dynamic aerodynamic flutter can cause edge ripples.
- **Hardware Cost:** Tier 2–3 (€12,000 to €25,000).

---

### Architecture A4: Calibrated 2D Metrology with Active Flatness Gating & Height Orthorectification

```
         ┌───────────────────┐       ┌───────────────────┐
         │ High-Res 2D Color │       │ Low-Cost Active   │  (Laser Profiler,
         │ Metrology Camera  │       │ Height Sensor     │   Photometric Stereo,
         └───────────────────┘       └───────────────────┘   or Stereo Pair)
                   │                           │
                   ▼                           ▼
            Full-Res Image                Height Map h(x,y)
                   │                           │
                   └─────────────┬─────────────┘
                                 ▼
         ┌───────────────────────────────────────────────┐
         │ 1. Flatness Gate: If max(h) > 2 mm ──► RE-LAY │
         │ 2. Orthorectify: x_corr = x_obs * (1 - h/H)   │
         │ 3. Execute Sub-Pixel GPS Measurement          │
         └───────────────────────────────────────────────┘
```

- **Concept:** Maintains high-resolution 2D imaging (A1 or A2) as the primary metrology sensor, but couples it with an auxiliary low-cost depth/height sensing modality (e.g., a calibrated multi-line laser projector, photometric stereo from the 4-quadrant raking lights, or a stereo pair).
- **Two Metrological Functions:**
  1. **Automated Flatness Gating (Wrinkle Rejection):** Scans the garment surface for out-of-plane buckles. If wrinkle amplitude exceeds $A > 1.5\text{ mm}$ along a critical POM axis, the system refuses to measure, flashing a visual laser contour onto the table prompting the operator to re-smooth the wrinkle.
  2. **Topographic Orthorectification:** Measures the coarse height map $h(x, y)$ of the garment (to $\pm 0.5\text{ mm}$ accuracy) and mathematically cancels perspective parallax:
     $$\mathbf{x}_{\text{ortho}} = \mathbf{x}_{\text{apparent}} \left( 1 - \frac{h(x, y)}{H} \right)$$
- **Hardware Cost:** Tier 1 (€3,500 to €6,500).

---

### Architecture A5: Dedicated Structured-Light 3D Camera System (Previous Team's Baseline)

```
              ┌──────────────────────────────────────┐
              │ Zivid 2+ L110 / M130 3D RGB-D Camera │
              └──────────────────────────────────────┘
                                 │
     ┌───────────────────────────┴───────────────────────────┐
     ▼                                                       ▼
RGB Texture Image                                    3D Point Cloud Mesh
(2448 x 2048, 0.44 mm/px)                           (Trueness Error ±0.35-0.60%)
```

- **Concept:** A commercial structured-light 3D camera (Zivid 2+ L110 class, costing $\sim €15,000$ with GPU) projects calibrated phase-shifted fringe patterns to reconstruct full 3D point clouds `[E, Zivid2+L110]`.
- **Metrological Evaluation:**
  - *Strengths:* Direct, simultaneous acquisition of 3D geometry and RGB texture; robust rejection of ambient lighting; vendor-maintained hardware calibration.
  - *Weaknesses:* Coarser 2D spatial sampling ($0.44\text{ mm/px}$); factory scale trueness specification ($\pm 0.35\%\text{ to }0.60\%$) introduces $2.5\text{ mm to }4.2\text{ mm}$ of scale uncertainty over $700\text{ mm}$; naive 3D geodesic surface measurement inflates lengths due to depth noise (Chapter 03 §2.3); FOV is too small for large garments.
- **Hardware Cost:** Tier 2 (€15,000 to €20,000).

---

### Architecture A6: Physically Constrained Restrained Lay Fixtures (ISO 10579 Compliant)

```
        Garment Placed ──► [Perforated Vacuum Table] ──► Instant Flatness & Strain Lock
                                    OR
        Garment Placed ──► [Transparent Compression] ──► Mechanical Planarization
                           [Acrylic Platen Lowered ]
```

- **Concept:** Implements the core principle of **ISO 10579 (*Dimensioning of Non-Rigid Parts*)** by replacing the uncontrolled "hand-smoothed free state" with an explicit **restrained state**:
  - *Option A6-Vac (Perforated Vacuum Table):* The table top features a fine grid of micro-perforations connected to an industrial vacuum blower (standard in garment cutting rooms, e.g., Lectra/Gerber automated cutters `[E, LectraParagon]`). When activated, atmospheric pressure exerts a uniform downward clamping force ($1\text{ to }3\text{ kPa}$), pulling out fabric ripples and locking the boundary in place.
  - *Option A6-Platen (Compression Platen):* A hinged, counterweighted optical-grade glass or acrylic sheet is lowered onto the garment, compressing folds and establishing a deterministic $Z = 0$ plane.
  - *Option A6-Tension (Actuated Tensile Clamps):* Dedicated pneumatically actuated clamps with inline load cells for measuring force-defined POMs (e.g., elastic waists stretched to $15.0\text{ N}$).
- **Metrological Caveat:** Physical restraint modifies fabric dimensions. The restrained state *becomes the legal measurand* and must be calibrated against the contractual manual baseline.
- **Hardware Cost:** Tier 1–2 (€3,000 to €8,000).

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

- **Concept:** Designed specifically for **S-AQL batch inspection**. The operator lays the garment on an A1 or A2 optical table. A high-resolution camera captures the image. A lightweight neural network detects candidate landmark regions and displays them on a touch monitor.
- **Division of Cognitive and Metrological Labor:**
  - *The Human Does Semantics:* The human operator visually verifies feature identity (e.g., confirming "this is the front collar edge, not the back collar") by tapping the screen or dragging a proposal box within $5\text{ mm}$ of the true feature.
  - *The Machine Does Metrology:* The algorithm samples the full-resolution gradient within the operator's clicked ROI, fits a sub-pixel continuous contour (Steger edge operator), executes the formal GPS operator, and calculates the true distance.
- **Active Learning Data Engine:** Every confirmed or corrected point is automatically logged as a verified, metrologically grounded ground-truth training pair on real factory production garments. Automation can be dialed up POM-by-POM as model reliability proves out.
- **Hardware Cost:** Tier 1 (€3,000 to €6,000).

---

### Architecture A8: CAD / Pattern-Constrained Non-Rigid Registration Layer

- **Concept:** This is a software layer applied atop A1–A4. The system ingests the 2D CAD cutting pattern (DXF/AAMA format). A non-rigid registration algorithm (Coherent Point Drift, Thin-Plate Splines) aligns the pattern boundary with the observed image contours, inferring the positions of partially occluded landmarks.
- **Mandatory Metrological Safeguard:** The registration must act *strictly as a topological prior* to establish feature correspondence, never to pull or snap final coordinates toward nominal values. Must be audited with seeded out-of-tolerance garments to verify slope $\beta \in [0.95, 1.05]$ (Document 03 §5.2) `[I]`.
- **Hardware Cost:** €0 (Software layer).

---

### Architecture A9: Design-for-Measurability (In-Situ Invisible Optical Fiducials)

- **Concept:** Adapts semiconductor scribe-line targets and ISO 3759 marked points to apparel:
  - **A9-V (Validation Reference Standard):** Certified QC technicians mark true landmark intersections on test garments using **invisible UV-fluorescent ink** (visible only under $365\text{ nm}$ black light). Under white light, the garment appears pristine for AI testing; under UV strobe, exact ground-truth coordinates are captured with zero definitional ambiguity `[E method, Kouchi2011]`.
  - **A9-P (Production In-Situ Marking):** Automated inkjet printheads integrated into CNC fabric cutting tables deposit microscopic UV-fluorescent dots in seam allowances during ply cutting.
- **Hardware Cost:** Consumables / Specialized illumination.

---

### Architecture A10: Fully Robotic Handling, Unfolding, and Tensioning

- **Concept:** Autonomous dual-arm industrial robots equipped with pneumatic grippers pick unconstrained garments from a bin, execute dynamic flinging/unfolding actions (e.g., FlingBot `[E, FlingBot]`), smooth the garment on an inspection table, and present it to an optical station.
- **Metrological Feasibility:** **Not Viable in the Near/Medium Term.** Current state-of-the-art cloth manipulation achieves $>80\%$ spatial coverage, but leaves uncontrolled in-plane shear strains ($>5\%$) and edge curls `[E, Lips2024]`. The resulting garment state variability ($u_B > 10\text{ mm}$) completely violates industrial QC tolerances.
- **Hardware Cost:** Tier 3 (> €80,000).

---

## 2. Comprehensive Multi-Attribute Comparison Matrix

The table below evaluates all candidate architectures across 12 decisive engineering, metrological, and economic dimensions `[I]`:

| Metric / Dimension | A0 / A0+ Manual / Digital Tape | A1 Calibrated 2D + Raking Light | A2 Backlit Transmission + Raking | A3 Line-Scan / CIS Conveyor | A4 2D + Active Height Gating | A5 Structured-Light 3D (Zivid) | A6 Restrained State (Vacuum) | A7 Human-in-the-Loop (Collaborative) | A8 CAD Non-Rigid Layer | A9-V Invisible UV Fiducials | A10 Autonomous Robotics |
|---|---|---|---|---|---|---|---|---|---|---|---|
| **Achievable Standard Uncertainty ($u_c$)** | $1.5\text{–}3.5\text{ mm}$ (Uncontrolled) | $1.0\text{–}1.8\text{ mm}$ (Good lay) | **$0.8\text{–}1.4\text{ mm}$** (High contrast) | $0.8\text{–}1.5\text{ mm}$ (Belt synced) | **$0.8\text{–}1.4\text{ mm}$** (Parallax free) | $2.0\text{–}4.5\text{ mm}$ (Scale/noise lim) | **$0.6\text{–}1.2\text{ mm}$** (Restrained) | **$0.8\text{–}1.5\text{ mm}$** (Human semantic) | N/A (Software layer) | **$0.2\text{–}0.4\text{ mm}$** (Fiducial limit) | $>10\text{ mm}$ (Handling strain) |
| **Attacks Definitional Error ($u_A$)** | No (Inspector bias) | Partial (via GPS ops) | Partial (Seam translucency) | Partial (GPS ops) | Partial (GPS ops) | No | Yes (Explicit state) | **Yes** (Interactive consensus) | Partial (Topology check) | **Directly eliminates** | No |
| **Attacks Garment State ($u_B$)** | No | No (Protocol only) | No (Protocol only) | Partial (Belt draw) | **Yes** (Rejects wrinkles) | Partial (Detects relief) | **Directly eliminates wrinkles** | No (Protocol only) | No | No | Severely degrades state |
| **Attacks Optical/Sensing ($u_C$)** | N/A (Manual) | **Yes** (Calibrated) | **Yes** (Calibrated) | **Yes** (Orthographic) | **Yes** (Height corr.) | Partial (Coarse scale) | Yes (Flat plane) | **Yes** (Full-res sub-px) | N/A | **Yes** (Sharp targets) | Yes |
| **Attacks Feature Extraction ($u_D$)** | Human eye | **Yes** (Raking shadows) | **Directly** (Backlight silhouette) | Yes (High-res) | Yes (Raking shadows) | Poor on dark fabrics | Yes (Flat edges) | **Yes** (Sub-px snapping) | Partial | **Directly eliminates** | Poor (Folded edges) |
| **Attacks Semantic Error ($u_E$)** | Human brain | Partial (Learned AI) | Partial (Learned AI) | Partial (Learned AI) | Partial (Learned AI) | Partial (Learned AI) | N/A | **Directly eliminates gross errors** | **Yes** (Plausibility checks) | **Directly eliminates** | Fails |
| **Risk of Prior Shrinkage ($u_F$)** | None | Low | Low | Low | Low | Low | Low | None | **HIGH (Requires audit)** | None | Low |
| **Scenario Suitability: S-AQL** | Baseline | **Excellent** | **Excellent** | Moderate | **Excellent** | Moderate | **Excellent** | **Optimal (Best fit)** | Excellent (Layer) | **Validation only** | Unviable |
| **Scenario Suitability: S-Inline**| Strictly Unviable | With fast lay | With fast lay | **Optimal (Best fit)**| With fast lay | Marginal | Slows handling | Unviable (Too slow) | Optimal (Layer) | Unviable | Long-term R&D |
| **Cycle Time per Garment** | $120\text{–}300\text{ s}$ | $20\text{–}45\text{ s}$ | $20\text{–}45\text{ s}$ | **$5\text{–}12\text{ s}$** | $25\text{–}50\text{ s}$ | $30\text{–}60\text{ s}$ | $30\text{–}50\text{ s}$ | $30\text{–}60\text{ s}$ | $+2\text{ s}$ compute | $60\text{ s}$ | $>120\text{ s}$ |
| **Hardware Capex Tier** | Tier 0 (<€500) | **Tier 1 (€3k–€5k)** | **Tier 1 (€4k–€8k)** | Tier 2–3 (€15k–€25k)| **Tier 1 (€4k–€7k)** | Tier 2 (€15k–€20k) | Tier 1–2 (€3k–€8k) | **Tier 1 (€3k–€6k)** | €0 (Software) | Tier 0 (Ink/lamps) | Tier 3 (>€80k) |
| **Primary Failure Modes** | Transcription, tape stretch, human fatigue | Perspective parallax, low seam contrast | Union silhouette on multi-ply hems | Belt speed jitter, fabric skew on belt | Laser speckle, dark fabric absorption | Scale bias, FOV truncation, depth noise | Restraint bias vs free state | Operator click fatigue on high volume | Over-regularization toward nominal | Chemical compliance, brand rejection | Gripper snagging, wrinkle lock-in |

---

## 3. Factory Throughput and Takt Time Analysis

The total cycle time per garment $t_{\text{cycle}}$ breaks down into four sequential stages `[I]`:

$$t_{\text{cycle}} = t_{\text{handling}} + t_{\text{acquisition}} + t_{\text{compute}} + p_{\text{abst}} \cdot t_{\text{manual\_recheck}}$$

```
  Cycle Time Comparison (Seconds per Garment)
  ─────────────────────────────────────────────────────────────────────────────
  A0 (Manual Tape)         [████████████████████████████████████████] 180s
  A1 (High-Res 2D)         [███████] 32s  (Handling: 25s, Cap: 2s, Compute: 5s)
  A2 (Backlight Table)     [███████] 32s  (Handling: 25s, Cap: 2s, Compute: 5s)
  A4 (2D + Flatness Gate)  [████████] 38s (Handling: 25s, Flatness/Align: 8s, Compute: 5s)
  A6 (Vacuum Table)        [████████] 36s (Handling: 20s, Vacuum Draw: 8s, Compute: 5s)
  A7 (Human-in-Loop)       [███████████] 50s (Handling: 25s, Screen Confirm: 20s, Compute: 5s)
  A3 (Inline Conveyor)     [██] 8s (Continuous Motion Feeding)
  ─────────────────────────────────────────────────────────────────────────────
```

### Decisive Throughput Insight:
In every stationary architecture (A1, A2, A4, A5, A6, A7), **physical garment handling dominates cycle time** ($20\text{ to }30\text{ seconds}$ for a trained operator to lay and smooth a garment). 
- Optical capture ($<1\text{ s}$) and GPU geometric inference ($2\text{ to }5\text{ s}$) are negligible compared to handling.
- Therefore, investing in expensive high-speed GPU clusters or sub-millisecond cameras produces zero factory throughput advantage. 
- The only engineering lever that dramatically reduces cycle time is moving to a **continuous conveyorized feed (A3)** or optimizing ergonomic handling stations.

---

## 4. Economic Model: Is a €15,000 Setup Necessary?

### 4.1 Deconstructing Equipment Capex Tiers
The table below contrasts the actual market hardware costs of candidate architectures `[E prices; I]`:

| Hardware Subsystem | High-Res 2D Station (A1/A2/A4) | Structured-Light 3D Station (A5 - Zivid Class) |
|---|---|---|
| **Primary Optical Sensor** | €1,800 – €3,500 (24–61 MP Industrial Sony/Basler) | €12,500 – €16,000 (Zivid 2+ L110 3D Structured Light) |
| **Optical Lens** | €400 – €800 (Low-distortion 25–35mm C-mount) | Integrated factory optics |
| **Illumination System** | €800 – €1,800 (LED edge-lit backlight + 4 raking bars) | Integrated structured-light projector |
| **Calibration Standards** | €500 – €1,000 (Laser-etched quartz/ceramic grid frame) | Factory datasheet calibration (Unverified in-situ) |
| **Physical Table & Gantry** | €800 – €1,500 (Extruded aluminum gantry + optical table) | €1,000 – €2,000 (Heavy vibration-damped gantry) |
| **Compute Hardware** | €1,000 – €1,500 (Standard industrial PC / RTX 4060) | €2,500 – €4,000 (High-end GPU workstation for point clouds) |
| **TOTAL HARDWARE CAPEX** | **€5,300 – €9,600** (Base A1 is **< €5,000**) | **€16,000 – €22,000** |

#### Metrological and Economic Conclusion:
A €15,000+ structured-light 3D camera is **neither necessary nor technically superior** for flat-lay garment inspection.
- The 2D optical approach (A1/A2) delivers **$3\times$ higher spatial resolution ($0.13\text{ vs }0.44\text{ mm/px}$)**, superior scale stability when tied to in-situ grid frames, and zero geodesic depth-noise inflation.
- High-resolution 2D hardware with transmission backlighting costs **less than half the price** of a Zivid camera while achieving significantly lower planar measurement uncertainty.

---

### 4.2 Comprehensive Industrial Value and ROI Formulation
The annualized Net Economic Value ($V_{\text{annual}}$) generated by an automated inspection station operating in a factory is modeled from first principles as `[I]`:

$$V_{\text{annual}} = N_{\text{garments}} \cdot \left[ \Delta t_{\text{labor}} \cdot C_{\text{labor}} + \Delta P_{\text{escape}} \cdot C_{\text{defect}} - \Delta P_{\text{false\_reject}} \cdot C_{\text{scrap}} - p_{\text{abst}} \cdot t_{\text{manual}} \cdot C_{\text{labor}} \right] - \text{Amortized Cost}$$

where:
- $N_{\text{garments}}$: Total garments inspected per year ($\approx 250,000\text{ units/year}$ for S-AQL; $\approx 1,500,000$ for S-Inline).
- $\Delta t_{\text{labor}} = t_{\text{manual}} - t_{\text{auto}}$: Labor time saved per garment ($180\text{ s} - 35\text{ s} = 145\text{ s} = 0.040\text{ hours}$).
- $C_{\text{labor}}$: Factory loaded labor cost per hour ($€3.00\text{ to }€8.00/\text{hour}$ in major apparel manufacturing hubs: Bangladesh, Vietnam, Turkey).
- $\Delta P_{\text{escape}}$: Reduction in defective garments shipped to buyer ($0.5\%\text{ to }1.5\%$).
- $C_{\text{defect}}$: Buyer chargeback / return cost per escaped garment ($€15\text{ to }€50/\text{unit}$).
- $\Delta P_{\text{false\_reject}}$: Additional conforming garments incorrectly rejected by guard-banding.
- $p_{\text{abst}}$: System abstention rate routed back to manual inspection ($5\%\text{ to }10\%$).

#### Parametric Sensitivity Analysis:
1. **Labor Savings Alone Justify Hardware in 6 to 12 Months:** Inspecting 250,000 garments/year saving $145\text{ s}$ per garment at $€4.00/\text{hour}$ yields **$€40,277 in direct annual labor savings**, paying off a €6,000 station (A1/A2) in under two months!
2. **Abstention Rate ($p_{\text{abst}}$) Severely Erodes Profitability:** If an automated algorithm exhibits high uncertainty and abstains on $35\%$ of garments, direct labor savings drop by more than a third. This demonstrates why the **Human-in-the-Loop architecture (A7)**, with an effective abstention rate of $0\%$, is commercially superior for S-AQL deployment.
3. **Escaped Defect Penalty Dominates Everything:** In brand manufacturing, a single lot rejection at a European or US distribution center incurs chargebacks of tens of thousands of euros. Reducing Consumer's Risk ($R_C$) by improving measurement uncertainty from $u = 2.5\text{ mm}$ to $u = 1.0\text{ mm}$ delivers vastly more enterprise value than shaving two seconds off cycle time.
