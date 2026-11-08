# R2 - Domain adaptation and TTA expert
Primary skill: skills/paperjury_personas.md. Also used: skills/critique_framework.md, skills/neurips_form.md, skills/00_README.md (objective-error stage).
CVPR rating: 3/6. Confidence: 4/5.
NeurIPS-form sub-scores: Soundness 2/4, Presentation 3/4, Contribution 3/4, Overall 4/10.
One-line verdict: a careful and valuable diagnosis of a hardware shortcut in GelSight data, paired with a method whose evaluation has a transductive headline, a mis-specified AdaBN baseline, a low-pass width apparently chosen on the test sessions, and a figure that contradicts the text on SITR.

## Pass 1: candidate fatal flaws (blunt)
1. The headline 61.6 is transductive (test sessions inside the alignment stream); the abstract does not say so and the inductive row has no absolute accuracy.
2. "test-time BatchNorm [AdaBN]" is not AdaBN as published; four of five TTA baselines are one BN failure counted four times; the cited label-shift-robust methods (NOTE, LAME) are not run.
3. The one hyperparameter, sigma = 33 px, appears to have been selected on the 55 test sessions.
4. Fig. 3 shows "Algorithm 1 (causal)" recovering most of the SITR accuracy while the text says the reading does not help there; "state augmentation" is defined nowhere.
5. Standard practice, bare-gel subtraction, is never run as a baseline although campaign D records bare-gel frames; no stream-free per-crop normalisation either.
6. "Improves every domain adaptation method" holds for the 55-session mean but the reading lowers every retraining method on the gel-change stratum.
7. The headline view was picked because both parts work there; in the frame view the reading hurts ResNet-50 and adds nothing to ConvNeXt, whose aligned-only model is within 2.4 points of the headline.
8. Novelty is not calibrated: the reading is a low-pass per-domain mean subtraction, and the paper never says whether it is a diagnosis paper or a method paper.

## Pass 2: forensic interrogation
1. Sec. 5: "The stream therefore contains the test sessions themselves, as a working robot's stream would". Honest in the body and in Limitations, but the abstract reads as inductive, and Table 2 row i gives only a delta ("11.6 [5.1, 18.6] vs b") with "--" for the absolute, so the inductive accuracy of the full method cannot be recovered. Fixable: say "transductive" in the abstract and report a, b, d on the 28 held-out sessions.
2. Sec. 5.1: "Test-time BatchNorm, Tent, SAR and DeYO estimate normalisation statistics from the current batch, and consecutive episodes come from one session, so every batch holds a single material." I checked Li et al. 2018 (arXiv 1603.04779): its Algorithm 1 computes statistics over "all images of target domain". The paper's whole-configuration stream is exactly that target domain, and the retraining methods already receive it, so AdaBN as published never sees a single-class batch. The 37.8 "shuffled" number is the nearest thing but is presented as a benchmark setting, not as the method. The RxRx1 paper the authors cite (Sypetkowski et al. 2023) reports domain-level AdaBN as the one method that generalised to unseen batches (75.1 to 87.1). Tent, SAR and DeYO inherit the same BN issue, so Table 1's TTA block is one failure counted four times; NOTE and LAME, cited in Sec. 2 as handling temporal correlation, are the fair comparators and are absent. Batch size is unstated. Fixable.
3. Sec. 5.2: "Narrower low-passes ($\sigma$ of 8 or 16\,px) lose ten to twenty points, and wider ones (66 or 132\,px) lose up to two." These are test-set numbers and no validation split is described; the pre-registration covers only the A+B fold. The sensitivity is of the order of the reading's own gain (12.4). Fixable if sigma was fixed a priori from the physics or chosen on training-configuration data.
4. The intro says "SITR's Minis also differ in their field of view and contrast, and there the reading does not help, but alignment does." In fig_confound.png, "Algorithm 1 (causal)" lifts the k=1 accuracy from about 37 to about 73 and halves the top-class share. Sec. 5.4 uses a CNN from scratch and Fig. 3 a frozen DINOv2, so both may be true, but the paper does not say so, and "state augmentation" is never defined. An objective figure-versus-text conflict on the method's stated limit. Must be fixed.
5. The intro names "The first subtracts a frame of the unloaded gel" as practice one, and Sec. 3 says "Campaign D also records frames of the bare gel." The reading is never compared with it, so the claim that the stream replaces the blank frame is untested. Fixable.
6. Discussion: "The reading lowers the accuracy on the eight sessions recorded on the other gel of the same unit, both alone and on top of alignment." Every "+ reading" row in Table 1's gel column is lower (CDAN+E 53.1 to 44.5). The abstract's "every" is a 47-session statement. Wording.
7. Sec. 5.3: "On tube crops both parts of the method work for every encoder, which is why the paper's comparisons use that view." Choosing the view after seeing where the method works is selection; ConvNeXt-T frame reaches 59.2 by alignment alone. Not fatal, but the headline should say the reading's share depends on crop size.
8. Sec. 2: "Our estimate is such a flat-field, taken from a robot's stream of contacts and restarted when the hardware changes." Candid on lineage, but silent on input-side DA normalisation (per-domain or per-image standardisation, Retinex-style illumination removal) and on whether this is a diagnosis paper or a method paper.

## Weaknesses (paperjury schema, by severity)
- W1. Headline is transductive and the inductive absolute is missing. Anchor: "The stream therefore contains the test sessions themselves, as a working robot's stream would". Sec. 5 / Table 2 row i. Major, substantive. Settle: absolute a/b/d on the 28 held-out sessions; "transductive" in the abstract.
- W2. AdaBN baseline is not AdaBN; TTA block is one failure counted four times; NOTE/LAME missing. Anchor: "Test-time BatchNorm, Tent, SAR and DeYO estimate normalisation statistics from the current batch". Sec. 5.1 / Table 1. Major, substantive. Settle: whole-stream AdaBN, NOTE, LAME, batch size.
- W3. Low-pass width selected on test sessions. Anchor: "Narrower low-passes ($\sigma$ of 8 or 16\,px) lose ten to twenty points, and wider ones (66 or 132\,px) lose up to two." Sec. 5.2. Major, substantive. Settle: selection protocol.
- W4. Fig. 3 contradicts the text on SITR; "state augmentation" undefined. Anchor: "there the reading does not help, but alignment does." Sec. 1 / Sec. 3 / Fig. 3. Major, substantive (presentation of a central limit). Settle: describe both curves and reconcile with Sec. 5.4.
- W5. No bare-gel-subtraction baseline, no stream-free per-crop standardisation baseline. Anchor: "Campaign D also records frames of the bare gel." Sec. 3 / Table 1. Major, substantive. Settle: both rows on fold D (RxRx1 found per-image standardisation worth 14 points on a similar flat-field problem).
- W6. "Every method" claim hides a consistent loss on the gel stratum. Anchor: "It also improves every domain adaptation method we add it to". Abstract. Minor, substantive.
- W7. Comparison view chosen post hoc. Anchor: "which is why the paper's comparisons use that view." Sec. 5.3. Minor, substantive.
- W8. No seed spread; intervals are bootstraps over sessions of seed means. Anchor: "Table rows average three training seeds". Sec. 5. Minor, substantive.
- W9. Object count on SITR: "SITR~\cite{sitr2025} pressed the same 20 objects" vs "a 16-object classifier". SITR's paper has 16 classification objects. Minor, mechanical.
- W10. Sign typo in Table 2 row e: "$-$0.8 [$-$2.0, $-$-0.5]". Minor, mechanical.

## Strengths
- The four-month measurement and the injected-tie control on SITR (Fig. 3, raw curve) cleanly demonstrate shortcut learning; the top-class share is a good diagnostic.
- Pre-registered fold, honest negative rows (f, g, h), limits stated in the text.
- The reading is training-free and composes with every retraining method on the unit stratum, paired intervals excluding zero.
- The dataset release (two units, three gels, bare-gel frames, force) is the most useful artefact.

## Questions for the authors
1. What is the absolute accuracy of rows a, b and d on the 28 sessions of row i?
2. How was sigma = 33 px chosen, and on what data?
3. In Fig. 3, what is "state augmentation", and why does "Algorithm 1 (causal)" help on SITR there but not in Sec. 5.4?
4. What batch size did the TTA methods use, and what does AdaBN give with statistics over the whole configuration stream?
5. On fold D, what does subtracting the recorded bare-gel frame give, and what does per-crop per-channel standardisation give?
6. 16 or 20 SITR objects?

## What would change my score
To 4: Q1, Q2 and Q3 answered with the reconciled figure, plus whole-stream AdaBN and the bare-gel baseline in Table 1. To 5: in addition, NOTE and LAME run, seed spread reported, and the abstract stating the transductive setting and the gel-stratum loss.

## Claims resting on the unseen supplement
Sparsh failing "in the same way"; the pre-registration file; the SDAT/MIC weight choice; the 3.7x SITR contrast ratio; the SITR CNN protocol and 5-class stream; 85.0% detection on spliced streams; the HCT negative; the CNN architecture. It must show the pre-registered predictions verbatim and a SITR protocol detailed enough to reconcile with Fig. 3.

## Coverage
Abstract: thorough, "every" claim checked against Table 1. Sec. 1: thorough, 73.9/37.0/62% checked against Fig. 3. Sec. 2: thorough, AdaBN, RxRx1, SITR and Burns-Steinhardt checked at source. Sec. 3: light on optics, thorough on the shortcut paragraph. Sec. 4: thorough, Algorithm 1 read as a low-pass per-domain mean difference. Sec. 5: thorough, all tables cross-checked. Sec. 6: thorough. Sec. 7: read.

## Limitations of my review
No supplement or code seen. Tactile-specific citations not verified beyond SITR. The Fig. 3 versus Sec. 5.4 conflict may dissolve once the two setups are explained. I name no specific uncited input-normalisation paper; one search found no canonical one.
