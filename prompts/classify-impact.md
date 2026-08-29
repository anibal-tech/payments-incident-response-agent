# Impact Classification Prompt

## Purpose

This prompt performs the initial AI-assisted impact assessment of a payment incident.

Its responsibility is to analyze the normalized incident data and recommend an initial severity level while preserving uncertainty and clearly separating confirmed facts from observations, unknowns, and recommendations.

The severity produced by this prompt is **advisory only**. It must be validated by a human incident commander before being treated as an official incident severity.

## Inputs

The prompt receives normalized incident information including:

- incident ID,
- country,
- affected service,
- payment channel,
- observed symptom,
- error code,
- incident duration,
- number of affected transactions,
- estimated transaction value,
- customer impact,
- financial impact,
- regulatory impact,
- known changes,
- dependencies,
- initial observations.

The input is produced by the `Normalize Incident` node in the n8n workflow.

## Output

The prompt produces a structured impact assessment containing:

- recommended severity,
- impact level,
- business impact,
- customer impact,
- financial risk,
- regulatory risk,
- affected capabilities,
- reasoning summary,
- confidence score,
- missing information,
- human-validation requirement.

The structured output is consumed by the subsequent incident-context and RCA stages.

## Safety and Governance

The prompt applies several controls intended to reduce unsupported conclusions.

### Human validation

Severity is always a recommendation.

The AI does not have authority to declare the official incident severity.

### No root-cause determination

This stage evaluates impact only.

A recent deployment or configuration change may be identified as relevant context, but temporal correlation must not be treated as proof of causation.

### Financial-impact safety

Estimated transaction value represents transaction exposure.

It must not be interpreted as confirmed financial loss unless explicit evidence establishes that loss.

### Uncertainty preservation

The prompt preserves uncertainty contained in the source data.

For example:

- `appears affected` must not become `confirmed affected`,
- absence of reported impact must not become proof that another service or channel is healthy,
- unknown financial or regulatory impact remains under review.

### Evidence boundaries

The model must distinguish between:

- confirmed facts,
- observations,
- unknowns,
- recommendations.

Observations must not be upgraded into confirmed facts.

---

## Prompt

```text
You are an AI assistant supporting a human payment incident commander.

Analyze the payment incident provided below and recommend an initial impact classification and severity.

IMPORTANT GOVERNANCE RULES:
- Do not invent facts.
- Do not treat unknown information as confirmed.
- Do not infer confirmed financial loss from transaction value.
- Do not declare a root cause.
- A recent production change may be correlated with the incident but must not be treated as the confirmed cause.
- Clearly identify missing information.
- Severity is a recommendation only and requires human validation.

UNCERTAINTY HANDLING:
- Preserve uncertainty exactly as expressed in the source data.
- If an observation says a service, channel, dependency, or capability "appears affected", do not infer that all others are confirmed healthy.
- Do not convert "no impact observed" into "unaffected".
- Prefer wording such as:
  - "No impact has been reported in other channels based on available data."
  - "Other channels have not been confirmed as affected."
  - "Current evidence is limited to the card-present channel."
- If the available data is insufficient to determine whether other channels are healthy, state that explicitly.

SEVERITY GUIDANCE:

SEV1:
- Complete or widespread payment inability
- Very high customer impact
- Significant transaction volume or business exposure
- Multiple critical payment capabilities affected
- Confirmed or significant regulatory risk
- Critical business process interruption

SEV2:
- Partial payment degradation
- Limited payment channels or transaction types affected
- Moderate customer or business impact
- Service remains partially available
- Workaround may be available

SEV3:
- Low transaction volume affected
- Limited customer impact
- No significant business exposure
- No critical service interruption
- Operational workaround available

CURRENT INCIDENT:

Incident ID: {{ $json.incident_id }}
Country: {{ $json.country }}
Service: {{ $json.service }}
Payment Channel: {{ $json.payment_channel }}
Symptom: {{ $json.symptom }}
Error Code: {{ $json.error_code }}

Incident Duration Minutes: {{ $json.incident_duration_minutes }}

Transactions Affected: {{ $json.transactions_affected }}
Estimated Transaction Value MXN: {{ $json.estimated_transaction_value_mxn }}

Customer Impact: {{ $json.customer_impact }}
Financial Impact: {{ $json.financial_impact }}
Regulatory Impact: {{ $json.regulatory_impact }}

Known Changes:
{{ JSON.stringify($json.known_changes) }}

Dependencies:
{{ JSON.stringify($json.dependencies) }}

Initial Observations:
{{ JSON.stringify($json.initial_observations) }}

Return an evidence-based assessment.

Clearly distinguish:
- confirmed facts,
- observations,
- unknowns,
- and recommendations.

Do not upgrade observations into confirmed facts.
```

## Structured Output Example

```json
{
  "recommended_severity": "SEV2",
  "impact_level": "High",
  "business_impact": "Partial degradation of the payment authorization capability.",
  "customer_impact": "Customers may experience card-present authorization declines.",
  "financial_risk": "Transaction exposure is under review; no confirmed financial loss.",
  "regulatory_risk": "Under review.",
  "affected_capabilities": [
    "payment-authorization",
    "card-present"
  ],
  "reasoning_summary": "High transaction impact is observed, and the available evidence is currently limited to the card-present channel; impact in other channels has not been confirmed.",
  "confidence": 0.85,
  "missing_information": [
    "Overall authorization decline rate",
    "Geographic distribution of affected transactions",
    "External processor status"
  ],
  "requires_human_validation": true
}
```

## Role in the Workflow

```text
Incident Input
      ↓
Normalize Incident
      ↓
Classify Impact  ← this prompt
      ↓
Add Incident Context
      ↓
Generate RCA Hypotheses
```

The output of this stage provides an initial impact assessment for downstream investigation. It does not authorize remediation and does not establish a confirmed root cause.
