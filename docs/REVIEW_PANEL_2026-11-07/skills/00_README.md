# Review skills bundle (fetched 2026-11-07 from GitHub)

Each reviewer agent is assigned ONE primary skill (its procedure and output form) and may draw on the others. The files are verbatim or lightly trimmed copies of the public skill files.

| File | Source | Repo stars | What it gives |
|---|---|---|---|
| `neurips_form.md` | SakanaAI/AI-Scientist `ai_scientist/perform_review.py` (the NeurIPS reviewer form, reused by lingzhi227/agent-research-skills `self-review` and AgentLaboratory) | 12.5k (AI-Scientist) | the review form and score scales, the THOUGHT → JSON output template, the three reviewer personas, the reflection round |
| `deer_flow_academic_paper_review.md` | bytedance/deer-flow `skills/public/academic-paper-review/SKILL.md` | ~20k (deer-flow) | three-phase procedure: claims-vs-evidence extraction, literature positioning, methodology rating table, structured output, pitfalls checklist |
| `paperjury_personas.md` | Spark-To-Paper-Skills/paperjury (codex mirror u7079256/paperjury-codex) `references/reviewer-personas.md` | 1.2k | the "unflinching academic gatekeeper" core, the two-pass fatal-flaw diagnostic, the vision-venue (CVPR) style profile, the evidence-anchor rule (every weakness rests on a verbatim quote) |
| `manuscript_review_checklists.md` | neuromechanist/research-skills `plugins/manuscript/skills/paper-review/references/*` | — | methodology checklist, figure review guide, Critical/Major/Minor output template with severity calibration |
| `critique_framework.md` | wentorai/research-plugins `skills/research/paper-review/paper-critique-framework/SKILL.md` and `peer-review-guide/SKILL.md` | 310 | three-pass read, red flags (data leakage, selective reporting, overclaiming), the Strong Accept → Strong Reject calibration guide, pitfalls to avoid |

Also consulted but not bundled: agentscope-ai/openjudge `skills/academic-eval/01-paper-review` (1.1k stars; a Python pipeline, not a prompt: its stages are safety → correctness (objective errors) → review 1–6 → criticality → BibTeX check. We reuse its stage idea: look for objective errors (arithmetic, internal inconsistencies between numbers in different places) before judging quality).
