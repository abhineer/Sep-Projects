# CreditCoach: 4-Week Task Plan
*Core path: 34 one-hour tasks; this is the safe, required build every team should be able to finish. Stretch Goals (bottom) are optional add-ons for teams with extra time.*

## Week 1: Foundations, RAG & UI (11 tasks)
**Demo Goal:** A live Gradio chat UI that answers a user's "why did my score drop?" question with a RAG-grounded explanation; no tools, memory, or guardrails yet, but it's clickable and shareable. Plus two written deliverables: an Amazon-style 6-pager and a PR/FAQ.

| # | Task (~1 hr) | Definition of Done | Evidence of Completion |
|---|---|---|---|
| 1 | Kickoff: assign roles, review requirements.md and Jordan Alvarez's persona/objective, agree on tech stack | Roles assigned (prompt/RAG, tools/MCP, memory, guardrails/caching, observability/UI owners); requirements.md read by everyone; stack agreed | A `docs/team.md` listing roles and stack, with each member confirming they've read requirements.md |
| 2 | Write an Amazon-style 6-pager for CreditCoach: narrative memo covering the problem, the customer (Jordan), the solution, goals & non-goals, key risks/mitigations, and success metrics | 6-pager committed as a narrative document (no slides/bullets-only sections); every section from the standard format is present and specific to CreditCoach, not generic | `docs/6-pager.md` in repo, reviewed and agreed on by the whole team |
| 3 | Write a PR/FAQ for CreditCoach: a mock press release announcing the launch, plus an FAQ covering customer questions and internal/guardrail questions | PR/FAQ committed; press release is written from the customer's (Jordan's) point of view; FAQ has at least 5 questions, including at least one on data handling and one on guardrails/predatory-product policy | `docs/pr-faq.md` in repo, reviewed and agreed on by the whole team |
| 4 | Set up the git repository: initialize repo, agree on branch strategy, add .gitignore, write a README | Repo exists remotely with main + feature branches; README lets a fresh clone run the project | A teammate clones the repo and runs it successfully from README alone |
| 5 | Draft the system prompt: financial-wellness tone, "never guarantee an outcome" rule, "never invent a figure" rule | Prompt file committed; 2 manual test prompts confirm the agent avoids guarantees and only cites tool-sourced figures | Prompt file in repo + pasted transcript of the 2 test runs |
| 6 | Generate a synthetic dataset of user credit-score histories, score-change factors, and account balances/limits | Dataset file committed covering several user profiles with a few months of score/account history each | Dataset file in repo + a summary count of profiles/months of history |
| 7 | Prepare the RAG corpus: credit-scoring factor explanations, financial-literacy content, and product-risk descriptions (e.g., payday loans, credit-repair scams) | Corpus covers all 6 sample queries in requirements.md, especially the score-drop explanation and the payday-loan risk case | Corpus files committed with document count |
| 8 | Build the ingestion pipeline: chunk and embed the corpus into a vector store | Pipeline runs with no errors; vector store has the expected chunk count | Console log showing chunk/embedding count |
| 9 | Implement retrieval and test against "why did my credit score drop 20 points?" | Relevant scoring-factor chunk(s) appear in the top-3 retrieved results | Logged query + retrieved chunks with a correct/incorrect judgment |
| 10 | Wire a minimal prototype: user question → grounded explanation (no tools yet) | Full query→explanation round trip runs without crashing and reflects the corpus data | Terminal/notebook transcript of one successful run |
| 11 | Build a Gradio chat UI for the prototype and deploy it locally with a shareable link | Gradio app launches and returns a grounded explanation for a real query | Screenshot of the running UI + shareable link posted to the team channel |

## Week 2: Tools, MCP & Memory (7 tasks)
**Demo Goal:** The same Gradio UI now pulls live score-factor and account data, and remembers the user's target-score goal across two visits; visible live in the chat.

| # | Task (~1 hr) | Definition of Done | Evidence of Completion |
|---|---|---|---|
| 12 | Design tool specs: `get_score_history(user_id, period)` and `get_account_summary(user_id)` | Written spec for both tools: inputs, outputs, error cases | `docs/tools.md` with both signatures and example input/output |
| 13 | Implement the score-history tool | Returns correct score data points and factor changes for a known period and a clear error for an invalid one | Test log showing both cases |
| 14 | Implement the account-summary tool | Returns correct balances, limits, and utilization ratio for a known user and a clear error for an unknown one | Test log showing both cases |
| 15 | Set up MCP to expose both tools to the agent; test a full round trip | Agent calls both tools via MCP and uses their results in a live response | Trace/log of one query showing the response built from tool output |
| 16 | Design the memory schema: target score, target date, and financial goal/purpose | Schema documented; a record can be written and read back correctly | Schema doc + log of one record written and retrieved |
| 17 | Integrate memory; test goal recall (e.g., "saving for a car, target 720 by next year") across 2 sessions | Goal stated in session 1 is correctly recalled, unprompted, in session 2 | Transcripts of both sessions showing the goal and its recall |
| 18 | Wire tools and memory into the Gradio UI via an expandable "agent trace" panel | Panel lists each tool call and the recalled goal for the response | Screenshot of the panel expanded on a real query |

## Week 3: Guardrails & Caching (7 tasks)
**Demo Goal:** In the live UI, show the agent refuse to endorse a payday loan and decline to guarantee a score outcome, and show a visible speed-up (cache hit badge) on a repeated factor-lookup query.

| # | Task (~1 hr) | Definition of Done | Evidence of Completion |
|---|---|---|---|
| 19 | Codify guardrail rules: no guaranteed outcomes, no predatory-product endorsements, no fabricated figures | Rules written as a checklist mapped to requirements.md's guardrail section | `docs/guardrails.md` listing each rule with its requirements.md reference |
| 20 | Implement guardrail checks verified against live tool output and the risk-content corpus | Every projection and every product-related answer passes through the guardrail check before reaching the user | Log entry showing a response being reframed/filtered by the guardrail layer |
| 21 | Test guardrails against "should I take out this payday loan?" and "can you guarantee 720?" | Both are correctly refused/reframed with an explanation and safer alternative where applicable; a benign query is not falsely blocked | Transcripts of both test runs |
| 22 | Implement caching for RAG embeddings and frequent score-factor lookups | Repeated identical queries hit the cache instead of re-querying | Log showing a cache miss then a cache hit on the repeat |
| 23 | Measure cache hit rate and latency improvement | Latency compared for cached vs. uncached calls with documented improvement | Before/after latency numbers committed to the repo |
| 24 | Run all 6 sample queries from requirements.md end-to-end; fix bugs | All 6 run and are compared against the expected-answers table | Filled-in expected-answers table with actual output and pass/fail per row |
| 25 | Surface guardrail status and cache hit/miss as visible badges in the Gradio UI | UI visibly shows guardrail refusals/reframes and cache hits | Screenshots showing both badge states |

## Week 4: Observability, Evals & Demo Readiness (9 tasks)
**Demo Goal:** Full live walkthrough: Gradio UI + observability dashboard, an eval score shown before/after your error-analysis fixes, and a predatory-product refusal on demand.

| # | Task (~1 hr) | Definition of Done | Evidence of Completion |
|---|---|---|---|
| 26 | Instrument observability: log retrievals, tool calls, guardrail triggers, and tool failures | Every event for one request shares a single trace ID | Exported trace for one request showing all event types tied together |
| 27 | Build an eval harness from the expected-answers table with pass/fail scoring | Each of the 6 rows is an automated test case with a scorer | Eval script committed, runnable with one command |
| 28 | Run the eval suite against the synthetic user data; record baseline scores | Suite runs successfully and produces a baseline score | Saved baseline report (score, timestamp, per-case pass/fail) |
| 29 | Do error analysis: categorize failures, find root causes, pick top 3 fixes | Every failing case is categorized (retrieval miss, tool error, guardrail miss, fabricated figure, latency) with a root cause and prioritized fix | Error-analysis table committed |
| 30 | Apply the top fixes and re-run the eval suite; record the improvement | Score improves measurably over baseline after the fixes | Before/after eval report showing the score delta |
| 31 | Build a dashboard: tool-call failure rate, guardrail trigger count, goal-recall accuracy | Dashboard shows real data and is reachable from the UI | Screenshot/link of the live dashboard with real run data |
| 32 | Handle edge cases: score/account API timeout, ambiguous "this loan" references, no matching corpus content | Each edge case produces a graceful fallback instead of a crash | Log/transcript of each edge case being triggered and handled |
| 33 | Prepare the demo script: Jordan persona, 2-3 live queries, a memory demo, the scorecard | Script covers all elements and is timed to the demo slot | Script document + timed rehearsal note |
| 34 | Final rehearsal, deploy the demo build, record a backup demo video | Live demo runs end-to-end without failure; build deployed and reachable; backup video exists | Deployment link + backup video link, both in README |

## Stretch Goals (optional; the core path above is the safe, required build)
- Baseline comparison: run the same requests through a vanilla LLM with no RAG/tools/guardrails, and show side-by-side why grounded, guardrailed credit guidance matters.
- Red-team your own agent: try to get it to endorse a predatory product or guarantee a score outcome anyway (misleading phrasing, indirect requests), then harden the guardrail against what worked.
- Add a "what-if" simulator: let the user model how paying down a balance or opening a new account might affect utilization (educational estimate only, clearly labeled as non-predictive).
- Set and hit a latency/cost budget (e.g., under 3s and under $0.01/query) and show the before/after numbers.
- (Add your own ideas here as the team comes up with them.)
