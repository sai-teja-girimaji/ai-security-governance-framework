# NIST AI RMF SOC Mapping

> Mapping NIST AI Risk Management Framework controls to Security Operations Center AI deployment use cases

**Reference:** NIST AI RMF 1.0 (January 2023)
**Scope:** AI agents, LLM-powered tools, and automated decision systems deployed within SOC environments

---

## Overview

The NIST AI RMF organizes AI risk management across four core functions: **GOVERN**, **MAP**, **MEASURE**, and **MANAGE**. This document maps each function to the specific controls, implementation requirements, and verification criteria relevant to SOC AI deployments.

---

## GOVERN

Establishes the organizational policies, accountability structures, and culture required to manage AI risk.

### Policies and Accountability

| Control Area | SOC AI Requirement | Verification |
|---|---|---|
| AI risk policy | Document the organization's risk tolerance for autonomous AI action in the SOC. Define acceptable and unacceptable autonomous actions explicitly. | Approved policy document with CISO sign-off |
| Accountability assignment | Assign a named owner for each AI system deployed in the SOC. The owner is accountable for the system's outputs and autonomous actions. | RACI matrix with named individuals |
| Role definitions | Define SOC analyst responsibilities when working alongside AI systems. Specify when human judgment overrides AI recommendations. | Updated SOC playbooks and job descriptions |

### Training and Awareness

| Control Area | SOC AI Requirement | Verification |
|---|---|---|
| AI limitations training | All SOC analysts using AI-assisted workflows must understand the system's failure modes, data dependencies, and confidence thresholds. | Training completion records |
| Escalation training | On-call analysts must be trained to recognize and respond to AI checkpoint notifications within defined SLA windows. | Tabletop exercise records |

### Incident Response for AI Failures

| Control Area | SOC AI Requirement | Verification |
|---|---|---|
| AI incident classification | Define a separate incident category for AI system failures, including autonomous actions on incorrect targets. | Updated incident classification taxonomy |
| AI incident response procedure | Document the response procedure when an AI agent takes an incorrect autonomous action. Include rollback, containment, and stakeholder notification steps. | Tested runbook |
| Post-incident review | Require a structured post-incident review for every AI-related incident. Outputs must feed back into the governance framework. | Review template and records |

---

## MAP

Establishes context, identifies risk categories, and assesses the impact landscape before AI deployment.

### Context Establishment

| Control Area | SOC AI Requirement | Verification |
|---|---|---|
| System context documentation | Document every data source the AI system accesses, every action it can take, and every external system it can call. | System context document reviewed by security architect |
| Stakeholder identification | Identify all stakeholders affected by AI autonomous actions, including IT operations, business units, and compliance. | Stakeholder register |
| Deployment boundary definition | Define the explicit boundary of the AI system's operational scope. Any action outside this boundary requires human approval. | Boundary document with signed acceptance |

### Impact Assessment

| Control Area | SOC AI Requirement | Verification |
|---|---|---|
| Impact category mapping | For each autonomous action the AI can take, document the worst-case impact if the action is incorrect. | Impact register per action type |
| Business continuity impact | Identify all Tier 1 assets where an incorrect autonomous action could trigger a business continuity event. These assets must be excluded from autonomous action scope. | Asset classification register |
| Cascading failure analysis | Assess whether an incorrect AI action on one asset could trigger downstream failures on connected systems. | Dependency mapping review |

### Risk Identification

| Control Area | SOC AI Requirement | Verification |
|---|---|---|
| Data quality risk | Assess the risk introduced by each log source feeding the AI system. Incomplete or noisy sources must be documented and either remediated or excluded. | Data quality assessment per source |
| Baseline risk | Assess the risk of deploying the AI system without established behavioral baselines. Document the observe-only period required before autonomous action is permitted. | Baseline readiness sign-off |

---

## MEASURE

Quantifies and evaluates AI system performance, security, and risk through systematic testing and monitoring.

### Performance Metrics

| Metric | SOC AI Definition | Measurement Frequency |
|---|---|---|
| True positive rate | Percentage of genuine threats correctly identified and acted upon by the AI agent | Weekly |
| False positive rate | Percentage of benign events incorrectly flagged or acted upon | Weekly |
| Escalation accuracy | Percentage of human escalations where the AI's recommended action was confirmed correct by the analyst | Monthly |
| Autonomous action accuracy | Percentage of autonomous containment actions that were confirmed correct in post-action review | Per incident |
| Checkpoint response rate | Percentage of human approval requests acknowledged within the defined SLA window | Weekly |

### Data Quality Metrics

| Metric | Threshold | Action if Breached |
|---|---|---|
| Timestamp normalization coverage | 100% of sources | Suspend agent ingestion from non-compliant source |
| Duplicate event rate | Less than 2% per source | Remediate deduplication before agent ingestion |
| Process-level telemetry coverage | Required sources 100% complete | Do not permit autonomous action on assets with incomplete telemetry |
| Log source uptime | Greater than 99% | Alert on gaps, pause autonomous action during extended outages |

### Security Evaluation

| Control Area | SOC AI Requirement | Frequency |
|---|---|---|
| Prompt injection testing | Test the AI system against known prompt injection techniques. Document results and remediation. | Quarterly |
| Adversarial input testing | Submit crafted log entries designed to manipulate the AI's reasoning. Verify the system handles them correctly. | Quarterly |
| Permission boundary testing | Verify that Tier 1 asset protections cannot be bypassed through any input manipulation or escalation path. | At deployment and after any system change |
| Model drift assessment | Evaluate whether the AI system's behavior has drifted from its baseline over time. | Monthly |

---

## MANAGE

Prioritizes, responds to, and monitors AI risks on an ongoing basis.

### Risk Prioritization

| Risk | Priority | Response |
|---|---|---|
| Incorrect autonomous action on Tier 1 asset | Critical | Immediate human review, incident declaration, system suspension pending investigation |
| Incorrect autonomous action on Tier 2 asset | High | Human review within 2 hours, post-action report |
| Incorrect escalation on Tier 3 asset | Medium | Log, review in weekly metrics, update model context if pattern identified |
| Human checkpoint missed due to notification failure | High | Investigate notification path, remediate within 24 hours |
| Data quality degradation below threshold | High | Suspend agent ingestion from affected source within 4 hours |

### Monitoring Requirements

| Item | Monitoring Approach | Alert Threshold |
|---|---|---|
| All autonomous actions | Real-time logging with asset tier, action type, confidence score, and outcome | Any Tier 1 action (alert immediately) |
| Human checkpoint acknowledgment | Track acknowledgment time per request | Any request unacknowledged beyond SLA window |
| AI system performance drift | Weekly comparison against baseline metrics | Any metric degrading more than 10% week-over-week |
| Data source quality | Continuous monitoring of timestamp, deduplication, completeness | Any source breaching defined thresholds |

### Review and Revalidation

| Activity | Frequency | Owner |
|---|---|---|
| Full framework review | Annually or after any significant AI system change | AI system owner + CISO |
| Tier classification review | Quarterly | SOC lead + IT operations |
| Behavioral baseline update | After major environment changes | Detection engineering team |
| Vendor/model update assessment | Before any model update is applied to production | Security architect |

---

## References

- NIST AI Risk Management Framework 1.0, January 2023: https://doi.org/10.6028/NIST.AI.100-1
- NIST AI RMF Playbook: https://airc.nist.gov/Docs/2
