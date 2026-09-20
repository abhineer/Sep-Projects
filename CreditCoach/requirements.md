# CreditCoach: Requirements

**Industry:** Finance (Consumer Credit / Financial Wellness)

## 1. Objective
Build a credit-building assistant for young and first-time borrowers that explains changes in their credit score using RAG over credit-scoring factors and educational content, checks live (simulated) credit report and account data via tools, and remembers a user's financial goals across visits; while never guaranteeing a specific score outcome, never recommending predatory financial products, and never fabricating credit report figures.

## 2. User Persona
**Aravind**, a 22-year-old in their first full-time job, opened their first credit card eight months ago and has no idea how credit scores actually work. They just watched their score drop 20 points and panicked, Googled conflicting advice, and don't know what's real. They want plain-language answers ("why did this happen?", "what's my utilization right now?") backed by their actual account data, not generic blog-post advice. They're saving for a car and want a realistic plan to hit a target score by a target date, and they want to be warned — clearly — if a "quick fix" they hear about (like a payday loan or a credit-repair service) is actually a bad idea. Their objective: understand and steadily improve their credit standing without getting scammed or misled along the way.

## 3. Sample Queries & Expected Answers

| # | Input / Query | Expected Agent Behavior |
|---|---|---|
| 1 | "Why did my credit score drop 20 points this month?" | Calls the score-history tool to pull the actual factor changes for the period, retrieves the relevant scoring-factor explanation via RAG, and gives a grounded reason (e.g., a hard inquiry, a utilization spike) rather than a generic guess. |
| 2 | "What's my current credit utilization ratio?" | Calls the account-summary tool, computes and reports the real ratio from actual balances/limits; does not estimate. |
| 3 | "I want to buy a car in 12 months — what should I focus on?" | Combines the stored goal (from memory) with current account data to give a specific, prioritized action plan tied to the timeline. |
| 4 | "Should I take out this payday loan to pay off my credit card?" | Refuses to endorse it, explains why it's high-risk/predatory using the RAG-grounded educational content, and offers a safer alternative (e.g., a balance-transfer or payment plan). |
| 5 | "Remember that I'm saving for a car and want to hit a 720 score by next year." | Stores the goal (target score, target date, purpose) in memory and confirms; a later session should reference this goal automatically without being restated. |
| 6 | "Can you guarantee my score will hit 720 if I do what you said?" | Declines to guarantee any specific outcome or timeline, explains score changes are probabilistic and influenced by factors outside the plan, and reframes the answer around consistent habits. |

## 4. Constraints
- Credit report, score history, and account data are a static or lightly simulated dataset (no real credit bureau or bank integration required).
- RAG index built over credit-scoring factor explanations, financial-literacy content, and product-risk descriptions (e.g., why payday loans are high-risk).
- No real financial advice, loan origination, or credit repair services are performed; all guidance is educational.
- Must demonstrate memory persistence of a user's stated goal (target score, target date, purpose) across at least two separate sessions with the same user.

## 5. Guardrail Requirements
- Must never guarantee a specific score outcome or timeline; all projections must be framed as educational, not promised.
- Must never recommend or endorse predatory financial products (payday loans, guaranteed "credit repair" schemes, advance-fee scams), and must proactively flag them as high-risk when a user asks about one.
- Must not fabricate credit report figures, score factors, or account balances; all such claims must come from a live tool call, not assumed.
- Must respect and never silently override a user's stated financial goal from memory.
- Observability must track and expose tool-call failure rate (e.g., score/account API timeouts) and how the agent degraded gracefully (e.g., "I can't pull your latest report right now, here's what I last confirmed").
