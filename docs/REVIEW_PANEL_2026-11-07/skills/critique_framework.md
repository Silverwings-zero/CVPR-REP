# Paper Critique and Peer Review Framework (wentorai/research-plugins, paper-critique-framework + peer-review-guide — verbatim excerpts)

## The Three-Pass Review Method
### Pass 1: Orientation
Read title, abstract, introduction (first and last paragraphs), section headings and figure captions, conclusion. Answer: What is the main claim? What type of contribution (empirical, theoretical, system)? Within the venue's scope? Does the abstract accurately represent the content? Novel or incremental?
### Pass 2: Detailed Read
Annotate: ? = I don't understand this; ! = strong point; X = I disagree / see a problem; → = needs more evidence; ≈ = similar to existing work, check novelty.
Focus on claims vs. evidence (is every major claim supported by data?), methodology, experimental design (are baselines fair? are ablations sufficient?), figures and tables, writing quality.
### Pass 3: Verification
Check key references (do they say what the authors claim?); spot-check derivations; examine statistical claims; check for cherry-picking (only best runs? selected metrics?); look for missing baselines that should have been compared.

## Review Report Structure
```
## Summary (3-5 sentences)
## Strengths  - S1, S2, S3 with evidence from the paper
## Weaknesses (ordered by severity)  - W1 (Major): weakness + why it matters + suggestion to fix ... W3 (Minor) ...
## Questions for Authors  - Q1, Q2 (things the authors might be able to address)
## Detailed Comments  (page/line-specific comments, typos)
## Overall Assessment
Recommendation: [Strong Accept / Accept / Weak Accept / Borderline / Weak Reject / Reject / Strong Reject]
Confidence: [High / Medium / Low]
```

## Assessment Criteria
| Dimension | Questions | Weight |
|---|---|---|
| Novelty | Is the idea new? Beyond incremental? | High |
| Significance | Would this matter to the community? Does it advance the field? | High |
| Soundness | Are the methods correct? Are conclusions supported? | High |
| Clarity | Well-written? Can it be understood and reproduced? | Medium |
| Completeness | Related work covered? Experiments thorough? | Medium |
| Reproducibility | Could someone replicate this? Code/data available? | Medium |

## Calibration Guide
- Strong Accept: significant contribution, technically sound, well-written. Would be a highlight of the venue.
- Accept: solid contribution with minor issues. Advances the field.
- Weak Accept: has merit but notable weaknesses. Contribution is real but modest. Borderline for this venue; would be accepted at a less selective venue.
- Borderline: equal arguments for and against. Significant weaknesses offset by some novelty.
- Weak Reject: interesting direction but fundamental issues not addressed. Major revisions needed that likely require a new submission cycle.
- Reject: significant problems in novelty, soundness, or relevance.
- Strong Reject: fundamental flaws.

## Common Red Flags (peer-review-guide)
Statistical: multiple comparisons without correction; selective reporting (only positive results shown); missing spread (only point estimates); small sample with large claims.
Methodological: lack of control or baseline; data leakage in ML (test data used during training/validation/selection); confounding variables not addressed; circular reasoning.
Writing: claims not supported by the data presented; overclaiming in the title or abstract; missing limitations; insufficient detail for reproducibility.

## Pitfalls to Avoid
| Pitfall | Better approach |
|---|---|
| "The writing needs improvement" (vague) | Give 2–3 specific examples with suggested fixes |
| Rejecting for not solving YOUR problem | Evaluate the paper on its own stated goals |
| Demanding impossible experiments | Suggest feasible improvements within scope |
| Ignoring supplementary material | Note what the supplement is said to contain |
| Being harsh without being constructive | Every weakness should include a suggestion |

## Reviewing Empirical Papers
Are datasets described completely (size, source, splits, preprocessing)? Are baselines appropriate and fairly tuned? Statistical significance: error bars, multiple runs? Ablations: which components contribute to the gain?
