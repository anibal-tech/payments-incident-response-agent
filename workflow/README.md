# n8n Workflow

This folder contains the public, sanitized export of the **Payments Incident Response Agent - MVP** workflow.

## File

- `n8n-workflow.json`

## Purpose

The workflow demonstrates an AI-assisted payment incident response process built with n8n.

It supports:

- incident normalization,
- initial severity recommendation,
- contextual enrichment,
- ranked RCA hypothesis generation,
- deterministic ownership mapping,
- prioritized action planning,
- executive incident brief generation,
- explicit human review,
- approval gating.

The workflow is designed as **decision support for human incident management**, not as an autonomous incident commander.

## Importing the Workflow

1. Open your n8n workspace.
2. Create a new workflow.
3. Import `n8n-workflow.json`.
4. Reconnect the OpenAI Chat Model nodes using your own OpenAI-compatible credential.
5. Review all node configurations before execution.
6. Keep the Human Review node in its default safe state for the initial test.
7. Run the workflow using `Execute Workflow`.

## OpenAI Configuration

The public workflow export does not contain credentials.

After importing the workflow, reconnect each OpenAI Chat Model node with a credential configured in your own n8n environment.

The MVP was tested using:

- n8n
- OpenAI-compatible chat model
- `gpt-5-mini`

Model availability may depend on your OpenAI and n8n configuration.

## Safe Default State

The public workflow intentionally starts with the Human Review node configured as:

```text
review_status = Pending
severity_validated = false
production_action_approved = false
external_communication_approved = false
```

Therefore, the default execution path is:

```text
Human Review
    ↓
Approval Gate
    ↓
FALSE
    ↓
Await Human Approval
    ↓
BLOCKED_PENDING_HUMAN_APPROVAL
```

The workflow must not proceed to an approved state unless the human approval fields are explicitly changed.

## Human-Controlled Execution

Even when the approval gate evaluates to TRUE, the workflow returns:

```text
workflow_status = APPROVED_FOR_HUMAN_CONTROLLED_EXECUTION
automation_allowed = false
```

This is intentional.

Approval means that authorized humans may proceed with approved actions.

It does **not** authorize the AI workflow to automatically:

- perform production rollbacks,
- modify payment routing,
- fail over processors,
- change fraud rules,
- alter production configuration,
- send merchant or customer communications,
- contact regulators,
- declare the RCA confirmed,
- close the incident.

## Data Safety

The workflow and sample data are fictional and sanitized.

Do not use:

- PAN,
- CVV,
- PIN data,
- customer PII,
- credentials,
- authentication tokens,
- encryption keys,
- unredacted payment payloads,
- confidential internal URLs.

Transaction-level investigation should use only sanitized, masked, tokenized, or non-sensitive diagnostic data.

## Public Export Sanitization

The repository version of `n8n-workflow.json` has been sanitized before publication.

The public file does not contain:

- OpenAI API credentials,
- credential IDs,
- tokens,
- secrets,
- n8n instance identifiers,
- internal workflow IDs,
- private environment URLs.

## MVP Scope

This workflow intentionally uses a simplified architecture for portfolio and demonstration purposes.

The current MVP does not include:

- ServiceNow integration,
- Jira integration,
- Microsoft Teams or Slack notifications,
- production monitoring integrations,
- a vector database,
- automated remediation,
- autonomous multi-agent execution.

These capabilities can be added in a future production-oriented architecture.

## Recommended Test

Run the workflow using the fictional incident provided in the workflow.

The expected default result is:

```text
recommended severity → requires human validation
RCA → hypotheses only
actions → proposed, not executed
production changes → require human approval
approval gate → FALSE
workflow status → BLOCKED_PENDING_HUMAN_APPROVAL
automation allowed → false
```

This verifies the workflow's fail-safe behavior.
