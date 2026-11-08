# AC meta-review: Stateful Touch (simulated CVPR 2027 panel, six reviewers)

Quotes checked against paper.tex. AC facts: Fig. 3 (fig_confound.pdf) is stale, its extra curves predating the current experiments; Table 2 row e is a LaTeX sign-doubling bug, correct value -0.8 [-2.0, 0.5].

## 1. Score table

| Rev | Angle | CVPR | Conf | S/P/C/O | One-line verdict |
|---|---|---|---|---|---|
| R1 | Tactile | 3/6 | 4/5 | 3/3/2/4 | A careful, honest diagnosis of a hardware shortcut in GelSight material recognition, but the fix is never compared with the field's default remedy (a reference frame of the bare gel), the headline is reported in a non-standard input view where it happens to work, and one SITR claim contradicts the paper's own figure. |
| R2 | DA / TTA | 3/6 | 4/5 | 2/3/3/4 | A careful and valuable diagnosis of a hardware shortcut in GelSight data, paired with a method whose evaluation has a transductive headline, a mis-specified AdaBN baseline, a low-pass width apparently chosen on the test sessions, and a figure that contradicts the text on SITR. |
| R3 | Statistics | 4/6 | 4/5 | 3/3/3/5 | Unusually careful statistical reporting for a vision paper, but the generalisation claim rests on two hardware transitions, the headline configuration was chosen after seeing the full test grid, and the text contradicts its own Fig. 3 on where the reading fails. |
| R4 | Generalist | 4/6 | 4/5 | 3/3/2/5 | A careful, honest diagnosis of a sensor-state shortcut with a valuable dataset and fair baselines, but a thin method (flat-field plus unchanged CDAN+E), a headline configuration chosen where both parts work, and one sensor model, one task; borderline at CVPR, needs a champion. |
| R5 | Presentation | 4/6 | 4/5 | 3/3/3/5 | Plain, readable prose and numbers that mostly agree across sections, undercut by a figure that contradicts the text, an abstract claim that Table 3 contradicts, a sign error in Table 2, and a missing dataset table and qualitative figure. |
| R6 | Reproducibility | 4/6 | 4/5 | 3/3/3/5 | A genuinely new and carefully measured finding with a modest but honest method and a valuable dataset; held back by a transductive headline without its inductive counterpart, a figure that contradicts the text, and main-text reproducibility gaps that the unseen supplement and release must close. |

## 2. Meta-review

The panel agrees on the contribution. Over four months, two GelSight Mini units and three gels, the paper measures an additive hardware state in every contact image, shows that one-object-per-session collection turns it into a class shortcut, and confirms the mechanism by injecting a class-to-sensor tie into SITR. All six call the measurement new, the SITR control the cleanest evidence, the reporting unusually honest, and the dataset the most durable artefact. The remedy, a stream flat-field plus unchanged CDAN+E, is modest and the paper says so.

The panel also agrees on what is wrong. The 61.6 headline is transductive and the inductive absolute is a dash in Table 2 row i. The comparison view was chosen after seeing that both parts work there, while a frame-view ConvNeXt-T nearly matches the headline by alignment alone. The abstract's "improves every domain adaptation method" fails on the gel stratum and in the frame view. The low-pass width is justified only by a sweep on test sessions, and spread per fold and per seed is absent. The two objective errors, Fig. 3 against the SITR text and the row e interval, are a stale figure and a template bug that cost credibility rather than substance.

Reviewers disagree on two things. R4 and R1 read an analysis and dataset paper under a method paper's title and score Contribution 2, while R6, R3 and R5 take the diagnosis as the contribution and score 3. R1 and R4 count the test-time adaptation collapse as a finding, while R2 holds that the AdaBN row is not AdaBN as published, that the four collapsed rows are one BatchNorm failure counted four times, and that NOTE and LAME are the fair comparators. R1 and R2 also want bare-gel subtraction, the field's default remedy, run on campaign D where the frames exist. Nobody argued against it.

Recommendation: borderline, leaning accept. No reviewer found a flaw that invalidates the finding, and every Major item is fixable from existing results except the bare-gel oracle and a whole-stream AdaBN row. The rebuttal must deliver the inductive absolutes, a replotted Fig. 3 with the SITR reading number for both models, the view dependence in the abstract, and the 47-session stratum relabelled as unit and gel. The bare-gel row is the one compute item that would move the two 3s.

## 3. Consensus findings (ranked by reviewer count, then severity)

1. **Row e interval** (R1 R2 R3 R4 R5 R6; Minor, objective error; FIGURE/TABLE). "$-$0.8 [$-$2.0, $-$-0.5] vs d$^5$" contradicts "Its lead is within the interval". Fix: print -0.8 [-2.0, 0.5], audit every negative bound.

2. **Fig. 3 contradicts the SITR text and carries an undefined curve** (R1 R2 R3 R5 R6; Major; FIGURE/TABLE + TEXT). "there the reading does not help, but alignment does" (Secs. 1, 5.4) against a plotted "Algorithm 1 (causal)" curve that doubles SITR accuracy, plus "state augmentation" defined nowhere. Fix: replot from current results with only the curves the text discusses, and give the reading's SITR number in Sec. 5.4, naming the model.

3. **Transductive headline, inductive absolute missing** (R2 R4 R6; R1 R3 note it; Major; ANALYSIS + TEXT). "The stream therefore contains the test sessions themselves"; row i prints "--". Fix: absolute a, b, d on the 28 held-out sessions (predictions exist), "transductive" in the abstract, row i beside 61.6.

4. **View chosen where both parts work** (R1 R2 R3 R4; Major; TEXT). "On tube crops both parts of the method work for every encoder, which is why the paper's comparisons use that view." Fix: view dependence and encoder range in the abstract, whether the view was fixed a priori, why the reading hurts frame ResNet-50.

5. **Sigma and detector constants justified by a test-set sweep; pre-registration outcome unreported** (R1 R2 R3 R6; R5 on the outcome; Major; TEXT). "Narrower low-passes ($\sigma$ of 8 or 16\,px) lose ten to twenty points" with no validation split, and "we wrote down our predictions before training any model" never says how many held. Fix: state when and on what data the constants were fixed; print how many predictions held.

6. **Method framing on an analysis paper** (R1 R2 R4 R6; Major per R4; TEXT). "We use CDAN+E~\cite{long2018cdan} unchanged." and "This is the retrospective flat-field of microscopy". Fix: lead with measurement, shortcut evidence and dataset; the reading as the implied baseline, alignment as recommended practice.

7. **Spread under-reported** (R2 R3 R5 R6; Major; ANALYSIS). "by three to ten points with intervals that exclude zero", with only the CDAN+E interval printed, and no per-fold or per-seed spread. Fix: per-fold rows a to d, per-seed SD, a paired interval per "+ reading" row, a cluster bootstrap over folds.

8. **"Every" hides the gel stratum and the frame view** (R2 R3 R5; R1 R6 on stratum size; Major; TEXT). "It also improves every domain adaptation method we add it to". The gel column falls for every aligner and the mechanism paragraph rests on 8 sessions of 4 materials. Fix: scope to tube crops over 55 sessions, name the loss early, call the stratum small.

9. **No protocol or dataset table; materials unnamed** (R3 R5 R6; Major for the dataset; FIGURE/TABLE). "assigns 64\pct\ of these test episodes to the four materials" appears in no table; "one of 20 materials" is never listed. Fix: per-fold table (campaigns, sessions, strata, materials, episodes) carrying 64 and 35, plus material names.

10. **Figure and table presentation** (R3 R4 R5; Minor; FIGURE/TABLE + TEXT). Fig. 1 "shows the medians over sessions" with no spread, legend inside the axes, pairing unstated. Table 2 bold undefined ("\textbf{61.6}" while row f shows 63.7), three-seed means beside five-seed deltas, rows f and g on different bases. Table 3 header switches between blocks. Fix each.

11. **Number slips** (Minor; TEXT). "ten adaptation methods" vs eleven (R3 R5); "three quarters" for 11.6/16.8 = 0.69 (R3 R4 R5); "the same 20 objects" vs "a 16-object classifier" (R1 R2 R6). Fix: eleven, "about seventy percent", name the SITR subset.

12. **Field's default remedy never run** (R1 R2, R4 Q4 adjacent; Major; COMPUTE). "The reading needs no labels, no training and no bare-gel frame." yet "Campaign D also records frames of the bare gel." Fix: Alg. 1 with the session's bare-gel frame as F-hat on fold D, low-passed and full-band, beside row b, plus per-crop standardisation (R2); new predictions only.

13. **Training and method details missing** (R5 R6; Major, reproducibility; TEXT). "fine-tuned at the native \SI{11.3}{\micro\metre}/px on 192-px crops along the contact, with flips, quarter turns and photometric jitter" is the whole recipe. Fix: recipe table, forward-referenced.

## 4. Single-reviewer findings

### Worth acting on

- **AdaBN mis-specified, TTA block one failure counted four times, NOTE/LAME absent, batch size unstated** (R2; Major; TEXT, COMPUTE for new rows). "Test-time BatchNorm, Tent, SAR and DeYO estimate normalisation statistics from the current batch", while Li et al. use whole-target statistics. Fix: batch size, relabel "per-batch BN", present the shuffled 37.8 as the domain-level variant, add whole-stream AdaBN and NOTE or LAME.
- **Unit and gel confounded in the 47-session stratum** (R4; Major; TEXT). "Unit 1 carried gel 1 until July 9, unit 2 carried gel 2 from July 10" against the header "unit (47)" and "the smooth field belongs mostly to the unit". Fix: relabel "unit and gel", soften the attribution.
- **Image-derived timeline, circular detector test** (R3; Major; TEXT). "Our laboratory log had recorded a change of unit and four gel replacements. The images show only the two changes above". Three replacements without a pattern change conflict with "a new gel on the same unit and camera removes it". Fix: what the operators concluded, non-image evidence, spliced streams as the independent test.
- **Fine-pattern attribution overstated** (R1; Minor; TEXT). "an optical property of the cartridge ... rather than texture on the deforming surface." Fix: stop at "belongs to the cartridge", name the coating hypothesis.
- **Sessions where the reading is identity** (R4; Minor; ANALYSIS). "the first session after a change is read relative to $G$ alone". Count them among the 55.
- **Qualitative figure and Fig. 2 correlations** (R5; Minor; FIGURE/TABLE). Read vs unread crop with a gel-stratum failure; correlations printed under Fig. 2's top row.
- **Detector rates for the other three spliced datasets** (R6; Minor; FIGURE/TABLE). Only the in-house 85.0% is given.
- **Aligner sees unread stream images while test images are read** (R5, R6 Q4; Minor; TEXT). "Only the test images are read." Explain, and say whether fold D has one target domain or two.
- **SITR Sec. 5.4 numbers without seeds or intervals** (R3; Minor; ANALYSIS). "from 12.8 to 66.8\pct".
- **Minor TEXT items.** "Imprint" with two meanings, "hardware configuration" before its definition, "tube" and "grouped confidence" unintroduced (R5). WILDS/Camelyon17 and Noiseprint uncited (R4). What each baseline knows about object boundaries (R1). LLM statement still a draft (R6).
- **Second task from co-recorded force** (R4; COMPUTE), or scope the abstract to material recognition (TEXT).

### Not worth acting on

- R1 W5, same-hardware months-apart control: needs new recordings, already in Limitations.
- R1's unreviewed gel-wear write-up: R1 says not a citation requirement.
- R4 "within a point" (gap 2.4) and R3 "Fig. 1 has no N" (caption gives 55): misreadings.

## 5. Disagreements

- **What the contribution is.** R4 and R1 (Contribution 2): a flat-field plus unchanged CDAN+E cannot carry a method paper, and the title presents one. R6 (Accept), R3 and R5 (Contribution 3): the measurement and the SITR injection are the contribution. Both agree the components are cited tools, so TEXT settles it.
- **The TTA collapse.** R1 S4 and R4 S3 count it as a finding beyond tactile. R2: one per-batch BatchNorm failure, and NOTE and LAME are the fair comparators. R2 is right on AdaBN's definition; the shuffled 37.8 partly answers.
- **Realism of "no bare-gel frame".** R1: a gripper's gel is bare whenever the jaws open, so the setting describes archives only. R6 accepts "needs only the robot's ordinary contacts" as the distinguishing feature. Finding 12 decides it.
- **Severity of sigma.** R2, R3: selected on test, Major. R6: a physical reason is given, so undocumented selection, not tuning. R1: Minor. The sensitivity is as large as the reading's gain, which favours R2 and R3.

## 6. Unverified reviewer claims

- Figure-only: Fig. 1 legend dots inside the axes, Fig. 2 top row unreadable (R5); Fig. 2 marker grid implying 1.2 mm pitch (R1 Q4).
- External: Tactile MNIST at 320x240, so Fig. 1's orange points are downsampled (R1); Tactile MNIST "three weeks", GelSlim "every hundred grasps" (R1, R6); SITR Minis differ by coating (R1); RxRx1 AdaBN gain (R2); RDP quote (R6).

## 7. Questions the rebuttal must answer

1. Absolute accuracy of rows a, b and d on the 28 inductive sessions of row i.
2. Fig. 3: what the two extra curves were, and the current reading number on SITR for the CNN from scratch and for frozen DINOv2.
3. Were the tube view, sigma=33 px and the detector constants fixed before the first cross-hardware result, and how many pre-registered predictions held?
4. Alg. 1 with the session's bare-gel frame as F-hat on fold D (low-passed and full-band), and per-crop standardisation.
5. What became of the three logged gel replacements the images do not show, and what non-image evidence dates the two retained changes?
6. Per-fold accuracy and session counts for rows a to d, per-seed SD, and a fold-level bootstrap for rows b and d.
7. TTA batch size, and AdaBN with statistics over the whole configuration stream.
8. How many of the 55 test sessions are first after a change, and is the fold-D CDAN+E target one domain or two?
9. 16 or 20 SITR objects; Tactile MNIST resolution for Fig. 1 and the "100%" collapse; how frozen Sparsh ran on A to C without blank frames.

## 8. What the supplement must contain

- Dated pre-registration text for the A+B fold with predicted versus observed numbers.
- Frozen Sparsh: variants, probe, preprocessing without blanks, 55-session accuracy and collapse metric.
- SITR: contrast measurement (3.7x), field of view, tie protocol without test-object leakage, CNN architecture, 5-class stream result, reading numbers per model with seeds.
- SDAT and MIC: both trade-off weights and results. HCT negative numbers.
- Detector: detection and false-alarm rates on all four spliced datasets.
- Recipe table: optimizer, schedule, batch size, iterations, jitter, contact localisation, frame choice, discriminator, compute.
- Dataset: 20 materials with photos and force caps, session table (date, unit, gel, material, episodes), hardware, lubricant, 55-session list, bare-gel protocol.
- Sigma sweep table, per-fold and per-seed spreads, paired intervals for every "+ reading" row, row i absolutes.
