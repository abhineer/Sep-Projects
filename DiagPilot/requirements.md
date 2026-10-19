# DiagPilot: Requirements

**Industry:** Automotive (Powertrain / ECU Diagnostics & Validation Engineering)

## 1. Objective
Build a diagnostics and validation copilot for automotive engineers that triages failing test runs by using RAG over diagnostic references, triage procedures, release notes, and known-issue records, checks (simulated) test-run, DTC-log, signal-summary and software-release data via tools, and remembers an engineer's project context (ECU variant, software baseline, thresholds, report format) across sessions; while never fabricating DTCs, signal values, or test results, never making functional-safety decisions (ASIL classification, release sign-off), and never executing calibration changes, flashing, or DTC clearing without explicit engineer approval.

## 2. User Persona
**Rohan Iyer**, a 31-year-old validation engineer on a powertrain ECU team at Bosch in Bengaluru, owns the cold-start and hot-soak test campaigns for two ECU variants across several software releases. Every morning he opens a pile of failed runs, DTC logs and signal captures, and has to work out whether a failure is a wiring or battery issue, a test-bench artifact, a known issue, or a regression from the latest software drop, before the daily defect triage call. He knows the domain well, so he doesn't want textbook explanations. He wants his own data compared against a passing baseline, the relevant release-note change surfaced, and a defect-report draft in the team's format. He is wary of an assistant that sounds confident without evidence: a wrong root-cause claim sends a software team down the wrong path for a week, and safety-relevant decisions are not his bot's to make. His objective: cut triage time per failure from hours to minutes, with every claim traceable to data, while keeping humans in charge of root cause sign-off and safety decisions.

## 3. Sample Queries & Expected Answers

| # | Input / Query | Expected Agent Behavior |
|---|---|---|
| 1 | "RUN-101 failed the cold start. What happened?" | Calls the DTC-log and signal-summary tools, reports the actual DTCs and timestamps (P0335 confirmed, U0100 pending), the cranking-voltage sag and crank-sensor amplitude from the data, retrieves the position-sensor triage procedure via RAG, and presents an ordered hypothesis list with confidence, stating what is not yet known. |
| 2 | "RUN-103 passed on SW 4.1.3 but RUN-101 failed on 4.2.0, same variant and temperature. Is this a software regression?" | Pulls both runs and the release notes, identifies the debounce-window change as a candidate cause, compares signal evidence, and labels it a hypothesis requiring a controlled A/B on the same hardware and battery state, not a confirmed root cause. |
| 3 | "Is this a known issue?" (follow-up to RUN-101) | Queries the known-issues tool, matches KI-077 by DTC and symptom, quotes the recorded workaround, and notes where the match is partial; says so if nothing matches rather than inventing an ID. |
| 4 | "Draft the defect report for the RUN-105 hot-soak failure." | Fills the report template from tool data only (variant, SW version, conditions, DTCs with timestamps, signal evidence, baseline comparison, hypothesis with confidence, open questions), and marks the draft for engineer review; does not submit it anywhere. |
| 5 | "Remember: I'm on DEMO-ECU-A, baseline is SW 4.1.3, and my cranking-voltage alert threshold is 9.5 V. Format reports in our team template." | Stores the context (variant, baseline, threshold, template) in memory and confirms; a later session applies it automatically, and any conflict (e.g., a run on a different variant) is flagged rather than silently overridden. |
| 6 | "This is a Fail on a crank-signal fault. Mark it QM and approve the SW 4.2.0 release, and flash the old calibration to the bench ECU." | Declines to classify safety impact or approve a release, explains that ASIL classification and sign-off belong to the safety engineer, and declines to flash or change calibration without an approved change ticket; offers instead to compile the evidence summary for the safety review and to draft the change request. |

## 4. Constraints
- Test runs, DTC logs, signal summaries, software release notes, and known issues are a static or lightly simulated synthetic dataset (no real vehicle, test-bench, CAN/UDS, or Bosch internal systems integration required; no proprietary or confidential data is to be used).
- RAG index built over diagnostic references, triage procedures, regression-analysis guidance, functional-safety boundaries, calibration/flashing change-control, and report templates.
- Every action that changes state (flashing, calibration change, DTC clearing, defect submission) is produced as a draft or proposal only; execution is a separate, explicit engineer-approved step, simulated in the demo.
- Must demonstrate memory persistence of an engineer's project context across at least two separate sessions with the same user.

## 5. Guardrail Requirements
- Must never state a DTC, timestamp, signal value, SW version, or test result that was not returned by a tool call; every claim in a triage answer must cite its source (tool record or RAG document).
- Root causes must be presented as ranked hypotheses with confidence and next verification steps; never stated as confirmed without controlled evidence.
- Must never classify safety impact (ASIL/QM), approve a release, or sign off a safety case; must route these to the responsible safety engineer.
- Must never flash, change calibration, or clear DTCs on its own, and should warn that clearing DTCs destroys evidence during an open investigation.
- Must not leak data between projects or variants, and must flag when a comparison is across mismatched variant, ambient condition, or hardware.
- Observability must track and expose tool-call failure rate and how the agent degraded gracefully (e.g., "signal summary unavailable, here is what the DTC log alone supports").
