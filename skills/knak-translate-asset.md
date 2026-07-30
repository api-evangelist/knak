---
name: Run a Knak asset through a translation round trip
description: Create a translation request for a Knak asset, download the source file, upload the translated file, and track the request to completion using the Enterprise API and the translation webhooks.
api: openapi/knak-enterprise-openapi-original.yml
operations:
  - GET /translation-languages
  - GET /assets
  - POST /translation-requests
  - GET /translation-requests
  - GET /translation-requests/{id}
  - GET /translation-requests/{id}/download-source
  - POST /translation-requests/{id}/upload-translation
  - PATCH /translation-requests/{id}
---

# Run a Knak asset through a translation round trip

Base URL `https://enterprise.knak.io/api/published/v1`. The Enterprise spec declares no
`operationId` values, so steps are identified by method and path.

## Before you start

- `Authorization: Bearer <token>` on every request.
- This flow is the API side of Knak's Custom Translations integration. If you are building
  the translation service that Knak calls, you also implement the contract Knak documents
  at <https://developer.knak.com/integrations/setup-custom-translation-integration/>.
- No idempotency key exists. `POST /translation-requests` is not safe to blind-retry.

## Steps

1. **List supported languages.** `GET /translation-languages` and resolve the target
   language codes you intend to request.
2. **Identify the asset.** `GET /assets` with `filter[name]` or `filter[brand]`, or go
   straight to `GET /assets/{asset_id}` if you hold the id. Note `attributes.language_code`
   — that is the source language.
3. **Create the request.** `POST /translation-requests` with the asset and the target
   languages. Record the returned request id.
4. **Fetch the source.** `GET /translation-requests/{id}/download-source` returns the file
   to hand to your translation pipeline.
5. **Return the translation.** `POST /translation-requests/{id}/upload-translation` with
   the translated file.
6. **Advance the state.** `PATCH /translation-requests/{id}` to update the request, and
   `GET /translation-requests/{id}` to read its current state. `GET /translation-requests`
   lists outstanding work and paginates with `page` / `per_page`.

## Events

Two webhooks cover this flow: `asset.translation_requested` and
`translation_request.created`. Both POST JSON signed with a SHA-256 HMAC in the
`knak-signature` header, and are retried up to three times at 60 second intervals until a
successful status is returned. Verify the signature against the raw body with a timing-safe
comparison before acting. See `asyncapi/knak-enterprise-webhooks.yml`.

## Notes

A translated asset is a derivative: the resulting asset references its origin through
`attributes.parent_asset_id`. Use that to reconcile translations back to the source asset.
