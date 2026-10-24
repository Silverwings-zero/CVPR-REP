# CVPR 2027 — one-page story (working draft, 2026-10-22)

Numbers: native-tube protocol, train June–July (eras A–C) → test Sep–Oct (era D), 20 materials,
macro accuracy / macro-F1 (chance 5 %). Details and every run: `docs/TACREP_LOOP.md`.

## Claim (v2, author's framing 2026-10-24)
A **sensor-state-aware tactile representation** for robot learning: every observation = the sensor's
current state + what is touched; the loaded state (recent contacts) and the unloaded state (bare gel
between grasps) are estimated from the robot's own stream; encoder-agnostic (works on frozen
foundation models), serves material / force / contact, learnable without labels, holds across
months, power cycles, gel swaps and a second unit. The day signature is the diagnosis solved along
the way, not the headline. Title: "Tactile Representations That Last: Sensor-State-Aware Touch
Representations for Robot Learning Across Months of Use".

## Claim (v1, superseded)
A vision-based tactile image is the touched surface **plus a day-specific signature of the
sensor itself**. Learned tactile perception latches onto that signature and fails on a later
day. The signature can be measured and removed using only the robot's ordinary contacts — no
labels, no blank frames, no calibration presses, no retraining.

## Evidence chain
1. **The signature exists (model-free).** At a fixed image position, a contact's 190–370 µm
   band matches the average of *other objects touched the same day* (r 0.41) but not another
   era's (0.15); markers masked, all 55 sessions. It is broadband (gel surface under load —
   coating, lubricant film, residue); not compression (≈3 % of its power), not a print-through
   of the sample's backing (rigid samples share it most: 0.62). **It replicates on another lab's
   GelSight Mini** (Real Tactile MNIST, 24 runs / 15 days / 2 gels, public): a touch matches the
   average of other runs at 0.39 same day, 0.34 other day, 0.19 after the gel swap.
2. **Blank frames cannot see it** (blank vs same-session contacts r 0.06) — why flat-fielding by
   a no-contact frame fails, and why the usual "per-session reference" recipe falls short.
3. **It is the shortcut — for every encoder.** A supervised encoder collapses on the later era
   (46 % of episodes to one class; 15.7 % / F1 9.0); so do frozen tactile foundation models
   (Sparsh DINO / MAE / I-JEPA 8.9–11.2 %, 47–52 % to one class) and DINOv2 (22.0 %, 37 %).
4. **Remove it from the contact stream — encoder-agnostic.** The sensor state = the position-wise
   mean of recent contacts. Subtracting the PREVIOUS session's state (a different object, no
   information about the current material) lifts frozen Sparsh-I-JEPA 11.2 → 35.5, the supervised
   CNN 16.6 → 51.0 ± 0.5 (single native frames), and 40.4 when that session was the same day;
   online (start from the last state, update every contact) helps from the first contact.
   Standard photometric jitter is a strong labelled baseline at single frames (45.2 raw; 54.3 with
   calibration = best deployable pipeline with labels); it does not work on integrated frames (23.6).
5. **Adaptation the usual way does not deploy.** AdaBN over a whole era (20 objects) reaches
   40.2, but per session — one object at a time, what a robot has — it collapses to 4–6 %.
6. **Learn invariance without labels (no test-time data needed).** Re-render each training
   contact under other sessions' states (state swap, with position-matched decoy negatives).
   From scratch: 27.6 ± 2.2 / 26.3 ± 1.4 (3 seeds) vs supervised 15.7 / 9.0. As a small adapter on
   a FROZEN tactile foundation model: Sparsh-I-JEPA 11.2 → **34.9 ± 0.6** (3 seeds), Sparsh-MAE
   10.0 → 32.6, Sparsh-DINO 8.9 → 28.5 zero-shot; decoy negatives add +3.1 [+0.5, +6.1] — the
   objective upgrades existing foundation models. It is the best option WITHOUT labels; a labelled
   CNN with jitter does better (45 raw), so it is not claimed as the best zero-shot number.
7. **Integrating frames feeds the shortcut.** Averaging 8 native frames raises the reliability of
   fine texture (0.14 → 0.48 at 34–100 µm) — but that stable content is mostly the sensor's day
   signature, so cross-day accuracy after calibration FALLS with integration (1 frame 50.8 →
   8 frames 35.8, supervised + previous-session calibration). The material protocol therefore
   uses single native frames; integration is reported as an ablation.

## Scope (author, 2026-10-22): one representation for three objectives
material (what am I holding), normal force (how hard), contact (did contact start). Crops carry
no force, so the shared representation reads the whole native frame (`scripts/tacrep_frame.py`;
cache `data/processed/frame_cache`: 4 frames per episode at force quantiles + era-D lead-in /
pre-contact / onset frames + 620 blanks). Label-free training: state swap of whole-frame fields,
with the same episode's other force levels as hard negatives (keeps force without force labels).
Frozen heads: material (A–C → D), force (Nano17, A–C → D, by gel), contact (trained on the 7
lead-in sessions, tested on the other era-D sessions incl. the new gel; baseline: differencing
against the session's blank). First check: contact separates almost perfectly from frozen
features; contact-state calibration must not be applied to unloaded frames.

## The three objectives, measured (2026-10-23)
- **Material** (appearance): needs the LOADED state removed (stream calibration / state swap).
- **Normal force** (geometry): marker spreading against the bare-gel marker grid (order-free affine
  summary) predicts Nano17 force across days (R² 0.43–0.48, r 0.66–0.71) and across months with a
  single borrowed unloaded grid (June–July → Sep–Oct R² 0.48, r 0.72); appearance-based force does
  not (learned ≤ 0.1, model-free 0.22). A new gel keeps the force ranking (r ≈ 0.70) but changes
  the force-to-displacement scale (stiffness): absolute force needs a one-off per-gel scale.
- **Contact**: a head on frozen features rejects 0 % of other days' bare gel; referencing the
  session's own bare gel (embedding difference) restores 70–86 % specificity at ~100 % recall
  (blank differencing 88.6 %).
- Spectrum: appearance-dependent cues suffer the day signature; geometric cues survive it.

## Method (one figure)
contact stream → state field F(p) (position-wise mean of recent contacts, image coordinates)
→ (a) stream calibration x − F(p) + G(p) at test time; (b) state-swap training x' = x − F_s + F_s'
with InfoNCE (decoy negatives: other surfaces under the same state and position).

## Pending (running tonight)
native normal-map alignment; TactSpace replica on the tubes; stronger swap variants; clean time gaps (A → B vs
A → D on the same 15 materials; B + C → D as the gel-change control); second seeds;
re-render check (an era classifier should flip on re-rendered contacts).

## Second unit (public Real Tactile MNIST, digits from single touches)
Measurement replicates (day signature: same day 0.39 vs other day 0.34 vs other gel 0.19). Method:
stream calibration transfers — every encoder collapses raw (37–84 % to one class) and recovers with
the previous run's state (supervised F1 11.8 → 18.0; frozen Sparsh chance → 17.5–19.8), across
later days and the gel swap. Per-run AdaBN works there (runs hold all digits), confirming that our
per-session collapse is the one-object-per-session effect. The label-free swap / adapter does not
transfer to coarse digit shape: it is claimed as a texture result on our unit only.

## Risks / threats to state
one sensor unit in our own data (mitigated: the day signature replicates model-free on Real Tactile
MNIST, another GelSight Mini; GelFabric is a further option); session = material (handled:
previous-session calibration uses other objects); lubricant re-application confounds "use";
absolute accuracy is modest (20-way, zero-shot) — report macro-F1, collapse, 5-candidate.

## Story decisions (resolved 2026-10-22 / 24)
- Representation framing (author, Oct 24): a state-aware composite stated as a FINDING — appearance
  cues carry the day signature (material: state-removed embedding), geometric cues survive it
  (force: marker spreading vs the unloaded grid), contact: reference to the session's bare gel.
  Not claimed as one learned network.
- Scope: all three objectives (material, force, contact) — author.
- Public data allowed for replication (not new collection) — done: Real Tactile MNIST.

## Abstract draft v0 (2026-10-23; numbers from docs/CVPR_RESULTS.md)
Vision-based tactile sensors are moving into robot hands that touch objects for weeks, across
power cycles and gel replacements, yet learned tactile perception is trained and tested within
short recording periods. We show that this hides a systematic failure: a tactile image carries a
*day signature* of the sensor itself. Across 2,711 recorded grasps over four months (20 materials,
co-recorded normal force, bare-gel frames), a contact image shares a fixed-position pattern with
every other object touched the same day but not with other days; the pattern is absent from
no-contact frames, so standard background calibration cannot remove it, and it replicates on a
second GelSight Mini in public data. Supervised encoders and tactile foundation models exploit it
and collapse on a later day (9–16 % at chance 5 %, up to half of all predictions in one class).
We separate sensor state from touch using only the robot's ordinary stream. The loaded state —
the position-wise mean of recent contacts — can be removed at test time (frozen Sparsh improves
3×, best when the state comes from another object touched the same day), or used at training
time: re-rendering contacts under other sessions' states trains, without labels, a small adapter
that makes frozen tactile foundation models 3× more accurate on later days with no test-time data
at all (Sparsh-I-JEPA 11 → 35 %).
The unloaded state — the bare gel between grasps — references geometry and contact: normal force
from marker spreading transfers across months (r 0.72), and contact detection recovers from 0 % to
86 % specificity on other days' bare gel.
