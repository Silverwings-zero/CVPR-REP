# Exploration plan after three review rounds (2026-10-29)

Inputs: three simulated review rounds (3/3/3 each), and three expert critiques in `docs/critique_2026-10-29/`
(`ml_design.md`, `data_audit.md`, `method_novelty.md`). Compute: USC CARC Discovery, A40 48 GB, job arrays
(`HapTeleop/cluster/`). Constraints that stay: no new data collection, native resolution (one labelled Sparsh
exception), paper work in this repo.

## What the critiques established

1. **The ceiling is the learner, not the sensor state.** Of the 44 points lost on campaign D, about 21 are the
   ordinary drop to a new session on the same hardware, about 19 come from the five materials with a single session
   per hardware, and only about 5 are residual hardware shift. A head trained on half of a test session's episodes
   scores 97-99 % on the other half: the model recognises sessions, not materials.
2. **We use about 2 % of each episode** (12 fixed crops from 4 of ~205 contact frames); accuracy is still rising at
   12 crops, and more frames help more than more positions.
3. **The recipe is untuned:** our CNN underfits, a fine-tuned ResNet-18 memorises; no geometric augmentation; no
   session-held-out validation; no encoder was ever trained on the reading.
4. **Better state estimates cannot help:** two objects already reach the transductive per-state mean (54.5).
5. **Six classes carry most of the remaining gap.** Textured rubber and EPDM are different pieces in D, wood was
   recorded at another grasp height, and canvas, fleece and ceramic fail under every reading in both directions.
   The other 11 classes reach 82-83 % on D after the reading, above their same-unit level.
6. **Hardware is confounded with protocol:** for the 15 unit-1 materials every unit-2 session is also a D session,
   and lubrication, operator, counter-surface, speed set and three months of ageing change at the same step.
7. **The evaluation cannot rank methods:** effectively 20 test sessions, so differences below about 5 points (one
   session) are invisible.
8. **Why earlier learned methods lost:** their pairs were built with the same additive model the reading inverts,
   session means contain the material's own imprint, and hardware is a function of class in training, so invariance
   to hardware cannot be learned from labels alone.

## Phase 0: an evaluation that can rank methods (first, ~2 days)

- **Pooled session-level protocol:** every cross-hardware session test (main split, reversed split, and the
  leave-one-campaign-out folds), about 40 sessions, stratified by hardware relation, with session-bootstrap
  intervals. Model selection only on session-held-out validation inside the training campaigns, never on D.
- **Cleaned label set:** 18-way main result (textured rubber dropped, EPDM reported as specimen generalisation),
  strata for the 11 clean classes, canvas/fleece, the gel change, wood. 20-way kept as a secondary row.
- **Replicated tie events:** inject the class-hardware tie into our own 15 bridge materials (recorded on both units)
  with many random draws of tied classes, both directions, many session orders. Same on SITR (12 Mini pairs),
  Tactile MNIST and FEATS.
- **Matched controls:** A-only training tested on B versus D on the 11 clean classes; D episodes restricted to the
  training speed set and force bins.

## Phase 1: fix the learner (expand broadly, ~5 days)

Every family is judged on the pooled protocol against the same best baseline (mean subtraction), three or more
seeds, with a success bar of +5 points and an interval that excludes zero.

- **E1 Whole-episode models.** Many contact frames and positions per episode with learned aggregation (attention /
  multiple-instance pooling); larger native views (384-768 px crops at 11 um/px).
- **E2 A real recipe.** Geometric and photometric augmentation, longer schedules, regularisation, session-held-out
  early stopping; pretrained backbones (DINOv2, Sparsh, ImageNet ViT/ConvNeXt) fine-tuned at native resolution
  (full or LoRA).
- **E3 Session invariance.** Cross-session supervised contrastive learning, leave-one-session-out meta-learning,
  GroupDRO over sessions, cross-session mixup; self-supervision on the ~550 k unlabelled contact frames and on public
  multi-session sets.

## Phase 2: state handling inside the model (in parallel, ~6 days)

- **E4 State-conditioned encoder.** A context set of recent unlabelled contacts (never the anchor's own object)
  becomes a state token that conditions the encoder (FiLM or cross-attention). Trained with randomised states that
  include gain, colour response and field of view, sampled from the variation measured across our units, SITR and
  Tactile MNIST. Control: the same randomisation without the context.
- **E5 Learned non-additive normalisation.** Registration, gain and offset per hardware estimated from unlabelled
  frames; first as a diagnostic on SITR and Tactile MNIST (does the from-scratch CNN recover, does the unit probe fall).
- **E6 Bridge-material invariance.** The 15 materials recorded on both units, SITR's objects on four Minis and Tactile
  MNIST's digits on two gels as natural pairs.

## Phase 3: generality

- Pooled multi-dataset training with leave-hardware-out tests on SITR, Tactile MNIST, FEATS and HCT.
- One or two further public audits with sessions, days or sensors held out.

## Decision rules

- A family that does not reach +5 points on the pooled protocol after its budget stops and is reported as a negative.
- By about Nov 8 (abstract Nov 10) we choose the story: a method paper if a family clears the bar on the pooled
  protocol and on at least one public set, otherwise the analysis-and-benchmark paper, with the best learner as its
  strong baseline.

## Questions for the data owner

1. Were D's canvas, fleece, silk and ceramic the same pieces, the same side and the same orientation as in June/July?
2. Was June's "paper" the same sheet as resume paper?
3. Was lubricant applied in June when no pause was logged?

## Progress (2026-10-30)

The pooled protocol is in place: 55 cross-hardware session tests (campaign D, campaign C, and the reversed split),
session bootstrap intervals, and paired comparisons on matched sessions (`HapTeleop/scripts/xp/`). All numbers below
are seed 0 unless stated, and none is final until three seeds are in.

- **Recipe.** Geometric augmentation lifts the from-scratch CNN from 27.0 to 33.3 pooled. Reading test images
  relative to a low-pass estimate of the new state's illumination (`test_lp`) adds about ten points (43.9 pooled,
  54.7 on D). Reading training images as well hurts, because the training estimate of unit 2 comes from only three
  objects. Larger central views (384 px) collapse, because they show more of the hardware pattern.
- **Alignment with the new hardware's unlabelled stream (E3/E5).** A domain adversary (DANN) on top of the recipe,
  read with `test_lp`, gains 9.2 points over the recipe on matched sessions (interval +3.2 to +15.0). CORAL gains
  7.3, and the same 7.3 when it sees only the first half of each new state's sessions. The gain sits on the reversed
  split (38.8 to 56.0); on D the illumination reading alone already removes most of the shift. Alignment ends the
  collapse: the share of test images in the most predicted class falls from 28 to about 10 percent.
- **Running.** Seeds 1 and 2 on all three folds; a hardware-state adversary over every state, an
  information-maximisation term against the collapse, and a class-conditional session adversary; and the SITR tie
  (from-scratch CNN, all 12 ordered Mini pairs) with alignment against the new Mini's unlabelled images, the case
  the additive readings could not repair.

## Progress (2026-10-31) and the story decision

**Decision (data owner, 2026-10-31): method paper.** The method is ours as a whole: the framing (a hardware change
is a shortcut problem, and the robot's own unlabelled touches after the change are the signal to adapt with), the
test-time low-pass reading, and the grouped confidence objective. Its alignment and confidence losses are adapted
from unsupervised domain adaptation (DANN; information maximisation as in SHOT) and are cited as such, with an
ablation of every part. HCT is a different sensor (DIGIT) whose day shift is mostly a change of objects and of the
material mix, not of the sensor's look, so it is out of scope; one limitation sentence states that.

Ablation on the pooled protocol (55 cross-hardware sessions, three seeds unless stated, numbers are cross-hardware
episode accuracy; differences are paired on matched sessions against the full method):

| Pipeline | Pooled | Against the full method |
|---|---|---|
| Recipe without the low-pass reading | 31.9 | −21.4 |
| + low-pass reading | 42.3 | −11.0 |
| + alignment with the new hardware's stream | 51.1 | −2.2 (interval −5.7 to +1.1) |
| + grouped confidence (full method) | 53.3 | |
| Full method without the low-pass reading | 49.0 | −4.3 (−7.6 to −1.2) |
| Full method without alignment | 50.4 | −2.9 (−5.8 to −0.7) |
| Per-touch instead of grouped confidence (seed 0) | 52.2 | −1.1 (not significant) |
| Random groups instead of grasps (seed 0, D and reversed split) | 55.0 | −0.6 (not significant) |

- The full method gains 11.0 over the recipe (interval +5.4 to +17.0). Alignment and grouped confidence each give
  most of that gain alone and do not add up; removing either from the full method costs a little but significantly.
- The gain holds on sessions the stream never saw: aligning to the first half of each new state's sessions and
  scoring only the second half gives +12.3 (+5.6 to +19.7) over the recipe against +13.4 for the full stream.
- What the grouping contributes is the averaging, not the grasp identity: random groups of touches do as well.
- Weakness to report: alignment lowers the gel-change stratum (recipe 39.6, full method 37.6, recipe without the
  low-pass reading 46.3).
- SITR (two seeds, 12 ordered Mini pairs): accuracy on the classes learned on the old Mini rises from about 13 to 72
  with alignment and per-image confidence. With only 5 of the 20 classes in the new Mini's unlabelled images, the
  gain roughly halves but stays above no adaptation in every pair: the stream must cover the materials.
- Tactile MNIST cannot test the tie: single touches do not identify a printed digit even on one gel (about 20 on
  later days of the same gel), so every variant sits at chance.
- Running: per-touch confidence seeds 1-2, causal-stream seeds, the random-group objective on SITR with three seeds,
  group sizes.
