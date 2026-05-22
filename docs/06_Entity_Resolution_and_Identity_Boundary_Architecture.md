# 06 — Entity Resolution and Identity Boundary Architecture

**Project:** Orb  
**Version:** 0.1  
**Date:** 2026-05-07  

> **Document type:** Architecture manuscript.  
> **Scope boundary:** This document describes module responsibilities, architectural boundaries, canonical objects, data flows, and integration contracts. It intentionally avoids implementation details such as code, deployment topology, specific model prompts, database tuning, vendor procurement, or operational runbooks.  
> **Parent manuscript:** Orb, Version 0.1.  
> **Core principle:** OSINT grounds evidence; the knowledge graph defines the truth boundary; the agent produces scenario-specific hypotheses; the simulation trace remains separate from real-person facts.


## 1. Architectural Purpose

The Entity Resolution and Identity Boundary module determines which accounts, sources, organizations, events, aliases, and claims are actually associated with a consented individual. It is the platform's identity safety layer.

The core architectural question is:

```text
Is this external entity truly associated with the consented person,
and is the association strong enough to be used in agent state or simulation?
```

## 2. Reference Patterns and Existing Products

This module references entity resolution and investigative graph patterns from:

- Senzing, which emphasizes entity-resolved knowledge graphs and identifying consolidated nodes that represent the same real-world entity.[^senzing]
- Quantexa, which describes entity resolution as a foundation for trusted, contextual decision intelligence.[^quantexa]
- DataWalk, which combines entity resolution, graph, and investigative intelligence patterns.[^datawalk]
- Neo4j, whose graph model is a useful reference for representing nodes, relationships, and graph data science workflows.[^neo4j]
- Graphistry, which demonstrates analyst-facing graph visualization for large, complex relationship data.[^graphistry]
- i2 Analyst's Notebook, a long-standing reference for visual link analysis and investigative collation.[^i2]

The adaptation for this platform is conservative identity linking: the goal is not maximum match recall; it is safe, reviewable identity boundaries for individual simulation.

## 3. Position in the Platform

```text
Evidence Ingestion
  → Entity Resolution and Identity Boundary
  → Claim/Event/Relation Extraction
  → Temporal Evidence Knowledge Graph
  → Review and Adjudication
  → Agent Construction
```

## 4. Core Responsibilities

- Maintain the Identity Boundary Graph.
- Generate candidate associations between the person and external entities.
- Classify association strength.
- Preserve contradictory and rejected identity evidence.
- Route uncertain associations to review.
- Decide whether a linked entity is usable in extraction, graph storage, agent state, or simulation.
- Prevent weak or disputed identity links from contaminating agent memory.

## 5. Non-Responsibilities

This module does not:

- Crawl public sources.
- Extract behavioral claims.
- Decide simulation outputs.
- Override participant rejection.
- Model non-consented third parties.

## 6. Logical Component Architecture

```text
Entity Resolution and Identity Boundary Module
 ├── Identity Boundary Graph
 ├── Candidate Association Generator
 ├── Association Evidence Scorer
 ├── Conflict and Collision Detector
 ├── Participant Confirmation Interface Contract
 ├── Reviewer Adjudication Interface Contract
 ├── Entity Boundary Record Registry
 ├── Usability Gate for Agent State
 └── Rejected/Disputed Link Archive
```

## 7. Entity Types

```text
person
account
profile
alias
email_or_contact_handle
organization
event
publication
media_item
location
topic
third_party_subject
```

Not every entity is eligible for agent state. For example, a `third_party_subject` should not become a modeled person without consent.

## 8. Association Levels

Recommended association levels:

| Level | Label | Architectural meaning | Agent-state eligibility |
|---|---|---|---|
| A | `participant_confirmed` | The person confirmed the association. | Eligible. |
| B | `self_linked_public_evidence` | The person's confirmed source links to the entity. | Eligible, subject to scope. |
| C | `organization_confirmed` | Trusted organization page confirms the association. | Eligible, subject to scope. |
| D | `multi_source_corroborated` | Several independent public sources align. | Review recommended. |
| E | `weak_candidate` | Name, image, topic, or location similarity only. | Not eligible. |
| F | `rejected_or_disputed` | Rejected by participant or reviewer. | Not eligible. |

## 9. Identity Boundary Record

```json
{
  "boundary_id": "boundary_001",
  "person_id": "person_001",
  "entity_id": "account_123",
  "entity_type": "public_account",
  "association_level": "participant_confirmed",
  "confidence": 1.0,
  "evidence_ids": ["ev_001"],
  "conflicting_evidence_ids": [],
  "review_status": "approved",
  "usable_for_claim_extraction": true,
  "usable_in_agent_state": true,
  "usable_in_simulation": true,
  "consent_scope_id": "scope_001",
  "valid_from": "2025-01-01",
  "valid_until": null
}
```

## 10. Identity Boundary Graph

Nodes:

```text
Person
Account
Profile
Source
Organization
Event
Publication
Alias
ThirdPartySubject
ReviewDecision
```

Edges:

```text
confirmed_controls
possibly_controls
self_links_to
organization_confirms
participated_in
affiliated_with
authored
mentioned_in
rejected_as
disputed_as
supersedes
```

Each edge must carry:

```text
association_level
confidence
evidence_ids
review_status
usable_in_simulation
consent_scope_id
third_party_minimization_status
valid_from
valid_until
```

## 11. Conflict Handling

Common conflicts:

```text
same name, different person
same handle reused by different person
discontinued affiliation
third-party attribution contradicted by participant
old public profile conflicts with current profile
location or organization mismatch
```

Conflict object:

```json
{
  "conflict_id": "conflict_001",
  "person_id": "person_001",
  "candidate_entity_ids": ["account_123", "account_456"],
  "conflict_type": "same_name_collision",
  "supporting_evidence_ids": ["ev_001"],
  "contradicting_evidence_ids": ["ev_002"],
  "resolution_status": "review_required",
  "agent_state_blocked": true
}
```

## 12. Architectural Flow

```text
evidence unit received
  → candidate associations generated
  → association evidence scored
  → conflicts detected
  → association level assigned
  → weak/conflicted links routed to review
  → confirmed links stored in Identity Boundary Graph
  → eligible links passed to extraction and KG modules
```

## 13. Usability Gate

Association records should expose separate eligibility flags:

```text
usable_for_source_discovery
usable_for_claim_extraction
usable_in_temporal_kg
usable_in_agent_state
usable_in_simulation
usable_in_output
```

A source may be useful for discovery while still being prohibited from agent state.

## 14. Integration Contracts

| Module | Contract |
|---|---|
| Evidence Ingestion | Receives evidence units and identity hints. |
| Participant Rights | Sends candidate identity links for confirmation/rejection. |
| Review | Receives weak, conflicting, or high-risk links. |
| Claim Extraction | Uses only eligible identity links for person-specific claims. |
| Temporal KG | Stores identity boundary nodes and edges. |
| Agent Construction | Uses only approved, in-scope identity associations. |
| Privacy/Third-Party | Prevents non-consented third-party profiling. |

## 15. Architectural Invariants

```text
Same name is not identity.
Similarity is not confirmation.
Candidate link is not agent memory.
Participant rejection overrides automated matching.
Disputed links cannot enter simulation.
Third-party subject links cannot create non-consented agents.
```

## 16. MVP Boundary

Minimum viable architecture:

```text
identity boundary graph
candidate association object
association levels
review status
confirmed/rejected/disputed link states
agent-state usability flag
participant confirmation flow
```

Deferred:

```text
large-scale probabilistic entity resolution
cross-tenant identity federation
advanced graph embeddings for candidate generation
biometric or face matching, which should remain excluded by default
```

## 17. References

[^senzing]: Senzing, entity-resolved knowledge graphs: https://senzing.com/knowledge-graph/
[^quantexa]: Quantexa entity resolution software: https://www.quantexa.com/platform/entity-resolution-software/
[^datawalk]: DataWalk entity resolution and investigative graph platform: https://datawalk.com/solutions/entity-resolution/
[^neo4j]: Neo4j documentation: https://neo4j.com/docs/
[^graphistry]: Graphistry visual graph intelligence: https://www.graphistry.com/
[^i2]: i2 Analyst's Notebook: https://i2group.com/solutions/i2-analysts-notebook
