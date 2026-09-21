# Sep-Projects — Agentic AI Capstone Ideas

Additional Agentic AI capstone project definitions, in the same format as the June cohort's `class-projects` repo: teams pick one project, commit 6-8 hours/week for 4 weeks, and build a demo-worthy agent covering prompt engineering, RAG, tools, MCP, memory, guardrails, caching, and observability.

## Projects

| Project | Industry |
|---|---|
| [SellerPulse](SellerPulse) | Retail / E-commerce — seller engagement agent |
| [CreditCoach](CreditCoach) | Finance — credit-building assistant |
| [ChangeRiskAdvisor](ChangeRiskAdvisor) | DevOps / SRE — pre-deployment change-risk advisor |
| [OpsPilot](OpsPilot) | DevOps / SRE — self-hosted SLM SRE agent (infra/eval project) |

> **Note:** OpsPilot is a solo infrastructure/evaluation project (~4 hrs/week, real GPU rental cost, 20 core tasks across 4 weeks + an optional fine-tuning Phase 2) rather than a team capstone — its `requirements.md`/`tasks.md` follow the same two-file format but not the team-capstone content described below (no Gradio UI, no 6-pager/PR-FAQ, no sample_data folder).

## Structure

Each project folder has:
- **`requirements.md`** — user persona and objective, sample queries with expected answers, constraints, and guardrail requirements (~1 page)
- **`tasks.md`** — a 34-task, 4-week plan (Amazon-style 6-pager and PR/FAQ deliverables in Week 1, Gradio UI from week 1, tools/MCP/memory, guardrails/caching, evals/observability), each task with a Definition of Done and evidence of completion, plus a weekly Demo Goal and an optional Stretch Goals section
- **`sample_data/`** — a small starter dataset (one xlsx with a few tables, one pdf as RAG source material) to bootstrap the Week 1 ingestion tasks

## How to use this repo

1. Self-organize into a team and pick a project.
2. Read that project's `requirements.md` in full before writing any code.
3. Work through `tasks.md` week by week — check off each task's Definition of Done and keep the listed evidence (logs, screenshots, reports) so the team can verify progress.
4. Hit the weekly Demo Goal before moving to the next week.
