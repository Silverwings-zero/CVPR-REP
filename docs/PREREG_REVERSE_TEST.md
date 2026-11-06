# Pre-registered held-out test: train on C + D, test on A + B (written 2026-10-28, before any model for it exists)

All design choices of the method were made on the split that trains on A + B + C and tests on D. This test reverses
the direction, with every setting frozen as it stands in commit `60e826b` of this repository (paper) and the scripts it
cites. Nothing below will be tuned after seeing the result, and the result will be reported whatever it is.

## Data
- Training: campaigns C (July 9-13) and D (September 22 - October 15). Hardware in training: wood on unit 1 + gel 1
  (S1, one session, July 9); cork, fibreglass, ABS, ceramic on unit 2 + gel 2 (S2, July 10-13); fifteen A/B materials and
  canvas on S2 (D, before October 9 18:42); cork, fibreglass, ABS, wood, ceramic on unit 2 + gel 3 (S3).
- Test: campaigns A (June 15-22) and B (July 6-8): the fifteen A/B materials on S1, which the training data hold only
  for wood. Classes: all 20 (chance 5 %); macro accuracy over the fifteen test materials, macro-F1, top-class share.

## Models (frozen recipe)
- `scripts/tacrep_swap.py --train-eras CD --test-eras AB --ce 1 --frames 1 [--aug] --seed {0,1,2}`: plain CNN and
  photometric-jitter CNN, three seeds each, 12 epochs, AdamW, single native 192-px crops, as for every other fold.

## Readings (frozen)
- raw: raw training and raw test images, MLP heads (three seeds).
- Algorithm 1: test crops read relative to the tracker's causal estimate on the whole stream in recording order (per-object
  mean, one-object prior, low-pass prior after a restart), restarted at the alarms of the whole-frame detector; training
  crops read relative to the mean of the other training sessions of their detected hardware epoch; G = mean of the
  training sessions.
- Detector: whole frames, K = 8, W = 30, m = 5, fine band 190-370 um, sigma 370 um, markers >= 60 px; thresholds =
  10th / 95th percentiles of the two statistics over the episodes of the training data's largest hardware state (S2),
  because the training data's first state (S1) holds a single session.
- Baseline: AdaBN with BatchNorm statistics re-estimated on all test crops (one hardware state).

## Predictions (directional, stated before running)
1. Raw, the plain CNN collapses onto wood, the only training class recorded on S1: wood is the most predicted class and
   receives more than a fifth of the test episodes.
2. Algorithm 1 raises the plain CNN's accuracy on the fifteen test materials above raw, and lowers the top-class share.
3. The jitter CNN collapses less raw and gains less from the reading than the plain CNN.
