---
name: Generate a Knak email and sync it to a marketing platform
description: Use the Knak Enterprise API to pick a brand and theme, generate or create an email asset, review its rendered content, and push it to a connected marketing automation platform, tracking the sync to completion.
api: openapi/knak-enterprise-openapi-original.yml
operations:
  - GET /brands
  - GET /themes
  - POST /assets/generate
  - POST /assets
  - GET /assets/{asset_id}
  - GET /assets/{asset_id}/content
  - GET /assets/{asset_id}/preview
  - GET /available-platforms
  - POST /assets/{asset_id}/marketing-platform-syncs
  - GET /marketing-platform-syncs/{sync_history}
---

# Generate a Knak email and sync it to a marketing platform

The Knak Enterprise API has no `operationId` values in its published OpenAPI, so every step
below is identified by HTTP method and path against the base URL
`https://enterprise.knak.io/api/published/v1`.

## Before you start

- Authenticate every request with `Authorization: Bearer <token>`.
- Prefer an OAuth 2.0 access token. A non-expiring API key from
  <https://enterprise.knak.io/account/api-access> is acceptable for development only, and
  requires an API user holding a role with the "Can Manage API Tokens" permission.
- There is **no idempotency key**. Do not blindly retry `POST /assets`,
  `POST /assets/generate` or `POST /assets/{asset_id}/marketing-platform-syncs` — a retry
  creates a second asset or a second sync. On an ambiguous failure, re-list and reconcile
  before retrying.

## Steps

1. **Choose the brand.** `GET /brands` and select the brand the email belongs to. Brand is
   the top-level container that scopes assets, folders, modules and custom fieldsets.
2. **Choose a theme.** `GET /themes`, optionally filtering with `filter[name]`. Only
   published themes are usable. Use `GET /themes/{theme_id}` to confirm the choice.
3. **Create the asset.** Either:
   - `POST /assets/generate` to produce an AI-generated email from a prompt, then poll
     `GET /assets/{asset_id}` and read `attributes.ai_generation_status` until generation
     has finished; or
   - `POST /assets` to create the asset directly from a theme.
4. **Review the result.** `GET /assets/{asset_id}/content` returns the asset HTML;
   `GET /assets/{asset_id}/preview` returns a rendered visual preview. Show the preview to
   a human before syncing — a sync writes into a live marketing platform.
5. **Set any required metadata.** `GET /assets/{asset_id}/custom-fields` then
   `PATCH /assets/{asset_id}/custom-fields` to populate the custom fieldset values your
   organisation requires.
6. **Confirm the destination.** `GET /available-platforms` lists platforms available to the
   company. A platform appears only when it has HTML downloads enabled or an integration
   configured.
7. **Trigger the sync.** `POST /assets/{asset_id}/marketing-platform-syncs`.
8. **Track it.** `GET /marketing-platform-syncs/{sync_history}` for the sync record, and
   `GET /sync-statuses/{sync_status_id}` for its status. Prefer the
   `asset.sync_confirmation_responded` webhook over polling where you can receive one.

## Errors

Knak signals errors with the HTTP status code and does not use `application/problem+json`.
`400` malformed request, `401` missing or expired token, `403` the API user's role lacks the
permission, `404` unknown id or an asset outside the account's visible brands. See
`errors/knak-problem-types.yml`.

## Conventions

Lists paginate with `page` and `per_page` (default 10, maximum 100), filter with
`filter[field_name]`, and sort with `sort=updated_at|created_at`. See
`conventions/knak-conventions.yml`.
