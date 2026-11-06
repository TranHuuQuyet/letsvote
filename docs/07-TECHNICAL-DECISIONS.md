# Technical Decisions — Let'sVote

This decision log records agreed assumptions. Update it when an open decision is resolved; do not silently change architecture.

## Accepted decisions

| ID | Decision | Rationale |
|---|---|---|
| ADR-001 | Multi-campaign platform, not a single-event clone. | Reusable product and domain modeling. |
| ADR-002 | Solo development with focused public MVP. | Realistic scope and demonstrable outcome. |
| ADR-003 | React + TypeScript frontend. | Modern web UI and existing frontend knowledge. |
| ADR-004 | C# / ASP.NET Core Web API backend. | Primary backend learning and internship target. |
| ADR-005 | PostgreSQL + EF Core. | Relational constraints and transactions support vote integrity. |
| ADR-006 | Modular Monolith with Clean Architecture dependency direction. | Clear boundaries without distributed-system overhead. |
| ADR-007 | Unique vote constraint on (CampaignId, EntryId, UserId). | Application checks alone cannot prevent concurrent duplicates. |
| ADR-008 | Voter identity comes from authenticated principal. | Prevents client identity spoofing. |
| ADR-009 | Votes are immutable in MVP. | Simpler integrity and auditability. |
| ADR-010 | Vote counts aggregate from Votes initially. | Avoids denormalized counter drift. |
| ADR-011 | OrganizationMembers is the single ownership/role source. | Avoids duplicate owner state. |
| ADR-012 | No microservices, Redis, or realtime infrastructure in MVP. | Avoids premature complexity. |

## Open decisions — resolve before related implementation

| ID | Question | Recommended default |
|---|---|---|
| OPEN-001 | Cookie or bearer authentication? | Secure HttpOnly cookies + CSRF when frontend/API can be same-site. |
| OPEN-002 | Organization slug uniqueness? | Globally unique. |
| OPEN-003 | Multiple categories per entry? | No; one optional category in MVP. |
| OPEN-004 | Normalize entry members? | Defer unless team/member display is required. |
| OPEN-005 | How are users added to an organization? | Owner adds existing registered account by email/username. |
| OPEN-006 | Campaign state update at StartAt/EndAt? | Vote API checks timestamps; background job may sync display state. |
| OPEN-007 | Result visibility modes? | Hidden, public during campaign, public after end. |
| OPEN-008 | Account deletion/anonymization policy? | Define before real public production use. |
| OPEN-009 | Hosting and object storage provider? | Choose based on current price, region, and simplicity at deployment. |
| OPEN-010 | Exact .NET/PostgreSQL versions? | Pin supported stable versions when implementation starts. |

## Deferred
Billing, anonymous voting, CAPTCHA/identity verification, Redis/cache, background job framework, realtime updates, microservices, advanced analytics, and exports.

## Change process
1. Record old assumption and new decision.
2. Explain why the change is needed.
3. Identify affected requirements, schema, API, tests, and deployment.
4. Update related documents in the same PR.
5. Add migration/compatibility plan if data contracts change.
