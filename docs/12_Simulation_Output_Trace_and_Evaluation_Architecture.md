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
