# Academic Paper Review Skill (bytedance/deer-flow, skills/public/academic-paper-review/SKILL.md — verbatim, install notes removed)

This skill produces structured, peer-review-quality analyses of academic papers. It follows established academic review standards used by top-tier venues (NeurIPS, ICML, ACL, Nature, IEEE) to provide rigorous, constructive, and balanced assessments. The review covers summary, strengths, weaknesses, methodology assessment, contribution evaluation, literature positioning, and actionable recommendations — all grounded in evidence from the paper itself.

## Review Methodology

### Phase 1: Paper Comprehension
Thoroughly read and understand the paper before forming any judgments.

Step 1.1 Identify paper metadata: title, venue/status, domain, paper type (empirical, theoretical, survey, position, systems).

Step 1.2 Deep reading pass:
1. Abstract & Introduction — identify the claimed contributions and motivation
2. Related Work — note how authors position their work relative to prior art
3. Methodology — understand the proposed approach in detail
4. Experiments / Results — examine datasets, baselines, metrics, and reported outcomes
5. Discussion & Limitations — note any self-identified limitations
6. Conclusion — compare concluded claims against actual evidence presented

Step 1.3 Key claims extraction. List the paper's main claims explicitly:
```
Claim 1: [Specific claim about contribution or finding]
Evidence: [What evidence supports this claim in the paper]
Strength: [Strong / Moderate / Weak]
Claim 2: [...]
```

### Phase 2: Critical Analysis

Step 2.1 Literature context search. Use web search to understand the research landscape:
- "[paper topic] state of the art [current year]"
- "[key method name] comparison benchmark"
- "[specific technique] limitations criticism"
- "survey [research area] recent advances"

Step 2.2 Methodology assessment:
| Criterion | Questions to Ask | Rating |
|---|---|---|
| Soundness | Is the approach technically correct? Are there logical flaws? | 1-5 |
| Novelty | What is genuinely new vs. incremental improvement? | 1-5 |
| Reproducibility | Are details sufficient to reproduce? Code/data available? | 1-5 |
| Experimental Design | Are baselines fair? Are ablations adequate? Are datasets appropriate? | 1-5 |
| Statistical Rigor | Are results statistically significant? Error bars reported? Multiple runs? | 1-5 |
| Scalability | Does the approach scale? Are computational costs discussed? | 1-5 |

Step 2.3 Contribution significance: Landmark (fundamentally changes the field) / Significant (clear improvement with solid evidence) / Moderate (incremental but valid) / Marginal (small gains, narrow applicability) / Below threshold (fundamental flaws, insufficient evidence).

Step 2.4 Strengths and weaknesses. For each: What (specific observation), Where (section/figure/table), Why it matters (impact on the paper's claims or utility).

### Phase 3: Review Synthesis — output template
```markdown
# Paper Review: [Title]
## Executive Summary
[2-3 paragraphs: core contribution, approach, main findings. State the overall assessment upfront.]
## Summary of Contributions
1. ...
## Strengths
### S1: [title]  [explanation with references to sections/figures/tables; why it is a strength]
## Weaknesses
### W1: [title]  [explanation with references; impact on the claims; how it could be addressed]
## Methodology Assessment
| Criterion | Rating (1-5) | Assessment |
## Questions for the Authors
## Minor Issues
## Literature Positioning
[How does this work relate to the state of the art? Are key related works cited? Are comparisons fair and comprehensive? What important related work is missing?]
## Recommendations
**Overall Assessment**: [Accept / Weak Accept / Borderline / Weak Reject / Reject]
**Confidence**: [High / Medium / Low] — [justification]
**Contribution Level**: [Landmark / Significant / Moderate / Marginal / Below threshold]
### Actionable Suggestions for Improvement
```

## Review Principles
- Always suggest how to fix it. Give credit where due. Be specific (exact sections, equations, figures, tables). Separate minor from major.
- ❌ "This paper is poorly written" → ✅ "Section 3.2 introduces notation X without formal definition, making ... Consider adding ..."
- Do NOT dismiss work based on author reputation. Evaluate the work on its own merits.

## Adaptation by paper type — Empirical: experimental design, baselines, statistical significance, ablations, reproducibility. Systems: architecture decisions, scalability evidence, real-world deployment.

## Common Pitfalls to Avoid
- ❌ Reviewing the paper you wish was written instead of the paper that was submitted
- ❌ Demanding additional experiments that are unreasonable in scope
- ❌ Penalizing the paper for not solving a different problem
- ❌ Being overly influenced by writing quality versus technical contribution
- ❌ Treating absence of comparison to your own work as a weakness
- ❌ Providing only a summary without critical analysis

## Quality Checklist
- [ ] Paper was read completely (not just abstract and introduction)
- [ ] All major claims are identified and evaluated against evidence
- [ ] At least 3 strengths and 3 weaknesses are provided with specific references
- [ ] The methodology assessment table is complete with ratings and justifications
- [ ] Questions for authors target genuine ambiguities, not rhetorical critiques
- [ ] Literature search was conducted to contextualize the contribution
- [ ] Recommendations are actionable and constructive
- [ ] The overall assessment is consistent with the identified strengths and weaknesses
- [ ] Minor issues are separated from major concerns
- Always disclose limitations of your review (e.g., "I could not verify X").
