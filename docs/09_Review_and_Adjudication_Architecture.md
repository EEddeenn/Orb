# 09 — Review and Adjudication Architecture

**Project:** Orb  
**Version:** 0.1  
**Date:** 2026-05-07  

> **Document type:** Architecture manuscript.  
> **Scope boundary:** This document describes module responsibilities, architectural boundaries, canonical objects, data flows, and integration contracts. It intentionally avoids implementation details such as code, deployment topology, specific model prompts, database tuning, vendor procurement, or operational runbooks.  
> **Parent manuscript:** Orb, Version 0.1.  
> **Core principle:** OSINT grounds evidence; the knowledge graph defines the truth boundary; the agent produces scenario-specific hypotheses; the simulation trace remains separate from real-person facts.


## 1. Architectural Purpose

The Review and Adjudication module converts candidate sources, identity links, claims, events, relations, and behavioral signals into approved, restricted, disputed, rejected, or aggregate-only knowledge.

The core architectural question is:

```text
Which candidate artifacts are accurate, in-scope, current, non-sensitive enough,
and permitted for agent use or simulation output?
```

## 2. Position in the Platform

```text
Source Governance
Evidence Ingestion
Entity Resolution
Claim/Event/Relation Extraction
Privacy/Sensitivity
       ↓
Review and Adjudication
       ↓
Temporal Evidence KG
Agent Construction
Retrieval
Simulation Output
```

Review is the bridge between OSINT discovery and agent-usable state.

## 3. Core Responsibilities

- Maintain review queues for source candidates, identity links, evidence units, claims, relations, behavioral signals, and simulation outputs.
- Support participant decisions and authorized reviewer decisions.
- Store review decisions as graph objects.
- Mark artifacts as approved, restricted, disputed, rejected, aggregate-only, or expired.
- Capture correction and staleness annotations.
- Resolve conflicts and contradictions.
- Route sensitive and third-party-heavy items to special review.
- Provide approved claim sets to Agent Construction.

## 4. Non-Responsibilities

This module does not:

- Crawl sources.
- Automatically infer truth without evidence.
- Create simulation behavior.
- Override governance policy.
- Convert simulation hypotheses into real facts.

## 5. Logical Component Architecture

```text
Review and Adjudication Module
 ├── Review Queue Manager
 ├── Participant Review Interface Contract
 ├── Expert Reviewer Interface Contract
 ├── Artifact Review State Machine
 ├── Conflict Resolution Workspace
 ├── Sensitivity Review Router
 ├── Third-Party Review Router
 ├── Correction and Staleness Annotator
 ├── Approved Claim Set Publisher
 ├── Disputed Evidence Archive
 └── Review Audit Exporter
```

## 6. Reviewable Artifact Types

```text
SourceCandidate
EvidenceUnit
IdentityBoundaryRecord
ClaimCandidate
EventCandidate
RelationCandidate
BehavioralSignalCandidate
ContradictionCandidate
AgentStateEntry
EvidencePacket
SimulationOutput
EvaluationFlag
```

## 7. Review Decision Vocabulary

```text
approved
approved_with_restrictions
approved_for_aggregate_only
approved_for_historical_replay_only
restricted
rejected
disputed
needs_more_evidence
mark_outdated
corrected
redact_then_approve
generalize_then_approve
retain_audit_only
delete
```

## 8. Review Decision Object

```json
{
  "review_decision_id": "review_001",
  "artifact_id": "claim_001",
  "artifact_type": "ClaimCandidate",
  "person_id": "person_001",
  "reviewer_type": "participant",
  "decision": "approved_with_restrictions",
  "restriction": "aggregate_only",
  "reason_codes": ["accurate_but_personal_context"],
  "correction_text": null,
  "valid_from": "2025-04-12",
  "valid_until": "2027-05-07",
  "usable_in_agent_state": false,
  "usable_in_aggregate_output": true,
  "decided_at": "2026-05-07T00:00:00Z"
}
```

## 9. Review Queues

Recommended queues:

```text
source_confirmation_queue
identity_link_queue
claim_review_queue
sensitive_claim_queue
third_party_minimization_queue
contradiction_queue
staleness_queue
agent_card_preview_queue
simulation_output_review_queue
withdrawal_or_correction_queue
```

Queue item fields:

```text
queue_item_id
artifact_id
artifact_type
person_id
project_id
risk_label
required_reviewer_type
current_status
priority
policy_decision_id
due_or_expiry_policy
recommended_actions
```

## 10. Conflict Resolution

Conflicts should be first-class objects.

Conflict types:

```text
identity_collision
source_contradiction
participant_dispute
stale_public_profile
third_party_attribution_conflict
sensitive_category_uncertainty
reviewer_disagreement
```

Resolution states:

```text
unresolved
participant_resolved
reviewer_resolved
restricted_until_new_evidence
rejected
superseded
```

## 11. Approved Claim Set Publisher

This component publishes a filtered view to Agent Construction.

Requirements:

```text
only approved or approved-with-restrictions artifacts
include evidence references
include validity periods
include uncertainty notes
include output eligibility
include sensitivity restrictions
include aggregate-only restrictions
exclude rejected/disputed/expired artifacts
```

## 12. Architectural Flows

### 12.1 Claim Review Flow

```text
claim candidate created
  → review queue item created
  → participant/reviewer sees evidence and proposed claim
  → decision recorded
  → Temporal KG updates review status
  → approved claim set refreshes
  → Agent Construction receives new usable state if allowed
```

### 12.2 Identity Link Review Flow

```text
candidate identity link created
  → evidence and matching reasons displayed
  → participant confirms/rejects/disputes
  → identity boundary updated
  → dependent claims reclassified
```

### 12.3 Sensitive Review Flow

```text
sensitive candidate created
  → explicit authorization checked
  → restricted queue
  → reviewer decides redact/generalize/restrict/reject
  → downstream retrieval/output labels updated
```

## 13. Integration Contracts

| Module | Contract |
|---|---|
| Participant Rights | Supplies participant decisions and correction requests. |
| Governance | Supplies allowed decisions and review routing rules. |
| Entity Resolution | Sends candidate and conflicting identity links. |
| Claim Extraction | Sends candidate claims/events/relations/signals. |
| Privacy/Sensitivity | Sends restricted sensitive and third-party items. |
| Temporal KG | Stores review decisions and usability flags. |
| Agent Construction | Receives approved claim sets and restrictions. |
| Simulation Output | Routes high-risk outputs for review before release. |

## 14. Architectural Invariants

```text
Unreviewed OSINT is discovery material, not agent state.
Rejected artifacts cannot re-enter through another path without new review.
Aggregate-only artifacts cannot be used in individual outputs.
Participant corrections update validity and review state.
Review decisions are auditable graph objects.
```

## 15. MVP Boundary

Minimum viable architecture:

```text
review queue manager
review decision object
participant review for sources and claims
identity link review
approved/rejected/disputed/aggregate-only states
approved claim set publisher
```

Deferred:

```text
multi-reviewer adjudication panels
advanced reviewer disagreement resolution
formal evidence grading rubrics
automated review prioritization
review analytics dashboard
```
