# 02 — Participant and Subject Rights Architecture

**Project:** Orb  
**Version:** 0.1  
**Date:** 2026-05-07  

> **Document type:** Architecture manuscript.  
> **Scope boundary:** This document describes module responsibilities, architectural boundaries, canonical objects, data flows, and integration contracts. It intentionally avoids implementation details such as code, deployment topology, specific model prompts, database tuning, vendor procurement, or operational runbooks.  
> **Parent manuscript:** Orb, Version 0.1.  
> **Core principle:** OSINT grounds evidence; the knowledge graph defines the truth boundary; the agent produces scenario-specific hypotheses; the simulation trace remains separate from real-person facts.


## 1. Architectural Purpose

The Participant and Subject Rights module provides the consented individual with visibility and control over their OSINT-grounded agent. It operationalizes the principle that a real-person agent should be inspectable, correctable, restrictable, exportable, and revocable by the person it represents.

The core architectural question is:

```text
How does a consented individual govern the evidence, claims, agent state,
and simulation uses associated with them?
```

## 2. Position in the Platform

```text
Participant and Subject Rights
  ↔ Governance and Consent
  ↔ OSINT Source Governance
  ↔ Evidence Ingestion and Provenance
  ↔ Review and Adjudication
  ↔ Agent Construction
  ↔ Simulation Trace and Output History
```

This module is the person-facing control surface for governance. It should be treated as an architectural component, not merely a UI.

## 3. Core Responsibilities

The module is responsible for:

- Participant onboarding and scope acknowledgement.
- Source confirmation: “this is me / this is not me.”
- Claim review: approve, reject, dispute, restrict, or mark stale.
- Agent card preview and state visibility.
- Simulation participation controls.
- Simulation history visibility.
- Correction requests.
- Deletion and withdrawal requests.
- Export of participant-readable agent records.
- Contesting outputs or review decisions.
- Routing decisions to the Governance and Review modules.

## 4. Non-Responsibilities

This module does not:

- Decide legal basis by itself.
- Crawl OSINT sources.
- Automatically approve evidence.
- Generate simulation outputs.
- Replace institutional review, legal review, or DPIA/PIA where required.

## 5. Logical Component Architecture

```text
Participant and Subject Rights Module
 ├── Participant Onboarding Surface
 ├── Consent Summary Viewer
 ├── OSINT Source Confirmation Workbench
 ├── Claim Review Workbench
 ├── Agent Card Preview
 ├── Simulation Participation Controls
 ├── Simulation History Viewer
 ├── Correction and Dispute Workflow
 ├── Withdrawal Workflow
 ├── Export Request Manager
 └── Participant Notification Center
```

## 6. Participant-Facing Data Views

### 6.1 Consent Summary View

Shows the current authorization boundary in plain language and structured form.

Fields:

```text
person_id
agent_id
approved_purposes
prohibited_purposes
allowed_source_types
excluded_source_types
allowed_output_types
output_recipients
retention_summary
withdrawal_options
last_policy_update
```

### 6.2 Source Confirmation View

Presents candidate public sources and asks the participant to confirm, reject, restrict, or dispute them.

Fields:

| Field | Meaning |
|---|---|
| `candidate_source_id` | Candidate source. |
| `source_type` | Profile, article, talk, event, organization page, etc. |
| `source_summary` | Participant-readable summary. |
| `identity_reason` | Why the system thinks it may be associated. |
| `risk_label` | Normal, sensitive, third-party-heavy, stale, etc. |
| `proposed_use` | Individual simulation, aggregate-only, discovery-only. |
| `participant_decision` | Confirm, reject, restrict, dispute, needs context. |

### 6.3 Claim Review View

Presents extracted claims for adjudication.

Decision options:

```text
approve
reject
dispute
correct
mark_outdated
aggregate_only
restricted_to_specific_purpose
sensitive_do_not_use
third_party_redaction_required
```

### 6.4 Agent Card Preview

Displays what the agent can use, not every raw evidence item.

Sections:

```text
identity boundary
approved sources
approved claims
communication style indicators
decision constraints
known uncertainties
excluded topics
simulation restrictions
revocation status
```

### 6.5 Simulation History Viewer

Shows simulations involving the participant.

Fields:

```text
run_id
scenario_id
purpose
participants
recipient_roles
evidence_packet_summary
output_summary
confidence_summary
policy_decisions
review_status
contest_option
```

## 7. Subject Rights Request Lifecycle

```text
request submitted
  → request classified
  → policy engine validates scope
  → affected artifacts identified
  → workflow routed to module owners
  → action completed or partially completed
  → participant notified
  → audit record stored
```

Request types:

```text
access
correction
deletion
withdrawal
restriction
export
contest_output
contest_identity_link
contest_claim
restrict_source
restrict_output_recipient
```

## 8. Canonical Objects

### 8.1 Subject Rights Request

```json
{
  "request_id": "srr_001",
  "person_id": "person_001",
  "request_type": "correction",
  "artifact_ids": ["claim_001"],
  "requested_action": "mark_outdated_and_replace",
  "reason": "affiliation ended in 2024",
  "status": "under_review",
  "submitted_at": "2026-05-07T00:00:00Z",
  "required_modules": ["review", "temporal_kg", "agent_construction"],
  "policy_decision_id": "decision_001"
}
```

### 8.2 Participant Review Decision

```json
{
  "review_decision_id": "prd_001",
  "person_id": "person_001",
  "artifact_id": "claim_001",
  "artifact_type": "claim",
  "decision": "approved_for_research_aggregate_only",
  "scope_restriction": "aggregate_only",
  "notes": "Accurate but should not be used in individual outputs.",
  "decided_at": "2026-05-07T00:00:00Z"
}
```

### 8.3 Participant Notification

```json
{
  "notification_id": "notif_001",
  "person_id": "person_001",
  "notification_type": "new_claim_review_required",
  "related_artifact_ids": ["claim_001", "claim_002"],
  "action_required": true,
  "deadline_policy": "do_not_use_until_reviewed"
}
```

## 9. Architectural Flows

### 9.1 Onboarding Flow

```text
participant invited
  → identity verified by approved process
  → Individual Simulation Contract displayed
  → OSINT Scope Addendum displayed
  → allowed and excluded sources confirmed
  → initial consent state created
  → source intake enabled
```

### 9.2 Source Confirmation Flow

```text
candidate source discovered
  → identity reason generated
  → risk label assigned
  → participant confirms/rejects/restricts
  → Entity Boundary module updated
  → approved sources proceed to evidence processing
```

### 9.3 Claim Review Flow

```text
claim candidate extracted
  → sensitivity and third-party labels attached
  → participant/reviewer adjudication requested
  → decision recorded
  → Temporal KG updated
  → Agent State Builder receives only approved claims
```

### 9.4 Withdrawal Flow

```text
participant requests withdrawal
  → Governance blocks collection/retrieval/simulation
  → deletion propagation starts
  → agent card revoked
  → evidence packets and indexes purged or tombstoned
  → simulation traces minimized according to retention policy
  → participant receives completion status
```

## 10. Integration Contracts

| Module | Required integration |
|---|---|
| Governance & Consent | Reads consent state; writes subject rights requests and review restrictions. |
| OSINT Source Governance | Receives participant-confirmed allowed/excluded source lists. |
| Entity Resolution | Receives confirmation/rejection of identity links. |
| Claim Extraction | Sends candidate claims to review queue. |
| Temporal KG | Stores review decisions and validity annotations. |
| Agent Construction | Receives approved claim set and excluded topics. |
| Simulation Trace | Exposes run involvement and contestable outputs. |
| Evaluation | Receives participant feedback without rewriting personal facts automatically. |

## 11. Architectural Invariants

```text
Participant rejection overrides automated association.
Participant correction triggers graph and agent-state review.
Participant withdrawal blocks future simulation immediately.
Unreviewed candidate claims do not enter agent state.
Simulation history is visible at the level permitted by policy.
Participant feedback is evaluation data, not automatically new personal truth.
```

## 12. MVP Boundary

Minimum viable architecture:

```text
onboarding and consent summary
source confirmation workbench
claim review workbench
agent card preview
withdrawal request workflow
simulation history summary
correction/dispute request object
```

Deferred:

```text
fine-grained comparative explanation views
multi-jurisdictional rights automation
participant-to-participant consent negotiation
advanced delegation and guardian flows
```

## 13. References

- GDPR Article 15, right of access: https://gdpr-info.eu/art-15-gdpr/
- GDPR Article 16, right to rectification: https://gdpr-info.eu/art-16-gdpr/
- GDPR Article 17, right to erasure: https://gdpr-info.eu/art-17-gdpr/
- GDPR Article 18, restriction of processing: https://gdpr-info.eu/art-18-gdpr/
- OAIC guidance on privacy and generative AI: https://www.oaic.gov.au/privacy/privacy-guidance-for-organisations-and-government-agencies/guidance-on-privacy-and-developing-and-training-generative-ai-models
