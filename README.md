# eitg-demo
Frontend demonstration of an AI powered environmental detection,monitoring,predictive analytics platform designed to generate probability based forecasts up to 15 days in advance using historical and real time data monitoring.

THIS REPO CONTAINS A FRONTEND DEMONSTRATION OF A PROJECT.

THE BACKEND,INFRA,DB AND COMPONENTS ARE NOT INCLUDED.

Feature Domains
1. Real-Time Air Quality Monitoring
16-Zone AQI Dashboard — Delhi NCR divided into Z01–Z16 (~81 km²/zone). Live AQI computed via CPCB 2014 formula (6 pollutants → worst sub-index) refreshed every 5 minutes via PyFlink tumbling window. WebSocket delta push to all connected clients.
5,184 Smart Pole Network — Uniform ~500m spacing. Each pole reports PM2.5, PM10, NO₂, SO₂, Cl₂, H₂S, NH₃, PID/VOC, CO, and gamma dose rate via MQTT/TLS (QoS=1). Named P{ZZ}-{NNN}. 180-second heartbeat timeout for offline detection.
Gas Time-Series Charts — Per-pole 6-hour channel history with anomaly threshold reference lines, zoom/pan, and sparkline trend display.
CEMS Integration — 20+ industrial stack monitors (refineries, power plants) via Modbus TCP / OPC-UA. Tracks SO₂, NOₓ, CO, CO₂, HCl, PM, NH₃, flow, pressure, temperature. 7-year retention in ClickHouse for regulatory compliance.
Wind Field Overlay — Animated wind barbs from Vaisala WMT700 meteorological towers; real-time speed (m/s) + direction.
Smog Probability Forecasting — 6-factor weighted score: AQI normalised + inversion likelihood + wind stagnation + humidity + stubble proximity + AOD. Calendar-month and season-aware.
Emission Source Attribution — Real-time breakdown: vehicular, industrial, stubble burning, construction, other.
Health Impact Assessment — WHO concentration-response functions for PM2.5, NO₂, O₃. Computes excess mortality per day and DALY (YLL + YLD) per zone. Population risk tiers: general, children, elderly, respiratory, cardiovascular.

3. Advanced Atmospheric Modelling
Gaussian Plume Dispersion — Briggs (1973) σy/σz coefficients, Briggs (1975) plume rise, Pasquill-Gifford stability classes A–F, ground reflection, wind profile power-law height correction. Foundation for dose accumulation, wargaming, STE, and UAV vector analysis.
Photochemistry (NOx–VOC–O₃) — Solar-angle-dependent J(NO₂), O₃ formation and titration, SO₂ → secondary PM2.5, NH₃ + H₂SO₄ → ammonium sulphate aerosol. Delhi traffic peak patterns (O₃ 13:00–15:00 IST).
Urban Canyon Correction — Street aspect ratio H/W classification (isolated / wake interference / skimming). Concentration enhancement up to 1.8× in skimming regime (e.g. Z08 Parliament area).
Nocturnal Inversion Modelling — Stability class F produces narrowest high-concentration plumes; integrated into dispersion solver.
Sensor Placement Optimisation — Particle swarm optimisation (PSO) minimises mean TIC detection latency across 25 threat scenarios × 16 source zones. Outputs recommended additional pole zones and latency improvement percentage.

5. Chemical Threat Detection
Anomaly Detection — Per-zone per-pollutant exponentially weighted moving average (α=0.05) with Welford online variance. Z-score thresholds: 2.0–3.5 = ELEVATED, 3.5–5.0 = HIGH, >5.0 = CRITICAL. Delhi-specific baseline priors.
TIC Fingerprinting (25-Agent Library) — 6-channel sensor vector matched via cosine similarity against 25 toxic industrial chemicals including Cl₂, phosgene, HCN, sarin precursor, mustard precursor, H₂S, NH₃, radioactive tracer. Confidence threshold 0.65. Rate-of-rise classification (instantaneous/continuous/gradual). Severity mapping to ERPG-1/ERPG-2/IDLH.
Sensor Spoofing Detection — Three independent methods: Mahalanobis distance (squared distance >20 = anomalous), physical consistency check (upwind must exceed downwind 2×), coordinated drift detection (≥3 poles identical drift = artificial). First defence: HMAC-SHA256 signature on every Kafka message.
Source Term Estimation (Inverse Plume) — Bayesian particle filter (N=500) iteratively inverts the Gaussian plume to estimate source location (lat/lon/zone), emission rate (g/s), release height (m), stability class, and confidence. Used for unknown/covert release identification.
Agent Mixture Detection — NNLS (non-negative least squares) deconvolution for 2-component agent mixtures from 9-channel sensor vectors. Six known patterns including white_star (Cl₂ + phosgene), nerve_precursor_binary, chloramine_cloud.
Surface Contamination & Persistence — Full physics chain: deposition flux → Arrhenius evaporation → hydrolysis (pH/humidity-dependent). Five agents modelled (VX, mustard, lewisite, sarin, chlorine) across four surface types (impermeable, porous, vegetation, water). Safe re-entry time: 3× t_IDLH for skin-hazard agents.
Dose Accumulator (Haber's Law) — Zone-level cumulative dose: C(ppm) × Δt(min). Per-agent independence; worst-agent sets AEGL level. 60-point rolling timeline per zone. Thread-safe with lock.

6. Radiological & Nuclear
Gamma Dose Rate / KERMA Accumulation — Mirion RDS-31 Geiger-Müller detectors on poles (CPM → dose rate conversion). Priority isotopes: Cs-137, Sr-90, I-131, Co-60. ICRP-103 dose limits (1 mSv/yr public / 20 mSv/yr worker / 250 mSv emergency). IAEA protective action levels: shelter at 1 mSv/h, KI at 10 mSv/h, evacuate at 10 mSv/h, emergency evacuation at 100 mSv/h.
ICRP-60 Committed Organ Dose — Three exposure pathways (inhalation, ingestion, external). Seven nuclides (Cs-137, I-131, Sr-90, Co-60, Pu-239, Am-241, Po-210). Tissue weighting factors applied. PAL triage levels: BELOW_PAL (<10 mSv), PAL1 (10–100), PAL2 (100–1000), PAL3 (>1000 mSv).
Radiological Plume Modelling — Pre-incident fallout dispersion; feeds dose accumulator and protective action engine.

7. Biological & Aerosol
Bio Aerosol Viability Decay — Seven agents: anthrax, plague, tularemia, Q fever, smallpox, VEE, botulinum. Physics: decay constants k_uv (UV kill), k_temp (thermal kill), t_half_dark_min (intrinsic half-life), id50 (infectious dose 50). Environmental modifiers: direct sunlight (rapid kill), heat >35°C (accelerated), humidity >80% (slight acceleration). Clearance time computed to <1% viability for safe re-entry. Triggered by PID spike (aerosolisation event) + syndromic surveillance surge 48–72h later.
Syndromic Surveillance — ER visit rate monitoring per zone. Poisson regression model: E[Y] = exp(α + β_AQI·AQI + β_season + β_elderly). Z-score >2.5σ above seasonal baseline triggers bio alert. Critical for detecting covert biological release in the first 4 hours before sensor confirmation.

8. Threat Attribution & Analysis
Bayesian Attack Attribution Classifier — Seven evidence nodes with individual likelihood ratios: off-hours release (LR 3.5), no facility inventory match (LR 4.0), UAV sighting (LR 12.0), multi-point simultaneous release (LR 8.0), wind-inconsistent plume direction (LR 2.5), spike magnitude (LR 2.0), prior intelligence warning (LR 15.0). Log-space Bayesian update → softmax → P(deliberate), P(accident), P(natural) + confidence level + dominant hypothesis.
Cross-Incident Correlation Engine — Three signals: temporal proximity (30-min window), spatial clustering (30 km radius haversine), chemical profile similarity (cosine on agent channels). Composite score 0.4×temporal + 0.3×spatial + 0.3×chem_sim; threshold 0.45. Attack pattern classification: SINGLE / LINEAR (upwind release line) / RADIAL (surround attack) / GRID (maximum coverage).
Probit Casualty Prediction — Ten agents (Cl₂, phosgene, HCN, sarin, VX, mustard, NH₃, H₂S, ClO₂, SO₂). Log-normal dose-response: P(effect) = Φ(a + b·ln(Ct) − 5). Three thresholds per agent: lethality (L), incapacitation (I), sensory (S). Triage categories: T4_EXPECTANT (>60% lethal), T1_IMMEDIATE (>50% incapacitated), T2_DELAYED (>20% incapacitated), T3_MINIMAL. Outputs ±30% confidence intervals.
UAV Vector Analysis — Inverts Gaussian plume to find probable UAV launch position, computes intercept corridor and bearing (degrees FROM source TO launch zone), estimates minimum payload size and release duration, produces confidence metric.
Counter-UAS Track Integration — DoD UAS group classification (GROUP1 <9 kg, GROUP2 <25 kg, GROUP3 <600 kg). RF signature Bayesian priors: P(chemical) 0.40–0.45 for custom RF, 0.60 for silent/GNSS-only. Engagement recommendations: EVACUATE_ONLY, KINETIC_INTERCEPT, GNSS_SPOOF, RF_JAM, NET_GUN, LASER_DAZZLE. Protected asset registry (EOC, AIIMS, Parliament, India Gate, Pusa Lab).

9. Response Planning & Incident Management
Shelter-in-Place vs Evacuation Engine — Physics-based decision: compare plume arrival time vs zone clearance time × 1.3 safety buffer. Already-exposed zones (AEGL>0) or source zone → SHELTER. Marginal → UNCERTAIN (defaults to shelter). Zone-specific clearance rates: Z13/Ridge 5.3 min, Z03/Gurgaon 14.2 min, Z09/Old Delhi 62.2 min (150 persons/min, 50% urban shelter effectiveness).
Evacuation Routing (Modified Dijkstra) — 16-zone adjacency graph with highway + secondary road edges. Route scoring: edge_time × (1 + contamination_penalty) × capacity_factor. Contamination penalties: AEGL-1 = 2×, AEGL-2 = 5×, AEGL-3 = 100×. Routes automatically avoid contaminated zones.
Wargaming Engine — Parallel scenario exploration across source zone × wind direction × release mass × agent type combinatorics. Thread pool execution (max_workers = CPU count). Output ranked by population impact. Pre-monsoon planning and worst-case analysis.
ICS Incident Command — NIMS/ICS compliant. Eight roles: IC, OPS, SAFETY, PIO, LOG, HAZMAT, MED, EVAC. Five incident types with task templates: CHEMICAL_RELEASE (12 tasks), UAV_CHEMICAL_DELIVERY, BIOLOGICAL_EVENT, MASS_CASUALTY, INDUSTRIAL_ACCIDENT. Four escalation levels: LOCAL → DISTRICT → STATE → NATIONAL. Persistent DB storage with in-memory active incident cache.
Responder Dosimetry Tracker — PPE protection factor modelling (Level A 0.001, B 0.01, C 0.05, D 0.50). NIOSH IDLH thresholds (30-min basis) per agent. Status thresholds: CLEAR (<25%), MONITOR (25–50%), ROTATE (50–100%), EVACUATE (≥100%). Rotation advisory triggered at 70% IDLH.
Decontamination Priority Queue — Zone ranking by decon urgency. Surface-specific methods: high-pressure wash (impermeable/concrete), foam + rinse (mixed), excavation (porous/soil). Resuspension risk estimation included.
Cascading Infrastructure Failure Simulation — 20-node Delhi NCR dependency graph (POWER, WATER, COMMS, TRANSPORT, HOSPITAL, EOC, FUEL, SUPPLY, SHELTER). BFS propagation with edge weights + node redundancy factors. Response capability score: Σ(service_level × criticality) / Σ(criticality) × 100. Recovery sequence optimisation.
Cloud Seeding Feasibility Assessment — 14 weighted meteorological criteria (cloud cover, base height, liquid water path, updraft, RH, wind, AQI trigger, aviation clearance, etc.). Weighted score 0–100: ≥65 = Recommended, 35–64 = Marginal, <35 = Not Viable. Expected rainfall (mm) and AQI reduction estimates. Accessible from the Interventions panel.

10. MOPP & Medical Countermeasures
MOPP Level Decision Engine — Five protection levels (MOPP0–4). Decision logic: AEGL level sets floor (AEGL-1 → MOPP2, AEGL-2 → MOPP3, AEGL-3 → MOPP4), agent-specific floor (sarin/VX/HCN → MOPP4 in 9 seconds), downwind amplifier, P(deliberate) >0.70 adds +1 level. Heat stress check triggers rotation when WBGT exceeds limit. Work-rate degradation: MOPP1 90%, MOPP2 80%, MOPP3 65%, MOPP4 50%.
Medical Countermeasures Inventory — Catalogue: MARK1 autoinjector, DuoDote, Cyanokit, BAL, RSDL, KI tablets, DTPA, ciprofloxacin, MOPP suits (Levels A–D). Sources: NDMA strategic reserve, AIIMS, Pusa depot, MoH stockpile, WHO push pack. Demand calculation: casualties × dose/casualty × safety multiplier. Coverage score = available / demand.

11. NATO-Standard Reporting
STANAG SITREP Generator — NATO STANAG 2954 / MC 0259 six-line format. Auto-numbered (SITREP-001, 002...). Auto-classification: UNCLASSIFIED → RESTRICTED → CONFIDENTIAL → SECRET based on agent type and P(deliberate). Exercise SITREPs tagged UNCLASSIFIED // EXERCISE. Triggers: manual button, escalation level change, every 30 min during Level 3–4 incidents.
NBC Message Suite (NBC1–NBC6) — Full message set per NATO doctrine: NBC1 (initial observer report), NBC2 (collated reports), NBC3 (downwind hazard prediction with AEGL level + arrival time), NBC4 (survey measurements in contaminated zone), NBC5 (radiological fallout dose rates), NBC6 (all-clear / re-entry authorisation). DTG format: DDHHMMZ MON YY.
Field Team Integration — Full lifecycle: register → deploy → beacon (10-second interval) → spot reading → marker placement. Marker types: HOT_BOUNDARY, POINT_SOURCE, CASUALTY_LOCATION, SAFE_PASSAGE, DECON_SITE. Medevac priority triage: P1_URGENT (litter), P2_PRIORITY (ambulatory), P3_ROUTINE. UHF guard 243.0 MHz (NATO standard).
26-Paradigm Neural AQI Forecasting Engine
The EITG Neural Engine is a unified PyTorch model with 26 sequential scientist stages sharing a D=128 latent space, producing 15-day zone-level AQI forecasts across all 16 zones.

13 Core Paradigms: Perceiver, Graph, AFNO , Swin Transformer , Mamba SSM, Neural ODE, ConvLSTM, Diffusion model, Chemical Transport, Wavelet decomposition, Sparse MoE, KAN , Liquid PBL.

13 Advanced Scientists: Bayesian, Optimal Transport, Topological, Tucker Decomposition, Riemannian, Harmonic Oscillator, Koopman Operator, Reservoir Computing, Scattering Transform, Hopfield Network, Fractional Diffusion, Langevin Dynamics, Cellular Automata.

A total of 26 advance sciencetist using Maths,Data Science,Atmospheric Chemistry,etc to determine the outcome.

CrossParadigmGate: Softmax per-zone weighting of all 26 residuals. Consensus mechanism suppresses outlier scientists. SADANLayer provides spatial attention + seasonal domain adaptation (monsoon, stubble, winter_fog, pre_monsoon regimes).

Inputs: 18 variables z-scored — PM2.5, PM10, NO₂, SO₂, CO, NH₃, O₃, Pb, benzene, toluene, HC, AOD, dust, temp, humidity, wind_u, wind_v.

Outputs:

15-day AQI forecast per zone
Aleatoric uncertainty σ
Conformal prediction bounds (guaranteed 90% coverage)
Quantile estimates: q05, q25, q50, q75, q95
Per-paradigm contribution weights (which scientist is dominant today)
