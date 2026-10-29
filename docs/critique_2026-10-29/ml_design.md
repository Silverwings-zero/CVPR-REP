# ML-design critique: why "Stateful Touch" stops at ~56 %

Numbers come from `outputs/tacrep/*` or from CPU probes on the saved embeddings (`emb_diag.py/.out`, `permat.py`, `variance.py`, `absorb.py`, `corr.py` in this folder). The logistic probes reproduce the paper's MLP heads to within about 2 points (fold A 76.7 vs 76.4, B 68.9 vs 68.3, D raw 19.4 vs 17.3).

**Verdict: the ~56 % ceiling is mostly the learning setup, not the hardware shift.** The model learns sessions, not materials, from 1–2 sessions per class. It sees about 2 % of each episode. The state reading is an input patch added after training to an encoder that never saw it. The evaluation, 20 test sessions, cannot rank anything finer than about 5 points.

## 1. Limits independent of the shift (ranked)

1. **The model recognises sessions, not materials (VERIFIED).** On the test sessions' own episodes (head trained on half of them, encoder never saw them), the frozen features reach **99.3 / 99.6 / 96.9 %** for folds A, B and D raw. Across sessions on the *same* hardware they reach only 76 / 68 %. Each class has 1–2 training sessions: 15 materials have A(S1)+B(S1)+D(S2), and 5 have C+D on different hardware (data listing). No supervised objective can learn which within-class variation to ignore from two sessions. Outcomes are per session and bimodal: 40 % of fold-A sessions are above 90 %, while Resume paper (24 %) and Textured rubber (11 %) fail. Per-material recall correlates 0.85 between folds A and B, but only 0.21–0.27 between A/B and D-tracker.
2. **The input uses about 2 % of each episode (VERIFIED).** Each episode gives 12 crops: 4 frames × 3 random positions inside the central 68 % (`extract_patch_tubes.py:65-71`). That is out of a median of ~205 contact frames (40-episode sample). The crops are cached and fixed, with no re-sampling, flips or translations (`tacrep_swap.py:212-233`). The training set is therefore 16,320 fixed images. A single crop is weak evidence: 63 % per-crop accuracy even *within* a session. Episode accuracy is still rising at 12 crops (fold A: 1→42.7, 3→63.4, 6→71.7, 12→76.7). Different frames carry more evidence than different positions (3 crops from 3 frames 65.3 vs one frame 55.7). Textured rubber fails on every split, including the same hardware. A 2.2-mm field of view being too small for its pattern is my guess (SUSPECTED).
3. **The training recipe is untuned (VERIFIED, secondary).**
   - The 0.5 M-parameter CNN underfits: training CE is 1.03 at epoch 12 (`t_sup_f1/history.json`) and per-crop training accuracy 72–74 %.
   - The ResNet-18 memorises: CE 0.004 and 100 % training accuracy.
   - There is no pretraining for the CNN, no geometric augmentation, a fixed 12 epochs, and no session-held-out validation (`tacrep_swap.py:147,244-265`).
   - Capacity does not fix it. ResNet features are richer (98 % within D sessions), but cross-session accuracy does not improve (53.1 vs CNN 54.6). Point 1 binds.

## 2. Why the reading works, and why it saturates

- **Why it works (VERIFIED).** The shift is additive and position-locked, and crop positions are known. The 11-channel input keeps raw BGR (`texture_highres.py:58-63`), which carries the absolute illumination. Raw, the S2-trained FiberGlass and Cork absorb about 74 % of D episodes (derived from recall and F1). A many-object mean cancels the object imprints, and per-channel BN methods cannot represent a per-pixel field.
- **The estimate is not the bottleneck (VERIFIED).**
  - The oracle per-state mean of all test images gives 54.5.
  - Two objects give 53.6 and nineteen give 54.6.
  - Every variant lies within about ±2 points of the tracker, with bootstrap CIs over materials spanning 0 (decay +0.0, no-detector +0.1, prior-G +0.4, bare gel −0.6).
  - **Subtracting an additive field is saturated.**
- **Decomposition of the 44 points lost on D, plain CNN (VERIFIED arithmetic):**
  - 20.8 points: the same-hardware cross-session floor of the 15 materials (72.3 % in folds A/B).
  - 4.6 points: residual shift on those 15 (66.1 vs 72.3).
  - 18.6 points: the 5 changed-pad materials (25.5 %). These have exactly one session per hardware and are never seen on their test hardware.
- **The remaining errors depend on the particular pair of sessions, not on how hard the material is.** Canvas scores 100 / 98.5 on S1 but 0 cross-unit in *both* directions (D and reversed). Fleece goes from 74 / 67 to 3 (19 reversed). Resume paper and PE foam fail on S1 folds but pass on D. The canvas and fleece failures point to non-additive hardware effects (pattern VERIFIED, mechanism SUSPECTED). With one test session per class, shift and session noise cannot be separated.

## 3. Method design flaws

1. **REAL, high: the encoder never sees the reading (VERIFIED).**
   - Every checkpoint has `calib=False`. `tracker_folds.py:105-125,175-179` only refits heads on frozen raw-trained encoders, and `--calib-source` offers own/prev/auto but no many-object state option (`tacrep_swap.py:117`).
   - The training-side reference is therefore a patch on the head. Its sign depends on how each encoder handles inputs it never saw: +1.4 for the CNN, **−13.7 for ResNet** (39.4 vs 53.1).
   - ResNet *re-collapses* onto the S2-trained classes: top share 31–35 % vs 16 %, FiberGlass 99 %, Cork 70 %, and the 15 cross-unit materials drop from 63.9 to 36.8.
   - Cause (SUSPECTED): the four S2 training sessions are referenced against a mean of 3 other objects. That reference carries class-specific imprints, so it ties class to input again.
2. **REAL, medium: the train and test readings differ, and the test estimate includes the object itself (VERIFIED).**
   - The training reference excludes the crop's own session (`tracker_folds.py:100-102`). The test estimate includes the current object (`state_tracker.py:101-113`).
   - Excluding it gives +1.1 (CI −2.1, +4.6) with the state head, +4.1 with the prev head (57.8 vs 53.7) and +1.8 for jitter. Cork goes from 50 to 76.
   - This is the same mechanism as per-session AdaBN at **6.2 %** (`t_sup_f1/results.json`). Here session = class, so any per-session statistic is a class statistic.
3. **REAL, medium: state augmentation swaps session fields, not hardware fields (VERIFIED).** It re-renders with the crop's own-session field (`tacrep_swap.py:9-12,216-220`), so it also swaps the object's mean imprint, which is noise correlated with the label. It costs points on known hardware (fold A 76.2 → 74.2, jitter+state-aug 68.7) and adds nothing on top of the reading.
4. **HARMLESS but not a contribution (VERIFIED).** The detector (no-detector decay 56.0 = tracker 56.0), object weighting (uniform −3.2, CI [−7.1, 0.0]; jitter +0.3) and the restart prior do not affect accuracy.
5. **Frozen MLP heads: harmless on raw inputs (VERIFIED by probe parity).** They matter only through 3.1. The "few-target-labels" numbers leak, however: the labelled episodes come from the very test sessions (`tacrep_eval.py:127-138`). They measure session matching, which is 97–99 %, not adaptation.

## 4. Can the evaluation rank methods?

- **There are 20 independent units, not 1,341 (VERIFIED).** 23–25 % of material×seed cells are below 10 % and 23–25 % above 90 %. Within-material seed SD is 3–4 points and encoder-seed SD 0.4–2.3. Seeds are not the variance that matters. One flipped session moves the score by 5 points.
- **Counted by session, the tracker recognises 12 of 20 sessions (exact 95 % CI 36–81 %).** Raw recognises 3 of 20.
- **The smallest detectable difference** (paired over materials, 80 % power) is about 4–5 points between similar variants and about 13 points against AdaBN or the previous session.
- **What the evaluation can rank:** raw vs reading. Tracker vs AdaBN on D is borderline (+13.3), and it was not significant on the reversed split.
- **What it cannot rank:** the tracker variants, plain vs jitter, or tracker vs bare gel or oracle.
- **The development split is also the test.** About 24 readings × 2 heads were compared on D, an upward bias of a couple of points (SUSPECTED).
- **Recommended protocol:**
  - Primary metric: session-level accuracy, or episode macro with a hierarchical session bootstrap, pooled over **all 40 cross-hardware session tests** (D 20 + reversed 15 + C 5).
  - Stratify by relation: same hardware, other gel, other unit, single-session class.
  - Secondary: NLL, macro-F1, top share.
  - Use at least 5 encoder seeds, declare the minimal effect in advance (one session, 5 points), use D only for development, and freeze the protocol before tuning.

## 5. Requirements the evidence imposes on a stronger method

1. **Learn within-class session invariance from more than 1–2 labelled sessions**, since the largest loss bucket is cross-session on the same hardware. That needs session-held-out validation, nuisance augmentation (lubrication, pressure, illumination, placement), self-supervision on the ~550 k unlabelled contact frames and public multi-session sets, and cross-session positives where a material repeats.
2. **Use the whole episode:** many frames and positions with learned aggregation. Pooling is unsaturated at 12 crops, and frames beat positions.
3. **Give each view more native context at 11 µm/px.** A 2.2 mm crop is weak even within a session, and the A40s make larger native views affordable without downscaling.
4. **Put state handling inside training, built identically at train and test.** The estimate must never contain the anchor's own object, never use few-object, class-correlated references, and augmentation must use hardware-level fields rather than session-level ones.
5. **Go beyond an additive correction:** canvas and fleece fail across units in both directions, and SITR's units differ in field of view and contrast. This needs invariance to gain and photometric response.
6. **Handle classes recorded in a single session explicitly.** They cost about 19 macro points.
7. **Beat per-pixel mean subtraction (oracle 54.5) by more than one session (≥ 5 points)** on the pooled 40-session protocol.
