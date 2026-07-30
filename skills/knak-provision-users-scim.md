---
name: Provision and deprovision Knak users
description: Manage Knak user lifecycle either through the SCIM 2.0 API driven by an identity provider, or through the Enterprise API user endpoints, including safe deprovisioning.
api: openapi/knak-scim-openapi-original.yml
operations:
  - GET /Users
  - POST /Users
  - GET /Users/{user_id}
  - PUT /Users/{user_id}
  - PATCH /Users/{user_id}
  - GET /users
  - GET /users/{user_id}
  - DELETE /users/{user_id}
---

# Provision and deprovision Knak users

Two surfaces manage the same people. Pick one and stay on it.

- **SCIM 2.0** — `https://enterprise.knak.io/scim/v2`. Use this when an identity provider
  is the system of record. Neither this spec nor the Enterprise spec declares
  `operationId` values, so steps are identified by method and path.
- **Enterprise API** — `https://enterprise.knak.io/api/published/v1`. Use this for
  read-and-audit work and for direct removal.

Authenticate both with `Authorization: Bearer <token>`.

## Identity-provider driven provisioning (preferred)

1. **Reconcile.** `GET /Users` on the SCIM base to list existing users. SCIM list responses
   follow SCIM 2.0 conventions, not the Enterprise `page` / `per_page` convention.
2. **Look up before creating.** `GET /Users/{user_id}` to confirm whether the subject
   already exists. Creating a duplicate is not reversible through a retry — there is no
   idempotency key on this API.
3. **Create.** `POST /Users` with the SCIM user resource.
4. **Update.** `PUT /Users/{user_id}` to replace the resource, or `PATCH /Users/{user_id}`
   to apply a partial change. Prefer `PATCH` for attribute-level changes such as
   activation and deactivation so you do not clobber attributes the IdP does not manage.

## Audit and removal through the Enterprise API

1. `GET /users` to list, with `page` / `per_page`, `filter[field_name]` and
   `sort=updated_at|created_at`. Each user carries `id`, `name`, `email` and `roles`.
2. `GET /users/{user_id}` for a single record.
3. `DELETE /users/{user_id}` to remove a user. This is destructive and has no undo through
   the API. Confirm with a human first, and check what the user owns —
   `Asset.attributes.creator_id` references users — before deleting.

## Errors

`401` invalid or expired bearer token. `403` the API user's role lacks the permission;
API token management requires the "Can Manage API Tokens" permission. `404` unknown user id.
See `errors/knak-problem-types.yml`.

## Related

SSO is delivered over SAML 2.0 rather than OpenID Connect. API user setup is documented at
<https://developer.knak.com/api-essentials/api-user-setup/>.
