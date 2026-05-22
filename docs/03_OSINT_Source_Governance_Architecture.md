# 03 — OSINT Source Governance Architecture

**Project:** Orb  
**Version:** 0.1  
**Date:** 2026-05-07  

> **Document type:** Architecture manuscript.  
> **Scope boundary:** This document describes module responsibilities, architectural boundaries, canonical objects, data flows, and integration contracts. It intentionally avoids implementation details such as code, deployment topology, specific model prompts, database tuning, vendor procurement, or operational runbooks.  
> **Parent manuscript:** Orb, Version 0.1.  
> **Core principle:** OSINT grounds evidence; the knowledge graph defines the truth boundary; the agent produces scenario-specific hypotheses; the simulation trace remains separate from real-person facts.


## 1. Architectural Purpose

The OSINT Source Governance module defines which public sources may be discovered, captured, normalized, linked, embedded, reviewed, and used for simulation. It converts “publicly accessible” into a governed source policy rather than a blanket permission.

The core architectural question is:

```text
For this person, project, purpose, and source type, is the source allowed,
restricted, review-required, aggregate-only, or excluded?
```

## 2. Reference Patterns and Existing Systems

This module borrows architectural patterns from investigative OSINT and threat-intelligence products, but adapts them to consented individual simulation.

Reference patterns:

| Reference | Useful pattern | Adaptation for this platform |
|---|---|---|
| Maltego | Transform-based discovery and graph-oriented investigation workflows.[^maltego] | Use discovery and link-analysis patterns, but require consent scope and review before agent use. |
| SpiderFoot | Automated OSINT collection across many source modules.[^spiderfoot] | Use modular source discovery ideas, but prevent exhaustive uncontrolled collection. |
| OpenCTI | Knowledge-graph-oriented threat intelligence ingestion and connectors.[^opencti] | Use source connector and graph discipline, but replace threat objects with consented personal evidence objects. |
| MISP | Structured sharing of indicators and threat intelligence objects.[^misp] | Use structured object and event sharing principles, but do not share personal agent evidence without scope. |
| i2 Analyst's Notebook | Visual link analysis and investigative collation.[^i2] | Use analyst review and link-context patterns, but require identity-boundary controls. |

The module should not imitate the “collect everything about a target” posture that many OSINT workflows optimize for. It should implement scoped, reviewable, purpose-bound discovery.

## 3. Position in the Platform

```text
Governance and Consent
  → OSINT Source Governance
  → OSINT Evidence Ingestion and Provenance
  → Entity Resolution and Identity Boundary
  → Claim/Event/Relation Extraction
```

The module sits before evidence capture. It determines what may enter the evidence pipeline at all.

## 4. Core Responsibilities

- Maintain a Source Registry.
- Define source categories and source risk labels.
- Enforce OSINT Scope Addenda.
- Define source discovery boundaries.
- Classify sources as allowed, restricted, review-required, aggregate-only, or excluded.
- Track source terms, license, access method, and capture policy.
- Define embedding and derived-artifact permissions.
- Define third-party handling policy per source type.
- Provide source candidates to participant/reviewer workflows.

## 5. Non-Responsibilities

This module does not:

- Store raw captured evidence.
- Extract claims or entities.
- Confirm identity links.
- Decide final factual truth.
- Build agent state.
- Run simulation.

## 6. Logical Component Architecture

```text
OSINT Source Governance Module
 ├── Source Type Taxonomy
 ├── Source Registry
 ├── Source Policy Evaluator
 ├── Source Discovery Boundary Manager
 ├── Source Risk Labeler
 ├── Source Terms and License Registry
 ├── Source Review Routing Rules
 ├── Third-Party Source Handling Rules
 ├── Embedding Eligibility Rules
 └── Source Candidate Queue
```

## 7. Source Type Taxonomy

Recommended categories:

```text
self_confirmed_public_account
self_authored_public_post
public_professional_profile
public_talk_or_interview
publication_or_report
official_organization_page
public_event_participation
third_party_media_mention
public_forum_reference
aggregated_public_database
public_image_or_video
public_geospatial_reference
```

Default excluded categories:

```text
leaked_data
doxxing_database
private_or_closed_group
login_required_data_without_permission
unverified_gossip
biometric_face_scraping
private_location_pattern
sensitive_inference_source
maliciously_published_private_data
```

## 8. Source Policy States

```text
allowed
allowed_after_participant_confirmation
allowed_after_reviewer_approval
aggregate_only
restricted
excluded
expired
disputed
withdrawn
```

A source may be public and still be `restricted` or `excluded` for this platform.

## 9. Source Registry Object

```json
{
  "source_id": "src_001",
  "source_type": "self_authored_public_article",
  "source_reference": "canonical_reference_or_url_hash",
  "publisher": "publisher_or_platform",
  "access_method": "public_web",
  "collection_allowed": true,
  "review_required": true,
  "allowed_for_individual_simulation": true,
  "allowed_for_aggregate_simulation": true,
  "embedding_allowed": "approved_claims_only",
  "license_status": "unknown_or_recorded",
  "terms_status": "review_required",
  "default_sensitivity": "normal",
  "third_party_handling": "detect_and_generalize",
  "retention_policy_id": "ret_raw_30_days",
  "consent_scope_id": "scope_001"
}
```

## 10. Source Discovery Boundary

Source discovery should be scoped by:

```text
person_id
confirmed aliases
participant-provided accounts
organization-confirmed links
approved topic scope
approved time range
approved source types
excluded terms or contexts
jurisdictional restrictions
```

Source discovery should return candidates, not evidence-ready memory.

Candidate Source fields:

```text
candidate_source_id
person_id
source_type
discovery_reason
identity_association_hint
risk_label
scope_match
requires_confirmation
requires_terms_review
recommended_status
```

## 11. Source Risk Labels

Risk labels guide routing:

```text
normal
sensitive_candidate
third_party_heavy
identity_uncertain
stale
terms_restricted
login_boundary_uncertain
media_context_only
aggregate_only_recommended
excluded_by_scope
```

## 12. Architectural Flow

```text
new project or person scope
  → load allowed source taxonomy
  → apply OSINT Scope Addendum
  → discover source candidates within boundary
  → assign source risk labels
  → route candidates to participant/reviewer confirmation
  → approved candidates move to Evidence Ingestion
  → rejected/excluded candidates are retained only as minimized audit state
```

## 13. Integration Contracts

| Upstream/downstream module | Contract |
|---|---|
| Governance & Consent | Receives approved purposes, source scope, excluded source types. |
| Participant Rights | Sends source candidates for confirmation/rejection/restriction. |
| Evidence Ingestion | Emits only approved or review-required source candidates. |
| Privacy/Sensitivity | Receives source-level sensitivity defaults and third-party policy. |
| Entity Resolution | Provides identity hints but not confirmed links. |
| Temporal KG | Supplies source registry nodes and source-scope edges. |

## 14. Architectural Invariants

```text
Source discovery is not source approval.
Public access is not simulation permission.
Candidate source links are not confirmed identity links.
Restricted sources cannot become agent memory.
Embedding is separately governed from capture.
Third-party-heavy sources require minimization before claim use.
```

## 15. MVP Boundary

Minimum viable source governance:

```text
source taxonomy
source registry
allowed/excluded source rules
candidate source queue
participant source confirmation
source risk label
source-to-consent-scope mapping
```

Deferred:

```text
automated web-scale discovery
advanced source reputation scoring
cross-platform source federation
full license/terms reasoning automation
large-scale monitoring feeds
```

## 16. References

[^maltego]: Maltego OSINT and investigations platform: https://www.maltego.com/
[^spiderfoot]: SpiderFoot open-source OSINT automation tool: https://github.com/smicallef/spiderfoot
[^opencti]: OpenCTI documentation: https://docs.opencti.io/latest/
[^misp]: MISP open-source threat intelligence platform: https://www.misp-project.org/
[^i2]: i2 Analyst's Notebook: https://i2group.com/solutions/i2-analysts-notebook
