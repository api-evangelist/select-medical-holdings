---
name: select-medical-patient-clinical-summary
description: >-
  Assemble a read-only clinical summary for one authorized patient from the Select Medical FHIR R4
  API — demographics, problems, medications, allergies, vitals and recent encounters.
api: Select Medical FHIR R4 API
generated: '2026-08-28'
method: generated
source: openapi/select-medical-holdings-fhir-r4-openapi.yml
operations:
  - searchPatient
  - readPatient
  - searchCondition
  - searchMedicationRequest
  - searchAllergyIntolerance
  - searchObservation
  - searchEncounter
---

# Patient clinical summary (Select Medical FHIR R4)

Read-only. Every operation below is a GET declared in the server's own CapabilityStatement.

## Before you start

- Base URL: `https://epicproxy.et0948.epichosted.com/FhirProxy/api/FHIR/R4`
- You need a SMART-on-FHIR OAuth 2.0 access token. There is no anonymous access to clinical data.
- Send `Accept: application/fhir+json` and `Authorization: Bearer <token>`.
- Your token is scoped to a patient compartment. Do not attempt to read across patients.

## Steps

1. **Establish the patient id.**
   - In an EHR launch you already have it from the `launch` context — skip to step 2.
   - Standalone: call `searchPatient` (`GET /Patient`) with the identifiers you were given
     (`identifier`, `family`, `given`, `birthdate`). Confirm exactly one match before continuing.
     If more than one Bundle entry comes back, stop and escalate to a human — do not guess.
   - Optionally confirm with `readPatient` (`GET /Patient/{id}`).

2. **Problems.** `searchCondition` — `GET /Condition?patient={id}`.
   Conforms to `us-core-condition-problems-health-concerns` and
   `us-core-condition-encounter-diagnosis`.

3. **Medications.** `searchMedicationRequest` — `GET /MedicationRequest?patient={id}`.

4. **Allergies.** `searchAllergyIntolerance` — `GET /AllergyIntolerance?patient={id}`.

5. **Vitals and labs.** `searchObservation` — `GET /Observation?patient={id}&category=vital-signs`.
   Observation carries 22 declared US Core profiles here; use `category` to narrow rather than
   pulling the whole resource type.

6. **Recent encounters.** `searchEncounter` — `GET /Encounter?patient={id}&date=ge{YYYY-MM-DD}`.

## Pagination

Responses are FHIR searchset Bundles. Use `_count` to set page size, then follow
`Bundle.link` where `relation == "next"` **verbatim** — it is a cursor, not an offset. Never
construct the next page URL yourself.

## Errors

Errors come back as a FHIR `OperationOutcome`, not RFC 9457 problem+json. Parse
`issue[].severity`, `issue[].code` and `issue[].diagnostics`.

- `401` — token missing or expired. Refresh (the server supports `permission-offline`) and retry.
- `403` — the token lacks a scope for that resource type. Re-authorize; do not retry as-is.
- `404` — no such resource. Not retryable.
- `429` — throttled. No `Retry-After` is published, so back off exponentially.

## Safety

- This is PHI. Return only what the requesting user is authorized to see.
- Stay read-only in this skill. Do not call any POST or PUT operation — see the write-safety skill
  for why writes need a human.
