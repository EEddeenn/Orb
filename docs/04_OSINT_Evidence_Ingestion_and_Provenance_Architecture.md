# 04 — OSINT Evidence Ingestion and Provenance Architecture

**Project:** Orb  
**Version:** 0.1  
**Date:** 2026-05-07  

> **Document type:** Architecture manuscript.  
> **Scope boundary:** This document describes module responsibilities, architectural boundaries, canonical objects, data flows, and integration contracts. It intentionally avoids implementation details such as code, deployment topology, specific model prompts, database tuning, vendor procurement, or operational runbooks.  
> **Parent manuscript:** Orb, Version 0.1.  
> **Core principle:** OSINT grounds evidence; the knowledge graph defines the truth boundary; the agent produces scenario-specific hypotheses; the simulation trace remains separate from real-person facts.


## 1. Architectural Purpose

The OSINT Evidence Ingestion and Provenance module converts approved public sources into auditable evidence units. It is the boundary between source discovery and structured intelligence.

The core architectural question is:

```text
How does a public source become a traceable, scoped, reviewable evidence unit
without becoming uncontrolled agent memory?
```

## 2. Reference Patterns and Existing Systems

This module references:

- SpiderFoot for modular OSINT source collection and analysis patterns.[^spiderfoot]
- Maltego for transform-style discovery and graph-oriented evidence workflows.[^maltego]
- OpenCTI for connector-driven intelligence ingestion into a knowledge graph.[^opencti]
- MISP for structured threat-intelligence object discipline and event-style packaging.[^misp]

The adaptation is strict: these systems inspire ingestion and object modeling, but this platform must add per-person consent scope, participant review, third-party minimization, and deletion propagation.

## 3. Position in the Platform

```text
OSINT Source Governance
  → Evidence Ingestion and Provenance
  → Privacy/Sensitivity and Third-Party Filtering
  → Entity Resolution
  → Claim/Event/Relation Extraction
```

The module does not decide that a claim is true. It records source material and provenance so later modules can extract and adjudicate claims.

## 4. Core Responsibilities

- Accept approved or review-required source candidates.
- Capture source material according to source policy.
- Record provenance metadata.
- Normalize evidence into canonical evidence units.
- Hash and version content.
- Label sensitivity and third-party indicators at a preliminary level.
- Bind evidence to consent scope and retention policy.
- Create derived artifact policy for claims, embeddings, packets, traces, and exports.
- Emit evidence units to entity resolution and extraction modules.

## 5. Non-Responsibilities

This module does not:

- Perform unrestricted source discovery.
- Confirm identity links.
- Approve claims for agent state.
- Generate simulation outputs.
- Decide participant-facing truth.

## 6. Logical Component Architecture

```text
OSINT Evidence Ingestion and Provenance Module
 ├── Source Candidate Intake
 ├── Scope and Capture Gate
 ├── Capture Record Builder
 ├── Evidence Normalizer
 ├── Provenance Metadata Builder
 ├── Content Hash and Version Registry
 ├── Preliminary Sensitivity Scanner
 ├── Third-Party Presence Detector
 ├── Evidence Quality Pre-Scorer
 ├── Retention and Derived Artifact Binder
 └── Evidence Unit Store Interface
```

## 7. Evidence Unit Lifecycle

```text
candidate_source
  → capture_authorized
  → captured
  → normalized
  → provenance_registered
  → sensitivity_scanned
  → third_party_scanned
  → identity_link_pending
  → extraction_ready
  → review_pending
  → approved / restricted / rejected / expired / deleted
```

## 8. Evidence Unit Object

```json
{
  "evidence_id": "ev_001",
  "source_id": "src_001",
  "source_type": "self_authored_public_article",
  "person_id": "person_001",
  "capture_record_id": "cap_001",
  "captured_at": "2026-05-07T00:00:00Z",
  "published_at": "2025-04-12T00:00:00Z",
  "observed_at": "2025-04-12T00:00:00Z",
  "content_hash": "sha256:...",
  "content_version": "v1",
  "consent_scope_id": "scope_001",
  "source_policy_state": "allowed_after_review",
  "preliminary_sensitivity_label": "normal",
  "third_party_subjects_detected": 2,
  "third_party_minimization_status": "required",
  "identity_confidence_hint": 0.82,
  "review_status": "pending",
  "retention_policy_id": "ret_raw_30_days",
  "derived_artifact_policy_id": "dap_001"
}
```

## 9. Capture Record Object

```json
{
  "capture_record_id": "cap_001",
  "source_id": "src_001",
  "captured_by": "system_or_reviewer_id",
  "capture_method": "approved_source_fetch",
  "access_context": "public_web",
  "terms_status": "recorded_or_review_required",
  "license_status": "recorded_or_unknown",
  "capture_timestamp": "2026-05-07T00:00:00Z",
  "raw_artifact_pointer": "artifact_reference",
  "normalization_profile": "text_document_v1",
  "integrity_status": "hash_recorded"
}
```

## 10. Provenance Model

Every evidence unit should answer:

```text
Where did this come from?
Who or what captured it?
When was it published, observed, captured, and verified?
Which consent scope authorized it?
Which source policy allowed or restricted it?
Which derived artifacts came from it?
Can it be deleted, retracted, or used only in audit?
```

Provenance edges:

```text
EvidenceUnit --captured_from--> Source
EvidenceUnit --within_scope_of--> ConsentScope
EvidenceUnit --governed_by--> SourcePolicy
EvidenceUnit --has_capture_record--> CaptureRecord
Claim --derived_from--> EvidenceUnit
EvidenceUnit --has_retention_policy--> RetentionPolicy
EvidenceUnit --has_derived_policy--> DerivedArtifactPolicy
```

## 11. Sensitivity and Third-Party Pre-Filtering

The ingestion module performs preliminary labels, not final adjudication.

Labels:

```text
normal
sensitive_candidate
third_party_detected
third_party_heavy
private_context_candidate
minor_candidate
criminal_allegation_candidate
political_or_religious_candidate
health_candidate
precise_location_candidate
```

Routing:

```text
normal → extraction ready
sensitive_candidate → restricted queue
third_party_heavy → minimization workflow
identity_uncertain → identity boundary review
terms_restricted → governance review
```

## 12. Evidence Quality Pre-Scoring

Quality dimensions:

```text
source_reliability
source_directness
self_authored_status
publication_recency
identity_confidence_hint
corroboration_potential
contradiction_risk
sensitivity_risk
purpose_relevance
review_requirement
```

The pre-score is advisory. It should not approve claims or agent state.

## 13. Derived Artifact Policy

Every evidence unit must attach a derived artifact policy.

Artifact classes:

```text
claim_candidates
entity_candidates
summaries
embeddings
vector_index_entries
agent_state_entries
evidence_packets
simulation_context_logs
simulation_outputs
evaluation_reports
exports
```

Possible policies:

```text
raw_only_no_derivatives
claims_only_no_embeddings
approved_claims_only_embeddings
aggregate_only_derivatives
block_and_purge_on_withdrawal
retain_minimized_audit_only
```

## 14. Architectural Flow

```text
source candidate received
  → source policy evaluated
  → capture authorized or blocked
  → capture record created
  → evidence unit normalized
  → provenance attached
  → preliminary sensitivity and third-party labels assigned
  → retention and derived artifact policies bound
  → evidence emitted to identity and extraction modules
```

## 15. Integration Contracts

| Module | Contract |
|---|---|
| Source Governance | Receives allowed source candidates and source policy. |
| Governance & Consent | Requests capture authorization and retention policy. |
| Privacy/Sensitivity | Sends preliminary sensitive and third-party labels. |
| Entity Resolution | Sends evidence units with identity confidence hints. |
| Claim Extraction | Sends extraction-ready evidence units. |
| Temporal KG | Creates evidence/source/provenance nodes and edges. |
| Review | Sends evidence requiring participant or reviewer confirmation. |

## 16. Architectural Invariants

```text
No evidence unit without provenance.
No evidence unit without consent scope.
No raw OSINT enters agent memory directly.
No embedding without derived artifact policy.
No third-party-heavy evidence bypasses minimization.
No withdrawn evidence remains retrieval-eligible.
```

## 17. MVP Boundary

Minimum viable ingestion architecture:

```text
source candidate intake
evidence unit object
capture record object
provenance fields
content hash
preliminary sensitivity label
third-party presence flag
retention policy binding
derived artifact policy binding
```

Deferred:

```text
large-scale continuous monitoring
multi-format media analysis
complex license reasoning
advanced provenance chain verification
federated evidence sharing
```

## 18. References

[^spiderfoot]: SpiderFoot open-source OSINT automation tool: https://github.com/smicallef/spiderfoot
[^maltego]: Maltego OSINT and investigations platform: https://www.maltego.com/
[^opencti]: OpenCTI documentation: https://docs.opencti.io/latest/
[^misp]: MISP open-source threat intelligence platform: https://www.misp-project.org/
