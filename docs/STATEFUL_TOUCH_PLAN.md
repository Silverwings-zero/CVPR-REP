# Stateful Touch — full plan (2026-10-27)

Decision (user): execute everything to the fullest; no feasibility-driven cuts. Theme: a stateful
tactile representation that a robot maintains from its own stream. The finding (the hardware state in
every image, learned as a class feature) stays the paper's backbone; the method becomes one specified
procedure with stated conditions, evaluated on our data and on public multi-hardware data.

## The procedure (to be written as Algorithm 1)

State: per-pixel running sum A(p) and count n(p) in frame coordinates over the current hardware epoch;
estimate F̂(p) = (A + k·G)/(n + k) with G the training mean and k = 2 pseudo-counts; markers inpainted.
1. **Observe** a contact crop x at position p (any object; no label).
2. **Detect** a hardware change: the fine-band (190–370 µm, ≥ 60 px from markers) correlation of the
   current window's mean with F̂, and the illumination-field distance; a change is declared when both
   fall outside the same-hardware range for m consecutive episodes (thresholds set on the training
   hardware only). On a change: start a new epoch (A, n ← 0), keep the old estimate as the fallback.
3. **Read** every image relative to the current state: x′ = x − F̂(p) + G(p); a bare-gel frame, when the
   gripper is open, enters A with its own count.
4. **Heads and encoders** see x′ at training and test (consistent protocol); frozen foundation models
   additionally get the label-free state-swap adapter (positives = contacts re-rendered under other
   epochs' states, decoys = other surfaces under the same state).
5. **Geometry** (force, contact) is read against the unloaded gel: marker displacement vs the bare-gel
   grid; contact from f(x) − f(bare gel).
Conditions, each backed by a measurement: additive rendering state (ratio = subtraction; per-pixel
response models do not help); the estimator's bias is the mean imprint of the averaged objects (own
session is the worst source; diversity cures it — convergence curve); covers rendering changes (unit)
and not mechanical ones (gel replacement; force scale changes).

## Work packages

### A. Our data (the long-term, confounded, real-world test)
- A1 `scripts/state_tracker.py`: the procedure as one module; streaming evaluation over the 55-session
  chronology (causal estimate per session; automatic epochs); change-detection ROC with thresholds
  calibrated on S1 only, ground truth Jul 10 and Oct 9 (latency in episodes, false alarms incl. the
  canvas session); accuracy per session over time with/without epoch resets.
- A2 Convergence: accuracy vs number of contacts and vs number of distinct objects in the estimate.
- A3 Hardware-decodability probes on frozen foundation models (Sparsh ×3, DINOv2) and our CNNs:
  "which unit/gel" from the image, given the material; before/after state-relative reading.
- A4 Paper numbers: far-from-marker signature (base → far60), updated Figure 1 and §3.

### B. Public multi-hardware data (generality)
- B1 SITR classification: identify the four Minis; SITR's protocol (head on one sensor, test on the
  others); rows: raw, their blank-subtracted, our stream estimate (no blank, no presses), + adapter;
  encoders: our CNN at 224 (no downscale of OUR data; SITR is released at 224), frozen Sparsh, ImageNet
  ViT-B; compare with SITR's published 73.2 / 90.2. Hardware decodability per encoder.
- B2 FEATS: 5 Minis × 6 gels; hardware decodability; force regression (their U-Net or a CNN) across
  held-out sensor/gel with raw vs state-relative inputs; the gel change as the mechanical boundary.
- B3 Real Tactile MNIST: change detection on its 24 runs (the gel swap), digit accuracy across runs
  with the tracker.
- B4 RCT (when downloaded by the user): materials on 3 DIGITs, same session — the only public
  texture-with-hardware-crossing set; raw / blank / our estimate.

### C. Paper
- Method section = Algorithm 1 + conditions + convergence + detection figures.
- Experiments: our rotation (+ tracker rows), hardware-split table, public-data table, probes.
- Abstract / intro / related work updated; discussion: scope and limits.
- Full adversarial review of the draft; fixes; LLM statement; supplement.

### D. Order
A1 → A2 → B1 → A3 → B2 → B3 → A4 → C → D; B4 whenever RCT arrives.

## Notes while executing
- SITR classification: the four GelSight Minis are sensors 0, 4, 5 and 6 (`gsrl/plot_classifier.py`:
  `confusion_across_sensor([0,4,5,6], ...)` = "intra"); each sensor folder has `calibration/0000.png`
  (background) + ~18 presses and `samples/class_XXXX/*.png` (224 px). 20 classes released, 16 used by the
  paper. Protocol: head trained on one sensor, zero-shot on the others.
- FEATS: samples are dicts {gs_img (240,320,3), grid_x/y/z (24,32), f_x,f_y,f_z}; files `<series>_<ns>_<shape>_<params>.npy`;
  no-contact frames `<series>_nocontact_sensor_<s>_gel_<g>.npy` (sensor 0-4, gel 0-5); splits train/val/test/
  test_unknown_indenters/test_diff_sensor_old_gel/test_diff_sensor_new_gel. Contact samples carry no sensor/gel
  id: assign by matching each sample's periphery (non-contact pixels) to the no-contact frames.
- FEATS assignment by periphery matching: sensor identity is clear per split (train/val/test mostly sensor 2,
  some 1/3/4; test_diff_sensor_* = sensor 0: old gel = gel 0, new gel = gels 1-4), gel identity within a
  sensor is ambiguous for contact images (margins < 1 grey level for ~60 %). Use sensor-level blanks for the
  "bg" reading; the "stream" reading needs no identity (mean of the split's contacts). `scripts/feats_crossgel.py`.
- SITR smoke test (20/class train, 10/class test, 2 epochs; train Mini-0): the hardware is fully decodable from
  every frozen feature (probe 93-100 % among the four Minis, class held fixed). Frozen DINOv2: Mini->Mini 67.8
  raw, 66.7 with SITR's background image, 74.3 with our contact-stream estimate. Sparsh-I-JEPA fed a still image
  as two identical frames is out of its regime (46.5 % even on the training sensor): report it as released, with
  the caveat; SITR's own baselines are ImageNet ViTs. Full runs (200/class, 12 epochs, train sensors 0/4/5/6):
  scratchpad sitr_block.sh -> outputs/tacrep/sitr_block.log.
