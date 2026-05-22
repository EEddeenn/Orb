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
