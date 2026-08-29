---
name: select-medical-write-safety
description: >-
  Mandatory pre-flight checks before any agent write to the Select Medical FHIR R4 API — an API that
  declares no delete, no idempotency and no validate operation, so writes are unretryable and
  irreversible.
api: Select Medical FHIR R4 API
generated: '2026-08-28'
method: generated
source: >-
  fhir/select-medical-holdings-r4-capabilitystatement.json and
  conventions/select-medical-holdings-conventions.yml
operations:
  - searchPatient
  - searchAllergyIntolerance
  - searchCondition
---

# Write safety (Select Medical FHIR R4)

Read this before any `POST` or `PUT`. The constraints below are declared by the server itself in
its CapabilityStatement — they are facts about this endpoint, not caution in general.

## What the server declares

Across **all 59** resource types:

| Capability | Declared value |
|---|---|
| `conditionalCreate` | `false` |
| `conditionalUpdate` | `false` |
| `conditionalDelete` | `not-supported` |
| `updateCreate` | `false` |
| `readHistory` | `false` |
| delete interaction | declared on **0** resource types |
| `$validate` operation | not declared |

`create` is declared on 11 resource types; `update` on 7.

## What that means

1. **A timed-out create cannot be safely retried.** There is no idempotency key and no
   `If-None-Exist` conditional create, so the server cannot de-duplicate your retry. Retrying a
   create whose response you did not see risks a duplicate clinical record.
2. **A write cannot be undone.** No delete exists on any resource type. The only correction path is
   a compensating `PUT` — and only on the 7 resource types that support update. For the other 4
   creatable types there is no correction path through the API at all.
3. **A write cannot be rehearsed.** No `$validate` operation is declared, so there is no dry run.
4. **An update can silently clobber.** No `ETag`/`If-Match` optimistic concurrency is advertised.
5. **You cannot audit what changed.** `readHistory` is `false`, so prior versions are unreadable.

## Required procedure

Before any write:

1. **Get explicit human approval.** Every write here is irreversible and touches a clinical record.
   Do not write on inferred intent.
2. **Search first.** Call the matching search operation — `searchCondition`,
   `searchAllergyIntolerance` and so on — to confirm the record does not already exist. This is the
   only duplicate protection available, and it is yours to implement.
3. **Validate locally.** Check the payload against the declared US Core 6.1.0 profile for that
   resource type before sending, because the server offers no validate endpoint.
4. **Send once.** Treat the call as at-most-once.
5. **On timeout or 5xx, do NOT retry.** Re-run the search from step 2 to find out whether the write
   landed, then escalate to a human with what you found.

## After a mistaken write

Stop and escalate. Correcting a clinical record at Select Medical requires a human and an
out-of-band process at the health system — there is no API path for it.
