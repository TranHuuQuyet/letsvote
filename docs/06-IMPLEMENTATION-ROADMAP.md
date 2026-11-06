# Implementation Roadmap — Let'sVote

- Delivery: solo developer
- Target: publicly accessible portfolio MVP
- Estimate: 8 focused weeks; adjust to actual learning pace
- Keep the application runnable at every milestone.

## 1. Milestones

### M0 — Product and repository foundation
- [ ] Confirm scope and open decisions.
- [ ] Create repository structure and conventions.
- [ ] Create .NET solution/projects and React + TypeScript app.
- [ ] Add PostgreSQL local environment using Docker Compose.
- [ ] Add example configuration and setup README.
- [ ] Add CI restore/build/test baseline.
Exit: frontend, API, and database run locally using documented steps.

### M1 — Identity and access
- [ ] Configure ASP.NET Core Identity.
- [ ] Register/login/logout/current-user flows.
- [ ] Enforce account status.
- [ ] Finalize authentication transport and CSRF strategy.
- [ ] Add authorization policies and tests.
Exit: account creation and protected endpoints work safely.

### M2 — Organizations and membership
- [ ] Organization persistence and CRUD.
- [ ] Creator becomes Owner through OrganizationMembers.
- [ ] Add/remove members and assign roles.
- [ ] Resource-based authorization.
- [ ] Cross-organization access tests.
Exit: organization data is isolated and roles work.

### M3 — Campaigns, categories, entries
- [ ] Campaign CRUD and lifecycle.
- [ ] Category management.
- [ ] Entry CRUD and publication.
- [ ] Public campaign/entry discovery.
- [ ] Image storage abstraction and validation.
- [ ] Validation and integration tests.
Exit: organizer publishes a campaign and public users browse it.

### M4 — Voting engine
- [ ] Vote entity and migration.
- [ ] Unique database constraint.
- [ ] Vote eligibility use case.
- [ ] Vote endpoint and stable error codes.
- [ ] Current user's vote history.
- [ ] Concurrent duplicate-vote integration test.
- [ ] Tests for time, state, publication, and cross-campaign cases.
Exit: concurrent duplicate requests cannot create multiple votes.

### M5 — Results and dashboards
- [ ] Leaderboard aggregation.
- [ ] Result visibility rules.
- [ ] Public results.
- [ ] Basic organization campaign overview.
- [ ] Audit important actions.
Exit: totals match database votes and visibility is enforced.

### M6 — Quality, security, usability
- [ ] Responsive public and organization UI.
- [ ] Loading, empty, validation, and error states.
- [ ] Unit/integration tests for critical rules.
- [ ] Rate limits and security headers.
- [ ] Structured logs and health endpoints.
- [ ] Basic load test for listing, leaderboard, voting.
- [ ] Accessibility/usability pass.
Exit: critical flows are predictable and diagnosable.

### M7 — Deployment and portfolio readiness
- [ ] Production Docker build.
- [ ] Deploy frontend/API and PostgreSQL.
- [ ] Configure object storage, HTTPS, and secrets.
- [ ] CI and deployment instructions.
- [ ] Seed/demo data and demo accounts.
- [ ] README architecture diagram and screenshots.
- [ ] Short demo video and known limitations.
Exit: reviewer can open demo and understand the architecture.

## 2. Suggested weekly sequence

| Week | Focus | Outcome |
|---|---|---|
| 1 | M0 | Local skeleton and CI |
| 2 | M1 | Authentication and policies |
| 3 | M2 | Organization and membership |
| 4 | M3 part 1 | Campaigns and lifecycle |
| 5 | M3 part 2 | Categories, entries, public browse |
| 6 | M4 | Vote integrity and concurrency |
| 7 | M5 + M6 | Results, audit, security, tests |
| 8 | M7 | Deployment, docs, demo |

This is an estimate. If .NET learning takes longer, reduce optional scope rather than skipping authorization or vote integrity.

## 3. Prioritized backlog

### P0 — MVP required
- [ ] Solution setup and local developer experience.
- [ ] Identity and session management.
- [ ] Organization and membership authorization.
- [ ] Campaign lifecycle.
- [ ] Category and entry management.
- [ ] Public browsing.
- [ ] Vote eligibility and duplicate protection.
- [ ] Leaderboard/results.
- [ ] Critical automated tests.
- [ ] Deployment and demo docs.

### P1 — After P0
- [ ] Organization dashboard metrics.
- [ ] Audit viewer.
- [ ] Object storage uploads.
- [ ] Search/pagination improvements.
- [ ] Basic admin moderation UI.
- [ ] Load test and performance report.

### P2 — Deferred
- [ ] Email invitations and notifications.
- [ ] Real-time leaderboard.
- [ ] Redis caching.
- [ ] Billing/subscriptions.
- [ ] Advanced fraud detection.
- [ ] Multi-tenant plan limits.
- [ ] Microservices/event-driven architecture.

## 4. Definition of Done
- [ ] Business rule and authorization are defined.
- [ ] Server-side validation exists.
- [ ] Persistence constraints/migrations are included.
- [ ] Success and failure responses are documented.
- [ ] Unit tests cover domain/application behavior.
- [ ] Integration tests cover persistence and authorization where relevant.
- [ ] Frontend handles loading, empty, success, and error states.
- [ ] Logs do not expose secrets.
- [ ] Documentation is updated.
- [ ] CI passes.

## 5. Portfolio evidence
Capture actual evidence as milestones complete: architecture diagram, ERD, OpenAPI page, concurrency test report, CI result, deployment topology, screenshots, and demo video. Only claim features in the CV that are implemented and demonstrable.
