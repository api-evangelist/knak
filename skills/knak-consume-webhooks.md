---
name: Receive and verify Knak webhooks
description: Stand up an endpoint that receives Knak asset and translation lifecycle events, verifies the HMAC signature, and handles the retry policy correctly.
api: openapi/knak-enterprise-openapi-original.yml
operations:
  - webhook asset.created
  - webhook asset.sync_requested
  - webhook asset.sync_confirmation_responded
  - webhook asset.approval_status_updated
  - webhook asset.translation_requested
  - webhook translation_request.created
---

# Receive and verify Knak webhooks

Knak declares its event surface in the `webhooks` object of the Knak Enterprise OpenAPI 3.1
document. There is no AsyncAPI document. The six events are listed in
`asyncapi/knak-enterprise-webhooks.yml`.

## Configure

Create and manage webhook endpoints in the Knak Enterprise UI at
<https://enterprise.knak.io/account/webhooks>. Setup is documented at
<https://help.knak.io/en/articles/7950399-knak-custom-integration-setup>. Reveal the
signing secret on the selected webhook and store it as a secret in your environment.

## Verify every request

Knak signs the request body with a SHA-256 HMAC using that secret and sends the hexadecimal
digest in the `knak-signature` header.

1. Read the **raw** body. Do not let a JSON body parser touch it first — re-serializing
   changes the bytes and the digest will not match.
2. Compute `HMAC-SHA256(raw_body, secret)` and hex-encode it.
3. Compare against `knak-signature` using a **timing-safe** comparison.
4. Reject the request if it does not match.

## Handle the retry policy

Knak retries a webhook **up to three times, 60 seconds apart**, until a successful response
status is returned. That means:

- Return a success status as soon as you have durably accepted the event. Do the slow work
  afterwards, off the request path.
- Make your handler **idempotent on your side**. Knak sends no delivery id or idempotency
  key, so deduplicate on the event type plus the resource id in the payload.
- After three failures the event is not delivered again. If the event matters, reconcile by
  polling the corresponding Enterprise API resource.

## The events

- `asset.created` — an email or landing page was created.
- `asset.sync_requested` — a sync to a marketing platform was requested.
- `asset.sync_confirmation_responded` — the outcome of a requested sync came back. This is
  the event to use instead of polling `GET /marketing-platform-syncs/{sync_history}`.
- `asset.approval_status_updated` — an asset's approval status changed.
- `asset.translation_requested` — a translation was requested for an asset.
- `translation_request.created` — a translation request record was created.
