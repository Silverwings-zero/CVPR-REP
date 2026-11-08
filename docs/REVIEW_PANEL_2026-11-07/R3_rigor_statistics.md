# R3 - Experimental rigor and statistics
Primary skill: skills/manuscript_review_checklists.md. Also used: skills/critique_framework.md (leakage, selective reporting, overclaiming), skills/00_README.md (correctness stage first), skills/paperjury_personas.md (evidence-anchor rule), skills/neurips_form.md (confidence scale).
CVPR rating: 4/6. Confidence: 4/5.
NeurIPS-form sub-scores: Soundness 3/4, Presentation 3/4, Contribution 3/4, Overall 5/10.
One-line verdict: Unusually careful statistical reporting for a vision paper, but the generalisation claim rests on two hardware transitions, the headline configuration was chosen after seeing the full test grid, and the text contradicts its own Fig. 3 on where the reading fails.

## Synopsis
The paper measures a sensor-specific additive component in GelSight Mini images over four months, shows that one-object-per-session datasets let a material classifier use it as a shortcut (on its own data and by injecting a class-to-sensor tie into SITR), and proposes a training-free "reading" (subtract the low-pass of a running mean of the robot's own contacts) plus CDAN+E alignment on the unlabelled stream. On 55 cross-hardware sessions a ResNet-50 goes from 32.4 to 44.8 (reading) and 61.6 (reading + alignment). Correctness pass: every Table 1 row reproduces its "all" value from the 47/8 strata within rounding, the intro's 73.9 to 37.0 and 62 % match Fig. 3, and the two citations I checked are accurate. The problems are a figure-versus-text contradiction, an image-derived ground truth, an effective sample of two hardware changes, and a headline chosen on test results. Major revision.

## Critical Issues
1. Sec. 1 and 5.4 vs Fig. 3. Quote (intro): "SITR's Minis also differ in their field of view and contrast, and there the reading does not help, but alignment does." Quote (5.4): "The reading cannot repair a tie between classes and Minis there". Fig. 3's curve "Algorithm 1 (causal)" lies far above "raw" at every k: at k=1 about 74 vs 37 % accuracy, with the top-class share falling from about 62 to 26 %. That is exactly the repair the text says cannot happen on SITR. Fig. 3 uses a frozen DINOv2 and Sec. 5.4 a CNN from scratch, so both may hold in their own setting, but the paper never says so, and a listed contribution is the method "with its limits shown". Fix: name the model and stream for each result, reconcile them in one paragraph, and reword the intro and Sec. 5.4 to what both support.

## Major Concerns
1. Sec. 3, image-derived timeline and circular detector evaluation. Quote: "Our laboratory log had recorded a change of unit and four gel replacements. The images show only the two changes above, and the operators confirmed them against their notes, so the timeline above is the one the images give." The hardware ground truth comes from the same fine-band and illumination statistics the Sec. 5.5 detector uses, so "9 episodes after the change of unit, 8 episodes after the gel replacement, and 2 false ones" is not an independent test. Worse, if three logged gel replacements left the pattern unchanged, that contradicts "The pattern does belong to the gel cartridge, because a new gel on the same unit and camera removes it." Fix: say what the operators concluded about the three other entries, give non-image evidence for the two retained changes, and evaluate the detector only on spliced streams where change points are known by construction.
2. Sec. 5 Protocol, effective sample size and spread. Quote: "with 95 % intervals from a bootstrap over sessions and paired bootstrap intervals for differences between methods." The 55 sessions sit in three folds and three configurations with two real transitions. Session errors are correlated within fold and configuration, so the session bootstrap overstates precision. Seed spread is never shown ("Table rows average three training seeds", no SD), and Table 2 mixes 3-seed means with 5-seed differences (row d: 61.6 minus 53.5 is 8.1, printed 7.3). Fix: per-fold accuracies for the main rows, per-seed SD, and a cluster bootstrap over folds or configurations.
3. Sec. 5.3, headline chosen on test. Quote: "On tube crops both parts of the method work for every encoder, which is why the paper's comparisons use that view." Seven encoder-view configurations are scored on the test sessions and the best one (61.6) becomes the abstract. In the frame view the reading "costs ResNet-50 nine points (-15.3 to -3.9)" and adds nothing to ConvNeXt-T, which reaches 60.4 with alignment alone. Fix: give the range over encoders in the abstract and state whether the tube view was fixed before any cross-hardware result was seen.
4. Table 1 and Sec. 5.1, missing intervals and the gel stratum. Quote: "the reading improves every one of them further, by three to ten points with intervals that exclude zero." Only the CDAN+E difference (7.3 [3.1, 11.7]) is printed; MCC gains 3.2 and FDA 2.8 with no interval. On the 8 gel sessions the reading lowers every aligner (CDAN+E 53.1 to 44.5, DANN 48.3 to 38.2, SDAT 41.1 to 36.4, MIC 28.2 to 24.2, MCC 39.0 to 29.8), which the Discussion admits but the abstract's "improves every domain adaptation method we add it to" does not. Fix: a paired-interval column for every "+ reading" row and a qualified abstract.
5. Table 2 row e, numbers disagree. Quote: "$-$0.8 [$-$2.0, $-$-0.5] vs d$^5$" prints an interval [-2.0, -0.5] excluding zero, while the text says "0.8 points above DANN with the reading (-0.5 to 2.0)", which includes zero. The conclusion "Its lead is within the interval" depends on which is right.
6. Sec. 4 and 5.2, hyperparameters and pre-registration. Quote: "Narrower low-passes (sigma of 8 or 16 px) lose ten to twenty points, and wider ones (66 or 132 px) lose up to two." The sweep is on test sessions; nothing says sigma=33, K=8, W=30, m=5 or the percentile thresholds were fixed beforehand. "we wrote down our predictions before training any model (supplement)" covers one fold and is uncheckable here. Fix: state what was fixed before the first cross-hardware run; print the fold-AB predictions and outcomes.
7. Fig. 3: the legend entry "state augmentation" is defined nowhere in the main text, and the Algorithm 1 curve stops at k=4 without explanation.

## Minor Concerns
- Intro: "assigns 64 % of these test episodes to the four materials" and "only 35 %" appear in no table; Table 1's "top" (33) is a different statistic.
- Contributions say "ten adaptation methods"; Table 1 lists eleven.
- Fig. 1: medians only, no spread or N; whether "other unit" and "other gel" pairs are same-object or other-object is unstated, so like-with-like against "same day, other object" is not guaranteed.
- Row i: "three quarters" is 11.6/16.8 = 0.69, on different session sets (28 vs 55).
- Sec. 5.4 SITR numbers carry no seeds or intervals.

## Checklist
| Item | Status | Evidence |
|---|---|---|
| Hypothesis stated | pass | "this paper asks what it does with it" |
| Controls | pass | SITR tie, row i, shuffled-stream TTA |
| Sample size justified | fail | two transitions, not named as the limiting N |
| Paired tests | pass | "paired bootstrap intervals" |
| Spread reported | partial | session CIs yes; seed SD and per-fold no |
| Double-dipping | fail | view and sigma judged on test; baselines tuned "on the first seed of this protocol" |
| Test data in selection | partial | stream holds test sessions; disclosed, controlled by row i |
| Figures match text | fail | Fig. 3 vs "does not help"; "state augmentation" undefined |
| Number consistency | fail | Table 2 row e vs text |
| Citations support claims | pass | SITR 90.23 vs 73.22, 18 presses, 1M sim; RDP "damage or gel replacements" verbatim |
| Limitations honest | pass | gel stratum, transductive stream |
| Data/code | stated | release promised |

## Strengths
Strata, the collapse metric and the losing gel stratum are printed. Negative results are kept (rows f, g, h; frame-view ResNet-50). Row i separates adaptation from memorisation. Table 1 is arithmetically consistent across all 17 rows.

## Questions for the authors
1. What did the operators conclude about the three logged gel replacements the images do not show?
2. Were the tube view and sigma=33 fixed before the first cross-hardware result?
3. Per fold: accuracy of rows a to d and session counts.
4. What is "state augmentation", and why does Algorithm 1 help a frozen DINOv2 on SITR when Sec. 5.4 says it cannot?
5. Which sign is right in Table 2 row e?

## What would change your score
Up to 5: resolve the Fig. 3 contradiction; per-fold and per-seed spread showing the gain is not concentrated in fold D; a fold-level bootstrap that still excludes zero for rows b and d; non-image evidence for the timeline. Down to 3: if the gain is mostly fold D, or the four logged gel replacements were real.

## Claims that rest on the unseen supplement
Sparsh "fail in the same way"; the fold-AB pre-registration; SDAT/MIC weight choice; SITR field of view and 3.7x contrast; the SITR tie setup; 85.0 % spliced detection; the CNN; the HCT negative. It must show the dated pre-registration text and the SITR protocol with seeds.

## Coverage
Abstract: thorough. 1 Intro: thorough. 2 Related: light (two citations verified). 3 Hardware state: thorough. 4 Method: thorough. 5 Experiments: thorough (tables recomputed). 6 Discussion: thorough. 7 LLM statement: light.

## Limitations of your review
No supplement, code or data; nothing rerun. Fig. 3 values read off the plot. Two citations verified (SITR Table 1 and Sec. 6.1; RDP Sec. IV-A), no others. I am a statistics reviewer, not a tactile-sensing specialist.
