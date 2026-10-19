# DiagPilot: 4-Week Task Plan
*Core path: 34 one-hour tasks; this is the safe, required build every team should be able to finish. Stretch Goals (bottom) are optional add-ons for teams with extra time.*

## Week 1: Foundations, RAG & UI (11 tasks)
**Demo Goal:** A live Gradio chat UI that answers "what does this DTC / failure pattern mean and how do I triage it?" with a RAG-grounded, cited explanation; no tools, memory, or guardrails yet, but it's clickable and shareable. Plus two written deliverables: an Amazon-style 6-pager and a PR/FAQ.

| # | Task (~1 hr) | Definition of Done | Evidence of Completion |
|---|---|---|---|
| 1 | Kickoff: assign roles, review requirements.md and Rohan's persona/objective, agree on tech stack | Roles assigned (prompt/RAG, tools/MCP, memory, guardrails/caching, observability/UI owners); requirements.md read by everyone; stack agreed | `docs/team.md` listing roles and stack, with each member confirming they've read requirements.md |
| 2 | Write an Amazon-style 6-pager for DiagPilot: problem, customer (Rohan), solution, goals & non-goals, risks/mitigations, success metrics (e.g., triage minutes saved, hypothesis accuracy) | Narrative document (no bullets-only sections); every section specific to DiagPilot | `docs/6-pager.md` reviewed and agreed on by the team |
| 3 | Write a PR/FAQ: press release from the engineer's point of view plus an FAQ | At least 5 FAQs, including one on confidential-data handling and one on safety-decision boundaries | `docs/pr-faq.md` reviewed and agreed on by the team |
| 4 | Set up the git repository: branch strategy, .gitignore, README | A teammate clones and runs it from README alone | Successful fresh-clone run |
| 5 | Draft the system prompt: expert-peer tone, "cite every claim", "hypotheses not verdicts", "no safety sign-off" | Prompt file committed; 2 manual test prompts show citations and no overclaiming | Prompt file + transcript of the 2 runs |
| 6 | Load and extend the synthetic dataset (runs, DTC log, signal summaries, SW releases, known issues) | At least 6 runs across 2 variants and 3 SW versions, including one clean A/B regression pair and one decoy (non-regression) failure | Dataset committed + summary counts |
| 7 | Prepare the RAG corpus: diagnostic reference, triage procedures, regression guidance, safety boundaries, change-control, report template | Corpus covers all 6 sample queries in requirements.md | Corpus files committed with document count |
| 8 | Build the ingestion pipeline: chunk and embed into a vector store, keeping source/section metadata for citations | Runs with no errors; chunk count as expected; each chunk carries a source reference | Console log with chunk/embedding count |
| 9 | Implement retrieval and test with "RUN-101 failed cold start, DTC P0335" | Position-sensor triage chunk(s) appear in top-3 | Logged query + retrieved chunks with correct/incorrect judgment |
| 10 | Wire a minimal prototype: question → cited, grounded explanation (no tools yet) | Round trip runs without crashing; answer cites corpus sources | Transcript of one successful run |
| 11 | Build a Gradio chat UI and deploy locally with a shareable link | App launches and returns a cited explanation for a real query | Screenshot + shared link |

## Week 2: Tools, MCP & Memory (7 tasks)
**Demo Goal:** The same UI now pulls real runs, DTCs, signals, release notes, and known issues for a failing run, and remembers the engineer's variant, baseline, and threshold across two sessions.

| # | Task (~1 hr) | Definition of Done | Evidence of Completion |
|---|---|---|---|
| 12 | Design tool specs: `get_run_details(run_id)`, `get_dtc_log(run_id)`, `get_signal_summary(run_id)`, `get_sw_release_notes(version)`, `search_known_issues(dtc, symptom)` | Written spec for each: inputs, outputs, error cases | `docs/tools.md` with signatures and examples |
| 13 | Implement run-details and DTC-log tools | Correct data for a known run; clear error for an unknown run | Test log with both cases |
| 14 | Implement signal-summary, release-notes, and known-issues tools | Correct results for known inputs; clear "no match" and error behavior | Test log with both cases per tool |
| 15 | Expose all tools through MCP; test a full round trip | Agent calls multiple tools via MCP and builds a response from them | Trace of one query showing tool calls and output |
| 16 | Design the memory schema: ECU variant, SW baseline, alert thresholds, report template | Schema documented; a record can be written and read back | Schema doc + write/read log |
| 17 | Integrate memory; test recall across 2 sessions (variant A, baseline 4.1.3, 9.5 V) | Context from session 1 is applied unprompted in session 2 | Transcripts of both sessions |
| 18 | Add an expandable "agent trace" panel to the UI | Panel lists each tool call, its arguments, and the recalled context | Screenshot of the panel on a real query |

## Week 3: Guardrails & Caching (7 tasks)
**Demo Goal:** Live in the UI: a grounded A/B regression analysis with cited evidence and labeled hypotheses, a refusal to sign off safety or flash a calibration, and a visible cache-hit badge on a repeated lookup.

| # | Task (~1 hr) | Definition of Done | Evidence of Completion |
|---|---|---|---|
| 19 | Codify guardrail rules as a checklist mapped to requirements.md section 5 | Every rule has an ID and a requirements reference | `docs/guardrails.md` |
| 20 | Implement guardrail checks: claim-to-source verification (every DTC/value in the answer exists in tool output), hypothesis labeling, safety/flash/clear-DTC action blocking, variant-mismatch warning | Every answer passes the checks before reaching the user | Log of a response being blocked or rewritten |
| 21 | Test guardrails with query 6 (ASIL + release approval + flash) and a benign query | Query 6 refused with the right routing and an alternative; the benign query is not falsely blocked | Transcripts of both |
| 22 | Implement caching for RAG embeddings and static lookups (release notes, known issues, reference chunks) | Repeated identical lookups hit the cache | Log of a miss then a hit |
| 23 | Measure cache hit rate and latency | Cached vs. uncached latency documented | Before/after numbers committed |
| 24 | Run all 6 sample queries end-to-end; fix bugs | Each compared with the expected-answers table | Filled table with actual output and pass/fail |
| 25 | Show guardrail status and cache hit/miss as UI badges | Both visibly shown | Screenshots of each badge state |

## Week 4: Observability, Evals & Demo Readiness (9 tasks)
**Demo Goal:** Full live walkthrough: UI + observability dashboard, an eval score before/after error-analysis fixes, and a safety-boundary refusal on demand.

| # | Task (~1 hr) | Definition of Done | Evidence of Completion |
|---|---|---|---|
| 26 | Instrument observability: log retrievals, tool calls, guardrail triggers, tool failures under one trace ID | All events for one request share a trace ID | Exported trace for one request |
| 27 | Build an eval harness from the expected-answers table with scorers (includes a check that every cited value matches tool output) | Each of the 6 rows is an automated test | Eval script runnable with one command |
| 28 | Run the suite and record a baseline | Baseline report with timestamp and per-case pass/fail | Saved report |
| 29 | Error analysis: categorize failures (retrieval miss, tool error, ungrounded value, overclaimed root cause, guardrail miss, latency), root causes, top 3 fixes | Every failure categorized with a prioritized fix | Error-analysis table |
| 30 | Apply the fixes and re-run | Measurable improvement over baseline | Before/after report |
| 31 | Build a dashboard: tool-failure rate, guardrail triggers, claim-grounding rate, memory-recall accuracy | Real run data, reachable from the UI | Screenshot/link |
| 32 | Handle edge cases: tool timeout, unknown run ID, mismatched-variant comparison, no known-issue match, conflicting evidence | Each produces a graceful fallback, not a crash or an invented answer | Log/transcript per case |
| 33 | Prepare the demo script: Rohan persona, 2-3 live queries (A/B regression, known issue, safety refusal), a memory demo, the scorecard | Script covers all and is timed | Script + rehearsal note |
| 34 | Final rehearsal, deploy the demo build, record a backup video | Live demo runs end-to-end; build reachable; backup video exists | Deployment link + video link in README |

## Stretch Goals (optional; the core path above is the safe, required build)
- Baseline comparison: run the same requests through a vanilla LLM with no RAG/tools/guardrails and show how often it invents DTCs or overclaims a root cause.
- Red-team your own agent: try to get it to approve a release, classify ASIL, or clear a DTC via indirect phrasing, then harden the guardrails.
- Add a CAN/UDS log parser tool that summarises a raw trace file into the signal-summary format.
- Add a multi-run clustering view that groups morning failures by probable cause before triage.
- (Add your own ideas here.)
