# 04 — Candidate measurement architectures and their trade-offs

*Part of the garment-measurement feasibility study. Evidence labels: **[E]** established (a cited source that was opened and checked), **[I]** engineering inference, **[H]** hypothesis. Source keys resolve in [references.md](references.md).*

**Ground rules for this document**
- No performance numbers are invented. Where a component's uncertainty can be *derived* (sensing geometry, document 03), the derivation is referenced. Everything that depends on garment state or definitional spread is marked **[H]**: it is unknown until the experiments in document 05 are run.
- No winner is picked. The comparison shows which architecture attacks which error component, at what cost and with what risk.
- Costs are **order-of-magnitude hardware estimates** unless a price is cited. Engineering and integration effort — which, on the previous team's evidence, dominates total cost — is discussed separately.

---

## 0. A design principle that applies to every architecture

Document 03 showed that the error components have very different natures:

- **Definitional spread (A)** and **garment state (B)** are fixed by *specification* and *physical process*.
- **Sensing (C)** and **feature extraction (D)** are fixed by *optics, lighting and geometry*.
- **Semantic localisation (E)** is fixed by *learning*, and its residual gross errors by *abstention*.

A credible architecture therefore has **three layers**, whatever the hardware:

```
 ┌──────────────────────────────────────────────────────────────────────────────┐
 │ 1. STATE LAYER      lay protocol / fixture → a reproducible, specified garment │
 │                     state; flatness check → re-lay or abstain                   │
 ├──────────────────────────────────────────────────────────────────────────────┤
 │ 2. SEMANTIC LAYER   (ML or human) garment type, part identity, rough location  │
 │                     of each POM's defining features; never the final coordinate │
 ├──────────────────────────────────────────────────────────────────────────────┤
 │ 3. METRIC LAYER     full-resolution, calibrated geometry: fit edges / seam lines │
 │                     / folds; construct endpoints by the complete POM operator;  │
 │                     per-POM quality metric → accept / abstain                   │
 └──────────────────────────────────────────────────────────────────────────────┘
```

The previous system appears to have merged layers 2 and 3 (a learned keypoint *was* the measurement point) and to have had no explicit layer 1 [H; document 03 §9]. The architectures below differ mainly in **how they implement layers 1 and 3**. Layer 2 is broadly similar across them, except where a human does it (A7) or the garment carries marks (A9).

---

## 1. The candidates

Each is described on a common template. "Attacks" refers to the error components of document 03 §1.

### A0 — Manual tape, with and without digital capture (baseline)

| | |
|---|---|
| **Concept** | Trained inspector, ISO 18890-style procedure, tape graduated in mm [E, ISO18890]. **A0+**: a Bluetooth digital tape that sends readings straight into inspection software (products marketed for apparel: Fujitsu, ALT, Fingertip; an older patent describes the same workflow) [E vendor/patent, DigitalTapes] |
| **Attacks** | A0+ removes transcription errors and adds traceability. Neither attacks A or B; the inspector's judgement *is* the semantic and metric layer |
| **Evidence** | No published reproducibility study of trained garment inspectors was found. Informal evidence of centimetre-level disagreement among untrained people [E, Paulauskaite2022] |
| **Throughput** | Minutes per garment for a full spec (to be measured in the pilot [H]) |
| **Cost** | Negligible hardware (tape to low hundreds of euros for a digital tape) |
| **Why it matters** | **It is the comparator.** Any automated system must be judged against A0 measured on the same garments in the same states (document 05, S7). If A0's reproducibility turns out to be poor, the bar for automation is lower than assumed |

### A1 — Calibrated high-resolution 2D flat-lay with a hybrid semantic + metric pipeline

| | |
|---|---|
| **Concept** | A fixed camera above a flat table (e.g. 24–61 MP; ≈ 0.13–0.20 mm/px over 1.2 m [I, document 03 §2.1]). Diffuse front light plus switchable raking light (low-angle, several azimuths) to reveal seam topography. A certified fiducial frame on the table verifies scale and distortion *in every image*. Fixed focus and aperture, locked settings, controlled camera height (≥ 1.5–2 m to limit perspective error; document 03 §2.2) |
| **Layers** | State: lay protocol plus flatness check (raking-light cues) with re-lay prompts. Semantic: ML part and feature proposals (can be trained on *real* images labelled against physical marks, plus physically based synthetic data). Metric: sub-pixel edge and seam-line fitting at full resolution, then POM construction by complete operators |
| **Attacks** | C and D strongly; E through the hybrid design; B only through protocol; A only if the operators are rewritten |
| **Evidence** | Sensing capability on rigid parts is established [E, Nogueira2023; InspecVision]. Performance on garments: **[H]** |
| **Throughput** | Capture in well under a second; processing seconds; handling dominates (tens of seconds for a careful lay [H]). Suits **S-AQL**; S-Inline only with fast handling |
| **Cost** | Tier 1 (€ low thousands to ~€10k hardware). Example price points: a 61 MP full-frame camera at US$3,900 MSRP [E, Prices]; an industrial 12 MP camera kit about US$4.8k [E, Prices]. Lenses, lighting, table and fiducial frame are order-of-magnitude estimates [I] |
| **Failure modes** | Perspective error from garment height if uncorrected; lighting-dependent edge location on rounded folds; low-contrast seams (dark on dark, fleece); garment colour vs background; consumer-camera instability (focus breathing, thermal drift) unless locked and verified |
| **Variant A1-D** | **Dual-sided capture:** a glass or acrylic table with a second camera below sees the back layer without flipping the garment. This removes a handling step for back POMs (across back, back neck width, CB length) [H; optical design needs checking for reflections and platen flatness] |

### A2 — Backlit transmission table (silhouette metrology) combined with A1

| | |
|---|---|
| **Concept** | Uniform LED backlight under a diffusing table top, as in industrial 2D gauges for flat and flexible parts [E, InspecVision]. The silhouette edge comes from transmitted light: independent of garment colour, prints and background choice. Front and raking light are used in alternate frames for seams |
| **Additional physics** | In transmission, **seam allowances, hems and pocket bags appear as darker multi-layer regions** in light or thin fabrics. That can reveal seam positions that are hard to see from the front [H — strongly fabric-dependent; opaque, dark or heavy fabrics will only give a silhouette] |
| **Attacks** | D (edge definition becomes a physical threshold rather than a learned mask); part of E (seam cues); removes the green-sheet chroma-key problem entirely |
| **Evidence** | Backlit gauging reaches ±35 µm over most of a 1.8 × 1.2 m table on flat stiff parts [E, InspecVision]. The garment edge will be far less sharp (fold radius, fuzz) [I] |
| **Cost** | Tier 1 (a large diffuse backlight adds a moderate cost to A1 [I]) |
| **Failure modes** | Multi-layer garments give a union silhouette (which layer's hem?); translucent fabrics blur; heat from the panel on long exposures (minor with LEDs) |

### A3 — Scanning acquisition: line-scan, contact image sensor or tiled cameras; conveyor-capable

| | |
|---|---|
| **Concept** | A line-scan camera or a CIS bar moves over the garment on a gantry, or the garment moves under it on a conveyor. Geometry is orthographic along the scan axis. Resolution is high (≈ 0.07 mm/px for a 16k line across 1.2 m [I]) |
| **Attacks** | C (perspective largely removed in one axis, and in both with telecentric line optics or CIS); enables **S-Inline** conveyor integration |
| **Evidence** | Scanning of flat stock is mature for documents and fabric webs. Commercial roll-fed wide-format scanners exist but pass media through rollers, so they are unsuitable for garments [E, Prices]. A garment-specific scanner would be a custom build **[H for performance]** |
| **Special error sources** | Scale along the scan axis depends on encoder or conveyor speed accuracy; belt stretch; the garment must stay flat and still on a moving belt; belt texture and colour affect contrast |
| **Cost** | Tier 2–3: moderate for a gantry station, high for a conveyor line; dominated by mechanics and integration [I] |
| **Failure modes** | Garment shifting during the scan; conveyor-induced wrinkles; synchronisation errors |

### A4 — 2D metrology plus low-cost flatness and height sensing

| | |
|---|---|
| **Concept** | Keep A1 or A2 as the metric instrument. Add a sensor whose only jobs are (1) to **detect non-flatness** (wrinkles, folds, lifted edges) and trigger re-lay or abstention, and (2) to supply a coarse **height map for perspective correction** (document 03 §2.2). Options: a laser light-section line, a stereo pair, photometric stereo from the raking lights already present in A1, or a consumer depth camera |
| **Rationale** | Wrinkles bias lengths short (document 03 §3.2). Detecting them is far easier than computing through them. Height correction needs only ~1 mm height accuracy to reduce the perspective term well below a millimetre at 1.5–2 m camera height [I] |
| **Attacks** | B (by rejection and re-lay, not correction) and C (perspective) |
| **Cost** | Tier 1 [I] |
| **Evidence** | **[H]** — the detection sensitivity needed (e.g. ridges ≥ 2–3 mm, wrinkle fields of 1 mm amplitude; document 03 §3.2) must be demonstrated |

### A5 — Structured-light 3D camera as the primary sensor (the previous team's class)

| | |
|---|---|
| **Concept** | A Zivid-class camera produces a colour point cloud. Measurements are taken on the 3D surface, geodesically or projected |
| **Specs** | Zivid 2+ L110: 2448 × 2048 colour sensor; 1090 × 850 mm field at 1.1 m; 0.44 mm spatial resolution; 260 µm point precision; dimension trueness < 0.35 % / < 0.40 % / < 0.60 % (70th percentile; focus distance / optimal range / full temperature range) [E, Zivid2+L110] |
| **Strengths** | Direct height (perspective correction, flatness detection); robust to colour and texture for *geometry*; mature industrial product; fast capture (50–1500 ms) [E] |
| **Weaknesses for this task [I]** | (1) 2D sampling coarser than inexpensive high-resolution 2D cameras. (2) Datasheet dimensional trueness is ≈ 2.5–4 mm over 700 mm unless re-verified in the measurement plane. (3) The field of view is marginal for trousers and outstretched long sleeves at the optimal distance (1090 × 850 mm). (4) Surface-path lengths from noisy depth are strongly biased unless heavily regularised (+27 % raw inflation at 0.44 mm spacing and 0.26 mm noise; document 03 §2.4). (5) It does not reduce definitional or handling components |
| **Where it is genuinely useful** | As a **research instrument** in the feasibility study, to *measure* garment state (flatness, wrinkle statistics, seam-roll height) and to quantify the perspective term. Possibly for garments that cannot be laid flat (A11) |
| **Cost** | Tier 2 (quote-based pricing [E, Prices]). The previous setup, including GPU, was ≈ €15k (user-reported) |

### A6 — Physically constrained lay: flattening, hold-down and tension fixtures

| | |
|---|---|
| **Concept** | Replace "hand-smoothed, without tension" with a **specified restrained state** in the ISO 10579 sense [E, ISO10579]. Options: (a) a perforated **vacuum table** that pulls the garment flat against the surface — standard in cutting rooms, where cutters work on "vacuum-compressed material" [E vendor, LectraParagon]; (b) a **glass or acrylic platen** lowered onto the garment; (c) **placement guides** (marked or projected alignment lines: CF on a table line, shoulder line on a guide); (d) **force-controlled stretch fixtures** for force-defined POMs (waist or neck extended) with a specified load; (e) a standardised **conditioning and timing** rule (e.g. measure ≥ N min after pressing) |
| **Attacks** | B directly (the only architecture that does); A for force-defined POMs (turns them into measurable quantities); C indirectly (less height variation) |
| **Crucial caveat** | **Every restraint changes the measurand.** Vacuum and platens flatten wrinkles but can also introduce strain (friction while the platen lands, vacuum drawing fabric into holes). Measurements in a restrained state may be biased relative to the contractual free-state method. That bias must be measured and either corrected or written into the buyer agreement (document 01 §3.4) [I] |
| **Evidence** | **[H]** — no published garment-measurement study with vacuum or platen states was found. The mechanism is plausible from fabric physics (document 03 §3) |
| **Throughput** | A vacuum table adds seconds; a platen adds handling; fixtures add minutes per force-defined POM [I] |
| **Cost** | Tier 1–2 (vacuum tables and platens are simple mechanically; force fixtures need design) [I] |

### A7 — Human-in-the-loop: the human does semantics, the machine does metrology

| | |
|---|---|
| **Concept** | Operator lays the garment (per protocol). The system proposes POM endpoints on a high-resolution image; the operator **confirms or corrects** each proposal with a touch or stylus. Alternatively, the operator places small removable markers at the endpoints before capture. The machine computes geometry at full resolution, with sub-pixel snapping to the nearest physical feature (edge, seam line) inside the operator's click region |
| **Attacks** | E (gross semantic errors virtually eliminated); C/D (machine geometry); A only if operators are trained on complete definitions. It does not remove human definitional spread, but makes it **measurable** (every correction is logged) |
| **Why it is attractive for S-AQL [I]** | It replaces the error-prone parts of manual measurement (tape handling, reading, transcription, stretching the garment with the tape) and keeps the part humans are good at (knowing which seam is the armhole seam). It is also a **data engine**: every confirmed or corrected point is a labelled training example tied to a real garment and a real definition, which is exactly what the previous team lacked. Automation can then be dialled up POM by POM as agreement statistics justify it |
| **Evidence** | **[H]** for garments. The analogue — experts clicking landmarks on images — has inter-observer spreads of about 1–2 mm on anatomical landmarks [E, Wang2016-ISBI; Kouchi2011] |
| **Throughput** | Seconds per POM for confirmation [H]. Slower than full automation, likely faster than tape [H] |
| **Cost** | Tier 1 (A1 or A2 hardware plus a touch screen) |

### A8 — CAD/template registration and constrained optimisation (a layer, not a sensor)

| | |
|---|---|
| **Concept** | Use pattern and CAD data (or a simulated flat-lay of the style) as a template. Register the observed contour and seam network to it (non-rigid registration); estimate all POM endpoints jointly subject to geometric constraints (seam topology, collinearity, known POM relationships) |
| **Benefits** | Robustness: missing or occluded features can be inferred; implausible detections are rejected; strong consistency checks for abstention |
| **Risk** | **Shrinkage toward nominal**, which hides exactly the deviations QC must find (document 03 §6). Symmetry constraints hide asymmetry defects |
| **Mandatory safeguard** | Use the template for *identification and plausibility checks*, not to *move* measured endpoints. Audit with seeded out-of-tolerance garments: the slope of measured vs reference deviation must be ≈ 1 (document 05) [I] |
| **Evidence** | Registration is mature in rigid and non-rigid inspection [E, Abenhaim2012 for non-rigid parts]. Garment use is **[H]** |

### A9 — Design for measurability: garments carry their own reference marks

| | |
|---|---|
| **Concept** | Borrow semiconductor "designed targets" and ISO 3759's marked reference points [E, ISO3759]: make the POM endpoints *physically marked* so definitional and semantic uncertainty vanish. **A9-V (validation):** marks applied by experts for experiments — UV-fluorescent invisible ink (as in Kouchi & Mochimaru's anthropometry method [E, Kouchi2011]), removable micro-dots, or sewn threads. **A9-P (production):** marks applied automatically at cutting (e.g. ink or UV marks tied to the pattern CAD) that survive sewing and remain unobtrusive |
| **Attacks** | A and E at the root |
| **Risks** | A9-P needs buyer acceptance (invisible to the consumer, wash-fast or wash-off, no chemical-compliance issues). Marks placed at cutting sit on pattern geometry; after sewing they reflect *pattern* locations, not necessarily finished-garment definitions (document 01 §5). Not all POM endpoints lie on a visible face |
| **Assessment** | A9-V is **essential** for the feasibility study. A9-P is a speculative research direction **[H]** |

### A10 — Robotic handling and flattening, with measurement (full S-Inline automation)

| | |
|---|---|
| **Concept** | Robots pick the garment, unfold and flatten it, and present it to an A1–A4 measurement station |
| **Evidence** | Cloth unfolding research optimises *coverage*, e.g. >80 % coverage within 3 actions for FlingBot [E, FlingBot]. Keypoint accuracy for manipulation is centimetre-level [E, Lips2024]. Neither is close to the reproducible metrological state that QC needs [I] |
| **Assessment** | Long-term. Only relevant once a manual-lay version (A1/A2/A6) has proven the measurement itself. Cost and complexity are high (Tier 3) [I] |

### A11 — On-form or inflated 3D measurement (brief)

A garment on a form, or inflated (the 1993 Levi Strauss patent inflates trousers with compressed air [E, US5530652A]), and captured multi-view, measures a *different measurand*: a restrained 3D state closer to "as worn". It suits fit assessment, not flat-lay spec POMs. Include only if a buyer defines POMs on-form [I].

---

## 2. Comparison matrix

Legend: ●● strong, ● partial, ○ none, with respect to reducing each error component; "—" not applicable.

| | A0 tape | A0+ digital tape | A1 hi-res 2D hybrid | A2 backlit + A1 | A3 scanning | A4 2D + flatness/height | A5 structured-light 3D | A6 constrained lay | A7 human-in-loop | A8 CAD layer | A9-V marks (validation) |
|---|---|---|---|---|---|---|---|---|---|---|---|
| **A definitional** | ○ | ○ | ○ (●● if operators rewritten) | ○ (same) | ○ (same) | ○ (same) | ○ (same) | ● (force-defined POMs) | ● (makes it measurable) | ● (consistency) | ●● |
| **B garment state** | ○ | ○ | ● (protocol only) | ● | ● | ●● (detect → re-lay) | ● (detects) | ●● | ● | ○ | ○ |
| **C sensing** | — | ● (reading) | ●● | ●● | ●● | ●● (height corr.) | ● (res., trueness) | ● | ●● | — | — |
| **D edges/seams** | human | human | ●● | ●● silhouette | ●● | ●● | ● | ● | ●● | ● | ●● |
| **E semantic** | human | human | ● (ML + abstain) | ● | ● | ● | ● | — | ●● | ●● (plausibility) | ●● |
| **F prior bias risk** | low | low | low if audited | low | low | low | low | low | low | **high unless audited** | none |
| **S-AQL fit** | baseline | ✔ | ✔ | ✔ | possible | ✔ | possible | ✔ | ✔✔ | layer | experiments |
| **S-Inline fit** | ✘ | ✘ | with fast handling | with fast handling | ✔ (conveyor) | with fast handling | possible | slows | ✘ | layer | ✘ |
| **Hardware tier** | 0 | 0 | 1 | 1 | 2–3 | 1 | 2 | 1–2 | 1 | software | consumables |
| **Main unknown** | reproducibility | reproducibility | B & A magnitudes | seam cues in transmission | flat on belt | detection sensitivity | value vs 2D | restraint bias | operator time | shrinkage | marking method |

**How to read it [I]:**
- No single architecture attacks A, B and C/D together.
- The combinations that plausibly do are: **A1 or A2 + A4 + A6** (with complete operators) for automated measurement, and **A1 or A2 + A7** for S-AQL, with A9-V used to validate either.
- A5 alone attacks the component that is *least* likely to be the bottleneck (sensing), and only partially.

---

## 3. Throughput per scenario

| Step | S-AQL | S-Inline |
|---|---|---|
| Handling (unfold, lay, smooth or flatten) | Operator, protocol-driven; dominant time [H] | Dominant constraint; needs conveyor feeding and/or fixtures; robotics is long-term |
| Capture | ≤ 1 s for 2D; 0.05–1.5 s for structured light [E, Zivid2+L110] | Same, or continuous (line-scan) |
| Processing | Seconds on a workstation GPU/CPU [I] | Must pipeline within takt |
| Back-side POMs | Flip (doubles handling) or dual-sided table (A1-D) | Dual-sided capture or skip back POMs |
| Abstention handling | Inspector measures by hand | Must be rare; routed off-line |

**The station takt time is set by handling, not by sensing or compute [I].** This is why the GPU budget of the previous system is not a structural requirement. Inference on a single image at the resolutions discussed needs modest hardware; training can be outsourced. The decisive S-Inline question is how fast a garment can be brought into a *reproducible, specified state* — a mechanical and procedural question.

---

## 4. Economics: is a €15k-class setup necessary?

### 4.1 Separate the cost layers

| Cost layer | What drives it | Magnitude |
|---|---|---|
| **Station hardware** | Camera(s), optics, lighting, table/backlight, fixtures, compute | Tier 1: € low thousands to ~€10k; Tier 2: ~€10–30k; Tier 3: tens to hundreds of thousands (conveyor or robotic lines) [I, order-of-magnitude] |
| **Development** | Measurement definitions, algorithms, validation | The previous team spent ~2 years. That is the dominant cost and the one most at risk of being wasted if the physical feasibility envelope is not established first |
| **Per-station deployment** | Installation, calibration artefacts, training, verification schedule | Modest but recurring [I] |
| **Operation** | Operator time per garment, abstention handling, re-calibration, drift monitoring | Scales with volume |

### 4.2 The hardware question, answered from the physics

- **For flat-lay spec measurement, a structured-light 3D camera is not necessary as the primary metric sensor** [I, from document 03 §2]:
  - Its 2D sampling is coarser than a high-resolution 2D camera costing a fraction as much.
  - Its datasheet dimensional trueness is not better than the target, so it must be re-verified anyway.
  - It does not reduce the components most likely to dominate (A and B).
- The two genuinely useful 3D functions — **height correction** and **flatness detection** — can plausibly be obtained with cheaper means (A4) **[H — to be confirmed in S1/S3 by running the Zivid and a 2D camera side by side]**.
- **A €15k station is not intrinsically unreasonable** for industrial inspection. The objection is not the price but the **mismatch between what was bought and what limits accuracy**. Money spent on state control (A6), calibration artefacts and a validation reference would likely buy more accuracy per euro than money spent on 3D sensing [H].

### 4.3 A value model to fill in during the pilot

Per station and year (symbols to be measured, not assumed):

```
Value  =  N_meas · (t_manual − t_auto − p_abst · t_manual) · c_labour        (inspector time saved)
        + N_meas · Δp_escape · c_escape                                      (fewer bad garments shipped)
        − N_meas · Δp_false_reject · c_false_reject                          (extra good garments rejected)
        + V_data                                                             (SPC / earlier drift detection; disputes avoided)

Cost   =  C_hw / L_hw + C_maint + C_calib + N_meas · t_auto · c_labour + C_dev / (N_stations · L_dev)
```

- **t_manual, t_auto** — time per garment for the full POM set (manual tape vs automated station, including handling).
- **p_abst** — abstention rate (garments or POMs routed back to manual).
- **Δp_escape, Δp_false_reject** — changes in false-accept and false-reject rates. They depend on uncertainty *and* production spread (document 01 §9).
- **c_escape** — cost of an escaped defect (chargebacks, returns, claims). Buyer-specific.
- **C_dev** — development cost, amortised over stations and lifetime.

**Sensitivities that matter [I]:**
1. **Abstention rate p_abst** directly erodes labour savings. An automated system that abstains on 30 % of garments saves less than one that confirms 100 % quickly with a human in the loop (A7).
2. **Guard-banding** (to limit consumer's risk) raises false rejects steeply once u/t exceeds about 0.1 (document 01 §9). Improving uncertainty from, say, 0.2t to 0.1t can be worth more than any labour saving.
3. **Development cost amortisation** favours architectures that reuse standard components and minimise custom learning (A1/A2/A7) over bespoke 3D or robotic stacks.

No monetary values are inserted here. The pilot (document 05) measures t_manual, t_auto, p_abst and the production spread. Labour rates and escape costs come from the factory and buyer.

---

## 5. Summary of the architectural position

| Statement | Status |
|---|---|
| The limiting components (definition, garment state) are not addressed by sensing sophistication alone | [I] from document 03 |
| A calibrated high-resolution 2D system (A1/A2) can make the sensing component small relative to the target | [I], supported by [E, InspecVision; Nogueira2023] on rigid parts; garment performance [H] |
| Learned models should propose, geometry should measure, and low-quality measurements should abstain | [I] |
| A physically constrained state (A6) is the only direct lever on handling variability, but it changes the measurand | [I]; magnitude [H] |
| Human-in-the-loop semantics (A7) is the lowest-risk route to a deployable S-AQL system *and* the best way to build a truthful training set | [H] |
| A structured-light 3D camera is most valuable as a research instrument for quantifying garment state, not as the production metric sensor for flat-lay POMs | [H], to be tested |
| Full S-Inline automation depends on fast, reproducible garment handling, which is not solved by current robotics research | [E, FlingBot; Lips2024] + [I] |

---

## Sources cited in this document

Full annotated entries in [references.md](references.md).

- `[ISO18890]` https://cdn.standards.iteh.ai/samples/63693/131110d2f792493aabb7200cbd9136ba/ISO-18890-2018.pdf
- `[ISO10579]` https://cdn.standards.iteh.ai/samples/54707/4d70c7544c894dce8a99e89b862cdf09/ISO-10579-2010.pdf
- `[ISO3759]` https://cdn.standards.iteh.ai/samples/57309/1af372753bf84b3cb0288a274429c73d/ISO-3759-2011.pdf
- `[DigitalTapes]` Fujitsu apparel digital tape (via https://www.thetailoredco.com/digital-clothes-measuring-tools/); ALT https://www.bagel-labs.com/product/alt-smart-tape-measure-for-garments/ ; Fingertip https://www.getfingertip.io/features/digital-tape ; US 2014/0101948 https://patents.justia.com/patent/20140101948
- `[Paulauskaite2022]` https://doi.org/10.3390/app12094470
- `[Nogueira2023]` https://pmc.ncbi.nlm.nih.gov/articles/PMC10346874/
- `[InspecVision]` https://www.inspecvision.com/planar2d-models/planar-p220.35
- `[Prices]` Sony α7R V MSRP (Engadget) https://www.engadget.com/cameras/sonys-61mp-a7r-v-mirrorless-camera-is-on-sale-at-its-lowest-price-ever-113045739.html ; Basler boost kit (Edmund Optics) https://www.edmundoptics.com/p/basler-boost-boa4112-68-color-camera-kit/42275/ ; Zivid quote request https://www.zivid.com/quote-request ; wide-format roll-fed scanners (vendor listings via search)
- `[Zivid2+L110]` https://www.zivid.com/hubfs/User%20guides%20and%20datasheets/Zivid%202+%20L110%20Datasheet.pdf
- `[LectraParagon]` https://www.lectra.com/en/fashion/products/gerber-paragon-fashion
- `[Wang2016-ISBI]` via https://pmc.ncbi.nlm.nih.gov/articles/PMC12313948/ · `[Kouchi2011]` https://pubmed.ncbi.nlm.nih.gov/20947062/
- `[Abenhaim2012]` https://link.springer.com/article/10.1007/s00170-012-3929-2
- `[FlingBot]` https://arxiv.org/abs/2105.03655 · `[Lips2024]` https://arxiv.org/abs/2401.01734
- `[US5530652A]` https://patents.google.com/patent/US5530652A/en
