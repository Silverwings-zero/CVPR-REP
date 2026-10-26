# Writing and literature survey for the CVPR 2027 submission (2026-10-25/26)

Three parts: (1) how eight strong CVPR/ICLR/NeurIPS papers on related topics write their abstract
and introduction (verified by reading the texts); (2) the tactile literature — what is evaluated, what
is claimed, where our claims stand; (3) imaging-science analogues of our ingredients (flat-field from
the image collection, sensor fingerprints, batch/site effects, temporal shift, test-time adaptation).
Parts 2–3 follow when their surveys land. Novelty verdict at the end.

## Part 1 — abstract and introduction anatomy of eight reference papers

| paper | abstract | introduction | contributions |
|---|---|---|---|
| Sparsh (Higuera et al., CoRL 2024, 2410.24090) | 206 w, 7 S: claim · context+gap · why hard · approach · method+scale · benchmark · key number + one insight | 4 P: hook → practice and its cost → "in this work" + data → results with the number | numbered 1–3, 8–14 w noun phrases |
| SITR (Gupta et al., ICLR 2025, 2502.19638) | 126 w, 6 S, no number anywhere (weakness) | 6 P: context → problem → nearest prior work and why it fails → proposal with 3 innovations → motivation → claims | 1–3, full sentences 30–60 w (too long) |
| Torralba & Efros, CVPR 2011 (DOI 10.1109/CVPR.2011.5995347) | 211 w, 9 S: context · tension · rhetorical question · goal · what was done · "results, some rather surprising" · hope | 5 P, no bullets: the "Name That Dataset" game → toy experiment with number vs chance → confusion matrix, "signature" → lesson + control (car crops: 61 % vs 20 %) → aim | none |
| Geirhos et al., Nature MI 2020 (2004.07780) | 146 w, 6 S: context · turn · thesis · one-sentence definition · breadth · deliverable | 6 P (perspective): analogy → urgency → catalogue of failures → shortcuts explained (cow/grass, hospital token) → "not a novel phenomenon" → roadmap | none |
| Wild-Time (Yao et al., NeurIPS 2022, 2211.14238) | 196 w, 10 S: definition · subtype · opportunity · gap (11 w) · what · what · protocols · key number (20 % drop) · verdict (8 w) | 5 P: definition + three dated collapses → prior benchmarks and why inadequate, ending "there does not exist..." → "this paper presents" → what was evaluated → conclusion + release | none |
| Tent (Wang et al., ICLR 2021, 2006.10726) | 130 w, 7 S: premise · setting · method · mechanism · result · result · cost | 5 P + bullets | 4 bullets, 25–35 w, verb-led, each with a number or object |
| ParamISP (Kim et al., CVPR 2024, 2312.13313) | 171 w, 6 S | 7 P + bullets; Fig. 1 caption = "same scene, different acquisition state, measured difference" | 4 bullets, 25–45 w |
| ViT registers (Darcet et al., ICLR 2024, 2309.16588) | 129 w, 5 S: context · "identify and characterize artifacts" · what + mechanism · fix · consequences | 6 P, no list: generic features → anomaly observed → characterisation with numbers → probing → interpretation (labelled) → hypothesis test | none |

Exemplary sentences (for calibration of tone):
- Torralba: "Curiously, the best classifier performs rather well at 39% (chance is 1/12 = 8%), and ... there is no evidence of saturation." — number, chance level, trend, inline.
- Geirhos: "The model had unexpectedly learned to identify particular hospital systems with near-perfect accuracy (e.g. by detecting a hospital-specific metal token on the scan)." — the shortcut named by its physical carrier. Nearest analogue of our story.
- Wild-Time: "Existing methods are unable to close this gap." — 8-word close.
- Tent: "tent reaches 44.0% error on ImageNet-C, better than ... robust training (50.2%) and ... test-time normalization (49.9%)." — number with both comparators in one sentence.
- Registers: "We propose the following interpretation" — mechanism labelled as interpretation, then tested.

### Abstract template (~180 w, 8 sentences)
1. Context (≤20 w): what the sensor's images are used for and the assumption everyone makes.
2. Finding (≤25 w): frames carry a per-campaign state shared by every image of the campaign.
3. Measurement (≤25 w): probe + chance level + persistence.
4. Consequence (≤25 w): within-campaign vs next-campaign numbers; what standard adaptation closes.
5. Explanation (≤20 w): the physical cause.
6. Fix (≤25 w): estimate the state from the contact stream, remove it before the encoder.
7. Result (≤25 w): after-fix number with comparators inline.
8. Scope + release (≤15 w).
Rules: one number per sentence; name the chance level at the first probe accuracy; "surprisingly" at most once; no adjective on the method except "simple".

### Introduction template (5 P, ~900 w; contributions ≤100 w)
P1 context (120–150 w): one concrete opening sentence; what a frame is assumed to contain; the practice the paper tests. ≤4 citations.
P2 the gap (150–200 w): each nearest line of work in one clause and why it does not cover this case (cross-sensor: Sparsh, SITR; TTA: Tent, AdaBN; dataset bias: Torralba, Geirhos); the gap as a fact about the literature.
P3 what we did and found (200–250 w): the probe first (protocol in two sentences, number vs chance) → the control that rules out the obvious explanation → characterisation with numbers → the collapse number → one-sentence interpretation, labelled.
P4 what we propose (120–150 w): hypothesis-test framing; estimate, remove, train in three sentences; headline number with two comparators; scope.
P5 contributions: ≤4 verb-led bullets of ≤25 w, each with a number or object. No roadmap paragraph.

### Twelve habits that read as machine-written, and the alternative used by these papers
1 triads → one adjective or none · 2 em-dash asides → a second sentence, parentheses for a number · 3 "X is not Y; it is Z" → state Z · 4 stacked appositives → name it, then one sentence · 5 nominalisations → verbs with objects · 6 "Notably/Importantly" → main clause · 7 hedged results → number + comparators · 8 "In this paper, we" everywhere → once, at the pivot · 9 synonym rotation (artefact/signature/fingerprint) → one defined term · 10 "fundamentally/paradigm/unlock" → dated concrete failures · 11 significance without an observer → who is affected and how · 12 full-sentence bullets of 40–60 w → noun phrases of 8–14 w or verb-led ≤25 w.
Two more: the discovery goes into the Figure 1 caption as "same stimulus, different acquisition, visible difference"; "novel" at most once, and the papers that use it are the ones without a number in the abstract.

## Part 2 — tactile literature (verified by reading; arXiv IDs)

### A. Foundation models / representation learning — how they evaluate
| paper | protocol | relation |
|---|---|---|
| Sparsh (2410.24090, CoRL 2024) | within-collection splits; downstream sensors are different units; subtracts a background frame "to increase robustness to ... light placement variations ... manufacturing discrepancies" and says it "helps models generalize across the same type of sensor" | closest FM statement that blank subtraction removes unit appearance; no same-unit later session; no contact-stream estimate |
| T3 (2406.13640, CoRL 2024) | random splits within each dataset | within-collection |
| UniTouch (2401.18084, CVPR 2024) | official/object splits; "sensor-specific tokens" model "calibration and intensity profiles" | state as a per-sensor-TYPE token, never per session |
| AnyTouch (2502.12191, ICLR 2025) / AnyTouch 2 (2602.09617, ICLR 2026) | dataset splits; background subtraction; App. K replaces the Mini's gel at test time for two policies (0.30→0.30, 0.80→0.75, "minor drop") | the only FM paper with a same-unit gel-swap test — 20-trial policy test, no feature analysis |
| TVL (2402.13232, ICML 2024) | 99/1 frame-random split; per-trajectory background = mean of out-of-contact frames | reference from blanks within each touch; RCT shows the split leaks |
| Touch100k (2406.03813), TLA (2503.08548) | within-collection / simulation | — |
Verdict A: no foundation-model paper holds out a later session of the same physical unit.

### B. Cross-sensor / sensor-invariant representations
- SITR (2502.19638, ICLR 2025): four Minis "differ in brightness and color due to manufacturing differences and coating"; ViT 96.8 % same unit → 36.9 % other Mini; SITR 90.2 % with background subtraction + 18 calibration presses + simulated optics. Strongest prior that same-model units differ by a fixed illumination/colour pattern and that classifiers collapse across units; no temporal axis.
- BIDETA (2609.08673, 2026): frozen Sparsh/TVL trained on DIGIT → 6–9 % on Minis; a few labelled contacts recover 85–87 %.
- TacVerse (2606.25877, 2026): 7 sensors, chronological trial splits within a campaign; grating 0.90→0.25 across Mini variants; mentions "sensor wear over time" unmeasured.
- Castaño-Amorós & Gil (2608.18240, 2026): Mini-to-Mini variation = "different color backgrounds"; VQGAN re-renders to a canonical background, subtracts the reconstructed background. Re-rendering to a canonical state, blank-based, unit-level.
- SPLIT (2604.24449, 2026): β-VAE latent arithmetic — subtract the blank-sensor latent, add another unit's, to synthesise contacts in that unit's optics (augmentation); force MAE on unseen DIGITs 6.09 → 1.64 N. CLOSEST to our label-free state swap; blank-based, cross-unit, no temporal states, no FM adaptation.
- GenForce (2503.01058, Nature Comm. 2026): marker fields as the unified signal; marker-to-marker translation conditioned on the target's non-contact reference; force error 4.8 N → <1 N. Supports "marker geometry is the invariant force carrier".
- CTTP (2410.11834), HTT (2606.29948, CoRL 2026; subtracts the first two frames of each episode), Khalifa language distillation (2609.14783), TactX (2606.31236), UniT (2408.06481, RA-L 2025): cross-sensor-type; none addresses one unit over time.
Verdict B: instance shift is well documented; nobody isolates the same unit's state over months; every fix uses blanks, calibration probes or labels.

### C. Drift, wear, calibration over time
- GelSlim (Donlon et al., 1803.00628, IROS 2018) — THE key prior: "calibration framework to regularize the sensor output over time and across sensor individuals"; signal-quality metrics every 100 grasps over 3,300 grasps; brightness drops after ~1,750 grasps; wear spatially non-uniform; correction = subtract a Gaussian-filtered blank re-captured over use + restore a brightness reference. A blank-based, periodically re-estimated state subtraction on one unit — our blank variant is anticipated here, without any learned-classifier evaluation.
- Davis & Stuart (2511.07797, 2025): degradation = MAE of the unloaded image over 1,000 cycles; "recovery through recalibration ... future work".
- Cottrell (2608.24242, 2026): protective-film fingertip; PolyTouch (2504.19341, ICRA 2025): Mini gels fail at 1–3.3 h continuous rubbing; TacEva (2509.19037); TacO (2605.21976): one-hour viscoelastic "training phase".
- Wang et al. (2203.15069, 2022; resistive glove, not VBTS): random split 98.9 % vs leave-one-session-out 49.6 %, response falls ~40 % over five sessions — an explicit session effect in another sensor family.

### D. Deployment practice
- Per-episode blank reference is the norm: TACTIC (2609.30969, 2026) averages 50 blank frames per rollout and collects with four DIGIT cartridge pairs "to reduce overfitting to cartridge-specific characteristics"; HTT, Sparsh, SITR, AnyTouch 2 subtract backgrounds; ManiFeel (2505.18472), TacO, PolyTouch feed raw images.
- NeuralFeels (2312.13469, Science Robotics 2024): backgrounds from 25 DIGITs + LED randomisation in simulation. RotateIt (2309.09979): reduces the image to a contact location. Reactive Diffusion Policy (2503.02881, 2025): marker-displacement PCA because it is "more robust to texture and lighting changes due to damage or gel replacements". FEATS (2411.03315, IROS 2025): raw-image force on a second Mini + new gel: 0.80 → 2.99 N; attributes it to "camera alignment and sensor-specific image variations". Tactile MNIST (2506.06361): gel replaced after 76,800 touches, gel type in metadata, never analysed.
- Same unit months apart: none. Reference estimated from contacts rather than blanks: none.

### E. Session / shortcut effects reported
- RCT (He, Färber, Calandra, 2606.31694, 2026): frame-random TVL splits leak (raw-pixel NN recovers the sequence 98.3 %); held-out DIGIT instance 21.5 % R@1 vs 65.5 % held-out position — "sensor-instance robustness is a distinct bottleneck". Closest published instance-shortcut statement; no temporal axis.
- SenseAlign (2610.28511), RobustTouch (2602.15873), 2026: label-free test-time adaptation of TVL models under synthetic corruptions / other sensor types.
- No paper uses "session signature" or "batch effect" for VBTS.

### Novelty verdict per claim (tactile side)
1. Campaign-persistent state in the bare gel, changing at rebuild/gel swap — PARTIALLY KNOWN (unit/gel colour-illumination differences: SITR, FEATS, Castaño-Amorós, GelSlim brightness drift); the within-unit months-long persistence/transition structure is new.
2. Classifiers collapse because they identify sessions by state — NEW for VBTS (analogues: RCT held-out instance; Wang 2022 leave-one-session-out; SITR/BIDETA cross-unit).
3. Subtracting a state estimate undoes it — blank variant KNOWN (GelSlim 2018; Sparsh; SITR; HTT; TACTIC; AnyTouch 2); the contact-stream (previous object / campaign mean) estimate is NEW. Must be positioned against GelSlim and TVL.
4. Label-free re-rendering + decoys for frozen FMs — PARTIALLY KNOWN (SPLIT, Castaño-Amorós, GenForce, SITR, BIDETA); the label-free temporal-state version for a frozen FM is unoccupied.
5. Force from marker geometry, appearance fails across gel/unit — KNOWN in principle (Yuan 2017; GenForce; RDP; FEATS 3.7×; SPLIT 3.7×); present as confirmation across campaigns.
6. Replication on Real Tactile MNIST — NEW.
Ten papers a reviewer expects: Sparsh, UniTouch, SITR, AnyTouch/AnyTouch 2, T3, GelSlim, TacVerse, RCT, SPLIT, GenForce; secondary BIDETA, TACTIC, FEATS, Castaño-Amorós, Tactile MNIST, Geirhos.
Flagged as not read in full: Yuan 2017 (Sensors 17(12):2762), TactX; DIGIT 360 pages had no drift content.

## Part 3 — imaging-science and ML analogues (verified; arXiv IDs / DOIs)
Verdict up front: every ingredient has a named home; the conjunction is open.
1. Flat-field FROM THE COLLECTION: Singh, Bray, Jones, Carpenter 2014 (J. Microscopy 256(3):231, CellProfiler: illumination function = average of all images of a batch, median-smoothed, divided out — our estimator verbatim, "objects average out"); BaSiC (Peng et al., Nat. Commun. 8:14836, 2017: low-rank+sparse flat-field + additive dark-field + drift); CIDRE (Smith et al., Nat. Methods 2015); Likar 2000 / Tomazevic 2002 (J. Microscopy: PROSPECTIVE (blank) vs RETROSPECTIVE (from the data) correction — adopt this wording); Harris & Chiang 1999 (IEEE TIP 8(8):1148: scene-based non-uniformity correction for IR arrays = per-pixel temporal mean over a moving scene; names our failure mode "ghosting" when content does not average out); Kim & Pollefeys 2008, Goldman 2010 (vignetting from image sequences).
2. Fingerprints fixed in image coordinates: Lukas, Fridrich, Goljan 2006 (IEEE TIFS 1(2):205, PRNU = mean of noise residuals); Marra 2018 (1812.11842: fingerprint convergence with N); Noiseprint (1808.08396: position-dependent fingerprint); Zech 2018 (1807.00431: CNN identifies the hospital from X-rays at 99.95 %); Rahbar 2026 (2608.10295: MRI site decodable from raw downsampled pixels); Grommelt 2024 (2403.17608).
3. Batch / site effects: ComBat (Johnson 2007; Fortin 2017/2018 NeuroImage) — our per-campaign mean is a location-only pixel-wise ComBat; Glocker 2019 (1910.04597: site still classifiable after harmonisation); Howard 2021 (Nat. Commun. 12:4423: site signatures in TCGA histology); Schmitt 2021 (JMIR: scanner and slide-preparation DATE are learnable); TVN (Ando 2017), Caicedo 2017, Arevalo 2024; RxRx1 (Sypetkowski 2023, 2301.05768: AdaBN ~96 % on unseen batches; gradient reversal removes batch decodability WITHOUT improving generalisation — matches our DANN result); Moyer 2020 (scanner-invariant representation).
4. Temporal shift / dataset bias / shortcuts: Torralba & Efros 2011; Tommasi 2015; Liu & He 2025 (2403.08632, ICLR oral: dataset identity decodable at 84.7 %, survives jitter/noise/blur/32-px downsampling); Geirhos 2020; Lapuschkin 2019; Oakden-Rayner 2020; DeGrave 2021 (Nat. MI: acquisition setup as COVID shortcut — closest story); Wild-Time 2022; CLEAR 2021.
5. Normalisation-statistics TTA and its failures: AdaBN (1603.04779), Tent (2006.10726), Schneider 2020, Nado 2020; Burns & Steinhardt 2021 (2103.05898, CVPR: BN alignment hurts under label shift — why per-session AdaBN collapses); NOTE 2022, LAME 2022, SAR 2023, Zhao 2023.
6. Re-rendering under another acquisition condition: FDA (Yang & Soatto, CVPR 2020, 2004.05498: amplitude swap) and FACT (CVPR 2021) — nearest ML precedent for our label-free swap (theirs a global spectrum, ours a spatial field); MixStyle; stain augmentation (Tellez 2019; Kang CVPR 2023); CDCL (Haslum 2022: positives = same treatment, other batch).
Two reviewer traps: (1) removing state decodability ≠ improving accuracy (RxRx1; 2608.10295) — report both (we do: era probe vs accuracy); (2) NUC "ghosting": the "imprints average out" assumption needs the diversity check (we have it: own session worst, random/campaign best).
Eight papers an imaging reviewer expects: BaSiC; Singh 2014; Lukas 2006; Harris & Chiang 1999; Torralba & Efros 2011; Geirhos 2020; Zech 2018 or Howard 2021; Fortin 2017. Next: Tent/AdaBN, Burns 2021, FDA 2020, Liu & He 2025.

## Novelty verdict (synthesis, 2026-10-26)
KNOWN and must be cited as standard: subtracting a blank/background frame (GelSlim 2018 periodic blank
recalibration over 3,300 grasps; Sparsh, SITR, HTT, TACTIC, AnyTouch 2); estimating a flat-field from
the image collection without blanks (Singh 2014, BaSiC, CIDRE, scene-based NUC); classifiers learning the
acquisition device/site/date (Zech, Howard, Schmitt, Torralba, Liu & He); unit-to-unit VBTS appearance
differences and the cross-unit collapse (SITR 96.8 → 36.9; BIDETA; FEATS); instance shortcut in tactile
data (RCT 2026); session effect in a resistive glove (Wang 2022); re-rendering under another acquisition
condition as a training view (FDA, stain augmentation, SPLIT's blank-latent swap for VBTS units).
NEW: (a) the same unit followed over four months with a later campaign held out — no VBTS paper does it;
(b) the characterisation of the state as campaign-level, image-fixed, present in the bare gel, with the
transition at the rebuild and the gel swap; (c) the collapse mechanism — classes a single session's state
identifies become the sink for an unseen state — with its physical cause (the later campaign's state is
nearest to C's); (d) the retrospective estimate from the CONTACT stream (previous object / campaign) shown
equivalent to the blank, which makes the correction applicable to archives; (e) the leave-one-campaign-out
rotation showing where subtraction helps, is unnecessary, or cannot (gel); (f) the label-free state swap
with decoys on a frozen tactile FM; (g) replication on Real Tactile MNIST.
HONEST POSITIONING: a measurement-and-diagnosis paper with a simple, standard-practice-derived fix, on a
sensor family where nobody has looked; not a new algorithm. The contribution list must say so.
