# Data audit: tactile material dataset (campaigns A–D)

This was a read-only audit of the raw logs, the caches and the stored results. No model was trained. The scripts and their outputs are in this folder (`runmeta.py`, `protocol.py`, `cache_integrity.py`, `spectra*.py`, `montage.py`, `confusions.py`, `ceiling.py`, plus `*.json` and `montage_*.png`). Accuracies are per-material numbers from `tracker_folds.json`, given as plain CNN / jitter CNN, 3 seeds each.

## Verdict
- **Most of the gap between D after state subtraction (56 %) and the same-unit folds (72–73 %) sits in six classes with data problems.** Three are VERIFIED: textured rubber, EPDM and wood. Three are SUSPECTED: canvas, fleece and ceramic. On the 11 unit-1 materials with no flag, D reaches 83 / 82 %, which is above their same-unit level of 74 / 77 %.
- **The raw collapse to 16.6 % is not a data problem.** It is the known shortcut onto the July unit-2 classes.
- **For the 15 unit-1 materials, the hardware effect cannot be separated from the campaign-D protocol.** Every unit-2 session of these materials is also a D session, and the protocol changed in the same step.

## 1–2. Classes × campaigns
Each cell gives cached episodes · median Nano17 normal force on the crops (N). Flags:
- **s**: strict protocol (bare lead-in, re-grip every episode)
- **b**: board (riser) geometry
- **o**: second operator
- **L0**: no lubrication pause logged
- **★**: spectrally odd session (its imprint spectrum deviates by more than 0.28 log units beyond the campaign-wide shift)

"Same-unit" is the mean of folds A and B, read raw. "D" is the tracker reading with the state estimate (trk|state).

| class | A (S1) | B (S1) | C | D | same-unit p/j | D p/j | status |
|---|---|---|---|---|---|---|---|
| Textured_Rubber | 38·2.9 | 30·3.0 | – | 59·3.2 s L0 | 8/1 | 0/0 | new piece in D (VERIFIED); A/B imprint featureless |
| EPDM_Foam | 38·1.2 o | 30·0.8 | – | 68·1.0 s | 88/64 | 75/62 | new piece in D (VERIFIED) |
| Canvas_1 | 38·2.5 | 30·3.3 | – | 64·3.0 ★ | 99/99 | 0/0 | D anisotropy 14.6 vs 6.7/7.4 (SUSPECTED) |
| Fleece | 38·2.0 | 30·1.8 | – | 68·1.6 s ★ | 71/90 | 3/1 | SUSPECTED change of piece, side or nap |
| Silk_1 | 38·3.3 | 30·3.3 | – | 67·3.5 ★ | 90/93 | 100/100 | D striated (anisotropy 36 vs 7–8), still recognised |
| Polyethylene_Foam | 38·1.0 | 30·0.5 | – | 68·1.1 s | 33/56 | 78/100 | D ran at EPDM's force levels (relabelled session) |
| Resume_Paper | 38·3.2 | 30·3.7 | – | 67·3.6 s L0 | 14/33 | 73/24 | weak imprint; June id was "paper" |
| Leather_1_Back | 38·1.8 o★ | 30·1.8 | – | 68·1.5 | 61/50 | 99/98 | A session odd |
| Leather_1_Front | 38·2.1 o | 30·2.3 ★ | – | 68·2.9 | 85/84 | 100/100 | B session odd |
| Vinyl_1 | 38·2.7 | 30·2.0 ★ | – | 67·1.6 s | 73/82 | 81/60 | B session odd |
| Aluminum_Foil | 38·1.7 | 30·1.9 | – | 68·2.1 | 74/68 | 63/78 | B weak (40–63) |
| Glitter_Paper | 38·3.2 | 30·3.1 | – | 66·2.5 L0 | 97/94 | 55/70 | partial recovery |
| Bubble / Cardboard / Pleather | 38 | 30 | – | 67–68 | 93–100 | 77–99 | clean |
| Wood | – | – | 68·3.4 (S1, box) | 68·2.4 b L0 (S3) | – | 23/10 | geometry change (VERIFIED) |
| Ceramic | – | – | 68·4.0 b (S2) | 68·4.0 b (S3) | – | 0/1 | July speckle gone, anisotropy 9.1→4.7 (SUSPECTED) |
| ABS / Cork / FiberGlass | – | – | 68 (S2) | 68 (S3) | – | 11–62 | one session per hardware state |

**Protocol, from the logs (VERIFIED):**
- **Force:** the same in every campaign except D foam, which ran at 0.91/1.82 N instead of 0.5/1.01 N.
- **Speed:** A used 50/70/100 mm/s plus mixed strokes, B only 10/25/35, and C and D all six. A and B share no speed.
- **Stroke and recording:** stroke geometry, codec (FMP4) and resolution are the same everywhere. The soft stop is 6.0 N in A/B and 6.7 N in C/D. Eight June sessions ran at about 16 fps instead of 18.7.
- **Grasp height:** 0.31 m for the box materials and 0.49 m for ABS and ceramic in both campaigns. Wood moved from 0.315 m to 0.491 m.
- **Lubrication pauses per session:** A and B median 0 (range 0–3), C 0–3, D 13. Four D sessions logged none.
- **Operator:** a second operator recorded four A sessions. D was run by the owner through the scripted flow.
- **D only:** the Nano17 sits on a different counter-surface, 7 classes used the strict protocol, and grip starts at 69–95 % of nominal against 27–192 % in the archive.

**What changes together with the hardware:** for the 15 unit-1 materials, all of the above changes at the same step as the unit, along with three months of sample ageing and two specimen swaps. The bare-gel evidence for a sensor state does not depend on protocol, but the remaining loss of accuracy cannot be pinned on the hardware. Strict and normal D sessions score about the same (81 vs 84 %, excluding canvas, fleece and textured rubber; SUSPECTED, n = 12).

**Near-duplicate classes (VERIFIED, from nearest-centroid confusions on the stored embeddings in folds A and B):**
- **Weak-imprint cluster:** Polyethylene_Foam, Textured_Rubber (A/B), Resume_Paper, Wood (C), Fleece and Glitter_Paper have centroid cosines of 0.79–0.95. Foam is the attractor: 38/38 textured-rubber episodes in A go to foam, and June textured rubber versus foam has a cosine of 1.00.
- **Second cluster:** Aluminum_Foil, Bubble_Envelope and Cardboard (cosine 0.86–0.90).
- **The pairs the owner named are not the problem.** Leather front and back have cosine ≤ 0.5 and are rarely confused, and pleather is the most isolated class. Vinyl goes to Leather_1_Front (18–20 of 38).
- **Model-free:** crop contrast is 1.0–1.4 grey levels in every session and is dominated by inpainting residue and codec blocks. Texture is visible only for canvas, D silk and D textured rubber.

## 3. Per-material results
- **Fail in every fold, including the same unit (unlearnable or ambiguous):** Textured_Rubber (0–9 %) and Resume_Paper (4–44 % on the same unit; 24–73 % in D, unstable). Foam is unstable in folds A and B (14–93 %).
- **Fail only across the hardware change:** Canvas drops from 99 % to 0 % and Fleece from 71–90 % to 1–3 %, under every reading, in both models and in the reversed fold. Trained on June only, the jitter CNN scores both at 100 % on July and 0 % on D while most classes transfer. State subtraction recovers every other unit-1 class; Glitter only partly (55–70 %).
- **Cross-gel group (no same-hardware reference):** ceramic is 0–2 % in both directions. Wood is 0–23 %, ABS 11–44 %, fiberglass 10–48 % and cork 50–88 %.
- **Odd sessions within unit 1:** A Leather_1_Back (11–53 % against 69–89 % for B), B Leather_1_Front (69–72 %), B Vinyl (50–67 %) and B Aluminum_Foil (40–63 %). The odd spectra (★) predict these, so the cause is physical, not the hardware.

## 4. Test-set soundness
- **No file-level leakage (VERIFIED).** Cache labels match the raw folders for all 2,701 episodes, the exclusions are applied, and session stamps never overlap between campaigns.
- **One session per class.** Each per-class D accuracy is a single draw, so the effective n is 20 (CI about ±13 points). Anything specific to a session becomes a class error. Canvas D, for example, was the first D session and its held grip crept up to 6.6 N.
- **Textured rubber D:** 9 of 68 episodes, all at f4.0, have no usable crop, so its test set leans toward low force.
- **Foam D:** the relabel from EPDM holds (read as foam at 78–100 %), but it used EPDM's force levels.
- **Optimistic but not leakage:** all design choices were tuned on fold D. The own-session, campaign-mean and tracker estimates are transductive over the test session, which here is the class. The previous-session estimate is the clean one (51 / 57 %).
- **Exclude or stratify:** textured rubber D and EPDM D (specimen generalisation), wood D (geometry), and canvas D, fleece D and ceramic (suspected). In folds A and B, flag A Leather_1_Back, B Leather_1_Front and B Vinyl.

## 5. How much of the ceiling the data explain

| subset (D trk|state, p/j) | D | same-unit on its S1 classes |
|---|---|---|
| all 20 | 56.0 / 54.6 | 72.3 / 73.0 |
| without TR, EPDM, Wood (verified only, 17) | 60.1 / 60.0 | – |
| without TR, EPDM, Canvas, Fleece (16) | 65.1 / 64.3 | 74.3 / 76.5 |
| also without Wood, Ceramic (14) | 72.8 / 72.8 | 74.3 / 76.5 |
| 11 clean S1→S2 classes | 83.1 / 81.7 | 74.3 / 76.5 |

- Canvas and fleece alone account for 62–70 % of the summed per-class gap between same-unit and D across the 15 unit-1 classes. On average, the other 13 score at least as well on D as on the same unit.
- **Estimate:** the verified issues alone explain about 4–5 points. With the suspected ones included, they explain about 17 of the roughly 18-point gap.
- **This is an upper bound, for three reasons:**
  - Five of the six flagged classes are among D's six worst, and ABS (11–18 %) is not flagged.
  - The D-fold model was trained on two unit-1 sessions per material, while the A/B-fold models had one.
  - The same-unit reference is itself held down by the weak-imprint classes.

**A cleaned protocol, without collecting new data:**
1. **Labels:** drop Textured_Rubber from the 20-way task and report EPDM D as specimen generalisation, so 18-way becomes the main result. Optionally merge foam, textured rubber and resume paper into one weak-imprint class, and report both versions.
2. **Strata:** the 11 clean classes as the main cross-unit number; canvas and fleece separately with their evidence; cork, fiberglass, ABS and ceramic as the gel change; wood on its own.
3. **Matched controls:** train on A only and test on B versus D on the same 11 classes. Limit D test episodes to the training speed set and to matched force bins, and drop D foam or bin it by force.
4. **Covariates:** report lubrication pauses, the strict flag and the operator for each session.
5. **Ask the owner:** were D canvas, fleece, silk and ceramic the same pieces, side and orientation? Was June's "paper" the same sheet as resume paper? Was lubricant applied in June when no pause was logged?
