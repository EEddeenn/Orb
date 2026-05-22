# 05 — Privacy, Sensitivity, and Third-Party Protection Architecture

**Project:** Orb  
**Version:** 0.1  
**Date:** 2026-05-07  

> **Document type:** Architecture manuscript.  
> **Scope boundary:** This document describes module responsibilities, architectural boundaries, canonical objects, data flows, and integration contracts. It intentionally avoids implementation details such as code, deployment topology, specific model prompts, database tuning, vendor procurement, or operational runbooks.  
> **Parent manuscript:** Orb, Version 0.1.  
> **Core principle:** OSINT grounds evidence; the knowledge graph defines the truth boundary; the agent produces scenario-specific hypotheses; the simulation trace remains separate from real-person facts.


## 1. Architectural Purpose

The Privacy, Sensitivity, and Third-Party Protection module is a cross-cutting risk-control layer. It protects both the consented individual and people who appear in OSINT sources but have not consented to being modeled.

The core architectural question is:

```text
Which personal, sensitive, or third-party information must be removed,
restricted, generalized, reviewed, aggregated, or blocked before it can influence an agent or simulation?
```

## 2. Position in the Platform

```text
OSINT Source Governance
  → Evidence Ingestion
  → Privacy/Sensitivity/Third-Party Protection
  → Entity Resolution
  → Claim Extraction
  → Temporal KG
  → Retrieval
  → Simulation Output
```

This module participates at multiple enforcement points, not only after ingestion.

## 3. Core Responsibilities

- Classify sensitive data candidates.
- Detect third-party subject mentions.
- Generalize, redact, restrict, or block third-party information.
- Define aggregate-only restrictions.
- Prevent non-consented people from becoming modeled agents.
- Prevent sensitive attributes from being inferred, retrieved, or output unless explicitly authorized.
- Maintain restricted evidence queues.
- Define small-cell suppression and named-person output restrictions for network simulations.
- Provide labels and restrictions to the knowledge graph, retrieval layer, and output guardrails.

## 4. Non-Responsibilities

This module does not:

- Decide final consent status.
- Replace legal review.
- Determine whether a source is in OSINT scope.
- Confirm identity links.
- Run simulations.

It provides classification, minimization, and protection architecture.

## 5. Logical Component Architecture

```text
Privacy, Sensitivity, and Third-Party Protection Module
 ├── Sensitivity Taxonomy
 ├── Sensitive Data Classifier
 ├── Third-Party Subject Detector
 ├── Third-Party Context Minimizer
 ├── Redaction and Generalization Policy
 ├── Restricted Evidence Queue
 ├── Aggregate-Only Output Controller
 ├── Small-Cell Suppression Rules
 ├── Sensitive Claim Review Router
 ├── Retrieval Restriction Labels
 └── Output Leakage Guardrails
```

## 6. Sensitivity Taxonomy

Default sensitive categories:

```text
racial_or_ethnic_origin
political_opinion
religious_or_philosophical_belief
trade_union_membership
genetic_data
biometric_identifier
health_data
sex_life_or_sexual_orientation
children_or_minor_status
financial_distress
criminal_allegation
precise_private_location_pattern
family_or_intimate_relationship
vulnerability_indicator
```

Each category should define:

```text
default_action
review_requirement
retrieval_eligibility
output_eligibility
aggregate_eligibility
retention_policy
explicit_authorization_requirement
```

## 7. Third-Party Subject Model

A third-party subject is a person who appears in source material but is not the consented individual being modeled.

Third-party subject states:

```text
detected
pseudonymized
generalized_to_role
redacted
authorized_context_only
restricted
excluded
```

Canonical fields:

```json
{
  "third_party_subject_id": "tps_001",
  "source_evidence_id": "ev_001",
  "relationship_to_consent_person": "colleague_or_unknown",
  "identifiability_level": "direct_name",
  "necessity_for_purpose": "low",
  "minimization_action": "generalize_to_role",
  "usable_in_graph": false,
  "usable_in_agent_state": false,
  "usable_in_output": false
}
```

## 8. Minimization Actions

Possible actions:

```text
retain_as_is
pseudonymize
generalize_to_role
generalize_to_cohort
redact_name_only
redact_full_reference
exclude_claim
aggregate_only
restricted_queue
```

Example transformations:

```text
"Jane Smith, Senior Manager at X" → "a senior colleague"
"three named coworkers objected" → "several colleagues objected"
"specific customer complaint from named customer" → aggregate-only customer concern theme
```

## 9. Sensitive Claim Routing

```text
sensitive candidate detected
  → check explicit authorization
  → check purpose necessity
  → check review requirement
  → classify use eligibility
  → route to restricted queue or delete/exclude
```

Use eligibility values:

```text
not_usable
review_required
aggregate_only
individual_use_explicitly_authorized
simulation_context_only
```

## 10. Restricted Evidence Queue

The queue holds artifacts that cannot continue automatically.

Fields:

```text
queue_item_id
artifact_id
artifact_type
person_id
sensitivity_category
third_party_category
reason_code
current_policy_state
required_review_type
allowed_resolution_states
created_at
expires_at
```

Allowed resolution states:

```text
approve_with_scope
approve_aggregate_only
redact_then_approve
generalize_then_approve
reject
delete
retain_audit_only
```

## 11. Aggregate-Only Controls

Aggregate-only restrictions are necessary when individual-level output would be too sensitive, too uncertain, or too likely to affect a person.

Aggregate-only triggers:

```text
third-party-heavy evidence
sensitive categories
small cohorts
workplace setting
power imbalance
uncertain identity association
media allegation
historical/stale claim
network centrality metric for real persons
```

Aggregate output constraints:

```text
minimum cohort size
small-cell suppression
no named examples
no unique-identifying descriptions
uncertainty bands
purpose reminder
no intervention targeting
```

## 12. Retrieval and Output Labels

Labels emitted to downstream modules:

```text
sensitivity_label
third_party_minimization_status
aggregate_only
no_individual_output
no_agent_state
review_required
expired_sensitive
identity_sensitive
relationship_sensitive
```

These labels must travel with evidence units, claims, agent state entries, evidence packets, and simulation traces.

## 13. Architectural Flows

### 13.1 Sensitive Evidence Flow

```text
evidence unit created
  → preliminary sensitivity scan
  → source and consent policy check
  → sensitive category label
  → restricted queue if needed
  → review decision
  → eligible claim extraction or exclusion
```

### 13.2 Third-Party Flow

```text
evidence unit contains other people
  → detect third-party subjects
  → classify identifiability
  → assess necessity
  → apply minimization action
  → store only permitted context
  → block non-consented agent creation
```

### 13.3 Output Leakage Flow

```text
simulation output candidate
  → detect sensitive inference
  → detect third-party leakage
  → detect named-person targeting
  → redact, aggregate, route to review, or block
```

## 14. Integration Contracts

| Module | Contract |
|---|---|
| Evidence Ingestion | Receives preliminary sensitive/third-party labels and minimization outcomes. |
| Entity Resolution | Blocks non-consented third-party profiling. |
| Claim Extraction | Filters sensitive and third-party claims before agent state eligibility. |
| Temporal KG | Stores restrictions and minimization status on nodes/edges. |
| Review | Routes sensitive claims and third-party-heavy evidence. |
| Retrieval | Enforces sensitivity and third-party labels during evidence packet assembly. |
| Simulation Output | Blocks sensitive inference, manipulation, and named third-party leakage. |
| Evaluation | Tracks leakage, false negatives, and policy regression failures. |

## 15. Architectural Invariants

```text
Mentioned third party ≠ modeled participant.
Relationship evidence ≠ permission to profile the relationship target.
Sensitive candidate ≠ usable agent memory.
Aggregate-only claims cannot enter individual outputs.
Redacted evidence cannot be re-expanded at retrieval time.
Simulation cannot infer excluded sensitive attributes.
```

## 16. MVP Boundary

Minimum viable architecture:

```text
sensitivity taxonomy
third-party subject detector label
restricted evidence queue
redaction/generalization status
aggregate-only flag
retrieval labels
output leakage guardrail labels
```

Deferred:

```text
advanced de-identification risk scoring
formal k-anonymity/differential output controls
jurisdiction-specific sensitive taxonomies
multi-party consent negotiation
```

## 17. References

- GDPR Article 9, special categories of personal data: https://gdpr-info.eu/art-9-gdpr/
- OAIC guidance on privacy and generative AI: https://www.oaic.gov.au/privacy/privacy-guidance-for-organisations-and-government-agencies/guidance-on-privacy-and-developing-and-training-generative-ai-models
- NIST AI RMF: https://airc.nist.gov/airmf-resources/airmf/
