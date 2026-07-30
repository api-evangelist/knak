---
name: Manage Knak Send contacts and fields
description: Define contact fields and load, update and remove contacts in Knak Send, using single and bulk operations with the correct field-value semantics.
api: openapi/knak-send-openapi-original.yml
operations:
  - listFields
  - createField
  - listFieldValues
  - listContacts
  - createContact
  - getContact
  - deleteContact
  - getContactFieldValues
  - replaceContactFieldValues
  - bulkUpsertContacts
  - bulkDeleteContacts
---

# Manage Knak Send contacts and fields

Base URL `https://send.knak.io/api/public/v1`. This is the one Knak API that declares
`operationId` values, so the operations above are the real ids from the specification.

## Before you start

Authenticate with `Authorization: Bearer <token>`, where the token is a JWT generated
through the Enterprise API Access menu at
<https://enterprise.knak.io/account/api-access>.

## Steps

1. **Define the shape first.** `listFields` to see the field definitions that already
   exist; `createField` to add one. Create fields before loading contacts that reference
   them.
2. **Inspect a field's domain.** `listFieldValues` returns the values recorded for a field
   — useful for validating input against what Send already knows.
3. **Load contacts.**
   - One at a time: `createContact`.
   - In volume: `bulkUpsertContacts` (`POST /contacts/bulk`). Prefer this for imports. It
     is an upsert, which makes it naturally repeatable — the closest thing Knak offers to
     an idempotent write. Note that Knak documents **no** idempotency-key contract, so
     `createContact` remains unsafe to blind-retry.
4. **Read back.** `listContacts` to page through (`page` / `per_page`, maximum 100), or
   `getContact` for one record.
5. **Manage field values.** `getContactFieldValues` reads a contact's values.
   `replaceContactFieldValues` is a `PUT` — it **replaces** the whole set. Read the current
   values first and merge client-side, or you will silently drop values you did not send.
6. **Remove.** `deleteContact` for one; `bulkDeleteContacts`
   (`DELETE /contacts/bulk`) for many. Both are destructive with no API-level undo —
   confirm with a human and keep a record of what you removed.

## Errors

Errors are signalled by HTTP status code with no `application/problem+json` body:
`400` malformed, `401` bad token, `403` insufficient permission, `404` unknown id.
See `errors/knak-problem-types.yml`.
