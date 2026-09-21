# OpsPilot: 4-Week Task Plan
*Core path: 20 tasks across 4 weeks at roughly 4 hours/week — this is a solo infrastructure/evaluation project, not a team capstone, so the cadence and task count are lighter than a Gradio-demo build. Phase 2 (optional; bottom) is a fine-tuning extension to pursue only after the core plan is done.*

## Week 1: SLM + Local Agent (5 tasks)
**Demo Goal:** A local agent (no GPU rented yet) that, given "checkout latency has increased, investigate," autonomously calls two or more tools against the synthetic dataset and produces an evidence-cited diagnosis — and a baseline scorecard comparing it to a larger hosted model on all 20 incidents.

| # | Task (~1 hr) | Definition of Done | Evidence of Completion |
|---|---|---|---|
| 1 | Set up a local 3–8B instruct model with reliable tool-calling support (e.g., a current Qwen3 4B/8B-class model) running locally | Model responds to a basic tool-calling test prompt with a valid structured tool call | Transcript of one successful structured tool call |
| 2 | Implement the six diagnostic tools (`get_metric`, `search_logs`, `get_deployment_history`, `get_kubernetes_status`, `get_service_dependencies`, `get_recent_alerts`) as mock functions | Each tool has a documented signature and returns realistic mock data matching requirements.md | `docs/tools.md` with all six signatures + one example call/response per tool |
| 3 | Build the synthetic incident dataset: ~20 incidents covering at least the 5 archetypes (bad deployment, DB saturation, memory leak, downstream dependency, traffic spike), each with a documented ground-truth cause, expected tool-call path, and key evidence | Dataset file committed with ground truth for every incident | Dataset file + a summary table of incident types/counts |
| 4 | Wire the agent loop: model decides which tool to call, the tool executes against the synthetic dataset, the result feeds back to the model, and it repeats until it reaches a diagnosis | Agent completes a full investigate→diagnose round trip without crashing, on at least 3 different incidents | Transcript of the "checkout latency" investigation matching the example trace in requirements.md (metric → deployment history → error rate → logs → diagnosis) |
| 5 | Run the Week 1 experiment: all ~20 incidents through your local SLM and a larger hosted model, scoring both on diagnosis correctness, tool selection, tool arguments, tool-call count, hallucinated facts, and answer quality | Baseline scorecard produced comparing both models across all six metrics | `week1_baseline.md`/csv with per-incident and aggregate scores |

## Week 2: GPU Deployment (5 tasks)
**Demo Goal:** The same agent now calls an SLM served through vLLM on a rented L4/A10 GPU instead of your laptop, reachable over HTTPS — plus a filled FP16 vs. INT8 vs. INT4 comparison table.

| # | Task (~1 hr) | Definition of Done | Evidence of Completion |
|---|---|---|---|
| 6 | Rent a single L4- or A10-class GPU instance; provision Docker + the NVIDIA Container Toolkit | `nvidia-smi` runs successfully inside a container on the rented instance | Log/screenshot of `docker run --gpus all nvidia-smi` output |
| 7 | Deploy the chosen SLM behind vLLM in FP16, exposing an OpenAI-compatible `/v1/chat/completions` endpoint | A `curl` request from your laptop reaches the GPU server and gets a valid completion | Curl command + response, plus the committed Docker run/compose config |
| 8 | Point the local agent loop at the remote vLLM endpoint instead of local inference; re-run the "checkout latency" investigation end-to-end | Full investigation trace reproduced against the remote SLM with correct tool calls and diagnosis | Transcript + measured end-to-end latency for that request |
| 9 | Quantize the same model to INT8 and INT4 and serve each via vLLM in turn | All three variants (FP16/INT8/INT4) reachable, each verified with a smoke-test completion | Three curl transcripts, one per precision |
| 10 | Run the Week 2 experiment: compare FP16 vs. INT8 vs. INT4 on VRAM, TTFT, tokens/sec, and diagnosis quality (reusing the Week 1 eval set) | Filled comparison table across the three precisions | `quantization_comparison.md` with the table and raw measurement logs |

## Week 3: Production API + Evaluation Harness (5 tasks)
**Demo Goal:** A FastAPI service exposing `/health`, `/investigate`, and `/chat`, instrumented with Prometheus metrics, plus a standalone evaluation harness scoring your SLM against a reference LLM on all 20 incidents across four axes.

| # | Task (~1 hr) | Definition of Done | Evidence of Completion |
|---|---|---|---|
| 11 | Build a FastAPI service separating the agent loop from inference: `/health` (checks vLLM liveness), `/investigate` (runs a full incident investigation), `/chat` (interactive Q&A) | All three endpoints respond correctly against the deployed vLLM backend | One example request/response per endpoint |
| 12 | Instrument the service with Prometheus metrics (`slm_requests_total`, `slm_request_latency_seconds`, `slm_tokens_generated_total`, `slm_tool_calls_total`, `slm_agent_failures_total`) plus per-request logging of prompt/completion tokens, TTFT, inference latency, GPU utilization/memory, tool calls, and total investigation time | `/metrics` endpoint exposes all five named metrics with real values after a test run | `/metrics` output after running several investigations |
| 13 | Build a standalone evaluation harness that runs all ~20 incidents through both your SLM (via `/investigate`) and a reference LLM, scoring diagnosis correctness, tool selection, tool arguments, and evidence grounding as four separate scores | Harness runs with one command and produces per-incident and aggregate scores on all four axes, for both models | `eval_harness/` script + a generated report |
| 14 | Specifically test evidence grounding: construct at least 2 adversarial incidents designed to tempt the model into diagnosing a cause it never actually queried, and confirm the evaluator catches it | Evaluator correctly flags at least one induced ungrounded diagnosis | Transcript of the failing case + the evaluator's flag |
| 15 | Run the full evaluation harness end-to-end on the GPU-served FP16 SLM and record it as the Week 3 baseline | Baseline report committed covering all 20 incidents across all four evaluation axes | `week3_eval_report.md`/csv |

## Week 4: Performance & Economics (5 tasks)
**Demo Goal:** A load-test report showing how the GPU-served SLM behaves under increasing concurrency, and a final cost-per-1,000-investigations comparison across FP16/INT8/INT4/a larger SLM/a hosted LLM that answers the core question with measured evidence.

| # | Task (~1 hr) | Definition of Done | Evidence of Completion |
|---|---|---|---|
| 16 | Load-test the `/investigate` endpoint at 1, 5, 10, 25, and 50 concurrent requests | For each concurrency level: requests/sec, p50/p95 latency, TTFT, tokens/sec, GPU utilization, VRAM, and failure rate recorded | `load_test_results.csv` covering all 5 concurrency levels |
| 17 | Plot concurrency vs. p95 latency and concurrency vs. throughput | Two charts committed showing the shape of the performance curve, including where it degrades | Chart images/notebook in the repo |
| 18 | Calculate cost per 1,000 investigations for your SLM configuration using the rented GPU's hourly rate and measured throughput at a realistic concurrency level | Documented, reproducible calculation with the formula shown (not a guess) | `cost_analysis.md` with the worked calculation |
| 19 | Run the same ~20-incident evaluation against a larger hosted LLM (API-based) and compute its cost per 1,000 investigations | Hosted-model row added to the comparison table with quality, latency, and cost figures | Updated comparison table including the hosted baseline |
| 20 | Assemble the final report: one table comparing SLM FP16 / SLM INT8 / SLM INT4 / a larger SLM / hosted LLM across quality, TTFT, tokens/sec, GPU, and cost per 1,000 investigations, with a clear conclusion on where (if anywhere) self-hosting makes economic sense | Final report answers the core question from requirements.md with measured evidence, not assumption | `FINAL_REPORT.md` with the completed comparison table and conclusion |

## Phase 2 (Optional Extension): Fine-Tune the SRE SLM
*Pursue only after Weeks 1–4 are complete. The question this phase answers: does fine-tuning improve SRE tool-use reliability enough to justify the added complexity?*

- Build a fine-tuning dataset of ~500–2,000 examples, each covering: incident → observation → tool selection → tool arguments → evidence → diagnosis → recommended remediation.
- Fine-tune the base SLM (the FP16 model from Week 2/3) on this dataset.
- Re-run the full Week 3 evaluation harness comparing the base SLM vs. the fine-tuned SLM on a held-out slice of the incident set, on all four evaluation axes.
- Re-run the Week 4 performance/cost analysis for the fine-tuned model and add it to the final comparison table.
- Write up the conclusion: did fine-tuning move the quality/cost frontier enough to justify the training time and infrastructure, given what Weeks 1–4 already established as the baseline?
