# Incident Action Plan Generation Prompt

## Purpose

This prompt generates a prioritized incident action plan from the available incident evidence, impact classification, RCA hypotheses, validation steps, and assigned owner roles.

Its purpose is to help the human incident commander and technical teams organize the next investigation, coordination, and remediation-preparation activities.

The AI generates recommendations only.

It does not execute actions, authorize production changes, assign individual people, or determine that a proposed remediation should proceed.

## Inputs

The prompt receives the consolidated output produced after RCA ownership mapping, including:

- normalized incident information,
- impact classification,
- ranked RCA hypotheses,
- validation steps,
- deterministic owner-role assignments,
- governance controls.

The RCA hypotheses remain unconfirmed at this stage.

Owner roles are provided by the deterministic ownership-mapping stage rather than generated freely by the language model.

## Output

The prompt produces a structured action plan containing:

- action-plan status,
- prioritized actions,
- unique action IDs,
- P1, P2, or P3 priority,
- action category,
- proposed action,
- owner role,
- RCA hypothesis or incident concern addressed,
- expected result,
- human-approval requirement,
- approval reason,
- immediate investigation focus,
- decisions requiring human review.

The resulting plan represents proposed actions, not executed actions.

## Priority Model

### P1 — Immediate investigation

P1 actions focus on immediate:

- diagnostics,
- evidence collection,
- monitoring,
- safe coordination,
- impact assessment.

These actions help determine what is happening before remediation decisions are made.

### P2 — Investigation and contingency preparation

P2 actions focus on:

- deeper investigation,
- contingency preparation,
- coordination,
- remediation readiness,
- decision support.

A P2 action may prepare a production remediation option, but preparation does not constitute approval to execute it.

### P3 — Prevention and improvement

P3 actions focus on:

- post-incident analysis,
- preventive improvements,
- testing,
- observability,
- governance,
- long-term resilience.

## Safety and Governance

### Recommendations, not execution

The AI may propose actions but must not represent them as authorized, completed, or executed.

Actions remain subject to the appropriate human operational processes.

### RCA uncertainty

Root cause is not confirmed when this prompt executes.

Actions should therefore help validate or rule out hypotheses rather than assume that a particular hypothesis is true.

A suspected component must not be described as having caused the incident unless confirmed evidence is explicitly available.

### Deterministic ownership

The model must use owner roles already assigned during RCA ownership mapping.

It must not:

- invent owner roles,
- assign individual employees,
- infer organizational responsibilities that are not present in the input.

This keeps ownership mapping separate from generative reasoning.

### No invented operational parameters

The model must not invent:

- numeric thresholds,
- diagnostic sample sizes,
- percentages,
- monitoring thresholds,
- alert thresholds,
- deadlines,
- time windows,
- SLAs,
- acceptance criteria.

When such information is required but unavailable, the model must use expressions such as:

- `human-defined threshold`,
- `approved diagnostic sample`,
- `appropriate investigation window`,
- `organization-defined SLA`,
- `TBD by incident commander`.

Reasonable-looking numbers must not be fabricated.

### Payment-data safety

The action plan must never request, expose, reproduce, store, or recommend collecting:

- PAN,
- CVV,
- PIN,
- PII,
- credentials,
- authentication secrets,
- encryption keys,
- sensitive payment payloads.

Transaction-level investigation must use sanitized, masked, tokenized, or otherwise non-sensitive identifiers and diagnostic metadata.

Useful diagnostic information may include:

- timestamps,
- correlation IDs,
- tokenized transaction references,
- response codes,
- component identifiers,
- non-sensitive diagnostic metadata.

External processor information must also be sanitized before being introduced into the AI workflow.

### Production safety

Production modifications require explicit human approval.

This includes:

- rollbacks,
- payment-routing changes,
- processor failover,
- fraud-rule changes,
- configuration changes,
- deployments,
- service restarts,
- feature-flag changes,
- other production modifications.

The AI may recommend investigation or prepare a remediation option.

It must not authorize or execute the modification.

### Communication safety

External communications may have operational, contractual, legal, or regulatory consequences.

Therefore, communications involving:

- customers,
- merchants,
- processors,
- regulators,
- legal teams,
- public channels,

must receive appropriate human approval when applicable.

---

## Prompt

```text
You are an AI assistant supporting a human payment incident commander.

Create a prioritized incident action plan based only on the incident, impact classification, RCA hypotheses, validation steps, and assigned owner roles provided below.

IMPORTANT GOVERNANCE RULES:
- Root cause is NOT confirmed.
- Do not invent evidence, telemetry, system state, owners, deadlines, completed actions, thresholds, sample sizes, percentages, time windows, SLAs, or operational parameters.
- Use only owner roles already assigned in the RCA analysis.
- Do not assign individual people.
- Do not state that an action has been executed.
- Prefer investigation and evidence collection before remediation while root cause remains unconfirmed.
- Do not treat transaction exposure as confirmed financial loss.

NO INVENTED OPERATIONAL PARAMETERS:
- Never invent numeric thresholds, sample sizes, percentages, monitoring thresholds, alert thresholds, deadlines, time windows, or acceptance criteria.
- If an operational parameter is required but is not supplied in the input, use wording such as:
  "human-defined threshold",
  "approved diagnostic sample",
  "appropriate investigation window",
  "organization-defined SLA",
  or "TBD by incident commander".
- Do not create example numbers, even when they appear reasonable.

PAYMENT DATA SAFETY:
- Never request, expose, reproduce, store, or recommend collecting PAN, CVV, PIN, PII, credentials, authentication secrets, encryption keys, or sensitive payment payloads.
- Transaction-level investigation must use sanitized, masked, tokenized, or non-sensitive identifiers and diagnostic metadata.
- Do not request full request or response payloads if they may contain sensitive payment or personal information.
- Prefer logs containing timestamps, correlation IDs, tokenized transaction references, response codes, component identifiers, and non-sensitive diagnostic metadata.
- External processor information must also be sanitized before being used by the AI workflow.

PRODUCTION SAFETY:
- Rollbacks, routing changes, processor failover, fraud-rule changes, configuration changes, deployment changes, service restarts, feature-flag changes, or any other production modification require explicit human approval.
- Customer, merchant, processor, regulatory, legal, or public communications require appropriate human approval when applicable.
- The AI may prepare or recommend these actions but must never represent them as authorized or executed.

UNCERTAINTY:
- Preserve uncertainty from the RCA analysis.
- Do not turn an RCA hypothesis into a fact.
- Do not state that a component caused the incident unless confirmed evidence is present.
- Actions should validate or rule out hypotheses rather than assume them to be true.

INPUT:
{{ JSON.stringify($json) }}

Create a prioritized action plan.

PRIORITIES:

P1 =
Immediate diagnostic, evidence-gathering, monitoring, or safe coordination activity needed to understand current impact.

P2 =
Near-term investigation, contingency preparation, or coordination needed to support a human remediation decision.

P3 =
Post-incident analysis, preventive improvement, testing, observability, or governance enhancement.

For each action:
- assign a unique action_id
- assign P1, P2, or P3
- specify the action category
- describe the proposed action
- use an owner role already present in the RCA analysis
- explain which RCA hypothesis or incident concern the action addresses
- state the expected result
- indicate whether human approval is required
- identify the approval reason when applicable

Do not execute any action.

Generate recommendations only.

If required information or an operational parameter is unavailable, explicitly mark it as requiring human definition rather than inventing a value.
```

## Structured Output Example

```json
{
  "action_plan_status": "Proposed - not executed",
  "actions": [
    {
      "action_id": "ACT-001",
      "priority": "P1",
      "category": "Investigation",
      "action": "Review sanitized authorization-service logs and trace affected card-present transactions using tokenized or non-sensitive diagnostic identifiers.",
      "owner_role": "Payments Platform Lead",
      "addresses": "RCA hypothesis rank 1 - authorization-service",
      "expected_result": "Determine whether available evidence supports or weakens the authorization-service hypothesis.",
      "requires_human_approval": false,
      "approval_reason": "Read-only diagnostic investigation using sanitized data."
    },
    {
      "action_id": "ACT-002",
      "priority": "P2",
      "category": "Potential Remediation",
      "action": "Evaluate rollback readiness for the authorization-service release if evidence supports a deployment-related regression.",
      "owner_role": "Payments Platform Lead",
      "addresses": "RCA hypothesis rank 1 - authorization-service",
      "expected_result": "Prepare a controlled remediation option for human consideration if evidence supports the hypothesis.",
      "requires_human_approval": true,
      "approval_reason": "Any production rollback requires explicit human authorization."
    }
  ],
  "immediate_focus": "Validate the highest-ranked hypotheses through evidence gathering while maintaining service stability.",
  "decisions_required": [
    "Human incident commander validation of recommended severity.",
    "Human authorization before any production remediation."
  ],
  "requires_human_review": true
}
```

## Role in the Workflow

```text
Generate RCA Hypotheses
        ↓
Map RCA Ownership
        ↓
Generate Action Plan  ← this prompt
        ↓
Generate Executive Brief
        ↓
Human Review
```

The action plan produced by this stage is subsequently summarized for senior technology and business stakeholders.

The action plan itself does not authorize execution.

## Separation of Responsibilities

The MVP deliberately separates three responsibilities:

```text
AI reasoning
    ↓
Propose investigation and remediation options

Deterministic workflow logic
    ↓
Map hypotheses to predefined owner roles

Human authority
    ↓
Validate severity
Review evidence
Approve production actions
Approve external communications
```

This separation prevents an AI-generated recommendation from becoming an operational decision simply because it was generated by the workflow.

## Design Principle

The central design principle of this prompt is:

> Investigate before remediating. Prepare before executing. Require human authority before changing production.

The AI helps organize the response to an incident, while operational authority remains outside the model.
