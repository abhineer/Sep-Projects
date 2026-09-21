# FraudPilot: 4-Week Task Plan
*Core path: 20 tasks across 4 weeks at roughly 4 hours/week — this is a solo infrastructure/evaluation project, not a team capstone, so the cadence and task count are lighter than a Gradio-demo build. Phase 2 (optional; bottom) is a fine-tuning extension to pursue only after the core plan is done.*

## Week 1: SLM + Local Agent (5 tasks)
**Demo Goal:** A local agent (no GPU rented yet) that, given "Alert ALRT-5521: investigate," autonomously calls two or more tools against the synthetic bank dataset and produces an evidence-cited risk assessment — and a baseline scorecard comparing it to a larger hosted model on all 20 alerts.

| # | Task (~1 hr) | Definition of Done | Evidence of Completion |
|---|---|---|---|
| 1 | Set up a local 3–8B instruct model with reliable tool-calling support (e.g., a current Qwen3 4B/8B-class model) running locally | Model responds to a basic tool-calling test prompt with a valid structured tool call | Transcript of one successful structured tool call |
| 2 | Implement the six investigative tools (`get_transaction_details`, `get_account_activity`, `get_customer_risk_profile`, `get_device_session_info`, `get_merchant_risk_data`, `get_recent_alerts`) as mock functions | Each tool has a documented signature and returns realistic mock data matching requirements.md | `docs/tools.md` with all six signatures + one example call/response per tool |
| 3 | Build the synthetic alert dataset: ~20 alerts covering at least the 5 archetypes (card-testing/bot attack, account takeover, legitimate travel purchase, synthetic identity, merchant-side compromise), each with a documented ground-truth outcome, expected tool-call path, and key evidence | Dataset file committed with ground truth for every alert | Dataset file + a summary table of archetype counts and outcomes |
| 4 | Wire the agent loop: model decides which tool to call, the tool executes against the synthetic dataset, the result feeds back to the model, and it repeats until it reaches a risk assessment | Agent completes a full investigate→assess round trip without crashing, on at least 3 different alerts | Transcript of the ALRT-5521 investigation matching the example trace in requirements.md (transaction → account activity → device/session → risk profile → prior alerts → assessment) |
| 5 | Run the Week 1 experiment: all ~20 alerts through your local SLM and a larger hosted model, scoring both on risk-assessment correctness, tool selection, tool arguments, tool-call count, hallucinated facts, and answer quality | Baseline scorecard produced comparing both models across all six metrics | `week1_baseline.md`/csv with per-alert and aggregate scores |

## Week 2: GPU Deployment (5 tasks)
**Demo Goal:** The same agent now calls an SLM served through vLLM on a rented L4/A10 GPU instead of your laptop, reachable over HTTPS — plus a filled FP16 vs. INT8 vs. INT4 comparison table.

| # | Task (~1 hr) | Definition of Done | Evidence of Completion |
|---|---|---|---|
| 6 | Rent a single L4- or A10-class GPU instance; provision Docker + the NVIDIA Container Toolkit | `nvidia-smi` runs successfully inside a container on the rented instance | Log/screenshot of `docker run --gpus all nvidia-smi` output |
| 7 | Deploy the chosen SLM behind vLLM in FP16, exposing an OpenAI-compatible `/v1/chat/completions` endpoint | A `curl` request from your laptop reaches the GPU server and gets a valid completion | Curl command + response, plus the committed Docker run/compose config |
| 8 | Point the local agent loop at the remote vLLM endpoint instead of local inference; re-run the ALRT-5521 investigation end-to-end | Full investigation trace reproduced against the remote SLM with correct tool calls and risk assessment | Transcript + measured end-to-end latency for that request |
| 9 | Quantize the same model to INT8 and INT4 and serve each via vLLM in turn | All three variants (FP16/INT8/INT4) reachable, each verified with a smoke-test completion | Three curl transcripts, one per precision |
| 10 | Run the Week 2 experiment: compare FP16 vs. INT8 vs. INT4 on VRAM, TTFT, tokens/sec, and risk-assessment quality (reusing the Week 1 eval set) | Filled comparison table across the three precisions | `quantization_comparison.md` with the table and raw measurement logs |

## Week 3: Production API + Evaluation Harness (5 tasks)
**Demo Goal:** A FastAPI service exposing `/health`, `/investigate`, and `/chat`, instrumented with Prometheus metrics and PII masking, plus a standalone evaluation harness scoring your SLM against a reference LLM on all 20 alerts across four axes.

| # | Task (~1 hr) | Definition of Done | Evidence of Completion |
|---|---|---|---|
| 11 | Build a FastAPI service separating the agent loop from inference: `/health` (checks vLLM liveness), `/investigate` (runs a full alert investigation), `/chat` (interactive analyst Q&A) | All three endpoints respond correctly against the deployed vLLM backend | One example request/response per endpoint |
| 12 | Instrument the service with Prometheus metrics (`slm_requests_total`, `slm_request_latency_seconds`, `slm_tokens_generated_total`, `slm_tool_calls_total`, `slm_agent_failures_total`) plus per-request logging of prompt/completion tokens, TTFT, inference latency, GPU utilization/memory, tool calls, and total investigation time — with account/card numbers masked in every log line | `/metrics` endpoint exposes all five named metrics with real values, and a log review confirms no unmasked account/card numbers appear | `/metrics` output + a sample masked log line after running several investigations |
| 13 | Build a standalone evaluation harness that runs all ~20 alerts through both your SLM (via `/investigate`) and a reference LLM, scoring risk-assessment correctness, tool selection, tool arguments, and evidence grounding as four separate scores | Harness runs with one command and produces per-alert and aggregate scores on all four axes, for both models | `eval_harness/` script + a generated report |
| 14 | Specifically test evidence grounding and the advisory-only guardrail: construct at least 2 adversarial alerts designed to tempt the model into fabricating evidence (e.g., an invented travel notice) or claiming it took action itself (e.g., "I've frozen the account"), and confirm the evaluator catches both failure modes | Evaluator correctly flags at least one induced ungrounded claim and one induced advisory-only violation | Transcripts of both failing cases + the evaluator's flags |
| 15 | Run the full evaluation harness end-to-end on the GPU-served FP16 SLM and record it as the Week 3 baseline | Baseline report committed covering all 20 alerts across all four evaluation axes | `week3_eval_report.md`/csv |

## Week 4: Performance & Economics (5 tasks)
**Demo Goal:** A load-test report showing how the GPU-served SLM behaves under increasing concurrency, and a final cost-per-1,000-investigations comparison across FP16/INT8/INT4/a larger SLM/a hosted LLM/manual analyst triage that answers the core question with measured evidence.

| # | Task (~1 hr) | Definition of Done | Evidence of Completion |
|---|---|---|---|
| 16 | Load-test the `/investigate` endpoint at 1, 5, 10, 25, and 50 concurrent requests | For each concurrency level: requests/sec, p50/p95 latency, TTFT, tokens/sec, GPU utilization, VRAM, and failure rate recorded | `load_test_results.csv` covering all 5 concurrency levels |
| 17 | Plot concurrency vs. p95 latency and concurrency vs. throughput | Two charts committed showing the shape of the performance curve, including where it degrades | Chart images/notebook in the repo |
| 18 | Calculate cost per 1,000 investigations for your SLM configuration using the rented GPU's hourly rate and measured throughput at a realistic concurrency level | Documented, reproducible calculation with the formula shown (not a guess) | `cost_analysis.md` with the worked calculation |
| 19 | Run the same ~20-alert evaluation against a larger hosted LLM (API-based) and compute its cost per 1,000 investigations; add a cited, documented estimate of manual analyst investigation cost as a sanity-check comparison | Hosted-model row and analyst-cost reference line added to your comparison table with quality, latency, and cost figures | Updated comparison table including the hosted baseline and analyst-cost reference |
| 20 | Assemble the final report: one table comparing SLM FP16 / SLM INT8 / SLM INT4 / a larger SLM / hosted LLM across quality, TTFT, tokens/sec, GPU, and cost per 1,000 investigations (with manual analyst cost as reference), with a clear conclusion on where (if anywhere) self-hosting makes economic sense for fraud-alert triage | Final report answers the core question from requirements.md with measured evidence, not assumption | `FINAL_REPORT.md` with the completed comparison table and conclusion |

## Phase 2 (Optional Extension): Fine-Tune the Fraud-Investigation SLM
*Pursue only after Weeks 1–4 are complete. The question this phase answers: does fine-tuning improve fraud-investigation tool-use reliability enough to justify the added complexity?*

- Build a fine-tuning dataset of ~500–2,000 examples, each covering: alert → observation → tool selection → tool arguments → evidence → risk assessment → recommended action.
- Fine-tune the base SLM (the FP16 model from Week 2/3) on this dataset.
- Re-run the full Week 3 evaluation harness comparing the base SLM vs. the fine-tuned SLM on a held-out slice of the alert set, on all four evaluation axes.
- Re-run the Week 4 performance/cost analysis for the fine-tuned model and add it to the final comparison table.
- Write up the conclusion: did fine-tuning move the quality/cost frontier enough to justify the training time and infrastructure, given what Weeks 1–4 already established as the baseline?
