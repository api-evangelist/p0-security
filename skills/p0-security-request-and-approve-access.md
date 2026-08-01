---
name: Request and approve just-in-time access with P0
description: >-
  Programmatically submit a P0 access command and then approve, deny, or revoke
  the resulting permission request via the P0 Just-in-Time Access API.
api: openapi/p0-security-jit-openapi.yml
operations:
  - submitCommand
  - approveRequest
  - denyRequest
  - revokeRequest
---

# Request and approve just-in-time access with P0

Use this skill to drive P0's just-in-time (JIT) access lifecycle from an
external system, bot, or CI/CD pipeline against `https://api.p0.app`.

## Authentication
Every call requires an HTTP **bearer (JWT)** token in the `Authorization`
header (`authentication/p0-security-authentication.yml`). All paths are scoped
to a P0 organization via `{orgId}`.

## Steps

1. **Submit an access command** — call `submitCommand`
   (`POST /o/{orgId}/command`) with a JSON body containing `scriptName` and the
   `argv` array (mirrors the `p0 request ...` CLI arguments). On success you get
   `{ ok: true, id, isPreexisting, isPersistent }`; keep `id` — it is the
   permission request identifier.

2. **Approve the request** — when a human/automated approver decides to grant,
   call `approveRequest`
   (`POST /o/{orgId}/permission-requests/{requestId}/approve`) with
   `requestId` = the `id` from step 1 and a body `{ "expirationLength": "2h" }`
   (e.g. `"30m"`, `"2h"`, `"1d"`). Access is provisioned for that duration.

3. **Deny the request** — to reject, call `denyRequest`
   (`POST /o/{orgId}/permission-requests/{requestId}/deny`). No body required.

4. **Revoke a granted permission** — to end access early, call `revokeRequest`
   (`POST /o/{orgId}/permission-requests/{requestId}/revoke`). No body required.

## Rules
- Handle `400` (malformed payload / invalid or expired request) and `401`
  (missing/invalid token) — see `errors/p0-security-problem-types.yml`.
- The API documents **no idempotency key**; do not blindly retry `submitCommand`
  on ambiguous failures — re-check state first.
- `approveRequest` grants privileged access; treat it as **safety-critical** and
  keep a human in the loop (see `agentic-access/p0-security-agentic-access.yml`).
