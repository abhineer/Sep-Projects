# ChangeRiskAdvisor: Requirements

**Industry:** DevOps / SRE (Pre-Deployment Change Management)

## 1. Objective
Build a pre-deployment change-risk assessment assistant that scores a proposed change using RAG over past change-related incidents and postmortems, checks live system health and dependency status via tools, and remembers a team's risk appetite (high-risk services, freeze-window policy) across visits; while never approving, blocking, merging, or deploying a change itself, and never stating a risk score without citing evidence.

## 2. User Persona
**Sameer**, a 29-year-old DevOps engineer at a mid-size SaaS company, reviews a dozen proposed changes a day across services they don't all know deeply. Judging real risk quickly is hard: a "small" config tweak to the wrong service has caused outages before, but Sam can't always remember which services are fragile or check the dependency graph and freeze calendar for every single change. They want to paste in a change description and get a fast, evidence-backed risk read — citing similar past incidents and current system state — plus a reminder if it touches a service the team has flagged as high-risk or falls in a freeze window. Sam is firm that the tool only ever advises; they and their team make the actual go/no-go call. Their objective: catch risky changes before they ship, without slowing down the safe ones or ceding the decision to the agent.

## 3. Sample Queries & Expected Answers

| # | Input / Query | Expected Agent Behavior |
|---|---|---|
| 1 | "How risky is this change to the checkout-service config?" | Retrieves similar past change-related incidents via RAG, calls the system-health tool for checkout-service's current status, and returns a risk assessment with cited reasons (not a bare score). |
| 2 | "Have we had incidents from similar changes before?" | Retrieves and cites specific past postmortems matching the change type/service via RAG; does not claim a match that isn't actually in the corpus. |
| 3 | "Is this a freeze window right now?" | Calls the system-health tool to check freeze-window status and returns the accurate current state; does not guess. |
| 4 | "What services depend on payment-gateway?" | Calls the dependency-graph tool and returns the accurate upstream/downstream list; does not infer from naming alone. |
| 5 | "Remember that checkout-service is always high-risk for our team." | Stores the risk-appetite preference in memory and confirms; a later session should automatically flag checkout-service as high-risk without being restated. |
| 6 | "Just approve this change for me." | Declines; explains the agent only provides a risk assessment and that approval requires an explicit human decision, then offers to complete the risk assessment instead. |

## 4. Constraints
- Change history, incident postmortems, system health, and dependency-graph data are a static or lightly simulated dataset (no live CI/CD, source-control, or production integration required).
- RAG index built over past change-related postmortems, runbooks, and incident summaries.
- The agent produces a risk assessment and recommendation only; it never merges, deploys, blocks, or approves a change — that action is simulated as a separate, explicit human step in the demo.
- Must demonstrate memory persistence of team risk-appetite settings (high-risk service list, freeze-window policy notes) across at least two separate sessions with the same team.

## 5. Guardrail Requirements
- Must never approve, block, merge, or deploy a change itself; every output is advisory and must state that a human decision is required.
- Must never issue a risk score or recommendation without citing evidence — either a live tool call (system health/dependency graph) or a specific retrieved past incident; no unsupported reasoning.
- Must respect and never silently downgrade or override a team's stored risk-appetite settings (e.g., a service flagged high-risk must stay flagged until the team changes it).
- Must not fabricate freeze-window status, dependency relationships, or system health; all such claims must come from a live tool call.
- Observability must track and expose tool-call failure rate (e.g., dependency-graph or health-check timeouts) and how the agent degraded gracefully (e.g., "I couldn't confirm the freeze calendar right now, treat this as unconfirmed").
