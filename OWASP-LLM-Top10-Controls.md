# OWASP LLM Top 10 Controls for SOC Environments

> Defensive controls mapped to OWASP Top 10 for Large Language Model Applications, applied to Security Operations Center deployments

**Reference:** OWASP Top 10 for LLM Applications v1.0
**Scope:** LLM-powered tools, AI investigation agents, and RAG systems deployed within SOC environments

---

## Overview

The OWASP LLM Top 10 defines the ten most critical security risks for applications built on large language models. In a SOC context, these risks carry elevated consequence because the LLM is connected to security telemetry, has access to sensitive threat intelligence, and in many deployments can trigger real response actions.

Each risk below includes a SOC-specific threat scenario, defensive controls, and detection indicators.

---

## LLM01: Prompt Injection

**Risk:** Malicious content embedded in inputs manipulates the LLM to override its instructions, bypass safety controls, or execute unintended actions.

**SOC Threat Scenario:** A threat actor embeds prompt injection payloads in log data, email subjects, or file names. When the AI investigation agent processes this telemetry, the injected instruction overrides its system prompt and causes it to suppress alerts, exfiltrate case data, or report false verdicts.

**Defensive Controls:**

| Control | Implementation |
|---|---|
| Input sanitization | Strip or encode control characters, special tokens, and instruction-like patterns from all data sources before LLM ingestion |
| Prompt structure hardening | Separate system instructions from user/data content using structural delimiters. Never concatenate untrusted data into system prompt sections. |
| Instruction hierarchy enforcement | Implement a privilege hierarchy where system-level instructions cannot be overridden by content processed from data sources |
| Output validation | Validate all LLM outputs against expected response schemas before any action is taken |
| Sandboxed processing | Process untrusted data (log entries, email content, file metadata) in an isolated context with restricted tool access |

**Detection Indicators:**
- LLM output contains instruction-like language not present in the system prompt
- Agent behavior deviates from expected workflow patterns
- Unusual tool calls or data access patterns following ingestion of specific data sources

---

## LLM02: Insecure Output Handling

**Risk:** LLM-generated output is passed to downstream systems without validation, enabling injection attacks, code execution, or unintended system interactions.

**SOC Threat Scenario:** An AI agent generates a SOAR playbook action based on LLM output. Without output validation, a manipulated response could trigger unauthorized firewall rule changes, incorrect asset isolation, or unintended script execution.

**Defensive Controls:**

| Control | Implementation |
|---|---|
| Output schema validation | Define strict schemas for all LLM outputs that feed downstream actions. Reject any output that does not conform. |
| Action pre-execution review | For any LLM output that triggers a system action, validate the action against the permission model before execution |
| Output encoding | Treat LLM outputs as untrusted data when passing to downstream systems. Apply appropriate encoding for the target context. |
| Capability restriction | Restrict which downstream systems the LLM output can interact with. Tool calls must be explicitly allowlisted. |

**Detection Indicators:**
- LLM output contains system commands, script fragments, or markup not expected in normal responses
- Downstream action does not match the alert context that triggered it

---

## LLM03: Training Data Poisoning

**Risk:** Training data has been manipulated to introduce biases, backdoors, or vulnerabilities into the model's behavior.

**SOC Threat Scenario:** A fine-tuned model used for threat classification was trained on poisoned data that causes it to systematically misclassify a specific attacker's TTPs as benign. The adversary operates undetected because the model consistently suppresses relevant alerts.

**Defensive Controls:**

| Control | Implementation |
|---|---|
| Training data provenance | Document and verify the source of all data used for fine-tuning or RAG knowledge bases |
| Behavioral validation | After any model update or fine-tuning, run a structured validation suite against known-malicious samples before production deployment |
| Vendor model attestation | Require vendors to provide documentation of training data sources, filtering applied, and known limitations |
| Anomaly baselining | Establish a behavioral baseline for each model version. Flag deviations after updates. |

**Detection Indicators:**
- Model systematically under-performs on a specific threat actor's TTPs compared to others
- Classification patterns change significantly after a model update
- False negative rate for known attack patterns increases unexpectedly

---

## LLM04: Model Denial of Service

**Risk:** Adversarial inputs consume excessive computational resources, causing degraded performance or service unavailability.

**SOC Threat Scenario:** During an active incident, an attacker floods log sources with high-volume, context-heavy entries designed to exhaust the AI agent's processing capacity. The SOC loses AI-assisted investigation capability at the moment it is needed most.

**Defensive Controls:**

| Control | Implementation |
|---|---|
| Input length limits | Enforce maximum token limits for all inputs processed by the LLM |
| Rate limiting | Implement per-source rate limits on data ingested by the AI system |
| Queue management | Use an ingestion queue with priority weighting. High-severity alerts are processed before bulk log data. |
| Graceful degradation | Define a fallback mode where the SOC operates on manual triage if the AI system becomes unavailable |
| Resource monitoring | Monitor LLM processing latency and resource consumption. Alert on anomalous spikes. |

**Detection Indicators:**
- Sudden increase in processing latency following a spike in a specific log source's volume
- AI agent queue depth increasing without corresponding increase in alert volume
- Resource exhaustion correlated with a specific data source or time window

---

## LLM05: Supply Chain Vulnerabilities

**Risk:** Third-party models, plugins, datasets, or infrastructure components introduce risk into the AI system.

**SOC Threat Scenario:** A third-party threat intelligence plugin used by the AI investigation agent is compromised. The plugin returns manipulated reputation scores, causing the agent to suppress indicators belonging to an active attacker.

**Defensive Controls:**

| Control | Implementation |
|---|---|
| Vendor security assessment | Conduct a security assessment of all third-party AI components before integration |
| Plugin permission minimization | Third-party plugins are granted only the minimum permissions required for their function |
| Output verification | Do not rely solely on a single third-party source for any risk determination. Corroborate against at least one independent source. |
| Dependency inventory | Maintain a current inventory of all AI system dependencies including models, plugins, datasets, and infrastructure |
| Update validation | Test all third-party updates in a staging environment before production deployment |

**Detection Indicators:**
- Reputation verdicts from a plugin diverge significantly from other sources
- Unexpected changes in plugin behavior following an update
- Third-party API returning anomalous response patterns

---

## LLM06: Sensitive Information Disclosure

**Risk:** The LLM reveals sensitive information from its training data, system prompt, or context window.

**SOC Threat Scenario:** An analyst uses a cloud-hosted LLM for threat investigation. The model's context window contains sensitive incident data including IOCs, affected systems, and victim asset details. This data may be exposed through model outputs, logs, or vendor telemetry.

**Defensive Controls:**

| Control | Implementation |
|---|---|
| Data classification before ingestion | Apply data classification to all content before it enters the LLM context. Do not ingest data classified above the permitted sensitivity level. |
| PII and sensitive data masking | Mask or tokenize sensitive identifiers (IP addresses, hostnames, employee names) before LLM ingestion where the LLM does not require the actual values |
| System prompt confidentiality | Do not expose system prompt contents in model outputs. Test specifically for system prompt extraction attempts. |
| Vendor data handling review | For cloud-hosted LLMs, review and document the vendor's data handling, logging, and training policies |
| Context window scoping | Limit the context window to only the data required for the current task. Do not maintain persistent context across unrelated investigations. |

**Detection Indicators:**
- LLM output contains data not present in the current session's input
- Model produces outputs referencing historical incidents when queried about a current one
- System prompt fragments appear in model responses

---

## LLM07: Insecure Plugin Design

**Risk:** LLM plugins lack adequate access controls, input validation, or permission boundaries, enabling unauthorized actions.

**SOC Threat Scenario:** A SOAR integration plugin connected to the AI agent accepts action requests without validating that the requested action is within the approved permission tier for the target asset. An injected instruction causes the plugin to isolate a Tier 1 host.

**Defensive Controls:**

| Control | Implementation |
|---|---|
| Plugin permission model | Every plugin enforces the tiered autonomy model independently. Tier 1 protections are enforced at the plugin level, not the orchestration layer. |
| Input validation at plugin boundary | Each plugin validates inputs independently. Do not trust that upstream orchestration has validated the request. |
| Allowlisted actions only | Plugins accept only explicitly defined action types. Any action request outside the allowlist is rejected with an alert. |
| Plugin authentication | All plugin-to-system interactions use authenticated, audited API calls. No unauthenticated plugin actions. |
| Least privilege integration | SOAR, firewall, and endpoint integrations operate under dedicated service accounts with minimum required permissions |

**Detection Indicators:**
- Plugin receives action requests for assets outside its defined scope
- Plugin API calls originating from unexpected contexts or at unexpected times
- Action requests that do not correspond to an active investigation case

---

## LLM08: Excessive Agency

**Risk:** The LLM is granted more autonomy, permissions, or capabilities than are necessary, amplifying the impact of errors or manipulation.

**SOC Threat Scenario:** An AI investigation agent is configured with write access to firewall policy, endpoint isolation controls, DNS blocking, and Active Directory. A single incorrect decision or injected instruction can now cause multi-system impact across the entire security stack.

**Defensive Controls:**

| Control | Implementation |
|---|---|
| Minimum capability principle | Grant the AI agent only the capabilities required for its defined function. Review and remove any capability not actively used. |
| Tiered autonomy enforcement | Implement the AI agent permission model. Autonomous action is restricted to Tier 3 assets by default. |
| Capability audit | Document every capability granted to the AI agent. Review quarterly. Remove unused capabilities immediately. |
| Blast radius assessment | Before granting any new capability, assess the maximum damage possible if that capability is misused or triggered incorrectly. |
| Time-bounded elevated access | If elevated access is required for a specific investigation, grant it for a defined window only. Revoke automatically on expiry. |

**Detection Indicators:**
- Agent exercising capabilities not required for the current task
- Multi-system actions triggered from a single investigation event
- Capability usage patterns inconsistent with the agent's defined workflow

---

## LLM09: Overreliance

**Risk:** Analysts or systems trust LLM outputs without adequate verification, treating AI verdicts as authoritative.

**SOC Threat Scenario:** SOC analysts consistently accept AI investigation verdicts without independent review. When the AI model begins producing incorrect verdicts due to data drift or a poisoned input, the errors propagate into closed cases without detection.

**Defensive Controls:**

| Control | Implementation |
|---|---|
| Confidence thresholds | Require human review for all AI verdicts below a defined confidence threshold. Document the threshold and its rationale. |
| Mandatory review for high-impact actions | Any AI recommendation that would affect a Tier 1 or Tier 2 asset requires independent analyst verification regardless of confidence score. |
| Blind sample review | Periodically present analysts with AI verdicts without revealing the confidence score. Compare human and AI verdicts. Track divergence over time. |
| Analyst override documentation | Document every instance where an analyst overrides an AI verdict. Use this data to identify systematic errors. |
| AI verdict transparency | The AI system must provide reasoning and evidence for every verdict. Analysts must be able to inspect the evidence, not just the conclusion. |

**Detection Indicators:**
- Analyst override rate drops significantly without a corresponding improvement in confirmed accuracy
- Closed case rate increases without a corresponding increase in incident volume
- Analysts unable to explain the basis for an AI verdict when asked

---

## LLM10: Model Theft

**Risk:** The AI model's parameters, behavior, or proprietary configuration is extracted through API queries or other means.

**SOC Threat Scenario:** An adversary who has established a foothold in the network uses the internal AI investigation agent's API to probe its detection logic, identify blind spots, and map the detection rules it relies on. This intelligence is used to evade future detection.

**Defensive Controls:**

| Control | Implementation |
|---|---|
| API access controls | Restrict AI system API access to authenticated, authorized SOC systems and personnel only |
| Query rate limiting | Implement rate limits on API queries to prevent systematic probing |
| Query logging and anomaly detection | Log all queries to the AI system. Alert on query patterns consistent with capability probing. |
| Detection logic confidentiality | Do not expose detection rule content, threshold values, or model configuration through AI system outputs |
| Internal deployment preference | Where possible, deploy AI models internally rather than relying on cloud-hosted endpoints that expose query patterns to third parties |

**Detection Indicators:**
- High-volume, systematic queries to the AI API from a single source
- Queries designed to probe the boundaries of detection logic rather than investigate specific alerts
- API access from hosts or accounts that do not normally interact with the AI system

---

## References

- OWASP Top 10 for LLM Applications v1.0: https://owasp.org/www-project-top-10-for-large-language-model-applications/
