# 01 — Governance and Consent Architecture

**Project:** Orb  
**Version:** 0.1  
**Date:** 2026-05-07  

> **Document type:** Architecture manuscript.  
> **Scope boundary:** This document describes module responsibilities, architectural boundaries, canonical objects, data flows, and integration contracts. It intentionally avoids implementation details such as code, deployment topology, specific model prompts, database tuning, vendor procurement, or operational runbooks.  
> **Parent manuscript:** Orb, Version 0.1.  
> **Core principle:** OSINT grounds evidence; the knowledge graph defines the truth boundary; the agent produces scenario-specific hypotheses; the simulation trace remains separate from real-person facts.


## 1. Architectural Purpose

The Governance and Consent module is the control plane for the entire platform. It defines what the system is allowed to collect, derive, store, retrieve, simulate, show, export, retain, or delete for each consented individual and each approved use case.

This module must not be treated as an onboarding form or a legal-document archive. It is a runtime architecture component. Every downstream module must receive policy decisions from it before performing sensitive actions.

The core architectural question is:

```text
Given actor A, person P, purpose U, artifact X, and requested action R,
is this action allowed, denied, redacted, aggregate-only, or review-required?
```

## 2. Position in the Platform

```text
Governance and Consent
  ├── gates source discovery
  ├── gates OSINT capture
  ├── gates claim extraction
  ├── gates identity linking
  ├── gates agent construction
  ├── gates retrieval and evidence packets
  ├── gates simulation execution
  ├── gates output rendering and export
  ├── gates deletion and withdrawal
  └── receives audit and evaluation feedback
```

This module is cross-cutting. It is queried by all other modules and should be considered a dependency of every critical data flow.

## 3. Core Responsibilities

The module is responsible for:

- Maintaining a machine-readable consent ledger for each person-agent pair.
- Defining approved purposes, prohibited purposes, and output restrictions.
- Managing OSINT scope addenda for each individual or project.
- Classifying project and run-level risk.
- Producing policy decisions for requested actions.
- Maintaining subject rights, withdrawal, correction, and restriction states.
- Defining retention and derived-artifact deletion policies.
- Routing high-risk actions to human review, DPIA/PIA, or denial.
- Maintaining audit records for decisions, access, simulation runs, and exports.
- Preventing forbidden use cases even when a downstream component technically can perform them.

## 4. Non-Responsibilities

This module does not:

- Perform OSINT discovery or crawling.
- Extract claims from text.
- Resolve identity links.
- Generate agent responses.
- Run simulations.
- Decide whether a claim is factually true without evidence and review.

It authorizes, restricts, records, and governs those actions.

## 5. Logical Component Architecture

```text
Governance and Consent Module
 ├── Consent Ledger
 ├── Individual Simulation Contract Registry
 ├── OSINT Scope Manager
 ├── Purpose Registry
 ├── Prohibited Use Registry
 ├── Risk Classifier
 ├── Policy Engine
 ├── Review-Routing Rules
 ├── Subject Rights State Manager
 ├── Retention and Deletion Policy Manager
 ├── Derived Artifact Control Registry
 ├── Audit Event Collector
 └── Governance Reporting View
```

### 5.1 Consent Ledger

The Consent Ledger stores current and historical authorization state. It must be versioned because later simulations need to know which consent version authorized a given run.

Canonical fields:

| Field | Purpose |
|---|---|
| `consent_id` | Unique consent record. |
| `person_id` | Real individual. |
| `agent_id` | Associated agent. |
| `contract_id` | Individual Simulation Contract. |
| `consent_status` | `active`, `restricted`, `withdrawal_requested`, `withdrawn`, `expired`. |
| `approved_purposes` | Purpose identifiers allowed by the individual. |
| `prohibited_purposes` | Explicitly forbidden purposes. |
| `allowed_source_types` | OSINT or self-report source categories allowed. |
| `excluded_source_types` | Source categories forbidden even if publicly accessible. |
| `allowed_outputs` | Output types allowed. |
| `prohibited_outputs` | Output types forbidden. |
| `recipients_allowed` | Actors or roles allowed to view outputs. |
| `retention_policy_id` | Governs raw and derived artifacts. |
| `jurisdiction_profile` | Applicable jurisdictional controls. |
| `notice_version` | Notice/consent language version. |
| `effective_from` | Start timestamp. |
| `effective_until` | End timestamp, if any. |
| `withdrawal_state` | Withdrawal workflow state. |

### 5.2 Individual Simulation Contract Registry

The Individual Simulation Contract defines the relationship between the person, the platform operator, approved purposes, and permissible data/use boundaries.

Architectural sections:

```text
Identity section
Authorization section
OSINT scope section
Output and recipient section
Review and subject-rights section
Retention and deletion section
Risk and escalation section
```

The contract is not a PDF-only object. It must produce policy-readable permissions and restrictions.

### 5.3 OSINT Scope Manager

The OSINT Scope Manager prevents public-data creep. It specifies what public sources are in scope for each person or project.

Canonical fields:

| Field | Purpose |
|---|---|
| `scope_id` | OSINT scope record. |
| `person_id` / `project_id` | Scope owner. |
| `allowed_source_types` | Public professional profiles, self-authored posts, publications, public talks, etc. |
| `excluded_source_types` | Leaked data, private groups, unverified gossip, doxxing databases, biometric scraping. |
| `time_range` | Collection and validity window. |
| `topic_scope` | Domain and topic boundaries. |
| `identity_confirmation_rule` | What is required before a source can be linked. |
| `review_required` | Whether participant or reviewer approval is mandatory. |
| `sensitive_category_rule` | Default handling for sensitive data. |
| `third_party_handling_rule` | Redaction, generalization, or exclusion policy. |
| `embedding_policy` | Whether raw evidence, approved claims, or no content may be vectorized. |

### 5.4 Purpose Registry

Purposes must be explicit and stable. A vague purpose such as “analytics” is not sufficient.

Example purpose taxonomy:

```text
personal_self_simulation
research_simulation
communication_rehearsal
organizational_tabletop_exercise
aggregate_market_research
policy_response_simulation
training_and_roleplay
```

Each purpose has:

```text
permitted source categories
permitted output types
default risk class
review requirements
retention defaults
allowed recipients
prohibited transitions
```

### 5.5 Policy Engine

The Policy Engine evaluates structured requests and emits structured decisions.

Request shape:

```json
{
  "actor_id": "user_001",
  "person_id": "person_001",
  "agent_id": "agent_001",
  "purpose_id": "research_simulation",
  "artifact_id": "claim_001",
  "requested_action": "retrieve_for_simulation",
  "scenario_id": "scenario_001",
  "output_type": "individual_probabilistic_response"
}
```

Decision shape:

```json
{
  "decision_id": "decision_001",
  "decision": "allow",
  "reason_codes": ["approved_purpose", "reviewed_claim", "no_sensitive_data"],
  "rule_ids": ["purpose_allowed", "claim_review_status_required"],
  "review_required": false,
  "redaction_required": false,
  "aggregate_only": false,
  "timestamp": "2026-05-07T00:00:00Z"
}
```

Decision values:

```text
allow
deny
redact
aggregate_only
review_required
restricted_until_review
block_due_to_withdrawal
block_due_to_expiry
```

### 5.6 Risk Classifier

Risk classification operates at project, person, source, claim, scenario, output, and export levels.

Risk classes:

| Class | Architecture meaning |
|---|---|
| Green | Low-risk self-simulation or sandbox with no external consequence. |
| Yellow | Group or organizational scenario simulation with controlled outputs. |
| Red | Individual-level, sensitive, workplace, education, finance, health, public-sector, or reputational impact context. |
| Black | Prohibited: impersonation, manipulation, social scoring, unauthorized sensitive inference, criminal risk prediction, high-impact automated decisioning. |

### 5.7 Retention and Deletion Policy Manager

Deletion is not a single operation. It must propagate to raw evidence, normalized evidence, derived claims, embeddings, agent cards, evidence packets, simulation traces, exports, and backups.

Artifact classes:

```text
raw_source_capture
normalized_evidence_unit
claim_object
identity_boundary_record
agent_card
evidence_packet
prompt_or_context_log
simulation_output
simulation_trace
evaluation_report
embedding_or_vector_index
export
backup
audit_record
```

Deletion states:

```text
active
collection_blocked
retrieval_blocked
simulation_blocked
raw_deleted
derived_retracted
index_purged
export_revoked_or_stale
backup_pending_expiry
audit_minimized
complete
```

## 6. Primary Flows

### 6.1 Agent Creation Authorization

```text
new person-agent request
  → validate person in scope
  → create Individual Simulation Contract
  → define purposes
  → define OSINT Scope Addendum
  → assign risk level
  → register consent version
  → enable source intake
```

### 6.2 Runtime Policy Check

```text
requested action
  → assemble request context
  → evaluate person consent
  → evaluate project purpose
  → evaluate artifact state
  → evaluate source scope
  → evaluate sensitivity and third-party rules
  → emit policy decision
  → record audit event
```

### 6.3 Withdrawal Propagation

```text
withdrawal request
  → block future collection
  → block retrieval
  → block simulation
  → retract agent card
  → purge or tombstone artifacts according to policy
  → retain minimal audit record if required
  → notify dependent modules
  → update subject portal
```

## 7. Integration Contracts

| Downstream module | Governance output required |
|---|---|
| OSINT Source Governance | Allowed source types, excluded source types, time range, topic scope. |
| OSINT Evidence Ingestion | Capture decision, retention, sensitivity default, derived artifact policy. |
| Entity Resolution | Identity confirmation rule, review requirement, usable-in-simulation flag. |
| Claim Extraction | Permitted claim types, sensitive-category rules, review routing. |
| Temporal KG | Consent-scope edges, validity and expiry rules. |
| Review & Adjudication | Decision vocabulary, escalation policy. |
| Agent Construction | Approved claim set, excluded knowledge, agent policy. |
| Retrieval | Evidence packet policy decision. |
| Simulation | Scenario authorization, participant authorization, output restrictions. |
| Evaluation | Permitted calibration targets, prohibited writeback targets. |

## 8. Architectural Invariants

```text
No source without scope.
No claim without provenance.
No agent state without approval.
No retrieval without purpose authorization.
No individual output without recipient authorization.
No simulation result becomes a personal fact automatically.
No withdrawn artifact remains simulation-usable.
```

## 9. Architecture Risks

| Risk | Architectural control |
|---|---|
| Consent becomes a checkbox | Machine-readable contract and runtime policy decisions. |
| Purpose creep | Purpose registry and purpose-based access control. |
| Public data creep | OSINT Scope Addendum and source registry. |
| Derived artifact persistence | Deletion propagation model. |
| Unreviewed sensitive data enters agent state | Sensitive data queue and review routing. |
| Simulation output used for high-impact decisions | Output restrictions and prohibited use registry. |
| Governance exists only in prompts | Policy-as-code decisions outside the LLM. |

## 10. MVP Boundary

Minimum architecture for MVP:

```text
Consent Ledger
Individual Simulation Contract
OSINT Scope Addendum
Purpose Registry
Policy Decision object
Audit Event object
Withdrawal/Deletion Propagation state
```

Deferred but planned:

```text
jurisdiction-specific rule packs
full DPIA/PIA workflow automation
cross-organization consent federation
advanced differential output policies
formal policy verification
```

## 11. References

- NIST AI Risk Management Framework: https://airc.nist.gov/airmf-resources/airmf/
- GDPR Article 5, personal data principles: https://gdpr-info.eu/art-5-gdpr/
- GDPR Article 9, special categories of data: https://gdpr-info.eu/art-9-gdpr/
- GDPR Article 22, automated decision-making and profiling: https://gdpr-info.eu/art-22-gdpr/
- GDPR Article 35, DPIA: https://gdpr-info.eu/art-35-gdpr/
- European Commission AI Act overview: https://digital-strategy.ec.europa.eu/en/policies/regulatory-framework-ai
- OAIC guidance on generative AI and privacy: https://www.oaic.gov.au/privacy/privacy-guidance-for-organisations-and-government-agencies/guidance-on-privacy-and-developing-and-training-generative-ai-models
