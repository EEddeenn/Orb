# 08 — Temporal Evidence Knowledge Graph Architecture

**Project:** Orb  
**Version:** 0.1  
**Date:** 2026-05-07  

> **Document type:** Architecture manuscript.  
> **Scope boundary:** This document describes module responsibilities, architectural boundaries, canonical objects, data flows, and integration contracts. It intentionally avoids implementation details such as code, deployment topology, specific model prompts, database tuning, vendor procurement, or operational runbooks.  
> **Parent manuscript:** Orb, Version 0.1.  
> **Core principle:** OSINT grounds evidence; the knowledge graph defines the truth boundary; the agent produces scenario-specific hypotheses; the simulation trace remains separate from real-person facts.


## 1. Architectural Purpose

The Temporal Evidence Knowledge Graph is the platform's truth boundary. It stores sources, evidence units, identity associations, claims, events, relations, consent scopes, review decisions, sensitivity labels, third-party controls, and time validity.

The core architectural question is:

```text
What do we know, from where, at what time, under which consent scope,
with what confidence, and for which purposes may it be used?
```

## 2. Reference Patterns and Existing Systems

This module references:

- Neo4j's property graph and graph data science ecosystem for representing nodes, relationships, and graph workflows.[^neo4j]
- OpenCTI's threat intelligence knowledge graph pattern, where connectors feed structured intelligence objects into a graph.[^opencti]
- Senzing's concept of entity-resolved knowledge graphs.[^senzing]
- Graphistry's visual graph intelligence patterns for analyst-facing exploration.[^graphistry]

The adaptation is temporal, consent-aware, and simulation-safe. The graph is not simply an intelligence graph; it governs what can and cannot become agent state.

## 3. Position in the Platform

```text
Evidence Ingestion
  → Entity Resolution
  → Claim/Event/Relation Extraction
  → Temporal Evidence KG
  → Review and Adjudication
  → Agent State Graph
  → Simulation Trace Graph
```

## 4. Core Responsibilities

- Store source, evidence, claim, event, relation, identity, consent, and review objects.
- Preserve provenance and evidence chains.
- Preserve time semantics: observed, published, captured, verified, valid, expired.
- Preserve review state and usability flags.
- Represent sensitivity and third-party restrictions.
- Provide queryable approved claim sets.
- Support historical replay and as-of-time views.
- Prevent simulation trace hypotheses from contaminating real-person evidence.

## 5. Non-Responsibilities

This module does not:

- Crawl sources.
- Generate claims.
- Approve review decisions by itself.
- Build LLM prompts directly.
- Run simulations.

## 6. Logical Component Architecture

```text
Temporal Evidence Knowledge Graph Module
 ├── Evidence Graph Store Interface
 ├── Identity Boundary Subgraph
 ├── Claim Graph
 ├── Event Graph
 ├── Consent Scope Subgraph
 ├── Review Decision Subgraph
 ├── Sensitivity and Third-Party Restriction Layer
 ├── Temporal Validity Layer
 ├── Approved Claim Set View
 ├── Historical Replay View
 ├── Agent-State Export View
 └── Trace Separation Guard
```

## 7. Node Families

```text
Person
Agent
Source
EvidenceUnit
CaptureRecord
Claim
Event
Relation
Account
Organization
Topic
Location
ConsentScope
ReviewDecision
PolicyDecision
ThirdPartySubject
SensitivityLabel
RetentionPolicy
DerivedArtifactPolicy
```

## 8. Edge Families

```text
captured_from
derived_from
supported_by
contradicted_by
within_scope_of
reviewed_by
authorized_by
governed_by
expires_at
valid_during
confirmed_controls
possibly_controls
rejected_as
disputed_as
authored
attended
affiliated_with
mentioned_in
contains_third_party
restricted_by
usable_for
not_usable_for
```

## 9. Time Semantics

The graph must preserve distinct time fields.

| Field | Meaning |
|---|---|
| `observed_at` | When the real-world event or statement occurred. |
| `published_at` | When the source was published. |
| `captured_at` | When the platform captured it. |
| `verified_at` | When review confirmed or adjudicated it. |
| `valid_from` | Earliest date for which the claim/relation is valid. |
| `valid_until` | Expiration or end of validity. |
| `decayed_weight` | Retrieval weight after temporal decay. |
| `as_of_time` | Query context for historical replay. |

## 10. Review and Usability Fields

Every claim/relation/event should include:

```text
review_status
review_decision_id
usable_for_claim_extraction
usable_in_agent_state
usable_in_individual_output
usable_in_aggregate_output
usable_for_historical_replay
usable_for_calibration
```

Review states:

```text
pending
approved
approved_with_restrictions
aggregate_only
restricted
rejected
disputed
expired
withdrawn
```

## 11. Approved Claim Set View

The Approved Claim Set is not a separate uncontrolled copy. It is a filtered graph view.

Filter dimensions:

```text
person_id
agent_id
purpose_id
consent_scope_id
review_status
source_type
sensitivity_label
third_party_minimization_status
valid_time
confidence_threshold
output_eligibility
```

Output:

```text
approved claims
approved events
approved relations
approved behavioral signals
excluded/restricted summary
uncertainty annotations
```

## 12. Historical Replay View

Historical replay requires an as-of-time graph.

```text
as_of_date = D
include only sources published/captured before D
include only claims valid as of D
apply consent and review state as allowed by replay policy
build agent state as of D
compare simulation output against later known data
```

Historical replay must not use future evidence.

## 13. Three-Graph Directionality

```text
Temporal Evidence Graph
  → Agent State Graph
  → Simulation Trace Graph
```

Forbidden automatic reverse flow:

```text
Simulation Trace Graph ↛ Temporal Evidence Graph
Simulation Trace Graph ↛ Real Person Profile
```

Manual review may create annotations, but simulation hypotheses are not evidence facts.

## 14. Graph Object Example

```json
{
  "claim_id": "claim_001",
  "node_type": "Claim",
  "subject": "person_001",
  "predicate": "publicly_discussed_topic",
  "object": "supply_chain_resilience",
  "source_ids": ["src_001"],
  "evidence_ids": ["ev_001"],
  "consent_scope_id": "scope_001",
  "review_status": "approved",
  "sensitivity_label": "normal",
  "third_party_minimization_status": "not_applicable",
  "valid_from": "2025-04-12",
  "valid_until": "2027-05-07",
  "usable_in_agent_state": true,
  "usable_in_individual_output": true,
  "confidence": 0.91
}
```

## 15. Integration Contracts

| Module | Contract |
|---|---|
| Evidence Ingestion | Receives source/evidence/provenance nodes. |
| Entity Resolution | Receives identity boundary nodes/edges. |
| Claim Extraction | Receives candidate claims/events/relations. |
| Review | Updates review decisions and usability flags. |
| Agent Construction | Queries approved claim set and uncertainty annotations. |
| Retrieval | Queries purpose-filtered evidence packets. |
| Simulation Trace | Writes trace to a separate graph, not evidence graph. |
| Evaluation | Queries historical replay and grounding metrics. |

## 16. Architectural Invariants

```text
No claim without evidence pointer.
No claim without consent scope.
No agent-usable claim without review state.
No temporal claim without validity semantics.
No simulation hypothesis in evidence graph.
No third-party context without minimization status.
```

## 17. MVP Boundary

Minimum viable graph:

```text
Person, Source, EvidenceUnit, Claim, ConsentScope, ReviewDecision
supported_by / derived_from / within_scope_of / reviewed_by edges
time fields
review and usability flags
approved claim set view
```

Deferred:

```text
large graph analytics
complex ontology reasoning
cross-domain semantic alignment
advanced visualization workspaces
federated graph exchange
```

## 18. References

[^neo4j]: Neo4j documentation: https://neo4j.com/docs/
[^opencti]: OpenCTI documentation: https://docs.opencti.io/latest/
[^senzing]: Senzing, entity-resolved knowledge graphs: https://senzing.com/knowledge-graph/
[^graphistry]: Graphistry visual graph intelligence: https://www.graphistry.com/
