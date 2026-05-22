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
