# CVPR 2027 — results sheet (native protocol; status 2026-10-23 03:30)

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
Single-frame protocol (A+B+C → D; raw / previous-session calibration):
| model (1 native frame) | raw | + prev.-session calibration |
|---|---|---|
| supervised (3 seeds) | 16.6 ± 0.8 / 10.5 | **51.0 ± 0.5 / 46.0** |
| supervised + state swap | 28.4 / 22.1 | 52.4 / 46.5 |
| supervised + photometric jitter (3 seeds) | **43.9 ± 2.3 / 40.3** | **53.9 ± 0.9 / 48.3** |
| supervised + jitter + state swap | 44.1 / 39.2 | 54.6 / 48.0 |
| label-free decoy swap (3 seeds) | 26.5 ± 1.6 / 23.6 | 31.0 / 27.8 (seed 0) |
| (jitter with 8-frame inputs, 2 seeds) | 22.8 / 17.7 | 40.5 / 35.5 |
Integration amplifies the stable fine-scale content = mostly the day signature. With single frames,
standard photometric jitter alone removes most of the collapse (45 raw; 24 with 8-frame inputs);
jitter + calibration is the best deployable pipeline with labels. The public-encoder and adapter
rows (T1) are single-frame already; the adapter (34.9) remains the best option without labels.
Resolution (single frame, supervised, raw / prev.-cal.): native 16.6 / 51.0 ± 0.5; 1/2 21.5 / 52.9;
1/4 23.1 / 49.6 — native resolution gives no measurable advantage for material across days.

## T2. Time gap, same 15 materials, trained on June (era A) only (15-way)
| model | → July (2–3 weeks) raw / calibrated | → Sep–Oct (3–4 months, same gel) raw / calibrated |
|---|---|---|
| supervised | 45.3 / **59.3** | 23.5 / **35.2** |
| label-free swap | 46.6 (F1 46.5) / 39.0 | 24.3 / 23.7 |
| frozen Sparsh-I-JEPA | 42.9 (F1 36.1) / — | 22.8 (F1 17.9) / — |
| **frozen Sparsh-I-JEPA + label-free swap adapter** | **49.2 (F1 46.5)** / 47.6 | **31.0 (F1 24.9)** / 26.4 |
Realistic calibration sources (supervised, A+B+C → D): previous session 35.8; when it was the same
day 40.4 (11 sessions) vs another day 30.1 (9); online (previous state updated per contact) 33.0,
helps from the first contact; own session's past contacts only: needs ~10–20 contacts.

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
- Day signature (55 sessions; 190–370 µm band; markers masked): a contact matches the position-wise
  mean of its own session's other contacts at r 0.53, of a different object the same day 0.41, of
  another era 0.15, of the other gel 0.16. Rigid samples 0.62 same day (no backing print-through);
  broadband (codec rows ≈ 3 % of power); absent from blanks (r 0.06).
- Second sensor (Real Tactile MNIST, GelSight Mini, 24 runs / 15 days / 2 gels, public): own run
  0.41, other run same day 0.39, other day 0.34, other gel 0.19 (170–410 µm).
- Temporal integration (34–100 µm reliability): 1 frame 0.14 → 4 frames 0.32 → 8 frames ~0.48.
- Drift of the unloaded gel (620 blanks, 44 sessions): use 0.13 per 5 contacts, power-up step 0.40,
  no drift on the second gel; gel replacement: pattern correlation 0.67 and a new force scale.
