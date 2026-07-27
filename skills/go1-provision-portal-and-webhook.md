---
name: Provision a Go1 portal and subscribe to events
description: Create a customer portal, then register a webhook to receive Go1 events and verify its signature.
api: openapi/go1-openapi.yml
operations:
  - createPortal
  - WebhookController_createPortalWebhookConfiguration
  - WebhookController_updatePortalWebhookConfiguration
---

# Provision a Go1 portal and subscribe to events

Onboard a new customer portal and start receiving Go1 events over webhooks.

## Auth
- OAuth 2.0 client_credentials token from `https://auth.go1.com/oauth/token` (see the discover-and-enroll skill).
- Base `https://gateway.go1.com`, header `Api-Version: 2025-01-01`.
- Scopes: `portal.write` to create a portal, `webhook.write` to manage webhooks.

## Steps
1. **Create portal** — `POST /portals` (`createPortal`). Capture the new portal `id`. Use the token whose context matches the parent/master account.
2. **Subscribe to events** — `POST /webhooks` (`WebhookController_createPortalWebhookConfiguration`) with `name`, `url` (your HTTPS endpoint) and `event_types` (e.g. `["enrollment.complete"]`). Available events: `content.add`, `content.remove`, `content.decommission`, `content.decommission.pending`, `content.library.add`, `content.library.remove`, `content.library.update`, `content.library.decommission`, `content.library.sync.request`, `enrollment.create`, `enrollment.complete`, `enrollment.delete`.
3. **Verify deliveries** — each delivery carries a `go1-signature` header (HMAC, version `v1`). Verify it with `@go1/webhook-verifier-js` using your shared secret and a 60-second timestamp tolerance before trusting the payload.
4. **Manage** — `PATCH /webhooks/{id}` (`WebhookController_updatePortalWebhookConfiguration`) to set `status: inactive` or update the config; `GET /webhooks` to read the current portal configuration.

## Rules
- Optionally secure delivery with `auth_type: oauth2` + an `oauth2` object so Go1 fetches a bearer token from your auth server per delivery.
- Event objects are `{id, event_type, webhook_version, sent, attempt_number, url, webhook_id, data}`; `webhook_version` is `3.0.0`.
- Make test webhooks `inactive` when no longer needed.
