# R6 - Reproducibility, resources, and the open-minded reader
Primary skill: skills/neurips_form.md. Also used: skills/manuscript_review_checklists.md (Reproducibility, Literature, Figure guide), skills/paperjury_personas.md (evidence-anchor rule, cross-section mandate), skills/00_README.md (objective-error pass first).
CVPR rating: 4/6. Confidence: 4/5.
NeurIPS-form sub-scores: Soundness 3/4, Presentation 3/4, Contribution 3/4, Overall 5/10.
One-line verdict: A genuinely new and carefully measured finding (a VBTS's unit and gel leave a stable imprint that session-tied datasets turn into a class shortcut) with a modest but honest method and a valuable dataset; held back by a transductive headline without its inductive counterpart, a figure that contradicts the text, and main-text reproducibility gaps that the unseen supplement and release must close.

## THOUGHT
I read as the reviewer who asks first what is new. The new thing is not the low-pass subtraction or CDAN+E, which the authors themselves trace to microscopy flat-fields and Long et al. It is the diagnosis: a GelSight Mini's cartridge pattern and illumination field persist for months (0.34 and 0.29 correlation), vanish with a unit or gel swap (0.04, 0.08), and because tactile datasets record one object per session, a classifier learns "this imprint means one of these materials". The controlled injection on SITR (tie k classes to the test Mini, watch 62 percent of test images go to the tied class) is the cleanest piece of evidence in the paper and is replicated on Tactile MNIST. The observation that the images dated the hardware changes better than the lab log is a small gem. I searched the web and alphaXiv for prior work proposing a running flat-field from a robot's own contacts or the session-tied shortcut; I found the cited cross-sensor work (SITR, TacVerse, BIDETA, HTT, Castano-Amoros) but nothing proposing either. SITR's numbers quoted in related work match its arXiv text.

Objective-error pass: abstract numbers match Tables 1 to 3; 73.9 to 37.0 is indeed a halving; per-method deltas in the text match the tables; 35 June/July sessions plus 20 in D equals the 55 test sessions, so the protocol is internally consistent. Two cross-section problems: Fig. 3 plots an "Algorithm 1 (causal)" curve that roughly doubles accuracy on SITR, while the intro and Sec. 5.4 say the reading does not help on SITR; and Fig. 3 has a third curve, "state augmentation", defined nowhere. The headline 61.6 uses a stream that contains the test sessions; the inductive row i gives only a delta and a dash for the absolute.

Reproducibility: the main text omits optimizer, schedule, batch size, iterations, how the contact is located for the 192-px crops, which eight frames, how contact frames are detected, and the 20 material names. These are release-fixable, but a CVPR reader must be able to find them. Statistics are good: bootstrap over sessions, paired intervals, 3 to 5 seeds, a pre-registered fold, a collapse metric. Baseline selection on the first seed of the test protocol is disclosed and favours baselines, but the choice of sigma is shown only as a test-set sweep.

Desired outcome: accept after the authors reconcile Fig. 3 with the text, give the inductive absolute, and move the minimum training and data details into the main text or a clearly promised supplement.

## REVIEW JSON
```json
{
  "Summary": "The paper shows that images from a GelSight Mini carry a stable hardware state (a fine cartridge pattern plus a smooth illumination field) that persists for months and changes abruptly when the unit or gel is replaced. Because tactile datasets record one object per session, material classifiers learn this state as a class cue and collapse after a hardware change; the authors demonstrate this on their own four-month, two-unit, three-gel recordings and by a controlled class-to-sensor tie on the public SITR data. They propose a training-free reading that subtracts the low-pass part of a running mean of the robot's own contacts (restarted by a two-window change detector), optionally combined with CDAN+E alignment on the same unlabelled stream. On 55 cross-hardware sessions the reading lifts a ResNet-50 from 32.4 to 44.8 percent and the full method reaches 61.6 percent; the reading improves every UDA method tried. A 31 GB dataset with force, kinematics, bare-gel frames, splits and code is promised under CC BY 4.0 / MIT.",
  "Strengths": [
    "A new and well-measured phenomenon: the hardware imprint is quantified across four months and replicated on another lab's Mini (Fig. 1), and its two components are localised (Fig. 2, warping test, gel-swap test).",
    "The shortcut is established causally by injecting a class-to-sensor tie into SITR and Tactile MNIST (Fig. 3), not only by correlation on the authors' own data.",
    "Honest evaluation practice: bootstrap over sessions, paired intervals, 3 to 5 seeds, a pre-registered fold, a collapse metric (top), negative ablations kept (rows f, g, h), and the stratum where the method hurts (8 gel sessions) is reported and explained.",
    "Broad comparison: five TTA and five UDA methods plus FDA, four encoders, two crop views, and an inductive check (row i).",
    "The dataset promise (two units, three gels, co-recorded Nano17 force, bare-gel frames, cross-hardware protocol) would be the first public VBTS set that spans hardware changes over months.",
    "Plain, readable prose; the contribution list matches what the experiments show."
  ],
  "Weaknesses": [
    "Headline is transductive and the inductive absolute is withheld. 'The stream therefore contains the test sessions themselves, as a working robot's stream would, and row~i of \\cref{tab:ablation} scores sessions that the stream never contained.' Row i reads 'i & d, stream of the first half only$^\\dagger$ & -- & 11.6 [5.1, 18.6] vs b$^\\dagger$'. The reader cannot know what accuracy the full method reaches on sessions it never saw; the abstract's 61.6 should be accompanied by that number.",
    "Figure 3 contradicts the text and contains an undefined method. The intro states 'SITR's Minis also differ in their field of view and contrast, and there the reading does not help, but alignment does', and Sec. 5.4 states 'The reading cannot repair a tie between classes and Minis there'. Yet Fig. 3's 'Algorithm 1 (causal)' curve lifts accuracy from about 37 to 75 percent at k=1 on SITR. A third curve, 'state augmentation', appears in the legend and is described nowhere; the caption only says 'We train on $20-k$ classes recorded on one Mini and $k$ classes recorded only on the test Mini'.",
    "Training recipe is not reproducible from the main text. 'fine-tuned at the native \\SI{11.3}{\\micro\\metre}/px on 192-px crops along the contact, with flips, quarter turns and photometric jitter.' No optimizer, learning rate, schedule, batch size, iterations, jitter ranges, how the contact is located for crop placement, which eight stroke frames, how 'contact' frames are detected ('no eight-frame span of contact inside the stroke'), nor GPU budget.",
    "The 20 classes of the released dataset are never named. 'The object is a box whose face carries one of 20 materials.' A dataset contribution needs its label set, the per-material maximum forces, the material pieces' provenance and the gripper/robot models in the paper or a referenced supplement table.",
    "Low-pass scale is justified by a test-set sweep only. 'Narrower low-passes ($\\sigma$ of 8 or 16\\,px) lose ten to twenty points, and wider ones (66 or 132\\,px) lose up to two.' No validation split is described, so it is unclear whether sigma=33 px was fixed before seeing cross-hardware results (the pre-registration might cover this but is in the supplement).",
    "Detector generality rests on one number. 'We also splice recordings of one unit or gel onto another in four datasets (ours, SITR, FEATS and Tactile MNIST) ... On the streams spliced from our recordings it detects 85.0\\pct\\ of the changes (supplement).' Detection rates for the other three datasets and the false-alarm rate are absent from the main text.",
    "Minor: SITR is described both as a '16-object classifier' (related work, matching SITR's paper) and as 'SITR~\\cite{sitr2025} pressed the same 20 objects' (Sec. 3); state which release subset is used. Row e prints '$-$-0.5', a typo. The RDP quote 'damage or gel replacements' could not be verified from the public abstract."
  ],
  "Originality": 3,
  "Quality": 3,
  "Clarity": 3,
  "Significance": 3,
  "Questions": [
    "What is the absolute accuracy of the full method on the 28 inductive sessions of row i, and of the recipe and reading-only on the same 28?",
    "Does Fig. 3's Algorithm 1 curve use the same reading as Sec. 5.4? If the reading helps a frozen DINOv2 on SITR but not a CNN trained from scratch, say so and reconcile the intro sentence. What is 'state augmentation'?",
    "Was sigma=33 px, K=8, W=30, m=5 fixed before the cross-hardware evaluation (in the pre-registration file)?",
    "In the fold holding out D, is the CDAN+E target domain the whole D stream (S2 and S3 together) with a single domain label, or two targets?",
    "Please report per-fold accuracies (D, C, A+B) for recipe, reading, and full method; the 55-session pooled mean mixes three training sets.",
    "How is 'contact' detected for the eight-frame span, and how are 192-px crops positioned along the contact at test time without labels?",
    "Which 20 materials, and from which release of SITR are the 20 objects taken?"
  ],
  "Limitations": [
    "The Limitations paragraph is candid: one sensor model, no same-hardware months-apart material recordings, lubricant confounded with drift, transductive stream, the method needs session boundaries. Add: the headline absolute is transductive, the gel-change stratum is 8 sessions of 4 materials, and the detector's cross-dataset rates are in the supplement."
  ],
  "Ethical Concerns": false,
  "Soundness": 3,
  "Presentation": 3,
  "Contribution": 3,
  "Overall": 5,
  "Confidence": 4,
  "Decision": "Accept"
}
```

## Reflection round (1/1)
Checking accuracy: every quoted anchor is verbatim from paper.tex; the Fig. 3 reading is from the full-size PNG (blue curve about 86, 75, 68, 53 at k = 0, 1, 2, 4). I verified SITR's 90 vs 73 and 18 presses against its arXiv text, so the related-work claim stands. The sigma weakness should be softened slightly: the paper gives a physical reason for the scale (fine band versus illumination), so the issue is undocumented selection, not evident tuning. I also want the Questions to ask for compute, since the form's "resources" criterion needs it. Scores stay: the problems are fixable in a revision, the finding and dataset are real. Refined JSON follows.

```json
{
  "Summary": "The paper shows that images from a GelSight Mini carry a stable hardware state (a fine cartridge pattern plus a smooth illumination field) that persists for months and changes abruptly when the unit or gel is replaced. Because tactile datasets record one object per session, material classifiers learn this state as a class cue and collapse after a hardware change; the authors demonstrate this on their own four-month, two-unit, three-gel recordings and by a controlled class-to-sensor tie on the public SITR data. They propose a training-free reading that subtracts the low-pass part of a running mean of the robot's own contacts (restarted by a two-window change detector), optionally combined with CDAN+E alignment on the same unlabelled stream. On 55 cross-hardware sessions the reading lifts a ResNet-50 from 32.4 to 44.8 percent and the full method reaches 61.6 percent; the reading improves every UDA method tried. A 31 GB dataset with force, kinematics, bare-gel frames, splits and code is promised under CC BY 4.0 / MIT.",
  "Strengths": [
    "A new and well-measured phenomenon: the hardware imprint is quantified across four months and replicated on another lab's Mini (Fig. 1), and its two components are localised (Fig. 2, warping test, gel-swap test).",
    "The shortcut is established causally by injecting a class-to-sensor tie into SITR and Tactile MNIST (Fig. 3), not only by correlation on the authors' own data.",
    "Honest evaluation practice: bootstrap over sessions, paired intervals, 3 to 5 seeds, a pre-registered fold, a collapse metric (top), negative ablations kept (rows f, g, h), and the stratum where the method hurts (8 gel sessions) is reported and explained.",
    "Broad comparison: five TTA and five UDA methods plus FDA, four encoders, two crop views, and an inductive check (row i).",
    "The dataset promise (two units, three gels, co-recorded Nano17 force, bare-gel frames, cross-hardware protocol) would be the first public VBTS set that spans hardware changes over months.",
    "Plain, readable prose; the contribution list matches what the experiments show."
  ],
  "Weaknesses": [
    "Headline is transductive and the inductive absolute is withheld. 'The stream therefore contains the test sessions themselves, as a working robot's stream would, and row~i of \\cref{tab:ablation} scores sessions that the stream never contained.' Row i reads 'i & d, stream of the first half only$^\\dagger$ & -- & 11.6 [5.1, 18.6] vs b$^\\dagger$'. The reader cannot know what accuracy the full method reaches on sessions it never saw; the abstract's 61.6 should be accompanied by that number.",
    "Figure 3 contradicts the text and contains an undefined method. The intro states 'SITR's Minis also differ in their field of view and contrast, and there the reading does not help, but alignment does', and Sec. 5.4 states 'The reading cannot repair a tie between classes and Minis there'. Yet Fig. 3's 'Algorithm 1 (causal)' curve lifts accuracy from about 37 to 75 percent at k=1 on SITR. A third curve, 'state augmentation', appears in the legend and is described nowhere; the caption only says 'We train on $20-k$ classes recorded on one Mini and $k$ classes recorded only on the test Mini'.",
    "Training recipe is not reproducible from the main text. 'fine-tuned at the native \\SI{11.3}{\\micro\\metre}/px on 192-px crops along the contact, with flips, quarter turns and photometric jitter.' No optimizer, learning rate, schedule, batch size, iterations, jitter ranges, how the contact is located for crop placement, which eight stroke frames, how 'contact' frames are detected ('no eight-frame span of contact inside the stroke'), nor GPU budget.",
    "The 20 classes of the released dataset are never named. 'The object is a box whose face carries one of 20 materials.' A dataset contribution needs its label set, the per-material maximum forces, the material pieces' provenance and the gripper/robot models in the paper or a referenced supplement table.",
    "Selection of the low-pass scale is undocumented. 'Narrower low-passes ($\\sigma$ of 8 or 16\\,px) lose ten to twenty points, and wider ones (66 or 132\\,px) lose up to two.' The physical reasoning for sigma=33 px is given, but no validation split is described and the sweep is on the test sessions; state whether sigma and the detector constants were fixed before the cross-hardware evaluation.",
    "Detector generality rests on one number. 'We also splice recordings of one unit or gel onto another in four datasets (ours, SITR, FEATS and Tactile MNIST) ... On the streams spliced from our recordings it detects 85.0\\pct\\ of the changes (supplement).' Detection rates for the other three datasets and the false-alarm rate are absent from the main text.",
    "Minor: SITR is described both as a '16-object classifier' (related work, matching SITR's paper) and as 'SITR~\\cite{sitr2025} pressed the same 20 objects' (Sec. 3); state which release subset is used. Row e prints '$-$-0.5', a typo. The RDP quote 'damage or gel replacements' could not be verified from the public abstract."
  ],
  "Originality": 3,
  "Quality": 3,
  "Clarity": 3,
  "Significance": 3,
  "Questions": [
    "What is the absolute accuracy of the full method on the 28 inductive sessions of row i, and of the recipe and reading-only on the same 28?",
    "Does Fig. 3's Algorithm 1 curve use the same reading as Sec. 5.4? If the reading helps a frozen DINOv2 on SITR but not a CNN trained from scratch, say so and reconcile the intro sentence. What is 'state augmentation'?",
    "Were sigma=33 px, K=8, W=30, m=5 fixed before the cross-hardware evaluation (in the pre-registration file)?",
    "In the fold holding out D, is the CDAN+E target domain the whole D stream (S2 and S3 together) with a single domain label, or two targets?",
    "Please report per-fold accuracies (D, C, A+B) for recipe, reading, and full method; the 55-session pooled mean mixes three training sets.",
    "How is 'contact' detected for the eight-frame span, and how are 192-px crops positioned along the contact at test time without labels?",
    "Which 20 materials, and from which release of SITR are the 20 objects taken?",
    "What GPU, wall-clock and iteration budget does one ResNet-50 seed take, and what is the per-frame cost of the reading and the detector?"
  ],
  "Limitations": [
    "The Limitations paragraph is candid: one sensor model, no same-hardware months-apart material recordings, lubricant confounded with drift, transductive stream, the method needs session boundaries. Add: the headline absolute is transductive, the gel-change stratum is 8 sessions of 4 materials, and the detector's cross-dataset rates are in the supplement."
  ],
  "Ethical Concerns": false,
  "Soundness": 3,
  "Presentation": 3,
  "Contribution": 3,
  "Overall": 5,
  "Confidence": 4,
  "Decision": "Accept"
}
```

## What is genuinely new here
The novel idea is to treat a vision-based tactile sensor as having a latent hardware state that is additive in the image, persistent for months, and replaced wholesale with the unit or gel, and to show that one-object-per-session collection turns this state into a class shortcut. The injection experiment on SITR is the paper's best contribution because it isolates the mechanism on data the authors did not collect. The flat-field reading and CDAN+E are known tools, which the paper says plainly; the novelty of the method is only the choice of source (the robot's own contacts, causal, restarted by a detector). The contribution list states the finding first, which is right, but the title and abstract foreground the method, and the finding's strongest evidence (Fig. 3) is damaged by the undefined third curve and the mismatch with Sec. 5.4. Reordering so that Fig. 3 is fully explained where it is first cited would make the idea land.

## Reproducibility checklist
- Hardware and acquisition: stated (GelSight Mini, 1640x1232, 11.3 um/px, 18.8 fps, Nano17, fixed exposure/white balance; Sec. 3 Data). Gripper, robot, gel variant, lubricant: not stated.
- Classes and label set: not stated (20 materials unnamed).
- Splits and protocol: folds stated; exact 55-session list promised in release.
- Stream formation: stated (all contacts on test hardware in test campaign, recording order, true change dates); whether S2 and S3 form one or two target domains: not stated.
- Detector: K, W, m, band, marker exclusion, percentile thresholds and their source: stated (Sec. 4).
- Low-pass: sigma=33 px stated; selection procedure not stated.
- CDAN+E: random projection 1024 above 4096, entropy weighting, GRL warm-start, weight one, TLL implementation: stated. Discriminator architecture, learning rate: not stated.
- Crops: 192 px, three positions, eight frames, 24-crop mean log-probability: stated; contact localisation and frame choice: not stated.
- Optimisation: optimizer, LR, schedule, batch size, iterations, jitter: not stated.
- Seeds: 3 or 5, stated. Statistics: bootstrap over sessions, paired: stated.
- Compute: "one small GPU" for the CNN only; otherwise not stated.
- Code and data: promised (MIT, CC BY 4.0), not yet available.

## What the released dataset needs to be adopted
Name the 20 materials with photos and per-material force caps; publish the session table (date, unit, gel, material, episode count) so the hardware timeline in Sec. 3 can be re-derived; include the contact-frame detector and crop extraction code so "tube crops" are reproducible; ship an evaluation script that outputs the 55-session mean, bootstrap interval and the collapse metric; give the baseline numbers of Table 1 as a leaderboard; document camera-to-Nano17 time alignment and the bare-gel frame protocol; state the gel type and lubricant. A held-out fourth configuration recorded later would make it a proper benchmark, but the paper's own constraint of no new collection is understood.

## Weaknesses ranked by severity
1. Sec. 5 Training and the stream; Table 2 row i. "The stream therefore contains the test sessions themselves". The headline is transductive and the inductive absolute is a dash. Fix: report the 28-session absolutes for a, b and d.
2. Intro and Sec. 5.4 vs Fig. 3. "there the reading does not help, but alignment does" against a plotted Algorithm 1 curve that doubles accuracy; "state augmentation" undefined. Fix: define all curves in the caption, reconcile or qualify the sentence (frozen DINOv2 versus CNN from scratch).
3. Sec. 5 Training. "fine-tuned at the native 11.3 um/px on 192-px crops along the contact, with flips, quarter turns and photometric jitter." No optimisation details. Fix: a recipe table in the supplement and a forward reference.
4. Sec. 3 Data. "The object is a box whose face carries one of 20 materials." Fix: a material table.
5. Sec. 5.2. Sigma sweep on test sessions without a stated selection rule. Fix: state when sigma was fixed; cite the pre-registration.
6. Sec. 5.5. Only the in-house splice rate is given. Fix: a four-dataset detection and false-alarm table.
7. Minor: 16 vs 20 SITR objects; "$-$-0.5"; unverified RDP quote.

## Questions for the authors
See the JSON Questions list (eight items). The two that could move my score are the inductive absolute and the Fig. 3 reconciliation.

## What would change your score
To 5/6: the inductive absolute for the full method is within a few points of 61.6, Fig. 3 is reconciled with Sec. 5.4 and every curve defined, and a recipe table plus material table are added (supplement acceptable if referenced). To 3/6: the inductive absolute collapses toward the reading-only 44.8, or the Fig. 3 Algorithm 1 curve turns out to be a different procedure from Algorithm 1.

## Claims that rest on the unseen supplement and what it must contain
- "Frozen Sparsh encoders fail in the same way (supplement)": Sparsh variants, probe, and the same 55-session numbers with the collapse metric.
- "we wrote down our predictions before training any model (supplement)": the pre-registration text with a timestamp and the predicted versus observed numbers.
- "SDAT and MIC keep the better of two trade-off weights ... (supplement)": both weights and both results.
- "The CNN, trained from scratch, has four stages (supplement)": architecture table and parameter count.
- "differ in contrast by a factor of 3.7 (supplement)" and the SITR four-class tie with a CNN: the contrast measurement and the full table including the 5-class stream.
- "detects 85.0% of the changes (supplement)": detection and false-alarm rates on all four spliced datasets, with the splice construction.
- "our method does not help there (supplement)" on HCT: the numbers.
- The 55-session list and the per-fold results.

## Coverage
- Abstract: thorough; numbers match the tables.
- 1 Introduction: thorough; the 64 percent and 35 percent fold-D numbers are not in any table.
- 2 Related work: thorough; SITR numbers verified against its arXiv text; RDP quote unverified; recent 2026 work covered.
- 3 Hardware state: thorough; measurement and timeline consistent (35 plus 20 sessions).
- 4 Method: thorough; detector and CDAN+E settings stated; crop and training details missing.
- 5 Experiments: thorough; all table-to-text deltas checked; row i dash and Fig. 3 issues found.
- 6 Discussion: thorough; limitations candid; "Every GelSight Mini dataset we examined" relies partly on the supplement.
- 7 LLM statement: thorough; mandatory under CVPR 2027, placed correctly, candid about implementation, experimental-detail choice, figures and drafting. Adequate. Suggest adding whether LLMs assisted literature search, since several "we found no study" claims depend on it, and note that the source marks the statement as a draft to be confirmed.
- Ethics and dual use: none found. Robot-only data, no humans; the PRNU-like fingerprint could identify a sensor unit, which carries no realistic harm.

## Limitations of your review
I did not run any code or see the supplement, splits, or release; the training recipe and all supplement-backed claims are judged on what the main text says. My literature check used five web queries and one alphaXiv search; I may have missed a tactile paper that proposes stream-estimated flat-fields. I read Fig. 3 values from the plot. I did not independently verify the Tactile MNIST three-week span or the RDP quotation.
