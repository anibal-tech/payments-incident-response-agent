# Architecture Overview

## Overview

The **Payments Incident Response Agent - MVP** is an AI-assisted workflow designed to support human teams during payment incidents.

The system combines:

- deterministic workflow logic,
- Large Language Model (LLM) reasoning,
- structured outputs,
- fictional historical incident context,
- deterministic ownership mapping,
- explicit human review,
- fail-safe approval controls.

The architecture deliberately separates **AI reasoning from operational authority**.

The AI may analyze incident information, generate hypotheses, recommend investigation activities, and prepare executive summaries.

It does not autonomously confirm root cause, authorize remediation, modify production systems, or close incidents.

---

## High-Level Architecture

```text
┌─────────────────────────────┐
│        Incident Input       │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│     Normalize Incident      │
│      Deterministic          │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│       Classify Impact       │
│          AI / LLM           │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│    Add Incident Context     │
│      Deterministic          │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│  Generate RCA Hypotheses    │
│          AI / LLM           │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│     Map RCA Ownership       │
│      Deterministic          │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│    Generate Action Plan     │
│          AI / LLM           │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│ Generate Executive Brief    │
│          AI / LLM           │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│        Human Review         │
│      Human Authority        │
└──────────────┬──────────────┘
               ↓
┌─────────────────────────────┐
│        Approval Gate        │
│      Deterministic IF       │
└─────────┬───────────┬───────┘
          │           │
       FALSE         TRUE
          │           │
          ↓           ↓
 Await Human     Approved for
   Approval      Human-Controlled
                 Execution
```

---

## Architectural Principles

### 1. AI for reasoning, deterministic logic for control

The architecture does not use an LLM for every workflow decision.

Generative AI is used where interpretation and reasoning are useful:

- impact assessment,
- RCA hypothesis generation,
- action-plan generation,
- executive summarization.

Deterministic logic is used where predictable behavior is preferable:

- incident normalization,
- context assembly,
- ownership mapping,
- approval evaluation,
- workflow-state routing.

This separation reduces unnecessary model dependency and makes critical control paths easier to understand and audit.

---

### 2. Human authority remains outside the model

The LLM can generate recommendations, but operational authority remains human-controlled.

The workflow reserves the following decisions for authorized humans:

- official severity validation,
- confirmed Root Cause Analysis,
- production remediation approval,
- external communication approval,
- regulatory communication decisions,
- incident closure.

The architecture therefore implements AI as a **decision-support layer**, not as an autonomous incident commander.

---

### 3. Structured AI outputs

The LLM stages use structured outputs rather than unrestricted natural-language responses.

This makes downstream processing more predictable and allows the workflow to reason over explicit fields such as:

```text
recommended_severity
confidence
missing_information
hypotheses
owner_role
requires_human_approval
decisions_required
requires_human_review
```

Structured output also makes it easier to validate, persist, audit, or integrate results in future versions.

---

## Workflow Stages

### Stage 1 — Incident Input

The workflow starts with a structured incident payload.

The MVP uses fictional payment-incident data containing information such as:

- service,
- payment channel,
- symptom,
- error code,
- affected transaction count,
- estimated transaction value,
- customer impact,
- financial impact,
- regulatory impact,
- known changes,
- dependencies,
- initial observations.

No real customer, merchant, cardholder, or production-sensitive information is required by the public MVP.

---

### Stage 2 — Normalize Incident

**Type:** Deterministic

The normalization stage prepares the incident for downstream analysis.

Responsibilities include:

- standardizing the incident structure,
- calculating incident duration from supplied timestamps,
- preserving source observations,
- attaching AI-governance metadata.

Example governance metadata:

```json
{
  "severity_requires_human_validation": true,
  "rca_is_hypothesis_only": true,
  "production_actions_require_human_approval": true
}
```

This establishes governance expectations before generative reasoning begins.

---

### Stage 3 — Classify Impact

**Type:** AI / LLM

The first generative stage analyzes the normalized incident and recommends an initial impact classification and severity.

The model evaluates available evidence while distinguishing:

- confirmed facts,
- observations,
- unknowns,
- recommendations.

Important constraints include:

- severity remains a recommendation,
- transaction exposure is not treated as confirmed financial loss,
- recent changes are not treated as confirmed causes,
- uncertainty must be preserved,
- missing information must be identified.

Output from this stage becomes context for the RCA process.

---

### Stage 4 — Add Incident Context

**Type:** Deterministic

This stage assembles the context required for RCA hypothesis generation.

The MVP combines:

```text
Normalized Incident
        +
Impact Classification
        +
Historical Incident References
        +
Ownership Matrix
        +
Governance Controls
```

Historical incidents are fictional and sanitized.

They provide investigation patterns rather than causal evidence.

A production implementation could replace or complement this static context with approved retrieval mechanisms, knowledge bases, observability systems, incident-management platforms, or Retrieval-Augmented Generation (RAG).

---

### Stage 5 — Generate RCA Hypotheses

**Type:** AI / LLM

The model generates ranked Root Cause Analysis hypotheses.

Each hypothesis may include:

- suspected failure domain,
- hypothesis description,
- supporting evidence,
- contradicting or weakening evidence,
- confidence score,
- validation steps,
- additional information required.

The output is explicitly labeled as hypotheses only.

The model must not:

- confirm RCA,
- invent telemetry,
- assume listed dependencies are affected,
- convert temporal correlation into causation,
- treat historical similarity as proof,
- invent diagnostic time windows.

Validation activities are designed around evidence gathering rather than autonomous remediation.

---

### Stage 6 — Map RCA Ownership

**Type:** Deterministic

Ownership assignment is intentionally separated from generative reasoning.

The workflow maps suspected failure domains to predefined owner roles.

Example:

```text
authorization-service
        ↓
Payments Platform Lead
```

This approach prevents the model from inventing individuals or organizational ownership.

The mapping assigns responsibility for investigation.

It does not confirm the associated RCA hypothesis.

---

### Stage 7 — Generate Action Plan

**Type:** AI / LLM

The action-plan stage converts available incident evidence and RCA hypotheses into prioritized recommendations.

Actions are organized as:

```text
P1 → Immediate diagnostics and evidence gathering

P2 → Investigation, coordination, and contingency preparation

P3 → Preventive improvements and post-incident activities
```

The model is prohibited from inventing operational parameters such as:

- thresholds,
- sample sizes,
- deadlines,
- percentages,
- SLAs,
- diagnostic windows.

When a parameter is required but unavailable, it must remain human-defined or TBD.

Production changes are recommendations only and require explicit human approval.

---

### Stage 8 — Generate Executive Brief

**Type:** AI / LLM

This stage converts the technical investigation into a concise executive view.

Unlike the RCA stage, it should not perform additional root-cause reasoning.

Its responsibility is controlled summarization.

The brief communicates:

- observed incident status,
- business impact,
- customer impact,
- transaction exposure,
- known facts,
- key unknowns,
- leading unconfirmed hypotheses,
- proposed actions,
- pending human decisions.

A key architectural concern at this stage is preventing **uncertainty compression**.

Executive simplification must not transform:

```text
appears affected
```

into:

```text
confirmed affected
```

or:

```text
possible cause
```

into:

```text
root cause
```

---

## Human Review Boundary

The Human Review node represents the transition between **AI-generated decision support** and **human operational authority**.

The MVP uses fail-safe defaults:

```text
review_status = Pending
severity_validated = false
production_action_approved = false
external_communication_approved = false
```

This means an untouched workflow cannot accidentally represent an incident as approved.

---

## Approval Gate

**Type:** Deterministic

The approval gate evaluates the human-review fields.

Conceptually:

```text
severity_validated
        AND
production_action_approved
        AND
external_communication_approved
```

When the conditions are not satisfied:

```text
workflow_status =
BLOCKED_PENDING_HUMAN_APPROVAL

automation_allowed = false
```

When the required conditions are explicitly satisfied:

```text
workflow_status =
APPROVED_FOR_HUMAN_CONTROLLED_EXECUTION

automation_allowed = false
```

The value of `automation_allowed` intentionally remains `false` on both branches.

The TRUE branch therefore does **not** authorize autonomous production execution.

It only indicates that the workflow has reached a state where explicitly approved actions may proceed through human-controlled operational processes.

---

## Control Plane vs. Reasoning Plane

The architecture can also be understood as two interacting planes.

```text
REASONING PLANE

Classify Impact
      ↓
Generate RCA Hypotheses
      ↓
Generate Action Plan
      ↓
Generate Executive Brief


CONTROL PLANE

Normalize Incident
      ↓
Context Assembly
      ↓
Ownership Mapping
      ↓
Human Review
      ↓
Approval Gate
      ↓
Workflow State
```

The reasoning plane produces recommendations.

The control plane determines how those recommendations move through the workflow.

The reasoning plane does not control the approval gate.

---

## Data Flow

At a simplified level:

```text
Incident Data
     ↓
Normalized Incident
     ↓
Impact Assessment
     ↓
Incident + Context
     ↓
RCA Hypotheses
     ↓
Ownership-Enriched RCA
     ↓
Proposed Action Plan
     ↓
Executive Incident Brief
     ↓
Human Review
     ↓
Controlled Workflow State
```

Each stage adds information while maintaining the original evidence boundaries.

---

## Failure-Safe Behavior

The architecture follows a fail-safe principle:

> Missing approval should block progression rather than imply authorization.

The default workflow state therefore requires explicit human intervention before reaching the approved branch.

Other fail-safe design decisions include:

- severity requires validation,
- RCA remains hypothetical,
- production changes require approval,
- external communications require approval,
- sensitive payment information is excluded,
- autonomous execution remains disabled.

---

## MVP Technology

The current MVP uses:

```text
Workflow Orchestration
└── n8n

Generative Reasoning
└── OpenAI-compatible LLM
    └── gpt-5-mini used during MVP testing

Structured Outputs
└── n8n Structured Output Parser

Context Retrieval
└── Static fictional historical incident context

Ownership Mapping
└── Deterministic workflow logic

Human Control
└── Human Review + deterministic Approval Gate
```

The architecture is intentionally lightweight so the core incident-response reasoning and governance model can be demonstrated without requiring enterprise infrastructure.

---

## Current MVP Boundaries

The MVP does not currently include autonomous integrations with:

- production payment systems,
- payment processors,
- ServiceNow,
- Jira,
- Microsoft Teams,
- monitoring platforms,
- regulatory systems,
- production configuration systems.

It also does not autonomously:

- rollback deployments,
- change payment routing,
- fail over processors,
- modify fraud rules,
- change production configuration,
- contact regulators,
- notify customers,
- close incidents.

These boundaries are intentional.

---

## Potential Future Architecture

A production-oriented evolution could introduce:

```text
Monitoring / Incident Platform
          ↓
Incident Event
          ↓
Normalization
          ↓
Approved Context Retrieval / RAG
     ↙        ↓         ↘
Runbooks   Historical   Observability
           Incidents      Context
          ↓
AI-Assisted Incident Analysis
          ↓
Human Review
          ↓
Policy / Approval Engine
          ↓
Approved Operational Integrations
          ↓
Audit Trail
```

Potential extensions include:

- vector-based incident retrieval,
- enterprise knowledge retrieval,
- observability integrations,
- ServiceNow or Jira integration,
- Microsoft Teams or Slack collaboration,
- role-based access controls,
- approval policies based on action type,
- audit logging,
- prompt and model versioning,
- evaluation datasets,
- model-output quality monitoring,
- incident feedback loops.

Any production implementation would require additional security, compliance, access-control, data-governance, operational-resilience, and payment-industry reviews.

---

## Design Decision Summary

| Concern | Architectural Decision |
|---|---|
| Impact interpretation | AI-assisted |
| Severity confirmation | Human |
| RCA hypothesis generation | AI-assisted |
| RCA confirmation | Human |
| Ownership mapping | Deterministic |
| Action recommendations | AI-assisted |
| Production authorization | Human |
| Executive summarization | AI-assisted |
| Approval evaluation | Deterministic |
| Autonomous production execution | Disabled |
| Sensitive payment data | Excluded |
| Historical incident context | Fictional / sanitized for MVP |

---

## Core Architectural Principle

The architecture is based on a simple separation:

```text
AI can analyze.
AI can hypothesize.
AI can recommend.
AI can summarize.

Deterministic logic can control workflow state.

Humans validate, authorize, and decide.
```

The goal is not to create an autonomous payment incident commander.

The goal is to demonstrate how generative AI can accelerate incident understanding and decision support while keeping consequential operational authority under explicit human control.
