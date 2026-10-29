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
