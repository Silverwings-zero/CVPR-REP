# R4 - CVPR generalist / area-chair reader
Primary skill: skills/critique_framework.md (three-pass read, assessment criteria, calibration guide). Also used: skills/paperjury_personas.md (gatekeeper core, evidence-anchor rule, vision-venue style profile), skills/neurips_form.md (sub-scores, confidence), skills/00_README.md (objective-error stage first).
CVPR rating: 4/6. Confidence: 4/5.
NeurIPS-form sub-scores: Soundness 3/4, Presentation 3/4, Contribution 2/4, Overall 5/10.
One-line verdict: A careful, honest diagnosis of a sensor-state shortcut with a valuable dataset and fair baselines, but a thin method (flat-field plus unchanged CDAN+E), a headline configuration chosen where both parts work, and one sensor model, one task; borderline at CVPR, needs a champion.

## Pass 1 answers (written before the detailed read)
Main claim: VBTS images carry an additive hardware state that one-object-per-session datasets turn into a class shortcut; a training-free running-mean reading plus CDAN+E on the robot's unlabelled stream repairs much of the loss (32.4 to 44.8 to 61.6). Type: empirical diagnosis plus dataset, thin method. Scope: borderline CVPR. Abstract numbers to check against Tables 1 and 2. Novelty: fixed-pattern noise, flat-fielding and batch effects under a new name; value must come from the tactile instance, the causal control and the data.

## Summary
Over four months, two GelSight Mini units and three gels, a 190 to 370 micrometre band of every contact image correlates with other objects on the same hardware (0.29 to 0.34) but not across a unit or gel change (0.04 to 0.08), and a ResNet-50 trained with one session per material collapses onto the classes recorded on the new hardware. The collapse is reproduced by construction on SITR's four Minis (73.9 to 37.0 percent with one class tied to the test Mini). The remedy subtracts the low-pass of a causal running mean of the robot's own contacts, restarts at detected hardware changes, and feeds the stream to CDAN+E when retraining. On 55 cross-hardware sessions the reading alone matches RoTTA and improves every UDA method; CDAN+E with reading reaches 61.6 percent. Alignment also works on SITR, where the reading does not.

## Strengths
S1. Clean, cross-checked diagnosis: a four-month correlation ladder (Fig. 1), a second lab's unit, bare-gel frames, a causal injection on a public dataset (Fig. 3); "62\pct\ of the test images are assigned to it" is concrete evidence.
S2. Honest reporting: the stratum where the method hurts ("The reading lowers the accuracy on the eight sessions recorded on the other gel"), the frame view where it costs ResNet-50 nine points, the n.s. row it declined to include.
S3. Fair, plentiful baselines (five TTA, five UDA, FDA, four encoders) on one encoder and budget with paired bootstrap intervals; the TTA collapse on an object-ordered stream matters beyond tactile.
S4. Related work cites what a generalist expects (Zech, DeGrave, Torralba and Efros, Geirhos, RxRx1, BaSiC, PRNU, ComBat) and says plainly this is "the tactile instance" of shortcut learning. The dataset with force and bare-gel frames is a genuine contribution.

## Weaknesses (ranked)
W1 (Major, contribution identity). Sec. 4: "We use CDAN+E~\cite{long2018cdan} unchanged." and "This is the retrospective flat-field of microscopy~\cite{singh2014illumination,peng2017basic}, taken from a robot's stream." A per-pixel running mean, a Gaussian low-pass and an off-the-shelf aligner is fine for an analysis paper, but the title ("Factoring ... out of Its Images"), the abstract ("We read touch together with the sensor's state") and the third contribution bullet present a method paper. Fix: lead with measurement, shortcut evidence and dataset; present the reading as the baseline the diagnosis implies and alignment as recommended practice.

W2 (Major, headline chosen where the method works). Sec. 5.3: "On tube crops both parts of the method work for every encoder, which is why the paper's comparisons use that view." Table 3: frame-view ConvNeXt-T with alignment alone reaches 59.2, within a point of the 61.6 headline with the reading adding nothing, and the reading costs frame-view ResNet-50 nine points (42.2 to 32.8). The training-free reading the abstract leads with helps in a view selected because it helps. Fix: report frame-view ConvNeXt-T as co-headline, explain why the reading hurts a ResNet-50 that already sees the illumination field, and state the view choice as a limitation.

W3 (Major, transductive headline without an inductive absolute). Sec. 5: "The stream therefore contains the test sessions themselves, as a working robot's stream would", and Table 2 row i prints "--" for accuracy with only "11.6 [5.1, 18.6] vs b". The 61.6 in abstract, intro and three tables is transductive; the inductive accuracy a reader would call generalisation is never given. Fix: report rows b and d on the 28 later sessions in absolute terms, next to 61.6 in the abstract. Also 11.6/16.8 is 0.69, nearer two thirds than "three quarters".

W4 (Major, unit and gel confounded in the main stratum). Sec. 3: "Unit 1 carried gel 1 until July 9, unit 2 carried gel 2 from July 10", yet Sec. 5 says of 47 sessions "the material was learned on the other unit" and Table 1 heads the column "unit (47)". Every such session crossed a unit and a gel at once. The fine pattern's attribution to the gel rests on the S2 to S3 swap, but "the smooth field belongs mostly to the unit" is not separable in this design. Fix: relabel the stratum "unit and gel", soften the attribution, check on SITR's four Minis.

W5 (Minor, generality). Sec. 6: "We followed two units and three gels of one sensor model." One task, one sensor model, and on DIGIT data "our method does not help there (supplement)". The abstract's "read touch together with the sensor's state" is wider than shown. The co-recorded Nano17 force makes a force-regression column a cheap second task.

W6 (Minor, mechanical). Table 2 row e prints "$-$0.8 [$-$2.0, $-$-0.5]", a double minus (text: -0.5 to 2.0). WILDS/Camelyon17 and Noiseprint, the canonical CV references for scanner and camera fingerprints, are missing. Fig. 1 is a dot plot, weak as a page-1 teaser; Fig. 2's illumination fields beside Fig. 3's collapse curves would be stronger.

## Fit and significance for CVPR
A vision paper in method, a robotics paper in subject. It reaches CVPR readers through shortcut learning, batch effects and sensor fingerprints, and admits the phenomenon is known elsewhere. The idea is medium-sized: a well-controlled, long-horizon demonstration that a camera-based sensor's fixed pattern becomes a class feature when classes are tied to sessions, plus the observation that microscopy's retrospective flat-field transfers directly. The four-month measurement and the SITR control are convincing; the Tactile MNIST tie ("sends 100\pct\ of the test touches to that digit") is one number without a figure. The value is not in the method, and a method-hungry panel will notice. The page budget is spent reasonably, though Sec. 3 repeats the intro's numbers. Writing is plain and clear, more narrative than CVPR norm, readable rather than a fault. With a tactile-expert champion this is an acceptable analysis-and-dataset paper; without one, two generalists score it 3 and it falls below the line.

## Questions for the authors
Q1. Absolute accuracy of rows b and d in the inductive setting of Table 2 row i?
Q2. Why does the reading hurt ResNet-50 on frame crops but help DINOv2-B?
Q3. Any data where only the unit changed, to attribute the illumination field?
Q4. A public, unreviewed GelSight Mini gel-wear study reports that re-estimating a per-pixel reference on the target gel recovers most of a swap's loss. How does the reading differ beyond causality and the low-pass?
Q5. How many of the 55 test sessions are first after a change, where the reading is identity by construction?

## What would change my score
To 5: an inductive absolute headline that holds up, frame-view ConvNeXt-T as co-headline with the ResNet-50 loss explained, the stratum relabelled, a force task from existing data. To 3: a supplement whose pre-registration or SITR protocol is weaker than described, or an inductive accuracy far below 61.6.

## Claims that rest on the unseen supplement
Pre-registered predictions for the A and B fold; frozen Sparsh failing the same way; SDAT and MIC weight selection; SITR contrast factor 3.7 and its tied-class protocol; CNN architecture; the HCT negative; 85.0 percent detection on spliced streams. It must show a dated pre-registration, the Sparsh confusions, and a SITR split without test-object leakage.

## Coverage
Abstract: thorough ("improves every domain adaptation method we add it to", checked against Table 1). 1 Intro: thorough ("we create the same situation in a dataset that does not have it"). 2 Related: thorough ("Our estimate is such a flat-field"). 3 Hardware state: thorough ("The pattern does belong to the gel cartridge"). 4 Method: thorough ("subtracts only the smooth part"). 5 Experiments, all tables, 5.1 to 5.5: thorough ("Running RoTTA on read images adds nothing"). 6 Discussion: thorough ("repairs the shortcut, not every effect of new hardware"). 7 LLM statement: light ("computed by a script from logged outputs").

## Draft meta-review
The paper shows, over four months and with a causal control on a public dataset, that GelSight Mini images carry a hardware-specific additive pattern that one-session-per-object datasets turn into a class shortcut, and releases the data. Reviewers agree the diagnosis is sound, honestly reported and fairly baselined. Concerns: a thin method (running-mean flat-field plus unchanged CDAN+E), a headline configuration chosen where both parts help while a frame-view ConvNeXt matches it with alignment alone, no inductive absolute, a unit-plus-gel stratum labelled as a unit change, one sensor model and one task. None is fatal. The decision turns on whether CVPR wants an analysis-and-dataset paper on a niche sensor; the committee leaned borderline.

## Limitations of my review
Not a tactile specialist; the GelSight optics and the 190 to 370 micrometre band are unverified. Supplement and code unseen. Five web queries (PRNU and Noiseprint; GelSight gel-swap prior work; Camelyon17 and WILDS; SITR; Tactile MNIST) confirmed the paper's descriptions of SITR and Tactile MNIST and found no peer-reviewed prior measurement of a classifier's loss after a gel swap.
