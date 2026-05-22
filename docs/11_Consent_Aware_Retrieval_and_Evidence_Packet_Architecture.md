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
