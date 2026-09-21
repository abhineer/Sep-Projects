# FraudPilot: Requirements

**Industry:** Finance (Bank Fraud & Transaction Monitoring — Self-Hosted SLM Infrastructure & Agentic Evaluation)

## 1. Core Question
> Can a 3–8B parameter small language model (SLM), running on a single rented GPU, reliably investigate a flagged bank transaction, decide which diagnostic tools to invoke, and produce an evidence-grounded risk recommendation — at a cost per investigation that beats manual analyst triage or a hosted LLM?

This is a research/infrastructure build, not a typical product build — the deliverable is a working agent *and* a report answering the core question with measured evidence, not an assumption. It is the same kind of project as [OpsPilot](../OpsPilot), applied to bank fraud/transaction-monitoring alert triage instead of production incident diagnosis.

## 2. Objective
Build FraudPilot, a self-hosted SLM-powered fraud-investigation agent that triages a flagged transaction alert in a simulated bank environment by autonomously calling investigative tools (transaction details, account activity/velocity, customer risk profile, device/session signals, merchant risk data, prior alert history), reaches an evidence-grounded risk recommendation, and is rigorously evaluated — first for tool-use correctness and diagnostic accuracy, then, once deployed on a rented GPU, for latency, throughput, and cost per investigation against quantized variants and a larger hosted model.

## 3. What This Project Is (and Isn't)
- A solo, self-paced infrastructure and evaluation project (~4 hours/week across a 4-week core plan), not a team capstone with a Gradio demo.
- No fine-tuning in the core plan. Build order: local inference → GPU inference → vLLM → API serving → agent loop → evaluation → performance testing → cost optimization → (optional Phase 2) fine-tuning.
- The environment is fully simulated: synthetic customers, accounts, transactions, and alerts stand in for real bank data. No real core-banking system, real customer PII, or live fraud-monitoring feed is touched.
- The agent is advisory only. It never freezes an account, blocks a card, or contacts a customer itself — every recommendation requires a human analyst or a separate downstream system to act, mirroring how real fraud-ops teams operate under regulatory and dispute-liability constraints.
- Renting a GPU is real, with a real hourly cost — that is a deliberate part of the experiment. Start with an L4/A10-class GPU, not an H100; Week 2 is about learning to operate an inference server yourself, not maximizing raw performance.

## 4. Agent Architecture

```
                        Alert
                          |
                          v
                   +-------------+
                   |    SLM      |
                   |   Agent     |
                   +------+------+
                          |
          +---------------+----------------+----------------+
          v               v                v                v
   get_transaction   get_account      get_customer      get_device_
     _details          _activity       _risk_profile      session_info
          |               |                |                |
          +---------------+----------------+----------------+
                          v
                   Risk Assessment
                          |
                          v
           Recommended Action (advisory only — human decides)
```

The agent must *investigate* rather than answer immediately: it decides which tool to call, reads the result, and decides whether to call another tool or conclude — this iterative tool-use loop is the core workload being built and evaluated, not a single-shot chatbot response.

## 5. The Six Investigative Tools

| Tool | Inputs | Returns |
|---|---|---|
| `get_transaction_details` | `transaction_id` | amount, merchant, MCC/category, channel (card-present/online), location, timestamp, status |
| `get_account_activity` | `account_id`, `window` | recent transaction count/velocity, average spend baseline, deviation from baseline |
| `get_customer_risk_profile` | `customer_id` | KYC risk tier, account tenure, watchlist flags, prior confirmed-fraud/false-positive history |
| `get_device_session_info` | `session_id` | device-seen-before flag, IP geolocation, VPN/proxy flag, distance from customer's home region |
| `get_merchant_risk_data` | `merchant_id` | merchant category, chargeback rate, recent fraud reports, risk score |
| `get_recent_alerts` | `account_id`, `window` | prior alerts for this account/customer and their confirmed outcomes |

Example call and response:

```
get_transaction_details(transaction_id="TXN-88213")
-> { "amount": 1240.00, "merchant": "QuickTech Electronics", "mcc": "5732",
     "channel": "card-not-present", "location": "Lagos, NG", "status": "approved" }

get_device_session_info(session_id="SESS-4471")
-> { "device_seen_before": false, "ip_geolocation": "Lagos, NG",
     "vpn_or_proxy_detected": true, "distance_from_home_region_km": 12500 }
```

## 6. Synthetic Bank Environment & Alert Dataset
The simulated environment contains: transactions, account activity/velocity, customer KYC/risk profiles, device/session signals, merchant risk data, and prior alert history. Build a dataset of roughly 20 alerts, covering at least these five archetypes:

| # | Archetype | Signature Evidence | Ground-Truth Outcome |
|---|---|---|---|
| 1 | Card-testing / bot attack | many small transactions across different merchants within minutes, new device, high velocity, several declines then one approval | Confirmed fraud |
| 2 | Account takeover (ATO) | new device, IP geolocation far from home region, VPN detected, large transaction shortly after a password/contact-info change | Confirmed fraud |
| 3 | Legitimate travel purchase | large one-off purchase at an unfamiliar merchant, but location matches a recent flight/hotel booking on file, no other risk flags | False positive |
| 4 | Synthetic identity / first-party fraud | new account, thin history, rapid credit-limit utilization, mismatched KYC details, no genuine spending pattern | Confirmed fraud |
| 5 | Merchant-side compromise | cluster of customers reporting fraud at the same merchant shortly after a breach; this customer's transaction matches the compromised-merchant pattern | Confirmed fraud (merchant-driven) |

Each alert needs a documented ground truth: the correct outcome (confirmed fraud / false positive / needs further verification), the expected tool-call path to discover it, and the key evidence a correct recommendation must cite.

## 7. Sample Interaction & Expected Investigation Behavior

**System/analyst prompt:** "Alert ALRT-5521: a $1,240 card-not-present transaction on account ACC-3390 was just flagged. Investigate."

**Expected agent behavior:** the agent investigates before answering — it does not guess from the alert alone.

| Step | Agent Action | Result |
|---|---|---|
| 1 | `get_transaction_details(TXN-88213)` | $1,240 card-not-present purchase at QuickTech Electronics, Lagos, NG |
| 2 | `get_account_activity(ACC-3390, 30d)` | customer's average transaction is $85; no prior international purchases |
| 3 | `get_device_session_info(SESS-4471)` | new device, VPN detected, IP in Lagos NG — 12,500 km from home region (Austin, TX) |
| 4 | `get_customer_risk_profile(customer)` | standard KYC tier, no prior confirmed fraud, no travel notice on file |
| 5 | `get_recent_alerts(ACC-3390, 90d)` | no prior alerts on this account |
| 6 | Risk Assessment | "High-risk pattern consistent with account takeover: new device, VPN-masked IP far from home region, transaction well outside spending baseline, no travel notice on file." **Recommended action:** hold transaction pending customer verification — advisory only, requires analyst/system action |

Each of the five archetypes above should produce an analogous multi-step trace, discoverable only by querying tools — the agent is never handed the ground-truth outcome up front.

## 8. Constraints
- Synthetic customer/transaction dataset only; no real core-banking, real customer PII, or live fraud-monitoring integration.
- No fine-tuning during the core 4-week plan; base instruct models only (e.g., a current Qwen3 4B/8B-class model or comparable 3–8B instruct model with reliable structured/tool-calling support).
- Everything runs in Docker; GPU serving goes through the NVIDIA Container Toolkit.
- Inference is served through vLLM behind an OpenAI-compatible API; the agent and the inference server are separate services (FastAPI agent layer calling a vLLM backend), not one monolith.
- GPU rental starts at L4/A10-class hardware; scaling to larger hardware is a later decision, not a default.
- All outputs are advisory: the agent never itself freezes an account, blocks a card, or contacts a customer — that action is simulated as a separate, explicit downstream step.

## 9. Evaluation Requirements
Every alert run must be scored on four separate axes — a correct-sounding recommendation for the wrong reason is a failure, not a pass:
1. **Risk-assessment correctness** — did it reach the true ground-truth outcome (confirmed fraud / false positive / needs further verification)?
2. **Tool selection** — did it choose an appropriate investigative tool for the situation?
3. **Tool arguments** — did it query the right account, customer, session, or time window?
4. **Evidence grounding** — does the final recommendation actually follow from evidence the agent retrieved during that investigation, not from evidence it never queried (e.g., a fabricated travel notice or an invented customer statement)?

Every SLM configuration tested (FP16 / INT8 / INT4 / a larger SLM) must also be compared against a larger hosted reference model on the same 20 alerts, so quality differences are measured, not assumed.

## 10. Guardrail Requirements
- Advisory only: the agent must never itself freeze, block, or clear an account or transaction — it always recommends, with an explicit statement that a human analyst or downstream system must decide and act.
- Evidence-grounded: every factual claim in the risk assessment must be traceable to a tool result actually retrieved in that investigation; must never fabricate a customer statement, travel notice, or history that no tool call returned.
- Must not overstate certainty: when evidence is ambiguous or conflicting, the agent must report "needs further verification" rather than force a confident fraud/not-fraud call.
- Must minimize PII exposure in logs and traces: full account and card numbers must be masked in observability output and evaluation reports (e.g., `ACC-****3390`), even though the underlying dataset is synthetic — the project should model real banking data-handling discipline.
- Observability must track and expose, per request: prompt/completion tokens, TTFT, inference latency, GPU utilization/memory, tool-call count, and failures — instrumented as Prometheus metrics (`slm_requests_total`, `slm_request_latency_seconds`, `slm_tokens_generated_total`, `slm_tool_calls_total`, `slm_agent_failures_total`).
- Cost figures (cost per 1,000 investigations) must be derived from actual measured throughput at a stated concurrency level, never from an assumed or estimated number, and should be sanity-checked against a documented, cited estimate of manual analyst investigation cost.

## 11. What You'll Learn
By the end, these should be explainable from direct experience, not from reading about them:
- **ML/LLM:** SLM architecture, tokenizer, context window, quantization, KV cache, batching
- **GPU:** VRAM, CUDA basics, GPU utilization, memory bandwidth, inference throughput
- **Inference:** transformers, vLLM, continuous batching, streaming, TTFT, tokens/sec
- **DevOps:** Docker, GPU containers, FastAPI, health checks, logging, Prometheus
- **Agentic AI:** tool calling, agent loops, structured outputs, tool-use evaluation, failure handling
- **Fraud/Risk Ops:** alert triage, fraud pattern recognition, risk scoring, evidence-based investigation, false-positive cost tradeoffs, why advisory-only/human-in-the-loop matters under regulatory and liability constraints
- **Economics:** GPU hourly cost, utilization, cost/request, self-hosting vs. API tradeoffs, self-hosted SLM vs. manual analyst cost
