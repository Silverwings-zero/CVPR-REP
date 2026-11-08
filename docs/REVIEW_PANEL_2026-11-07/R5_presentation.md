# R5 - Presentation, figures and consistency
Primary skill: skills/manuscript_review_checklists.md (Figure Review Guide, figures/claims items). Also used: skills/deer_flow_academic_paper_review.md (Step 1.3 claims extraction), skills/paperjury_personas.md (vision-venue style, evidence-anchor rule), skills/neurips_form.md.
CVPR rating: 4/6. Confidence: 4/5.
NeurIPS-form sub-scores: Soundness 3/4, Presentation 3/4, Contribution 3/4, Overall 5/10.
One-line verdict: Plain, readable prose and numbers that mostly agree across sections, undercut by a figure that contradicts the text, an abstract claim that Table 3 contradicts, a sign error in Table 2, and a missing dataset table and qualitative figure.

## Summary
Every GelSight Mini image carries a hardware state (fine cartridge pattern plus smooth illumination). The paper measures it over four months, shows that one-object-per-session datasets make it a shortcut (own data, and injected ties in SITR), and reads test images relative to a causal, label-free running estimate (low-pass of running mean minus training mean, restarted by a change detector), optionally with CDAN+E alignment on the unlabelled stream. ResNet-50 on 55 cross-hardware sessions: 32.4 to 44.8 to 61.6.

## Claims table
| Claim | Stated | Evidenced | Match |
|---|---|---|---|
| State drifts slightly, changes abruptly | Abstract | Sec. 3 text (0.13/0.40 vs 3.8/2.9 grey levels), Fig. 2 | Yes, text only |
| One tied object halves SITR accuracy (73.9 to 37.0, 62% to tied class) | Abstract, Intro | Fig. 3 raw curve | Yes, read off plot |
| 32.4 to 44.8 (chance 5), matches best of five TTA, 61.6 with alignment, holds for CNN, ConvNeXt, DINOv2 | Abstract, Intro | Tables 1 to 3 | Yes |
| Improves every DA method it is added to | Abstract, Intro | Table 1 yes; Table 3 frame ResNet-50 42.2 to 32.8 | No |
| 0.34 / 0.29 / 0.04 / 0.08; 2,711 grasps | Intro | Fig. 1, Sec. 3 | Yes |
| 64% of fold-D episodes to four materials, 35% on sixteen | Intro | No table or figure | Not evidenced |
| Ten adaptation methods, four encoders | Contributions | Table 1 has eleven; Table 3 four | Count off |
| On SITR the reading does not help | Intro, Sec. 5.4 | Fig. 3 blue curve: reading lifts 37 to about 73 at k=1 | Contradiction |
| Alignment keeps three quarters of its gain on unseen sessions | Sec. 5.2, Limitations | Table 2 row i: 11.6/16.8 = 0.69 | Overstated |

## Figures and tables
Fig. 1 (page 1). Axes, units, values and bold lead present. The legend is two bare dots inside the data area at 0.52 and 0.45, between real points at 0.48 and 0.41, so at print size they read as data. Medians over 55 sessions, no spread. It is a measurement plot: a page-1 reader learns same-hardware correlation is about 0.3 and cross-hardware about 0.05, not that classifiers exploit this nor that the reading repairs it. Fix: three-panel teaser (state image, collapse, repair), legend outside the axes, interquartile bars.

Fig. 2 (page 4). Bottom row clear with a labelled colour bar. The top row at column width is four 18 mm panels of grey blobs; the caption's claim that the two S2 halves share a pattern cannot be checked by eye, no correlation is printed, no grey scale is given. Fix: print pairwise correlations under the panels.

Fig. 3 (page 4). Axes and bands defined. Legend: "raw", "Algorithm 1 (causal)", "state augmentation". Caption and text discuss only "raw"; "state augmentation" is defined nowhere; the blue curve stops at k=4 unexplained and contradicts the text (W1).

Table 1. Seeds, chance, interval and "top" defined. Bold undefined. No spread on the unit and gel columns although conclusions rest on the 8-session gel stratum.

Table 2. "all" undefined here; bold undefined and inconsistent (61.6 bold, row f 63.7); row e misprinted (W3); mixed minus typography; five-seed deltas disagree with three-seed means in the same row (explained, but confusing); rows f and g use different bases (d and e) with no reason.

Table 3. Column meaning silently switches between blocks ("+ reading" above, "+ alignment" below). No intervals.

Missing: a qualitative figure (read versus unread crop, a gel-stratum failure case), a dataset or protocol table (the 20 materials are never named; per-fold counts and the 47/8 strata must be reconstructed from prose), and a table for the sigma sweep. Algorithm 1 serves acceptably as the method figure.

## Terminology
"Hardware state", "configuration S1 to S3", "estimate", "alignment", "stream", "session", "episode", "campaign" and "recipe" are each introduced once and used consistently. "Hardware configuration" is used in the introduction before Sec. 3 defines it. "Signature" occurs only in labels, never in text. "Imprint" has two opposite meanings: the hardware's mark on the image ("the imprint of the hardware it was recorded on") and the object's mark on the gel ("Every object imprints the gel differently", "their average imprint"). "Touch", "contact", "grasp", "episode" and "crop" are interchanged; in Sec. 5.2 "several touches" could mean episodes or crops. "Tube" first appears in a table caption and is never motivated. "Grouped confidence" exists only in Table 2.

## Method followability
What is subtracted, at what scale, from what, when the estimate restarts and what alignment trains on are all stated. Gaps: grey or RGB and whether the state is per channel; which frames form the mean contact frame; the estimate at positions no crop covered; how the 190 to 370 um fine band relates to the sigma 33 px low-pass, since the band between them is neither; and why the aligner sees unread stream images while the test images it classifies are read (only row h, reading training images, is ablated).

## Weaknesses, ranked
W1 (Major). Fig. 3 contradicts the text. Intro: "SITR's Minis also differ in their field of view and contrast, and there the reading does not help, but alignment does." Sec. 5.4: "The reading cannot repair a tie between classes and Minis there". Fig. 3's "Algorithm 1 (causal)" curve shows the reading raising accuracy from about 37 to 73 at k=1 and 74 to 87 at k=0. Either frozen DINOv2 (figure) and the CNN from scratch (text) behave differently, which must be said, or one is wrong. Fix: discuss all three curves, define or drop "state augmentation", reconcile the SITR sentences.

W2 (Major). Abstract: "It also improves every domain adaptation method we add it to". In Table 3's frame view, CDAN+E on ResNet-50 falls from 42.2 to 32.8 with the reading, as Sec. 5.3 admits. Fix: scope the claim to contact crops.

W3 (Major, objective error). Table 2 row e: "$-$0.8 [$-$2.0, $-$-0.5] vs d$^5$" prints as "[−2.0, −-0.5]". The text gives d minus e as "(-0.5 to 2.0)", so e minus d must be [−2.0, 0.5]. As printed the interval excludes zero and contradicts "Its lead is within the interval". Fix sign and typography.

W4 (Major). "A ResNet-50 trained on June and July assigns 64\pct\ of these test episodes to the four materials, and it recognises the sixteen materials that it learned on unit 1 at only 35\pct". These fold-D numbers appear nowhere else; tables average over folds; no dataset table exists. Fix: a protocol table (per fold: training campaigns, test sessions, strata, materials) carrying these numbers.

W5 (Minor). Bold undefined and inconsistent: "d & a $+$ alignment $+$ reading (ours) & \textbf{61.6}" while row f shows 63.7.

W6 (Minor). "Imprint" with two meanings (quotes above). Reserve it for the object.

W7 (Minor). Prose-only results: "Narrower low-passes ($\sigma$ of 8 or 16\,px) lose ten to twenty points", shuffled-stream TTA (37.8, 48.2), SITR alignment (12.8 to 66.8), unit discriminator (100% vs 64%), and a pre-registration whose outcome is never reported: "we wrote down our predictions before training any model (supplement)".

W8 (Minor). "compared with ten adaptation methods" versus eleven in Table 1; "three quarters" versus 0.69; Table 3 column switch; Fig. 1 legend inside data; Fig. 2 top row unreadable.

## Sentences to fix
1. "On 55 sessions recorded on hardware on which their material was not learned, the reading alone raises a ResNet-50 from 32.4 to 44.8\pct\ (chance 5\pct), as much as the best of five test-time adaptation methods." Rewrite: "We test on 55 sessions whose material was learned on a different unit or gel. There the reading alone raises a ResNet-50 from 32.4 to 44.8% (chance 5%), matching the best of five test-time adaptation methods."
2. "It also improves every domain adaptation method we add it to, and with alignment the ResNet-50 reaches 61.6\pct." Rewrite: "On contact crops it also improves each of the five domain adaptation methods we add it to, and with alignment the ResNet-50 reaches 61.6%."
3. "The images alone show when the hardware changed, and they corrected our laboratory log." Rewrite: "The images alone date the changes of hardware, and in Sec. 3 they correct two entries of our laboratory log."
4. "SITR's Minis also differ in their field of view and contrast, and there the reading does not help, but alignment does." Rewrite, if true: "SITR's Minis also differ in field of view and contrast. There the reading repairs the injected tie for a frozen DINOv2 (Fig. 3) but not for a CNN trained from scratch (Sec. 5.4), while alignment repairs both."
5. "Every object imprints the gel differently, so averaging over objects leaves the hardware." Rewrite: "Each object deforms the gel differently, so averaging over many objects cancels the objects and leaves the hardware state."
6. "An estimate built from a few objects still contains their average imprint, and that imprint lies mostly in the fine band." Rewrite: "An estimate built from only a few objects still contains the average of their contact textures, and that residue lies mostly in the fine band."
7. "For the fold that holds out A and B, which trains on later campaigns and tests on earlier ones, we wrote down our predictions before training any model (supplement)." Rewrite: "For the fold that holds out A and B we pre-registered our predicted outcomes before training any model; X of Y predictions held (supplement)."
8. "On those sessions alignment still adds about twelve points to the reading alone, three quarters of what it adds when the stream contains them." Rewrite: "On those sessions alignment still adds 11.6 points to the reading alone, about seventy percent of the 16.8 points it adds when the stream contains them."
9. "The first adds a few points with an interval that includes zero (f), and the second costs about five (g), so the method includes neither." Rewrite: "The grouped objective (row f) adds 3.4 points with an interval that includes zero, and the per-touch objective (row g, tested on the DANN variant) costs about five, so the method includes neither."
10. Table 2 caption: "$\Delta$: paired difference to the row named, with its 95\pct\ interval, over the seeds both runs have (three, or five where marked $^5$)." Rewrite: "all: episode accuracy over the 55 sessions, mean of three seeds. Delta: paired difference to the row named with its 95% bootstrap interval, over three seeds or five where marked ^5. Bold: our method."

## Strengths of the presentation
Plain, read-aloud prose with no em-dashes, no semicolon chains and no gratuitous emphasis. Every caption has a bold lead and most are self-contained. Intervals are paired bootstraps and the text says when they include zero, including for the paper's own lead. The "top" column is a good, defined diagnostic. The discussion claims only what Tables 1 and 3 show, including the gel-stratum and frame-view losses. Negative ablations (f to h) sit in the main table. All 68 citations resolve.

## What would change my score
To 5: fix W1 to W4 (reconcile Fig. 3 with the text and define its curves, scope the abstract claim, correct the Table 2 interval, add a protocol and dataset table), add a qualitative read-versus-unread figure with a failure case, and report the pre-registration outcome. To 3: if the Fig. 3 blue curve is right and the SITR text is wrong, the "limits shown" framing of the reading needs rewriting.

## Coverage
Abstract: thorough. 1 Intro: thorough. 2 Related: light (style and claims only). 3 Hardware state: thorough. 4 Method: thorough. 5 Experiments: thorough, all table arithmetic re-added. 6 Discussion: thorough. 7 LLM statement and references: light (rendering and key resolution checked).

## Limitations of this review
The supplement was not provided, so every "(supplement)" claim is unverified rather than judged. Fig. 3 values were read off the plot. I did not assess novelty, baseline vintage or statistical validity beyond internal consistency, and I ran no code.
