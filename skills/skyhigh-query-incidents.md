---
name: Query Skyhigh SSE incidents
description: Authenticate to the Skyhigh Security SSE API and page through DLP/policy incidents for a time window.
api: openapi/skyhigh-incidents-openapi-original.yml
operations: [v1QueryIncidentGroups, v1QueryIncidentInformationKeys, v1QueryIncidents]
---

# Query Skyhigh Security SSE incidents

Use this to pull DLP / policy incidents from the Skyhigh Security Service Edge (SSE) platform.

## 1. Pick the regional host
Skyhigh pins each tenant to a data-residency host — use the matching base URL:
`https://www.myshn.net` (US), `https://www.myshn.eu` (EU), `https://www.myshn.ca` (CA), `https://www.govshn.net` (Fed/Gov).
The API base path is `/shnapi/rest/external/api`.

## 2. Get an IAM access token
`POST /shnapi/rest/external/api/v1/token?grant_type=password&token_type=iam`
with HTTP **Basic Auth** (your Skyhigh CASB username/password). Some deployments
also require a `BPS-TENANT-ID` header. Use the returned token as
`Authorization: Bearer <access-token>` on every subsequent call, with
`Content-Type: application/json`.

## 3. Discover filter surface (optional)
- `v1QueryIncidentGroups` — list incident types and categories available for filtering.
- `v1QueryIncidentInformationKeys` — list the `Incident.information` keys for a type.

## 4. Query incidents
Call `v1QueryIncidents` with a `Criteria` body. Set a required `startTime` (window lower bound);
optional `endTime`, `actorIds`, `serviceNames`, `incidentCriteria`. Use the `limit` query
param (default 100, max 10000 — values above max are silently capped).

## 5. Page forward
The response is an `IncidentResponse` wrapping `incidents[]` plus `responseInfo`.
Feed `responseInfo.nextStartTime` back as the next request's `startTime` to continue.
Check `responseInfo.error` for warnings.

## Errors
Errors return a custom envelope `{code, message, target, details[]}` (not RFC 9457).
Handle `401` (token invalid/expired — re-auth), `429` (rate limited — back off and retry),
`400` (invalid Criteria fields — see `target`).
