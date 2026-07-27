---
name: Discover Go1 content and enroll a learner
description: Search the Go1 learning library, then create and confirm an enrollment for a learner.
api: openapi/go1-openapi.yml
operations:
  - learning-objects-v3-search
  - enrolmentCreateController_postV3
  - enrolmentLoadController_getSlimEnrollment
---

# Discover Go1 content and enroll a learner

Use the Go1 API to find a learning object and enroll a user in it.

## Auth
- Obtain an OAuth 2.0 token: `POST https://auth.go1.com/oauth/token` with `grant_type=client_credentials`, `client_id`, `client_secret`. Tokens are Bearer and valid for 12 hours.
- Send every request to `https://gateway.go1.com` with headers `Authorization: Bearer {token}` and `Api-Version: 2025-01-01`.
- Required scopes: `lo.read` to search, `enrollment.write` to enroll, `enrollment.read` to confirm.

## Steps
1. **Search content** — `GET /learning-objects` (`learning-objects-v3-search`). Filter with `keyword`, `language[]`, `type[]`, `providers[]`. Paginate with `limit` (default 20, max 50) and `offset`; for deep scans set `use_scroll=true` and follow `scroll_id` (limit up to 5000). Read `total` and pick the target learning object `id`.
2. **Create enrollment** — `POST /enrollments` (`enrolmentCreateController_postV3`) with the learner and the chosen `lo_id`. A `409` with `error_code: enrollment_exists` means the learner is already enrolled — treat as success and load the existing record.
3. **Confirm** — `GET /enrollments/{id}` (`enrolmentLoadController_getSlimEnrollment`) to verify state, or `GET /enrollments` (`enrolmentSearchController_get`) to list by user/portal.

## Rules
- Errors use `application/json` `{ref, error_code, message}` (not RFC 9457). Retry `500`/`503` with backoff; do not retry `400`/`403`.
- No `Idempotency-Key` is supported — guard duplicate creates with the `409 enrollment_exists` check above.
- The token's portal context determines which portal the enrollment is created against.
