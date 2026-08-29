# Sample Data

This folder contains the fictional and sanitized data used to demonstrate the **Payments Incident Response Agent - MVP**.

The sample dataset allows the workflow to simulate a realistic payment incident without using real customer, merchant, cardholder, processor, or production information.

The data supports three main functions:

```text
Current Incident
      +
Historical Incident Context
      +
Ownership Reference
      ↓
AI-Assisted Incident Analysis
```

---

## Files

### `sample_incident.json`

Contains the fictional payment incident used as the primary MVP test scenario.

The incident represents an elevated card-authorization decline scenario affecting the card-present payment channel.

The sample includes information such as:

- incident ID,
- reporting and incident timestamps,
- country,
- affected service,
- payment channel,
- observed symptom,
- error code,
- affected transaction count,
- estimated transaction value,
- customer impact,
- financial impact,
- regulatory impact,
- known changes,
- dependencies,
- initial observations.

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

The estimated transaction value represents **transaction exposure**, not confirmed financial loss.

The workflow is explicitly instructed to preserve this distinction.

---

### `historical_incidents.md`

Contains fictional historical payment incidents used as contextual references during RCA hypothesis generation.

The MVP includes examples involving areas such as:

- payment-routing configuration,
- external processor connectivity,
- fraud-rule configuration.

Historical incidents help the AI identify possible **investigation patterns**.

They are not evidence that the current incident has the same root cause.

Conceptually:

```text
Historical similarity
        ↓
Possible investigation direction
        ↓
Evidence gathering
        ↓
Human validation
```

Not:

```text
Historical similarity
        ↓
Confirmed root cause
```

The RCA prompt explicitly prevents historical similarity from being treated as proof of causation or from automatically increasing confidence in a hypothesis.

---

### `ownership_matrix.csv`

Contains a simplified role-based ownership matrix for payment-platform failure domains.

Example structure:

```csv
service,issue_type,primary_owner_role,backup_owner_role
payment-authorization,application,Payments Platform Lead,Application Support Lead
payment-routing,configuration,Payment Operations Lead,Payments Platform Lead
processor-connectivity,integration,Integration Lead,Payment Operations Lead
fraud-engine,risk,Fraud Platform Lead,Risk Operations Lead
```

The matrix is used to demonstrate deterministic ownership mapping.

Instead of asking the LLM to invent an owner, the workflow maps suspected technical domains to predefined organizational roles.

Conceptually:

```text
RCA Hypothesis
      ↓
Suspected Domain
      ↓
Deterministic Ownership Mapping
      ↓
Primary Owner Role
```

Ownership is assigned to **roles, not individuals**.

The mapping identifies responsibility for investigation. It does not validate or confirm the associated RCA hypothesis.

---

## How the Data Is Used

At a simplified level, the sample data supports the following workflow stages:

```text
sample_incident.json
        ↓
Incident Input
        ↓
Normalize Incident
        ↓
Classify Impact
        ↓
Add Incident Context
        ↑
        │
historical_incidents.md
        │
ownership_matrix.csv
        ↓
Generate RCA Hypotheses
        ↓
Map RCA Ownership
        ↓
Generate Action Plan
        ↓
Generate Executive Brief
```

In the current MVP, context is intentionally lightweight and static.

The project does not require a production database, vector database, observability platform, or incident-management system to demonstrate the core reasoning and governance model.

---

## Data Safety

All data in this folder is intended for demonstration purposes and must remain fictional and sanitized.

The repository must not contain:

- real PAN or card numbers,
- CVV,
- PIN,
- personally identifiable information (PII),
- real customer information,
- real merchant-sensitive information,
- authentication credentials,
- API keys,
- access tokens,
- encryption keys,
- internal production URLs,
- private infrastructure identifiers,
- sensitive payment request or response payloads.

Transaction-level investigation in the workflow should use only sanitized, masked, tokenized, or otherwise non-sensitive diagnostic information.

---

## Important Data Interpretation Rules

The sample data intentionally includes uncertainty.

For example:

```text
financial_impact = "Under review"
regulatory_impact = "Under review"
```

These values must remain uncertain until evidence establishes otherwise.

Similarly:

```text
"Only the card-present payment channel appears affected"
```

must not be transformed into:

```text
"All other payment channels are confirmed healthy"
```

The absence of reported impact is not evidence that another service, channel, dependency, processor, or region is unaffected.

---

## Known Changes Are Not Root Causes

The sample incident includes a recent production release as contextual information.

A recent change may be relevant to the investigation, but temporal proximity does not establish causality.

The intended reasoning is:

```text
Recent Change
      +
Incident Observation
      ↓
Investigation Hypothesis
      ↓
Evidence Collection
      ↓
Human RCA Confirmation
```

The workflow must not automatically conclude that the release caused the incident.

---

## Static Context in the MVP

The current implementation uses static fictional context to keep the MVP understandable, reproducible, and safe.

A future production-oriented implementation could replace or complement these files with approved sources such as:

```text
Incident Management Platform
          +
Observability Data
          +
Approved Runbooks
          +
Historical Incident Repository
          +
Service Ownership Catalog
          ↓
Controlled Retrieval / RAG
          ↓
AI-Assisted Incident Analysis
```

Any production retrieval mechanism would require appropriate access controls, data classification, sanitization, auditability, and security review.

---

## Folder Structure

```text
sample-data/
├── README.md
├── sample_incident.json
├── ownership_matrix.csv
└── historical_incidents.md
```

---

## Purpose of the Sample Dataset

The purpose of this dataset is not to simulate every aspect of a production payment environment.

It provides enough controlled context to demonstrate:

- incident normalization,
- AI-assisted impact classification,
- context-enriched RCA hypothesis generation,
- deterministic ownership mapping,
- evidence-oriented investigation planning,
- executive incident summarization,
- human review and approval controls.

This allows the project to demonstrate the architecture and Responsible AI approach without exposing real operational or payment data.
