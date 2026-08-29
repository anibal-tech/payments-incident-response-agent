# Payments Incident Response Agent

AI-assisted payment incident analysis and decision-support workflow orchestrated with n8n.

The project demonstrates how Generative AI can support payment incident response while keeping severity validation, Root Cause Analysis confirmation, production remediation, external communications, and incident decisions under explicit human control.

> **AI analyzes, hypothesizes, recommends, and summarizes. Evidence establishes facts. Humans authorize consequential actions.**

---

## Overview

Payment incidents require teams to rapidly understand business impact, investigate possible causes, coordinate technical ownership, define response actions, and communicate clearly with business and technology stakeholders.

The **Payments Incident Response Agent** explores how Generative AI and workflow automation can assist with that process.

The workflow receives structured payment-incident information and progressively transforms it into:

- an impact and severity recommendation,
- ranked RCA hypotheses,
- evidence-oriented validation steps,
- role-based ownership assignments,
- a prioritized action plan,
- an executive incident brief,
- a human-review decision point.

The solution follows a **human-in-the-loop** architecture.

AI provides decision support.

It does not autonomously confirm root cause, authorize production changes, communicate with regulators, or close incidents.

---

## Workflow

```text
Payment Incident
      ↓
Normalize Incident
      ↓
Classify Impact
      ↓
Add Incident Context
      ↓
Generate RCA Hypotheses
      ↓
Map RCA Ownership
      ↓
Generate Action Plan
      ↓
Generate Executive Brief
      ↓
Human Review
      ↓
Approval Gate
     ↙     ↘
 FALSE     TRUE
   ↓         ↓
Await      Approved for
Human      Human-Controlled
Approval   Execution
```

The workflow intentionally separates:

```text
AI Reasoning
    ↓
Impact assessment
RCA hypotheses
Action recommendations
Executive summarization

Deterministic Logic
    ↓
Incident normalization
Context assembly
Ownership mapping
Approval evaluation
Workflow-state routing

Human Authority
    ↓
Severity validation
RCA confirmation
Production-action approval
External-communication approval
Incident decisions
```

---

## What the MVP Does

### 1. Structured Incident Intake

The workflow receives structured payment-incident information including:

- affected service,
- payment channel,
- observed symptom,
- error code,
- transaction exposure,
- customer impact,
- financial-impact status,
- regulatory-impact status,
- known changes,
- dependencies,
- initial observations.

All sample incident data included in this repository is fictional and sanitized.

### 2. Impact Classification

An LLM analyzes the incident and recommends an initial severity classification.

The model must distinguish:

- confirmed facts,
- observations,
- unknowns,
- recommendations.

Severity remains advisory and requires human validation.

### 3. Incident Context

The MVP enriches the incident with fictional historical incident references and predefined ownership information.

Historical incidents are used to identify investigation patterns.

Historical similarity is **not** treated as proof of causation.

### 4. RCA Hypothesis Generation

The AI generates ranked Root Cause Analysis hypotheses containing:

- suspected failure domain,
- supporting evidence,
- contradicting or weakening evidence,
- confidence,
- validation steps,
- additional information required.

The output represents **hypotheses only**.

The AI does not declare a confirmed root cause.

### 5. Deterministic Ownership Mapping

Suspected failure domains are mapped to predefined owner roles using deterministic workflow logic.

This prevents the LLM from inventing individuals or organizational responsibilities.

Ownership identifies responsibility for investigation.

It does not confirm the associated RCA hypothesis.

### 6. Prioritized Action Plan

The workflow generates proposed actions using three priorities:

```text
P1 → Immediate diagnostics, evidence gathering, monitoring, and coordination

P2 → Deeper investigation, contingency preparation, and remediation readiness

P3 → Post-incident improvement, testing, observability, and governance
```

Production modifications remain subject to explicit human approval.

### 7. Executive Incident Brief

The final AI stage transforms the technical analysis into a concise executive view covering:

- recommended severity,
- business and customer impact,
- transaction exposure,
- known facts,
- key unknowns,
- leading unconfirmed hypotheses,
- proposed actions,
- pending decisions.

This stage performs controlled summarization rather than additional RCA reasoning.

### 8. Human Review and Approval Gate

The workflow ends with an explicit human-control boundary.

The default review state is:

```text
review_status = Pending
severity_validated = false
production_action_approved = false
external_communication_approved = false
```

Therefore, the workflow defaults to:

```text
workflow_status = BLOCKED_PENDING_HUMAN_APPROVAL
automation_allowed = false
```

Even when the required approvals are explicitly provided:

```text
workflow_status = APPROVED_FOR_HUMAN_CONTROLLED_EXECUTION
automation_allowed = false
```

Human approval does not grant autonomous execution authority to the AI workflow.

---

## Architecture

The MVP combines generative reasoning with deterministic workflow controls.

| Capability | Implementation |
|---|---|
| Incident normalization | Deterministic |
| Impact assessment | AI-assisted |
| Severity confirmation | Human |
| Context assembly | Deterministic |
| RCA hypothesis generation | AI-assisted |
| RCA confirmation | Human |
| Ownership mapping | Deterministic |
| Action recommendations | AI-assisted |
| Production authorization | Human |
| Executive summarization | AI-assisted |
| Approval evaluation | Deterministic |
| Autonomous production execution | Disabled |

This separation is intentional.

Generative AI is used where interpretation and reasoning are valuable.

Deterministic logic is used where predictable control behavior is required.

Human authority remains responsible for consequential operational decisions.

For a detailed description, see:

[Architecture Overview](./docs/architecture-overview.md)

---

## Responsible AI and Safety

Payment incident response is a high-consequence environment.

The project therefore includes explicit controls addressing risks such as:

- hallucination and unsupported inference,
- causal overreach,
- historical similarity bias,
- uncertainty compression,
- financial-impact misinterpretation,
- sensitive payment-data exposure,
- invented operational parameters,
- invented diagnostic time windows,
- unsafe production remediation,
- external communication risk,
- approval bypass.

### Key Guardrails

The workflow follows several core rules:

```text
Correlation ≠ Causation

RCA Hypothesis ≠ Confirmed Root Cause

Transaction Exposure ≠ Confirmed Financial Loss

Proposed Action ≠ Executed Action

Human Approval ≠ Autonomous AI Execution
```

The prompts also prohibit requesting or exposing sensitive information such as:

- PAN,
- CVV,
- PIN,
- PII,
- credentials,
- authentication secrets,
- encryption keys,
- sensitive payment payloads.

Transaction-level investigation should use sanitized, masked, tokenized, or otherwise non-sensitive diagnostic information.

For the complete safety model, see:

[Responsible AI and Safety](./docs/responsible-ai-and-safety.md)

---

## Prompt Pipeline

The AI reasoning layer is documented independently from the workflow implementation.

```text
classify-impact.md
        ↓
generate-rca-hypotheses.md
        ↓
generate-actions.md
        ↓
executive-incident-brief.md
```

Each prompt contains:

- purpose,
- inputs,
- expected output,
- governance controls,
- safety constraints,
- production prompt.

See:

[Prompt Documentation](./prompts/README.md)

---

## Sample Incident

The repository includes a fictional payment incident representing elevated card authorization declines.

Example:

```json
{
  "incident_id": "PAY-2026-001",
  "service": "payment-authorization",
  "payment_channel": "card-present",
  "symptom": "Elevated card authorization declines",
  "error_code": "91",
  "transactions_affected": 12850,
  "estimated_transaction_value_mxn": 4200000,
  "customer_impact": "High",
  "financial_impact": "Under review",
  "regulatory_impact": "Under review"
}
```

The estimated transaction value represents transaction exposure and must not be interpreted as confirmed financial loss.

The sample dataset also includes:

- fictional historical payment incidents,
- a role-based ownership matrix.

See:

[Sample Data](./sample-data/README.md)

---

## Sample Output

The workflow produces a human-reviewable executive incident brief.

For the sample scenario, the workflow demonstrates how AI can communicate:

```text
Recommended Severity
        ↓
Current Impact
        ↓
Known Facts / Unknowns
        ↓
Unconfirmed RCA Hypotheses
        ↓
Proposed Actions
        ↓
Decisions Requiring Human Authority
```

A human-readable example is available here:

[Sample Executive Incident Brief](./outputs/sample-executive-incident-brief.md)

---

## Technology

The MVP uses:

- **n8n** — workflow orchestration
- **OpenAI-compatible LLM** — generative reasoning
- **gpt-5-mini** — model used during MVP testing
- **Structured Output Parser** — predictable LLM outputs
- **JavaScript** — deterministic workflow logic
- **JSON** — incident and workflow data structures
- **CSV / Markdown** — fictional context and ownership references
- **Structured Prompting** — reasoning and governance controls
- **Human-in-the-Loop Design** — consequential decision control

---

## Repository Structure

```text
payments-incident-response-agent/
│
├── README.md
├── .gitignore
│
├── docs/
│   ├── README.md
│   ├── architecture-overview.md
│   └── responsible-ai-and-safety.md
│
├── prompts/
│   ├── README.md
│   ├── classify-impact.md
│   ├── generate-rca-hypotheses.md
│   ├── generate-actions.md
│   └── executive-incident-brief.md
│
├── sample-data/
│   ├── README.md
│   ├── sample_incident.json
│   ├── ownership_matrix.csv
│   └── historical_incidents.md
│
├── workflow/
│   ├── README.md
│   └── n8n-workflow.json
│
├── outputs/
│   ├── README.md
│   └── sample-executive-incident-brief.md
│
└── assets/
    └── diagrams/
        └── payments-incident-response-agent-flow.png
```

---

## Running the Workflow

A sanitized n8n workflow export is included in:

```text
workflow/n8n-workflow.json
```

To test the MVP:

1. Import the workflow into n8n.
2. Reconnect the OpenAI model nodes using your own credential.
3. Review the workflow configuration.
4. Keep the Human Review node in its fail-safe default state.
5. Execute the workflow from the Manual Trigger.
6. Verify that the Approval Gate follows the blocked branch when approvals remain false.

The public workflow export does not include the original OpenAI credential configuration.

For detailed instructions, see:

[Workflow Documentation](./workflow/README.md)

---

## MVP Boundaries

The current MVP intentionally does not connect autonomously to:

- production payment platforms,
- payment processors,
- ServiceNow,
- Jira,
- Microsoft Teams,
- monitoring platforms,
- regulatory systems,
- production configuration systems.

It does not autonomously:

- rollback deployments,
- change payment routing,
- fail over processors,
- modify fraud rules,
- modify production configuration,
- contact customers,
- contact regulators,
- close incidents.

These limitations are intentional design boundaries, not missing autonomous capabilities.

---

## Future Evolution

Potential future extensions include:

- Retrieval-Augmented Generation (RAG) for incident history,
- vector-based incident similarity search,
- approved runbook retrieval,
- observability integrations,
- ServiceNow or Jira integration,
- Microsoft Teams or Slack collaboration,
- role-based access control,
- policy-based approval engines,
- audit logging,
- prompt and model versioning,
- evaluation datasets,
- AI-output quality monitoring,
- incident feedback loops.

A production implementation would require additional security, compliance, privacy, operational-resilience, access-control, and payment-industry governance.

---

## Status

**MVP implemented and tested end-to-end.**

Current project artifacts include:

- functional n8n workflow,
- sanitized public workflow export,
- structured AI prompts,
- fictional sample incident data,
- historical incident context,
- deterministic ownership mapping,
- Human Review and Approval Gate,
- Responsible AI documentation,
- architecture documentation,
- representative executive incident output.

The project remains open to future extensions beyond the current MVP scope.

---

## Disclaimer

This project is a professional portfolio demonstration.

All incidents, transaction volumes, financial values, system names, organizations, historical incidents, and operational data used in this repository are fictional or sanitized.

No real payment, customer, cardholder, merchant, credential, or production information is included.

The project is not intended to represent a production-ready payment incident-management platform.

---

## Author

**Anibal Arias**

Technology Development Manager | Payments | Software Delivery | Applied Generative AI | Automation
