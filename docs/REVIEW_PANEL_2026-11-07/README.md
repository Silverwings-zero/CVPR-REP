# Simulated CVPR review panel, 2026-11-07

Six independent reviewer agents, each with a different angle and a different primary review skill fetched from public GitHub repositories (see `skills/00_README.md`), read `paper_as_reviewed.tex` (main text at commit 42c9e37 with every number macro resolved), the rendered pages and the figures. They did not see the supplement, earlier reviews or any authoring notes. `AC_meta_review.md` merges them, checks every quoted claim against the paper, and ranks the fixes with a tag (TEXT, FIGURE/TABLE, ANALYSIS, COMPUTE).

Scores (CVPR 1-6, confidence 1-5): R1 tactile expert 3/4, R2 domain adaptation 3/4, R3 statistics 4/4, R4 CVPR generalist 4/4, R5 presentation 4/4, R6 reproducibility 4/4. AC recommendation: borderline, leaning accept.

Confirmed after the panel: `fig/fig_confound.pdf` was stale (Oct 28, two curves from an earlier version of the experiments), and Table 2 row e doubled a minus sign in the template.

Fixed the same day (next commit): Fig. 3 replotted with the raw curve only; Table 2 row e now prints -0.8 [-2.0, 0.5] through reversed-difference macros (`\Xd...Neg`); "three quarters" (0.69) became "seven tenths" in Secs. 5 and 6; the related-work sentence now says SITR moved a classifier of 16 of its 20 objects.

## Applied 2026-11-08 (no compute)

Every TEXT, ANALYSIS and FIGURE/TABLE item of the consensus list and the single-reviewer "worth acting on" list was
applied from existing results in one pass (see the commit of this date): folds table with the 20 materials named,
"unit and gel (47)", a paired Delta column for every "+ reading" row, per-fold columns and the inductive absolute of
row i (60.1 vs 63.9 on the 28 later sessions), per-seed spread, where sigma and the detector constants were set,
the pre-registration outcome (three directional predictions, all held), "transductive" named, the view and headline
choice stated, SITR reading numbers per model, per-batch BatchNorm relabelled with batch size and AdaBN as the
shuffled variant, the gel-log sentence, first-after-change sessions counted, spliced detector rates on all four
datasets, Fig. 1 with interquartile bars and the legend outside, Fig. 3 with the raw and mean-subtracted curves.
The main text had no page slack, so about 55 lines were trimmed elsewhere; it still ends on page 8.

Not applied: the COMPUTE items (bare-gel estimate on fold D, whole-stream AdaBN, NOTE/LAME), the supplement's recipe
table and old-protocol relabelling (due with the supplement), and the framing disagreement, which is the authors' call.
