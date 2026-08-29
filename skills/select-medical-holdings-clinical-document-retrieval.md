---
name: select-medical-clinical-document-retrieval
description: >-
  Find and download clinical notes and documents for an authorized patient from the Select Medical
  FHIR R4 API, including resolving DocumentReference attachments to their binary content.
api: Select Medical FHIR R4 API
generated: '2026-08-28'
method: generated
source: openapi/select-medical-holdings-fhir-r4-openapi.yml
operations:
  - searchPatient
  - searchDocumentReference
  - readDocumentReference
  - readBinary
  - searchDiagnosticReport
---

# Clinical document retrieval (Select Medical FHIR R4)

Read-only. Resolves patient documents from `DocumentReference` through to `Binary` content.

## Before you start

- Base URL: `https://epicproxy.et0948.epichosted.com/FhirProxy/api/FHIR/R4`
- SMART-on-FHIR bearer token required. Send `Accept: application/fhir+json`.

## Steps

1. **Resolve the patient id** — `searchPatient` (`GET /Patient`), or take it from the SMART launch
   context. Confirm a single match.

2. **List documents.** `searchDocumentReference` — `GET /DocumentReference?patient={id}`.
   Narrow with `category`, `type` or `date` rather than pulling everything.
   Conforms to `us-core-documentreference`.

3. **Inspect one document.** `readDocumentReference` — `GET /DocumentReference/{id}`.
   Read `content[].attachment`. You will find either:
   - `attachment.data` — base64 content inline; decode and stop, or
   - `attachment.url` — a reference to a `Binary` resource.

4. **Fetch attachment content.** When you have a `Binary` reference, call `readBinary`
   (`GET /Binary/{id}`) with the **same** bearer token. Honour `attachment.contentType`
   (commonly `application/pdf`, `text/html` or `text/plain`) — do not assume JSON.

5. **Lab and diagnostic reports.** `searchDiagnosticReport` —
   `GET /DiagnosticReport?patient={id}`. Declares `us-core-diagnosticreport-lab` and
   `us-core-diagnosticreport-note`. Its `presentedForm` resolves to `Binary` the same way.

## Pagination

Follow `Bundle.link[relation="next"]` verbatim; set page size with `_count`.

## Errors

`OperationOutcome` envelope. `401` refresh and retry, `403` insufficient scope (re-authorize),
`404` not found, `429` back off without a published `Retry-After`.

## Safety

- Documents are unredacted PHI and frequently contain far more than the question asked for.
  Extract only the fields needed to answer, and never persist attachment bytes outside the
  authorized session.
- Read-only. This skill calls no write operation.
