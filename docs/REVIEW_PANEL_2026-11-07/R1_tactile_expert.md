# R1 - Tactile sensing expert
Primary skill: skills/deer_flow_academic_paper_review.md. Also used: skills/paperjury_personas.md (gatekeeper core, evidence-anchor rule), skills/neurips_form.md, skills/critique_framework.md, skills/manuscript_review_checklists.md.
CVPR rating: 3/6. Confidence: 4/5.
NeurIPS-form sub-scores: Soundness 3/4, Presentation 3/4, Contribution 2/4, Overall 4/10.
One-line verdict: A careful, honest diagnosis of a hardware shortcut in GelSight material recognition, but the fix is never compared with the field's default remedy (a reference frame of the bare gel), the headline is reported in a non-standard input view where it happens to work, and one SITR claim contradicts the paper's own figure.

## Summary
The paper argues that every GelSight Mini image carries a sensor-specific component (a fine pattern plus a smooth illumination field), measures it over four months, two units and three gels, and shows that when each material is recorded under one hardware configuration a classifier learns this component as a class cue and fails after the unit or gel is replaced. The failure is reproduced by injecting the same tie into SITR. The remedy is a training-free "reading" that subtracts the low-pass of a running mean of the robot's own contacts (a retrospective flat-field), plus an unchanged CDAN+E alignment with the unlabelled stream when retraining is possible. On 55 cross-hardware sessions a ResNet-50 goes from 32.4 to 44.8 (reading) and 61.6 (both). A dataset with native video, Nano17 force and bare-gel frames is promised.

## Claims vs evidence
- C1 State present, stable, replaced with unit or gel (Fig. 1, Sec. 3). Strong for S2, the only configuration spanning months.
- C2 Classifiers use it as a shortcut (64 percent to unit-2's four materials; SITR 73.9 to 37.0). Strong; the SITR control is same-time.
- C3 Reading matches the best TTA method and improves every UDA method (Table 1). Strong on tube crops only (Table 3).
- C4 Full method 61.6 (Table 2). Moderate: transductive stream; row i keeps three quarters of the gain.
- C5 Fine pattern is cartridge optics, not the surface. Weak, by the authors' own admission.
- C6 Reading does not help on SITR. Contradicted by Fig. 3 (W3).

## Strengths
S1. The Sec. 3 measurement is new for this community. Nobody has followed one unit and gel for months, and the Fig. 1 ordering (own session 0.48, same hardware months later 0.29, other unit 0.04) with the bare-gel match at 0.59 shows the fine band is hardware-bound rather than object, force or lubricant.
S2. The shortcut argument is built the right way: a natural occurrence, then an injected tie on a public dataset where every object was pressed on every Mini (verified in SITR).
S3. Reporting is unusually honest: collapse metric, paired bootstrap intervals, 3 to 5 seeds, the strata where the reading hurts, a pre-registered fold, negative ablation rows.
S4. The stream setting is realistic and shows that Tent, SAR, DeYO and test-time BN collapse on single-object batches.

## Weaknesses (ranked)
W1 (Major). The field's default remedy is never run. Sec. 4: "The reading needs no labels, no training and no bare-gel frame." and "in most archived data, no frame of the bare gel." Every GelSight pipeline the paper cites subtracts a reference frame, and the bare gel matches its session's contacts at 0.59, so a reference frame removes both the smooth field and the fine pattern pixel-exactly. On a gripper the gel is bare whenever the jaws open, so "no bare-gel frame" describes campaigns A to C, not a working robot. Campaign D has bare-gel frames, so the decisive control is cheap: replace F-hat in Alg. 1 by the session's bare-gel frame (low-passed and full-band) and report both next to row b. If the stream estimate matches the oracle, the reading is justified for archives; if the blank does not remove the shortcut, the paper becomes much stronger.
W2 (Major). The headline is reported in the view where the method works. Sec. 5.3: "On tube crops both parts of the method work for every encoder, which is why the paper's comparisons use that view." In the frame view the reading costs ResNet-50 nine points and adds nothing to ConvNeXt-T, whose 60.4 nearly equals the headline 61.6. This community recognises materials from whole frames or difference images at 224 to 320 px; that pipeline is absent ("We never resample a frame."), so a reader cannot tell whether 2-mm native crops create the shortcut's severity or merely expose it. Fix: add the standard whole-frame, blank-subtracted pipeline as a row and state the view dependence in the abstract.
W3 (Major). Text and Fig. 3 disagree on SITR. Sec. 1: "SITR's Minis also differ in their field of view and contrast, and there the reading does not help, but alignment does." Sec. 5.4: "The reading cannot repair a tie between classes and Minis there". Yet Fig. 3's blue curve, labelled in the legend as Algorithm 1 (causal), lifts SITR accuracy from about 37 to 70 at k=1 and 24 to 53 at k=4 with frozen DINOv2. Either the settings differ (frozen DINOv2 vs CNN from scratch) and the text must say so, or one is wrong. The legend's "state augmentation" is never defined in the main text.
W4 (Major, fixable by rewording). Sec. 3: "We therefore take it to be an optical property of the cartridge, fixed in the camera's view, rather than texture on the deforming surface." The only evidence is that a new gel removes it, which fits equally the reflective coating's pigment granularity on the deforming surface, since the coating is part of the cartridge. The warping test (0.59 to 0.54, median 2.1 px) cannot discriminate at a 17 to 33 px band, as the paper admits. State both hypotheses and stop at "belongs to the cartridge".
W5 (Minor to Major). Sec. 6: "No material was recorded on the same unit and gel months apart, so stability over months is shown for the state itself and not for a classifier." Every test session differs from training in hardware and in campaign, so the residual gap (61.6, not 90) cannot be attributed, and the four-month stability rests on one configuration with a two-month gap. A benchmark without a same-hardware, later-time control is less useful than the contribution list suggests.
W6 (Minor). Sec. 1 says SITR "pressed the same 20 objects on four GelSight Minis" while Sec. 2 says "a 16-object classifier"; SITR reports 16 (verified). Table 2 row e prints "-0.8 [-2.0, --0.5]" (sign glitch; the text says -0.5 to 2.0). The gel stratum has 8 sessions of 4 materials, too few for Sec. 6's mechanism claim. The reading "needs to know when the robot moves to a new object", which the TTA baselines are not told. The sigma sweep is reported on the test set.

## Methodology assessment
| Criterion | Rating | Assessment |
|---|---|---|
| Soundness | 3 | Careful statistics; W1 and W3 are the holes |
| Novelty | 3 | New diagnosis; method is a cited flat-field plus unchanged CDAN+E |
| Reproducibility | 4 | Data, code, splits, pre-registration promised |
| Experimental design | 3 | Good adaptation baselines; community baseline and standard view missing |
| Statistical rigor | 4 | Paired bootstrap, seeds, intervals throughout |
| Scalability | 3 | Reading is free; alignment needs a class-covering stream |

## Literature positioning
Checked: SITR (ICLR 2025) uses background subtraction, 18 calibration presses, 1M synthetic images, and reports 90.23 vs 73.22 for ViT-B; its four Minis differ by coating material, supporting the "more than additive" remark. Real Tactile MNIST is GelSight Mini data with the gel pad replaced halfway (two gels), stored at 320x240, so Fig. 1's orange points come from downsampled frames; its "three weeks" span and GelSlim's "every hundred grasps" I could not verify. An unreviewed web write-up on Mini gel wear already recommends re-estimating the reference from unlabelled target frames; not a citation requirement, but the idea is in the air. Shortcut and flat-field literature is cited appropriately.

## Questions for the authors
Q1. Alg. 1 with the session's bare-gel frame in place of F-hat, on campaign D: what does it give?
Q2. How is Fig. 3's reading gain on SITR (frozen DINOv2) consistent with "the reading does not help" (Sec. 5.4, CNN from scratch)? Give both numbers.
Q3. Define "state augmentation" (Fig. 3). Are the 20 SITR objects the released set or the paper's 16?
Q4. Fig. 2 shows a 4x4 marker grid in a 4.7-mm region, implying about 1.2 mm pitch; the Mini's is about 2 mm. Is the region size or the exclusion radius misstated?
Q5. How were frozen Sparsh encoders run on A to C without blank frames, given Sparsh's background subtraction?
Q6. Tactile MNIST: the "100%" collapse, single run and at which resolution?

## What would change my score
To 4: the bare-gel oracle row (Q1) and the standard whole-frame difference-image baseline, with W3 resolved and W4 reworded. To 5: in addition, evidence that a refreshed reference frame does not remove the shortcut, which would make the hardware-state framing necessary rather than convenient.

## Claims resting on the unseen supplement
Frozen Sparsh failing the same way; pre-registered predictions; SITR contrast factor 3.7; 85.0 percent spliced-change detection; CNN architecture; SDAT and MIC weight selection; HCT negative; SITR CNN tie setup. It must show how Sparsh was preprocessed, the pre-registration text with timestamps, and SITR reading numbers per setting.

## Coverage
Abstract: thorough; numbers match Tables 1 and 2. Sec. 1: thorough; 64, 35, 73.9, 37.0 checked; 20 vs 16 found. Sec. 2: thorough; SITR and Tactile MNIST verified. Sec. 3: thorough; band, exclusion radius, warping test, Fig. 2 geometry. Sec. 4: thorough; Alg. 1 line by line. Sec. 5: thorough; every number checked against Tables 1 to 3; row e typo. Sec. 6: thorough; limitations quoted. Sec. 7: light.

## Limitations of this review
No supplement, code or data seen. Tactile MNIST span, GelSlim's interval and the exact Mini marker pitch not verified beyond memory. No experiment rerun. Fig. 3 values in W3 are read off the plot.
