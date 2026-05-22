# 07 — Claim, Event, and Relation Extraction Architecture

**Project:** Orb  
**Version:** 0.1  
**Date:** 2026-05-07  

> **Document type:** Architecture manuscript.  
> **Scope boundary:** This document describes module responsibilities, architectural boundaries, canonical objects, data flows, and integration contracts. It intentionally avoids implementation details such as code, deployment topology, specific model prompts, database tuning, vendor procurement, or operational runbooks.  
> **Parent manuscript:** Orb, Version 0.1.  
> **Core principle:** OSINT grounds evidence; the knowledge graph defines the truth boundary; the agent produces scenario-specific hypotheses; the simulation trace remains separate from real-person facts.


## 1. Architectural Purpose

The Claim, Event, and Relation Extraction module converts evidence units into structured, reviewable knowledge candidates. It prevents the platform from turning OSINT directly into a profile blob.

The core architectural question is:

```text
What structured statements, events, relations, and behavioral signals can be derived
from this evidence, with provenance, confidence, scope, and review status attached?
```

## 2. Position in the Platform

```text
Evidence Ingestion
  → Entity Resolution
  → Claim/Event/Relation Extraction
  → Temporal Evidence Knowledge Graph
  → Review and Adjudication
  → Agent Construction
```

## 3. Core Responsibilities

- Extract claim candidates from evidence.
- Extract event candidates with time, actors, actions, and source references.
- Extract relation candidates between people, accounts, organizations, events, topics, and claims.
- Extract weak behavioral signals, such as communication style indicators or topic familiarity, only as candidates.
- Attach provenance, identity confidence, source confidence, sensitivity labels, third-party handling status, and review requirement.
- Detect contradictions and stale evidence.
- Send candidates to Review and Temporal KG modules.

## 4. Non-Responsibilities

This module does not:

- Confirm identity associations.
- Approve claims for agent use.
- Decide final simulation behavior.
- Store real-person permanent profiles.
- Infer sensitive attributes outside authorization.

## 5. Logical Component Architecture

```text
Claim, Event, and Relation Extraction Module
 ├── Evidence Intake Adapter
 ├── Extraction Schema Registry
 ├── Claim Candidate Extractor
 ├── Event Candidate Extractor
 ├── Relation Candidate Extractor
 ├── Behavioral Signal Candidate Extractor
 ├── Claim Type Classifier
 ├── Confidence and Source-Quality Annotator
 ├── Contradiction and Staleness Detector
 ├── Sensitivity and Third-Party Label Propagator
 ├── Review Routing Annotator
 └── Candidate Knowledge Exporter
```

## 6. Extraction Object Families

```text
ClaimCandidate
EventCandidate
RelationCandidate
BehavioralSignalCandidate
ContradictionCandidate
StalenessAnnotation
ExtractionBatch
```

## 7. Claim Type Taxonomy

| Claim type | Meaning | Agent-state eligibility |
|---|---|---|
| `observed_fact` | Evidence-supported public fact. | Eligible after review. |
| `self_authored_statement` | Statement authored by the participant. | Eligible after review and scope check. |
| `self_report` | Directly provided by participant. | Eligible under contract. |
| `third_party_attribution` | Someone else says something about the person. | Low weight; review required. |
| `contextual_fact` | Industry, organization, or event context. | Usable as context, not personal trait. |
| `inferred_preference` | Weak behavioral hypothesis. | Review required; uncertainty high. |
| `sensitive_implication` | Sensitive attribute or inference. | Restricted by default. |
| `third_party_context` | Information about another person. | Redact/generalize/aggregate by default. |
| `simulation_hypothesis` | Generated in a simulation. | Must remain in trace, not evidence graph. |

## 8. Claim Object

```json
{
  "claim_id": "claim_001",
  "subject": "person_001",
  "predicate": "publicly_discussed_topic",
  "object": "supply_chain_resilience",
  "claim_type": "self_authored_statement",
  "source_ids": ["src_001"],
  "evidence_ids": ["ev_001"],
  "identity_boundary_ids": ["boundary_001"],
  "confidence": 0.91,
  "source_quality_score": 0.87,
  "sensitivity": "normal",
  "third_party_minimization_status": "not_applicable",
  "consent_scope_id": "scope_001",
  "review_status": "pending",
  "valid_from": "2025-04-12",
  "valid_until": "2027-05-07",
  "staleness_status": "current"
}
```

## 9. Event Object

```json
{
  "event_id": "event_001",
  "event_type": "public_talk",
  "participants": ["person_001"],
  "organizations": ["org_001"],
  "topics": ["topic_001"],
  "location": "public_event_location_or_generalized_location",
  "observed_at": "2025-05-01T00:00:00Z",
  "published_at": "2025-05-02T00:00:00Z",
  "source_ids": ["src_002"],
  "evidence_ids": ["ev_002"],
  "confidence": 0.88,
  "public_or_private": "public",
  "review_status": "pending",
  "consent_scope_id": "scope_001"
}
```

## 10. Relation Object

```json
{
  "relation_id": "rel_001",
  "subject_entity_id": "person_001",
  "predicate": "affiliated_with",
  "object_entity_id": "org_001",
  "relation_type": "person_organization_affiliation",
  "evidence_ids": ["ev_003"],
  "confidence": 0.93,
  "association_level": "organization_confirmed",
  "review_status": "approved",
  "valid_from": "2024-01-01",
  "valid_until": null,
  "usable_in_agent_state": true
}
```

## 11. Behavioral Signal Candidate

Behavioral signals are not facts. They are weak, reviewable hypotheses derived from approved evidence.

Examples:

```text
communication_style_indicator
recurring_topic_interest
decision_constraint_indicator
risk_concern_indicator
preferred_channel_indicator
public_role_behavior_indicator
```

Object:

```json
{
  "signal_id": "sig_001",
  "person_id": "person_001",
  "signal_type": "decision_constraint_indicator",
  "signal_value": "implementation_cost",
  "supporting_claim_ids": ["claim_001", "claim_002"],
  "confidence": 0.64,
  "uncertainty_note": "Derived from public professional writing; not a direct self-report.",
  "review_status": "review_required",
  "usable_in_agent_state": false
}
```

## 12. Contradiction and Staleness

The extraction module should not hide conflicting evidence.

Contradiction object:

```json
{
  "contradiction_id": "contra_001",
  "claim_ids": ["claim_001", "claim_009"],
  "conflict_type": "position_changed_or_source_conflict",
  "resolution_status": "review_required",
  "agent_state_action": "exclude_until_reviewed"
}
```

Staleness status:

```text
current
aging
stale
expired
superseded
disputed
```

## 13. Extraction Flow

```text
evidence unit received
  → confirm identity boundary eligibility
  → apply sensitivity and third-party labels
  → extract claim candidates
  → extract event candidates
  → extract relation candidates
  → extract weak behavioral signal candidates
  → assign confidence and staleness
  → detect contradictions
  → route to Temporal KG and Review
```

## 14. Integration Contracts

| Module | Contract |
|---|---|
| Evidence Ingestion | Receives evidence units and provenance. |
| Entity Resolution | Uses identity boundary eligibility. |
| Privacy/Sensitivity | Propagates sensitive and third-party labels. |
| Temporal KG | Stores candidate and approved claims/events/relations. |
| Review | Routes candidates with required decision fields. |
| Agent Construction | Receives only approved claim sets and approved signals. |
| Evaluation | Receives unsupported inference and contradiction metrics. |

## 15. Architectural Invariants

```text
Extraction candidate is not approved knowledge.
Behavioral signal is not a personal fact.
Third-party attribution is not self-authored statement.
Historical position is not current intent.
Contradictions must be preserved, not overwritten.
Sensitive implications are restricted by default.
```

## 16. MVP Boundary

Minimum viable extraction architecture:

```text
claim candidate schema
event candidate schema
relation candidate schema
claim type taxonomy
confidence fields
review status
source/evidence pointers
sensitivity and third-party label propagation
```

Deferred:

```text
advanced stance modeling
rich discourse modeling
multimodal extraction
complex causal relation extraction
automated contradiction resolution
```

## 17. References

- Babel Street entity and relationship mapping pattern: https://www.babelstreet.com/modules/entity-and-relationship-mapping
- OpenCTI knowledge graph and connector pattern: https://docs.opencti.io/latest/
- MISP structured intelligence object pattern: https://www.misp-project.org/
- Neo4j LLM Knowledge Graph Builder pattern: https://neo4j.com/labs/genai-ecosystem/llm-graph-builder/
