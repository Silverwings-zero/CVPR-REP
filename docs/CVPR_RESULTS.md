# CVPR 2027 — results sheet (native protocol; status 2026-10-26)

**HARDWARE TIMELINE (confirmed 2026-10-26): S1 = unit 1 + gel 1 (Jun 15 – Jul 9: A, B, Wood); S2 = unit 2 +
gel 2 (Jul 10 – Oct 9 18:42: Cork/FiberGlass/ABS/Ceramic in July; the 15 A/B materials + Canvas in Sep–Oct);
S3 = unit 2 + gel 3 (Oct 9 19:37 on: the five rigid materials). "Campaign D" = S2 (15 materials, cross-unit
from S1) + S3 (5 rigid materials, cross-gel from S2). Fold D by hardware (sup raw/prev/blank): 15 cross-unit
8.4 / 52.0 / 61.6; 5 cross-gel 43.8 / 48.9 / 39.4; jitter 49.1 / 62.9 vs 33.7 / 33.8; Sparsh 5.0 / 39.4 /
adapter 36.6 vs 30.0 / 23.6 / 31.3. Signature by hardware relation: own 0.53, same day 0.40, same hw weeks
0.39, same hw months 0.32, other unit 0.15, other gel 0.16 (`signature_hardware.json`).**

**Paper protocol since 2026-10-25: single native frames; heads trained on embeddings calibrated with
their own previous session whenever the test embeddings are calibrated (consistent protocol,
`tacrep_calib_eval.py --train-prev`); raw rows = raw at both ends. The 8-frame numbers in T1 are kept
for the frame ablation only. T6 holds the reviewer controls and the hold-out.**

Every number below is from the final protocol (native 192-px crop-tubes with 8-frame
integration for material; whole native frames for force and contact). Raw runs and the full
history: `docs/TACREP_LOOP.md`. Macro accuracy / macro-F1 (%), 20-way unless stated, chance 5 %;
"collapse" = share of test episodes assigned to the single most predicted class.

## T1. Material, June–July → Sep–Oct (eras A+B+C → D)
| encoder | raw | + stream calibration, previous session (deployable) | + stream calibration, own session (upper ref.) | collapse (raw) |
|---|---|---|---|---|
| DINOv2-base (frozen) | 22.0 / 18.9 | 28.3 / 24.5 | 39.4 / 35.7 | 37 % |
| Sparsh-DINO (frozen) | 8.9 / 4.7 | 26.9 / 25.4 | 33.4 / 31.8 | 47 % |
| Sparsh-MAE (frozen) | 10.0 / 5.0 | 31.6 / 28.4 | 38.1 / 35.4 | 49 % |
| Sparsh-I-JEPA (frozen) | 11.2 / 6.9 | **35.5 / 32.2** | **40.8 / 37.5** | 52 % |
| TactSpace replica (2 seeds) | 10.1, 8.7 | — | — | 47–53 % |
| normal-map alignment (native) | 22.9 / 19.5 | — | 24.8 / 18.8 | 46 % |
| supervised CNN (2 seeds) | 15.7, 15.4 / 9.0, 8.1 | 35.8 / 31.3 | 31.4, 30.1 / 26.5, 26.2 | 43–46 % |
| supervised + photometric jitter | 23.6 / 18.6 | — | 37.7 / 34.0 | 39 % |
| supervised + state swap | 18.8 / 13.1 | **39.9 / 34.9** | 36.8 / 34.1 | 47 % |
| supervised on calibrated inputs (2 seeds) | — | 26.3 / 22.2 (mismatched source) | 38.1, 38.9 / 34.7, 34.5 | — |
| label-free state swap, base (3 seeds) | 27.1 ± 2.3 / 23.3 ± 2.0 | 27.0 / 24.2 (seed 0) | 27.6, 27.9, 26.6 | 18–27 % |
| **label-free state swap + decoys** (3 seeds) | **27.6 ± 2.2 / 26.3 ± 1.4** | — | 29.6, 27.0, 26.8 | 15–20 % |
| label-free swap + normal map | 28.2 / 23.8 | — | 32.7 / 28.4 | 38 % |
| **frozen Sparsh-I-JEPA + label-free state-swap adapter** (3 seeds) | **34.9 ± 0.6 / 31.8 ± 0.4** | **36.4 / 33.7** | — | 19–20 % |
| frozen Sparsh-MAE + adapter | 32.6 / 28.5 | 31.4 / 27.3 | — | 20 % |
| frozen Sparsh-DINO + adapter | 28.5 / 22.9 | 31.4 / 27.3 | — | 22 % |
| ablation: Sparsh-I-JEPA adapter without decoys | 31.9 / 28.2 | 36.3 / 33.9 | — | 24 % |
| frozen DINOv2 + adapter (general-vision) | 25.9 / 24.9 | 28.9 / 24.9 | — | 33 % |
Adaptation baselines (supervised CNN): AdaBN over the whole test era (20 objects in one state)
40.2, 39.5; **AdaBN per session (one object — what a robot has) 4.2–8.7 for every model**.
Era-within-material probe (chance 57.7, lower = more day-invariant): label-free swap 66–67,
supervised 77–79, TactSpace 83–84, Sparsh raw 77–79 → calibrated 62–69.

Statistics (era D has one session per material, so materials are the unit): 95 % bootstrap
intervals of single numbers are wide (e.g. adapter 35.2 [22.5, 48.8], supervised raw 15.7
[4.0, 30.5]); paired bootstrap of differences over the 20 materials:
| comparison | Δ acc | 95 % CI |
|---|---|---|
| swap adapter (zero-shot) − frozen Sparsh-I-JEPA | **+24.0** | [+13.9, +35.9] |
| previous-session calibration − raw (Sparsh-I-JEPA) | **+24.2** | [+8.2, +41.5] |
| previous-session calibration − raw (supervised CNN) | **+20.1** | [+6.0, +34.7] |
| label-free decoy swap − TactSpace replica | **+19.2** | [+4.2, +33.5] |
| label-free decoy swap − supervised raw | +13.6 | [−2.1, +28.7] |
| Sparsh adapter − from-scratch decoy swap | +5.9 | [−4.0, +15.8] |
| adapter + calibration − adapter zero-shot | +1.9 | [−7.3, +10.8] |
| adapter with decoys (3-seed mean) − without decoys | **+3.1** | [+0.5, +6.1] |
| Sparsh-MAE adapter − frozen Sparsh-MAE | **+22.6** | [+10.5, +35.9] |
| Sparsh-DINO adapter − frozen Sparsh-DINO | **+19.5** | [+8.5, +31.4] |
| DINOv2 adapter − frozen DINOv2 (general-vision) | +3.8 | [−4.3, +11.5] |

## T1b. Frames integrated per native crop (supervised CNN; raw / previous-session calibration)
| frames | 1 | 2 | 4 | 8 |
|---|---|---|---|---|
| raw | 17.3 / 11.1 | 15.2 / 8.5 | 14.0 / 8.1 | 15.7 / 9.0 |
| + previous-session calibration | **50.8 / 45.9** | 43.9 / 39.0 | 37.8 / 32.8 | 35.8 / 31.3 |
Single-frame protocol (A+B+C → D; raw / previous-session calibration; consistent head protocol):
| model (1 native frame) | raw (3 seeds where stated) | + prev.-session calibration | + campaign-mean state |
|---|---|---|---|
| supervised (3 seeds raw; calibration seed 0, seeds pending) | 16.6 ± 0.8 / 10.5 | **51.2 / 46.3** | **57.1 / 50.6** |
| supervised + state swap | 28.4 / 22.1 | 52.4 / 46.5 (raw-head protocol) | — |
| supervised + photometric jitter (3 seeds raw) | **43.9 ± 2.3 / 40.3** | **55.6 / 51.4** | 52.0 / 45.1 |
| supervised + jitter + state swap | 44.1 / 39.2 | 54.6 / 48.0 (raw-head protocol) | — |
| supervised + DANN (session GRL) | 18.3 / 11.7 | 52.9 / 48.0 | — |
| label-free decoy swap (3 seeds) | 26.5 ± 1.6 / 23.6 | 31.0 / 27.8 (seed 0) | — |
| (jitter with 8-frame inputs, 2 seeds) | 22.8 / 17.7 | 40.7 / 35.9 | — |
Single frames beat integrated frames on every new session, within or across campaigns (T6); white
pixel noise added to 8-frame means does NOT reproduce it (T6). With single frames, standard
photometric jitter alone removes most of the collapse (44 raw; 23 with 8-frame inputs) but is a
cross-state regulariser (+3 within the campaign, +27 across); jitter + calibration is the best
deployable pipeline with labels; the adapter (34.9) remains the best option without labels.
Resolution (single frame, supervised, raw / prev.-cal.): native 16.6 / 51.0 ± 0.5; 1/2 21.5 / 52.9;
1/4 23.1 / 49.6 — native resolution gives no measurable advantage for material across days.

## T2. Time gap, same 15 materials, trained on June (campaign A) only (15-way; single frame, 2026-10-25)
| model | → July (2–3 weeks, same campaign) raw / own-cal / prev | → Sep–Oct (3–4 months, next campaign) raw / own-cal / prev |
|---|---|---|
| supervised, 1 frame | 68.4 / 69.9 / **80.6** | 39.1 / 43.5 / 37.3 (online 41.7) |
| supervised + jitter, 1 frame | 71.0 / 70.4 / 73.3 | 43.4 / 42.4 / 42.5 |
| (eight-frame, old) supervised | 45.3 / 59.3 / — | 23.5 / 35.2 / — |
| (eight-frame, old) frozen Sparsh-I-JEPA → + swap adapter | 42.9 → 49.2 | 22.8 → 31.0 |
The one-session-per-material training set does NOT collapse across the break (18 % in one class vs
43 % for A+B+C) and calibration does not lift it there (39 → 37–43); within the campaign +12. AdaBN
(whole era) +9 on B, −9 on D. Realistic calibration sources for the A+B+C model (single frame, consistent):
previous session 50.5 ± 0.7 (3 seeds); online 46.6; own session 47.7; bare gel 56.1; campaign mean 54.9.
## T3. Normal force (Nano17), log fn
| input | split | R² | r |
|---|---|---|---|
| learned whole-frame representations (label-free, supervised, force-supervised) | A–C → D | ≤ 0.11 | ≤ 0.42 |
| model-free appearance deformation vs a fresh unloaded reference (blank) | D early → late | 0.09–0.22 | 0.36–0.47 |
| same, stale unloaded reference (previous / older session) | D early → late | −1.9 … −17.7 | — |
| **marker geometry vs the unloaded grid** (affine: divergence, mean displacement, shear) | D early → late (fresh / previous-session grid) | **0.43 / 0.48** | **0.66 / 0.71** |
| marker geometry, one borrowed unloaded grid | June–July → Sep–Oct (same gel) | **0.48** | **0.72** |
| marker geometry | Sep–Oct → June–July | 0.19 | 0.54 |
| marker geometry, old-gel map on the new gel | new gel grid / stale old grid | 0.05 / 0.36 | 0.71 / 0.70 |
Within-session Spearman (fn vs marker divergence) median 0.70. A new gel keeps the ranking but
changes the force-to-displacement scale (stiffness).

## T4. Contact (heads trained on the 7 lead-in sessions; tested on the other era-D sessions)
| detector | contact recall | bare-gel specificity (old gel / new gel) |
|---|---|---|
| differencing against the session's blank (no learning) | 100 | 88.6 |
| head on frozen whole-frame features (any model) | 98–100 | **0** (0 / 0) |
| head on z − z(session's bare gel), label-free swap model | 100 | 70.0 (77 / 59) |
| same, force + swap model | 99.9 | **86.2 (91 / 79)** |

## T2b. Second unit, method (Real Tactile MNIST: digit from single touches, 10-way, chance 10)
| event | row | raw | previous-run calibration | AdaBN per run |
|---|---|---|---|---|
| later days, same gel | supervised | 19.6 / 11.8 (47 % collapse) | 19.9 / 18.0 | 24.1 / 21.1 |
| | frozen Sparsh-I-JEPA | 10.0 / 4.2 (84 %) | 17.5 / 16.5 | — |
| | label-free swap | 17.4 / 13.2 | 19.8 / 19.1 | 25.1 / 21.4 |
| gel swap | supervised | 24.2 / 20.8 (37 %) | 27.3 / 25.5 | 32.0 / 30.8 |
| | frozen Sparsh-I-JEPA | 13.2 / 8.0 (52 %) | 19.8 / 18.4 | — |
| | label-free swap | 13.4 / 9.9 (58 %) | 20.2 / 18.7 | 17.2 |
Calibration transfers to a second unit (collapse removed, F1 up on every encoder); per-run AdaBN
works there because every run holds all ten digits (the control for our one-object-per-session
collapse); the label-free swap/adapter is a texture result on our unit, not a cross-unit one.

## T4b. Check: does the state field carry the day?
Era classifier (June vs July, 15 materials in both; balanced accuracy 76.6 %), held-out contacts
classified as July: real June 5.9 % · June re-rendered with another June state 7.5 % · **June
re-rendered with a July state 35.3 %** · real July 52.4 % (crops; episodes 0 / 0 / 19.3 / 53.1).

## T5. Measurements (model-free)
- CAMPAIGN signature (55 sessions; 190–370 µm band; markers masked; all partner pairs): a contact
  matches the position-wise mean of its own session's other contacts at r 0.53, of a different object
  the same day 0.41, of another day of the SAME campaign 0.38, of another campaign 0.18, of the other
  gel 0.16 → a campaign-level (weeks) rendering state. PRESENT IN THE BARE GEL: bare-gel frames vs the
  session's contact field 0.63 (crop level; the earlier "absent, r 0.06" was a broken whole-frame
  comparison — retracted). Fixed in image coordinates: warping the bare gel by the membrane
  displacement lowers its match to the field (0.60; > 8 px: 0.59 → 0.34) but raises its match to the
  same episode's contact frame (0.18 → 0.40) → imaging-chain state, not membrane, not fixture
  print-through (shared most by soft thick samples, 0.54; rigid 0.33 on the training gel).
- Second sensor (Real Tactile MNIST, GelSight Mini, 24 runs / 15 days / 2 gels, public): own run
  0.41, other run same day 0.39, other day 0.34, other gel 0.19 (170–410 µm).
- Temporal integration (34–100 µm reliability): 1 frame 0.14 → 4 frames 0.32 → 8 frames ~0.48.
- Drift of the unloaded gel (620 blanks, 44 sessions): use 0.13 per 5 contacts, power-up step 0.40,
  no drift on the second gel; gel replacement: pattern correlation 0.67 and a new force scale.


## T6. Reviewer controls and the hold-out (2026-10-25; single frame; consistent protocol unless stated)
### Which estimate of the state (test-time; supervised / supervised + jitter; acc / F1 on D; consistent protocol, `state_ablation_cons.json`)
| estimate | sup | sup + jitter |
|---|---|---|
| none (raw at both ends) | 17.3 / 11.1 | 45.2 / 41.6 |
| feature-space mean of the previous session | 26.5 / 19.7 | 28.5 / 23.0 |
| mean colour of the previous session | 43.6 / 35.5 | 48.5 / 44.9 |
| low-pass field of the previous session | 52.5 / 46.5 | 51.4 / 46.8 |
| own session's other contacts (LOO) | 47.7 / 41.5 | 50.2 / 45.1 |
| previous session's contacts | 51.2 / 46.3 | **55.6 / 51.3** |
| a random other test-campaign session | 49.0 / 42.8 | 52.2 / 47.0 |
| the session's bare gel (+ campaign offset) | **56.1 / 50.4** | 53.8 / 48.9 |
| mean of all 20 test-campaign sessions | 54.9 / 48.5 | 52.9 / 47.6 |
Any unlabelled frames of the deployment work (49–56 / 52–56); own session worst of the contact-based;
feature-space centring fails; low-pass ≈ full for the plain CNN, full +4 for the jitter CNN.
(Raw-head protocol file `state_ablation2.json` had campaign 57.1 / blank 49.9 / meancol 32.7 — superseded.)
### Within-campaign hold-out, A+C → B (15 materials, chance 6.7; raw / +prev / AdaBN whole / AdaBN per session)
| model | raw | + prev | AdaBN | AdaBN/session |
|---|---|---|---|---|
| supervised | 54.4 / 55.7 | 59.2 / 61.5 | 47.5 | 2.6 |
| + jitter | 57.9 / 57.7 | 61.3 / 61.5 | 42.2 | 3.4 |
| 8-frame inputs | 39.0 / 37.6 | 37.9 / 41.8 | 40.4 | 2.1 |
| label-free swap, from scratch | 27.0 / 31.5 | 20.9 / 26.0 | 1.6 | 0.0 |
Across the break (ABC → D) the same models: 17.3 → 51.2 (57.1 campaign mean); jitter 45.2 → 55.6;
AdaBN 49.1 / 40.4; per session 6.2 / 6.5. Time is not the cause; the state change is. Calibration is
safe (+5 within, +34 across); AdaBN is not (−7 to −16 within).
### Training-set structure (single frame; raw heads / consistent prev; 2026-10-25 07:00)
| train → D | raw | own-cal | prev | AdaBN | per-sess | collapse |
|---|---|---|---|---|---|---|
| A+C (one session per material, C present) | **13.2** | 34.5 | 37.7 | 35.3 | 4.8 | 41 % (onto C) |
| A+B (two sessions per material, no C) | 36.8 | 50.2 | 42.7 | 32.0 | 6.3 | 26 % |
| A+B + jitter | 46.8 | 47.4 | 46.5 | 37.0 | 5.6 | 19 % |
| A only | 39.1 | 43.5 | 37.3 | 30.2 | 6.7 | 18 % |
| A+B+C | 17.3 | 44.6 | 50.5 | 49.1 | 6.2 | 43 % (onto C) |
Collapse target = the C materials (D's low-pass state is 1.6 grey levels from C's mean field, 3.5 / 5.4
from A's / B's). On the 13 shared non-replaced materials: A→B 71 raw / 86 prev; A+B→D 56 / 65 (own 73),
jitter 70 raw / 68; A+B+C→D 9.7 / 60 (blank 64). → state subtraction undoes the collapse; for a
balanced set jitter suffices and calibration adds 9–17 to a plain CNN.
### Leave-one-campaign-out rotation (2026-10-25 evening; single frame; raw / prev / online = paper protocol)
| test | train | model | raw | prev | online | own-cal (D blank-trained: bare gel) | AdaBN | AdaBN/sess |
|---|---|---|---|---|---|---|---|---|
| A | BCD | plain | **76.4** | 66.1 | 71.9 | 68.1 | 49.5 | 1.3 |
| A | BCD | jitter | 73.2 | 69.5 | 71.6 | 73.4 | 51.1 | 2.5 |
| A | BCD | blank-trained | — | 70.1 | 69.8 | 70.1 | 64.2 | 2.3 |
| B | ACD | plain | 68.3 | 67.6 | 63.9 | 71.3 | 72.7 | 2.4 |
| B | ACD | jitter | **75.4** | 72.3 | 68.3 | 71.0 | 67.3 | 1.9 |
| B | ACD | blank-trained | — | 62.4 | 62.8 | 62.4 | 66.7 | 2.4 |
| C | ABD | plain | 19.5 | 22.3 | 13.3 | 6.3 | 1.1 | 2.4 |
| C | ABD | jitter | **30.6** | 32.4 | 26.8 | 25.8 | 7.6 | 2.1 |
| C | ABD | blank-trained | — | 9.6 | 11.1 | 9.6 | 1.4 | 1.2 |
| D | ABC | plain | 17.3 | 51.2 | 47.5 | 44.6 (bare gel 56.1) | 49.1 | 6.2 |
| D | ABC | jitter | 45.2 | **55.6** | 51.2 | 51.0 (bare gel 53.8) | 40.4 | 6.5 |
| D | ABC | blank-trained | — | 47.5 | 45.5 | 52.9 (bare gel) | 48.0 | 7.1 |
State subtraction pays only in fold D (unseen state, same gel); it is neutral-to-harmful when the test
state's campaign is absent but both other states are in training (A, B), and useless across the gel
replacement (C). Blank-trained inputs are worse than raw training + test-time subtraction everywhere.
Jitter is the most robust single choice. Frozen Sparsh-I-JEPA raw / prev per fold: A 42.8 / 37.2; B 36.4 / 36.7; C 4.5 / 2.2; D 11.2 / 35.5.
### The state tracker (Algorithm 1; 2026-10-27)
Detection over 2,701 episodes / 55 sessions, thresholds from the first hardware state: 10/210 settings catch
both hardware changes with zero false alarms; operating point τ_f 0.10, τ_i 3.0, m 3 → unit change after 17
episodes, gel change after 2; S1-percentile setting (τ_f 0.2, τ_i p95 1.89, m 8) → 22 / 7. Canvas (weak
pattern) and the Sep 28 power-cycle illumination jump do not alarm. Streamed reading of D (plain CNN): tracker
53.8 (cross-unit materials 60.5 incl. new pieces; 62.1 excl.) vs no-reset running mean 42.3 vs oracle previous
session 51.2. Tactile MNIST (153,600 touches, 24 runs): gel swap detected in 4 touches, no false alarm.
### Marker-residue control (signature ≥ 60 px from markers): own 0.48, same day 0.35, same hw weeks 0.34,
same hw months 0.29, other unit 0.04, other gel 0.08 (base: 0.53 / 0.40 / 0.39 / 0.32 / 0.15 / 0.16).
### Photometric go/no-go (fold D, plain / jitter): subtraction 51.7–56.3 / 53.0–55.8; ratio 51.8–56.1 / 51.0–55.4;
per-pixel location+scale 39–48 / 31–47; 3×3 colour-response alignment 47–48 / 46 → post-subtraction gap not
photometric; inverse-rendering method dropped.
### SITR smoke (train Mini-0, 20/class): hardware probe 93–100 % on every frozen feature; DINOv2 Mini→Mini raw
67.8 / SITR bg 66.7 / our stream 74.3. Full runs in progress.
### Other controls
- DANN (session-adversarial) 52.9 prev vs plain 51.2 → no gain.
- Noise control: 8-frame means + single-frame-level noise at train and test: 34.0 prev (8 frames
  35.8; 1 frame 51.2) → the single-frame advantage is not a noise-regularisation effect.
- Adapter positives: state swap 34.9 vs photometric jitter 23.4 (zero-shot) → the swap is the gain.
- Few-shot (target-campaign labels in the head; sup, 1 frame): 1 % raw 36.8 → calibrated 55.5;
  5 % 58.8 → 68.9; 10 % 70.1 → 76.5. Calibration stacks with labels.
- 18-way (replaced pieces Textured_Rubber / EPDM_Foam excluded): raw jitter CNN 46.1.
