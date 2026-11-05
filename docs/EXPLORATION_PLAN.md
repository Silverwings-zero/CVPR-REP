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

## Progress (2026-11-01): the method's evidence with three seeds

- **Grouped against per-touch confidence (three seeds, 55 sessions).** Per-touch confidence on top of alignment is
  2.4 points below grouped confidence (interval −4.8 to −0.3) and adds nothing over alignment alone (−0.2). Grouping
  is the form of the confidence objective that pays on our data. Random groups (all three folds, seed 0) and groups
  of 4 or 16 touches instead of 8 (seed 0) are all within a point of grasp groups: the averaging matters, the group
  identity and size do not.
- **Sessions the stream never saw (three seeds, all three folds, 28 late sessions).** Aligning to the first half of
  each new state's sessions and scoring only the second half: +9.8 over the recipe (interval +4.3 to +15.8), against
  +11.9 with the full stream; the difference (−2.0, −4.9 to +0.2) is not significant.
- **SITR with three seeds (12 ordered Mini pairs).** The full method (alignment with grouped confidence on random
  groups of the new Mini's unlabelled images) raises accuracy on the classes learned on the old Mini from 12.8 to
  72.0, better in all 12 pairs. Grouped confidence adds 15.3 over alignment alone (+9.9 to +21.1, 12 of 12 pairs).
  Grouped and per-image confidence tie on SITR (−0.1), so the advantage of grouping is specific to our data.
- **Running.** Encoder generality: ResNet-50 and ConvNeXt-T (pretrained, fine-tuned) on the tube view, recipe against
  full method, three folds and three seeds on the local GPU; DINOv2-B the same on the cluster (it needs more than
  12 GB).

## Positioning (2026-11-01): what is borrowed, what is ours, and CVPR precedents

Each part of the method, with the published work it must be credited to:

| Part | Source | How close our version is |
|---|---|---|
| Alignment with an adversary | DANN (Ganin and Lempitsky, ICML 2015; JMLR 2016) | The same loss. A binary old-vs-new-hardware classifier on the features, with gradient reversal. Only the ramp schedule differs. |
| Per-touch confidence and spread | Information maximisation (Krause et al., NeurIPS 2010; Hu et al., ICML 2017), used for adaptation by SHOT (Liang et al., ICML 2020) | The same two terms as SHOT's IM loss. This is the ablation row "per-touch instead of grouped". |
| Grouped confidence | Marginal entropy over augmented copies of one test image, from MEMO (Zhang, Levine and Finn, NeurIPS 2022). The batch-level IM family includes BNM (Cui et al., CVPR 2020). | The same functional form as MEMO: the entropy of an averaged prediction. Our groups are several real touches from the new hardware, not synthetic copies of one image. It is used during training on the unlabelled stream and paired with a spread term across groups. MEMO is a must-cite. The shuffle control shows that random groups do about as well, so the claim is "average before asking for confidence", not "the robot knows which touches share an object". |
| Test-time low-pass reading | Its kin is FDA (Yang and Soatto, CVPR 2020): low-frequency image content carries sensor and illuminant style. Classic flat-field correction is the same family. | Ours. Each hardware state's look is estimated from the unlabelled mean images of the sessions recorded before it on that hardware. Only its smooth part (Gaussian, sigma 33 px) is subtracted at test time, without training. |
| Framing and protocol | — | Ours. A hardware change is treated as a shortcut problem, and the robot's own unlabelled touches after the change serve as the adaptation signal. The protocol pools cross-hardware sessions, with strata and paired bootstraps. |

CVPR method papers built the same way, from known parts plus a new problem and one or two new pieces (all checked against the papers):

- **MM-SADA** (Munro and Damen, CVPR 2020). It uses the DANN adversary, with a gradient reversal layer per modality, against kitchen-to-kitchen shift in egocentric video. Its new part is a self-supervised signal from the data's own structure: whether RGB and flow come from the same action. In its ablation, the known adversary gives +3.9 of the +4.8 total. This is the closest structural precedent.
- **ADVENT** (Vu et al., CVPR 2019). It combines entropy minimisation (Grandvalet and Bengio 2005) with adversarial training. One of its two methods is plain entropy minimisation applied per pixel.
- **FDA** (Yang and Soatto, CVPR 2020). The core is a training-free low-frequency spectrum swap. The rest is a known entropy penalty and self-training.
- **BNM** (Cui et al., CVPR 2020). It is one modified confidence-and-diversity loss, justified by analysis.
- **Complete & Label** (Yi, Gong and Funkhouser, CVPR 2021). The domain gap comes from changing the sensor hardware (LiDAR beam patterns). The contribution is identifying that gap and canonicalising it away.
- **CamStyle** (Zhong et al., CVPR 2018). An off-the-shelf CycleGAN is used against camera-to-camera style shift in re-identification.

Claim pattern for our paper: the problem and protocol are new. The method's design follows from the problem: the state reading removes the smooth hardware look, alignment uses the robot's own post-change touches, and grouped confidence averages before asking for confidence. Alignment and the IM terms are "adapted from" DANN and SHOT, and grouped confidence is credited to MEMO's marginal entropy. Each part is justified by an ablation row. The baselines are DANN alone and DANN plus per-touch IM, which together amount to SHOT-IM-style adaptation, so the paper beats the standard recipes rather than merely using them.

## Baselines (2026-11-01): newer methods for the comparison table

Until today, the only methods run on the pooled protocol were DANN (2015) and CORAL (2016). The comparison now uses newer published methods. Each matches its official code, uses its paper's default hyperparameters, and runs on the same encoder, budget and unlabelled touches as our method.

- **Retraining with the new hardware's unlabelled touches**, the same information our full method uses: CDAN+E (NeurIPS 2018), MCC (ECCV 2020), SDAT (ICML 2022, CDAN with MCC and a sharpness-aware classification step) and MIC (CVPR 2023, on SDAT). UDA-Bench (Kalluri et al., ECCV 2024) found that newer methods often do no better than CDAN under a standardised setup, which is why CDAN stays in.
- **Adapting on the stream at test time, without retraining:** test-time BatchNorm, Tent (ICLR 2021), SAR (ICLR 2023), RoTTA (CVPR 2023) and DeYO (ICLR 2024). The stream is in recording order, so each batch holds one material. RoTTA was designed for exactly that kind of stream. The model is reset at each hardware change, which is the same knowledge our reading uses.
- **Fairness:** each retraining baseline gets the two trade-off weights we tried for our own method (1.0 and 0.3) on seed 0 and keeps its better one for seeds 1 and 2. Every baseline is also scored with our low-pass reading added. The table groups methods by the information they use. Test-time methods are compared with our reading alone, which also needs no training. Retraining methods are compared with the full method.
- **Code and runs:** `scripts/xp/train.py` (flags `--uda cdan`, `--mcc`, `--sam`, `--mic`) and `scripts/xp/tta.py`. Cluster wave 6 (job 13653737).

## Results (2026-11-02): newer baselines, test-time methods, encoders

- **No newer retraining method beats ours on seed 0.** Seeds 1 and 2 are running. Each baseline's better trade-off on seed 0 was 1.0 for CDAN+E, MCC and MIC, and 0.3 for SDAT. With our low-pass reading added, the closest is CDAN+E, 3.3 points below the full method; that interval just touches zero (−7.2 to 0.0). MCC and SDAT are 3.5 to 4.5 below, and MIC is 11 below. Without our reading, all four are 7 to 12 below. So the three-seed result decides whether we can write "beats CDAN+E" or only "matches it".
- **Our reading helps every published method.** It adds 3 to 5 points to each retraining baseline, so it works as a plug-in and is not tied to our training. MIC does worse than SDAT, the method it is built on. We report it at its paper's defaults.
- **Test-time methods that use batch statistics collapse on the robot's stream.** In recording order each batch holds one material. Test-time BatchNorm, Tent, SAR and DeYO normalise that material away and fall to chance on all three seeds. RoTTA was built for correlated streams. It survives, but it does not improve on the source model (+2.0, not significant). Our reading alone also needs no training, and it beats RoTTA by 8 points. The methods' own setting, a shuffled stream within each hardware state, is running (wave 7), so the collapse cannot be blamed on the stream order alone.
- **The training-time part carries over to pretrained encoders (three seeds, tube view).** Against each encoder's recipe with our reading, the full method adds 11 points for the CNN, 18 for ResNet-50 and 23 for ConvNeXt-T, so it adds more to larger encoders. DINOv2-B gains only 3 (+0.6 to +5.3). Its overall level on the 192-pixel tube crops is low (39, against 68 for ConvNeXt-T), while on whole frames with alignment it reaches 68 (seed 0).
- **On whole-frame views, alignment adds 28 to 48 points over each encoder's frame recipe (seed 0).** The low-pass reading does not help on frames. It hurts the ResNet-50 and centre-frame CNN rows and is neutral for ConvNeXt-T and DINOv2, so it is a tube-view component. With the reading, the centre-frame CNN scores zero on the top-10 materials of fold D. That is unexplained and must be checked before any frame row with the reading goes into the paper.
- **Next:** CDAN+E alignment combined with our grouped confidence (local queue 17, appended). This asks whether grouped confidence adds to a stronger aligner as well as to DANN.
- **Operations:** the cluster puller had pulled nothing since it was written, because rsync 3.2.4 and later no longer expand `$USER` on the remote side. It is fixed (HapTeleop 42ebb28).

## Results (2026-11-02, noon): three seeds change the baseline picture

- **CDAN+E with our reading ties the full method over three seeds** (+0.4, interval −2.3 to +3.2). On seed 0 it was 3.3 below; seeds 1 and 2 favoured it. Without our reading, CDAN+E is still 3.9 below (−7.5 to −0.4). SDAT and MCC with our reading are about 3 below, with intervals that just reach zero. MIC is 10 below with or without it.
- **Grouped confidence adds to CDAN+E as much as to DANN (seed 0).** CDAN+E with grouped confidence and our reading is 3.6 above CDAN+E with our reading (+1.2 to +6.8). DANN gains 3.7 from the same term on the same seed. If this holds over seeds, our two pieces (the reading and grouped confidence) improve whichever aligner sits under them, and the best row becomes CDAN+E with both.
- **Caution on our own ablation.** Over three seeds, the full method is 2.2 above DANN with our reading, and that interval includes zero (−1.1 to +5.7). Seed 0 alone showed +3.7. So the grouped-confidence gain is real on seed 0 for both aligners but not yet established across seeds. Seeds 3 and 4 for the four rows that decide this are on the cluster (wave 8): full method, DANN, CDAN+E, and CDAN+E with grouped confidence. Seeds 1 and 2 of CDAN+E with grouped confidence run locally.
- **Test-time methods on a shuffled stream (the methods' own setting).** BatchNorm, Tent, SAR and DeYO no longer collapse. They sit 4 points below the source model, which is not significant, and 14 points below our reading alone. RoTTA gains 7 over the source model, not significant, and stays 3 below our reading alone, also not significant. Adding our reading to any test-time method changes nothing, because re-estimating BatchNorm statistics already removes the smooth offset the reading removes. So the honest statement is that no test-time method beats the reading, which needs neither training nor a stream, and that the batch-statistics methods fail on the robot's real one-material stream.

## Results (2026-11-02, 13:30): grouped confidence does not add to CDAN+E over three seeds

- **The seed-0 gain did not hold.** Over three seeds, CDAN+E with grouped confidence and our reading is 0.4 above CDAN+E with our reading (−2.1 to +2.7). Seed 0 had shown +3.6 because CDAN+E's seed-0 run was unusually weak.
- **Where the three-seed ablation stands (tube view, CNN, with our reading unless noted).** The recipe without the reading scores about 32. Our reading lifts it to 42. Alignment with the robot's post-change touches adds about 9 more (DANN 51, CDAN+E 54). Grouped confidence adds 2.2 to DANN, with an interval that includes zero, and 0.4 to CDAN+E. So the two parts that clearly work are the reading and alignment with the new hardware's unlabelled touches. Grouped confidence is at best a small gain.
- **Story consequence (for the user).** If seeds 3 and 4 confirm this, grouped confidence cannot be sold as a contribution. The defensible method is then the framing, the reading, and alignment with the robot's own post-change touches using a published aligner. That is thinner as a method paper, which was the blocker the simulated reviews named.
- **Seeds 3 and 4** for the full method and DANN run on the cluster (wave 8, tasks 1 to 6). The CDAN+E rows run locally (queue 19).
- **The centre-frame anomaly is explained, not a bug.** The crop geometry of the reading is correct. The centre-frame recipe already collapses onto one material per fold: 43 percent of fold D's predictions go to one class, and 51 percent of fold AB's to another. The reading pulls every frame's look towards the training mean, and a classifier that reads the whole frame's look then sends almost everything to that same class (82 percent on fold D). Neither class is in the top ten, hence zero there. Tube crops, centred on the contact, do not carry the frame-wide look, which is why the reading helps on them. This belongs in the paper as the reading's scope: contact crops, not whole frames.
- **Five seeds on CDAN+E (16:40).** Grouped confidence on top of CDAN+E is +0.4 with our reading (−1.9 to +2.6) and +1.1 without it (−1.3 to +3.2). The null result on CDAN+E is confirmed. Seeds 3 and 4 of the full method and of DANN run locally (queue 20, about 20:00), after five hours of waiting in the cluster queue.

## Final ablation (2026-11-02, 20:00): five seeds, CNN, tube view

| Row | Pooled cross-hardware accuracy | Step over the row above |
|---|---|---|
| Recipe | about 32 | |
| Recipe with our reading | 42 | +10 (training-free) |
| DANN with the robot's post-change touches, with our reading | 51.7 | +9.4 (+4.4 to +14.5) |
| CDAN+E, same information, with our reading | 53.3 | +1.7 over DANN (+0.2 to +3.5) |
| DANN with grouped confidence, with our reading (our full method) | 53.4 | +1.7 over DANN (−1.7 to +5.3), not significant |
| CDAN+E with grouped confidence, with our reading | 53.8 | +0.4 over CDAN+E (−1.9 to +2.6), not significant |

- **Verdict.** The two parts that work are the training-free reading and alignment on the robot's own unlabelled touches after the change. Grouped confidence adds between nothing and two points on either aligner, and that gain is not significant at five seeds. It cannot be claimed as a contribution.
- **What still stands.** The reading lifts every published retraining method by 3 to 5 points. It beats every test-time method on the robot's stream, and no test-time method beats it on a shuffled stream. Alignment carries over to ResNet-50 and ConvNeXt-T, with larger gains than on the CNN. SITR's +15 for grouped confidence over alignment alone was on a different problem (a new Mini unit with label-tied classes). It is not evidence on our data.
- **Story decision needed from the user** (see the conversation of 2026-11-02).

## The method, final (2026-11-02, 23:00)

**Decision (data owner): what works is the method.** It has two parts. The first is the training-free low-pass reading of every test image relative to the running state estimate, with the detector that dates changes. The second is alignment of the encoder with the robot's unlabelled post-change stream, using CDAN+E (Long et al. 2018) adapted without changes to its loss. CDAN+E is the strongest aligner on five seeds (+1.7 over DANN, interval +0.2 to +3.5). Grouped and per-touch confidence become ablation rows, and neither is significant. The paper no longer discusses whether the method is novel.

**Paper rewritten to this method** (abstract, intro method paragraph and contributions, related work on adaptation, method section, experiments, discussion). The experiments now use the pooled 55-session protocol throughout. Every number comes from `HapTeleop/scripts/xp/paper_numbers_xp.py` → `numbers_xp.tex`. Runs still in flight print as a bold ??.

**Running, so every row uses the final method:**
- Local queue 21: CDAN+E on the first half of each state's stream, scored on the unseen second half (CNN); ResNet-50 and ConvNeXt-T tube with CDAN+E. Three seeds each.
- Local, before that: SITR with CDAN+E (12 Mini pairs, three seeds), and its 5-class stream.
- Cluster wave 9 (job 13689694): DINOv2 tube with CDAN+E; whole-frame ResNet-50, ConvNeXt-T and DINOv2 with CDAN+E (three seeds), and their frame recipes for seeds 1 and 2.
- Cluster wave 10 (job 13689926): variants of the reading, evaluated on saved models without retraining. The variants are the full field against the low-pass, low-pass widths of 8 to 132 px, the prior weight, the last session only, the own session (non-causal), and restarts at the detector's alarms instead of the true dates. They run on the recipe and on CDAN+E.

**Still to redo for the new method:** the SITR figure (Fig. 3 shows the old tracker reading), the method figure, and the supplement, which is still written for the old protocol.


## Architecture candidates (2026-11-04)

Asked for modules that could still move the numbers. Three were built, run in sequence, and each becomes a method part or an ablation row depending on what it measures.

1. **Factored state (reading variant `unit`, `unit_k3`).** Premise: the smooth field belongs to the unit, so a gel replacement on the same unit should not restart the estimate. Measured first: the gel swap moved the low-pass field by 2.6 grey levels against 4.7 for the unit swap (within-state session spread about 1), so the premise is only half true. Result: the unit stratum is unchanged and the gel stratum falls, 36.7 to 22.4 on the recipe model and 33.5 to 24.1 on the full model. **Negative.** The detector-dated and the stronger-prior variants gained on that stratum only because they read less. On those eight sessions the state is a valid cue for the classes learned under it, and every reading removes part of it (raw is best there on both models). Stated in the discussion.
2. **Contact-gated pooling (`--pool contact`).** The encoder's positions are pooled with softmax(tau * m) weights, m the local evidence of touch (|crop - full state estimate| over colour, Gaussian 8 px, mean 1 over the crop), tau learned from 1. Positions with no touch show only the sensor and count less; tau -> 0 recovers mean pooling. CNN and ResNet-50. Runs: local queue 23 (tube: `cnn_tube_geo_cdan_pool`, `cnn_tube_geo_pool`), cluster wave 11, job 13711773 (whole frames: `cnn_frame384_geo_cdan_pool`, `cnn_frame384_geo_cdan`, `cnn_frame384_geo_pool`), after a CPU smoke test (13711821) of the frame paths, which this machine cannot run.
   **Result on tubes (fold D, seed 0, 2026-11-04 03:45): negative in all four comparisons.** Recipe 35.2 -> 21.4 with the top-class share doubling (27 -> 53), recipe with reading 53.9 -> 47.9, CDAN+E 48.5 -> 45.2, full method 52.0 -> 49.4. The mask is itself a hardware cue: |crop - estimate| contains the state difference wherever the estimate is imperfect (the test estimate holds the training mean and few sessions; the training estimate holds many), so its pattern tells the classifier the hardware and strengthens the shortcut. Queue 23 stopped after the first pair; the whole-frame runs of wave 11 stay on the cluster as the last word for the view where the mechanism was expected to matter. A state-free touch evidence (the gel's own temporal change within a grasp, or marker displacement) would be the next design, not pursued before the deadline.
3. **Bootstrapped train-time reading (`--reading boot_lp`).** Training crops are read relative to a random subset of the other training sessions of their hardware (drawn per crop, the training mean as one more session), so a sparsely recorded state leaves no stable ghost of its few objects; stream and test crops are read relative to the causal estimate. Train and test then see the same kind of input, and the classes learned on a sparsely recorded state lose their state cue in training as well. The bootstrap uses 1/4-resolution low-passed session fields and matches the full-resolution reading to 0.001 grey levels. Learned band weights were dropped: on the full model every low-pass from 33 to 132 px lies within a point, so learned weights would reproduce the fixed reading. Runs: local queue 24 (`cnn_tube_geo_cdan_boot`, `cnn_tube_geo_boot`), after queue 23.
   **Result (three folds, three seeds, 2026-11-04 07:30): not a method part; ablation row h.** The full method trained with the bootstrapped reading reaches 51.6 against 53.7 for the raw-trained model (-2.1 [-4.9, +0.7]); tested raw it reaches 42.9 against 49.3. The recipe trained with it reaches 37.8 under the test reading against 42.0 for the raw-trained recipe (-4.1 [-9.2, +0.1]) and 32.4 raw against 31.8. The gel gain of the first pair (fold D seed 0: 18 -> 40) did not pool: gel 33.3 -> 35.8, unit 57.2 -> 54.3. Likely mechanism: the raw-trained model keeps the training states in its features and the test reading moves every new image towards them; a model trained on read images has no such anchor, and loses the state cue that the sparsely recorded classes were learned with. The reading stays a test-time operation.

Whole-frame rows from wave 9 (2026-11-04): the reading on whole frames depends on the encoder. On top of CDAN+E it costs ResNet-50 9.4 points [-15.3, -3.9], changes ConvNeXt-T by +1.3 [-1.9, +4.4] and gains DINOv2-B +10.7 [+6.2, +15.6]. ConvNeXt-T on whole frames, aligned and read, reaches 60.4, the highest number in the paper (the tube CNN method is 53.7). The frame paragraph of the encoders section was rewritten from "the reading does not help on frames" to this. Wave 11 (whole-frame CNN with and without pooling, job 13711773) was still pending at 07:40.

Also queued on SITR: CDAN+E with per-image confidence (`jitter_cdan_im`), to complete the confidence rows of the SITR table. Not pursued: state-swap augmentation (tried, `--aug state`, no gain), feature-statistics adaptation (shuffled BN sits below the recipe), a marker-displacement branch (prior art, different modality), confidence objectives on our stream (measured, nothing). To add as a baseline row: FDA (Yang and Soatto, CVPR 2020), the closest published method to the reading.

## Headline switch to ResNet-50 on tube crops (2026-11-04)

User decision: the headline model is the best-performing one. First pick, ConvNeXt-T on 384-px whole-frame crops, aligned and read: 60.4 pooled. Corrected the same day: ResNet-50 on the 192-px tube crops (ImageNet weights, default lr), aligned and read, reaches 61.6 (unit 64.5, gel 44.5), and on it both parts of the method are active (recipe 32.4, reading alone 44.8, CDAN+E alone 53.5, both 61.6; full vs reading +16.8 [+10.7, +23.3]), whereas on ConvNeXt-T whole frames the reading adds +1.3 [-1.9, +4.4] on top of alignment and nothing alone (14.8 -> 13.3). ResNet-50 is also a BatchNorm backbone, so all five test-time baselines apply, and a tube run takes 8 min (recipe) to 16 min (CDAN+E) on the local GPU, so the suite does not depend on the cluster queue. The ConvNeXt-frame waves 12, 13 and 13c were cancelled before they started (their task files stay in `cluster/tasks/` for the record); the existing ConvNeXt-frame rows remain in the encoders table as the best whole-frame model.

- Local queue 26 (99 runs, `cluster/tasks/local_queue26.tsv`, started 13:40): recipe with checkpoint (`r50_tube_geo_src`) and the five test-time methods in recorded and shuffled order; DANN five seeds; CDAN+E seeds 3-4; grouped confidence five seeds; per-touch confidence; MCC; SDAT; MIC; CORAL; the early-stream memorisation test. About 24 h.
- Local queue 27, after 26: every reading variant on the recipe checkpoint. Then queue 25 resumes (FDA on the CNN).
- Cluster wave 14, job 13722340 (27 tasks, 63 runs, `wave14_headline_r50.tsv`): SDAT and MIC at the second trade-off weight (the better on seed 0 enters the table, as for the CNN), FDA at beta 0.01 and 0.05, the bootstrapped reading (row h), the full model with checkpoint and all reading variants on it.

FDA (`--fda beta` in train.py): every training crop takes the amplitude spectrum of a stream crop inside the centred square of half-width beta times the crop size, phase unchanged; beta 0.01 on a 192-px crop swaps the lowest one or two frequencies, the band that the sigma 33 px low-pass of the reading removes, so it is the training-side counterpart of the reading and needs the stream before training. Smoke-tested end to end on the GPU (FDA with CDAN+E, prediction and test_lp written).

Macros: `paper_numbers_xp.py` generates every method row per model family (`method_rows(prefix)`); MAIN emits the plain `\Xp` macros and ALT the prefixed ones (`Hd` for ResNet-50 now). When the suite is complete, MAIN and ALT swap and the CNN rows become `\XpCnn...` in the encoders table; the paper text that describes the recipe (CNN from scratch) changes to ResNet-50 fine-tuned from ImageNet on the same native tube crops, and the CNN becomes the from-scratch row.

### Status 15:30 and the paper branch

- Queue 26 at task 11 of 117 at 15:15, no failure; measured pace: recipe 8 min, aligned run 16 min, each test-time line 13 min, so the queue ends about 18:00 on Nov 5, then queues 27 and 25 by about midnight. Wave 11 (CNN frame pooling, item 2) started at 15:00 on Nov 4, 8 h after submission; wave 14 is pending without an estimate.
- `paper_numbers_xp.py --headline {cnn,r50} [--out path]`: the headline family gets the plain `\Xp` macros, the other family a prefixed copy (`Hd` for ResNet-50 while the CNN is the headline, `Cnn` for the CNN once ResNet-50 is). The better trade-off weight of SDAT and MIC is now chosen by the script: the better of the two on seed 0 after the reading, which reproduces the CNN's rows (sdat03 51.6 vs sdat 50.6; mic 44.0 vs mic03 42.9). Measured raw, sdat would win by 0.1 with a single seed, so the rule is stated as "after the reading" and the supplement should say so.
- Branch `r50-headline` of CVPR-REP (e13a1d5) holds the paper written for the ResNet-50 headline: recipe paragraph (ResNet-50 from ImageNet, fine-tuned on native 192-px tube crops), abstract, intro, confound sentence, ablation caption, encoders table with the CNN as the from-scratch row, the frame paragraph, and FDA rows and a paragraph in the baselines. Every claim that waits for a ResNet-50 row carries a `TODO(r50)` comment. Main keeps the CNN headline so that Overleaf compiles with complete numbers; the branch merges when the suite lands.
- To watch when the rows land: on ResNet-50, DANN with grouped confidence and the reading (`r50_tube_geo_dann_imsess@test_lp`, an earlier run) scores 62.9 (unit 66.0, gel 45.0), above the full method at 61.6. DANN with the reading alone and CDAN+E with the grouped confidence are in queue 26. If the grouped confidence adds to alignment on this encoder by a margin clear of zero, it returns to the method and row f changes meaning (the rule of 2026-11-02: the method is what works, dropped parts are ablation rows). The recipe model's top-class share is about a third on ResNet-50 (33 on `r50_tube_geo`), not a quarter as on the CNN; the confound sentence changes with the `_src` run.

### Status 21:45: the ResNet-50 test-time rows and DANN (three seeds)

- Test-time methods on ResNet-50, robot's stream: BN, Tent, SAR, DeYO about 10.5 (collapse); RoTTA 41.7, +9.3 [+1.0, +17.9] over the unadapted 32.4, so on this encoder RoTTA works (on the CNN it did not, +2.0 n.s.). The reading 44.8 is +3.1 [-3.6, +9.7] over RoTTA (CNN: +8.2 [+0.1, +16.4]). Shuffled: BN family 37.8, RoTTA 48.2, +3.4 [-2.6, +9.2] over the reading. RoTTA on read images 42.1, +0.3 over RoTTA. RoTTA's gain is in the unit stratum and it loses the gel stratum (52.2 to 18.2), where the reading keeps 48.9. The branch now says the reading matches the best test-time method (abstract, intro, discussion, the test-time paragraph); main keeps the CNN claim.
- DANN on ResNet-50 (three seeds; seeds 3-4 running): 53.4 raw, 60.6 with the reading (+7.2). The full method is +1.1 [-1.3, +3.0] over DANN with the reading. DANN with grouped confidence and the reading (62.9) is +2.3 [-2.4, +7.3] over DANN with the reading: not clear of zero, so grouped confidence stays out unless CDAN+E with it (queue 26) shows a clear gain.
- The reading lowers the other-gel stratum on both encoders (ResNet-50: recipe 52.2 to 48.9, CDAN+E 53.1 to 44.5; CNN: 44.0 to 37.0, 40.5 to 33.3). The Discussion already states it.
- Pace: aligned runs take 16.8 min, the recipe 9.2 min. The 75 remaining lines of queue 26 take about 24 h, so queue 26 ends about 21:00 on Nov 5 and queues 27 and 25 by about 03:00 on Nov 6. Wave 11: 7 of 9 tasks done, two pending. Wave 14: not started.
