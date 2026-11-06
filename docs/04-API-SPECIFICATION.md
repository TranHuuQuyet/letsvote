# API Specification — Let'sVote

- Status: Endpoint plan, not an implemented OpenAPI contract
- Base path: /api/v1
- Style: REST over HTTPS

## 1. Conventions
- Plural resource names and UUID identifiers.
- UTC ISO-8601 timestamps.
- Derive authenticated user ID from server-side principal.
- Never trust client-supplied UserId for vote ownership.
- Validate all input on server.
- Paginate collections.
- Return DTOs, not EF Core entities.
- Use consistent error envelope.

## 2. Authentication

| Method | Endpoint | Access |
|---|---|---|
| POST | /auth/register | Public |
| POST | /auth/login | Public |
| POST | /auth/logout | Authenticated |
| GET | /auth/me | Authenticated |
| PATCH | /users/me | Authenticated |
| POST | /auth/change-password | Authenticated |

Finalize cookie vs bearer transport based on deployment topology. For same-site hosting, prefer secure HttpOnly cookies with CSRF protection. Do not store long-lived tokens in localStorage.

## 3. Public discovery

| Method | Endpoint | Description |
|---|---|---|
| GET | /campaigns | Browse public campaigns |
| GET | /campaigns/{campaignId} | Campaign detail |
| GET | /campaigns/{campaignId}/categories | Categories |
| GET | /campaigns/{campaignId}/entries | Browse/filter published entries |
| GET | /campaigns/{campaignId}/entries/{entryId} | Entry detail |
| GET | /campaigns/{campaignId}/leaderboard | Leaderboard |

Support page/pageSize and relevant filters such as status, categoryId, and q. Expose only public fields.

## 4. Organization management

| Method | Endpoint | Description | Access |
|---|---|---|---|
| POST | /organizations | Create organization | Authenticated |
| GET | /organizations | Current user's organizations | Authenticated |
| GET | /organizations/{organizationId} | Organization detail | Member |
| PATCH | /organizations/{organizationId} | Update organization | Owner |
| GET | /organizations/{organizationId}/members | List members | Member |
| POST | /organizations/{organizationId}/members | Add member | Owner |
| PATCH | /organizations/{organizationId}/members/{userId} | Change role | Owner |
| DELETE | /organizations/{organizationId}/members/{userId} | Remove member | Owner |

## 5. Campaign management

| Method | Endpoint | Description |
|---|---|---|
| POST | /organizations/{organizationId}/campaigns | Create Draft |
| GET | /organizations/{organizationId}/campaigns | List organization campaigns |
| GET | /campaigns/{campaignId}/manage | Management detail |
| PATCH | /campaigns/{campaignId} | Update allowed fields |
| POST | /campaigns/{campaignId}/publish | Validate and publish |
| POST | /campaigns/{campaignId}/pause | Pause voting |
| POST | /campaigns/{campaignId}/resume | Resume if eligible |
| POST | /campaigns/{campaignId}/end | End early |
| POST | /campaigns/{campaignId}/archive | Archive ended campaign |

Use explicit lifecycle action endpoints rather than arbitrary status assignment through generic PATCH.

## 6. Categories and entries

| Method | Endpoint |
|---|---|
| POST | /campaigns/{campaignId}/categories |
| PATCH | /categories/{categoryId} |
| DELETE | /categories/{categoryId} |
| POST | /campaigns/{campaignId}/entries |
| PATCH | /entries/{entryId} |
| POST | /entries/{entryId}/publish |
| POST | /entries/{entryId}/unpublish |
| POST | /entries/{entryId}/media |

Deletion is allowed only when safe. Changes after votes exist must not silently change historical meaning.

## 7. Voting

Cast vote: POST /api/v1/campaigns/{campaignId}/entries/{entryId}/votes

Request body is empty JSON. No UserId is accepted; server derives voter from authenticated principal.

Success: 201 Created.

Example response:

~~~json
{
  "id": "7f8a0a31-8c61-4b9f-ae15-9db0c0e7d2f1",
  "campaignId": "campaign-uuid",
  "entryId": "entry-uuid",
  "createdAtUtc": "2026-11-06T10:30:00Z"
}
~~~

Duplicate: 409 Conflict, code VOTE_ALREADY_CAST.

Other outcomes:
- 401 Unauthorized: not authenticated.
- 403 Forbidden: account cannot vote.
- 404 Not Found: resource missing or not visible.
- 409 Conflict: campaign paused/ended or duplicate (distinct error codes).
- 422 Unprocessable Entity: validation failure if adopted consistently.

Vote history: GET /api/v1/campaigns/{campaignId}/my-votes. Return only current user's records.

## 8. Results

GET /api/v1/campaigns/{campaignId}/leaderboard

Optional query parameters: categoryId, page, pageSize, sort. Return rank, public entry summary, vote count, and category summary subject to visibility settings.

## 9. Error response

Recommended Problem Details-compatible envelope:

~~~json
{
  "type": "https://letsvote.example/problems/vote-already-cast",
  "title": "Vote already cast",
  "status": 409,
  "code": "VOTE_ALREADY_CAST",
  "detail": "You have already voted for this entry in this campaign.",
  "traceId": "00-..."
}
~~~

Never expose stack traces, SQL details, internal exception messages, or sensitive identifiers.

## 10. Implementation priorities
1. Authentication and current-user context.
2. Organization authorization.
3. Campaign/category/entry CRUD and lifecycle.
4. Voting with unique constraint and concurrency tests.
5. Leaderboard aggregation.
6. Administration and audit.
7. Upload and operational endpoints.

Generate and verify the executable OpenAPI contract during implementation. This document is a design baseline.
