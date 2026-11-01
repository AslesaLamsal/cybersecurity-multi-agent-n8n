# Cybersecurity Alert Investigation System (Multi-Agent, n8n)

A four-agent AI workflow built in n8n that reviews simulated security alerts, links them to supporting logs and user/asset context, applies company escalation rules, and prepares an investigation recommendation for a human security analyst.

Built as the capstone project of the Pexus AI Readiness (PAIR) Programme. All data is simulated.

## Problem
Security teams receive many alerts, and most are not real incidents. Reviewing each one manually (finding logs, checking the user and asset, applying rules) is slow and inconsistent. This system automates the evidence-gathering and rule-application steps and leaves every decision to a human.

## Agents
| Agent | Responsibility |
|---|---|
| 1. Alert Intake and Evidence Linker | Organises the alert and logs into a timeline, classifies the activity and lists missing evidence. Does not judge severity. |
| 2. Context Analyst | Decides if the activity is expected for the user's role, access level, working hours or the asset's sensitivity. |
| 3. Evidence Evaluator | Rates evidence strength and separates attempts, blocked actions, successful access and data removal. Tracks gaps and contradictions. |
| 4. Risk Assessor | Applies 12 company rules and a points rubric to give a severity, rules cited and next steps for the analyst. |

## Workflow
Manual trigger (alert ID) -> data table lookups (ALERTS, LOGS, USERS, ASSETS) -> Build Case File (code: working-hours check, missing-log check) -> Agent 1 -> Agent 2 -> Agent 3 -> Agent 4 (each followed by a JSON parsing/validation node) -> Build Investigation Record (code) -> human review check -> Save Investigation.

## Safeguards enforced in code
- Risk score is recomputed from the score breakdown instead of trusting the model's arithmetic.
- Severity is capped at Medium when evidence strength is below 0.5 (Rule 11).
- A case is flagged for human review if severity is High/Critical, confidence is below 0.6, the acting account is unknown, evidence is contradictory, or there are several evidence gaps.
- The system performs analysis only. It never disables accounts, resets passwords, blocks users, deletes files or changes permissions.

## Testing
All 12 simulated alerts were run and covered: evidence supporting the alert, normal business explanations, sensitive data, incomplete evidence, different roles/assets and varying severity (Low to Critical). Issues found and fixed during testing included unreliable model arithmetic (moved to code) and unreliable structured output (replaced with validated JSON parsing nodes).

## How to run
1. Import `cybersecurity-alert-investigation.json` into n8n (Workflows -> Import from file).
2. Add your own OpenRouter credential to the four chat model nodes.
3. Create the data tables ALERTS, LOGS, USERS, ASSETS and INVESTIGATIONS with the fields used in the workflow.
4. Set the alert ID in the Input node and run with the manual trigger.

## Limitations
- Reasons only about the simulated data in the tables, not real telemetry.
- Free-tier model: output can vary between runs and rate limits apply.
- Alerts are investigated independently, so related alerts (e.g. two phishing alerts) are not correlated.
- The notification step prepares a message but does not send it.
- The webhook trigger is included but the tested path is the manual trigger.

## Author
Aslesa Lamsal
