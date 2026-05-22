# Module Architecture Manuscripts

**Project:** Orb  
**Version:** 0.1  
**Date:** 2026-05-07  
**Parent manuscript:** [ARCHITECTURE.md](ARCHITECTURE.md)  

> **Document type:** Architecture manuscript.  
> **Scope boundary:** This document describes module responsibilities, architectural boundaries, canonical objects, data flows, and integration contracts. It intentionally avoids implementation details such as code, deployment topology, specific model prompts, database tuning, vendor procurement, or operational runbooks.  
> **Parent manuscript:** Orb, Version 0.1.  
> **Core principle:** OSINT grounds evidence; the knowledge graph defines the truth boundary; the agent produces scenario-specific hypotheses; the simulation trace remains separate from real-person facts.


## Module Set

This package decomposes the platform into twelve architecture modules:

| # | Module | Primary plane | Primary question |
|---:|---|---|---|
| 01 | Governance and Consent | Governance & Consent | What is the system allowed to do? |
| 02 | Participant and Subject Rights | Governance & Consent | How do real individuals inspect, control, correct, restrict, or withdraw their agents? |
| 03 | OSINT Source Governance | OSINT Evidence | Which public sources may be discovered, captured, and used? |
| 04 | OSINT Evidence Ingestion and Provenance | OSINT Evidence | How does OSINT become auditable evidence rather than uncontrolled memory? |
| 05 | Privacy, Sensitivity, and Third-Party Protection | Cross-cutting | How are sensitive data and non-consented people protected? |
| 06 | Entity Resolution and Identity Boundary | Entity & Knowledge Graph | Which accounts, sources, events, and relations are actually associated with the consented person? |
| 07 | Claim, Event, and Relation Extraction | Entity & Knowledge Graph | How does evidence become structured, reviewable knowledge? |
| 08 | Temporal Evidence Knowledge Graph | Entity & Knowledge Graph | What is the truth boundary, and how does it preserve time, provenance, consent, and confidence? |
| 09 | Review and Adjudication | Governance / Graph / Agent | Which candidate evidence and claims become agent-usable? |
| 10 | Individual Agent Construction | Individual Agent | How are approved claims converted into a bounded, non-impersonating agent? |
| 11 | Consent-Aware Retrieval and Evidence Packet | Individual Agent / Runtime | What can an agent know in a specific scenario and purpose? |
| 12 | Simulation, Output, Trace, and Evaluation | Simulation & Evaluation | How do individual agents run in scenarios, produce outputs, and get evaluated? |

## Recommended Reading Order

```text
01 Governance and Consent
02 Participant and Subject Rights
03 OSINT Source Governance
04 OSINT Evidence Ingestion and Provenance
05 Privacy, Sensitivity, and Third-Party Protection
06 Entity Resolution and Identity Boundary
07 Claim, Event, and Relation Extraction
08 Temporal Evidence Knowledge Graph
09 Review and Adjudication
10 Individual Agent Construction
11 Consent-Aware Retrieval and Evidence Packet
12 Simulation, Output, Trace, and Evaluation
```

## End-to-End Architecture Backbone

```text
Consent and purpose framing
  → Source governance
  → OSINT evidence capture
  → Sensitivity and third-party filtering
  → Entity boundary resolution
  → Claim/event/relation extraction
  → Temporal evidence graph
  → Participant/reviewer adjudication
  → Agent card and agent state
  → Consent-aware retrieval
  → Scenario and simulation runtime
  → Output guardrails
  → Simulation trace
  → Evaluation and calibration
```

## System Boundary Rules

```text
OSINT Source ≠ Evidence
Evidence ≠ Claim
Claim ≠ Agent State
Agent State ≠ Real Person
Simulation Output ≠ Fact
Simulation Trace ≠ Personal Profile
```

## Reference Families

The OSINT modules reference patterns from Maltego, SpiderFoot, OpenCTI, MISP, i2 Analyst's Notebook, and investigative graph products.  
The graph/entity modules reference patterns from Neo4j, Senzing, Quantexa, DataWalk, and Graphistry.  
The agent modules reference patterns from Generative Agents research, LLM agents grounded in self-reports, LangGraph, AutoGen, CrewAI, and Concordia.  
The simulation modules reference patterns from Mesa, NetLogo, AnyLogic, GAMA, Concordia, AgentSociety, and OASIS.



---


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



---


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



---


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



---


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



---


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



---


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



---


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



---


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



---


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



---


# 10 — Individual Agent Construction Architecture

**Project:** Orb  
**Version:** 0.1  
**Date:** 2026-05-07  

> **Document type:** Architecture manuscript.  
> **Scope boundary:** This document describes module responsibilities, architectural boundaries, canonical objects, data flows, and integration contracts. It intentionally avoids implementation details such as code, deployment topology, specific model prompts, database tuning, vendor procurement, or operational runbooks.  
> **Parent manuscript:** Orb, Version 0.1.  
> **Core principle:** OSINT grounds evidence; the knowledge graph defines the truth boundary; the agent produces scenario-specific hypotheses; the simulation trace remains separate from real-person facts.


## 1. Architectural Purpose

The Individual Agent Construction module turns approved evidence and claims into a bounded, evidence-constrained individual agent. It is not a profile generator and not an impersonation engine.

The core architectural question is:

```text
Given approved claims, self-reports, consent scope, uncertainty, and restrictions,
how should the platform construct a simulation actor that can generate scenario-specific hypotheses without claiming to be the person?
```

## 2. Reference Research, Frameworks, and Products

This module references several agent architectures and frameworks:

- Generative Agents introduced a memory-reflection-planning architecture for believable behavior simulation.[^gen-agents]
- LLM agents grounded in self-reports demonstrated person-specific simulation from interviews and surveys.[^self-reports]
- LangGraph provides stateful agent orchestration patterns, including memory and human-in-the-loop controls.[^langgraph]
- AutoGen provides multi-agent conversation and coordination patterns.[^autogen]
- CrewAI provides production-oriented multi-agent orchestration concepts with agents, crews, flows, memory, knowledge, guardrails, and observability.[^crewai]
- Concordia provides a generative social simulation library with environment mediation through a Game Master pattern.[^concordia]

The platform adapts these patterns by adding consent-aware memory, evidence packets, non-impersonation, uncertainty, and strict separation between simulation memory and real-person facts.

## 3. Position in the Platform

```text
Approved Claim Set
  → Agent State Builder
  → Agent Card
  → Agent State Graph
  → Memory Layers
  → Consent-Aware Retrieval
  → Simulation Runtime
```

## 4. Core Responsibilities

- Build an Agent Card from approved claims and authorized self-reports.
- Separate identity boundary, authorization boundary, evidence-backed state, uncertainty, memory policy, simulation policy, and output policy.
- Generate agent-usable state entries with evidence references.
- Maintain an uncertainty map.
- Maintain excluded topics and do-not-use knowledge.
- Define memory layers.
- Provide simulation runtime with a governed agent interface.
- Prevent first-person impersonation and unauthorized sensitive inference.

## 5. Non-Responsibilities

This module does not:

- Crawl or capture OSINT.
- Approve claims.
- Build source evidence.
- Run environment dynamics.
- Produce output without runtime policy checks.
- Update evidence graph based on simulation outputs automatically.

## 6. Logical Component Architecture

```text
Individual Agent Construction Module
 ├── Approved Claim Set Intake
 ├── Agent Card Builder
 ├── Agent State Graph Builder
 ├── Evidence-Backed State Synthesizer
 ├── Behavioral Hypothesis Layer
 ├── Uncertainty Map Builder
 ├── Exclusion and Do-Not-Use Manager
 ├── Memory Policy Builder
 ├── Simulation Policy Builder
 ├── Output Policy Builder
 ├── Evaluation History Binder
 └── Agent Revocation Handler
```

## 7. Agent Card Structure

```text
Agent Card
 ├── Identity Boundary
 ├── Authorization Boundary
 ├── Evidence-Backed State
 ├── Behavioral Hypothesis Layer
 ├── Uncertainty Map
 ├── Memory Policy
 ├── Simulation Policy
 ├── Output Policy
 ├── Evaluation History
 └── Revocation Status
```

## 8. Agent Card Object

```json
{
  "agent_id": "agent_001",
  "person_id": "person_001",
  "agent_card_version": "1.0",
  "identity_boundary": {
    "verified_person": true,
    "confirmed_accounts": ["account_123"],
    "excluded_accounts": [],
    "disputed_links": []
  },
  "authorization_boundary": {
    "approved_purposes": ["research_simulation"],
    "prohibited_purposes": ["employment_decision", "impersonation"],
    "allowed_source_types": ["self_authored_public_article"],
    "excluded_source_types": ["leaked_data", "private_social_content"]
  },
  "evidence_backed_state": {
    "public_roles": ["operations advisor"],
    "known_topics": ["supply chain resilience", "implementation risk"],
    "communication_style_indicators": ["concise", "evidence-oriented"],
    "decision_constraint_indicators": ["cost", "coordination burden"]
  },
  "uncertainty_map": {
    "current_view_on_specific_policy": "unknown",
    "private_constraints": "not_available",
    "recent_direct_statement": "not_available"
  },
  "simulation_policy": {
    "must_disclose_simulated": true,
    "must_include_uncertainty": true,
    "must_cite_evidence": true,
    "no_first_person_impersonation": true,
    "no_sensitive_inference_without_consent": true,
    "no_high_impact_decision_use": true
  },
  "revocation_status": "active"
}
```

## 9. Agent State Layers

```text
Layer 1: Confirmed Facts
- reviewed and approved identity, role, affiliation, event, and source facts.

Layer 2: Self-Reported Preferences
- direct participant statements, survey answers, or interview-derived preferences.

Layer 3: Publicly Evidenced Positions
- self-authored public statements and reviewed public communication.

Layer 4: Contextual Background
- industry, organization, event, or role context.

Layer 5: Weak Behavioral Hypotheses
- reviewed but uncertain signals, always with confidence and uncertainty.

Layer 6: Excluded Knowledge
- topics, sources, claims, and data categories the agent must not use.
```

## 10. Agent State Entry

```json
{
  "state_entry_id": "ase_001",
  "agent_id": "agent_001",
  "state_type": "decision_constraint_indicator",
  "value": "implementation_cost",
  "evidence_claim_ids": ["claim_001", "claim_002"],
  "confidence": 0.68,
  "uncertainty_note": "Derived from public professional writing and participant-confirmed context.",
  "sensitivity_label": "normal",
  "review_status": "approved",
  "usable_for_purposes": ["research_simulation"],
  "not_usable_for_purposes": ["employment_decision"],
  "valid_until": "2027-05-07"
}
```

## 11. Memory Architecture

```text
Evidence Memory
- Long-lived, approved, retractable, evidence-backed.

Scenario Memory
- Information observed within a scenario.

Simulation Memory
- Temporary runtime state for a specific run.

Reflection Memory
- Runtime summaries or inferred reasoning traces.
```

Critical rule:

```text
Scenario, simulation, and reflection memory cannot automatically update Evidence Memory.
```

## 12. Non-Impersonation Architecture

The agent should be framed as:

```text
an evidence-constrained simulation model for a consented individual
```

It should never be framed as:

```text
the person themselves
an authorized spokesperson
an exact digital twin
an oracle of real intent
```

Output posture:

```text
third-person or clearly simulated framing
confidence and uncertainty
source references
not representative of actual intent
not for high-impact decisions
```

## 13. Uncertainty Map

Uncertainty categories:

```text
insufficient_evidence
stale_evidence
third_party_attribution_only
identity_association_uncertain
specific_topic_unknown
private_constraint_unknown
scenario_assumption_dependent
sensitive_information_excluded
```

Uncertainty affects retrieval, generation, output confidence, and review routing.

## 14. Architectural Flow

```text
approved claim set received
  → apply consent and purpose filters
  → build identity boundary section
  → build evidence-backed state entries
  → build weak behavioral hypothesis layer
  → attach uncertainty map
  → attach exclusion rules
  → attach memory/simulation/output policies
  → publish Agent Card
  → expose agent to retrieval and simulation runtime
```

## 15. Integration Contracts

| Module | Contract |
|---|---|
| Review | Receives approved claim set and restrictions. |
| Temporal KG | Queries claim evidence, validity, and review states. |
| Governance | Receives purpose, output, and revocation policies. |
| Retrieval | Exposes agent state and allowed evidence references. |
| Simulation | Provides bounded agent interface and memory rules. |
| Evaluation | Receives fidelity and safety metrics; updates uncertainty, not facts. |
| Participant Rights | Provides Agent Card preview and export. |

## 16. Architectural Invariants

```text
Agent state must be evidence-backed or explicitly marked as uncertainty.
Agent cannot use excluded knowledge.
Agent cannot claim to be the real person.
Agent cannot represent real-world intent or commitment.
Simulation memory does not become evidence memory.
Revoked agent cards cannot be retrieved or simulated.
```

## 17. MVP Boundary

Minimum viable architecture:

```text
agent card object
agent state entry object
identity boundary section
authorization boundary section
evidence-backed state
uncertainty map
memory policy
non-impersonation simulation policy
revocation status
```

Deferred:

```text
long-running autonomous agent workflows
complex tool-using agents
rich affective/cognitive modeling
continuous self-updating agents
automatic personality reconstruction
```

## 18. References

[^gen-agents]: Park et al., “Generative Agents: Interactive Simulacra of Human Behavior,” arXiv, 2023: https://arxiv.org/abs/2304.03442
[^self-reports]: Park et al., “LLM Agents Grounded in Self-Reports Enable General-Purpose Simulation of Individuals,” arXiv, 2024: https://arxiv.org/abs/2411.10109
[^langgraph]: LangGraph overview: https://docs.langchain.com/oss/python/langgraph/overview
[^autogen]: Microsoft AutoGen multi-agent concepts: https://microsoft.github.io/autogen/stable/user-guide/core-user-guide/core-concepts/agent-and-multi-agent-application.html
[^crewai]: CrewAI documentation: https://docs.crewai.com/
[^concordia]: Google DeepMind Concordia: https://github.com/google-deepmind/concordia



---


# 11 — Consent-Aware Retrieval and Evidence Packet Architecture

**Project:** Orb  
**Version:** 0.1  
**Date:** 2026-05-07  

> **Document type:** Architecture manuscript.  
> **Scope boundary:** This document describes module responsibilities, architectural boundaries, canonical objects, data flows, and integration contracts. It intentionally avoids implementation details such as code, deployment topology, specific model prompts, database tuning, vendor procurement, or operational runbooks.  
> **Parent manuscript:** Orb, Version 0.1.  
> **Core principle:** OSINT grounds evidence; the knowledge graph defines the truth boundary; the agent produces scenario-specific hypotheses; the simulation trace remains separate from real-person facts.


## 1. Architectural Purpose

The Consent-Aware Retrieval and Evidence Packet module controls what an agent can know during a specific scenario. It is the runtime gate between the Temporal Evidence Knowledge Graph, the Agent Card, and the LLM/ABM simulation engine.

The core architectural question is:

```text
For this agent, scenario, purpose, output type, and current consent state,
which approved claims may be retrieved, summarized, redacted, or excluded?
```

## 2. Reference Patterns and Existing Systems

This module references retrieval and agent orchestration patterns from:

- LangGraph's stateful agent architecture and human-in-the-loop control patterns.[^langgraph]
- CrewAI's knowledge and memory patterns for grounding agents in external information.[^crewai-knowledge]
- AutoGen's multi-agent messaging and human/tool integration concepts.[^autogen]
- Generative Agents' memory retrieval and reflection architecture.[^gen-agents]

The adaptation is that retrieval is not just relevance-based. It is consent-aware, purpose-aware, sensitivity-aware, third-party-aware, and simulation-specific.

## 3. Position in the Platform

```text
Agent Card
Temporal Evidence KG
Governance and Consent
Scenario Spec
       ↓
Consent-Aware Retrieval and Evidence Packet
       ↓
LLM Agent Runtime
Simulation Trace
```

## 4. Core Responsibilities

- Receive scenario-specific evidence requests.
- Query Governance for purpose, consent, and artifact eligibility.
- Query the Temporal KG for approved claims and state entries.
- Apply sensitivity and third-party restrictions.
- Apply recency, confidence, and contradiction rules.
- Assemble bounded evidence packets.
- Include excluded-claim summaries for transparency without exposing restricted content.
- Prevent raw OSINT or unapproved evidence from reaching the agent.
- Write retrieval decisions to the simulation trace.

## 5. Non-Responsibilities

This module does not:

- Approve claims.
- Create new facts.
- Run simulations.
- Generate final user-facing outputs.
- Fine-tune models on personal data.

## 6. Logical Component Architecture

```text
Consent-Aware Retrieval and Evidence Packet Module
 ├── Runtime Retrieval Request Intake
 ├── Purpose and Consent Gate
 ├── Agent Card Scope Reader
 ├── Approved Claim Query Planner
 ├── Sensitivity and Third-Party Filter
 ├── Recency and Confidence Weighting Layer
 ├── Conflict and Contradiction Assembler
 ├── Evidence Packet Builder
 ├── Excluded Claim Summary Builder
 ├── Source Integrity and Prompt-Injection Labeler
 ├── Usage Restriction Binder
 └── Retrieval Audit Emitter
```

## 7. Retrieval Request Object

```json
{
  "retrieval_request_id": "rr_001",
  "scenario_id": "scenario_001",
  "run_id": "run_001",
  "agent_id": "agent_001",
  "person_id": "person_001",
  "purpose_id": "research_simulation",
  "query_intent": "simulate_initial_reaction_to_policy",
  "requested_output_type": "probabilistic_scenario_response",
  "as_of_time": "2026-05-07T00:00:00Z",
  "scenario_phase": "t0_policy_announcement"
}
```

## 8. Retrieval Filter Stack

```text
1. Consent status filter
2. Purpose filter
3. Source scope filter
4. Review status filter
5. Sensitivity filter
6. Third-party minimization filter
7. Output eligibility filter
8. Time validity filter
9. Confidence threshold filter
10. Contradiction and staleness handling
11. Scenario relevance selection
12. Usage restriction binding
```

## 9. Evidence Packet Object

```json
{
  "packet_id": "packet_001",
  "retrieval_request_id": "rr_001",
  "scenario_id": "scenario_001",
  "run_id": "run_001",
  "agent_id": "agent_001",
  "purpose_id": "research_simulation",
  "allowed_claims": [
    {
      "claim_id": "claim_001",
      "summary": "The person has publicly written about supply chain resilience.",
      "claim_type": "self_authored_statement",
      "confidence": 0.91,
      "recency_weight": 0.78,
      "evidence_refs": ["ev_001"],
      "validity": "current"
    }
  ],
  "allowed_agent_state_entries": ["ase_001"],
  "excluded_claim_summary": [
    {
      "reason": "outside_consent_scope",
      "category": "private_social_content"
    },
    {
      "reason": "sensitive_implication",
      "category": "political_opinion"
    }
  ],
  "third_party_redactions": [
    {
      "category": "named_colleague",
      "action": "generalized_to_role_context"
    }
  ],
  "uncertainty_notes": [
    "No recent direct public statement about this exact policy."
  ],
  "usage_restrictions": [
    "simulation_only",
    "not_representative_of_actual_intent",
    "not_for_high_impact_decision"
  ],
  "policy_decision_ids": ["decision_001"]
}
```

## 10. Excluded Claim Summary

Excluded content should be disclosed at category level when useful for transparency, without leaking restricted facts.

Examples:

```text
outside consent scope
not reviewed
identity association uncertain
sensitive category excluded
third-party context redacted
stale or expired
aggregate-only not permitted for individual output
withdrawn source
```

## 11. Conflict Handling in Retrieval

If the graph contains contradictory approved claims, the evidence packet should not silently pick one.

Packet conflict entry:

```json
{
  "conflict_id": "conflict_001",
  "summary": "Approved sources differ on whether the affiliation is current.",
  "claim_ids": ["claim_001", "claim_009"],
  "recommended_agent_posture": "treat_as_uncertain",
  "output_requirement": "include_uncertainty"
}
```

## 12. Source Integrity and Prompt-Injection Labels

OSINT content may include instructions or manipulative text. Retrieval should label source integrity risk before passing summaries to the agent.

Labels:

```text
source_clean
source_contains_instructional_text
source_contains_prompt_injection_candidate
source_untrusted_third_party
source_terms_restricted
source_poisoning_suspected
```

Architecture rule:

```text
The evidence packet should provide factual summaries and references, not raw source text that can instruct the agent outside system policy.
```

## 13. Architectural Flow

```text
simulation runtime requests context
  → retrieval request object created
  → governance policy evaluated
  → agent card scope read
  → approved claim set queried
  → filters applied
  → recency/confidence/conflict annotations added
  → evidence packet assembled
  → packet policy checked
  → packet emitted to agent runtime
  → retrieval audit written to trace
```

## 14. Integration Contracts

| Module | Contract |
|---|---|
| Governance | Policy decisions for retrieval and output eligibility. |
| Temporal KG | Approved claims, validity, sensitivity, review status. |
| Agent Construction | Agent state entries, exclusion rules, memory policy. |
| Privacy/Sensitivity | Retrieval labels and third-party minimization. |
| Simulation Runtime | Evidence packet per agent per timestep/scenario phase. |
| Output Guardrails | Usage restrictions, uncertainty notes, evidence refs. |
| Simulation Trace | Records packet, exclusions, and policy decisions. |

## 15. Architectural Invariants

```text
Agents receive evidence packets, not raw unrestricted OSINT.
Relevance never overrides consent.
High confidence never overrides source scope.
Approved aggregate-only claims cannot enter individual evidence packets.
Excluded content can be summarized by category but not revealed.
Every evidence packet is traceable to policy decisions.
```

## 16. MVP Boundary

Minimum viable architecture:

```text
retrieval request object
evidence packet object
consent and purpose filters
review-status filter
sensitivity and third-party filters
excluded claim summary
usage restriction list
policy decision logging
```

Deferred:

```text
advanced semantic routing
multi-hop graph retrieval
dynamic retrieval over long simulations
automated source-poisoning scoring
multi-agent shared-memory negotiation
```

## 17. References

[^langgraph]: LangGraph overview: https://docs.langchain.com/oss/python/langgraph/overview
[^crewai-knowledge]: CrewAI Knowledge documentation: https://docs.crewai.com/en/concepts/knowledge
[^autogen]: Microsoft AutoGen multi-agent concepts: https://microsoft.github.io/autogen/stable/user-guide/core-user-guide/core-concepts/agent-and-multi-agent-application.html
[^gen-agents]: Park et al., “Generative Agents: Interactive Simulacra of Human Behavior,” arXiv, 2023: https://arxiv.org/abs/2304.03442



---


# 12 — Simulation, Output, Trace, and Evaluation Architecture

**Project:** Orb  
**Version:** 0.1  
**Date:** 2026-05-07  

> **Document type:** Architecture manuscript.  
> **Scope boundary:** This document describes module responsibilities, architectural boundaries, canonical objects, data flows, and integration contracts. It intentionally avoids implementation details such as code, deployment topology, specific model prompts, database tuning, vendor procurement, or operational runbooks.  
> **Parent manuscript:** Orb, Version 0.1.  
> **Core principle:** OSINT grounds evidence; the knowledge graph defines the truth boundary; the agent produces scenario-specific hypotheses; the simulation trace remains separate from real-person facts.


## 1. Architectural Purpose

The Simulation, Output, Trace, and Evaluation module runs individual, small-group, and network simulations using evidence-constrained individual agents inside governed environments. It produces probabilistic, uncertainty-qualified outputs and records auditable simulation traces.

The core architectural question is:

```text
How do bounded individual agents interact with scenario environments over time,
and how are their outputs explained, constrained, traced, evaluated, and calibrated?
```

## 2. Reference Research, Frameworks, and Products

This module references both traditional agent-based modeling and LLM-based social simulation:

| Reference | Useful pattern | Adaptation |
|---|---|---|
| Mesa | Python ABM framework with agents, spaces, schedulers, visualization, and data analysis patterns.[^mesa] | Reference for ABM-style environment mechanics and repeatable runs. |
| NetLogo | Established agent-based modeling environment for social and natural systems.[^netlogo] | Reference for accessible ABM concepts and emergence-focused modeling. |
| AnyLogic | Commercial multi-method simulation combining agent-based, discrete-event, and system dynamics modeling.[^anylogic] | Reference for hybrid simulation architecture and business scenario modeling. |
| GAMA | Spatially explicit agent-based simulation platform.[^gama] | Reference for spatial/social environment modeling. |
| Concordia | Generative social simulation library with a Game Master pattern.[^concordia] | Reference for environment-mediated natural-language agent actions. |
| AgentSociety | Large-scale LLM-driven social simulation with realistic social environments.[^agentsociety] | Reference for society-scale generative simulation. |
| OASIS | Scalable LLM/ABM-style social media simulator supporting up to one million users.[^oasis] | Reference for information diffusion, action spaces, recommendation systems, and large-scale social media simulation. |
| Generative Agents | Memory-reflection-planning architecture for believable behavior.[^gen-agents] | Reference for individual-level agent cognition, adapted with consent and evidence controls. |

## 3. Position in the Platform

```text
Scenario Builder
Agent Card
Consent-Aware Evidence Packet
Environment Model
       ↓
LLM + ABM Simulation Runtime
       ↓
Output Guardrails
       ↓
Simulation Trace Graph
       ↓
Evaluation and Calibration
```

## 4. Core Responsibilities

- Define scenario specifications.
- Define environment state, rules, observations, actions, and interventions.
- Run individual, group, and network simulations.
- Combine LLM agent reasoning with ABM-style state progression.
- Enforce output guardrails.
- Generate simulation traces.
- Produce evidence-grounded, uncertainty-qualified outputs.
- Evaluate evidence grounding, individual fidelity, simulation validity, safety, and calibration.
- Ensure simulation results do not become real-person facts automatically.

## 5. Non-Responsibilities

This module does not:

- Capture OSINT.
- Approve claims.
- Override consent policy.
- Create person facts from generated hypotheses.
- Produce high-impact automated decisions.

## 6. Logical Component Architecture

```text
Simulation, Output, Trace, and Evaluation Module
 ├── Scenario Builder
 ├── Environment Model
 ├── Participant and Agent Loader
 ├── Observation Model
 ├── Consent-Aware Evidence Packet Interface
 ├── LLM Agent Runtime Interface
 ├── ABM / Environment State Runtime
 ├── Action Space and Policy Filter
 ├── Interaction Engine
 ├── Intervention Engine
 ├── Output Guardrail Layer
 ├── Simulation Trace Graph Writer
 ├── Evaluation Runner
 ├── Calibration Manager
 └── Simulation Reporting View
```

## 7. Scenario Spec

```json
{
  "scenario_id": "scenario_001",
  "purpose": "research_simulation",
  "participants": ["agent_001", "agent_002"],
  "environment_type": "policy_response",
  "time_horizon": "72_hours",
  "initial_state": {
    "policy_announced": false,
    "public_awareness": "low"
  },
  "event_schedule": [
    {"time": "t0", "event": "policy_announcement"},
    {"time": "t12h", "event": "industry_media_coverage"}
  ],
  "channels": ["email", "public_news", "professional_network"],
  "allowed_actions": ["read", "ask_question", "share_with_comment", "remain_silent"],
  "forbidden_actions": ["impersonate_real_person", "reveal_sensitive_data"],
  "interventions": ["provide_transition_resources", "delay_timeline"],
  "metrics": ["support_likelihood", "concern_topics", "information_diffusion"],
  "review_requirements": ["individual_output_review_if_red_risk"]
}
```

## 8. Environment Model

Environment components:

```text
clock
world_state
information_channels
social_network_context
institutional_rules
event_scheduler
action_executor
resource_model
observation_model
intervention_engine
metric_collector
```

Environment state object:

```json
{
  "environment_state_id": "envstate_001",
  "scenario_id": "scenario_001",
  "timestep": "t12h",
  "public_events": ["industry_media_coverage"],
  "channel_states": {
    "public_news": "active",
    "professional_network": "moderate_discussion"
  },
  "active_interventions": [],
  "network_state_summary": "aggregate_or_pseudonymized",
  "resource_constraints": {
    "transition_support": "not_announced"
  }
}
```

## 9. LLM + ABM Hybrid Runtime

LLM agent responsibilities:

```text
interpret observations
map evidence packet to scenario concerns
generate candidate actions
produce simulated dialogue or response
express uncertainty
explain evidence basis
```

ABM/environment responsibilities:

```text
advance time
schedule events
apply institutional rules
propagate information
update network state
execute allowed actions
collect metrics
repeat trials
compare interventions
```

Hybrid separation:

```text
LLM generates candidate semantic behavior.
Policy filter validates candidate behavior.
Environment executes allowed behavior.
Trace records evidence, policy, action, and outcome.
```

## 10. Simulation Loop

```text
for each timestep:
    environment emits events
    agent receives observation
    retrieval module assembles evidence packet
    policy engine validates packet
    agent interprets scenario
    agent proposes candidate actions
    policy engine filters candidate actions
    environment executes valid actions
    environment updates state
    output guardrails classify generated text
    trace graph records step
```

## 11. Action Object

```json
{
  "action_id": "act_001",
  "run_id": "run_001",
  "agent_id": "agent_001",
  "timestep": "t0",
  "action_type": "ask_question",
  "action_summary": "Agent asks about implementation timeline and resource burden.",
  "evidence_packet_id": "packet_001",
  "confidence": 0.68,
  "policy_status": "allowed",
  "blocked_reason": null
}
```

## 12. Simulation Modes

### 12.1 Individual Response Simulation

One agent, one scenario.

Output:

```text
likely concerns
likely questions
support conditions
opposition conditions
confidence
uncertainty
source basis
```

### 12.2 Small-Group Interaction Simulation

Multiple consented agents in a bounded environment.

Architecture additions:

```text
turn-taking
shared observations
private observations
relationship context
meeting or negotiation state
conflict and consensus tracking
```

### 12.3 Network / Society Simulation

Consented agents plus role-based or synthetic agents.

Architecture additions:

```text
network topology
information diffusion
recommendation or exposure model
cohort-level metrics
small-cell suppression
synthetic non-consented nodes
intervention comparisons
```

Default posture:

```text
aggregate-only
no named influence ranking
no vulnerability targeting
no person-level intervention recommendation
```

## 13. Output Guardrails

Output requirements:

```text
simulation disclosure
third-person or clearly simulated framing
confidence band
uncertainty factors
evidence basis
assumptions
alternative outcomes
forbidden conclusions
human review flag
```

Blocked output categories:

```text
first-person impersonation
sensitive attribute inference
manipulation strategy
vulnerability exploitation
named influence ranking
high-impact decision recommendation
unsupported allegation
real-intent assertion
```

Output object:

```json
{
  "output_id": "out_001",
  "run_id": "run_001",
  "agent_id": "agent_001",
  "scenario_id": "scenario_001",
  "output_type": "probabilistic_scenario_response",
  "simulated_response": "The agent is more likely to ask about timeline and resource burden.",
  "confidence": 0.68,
  "evidence_basis": ["claim_001", "claim_002"],
  "uncertainty_factors": ["no_recent_direct_statement"],
  "usage_restrictions": ["simulation_only", "not_actual_intent"],
  "review_required": false,
  "policy_decision_ids": ["decision_001"]
}
```

## 14. Simulation Trace Graph

Trace graph elements:

```text
Scenario
Run
Timestep
Observation
EvidencePacket
CandidateAction
BlockedAction
ExecutedAction
EnvironmentTransition
Output
Metric
PolicyDecision
ReviewDecision
```

Trace object:

```json
{
  "run_id": "run_001",
  "scenario_id": "scenario_001",
  "agent_ids": ["agent_001"],
  "started_at": "2026-05-07T00:00:00Z",
  "evidence_packets": ["packet_001"],
  "steps": [
    {
      "timestep": "t0",
      "observation_id": "obs_001",
      "candidate_actions": ["ask_question", "wait_for_detail"],
      "blocked_actions": [],
      "executed_action": "act_001",
      "output_id": "out_001",
      "policy_decisions": ["decision_001"]
    }
  ],
  "metrics": {
    "evidence_coverage": 0.92,
    "unsupported_inference_rate": 0.04,
    "sensitive_leakage_rate": 0.0,
    "third_party_leakage_rate": 0.0
  }
}
```

Critical rule:

```text
Simulation Trace Graph is not Evidence Graph.
Trace hypotheses do not become personal facts without separate review.
```

## 15. Evaluation Architecture

Evaluation categories:

```text
evidence_grounding
individual_fidelity
simulation_validity
safety_and_governance
calibration_and_drift
```

### 15.1 Evidence-Grounding Metrics

```text
evidence_coverage
unsupported_inference_rate
citation_completeness
contradiction_handling_score
freshness_score
third_party_minimization_score
```

### 15.2 Individual-Fidelity Metrics

```text
participant_agreement
holdout_questionnaire_accuracy
historical_replay_accuracy
communication_style_similarity
uncertainty_appropriateness
refusal_when_evidence_insufficient
```

### 15.3 Simulation-Validity Metrics

```text
run_stability
scenario_sensitivity
network_diffusion_fit
intervention_effect_consistency
baseline_comparison
```

### 15.4 Safety Metrics

```text
sensitive_leakage_rate
identity_error_rate
forbidden_output_rate
impersonation_risk_score
review_override_rate
withdrawal_compliance_rate
third_party_leakage_rate
policy_regression_failure_rate
```

## 16. Calibration Architecture

Calibration may update:

```text
retrieval weighting
confidence models
recency decay
scenario assumptions
uncertainty thresholds
evaluation baselines
guardrail thresholds
```

Calibration must not automatically update:

```text
real-person facts
identity associations
consent permissions
sensitive claims
approved source scope
```

## 17. Integration Contracts

| Module | Contract |
|---|---|
| Governance | Scenario authorization, action/output restrictions, policy decisions. |
| Agent Construction | Agent card and memory policy. |
| Retrieval | Evidence packet per agent per timestep. |
| Temporal KG | Approved claims and evidence references. |
| Output Guardrails | Blocks or routes risky output. |
| Review | Reviews high-risk outputs and contested traces. |
| Participant Rights | Shows simulation history and contest options. |
| Evaluation | Provides calibration and risk signals. |

## 18. Architectural Invariants

```text
Simulation output is not a fact.
Agent action is not real-world intent.
Network metrics cannot become named targeting recommendations.
High-risk outputs require policy and review gates.
Calibration cannot rewrite personal evidence automatically.
Every run must be traceable and replayable at the architecture-object level.
```

## 19. MVP Boundary

Minimum viable architecture:

```text
scenario spec
single-agent runtime interface
evidence packet input
simple environment state
candidate action object
output guardrail object
simulation trace object
basic evidence-grounding and safety metrics
```

Deferred:

```text
large-scale network simulation
recommendation-system modeling
spatial environments
complex interventions
multi-run calibration dashboards
advanced group dynamics metrics
```

## 20. References

[^mesa]: Mesa documentation: https://mesa.readthedocs.io/
[^netlogo]: NetLogo documentation: https://docs.netlogo.org/
[^anylogic]: AnyLogic agent-based modeling overview: https://www.anylogic.com/use-of-simulation/agent-based-modeling/
[^gama]: GAMA Platform documentation: https://gama-platform.org/wiki/Home
[^concordia]: Google DeepMind Concordia: https://github.com/google-deepmind/concordia
[^agentsociety]: AgentSociety paper: https://arxiv.org/abs/2502.08691
[^oasis]: OASIS paper: https://arxiv.org/abs/2411.11581
[^gen-agents]: Park et al., “Generative Agents: Interactive Simulacra of Human Behavior,” arXiv, 2023: https://arxiv.org/abs/2304.03442



---
