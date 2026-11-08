# Manuscript review checklists (neuromechanist/research-skills, plugins/manuscript/skills/paper-review/references — verbatim, EEG-specific items removed)

Review validity depends on independence: a reviewer that shares the conversation that produced the manuscript is biased toward it. Panel lenses: methods/design, statistics, novelty/significance, plus reproducibility for methods-heavy or hardware papers. Each reviewer reads the whole manuscript but weights its lens; a synthesis pass merges them into one Critical/Major/Minor review and surfaces genuine disagreement rather than averaging it away.

## Methodology Assessment Checklist
### Experimental Design
- [ ] Research question or hypothesis is clearly stated
- [ ] Study design is appropriate for the research question
- [ ] Controls or comparison conditions are adequate
- [ ] Sample size is justified (or acknowledged as a limitation)
- [ ] Inclusion/exclusion criteria are clearly stated and justified
- [ ] Randomization or counterbalancing is described (if applicable)
- [ ] Potential confounds are identified and addressed or acknowledged
### Data Collection
- [ ] Equipment and software versions are specified
- [ ] Acquisition parameters (sampling rates, resolution, calibration) are reported
- [ ] Data collection protocol is described in sufficient detail to replicate
### Statistical Analysis
- [ ] Statistical tests are named and justified
- [ ] Correct test variant is used (paired vs. unpaired)
- [ ] Multiple comparison correction is applied where many comparisons are made
- [ ] Effect sizes / spread are reported alongside point estimates
- [ ] Sample sizes per group are adequate for the claim made
- [ ] Confidence intervals are reported where appropriate
- [ ] No "double-dipping": features used for selection are independent of analysis targets
### Figures and Visualizations
- [ ] Axes are labeled with units; legends are present and complete
- [ ] Error bars are defined (SD, SE, CI)
- [ ] For small N (<5): individual data points are shown
- [ ] Figures match what is described in the text
- [ ] No figure duplicates information already in another figure or table
### Reproducibility
- [ ] Methods are detailed enough for independent replication
- [ ] Custom code is shared or code availability is addressed
- [ ] Data are shared or data availability is addressed
- [ ] For hardware papers: schematics or component lists are provided
- [ ] Software versions and key parameters are specified
### Literature and Context
- [ ] Literature review includes recent work (last 2-3 years)
- [ ] Competing methods or interpretations are cited
- [ ] Cited papers actually support the claims they are attached to
- [ ] The paper positions itself relative to existing work (novelty claim is supported)
- [ ] Limitations are discussed honestly

## Statistical red flags (from the statistical review guide)
- Subgroup analysis with insufficient power: conclusions drawn from groups of 2–3 items.
- Correlation driven by outliers or by a gap between two clusters.
- Double-dipping: the same data used for selection (threshold choice, model choice) and for evaluation.
- Reporting checklist per result: what was compared, how many runs, the spread, and whether the difference exceeds it.

## Figure Review Guide
- Axes labeled with units, readable fonts, all elements defined; self-contained captions (understandable without the main text); colorblind-safe palettes.
- Bar plots: N<5 → show individual points; N 5–9 → prefer points or boxes.
- Common issues: figure contradicts the text; redundant figures; missing statistical annotations; inappropriate scale (y-axis not at zero without justification); undeclared processing.
- Questions per figure: Does it accurately represent the data as described? Could a reader be misled by the presentation choice? Is this the most appropriate visualization? Would a different visualization reveal something hidden? Is the figure necessary?

## Output template
```
## Synopsis
[1 paragraph: what the paper does, methods, main findings, strengths, overall assessment, readiness and level of revision needed.]
## Critical Issues
[Would prevent publication in current form: fundamental methodological flaws, invalid approaches, claims that cannot be supported by the methods.]
1. [Cite location] [Problem, why it matters, constructive suggestion.]
## Major Concerns
[Require significant revision: incomplete analyses, missing comparisons, overreached conclusions, gaps that weaken but do not invalidate.]
## Minor Concerns
[Clarity, polish, completeness.]
## References
```
### Severity calibration
Critical: fundamental methodological flaw (design cannot test the hypothesis); invalid approach (double-dipping); claims unsupported by the methods used; lack of reproducibility (insufficient detail, no data/code for computational work).
Major: incomplete analysis (missing controls or comparisons); overreached conclusions; missing relevant literature that changes interpretation; inadequate sample size without acknowledgment; figures that misrepresent the data; important methodological details missing.
Minor: writing clarity; missing definitions or inconsistent terminology; redundant text or figures; citation completeness; axis labels, legends; minor statistical reporting issues.
