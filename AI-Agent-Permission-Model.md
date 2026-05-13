# AI Agent Permission Model

> Tiered autonomy framework for AI agents deployed in Security Operations Center environments

---

## Overview

An AI agent in a SOC environment can take actions ranging from read-only log queries to autonomous host isolation. The impact of an incorrect decision varies enormously depending on the asset targeted. A misconfiguration that isolates a developer workstation is a minor disruption. The same misconfiguration applied to a domain controller or clearing system is a business continuity event.

This framework defines a tiered permission model that governs what autonomous actions an AI agent is permitted to take, based on the criticality of the target asset. The tiers are enforced at the agent and integration layer, not at the policy or process layer.

**Core principle:** Autonomous AI action is a privilege earned by the environment, not a default granted by the vendor.

---

## Asset Classification

Before the permission model can be applied, every asset in scope must be classified into a tier. Classification is the responsibility of the SOC lead and IT operations teams, reviewed quarterly.

### Classification Criteria

An asset is classified as **Tier 1** if any of the following apply:

- Disruption would directly cause a business continuity event (trading halt, payment processing failure, customer-facing outage)
- Recovery time objective (RTO) is less than 4 hours
- The asset processes regulated data (PCI, HIPAA, financial settlement)
- Compromise would give an attacker enterprise-wide access (domain controllers, PKI infrastructure, secrets management)
- The asset is a dependency for 10 or more production systems

An asset is classified as **Tier 2** if any of the following apply:

- Disruption causes significant operational impact but is recoverable within business hours
- The asset supports internal business operations but not customer-facing services
- Recovery is well-documented with a tested runbook
- Compromise would give an attacker access to a defined business unit but not enterprise-wide access

An asset is classified as **Tier 3** if all of the following apply:

- Disruption causes minimal operational impact
- Recovery is straightforward and does not require change management approval
- The asset does not process sensitive or regulated data in production
- Isolation does not affect other assets' availability

---

## Tier Definitions

### Tier 1: Critical Assets

| Field | Definition |
|---|---|
| Examples | Domain controllers, PKI infrastructure, clearing and settlement systems, production databases, core network infrastructure (firewalls, core routers), secrets management systems, backup infrastructure |
| Autonomous action | Not permitted under any circumstances |
| Permitted AI actions | Read-only investigation: log queries, telemetry retrieval, IOC enrichment, case summarization, threat intelligence lookup |
| Required response | Mandatory human escalation before any containment, blocking, or remediation action |
| Escalation path | On-call SOC analyst (primary) with acknowledgment-required alert. If not acknowledged within SLA window, escalate to SOC lead. |
| SLA window | 15 minutes for initial acknowledgment |
| Override | No autonomous action override exists for Tier 1 assets. Any containment action on a Tier 1 asset requires explicit analyst approval through the defined escalation path. |

### Tier 2: Standard Assets

| Field | Definition |
|---|---|
| Examples | Application servers, web servers, collaboration platforms, file servers, internal tooling, cloud workloads supporting internal operations |
| Autonomous action | Not permitted at initial deployment. May be permitted after 90 days of validated behavioral baselines with explicit sign-off from SOC lead and security architect. |
| Permitted AI actions | Full investigation workflow: log queries, telemetry retrieval, IOC enrichment, case summarization, threat intelligence lookup, lateral movement mapping, recommended response actions |
| Required response | Human-confirmed response before containment executes. The AI agent may recommend and prepare the action, but a human analyst must confirm before execution. |
| Escalation path | On-call analyst receives recommended action with supporting evidence. Analyst confirms or modifies before execution. |
| SLA window | 30 minutes for analyst confirmation |
| Fallback if SLA missed | Investigation-only mode. No automated containment. Escalate to SOC lead. |

### Tier 3: Low-Risk Assets

| Field | Definition |
|---|---|
| Examples | Developer workstations, test environments, staging systems, lab infrastructure, decommissioned systems still in inventory |
| Autonomous action | Permitted within defined scope after 30-day behavioral baseline period |
| Permitted AI actions | Full investigation and response workflow including autonomous containment, network isolation, and process termination within defined scope |
| Scope limits | Autonomous actions are limited to: network isolation of the target host, termination of identified malicious processes, collection of forensic artifacts. Actions that affect other assets (firewall rule changes affecting multiple hosts, AD policy changes) require human approval regardless of tier. |
| Audit requirement | All autonomous actions are logged with: timestamp, target asset, action taken, confidence score, triggering alert, and case ID |
| Review requirement | All autonomous actions on Tier 3 assets are reviewed by an analyst within 2 hours of execution |

---

## Permission Matrix

| Action Type | Tier 1 | Tier 2 | Tier 3 |
|---|---|---|---|
| Log queries and telemetry retrieval | Permitted | Permitted | Permitted |
| IOC enrichment and threat intelligence lookup | Permitted | Permitted | Permitted |
| Case summarization and report generation | Permitted | Permitted | Permitted |
| Lateral movement mapping | Permitted | Permitted | Permitted |
| Recommended action generation | Permitted | Permitted | Permitted |
| Network isolation of target host | Not permitted | Human-confirmed | Autonomous (in scope) |
| Process termination on target host | Not permitted | Human-confirmed | Autonomous (in scope) |
| Firewall rule modification | Not permitted | Not permitted | Human-confirmed |
| Active Directory account action | Not permitted | Not permitted | Human-confirmed |
| DNS blocking | Not permitted | Human-confirmed | Human-confirmed |
| Email quarantine | Not permitted | Human-confirmed | Autonomous (in scope) |
| Forensic artifact collection | Permitted | Permitted | Permitted |
| Case closure | Not permitted | Human-confirmed | Autonomous (in scope) |

---

## Implementation Requirements

### Technical Enforcement

The permission model must be enforced at the integration layer, not the orchestration layer. This means:

1. **SOAR integration:** The SOAR platform's API calls enforce tier-based restrictions. An action request for a Tier 1 asset returns an error regardless of what the AI agent requested.
2. **Endpoint integration:** EDR/endpoint management integrations verify asset tier before executing any action. Tier 1 and Tier 2 assets require a human-signed approval token.
3. **Firewall integration:** Firewall API integrations require human-approved change tickets for all actions. The AI agent cannot directly execute firewall changes.
4. **Fallback behavior:** If the permission enforcement layer cannot determine an asset's tier (asset not in inventory, classification lookup failure), the asset is treated as Tier 1.

### Human Approval Mechanism

Human approval is only valid if it meets all of the following criteria:

- Delivered through an acknowledgment-required mechanism (PagerDuty, OpsGenie, or equivalent). Slack or email are not sufficient for Tier 2 approvals.
- The approving analyst has reviewed the supporting evidence, not just the recommended action
- Approval is logged with the analyst's identity, timestamp, and case reference
- Approval was given within the defined SLA window

### Asset Inventory Integration

- The permission model requires a current, accurate asset inventory with tier classifications
- The AI agent queries the inventory in real time before any action execution
- Inventory must be updated within 24 hours of any asset classification change
- New assets default to Tier 1 until formally classified

---

## Override Procedures

### Emergency Override (Tier 2 Only)

In exceptional circumstances, a SOC lead may authorize a Tier 2 autonomous action without the standard human confirmation window. This requires:

1. SOC lead authorization via the defined approval system (not verbal, not chat)
2. Documented justification referencing the specific incident
3. Post-incident review within 24 hours

**There is no emergency override for Tier 1 assets.**

### Tier Reclassification

If an analyst believes an asset's tier classification is incorrect during an active incident:

1. The asset continues to operate under its current tier classification
2. A reclassification request is submitted through the standard change process
3. Reclassification takes effect after review and approval, not immediately during an incident

---

## Audit and Logging Requirements

Every AI agent action must be logged with the following fields:

| Field | Description |
|---|---|
| Timestamp | UTC timestamp of action execution |
| Agent ID | Identifier of the AI agent instance |
| Case ID | Investigation case reference |
| Target asset | Hostname, IP, or asset identifier |
| Asset tier | Tier classification at time of action |
| Action type | Categorized action from defined taxonomy |
| Action parameters | Specific parameters of the action taken |
| Triggering alert | Alert or event that initiated the investigation |
| Confidence score | AI agent's confidence score for the action |
| Approval status | Autonomous / Human-confirmed (with approver ID) |
| Outcome | Success / Failure / Partial |

Logs must be retained for a minimum of 12 months and must be tamper-evident.

---

## Review Schedule

| Activity | Frequency | Owner |
|---|---|---|
| Asset tier classification review | Quarterly | SOC lead + IT operations |
| Permission matrix review | Annually or after any AI system change | Security architect + SOC lead |
| Autonomous action audit | Monthly | SOC analyst (sampling) |
| Override procedure exercise | Bi-annually | SOC lead |
