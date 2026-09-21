# OpsPilot: Requirements

**Industry:** DevOps / SRE (Self-Hosted SLM Infrastructure & Agentic Evaluation)

## 1. Core Question
> Can a 3–8B parameter small language model (SLM), running on a single rented GPU, reliably diagnose common production incidents and decide which diagnostic tools to invoke — at an economically viable cost?

This project is a research/infrastructure build, not a typical product build: the deliverable is a working agent *and* a report answering the core question with measured evidence, not an assumption.

## 2. Objective
Build OpsPilot, a self-hosted SLM-powered SRE agent that investigates production incidents in a simulated environment by autonomously calling diagnostic tools (metrics, logs, deployment history, Kubernetes status, service dependencies, alerts), reaches an evidence-grounded diagnosis, and is rigorously evaluated — first for tool-use correctness and diagnostic accuracy, then, once deployed on a rented GPU, for latency, throughput, and cost per investigation against quantized variants and a larger hosted model.

## 3. What This Project Is (and Isn't)
- A solo, self-paced infrastructure and evaluation project (~4 hours/week across a 4-week core plan), not a team capstone with a Gradio demo.
- No fine-tuning in the core plan. Build order: local inference → GPU inference → vLLM → API serving → agent loop → evaluation → performance testing → cost optimization → (optional Phase 2) fine-tuning. Fine-tuning first would spend the early weeks on training infrastructure instead of the stated objective: deployment and evaluation.
- The environment is fully simulated: a synthetic incident dataset stands in for real production telemetry. No real production system, customer data, or live Kubernetes cluster is touched.
- Renting a GPU is real, with a real hourly cost — that is a deliberate part of the experiment, not a simulated constraint. Start with an L4/A10-class GPU, not an H100; the point of Week 2 is learning to operate an inference server yourself, not maximizing raw performance.

## 4. Agent Architecture

```
                         User
                           |
                           v
                    +-------------+
                    |    SLM      |
                    |   Agent     |
                    +------+------+
                           |
             +-------------+--------------+
             v             v              v
        query_metrics   search_logs   deployments
             |             |              |
             +-------------+--------------+
                           v
                     Diagnosis
                           |
                           v
                    Recommended Action
```

The agent must *investigate* rather than answer immediately: it decides which tool to call, reads the result, and decides whether to call another tool or conclude — this iterative tool-use loop is the core workload being built and evaluated, not a single-shot chatbot response.

## 5. The Six Diagnostic Tools

| Tool | Inputs | Returns |
|---|---|---|
| `get_metric` | `service`, `metric`, `window` | current value, baseline value, unit |
| `search_logs` | `service`, `query`, `window` | representative matching log lines |
| `get_deployment_history` | `service`, `window` | recent deployments, version, time since deploy |
| `get_kubernetes_status` | `service`/`pod` | pod status, restarts, OOMKilled events |
| `get_service_dependencies` | `service` | upstream/downstream dependent services |
| `get_recent_alerts` | `service`, `window` | recently fired alerts for the service |

Example call and response:

```
get_metric(service="checkout", metric="p95_latency", window="30m")
-> { "current": 4.8, "baseline": 1.2, "unit": "seconds" }

search_logs(service="checkout", query="timeout", window="30m")
-> [ "...representative log lines..." ]
```

## 6. Synthetic Production Environment & Incident Dataset
The simulated environment contains: application logs, metrics, deployment history, Kubernetes status, service dependencies, and recent alerts. Build a dataset of roughly 20 incidents, covering at least these five archetypes:

| # | Archetype | Signature Evidence |
|---|---|---|
| 1 | Bad deployment | checkout latency ↑, 5xx ↑, `checkout-v17` deployed 12 min ago with a 35% error rate vs. 0.4% on `checkout-v16` |
| 2 | Database saturation | checkout latency ↑, CPU normal, DB connection pool at 98%, DB query latency ↑, no recent deployment |
| 3 | Memory leak | pod memory rising continuously, OOMKilled events, container restarts ↑ |
| 4 | Downstream dependency | checkout latency ↑, payment-service latency ↑, checkout CPU normal |
| 5 | Traffic spike | requests/sec ↑ 5x, CPU ↑, latency ↑, no deployment |

Each incident needs a documented ground truth: the correct root cause, the expected tool-call path to discover it, and the key evidence a correct diagnosis must cite. This is what makes the dataset a controlled evaluation environment rather than just flavor text.

## 7. Sample Interaction & Expected Investigation Behavior

**User:** "Checkout latency has increased significantly in the last 15 minutes. What's happening?"

**Expected agent behavior:** the agent investigates before answering — it does not guess from the prompt alone.

| Step | Agent Action | Result |
|---|---|---|
| 1 | `get_metric(checkout, p95_latency, 30m)` | p95 increased from 1.1s → 4.7s |
| 2 | `get_deployment_history(checkout, 30m)` | `checkout-v17` deployed 12 minutes ago |
| 3 | `get_metric(checkout-v17, error_rate, 15m)` | error rate 31% |
| 4 | `search_logs(checkout-v17, "timeout")` | matching timeout log lines |
| 5 | Diagnosis | "Likely regression introduced in checkout-v17" (evidence-cited, not asserted) |

Each of the five archetypes above should produce an analogous multi-step trace, discoverable only by querying tools — the agent is never handed the answer up front.

## 8. Constraints
- Synthetic incident dataset only; no live production, Kubernetes, or customer-data integration.
- No fine-tuning during the core 4-week plan; base instruct models only (e.g., a current Qwen3 4B/8B-class model or comparable 3–8B instruct model with reliable structured/tool-calling support).
- Everything runs in Docker; GPU serving goes through the NVIDIA Container Toolkit.
- Inference is served through vLLM behind an OpenAI-compatible API; the agent and the inference server are separate services (FastAPI agent layer calling a vLLM backend), not one monolith.
- GPU rental starts at L4/A10-class hardware; scaling to larger hardware is a later decision, not a default.

## 9. Evaluation Requirements
Every incident run must be scored on four separate axes — a good diagnosis for the wrong reason is a failure, not a pass:
1. **Diagnosis correctness** — did it identify the true root cause?
2. **Tool selection** — did it choose an appropriate diagnostic tool for the situation?
3. **Tool arguments** — did it query the right service, metric, and time window?
4. **Evidence grounding** — does the final diagnosis actually follow from evidence the agent retrieved during that investigation, not from evidence it never queried?

Every SLM configuration tested (FP16 / INT8 / INT4 / a larger SLM) must also be compared against a larger hosted reference model on the same 20 incidents, so quality differences are measured, not assumed.

## 10. Guardrail Requirements
- Must investigate before answering: the agent must call at least one diagnostic tool before producing a diagnosis; it must never answer from the prompt alone.
- Diagnosis must be evidence-grounded: every claim in the final diagnosis must be traceable to a tool result actually retrieved in that investigation (e.g., never claim "the database is overloaded" without having queried the database).
- Must not overstate confidence: when evidence is ambiguous or conflicting, the agent must report that rather than force a single confident diagnosis.
- Observability must track and expose, per request: prompt/completion tokens, TTFT, inference latency, GPU utilization/memory, tool-call count, and failures — instrumented as Prometheus metrics (`slm_requests_total`, `slm_request_latency_seconds`, `slm_tokens_generated_total`, `slm_tool_calls_total`, `slm_agent_failures_total`).
- Cost figures (cost per 1,000 investigations) must be derived from actual measured throughput at a stated concurrency level, never from an assumed or estimated number.

## 11. What You'll Learn
By the end, these should be explainable from direct experience, not from reading about them:
- **ML/LLM:** SLM architecture, tokenizer, context window, quantization, KV cache, batching
- **GPU:** VRAM, CUDA basics, GPU utilization, memory bandwidth, inference throughput
- **Inference:** transformers, vLLM, continuous batching, streaming, TTFT, tokens/sec
- **DevOps:** Docker, GPU containers, FastAPI, health checks, logging, Prometheus
- **Agentic AI:** tool calling, agent loops, structured outputs, tool-use evaluation, failure handling
- **SRE:** incident diagnosis, observability, metrics/logs, latency, error rates, deployment correlation
- **Economics:** GPU hourly cost, utilization, cost/request, self-hosting vs. API tradeoffs
