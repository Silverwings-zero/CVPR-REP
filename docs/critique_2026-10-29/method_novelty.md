# Method novelty: frank advice (paper_r3, 2026-10-29)

**Verdict: reframe the paper as analysis plus benchmark. Keep the reading as a strong closed-form baseline with its assumptions stated. At most, run one method bet with a fixed time box and a kill rule written in advance.**

Literature tags: [checked] means I read it this session through alphaXiv. [unchecked] means it comes from memory or the paper's bibliography.

## 1. Why the running per-pixel mean is weak as a method

- **It is an estimator, not an algorithm.** The reading x − F̂ + G is per-domain, per-pixel input centring. That is the retrospective flat-field (BaSiC and CIDRE, both cited) and the same idea as cepstral mean normalisation [unchecked]. Every part specific to Algorithm 1 changes the result by less than the noise:

  | variant | accuracy |
  |---|---|
  | Algorithm 1 | 56.0 |
  | decay, no detector | 56.0 |
  | transductive mean per state | 54.5 |
  | bare-gel frame | 55.4 |
  | uniform weighting | 52.7 |

  So the contribution is the inductive bias ("additive, per-pixel, shared across objects"), not the procedure.
- **It is at its ceiling.** The oracle state mean, the bare gel and the tracker agree. The missing ~44 points are not state: the June-only cross-unit loss of about 30 points is untouched, and 10% target labels give 76.5. No better additive estimator can show more.
- **The gain only appears because of one confound.** It needs the split's flaw, where four classes exist only on S2. Elsewhere the reading costs: −1 to −2 points on folds A and B, and −6 on fold C.

| Assumption | Where it fails | Where it shows |
|---|---|---|
| Additive | SITR contrast ×3.7 and colour; Tactile MNIST; force depends on gel stiffness (mechanics). | From-scratch CNN on SITR at 11.9→13.1 (18.7 with location-scale). Unit probe 100% after the reading. Force R²≤0.11. Our data is only first-order: ratio = subtraction. |
| Pixel-aligned | SITR Mini 4 sees a different field of view; HCT is hand-held. Ours holds only because of the fixture and the fixed crop positions. | SITR probe does not move. HCT stream 23.2 < raw 28.3. The crop-based detector missed the unit change. |
| Per-position | Variance falls only with the number of objects per pixel. Over-parameterised: for the CNN the smooth field alone matches the full field (51.6 vs 50.5). | SITR mean of 20 images 63.9 < raw 73.9; one object 48.7. |
| Stationary per epoch | Session illumination sits 0.5–1.3 grey levels from the hardware mean; HCT days differ. | Previous session beats the tracker for the jitter CNN (56.6 vs 54.6). False alarms at day boundaries. |
| Objects average out | Cold start; the S2 training estimate comes from 3 objects; HCT days are dominated by one material. | Fold C 21.9→15.9. Rigid four 53.2→26.2. Sparsh on the rigid four 0.1–4.2. |
| (hidden) object = class, one presentation each | Subtracting earlier imprints acts as an implicit "not a previous class" prior. | Uniform weighting costs 3.3 points. No session-order permutation test, so the effect is unmeasured. |
| (hidden) state independent of load | The loaded contact mean ≠ the bare gel. | The reading breaks on unloaded frames (contact detection). |

## 2. Why the learned attempts failed

1. **Same generator, lower ceiling.**
   - Swap positives and state augmentation use x − F_s + F_s′, the same additive model the reading inverts. A learned invariance can at best equal the reading. It falls short because it learns from the span of training states, while the reading computes the target's own F.
   - Evidence: augmentation adds nothing on top of the reading (55.3 vs 56.0).
   - Evidence: the adapter's swap positives beat jitter positives raw (34.9 vs 23.4) but not after the reading (36.4 vs 37.8). Its whole advantage approximates the subtraction.
   - It wins on SITR (75.0 vs 70.6) only because the tied classes put the test Mini's state into training.
2. **Wrong nuisance, trivially satisfiable.**
   - F_s is a session mean: hardware, plus the day's illumination, plus that material's mean imprint (session = material).
   - The pool holds ~31 S1 sessions, 4 S2 and no S3. About 88% of swaps stay within S1, which is not the cause of the failure. Every swap also declares the material's mean appearance a nuisance.
   - The two views share the residual r pixel for pixel. InfoNCE is solved by any high-pass instance code, and the input already has high-pass channels. No cross-contact positive forces materials to group (InfoMin argument [unchecked]).
   - The label-free swap's 26.5 beats supervised raw (16.6) mainly because, without labels, there is no class–state shortcut to learn.
3. **Session = class.**
   - DANN's session adversary is a class adversary. It can remove only A-vs-B day variation within S1, never S1-vs-S2, because the S2 classes have only S2 sessions.
   - The adapter's few-session batches make most negatives and decoys same-material crops (false negatives).
   - Grouping by hardware is worse. In training, hardware is a function of class (four rigid classes ⇔ S2), so hardware invariance deletes class information. This is not identifiable from (x, y, d) alone.
   - GroupDRO and DFR (asked for by R2 and R3) need (class, hardware) cells that are empty on both the main and the reversed split. Say so explicitly, and run them only where such cells exist.
4. **Wrong space, too little support.**
   - The adapter works on pooled frozen features, where the pixel offset is already entangled with content (feature-space subtraction 29.2 vs 50.5 in pixels).
   - Training has ~16K crops and only two hardware states. A learned invariance covers only the nuisance it has seen.
   - For scale: SITR's own ablation [checked] goes from 61% to 82% cross-sensor transfer as simulated configurations rise from 10 to 100.

## 3. Candidate directions

A shared constraint: on our data, state correction is saturated (54.5–56.0), so any new method can at best tie there. Its win must come from cold start, non-additive or public cases.

**D1. In-context state encoder trained on randomised sensor states**
- **Design:**
  - FiLM or spatial modulation from an object-balanced context: the mean field, second moments and coverage of recent unlabelled contacts.
  - Episodic training over thousands of synthetic states. Each applies random additive fields, per-pixel gains, 3×3 colour mixing, gamma, field-of-view warps and blur to real residuals from our data, SITR, Tactile MNIST and FEATS.
  - Training includes single-class contexts.
- **Must beat:**
  - the reading;
  - **the same randomisation as plain augmentation, with no context** (the decisive ablation);
  - ContextViT-style mean-token conditioning;
  - and must approach SITR's calibrated 90% without calibration presses.
- **Evidence it can use:**
  - SITR leave-one-Mini-out and the injected tie with a from-scratch CNN (11.9);
  - Tactile MNIST (0.0);
  - the unit probe;
  - our cold-start curve and fold C.
- **Closest prior work:**
  - SITR [checked]: 18 calibration presses as tokens, 1M simulated images, 100 configurations.
  - ContextViT [checked]: a context token amortised from images of the same group.
  - ARM and CS-ARM-BN [checked, ICML 2026]: negative controls stabilise in-context BatchNorm under label shift. This is the microscopy twin of the multi-object finding.
- **Novelty margin:** thin. No calibration objects, a context that survives one class per context, and the state as a spatial field.
- **P(changes reviewers' minds by Nov 16) ≈ 15%.** The main risk is that augmentation alone matches it.

**D2. Invariance from naturally paired "bridge" materials**
- **Design:** the 14–15 materials recorded on both units serve as positives across states (conditional variance penalty, or SupCon across states); the others stay tied.
- **Evaluation:** R2's randomised injection on our data, with varied counts of tied and bridging materials, both directions, many draws and session-order permutations; repeat on SITR and Tactile MNIST.
- **Must beat:** ERM on identical data, the reading, the reading plus state augmentation, and GroupDRO/DFR, which become applicable here.
- **Closest prior work:** CoRe (Heinze-Deml & Meinshausen) [unchecked], SITR's SupCon [checked], ZDDA [checked, existence only].
- **P ≈ 10% as a method**, because novelty is low. Run the protocol regardless: it supplies replicated tie events and power.

**D3. Geometric and photometric state model fitted from the stream**
- **Design:** register the field of view from the gel border in the stream mean or from the marker lattice, then apply per-pixel gain and a colour matrix.
- **Must achieve:** fix the SITR and Tactile MNIST CNN ties and bring the probe below 100%, without losing on our data.
- **Closest prior work:** classical calibration. SITR affine-corrects the Wedge [checked], and our own location-scale and colour readings were negative.
- **P ≈ 5–10% as a contribution.** Use it as a 1–2 day diagnostic that turns "fails on SITR" into "explained by geometry".

The inverse-rendering plan in `docs/METHOD_DESIGN.md` faces the GBR ambiguity and would learn R_h from only 2–3 real hardware states. P < 5%.

## 4. Recommendation

**Reframe as analysis and benchmark.** Five reasons:

1. The additive correction is at its ceiling on the only split where it helps.
2. Learned invariance is non-identifiable under complete class–hardware confounding. Additivity is the identifying assumption, and the reading already uses it optimally.
3. The only learnable lever is synthetic state diversity. It is built on the same image model, so its possible wins sit on small public sets where SITR already reaches 90% with calibration.
4. "Infer the context from an unlabelled set, then condition on it" is already taken by ContextViT, ARM, CS-ARM-BN and SITR.
5. There are 18 days left, writing included.

The evidence, and all three reviewers, do not support a "general state-aware representation" headline.

**Make the analysis version CVPR-grade with:**
- replicated tie events with power (the D2 protocol), with a primary comparison against a plain running mean, AdaBN and positional normalisation, fixed in advance;
- a predictive diagnostic: the additive share of variance between states should predict the reading's gain on our data, SITR, Tactile MNIST, FEATS and HCT, so that the failures become confirmed predictions;
- an identifiability paragraph on why DG baselines cannot apply here;
- one or two more naturally tied public audits: ObjectFolder-Real, Touch-and-Go or TacQuad.

**Optional bet:** D1, time-boxed to ≤6 days. Kill rule, written in advance: it must beat randomisation-only by ≥5 points on both the SITR tie and Tactile MNIST, and tie the reading on our data. Otherwise drop it.
