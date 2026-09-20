# ChangeRiskAdvisor: 4-Week Task Plan
*Core path: 34 one-hour tasks; this is the safe, required build every team should be able to finish. Stretch Goals (bottom) are optional add-ons for teams with extra time.*

## Week 1: Foundations, RAG & UI (11 tasks)
**Demo Goal:** A live Gradio chat UI that answers a "how risky is this change?" question with a RAG-grounded assessment citing past incidents; no tools, memory, or guardrails yet, but it's clickable and shareable. Plus two written deliverables: an Amazon-style 6-pager and a PR/FAQ.

| # | Task (~1 hr) | Definition of Done | Evidence of Completion |
|---|---|---|---|
| 1 | Kickoff: assign roles, review requirements.md and Sam Okafor's persona/objective, agree on tech stack | Roles assigned (prompt/RAG, tools/MCP, memory, guardrails/caching, observability/UI owners); requirements.md read by everyone; stack agreed | A `docs/team.md` listing roles and stack, with each member confirming they've read requirements.md |
| 2 | Write an Amazon-style 6-pager for ChangeRiskAdvisor: narrative memo covering the problem, the customer (Sam), the solution, goals & non-goals, key risks/mitigations, and success metrics | 6-pager committed as a narrative document (no slides/bullets-only sections); every section from the standard format is present and specific to ChangeRiskAdvisor, not generic | `docs/6-pager.md` in repo, reviewed and agreed on by the whole team |
| 3 | Write a PR/FAQ for ChangeRiskAdvisor: a mock press release announcing the launch, plus an FAQ covering customer questions and internal/guardrail questions | PR/FAQ committed; press release is written from the customer's (Sam's) point of view; FAQ has at least 5 questions, including at least one on data handling and one on the "advisory only, never auto-approves" guardrail | `docs/pr-faq.md` in repo, reviewed and agreed on by the whole team |
| 4 | Set up the git repository: initialize repo, agree on branch strategy, add .gitignore, write a README | Repo exists remotely with main + feature branches; README lets a fresh clone run the project | A teammate clones the repo and runs it successfully from README alone |
| 5 | Draft the system prompt: risk-advisor tone, "advisory only, never approve/block" rule, "cite evidence for every score" rule | Prompt file committed; 2 manual test prompts confirm the agent never issues an approval and always cites a source for its assessment | Prompt file in repo + pasted transcript of the 2 test runs |
| 6 | Generate a synthetic dataset of past change-related incidents/postmortems, service dependency graphs, and system health snapshots | Dataset file committed covering several services with multiple past incidents each | Dataset file in repo + a summary count of services/incidents |
| 7 | Prepare the RAG corpus: past change-related postmortems, runbooks, and incident summaries | Corpus covers all 6 sample queries in requirements.md, especially the checkout-service risk assessment and the similar-incident lookup | Corpus files committed with document count |
| 8 | Build the ingestion pipeline: chunk and embed the corpus into a vector store | Pipeline runs with no errors; vector store has the expected chunk count | Console log showing chunk/embedding count |
| 9 | Implement retrieval and test against "how risky is this change to checkout-service config?" | Relevant past-incident chunk(s) appear in the top-3 retrieved results | Logged query + retrieved chunks with a correct/incorrect judgment |
| 10 | Wire a minimal prototype: change description → grounded risk assessment (no tools yet) | Full query→assessment round trip runs without crashing and reflects the corpus data | Terminal/notebook transcript of one successful run |
| 11 | Build a Gradio chat UI for the prototype and deploy it locally with a shareable link | Gradio app launches and returns a grounded risk assessment for a real query | Screenshot of the running UI + shareable link posted to the team channel |

## Week 2: Tools, MCP & Memory (7 tasks)
**Demo Goal:** The same Gradio UI now checks live system health/freeze-window status and dependency graphs, and remembers the team's high-risk service list across two visits; visible live in the chat.

| # | Task (~1 hr) | Definition of Done | Evidence of Completion |
|---|---|---|---|
| 12 | Design tool specs: `check_system_health(service_name)` (health, active incidents, freeze-window status) and `get_dependency_graph(service_name)` (upstream/downstream dependents) | Written spec for both tools: inputs, outputs, error cases | `docs/tools.md` with both signatures and example input/output |
| 13 | Implement the system-health tool | Returns correct health/freeze status for a known service and a clear error for an unknown one | Test log showing both cases |
| 14 | Implement the dependency-graph tool | Returns the correct dependent-service list for a known service and a clear error for an unknown one | Test log showing both cases |
| 15 | Set up MCP to expose both tools to the agent; test a full round trip | Agent calls both tools via MCP and uses their results in a live response | Trace/log of one query showing the response built from tool output |
| 16 | Design the memory schema: high-risk service list and freeze-window policy notes | Schema documented; a record can be written and read back correctly | Schema doc + log of one record written and retrieved |
| 17 | Integrate memory; test risk-appetite recall (e.g., "checkout-service is always high-risk") across 2 sessions | Preference stated in session 1 is correctly recalled, unprompted, in session 2 | Transcripts of both sessions showing the preference and its recall |
| 18 | Wire tools and memory into the Gradio UI via an expandable "agent trace" panel | Panel lists each tool call and the recalled risk-appetite settings for the response | Screenshot of the panel expanded on a real query |

## Week 3: Guardrails & Caching (7 tasks)
**Demo Goal:** In the live UI, show the agent decline to approve a change and instead complete the risk assessment, and show a visible speed-up (cache hit badge) on a repeated dependency-graph query.

| # | Task (~1 hr) | Definition of Done | Evidence of Completion |
|---|---|---|---|
| 19 | Codify guardrail rules: no auto-approve/block/merge/deploy, no unsupported risk claims, no fabricated freeze/dependency data | Rules written as a checklist mapped to requirements.md's guardrail section | `docs/guardrails.md` listing each rule with its requirements.md reference |
| 20 | Implement guardrail checks verified against live tool output and the incident corpus | Every risk assessment passes through the guardrail check (evidence-cited, advisory-only) before reaching the user | Log entry showing a response being labeled/filtered by the guardrail layer |
| 21 | Test guardrails against "just approve this change for me" and an unsupported-claim probe (asking for a risk score with no evidence available) | Approval request is correctly declined with an explanation; assessment without evidence is correctly flagged as unconfirmed rather than guessed | Transcripts of both test runs |
| 22 | Implement caching for RAG embeddings and frequent dependency-graph/health-check queries | Repeated identical queries hit the cache instead of re-querying | Log showing a cache miss then a cache hit on the repeat |
| 23 | Measure cache hit rate and latency improvement | Latency compared for cached vs. uncached calls with documented improvement | Before/after latency numbers committed to the repo |
| 24 | Run all 6 sample queries from requirements.md end-to-end; fix bugs | All 6 run and are compared against the expected-answers table | Filled-in expected-answers table with actual output and pass/fail per row |
| 25 | Surface guardrail status (advisory-only reminder) and cache hit/miss as visible badges in the Gradio UI | UI visibly shows the advisory disclaimer and cache hits | Screenshots showing both badge states |

## Week 4: Observability, Evals & Demo Readiness (9 tasks)
**Demo Goal:** Full live walkthrough: Gradio UI + observability dashboard, an eval score shown before/after your error-analysis fixes, and an approval-request refusal on demand.

| # | Task (~1 hr) | Definition of Done | Evidence of Completion |
|---|---|---|---|
| 26 | Instrument observability: log retrievals, tool calls, guardrail triggers, and tool failures | Every event for one request shares a single trace ID | Exported trace for one request showing all event types tied together |
| 27 | Build an eval harness from the expected-answers table with pass/fail scoring | Each of the 6 rows is an automated test case with a scorer | Eval script committed, runnable with one command |
| 28 | Run the eval suite against the synthetic change/incident data; record baseline scores | Suite runs successfully and produces a baseline score | Saved baseline report (score, timestamp, per-case pass/fail) |
| 29 | Do error analysis: categorize failures, find root causes, pick top 3 fixes | Every failing case is categorized (retrieval miss, tool error, guardrail miss, unsupported claim, latency) with a root cause and prioritized fix | Error-analysis table committed |
| 30 | Apply the top fixes and re-run the eval suite; record the improvement | Score improves measurably over baseline after the fixes | Before/after eval report showing the score delta |
| 31 | Build a dashboard: tool-call failure rate, guardrail trigger count, high-risk-flag hit rate | Dashboard shows real data and is reachable from the UI | Screenshot/link of the live dashboard with real run data |
| 32 | Handle edge cases: health/dependency API timeout, ambiguous "this change" references, no matching corpus incident | Each edge case produces a graceful fallback instead of a crash | Log/transcript of each edge case being triggered and handled |
| 33 | Prepare the demo script: Sam persona, 2-3 live queries, a memory demo, the scorecard | Script covers all elements and is timed to the demo slot | Script document + timed rehearsal note |
| 34 | Final rehearsal, deploy the demo build, record a backup demo video | Live demo runs end-to-end without failure; build deployed and reachable; backup video exists | Deployment link + backup video link, both in README |

## Stretch Goals (optional; the core path above is the safe, required build)
- Baseline comparison: run the same requests through a vanilla LLM with no RAG/tools/guardrails, and show side-by-side why grounded, evidence-cited risk assessment matters.
- Red-team your own agent: try to get it to issue an approval or a risk score without evidence anyway (misleading phrasing, indirect requests), then harden the guardrail against what worked.
- Add a "blast radius" visualization rendering the dependency graph for the service under change.
- Set and hit a latency/cost budget (e.g., under 3s and under $0.01/query) and show the before/after numbers.
- (Add your own ideas here as the team comes up with them.)
