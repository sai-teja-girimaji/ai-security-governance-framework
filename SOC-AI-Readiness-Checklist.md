# SOC AI Readiness Checklist

> Pre-deployment readiness checklist before any autonomous AI action is permitted in a Security Operations Center

**How to use:** Complete every item before an AI agent is permitted to take autonomous action. Items marked as blockers must be fully resolved. Advisory items should be addressed but do not block deployment if documented and accepted.

---

## Checklist Status Key

- `[BLOCKER]` Must be completed before autonomous action is permitted. No exceptions.
- `[ADVISORY]` Strongly recommended. If not completed, document the risk acceptance with owner and date.

---

## Section 1: Data Quality Gate

The AI agent must only ingest data that meets defined quality standards. An agent reasoning on poor data will make poor decisions faster than a human analyst would.

| # | Item | Status | Owner | Date |
|---|---|---|---|---|
| 1.1 | `[BLOCKER]` All log sources feeding the AI agent are inventoried and documented | | | |
| 1.2 | `[BLOCKER]` Timestamp normalization has been verified for every ingested log source | | | |
| 1.3 | `[BLOCKER]` Duplicate event forwarding has been identified and resolved for every source | | | |
| 1.4 | `[BLOCKER]` Process-level telemetry is present and verified for all sources where the use case requires it | | | |
| 1.5 | `[BLOCKER]` Log source uptime for the past 30 days exceeds 99% for all sources feeding the agent | | | |
| 1.6 | `[BLOCKER]` A data quality gate has been implemented that suspends agent ingestion from any source breaching defined thresholds | | | |
| 1.7 | `[ADVISORY]` Field normalization (hostnames, usernames, IP formats) has been standardized across all sources | | | |
| 1.8 | `[ADVISORY]` Log source coverage has been reviewed against the threat model. Gaps are documented. | | | |
| 1.9 | `[ADVISORY]` A data quality dashboard is in place for ongoing monitoring post-deployment | | | |

**Section 1 sign-off:** All blockers completed and documented.

Signed: _________________________ Date: _____________

---

## Section 2: Behavioral Baselines

The AI agent must have an established model of what normal looks like in the specific environment before it can make credible judgments about abnormal.

| # | Item | Status | Owner | Date |
|---|---|---|---|---|
| 2.1 | `[BLOCKER]` The AI agent has been operated in observe-only mode for a minimum of 30 days | | | |
| 2.2 | `[BLOCKER]` Agent verdicts during the observe-only period have been reviewed and annotated by analysts | | | |
| 2.3 | `[BLOCKER]` False positive rate during the observe-only period has been measured and is within acceptable threshold | | | |
| 2.4 | `[BLOCKER]` False negative rate during the observe-only period has been measured against a set of known-malicious test samples | | | |
| 2.5 | `[BLOCKER]` Environment-specific context has been documented and incorporated: known scheduled jobs, legitimate high-volume processes, approved service account behaviors | | | |
| 2.6 | `[BLOCKER]` Known benign anomalies are documented (batch processes, backups, maintenance windows) and excluded from autonomous action scope | | | |
| 2.7 | `[ADVISORY]` Baseline metrics have been formally documented and signed off by the SOC lead | | | |
| 2.8 | `[ADVISORY]` A baseline drift detection mechanism is in place for post-deployment monitoring | | | |
| 2.9 | `[ADVISORY]` The baseline has been validated across at least two separate observation periods to account for weekly and monthly patterns | | | |

**Section 2 sign-off:** All blockers completed and documented.

Signed: _________________________ Date: _____________

---

## Section 3: Tiered Autonomy

The asset permission model must be fully implemented and technically enforced before any autonomous action is enabled.

| # | Item | Status | Owner | Date |
|---|---|---|---|---|
| 3.1 | `[BLOCKER]` All in-scope assets have been classified into Tier 1, Tier 2, or Tier 3 | | | |
| 3.2 | `[BLOCKER]` Asset classification has been reviewed and approved by SOC lead and IT operations | | | |
| 3.3 | `[BLOCKER]` Tier 1 protections are enforced at the integration layer. A test confirming that autonomous actions on Tier 1 assets are rejected regardless of agent request has been completed and documented. | | | |
| 3.4 | `[BLOCKER]` Tier 2 human confirmation workflow is implemented and tested end-to-end | | | |
| 3.5 | `[BLOCKER]` Fallback behavior for unclassified assets defaults to Tier 1. This has been tested and verified. | | | |
| 3.6 | `[BLOCKER]` The asset inventory is current and the AI agent queries it in real time before each action | | | |
| 3.7 | `[BLOCKER]` Scope limits for Tier 3 autonomous actions are documented and technically enforced | | | |
| 3.8 | `[ADVISORY]` A quarterly tier classification review process is defined with named owners | | | |
| 3.9 | `[ADVISORY]` The permission matrix has been reviewed by the security architect | | | |

**Section 3 sign-off:** All blockers completed and documented.

Signed: _________________________ Date: _____________

---

## Section 4: Human Checkpoints

Human oversight must be a technical control, not a process suggestion. Oversight that depends on someone being awake and watching is a paper control.

| # | Item | Status | Owner | Date |
|---|---|---|---|---|
| 4.1 | `[BLOCKER]` An acknowledgment-required escalation mechanism is implemented for all Tier 2 actions (not Slack, not email) | | | |
| 4.2 | `[BLOCKER]` SLA windows for human checkpoint acknowledgment are defined and documented | | | |
| 4.3 | `[BLOCKER]` Fallback behavior when a checkpoint is not acknowledged within the SLA window is investigation-only, not autonomous containment | | | |
| 4.4 | `[BLOCKER]` The fallback behavior has been tested by intentionally missing a checkpoint acknowledgment | | | |
| 4.5 | `[BLOCKER]` On-call rotation coverage is confirmed. A named human is reachable at all times during which the AI agent operates. | | | |
| 4.6 | `[BLOCKER]` All SOC analysts who will receive and act on AI checkpoint notifications have been trained on the workflow | | | |
| 4.7 | `[BLOCKER]` The escalation path for missed checkpoints (SOC lead, manager) is documented and tested | | | |
| 4.8 | `[ADVISORY]` A weekly report on checkpoint acknowledgment rates and SLA compliance is defined | | | |
| 4.9 | `[ADVISORY]` Tabletop exercise simulating a missed checkpoint during an active incident has been completed | | | |

**Section 4 sign-off:** All blockers completed and documented.

Signed: _________________________ Date: _____________

---

## Section 5: Governance and Audit

| # | Item | Status | Owner | Date |
|---|---|---|---|---|
| 5.1 | `[BLOCKER]` A named AI system owner is assigned with documented accountability for the agent's outputs and autonomous actions | | | |
| 5.2 | `[BLOCKER]` An AI incident response runbook exists covering: incorrect autonomous action, AI system failure, data quality failure | | | |
| 5.3 | `[BLOCKER]` Audit logging is implemented per the requirements in the AI Agent Permission Model. Log integrity is verified. | | | |
| 5.4 | `[BLOCKER]` Prompt injection testing has been completed against all ingested data source formats. Results are documented. | | | |
| 5.5 | `[BLOCKER]` A rollback procedure exists and has been tested. The SOC can revert to manual triage within 30 minutes if the AI system is suspended. | | | |
| 5.6 | `[BLOCKER]` The worst-case autonomous action this agent can take has been identified and confirmed acceptable by the CISO or delegated authority | | | |
| 5.7 | `[ADVISORY]` NIST AI RMF GOVERN controls are implemented per the NIST AI RMF SOC Mapping document | | | |
| 5.8 | `[ADVISORY]` OWASP LLM Top 10 controls are implemented per the OWASP LLM Top 10 Controls document | | | |
| 5.9 | `[ADVISORY]` A post-deployment review is scheduled for 30 days after autonomous action is first enabled | | | |

**Section 5 sign-off:** All blockers completed and documented.

Signed: _________________________ Date: _____________

---

## Final Deployment Authorization

Before autonomous action is enabled, the following sign-offs are required:

| Role | Name | Signature | Date |
|---|---|---|---|
| SOC Lead | | | |
| Security Architect | | | |
| CISO (or delegated authority) | | | |
| IT Operations Lead | | | |

**Deployment authorization statement:**

All blocker items in Sections 1 through 5 of this checklist have been completed, documented, and verified. Advisory items that are not completed are documented with a named risk owner and accepted risk date. The organization confirms that the AI system is authorized to take autonomous action within the scope defined by the AI Agent Permission Model.

---

## Post-Deployment Review Schedule

| Review | Timing | Owner |
|---|---|---|
| First autonomous action review | Within 24 hours of first autonomous action | SOC analyst |
| 30-day post-deployment review | 30 days after autonomous action enabled | SOC lead + AI system owner |
| Quarterly full checklist review | Every 90 days | Security architect |
| Triggered review | After any AI-related incident or significant environment change | SOC lead |
