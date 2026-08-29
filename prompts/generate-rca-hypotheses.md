# RCA Hypothesis Generation Prompt

## Purpose

This prompt generates ranked root cause hypotheses for a payment incident using the available incident evidence, impact classification, and historical incident references.

Its purpose is to help human incident management and technical teams identify plausible investigation paths without presenting AI-generated hypotheses as confirmed root causes.

The prompt is designed to support investigation, not replace the formal Root Cause Analysis (RCA) process.

## Inputs

The prompt receives:

- normalized incident information,
- AI-assisted impact classification,
- historical incident references.

The historical incidents are fictional examples included in the MVP as contextual reference material.

They may help identify investigation patterns, but similarity to a previous incident must never be interpreted as proof that the same root cause is occurring again.

## Output

The prompt produces a structured RCA analysis containing:

- RCA status,
- ranked hypotheses,
- suspected failure domain,
- hypothesis description,
- supporting evidence,
- contradicting or weakening evidence,
- confidence score,
- validation steps,
- additional information required,
- overall assessment,
- human-validation requirement.

Each hypothesis remains explicitly unconfirmed until validated through evidence and human investigation.

## Safety and Governance

### RCA remains human-controlled

The AI does not declare a root cause.

The final RCA remains the responsibility of human incident management and the appropriate technical teams.

### Correlation is not causation

A deployment, configuration change, or other event occurring before an incident may justify investigation.

It does not establish causation.

Temporal correlation must therefore remain explicitly identified as evidence for investigation rather than proof of root cause.

### Historical incidents are reference material

Historical incidents may be used to recognize investigation patterns.

They must not be used as evidence that the current incident has the same root cause.

Historical similarity alone must not increase confidence in a hypothesis.

### Evidence boundaries

The model must not invent:

- telemetry,
- logs,
- processor responses,
- fraud signals,
- configuration changes,
- deployment details,
- system behavior,
- technical mechanisms.

Technical explanations that are not supported by the available evidence may only be mentioned as explicitly hypothetical investigation possibilities.

### Dependency uncertainty

The presence of a dependency in the incident context does not mean that dependency is affected.

Similarly, the absence of reported impact does not prove that a dependency or payment channel is healthy.

### Diagnostic time-window safety

The model must not invent diagnostic time windows.

When an explicit incident window exists, it may use that window.

If broader analysis before or after the incident would be useful, the model must refer to an:

`appropriate human-defined investigation window`

rather than inventing start or end times.

### Payment-data safety

Validation activities must not request or expose:

- PAN,
- CVV,
- PIN,
- PII,
- credentials,
- authentication secrets,
- sensitive payment payloads.

Transaction-level investigation must use sanitized, tokenized, masked, or otherwise non-sensitive diagnostic information.

### Production safety

Validation steps are diagnostic and evidence-gathering activities.

The AI must not automatically execute or recommend executing production modifications without human approval, including:

- rollback,
- routing changes,
- processor failover,
- fraud-rule changes,
- configuration changes,
- deployments.

---

## Prompt

```text
You are an AI assistant supporting a human payment incident commander.

Your task is to generate ranked root cause hypotheses for the payment incident below.

IMPORTANT GOVERNANCE RULES:
- Do not declare any root cause as confirmed.
- Historical incidents are reference material only.
- Similarity to a historical incident does not prove causality.
- Do not invent telemetry, logs, processor responses, fraud signals, configuration changes, deployment details, system behavior, or technical mechanisms that are not present in the input.
- Do not assume that a listed dependency is affected.
- Do not assume that a recent deployment caused the incident.
- Temporal correlation is evidence for investigation, not proof of causation.
- Clearly distinguish supporting evidence from contradicting or weakening evidence.
- Confidence must reflect the quality of available evidence.
- If evidence is insufficient, explicitly say so.
- Production actions require human approval.
- The final RCA remains the responsibility of human incident management and technical teams.

HYPOTHETICAL EXAMPLES:
- Technical mechanisms such as malformed requests, incorrect field mappings, routing errors, fraud thresholds, processor behavior, configuration errors, or software regressions may only be mentioned when supported by input evidence.
- If such mechanisms are useful as investigative possibilities but are not supported by evidence, explicitly label them as hypothetical examples, not observed behavior.
- Never present a hypothetical mechanism as something that occurred.

UNCERTAINTY HANDLING:
- Preserve words such as "appears", "possible", "potential", "reported", and "under review".
- Do not convert "appears affected" into "confirmed affected".
- Do not infer that other channels, services, dependencies, processors, or systems are healthy merely because no impact was reported.
- Do not convert absence of evidence into evidence of absence.

HISTORICAL INCIDENT SAFETY:
- Historical incidents may be used to identify investigation patterns.
- Do not increase confidence merely because a historical incident had similar symptoms.
- Clearly state when historical similarity is being used only as an investigative reference.

CURRENT INCIDENT:
{{ JSON.stringify($json.incident) }}

IMPACT CLASSIFICATION:
{{ JSON.stringify($json.impact_classification) }}

HISTORICAL INCIDENT REFERENCES:
{{ JSON.stringify($json.historical_incidents) }}

Generate the most plausible RCA hypotheses based only on the available evidence.

For each hypothesis:
1. Give it a rank.
2. Name the suspected component or failure domain.
3. Describe the hypothesis using explicitly uncertain language.
4. List supporting evidence actually present in the input.
5. List contradicting or weakening evidence.
6. Provide a confidence score from 0.0 to 1.0.
7. Provide validation steps that a human technical team should perform.
8. Identify additional information required.

VALIDATION SAFETY:
- Validation steps must be diagnostic and evidence-gathering activities.
- Do not request PAN, CVV, PII, authentication secrets, credentials, or sensitive payment payloads.
- Refer only to sanitized, tokenized, masked, or non-sensitive diagnostic information where transaction-level analysis is needed.
- Do not recommend executing rollback, routing changes, processor failover, fraud rule changes, configuration changes, deployments, or any other production modification automatically.
- Do not invent diagnostic time windows.
- Use the actual incident window when explicitly provided.
- If broader pre-incident or post-incident analysis is useful,
  refer to an "appropriate human-defined investigation window"
  without inventing start or end times.

The output represents hypotheses for investigation only, not a confirmed RCA.
```

## Structured Output Example

```json
{
  "rca_status": "Hypotheses only - not confirmed",
  "hypotheses": [
    {
      "rank": 1,
      "suspected_domain": "payment-routing",
      "hypothesis": "A routing-related configuration or deployment change may be contributing to elevated card-present authorization declines.",
      "supporting_evidence": [
        "Authorization declines increased shortly after a production release.",
        "Authorization latency remains within expected range.",
        "Only card-present traffic appears affected."
      ],
      "contradicting_evidence": [
        "No routing configuration change has been confirmed.",
        "No routing telemetry is available."
      ],
      "confidence": 0.65,
      "validation_steps": [
        "Compare routing configuration before and after the release.",
        "Review sanitized routing decision logs for affected transactions.",
        "Compare card-present routing behavior with other available payment-channel evidence."
      ],
      "additional_information_required": [
        "Routing configuration history",
        "Sanitized routing decision logs",
        "Processor response distribution"
      ]
    }
  ],
  "overall_assessment": "Available evidence supports investigation of several possible failure domains, but no root cause is confirmed.",
  "requires_human_validation": true
}
```

## Role in the Workflow

```text
Add Incident Context
        ↓
Generate RCA Hypotheses  ← this prompt
        ↓
Map RCA Ownership
        ↓
Generate Action Plan
```

The RCA hypotheses generated by this stage are subsequently mapped to owner roles using deterministic workflow logic.

Ownership assignment does not validate or confirm a hypothesis.

The hypotheses remain investigative recommendations until supporting evidence is reviewed by the appropriate human technical teams.

## Design Principle

The central design principle of this prompt is:

> AI may propose what to investigate. Evidence and authorized humans determine what actually happened.

This separation helps prevent an AI-generated explanation from being mistaken for a confirmed Root Cause Analysis.
