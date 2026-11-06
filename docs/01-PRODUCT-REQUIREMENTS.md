# Product Requirements — Let'sVote

- Status: Draft baseline
- Product version: MVP v1
- Last updated: 2026-11-06

## 1. Product overview

Let'sVote is a multi-campaign online voting and competition management platform. Organizations create competitions, organize entries, collect authenticated votes, and publish results.

**Tagline:** Create. Compete. Vote.

### Problem
Organizations often coordinate competitions through disconnected forms, spreadsheets, social posts, and manual counting. This makes setup difficult and results harder to audit.

### Goal
Provide one platform where an organization configures a campaign, publishes entries, collects community votes, and reviews trustworthy results.

### Target users
- Schools, clubs, student communities, event organizers, and small organizations.
- Organizers who need to manage competitions without building a custom website.
- Participants and voters who need a simple voting experience.

## 2. Product principles
1. Integrity first: enforce voting rules in backend and database.
2. Clear ownership: isolate organization data by membership.
3. Simple MVP: complete the campaign-to-vote journey before SaaS complexity.
4. Transparent results: explain what is counted and when voting is open.
5. Auditable operations: record important administrative actions.
6. Accessible UX: responsive and usable on mobile and desktop.

## 3. Actors and permissions

| Actor | Scope | Main capabilities |
|---|---|---|
| Visitor | Public | Browse public campaigns, entries, visible results |
| Voter | Own account | Sign in and cast eligible votes |
| Organization Owner | Organization | Manage organization, members, campaigns, entries |
| Organizer | Assigned organization | Manage permitted campaigns and entries |
| Platform Admin | Entire platform | Moderate users, organizations, campaigns, audit records |

MVP organization roles: Owner and Organizer. Platform Admin is a separate platform-level permission.

## 4. MVP scope

### Public experience
- Landing page and campaign discovery.
- Campaign detail with status, schedule, description, categories.
- Entry listing with category filtering and search.
- Entry detail.
- Authenticated voting.
- Leaderboard/results subject to visibility settings.
- Registration, login, logout, basic profile.

### Organization workspace
- Create/update organization and view overview.
- Manage members and roles.
- Create, edit, publish, pause, resume, end, archive campaigns.
- Manage categories.
- Create, edit, publish, unpublish entries.
- View campaign totals and leaderboard.

### Platform administration
- View organizations, users, campaigns.
- Suspend/reactivate accounts when required.
- Review audit events.

### Explicitly out of scope
- Payments, subscriptions, billing.
- Anonymous voting.
- Vote editing, cancellation, or transfer.
- Real-time WebSocket updates.
- Microservices/event streaming.
- Redis before measurement demonstrates need.
- Native mobile apps.
- Advanced identity verification or claims that one account equals one human.
- Complex permission matrices.
- Email invitations in the first release.

## 5. Core workflows

### Organization workflow
1. User registers and signs in.
2. User creates an organization; creator becomes Owner.
3. Owner creates a Draft campaign.
4. Owner/Organizer creates categories and entries.
5. Eligible entries are published.
6. Campaign is validated and published.
7. Campaign becomes Scheduled or Active depending on StartAt.
8. Voters cast votes during the eligible window.
9. At EndAt, voting is rejected immediately and results become final.
10. Organization may archive the campaign.

### Voter workflow
1. Visitor browses active/upcoming campaigns.
2. Visitor explores categories and entries.
3. Voter signs in.
4. Voter selects Vote on an eligible entry.
5. Backend validates campaign, entry, account, and duplicate rule.
6. Vote is persisted and counted.
7. Voter sees confirmation and visible results.

## 6. Functional requirements

### Identity
- FR-AUTH-01: Register with unique email and password.
- FR-AUTH-02: Login and logout.
- FR-AUTH-03: View/update permitted profile fields.
- FR-AUTH-04: Change password.
- FR-AUTH-05: Disabled accounts cannot authenticate or vote.

### Organizations
- FR-ORG-01: Authenticated user can create an organization.
- FR-ORG-02: Creator receives Owner membership.
- FR-ORG-03: Owner can update organization details.
- FR-ORG-04: Owner can add/remove members and assign supported roles.
- FR-ORG-05: Private management data is inaccessible to non-members.

### Campaigns and entries
- FR-CAM-01: Owner/Organizer can create and edit Draft campaigns.
- FR-CAM-02: Campaign includes title, slug, description, schedule, status, result visibility, organization.
- FR-CAM-03: Campaign contains categories and entries.
- FR-CAM-04: Entry belongs to exactly one campaign and optionally one category in that campaign.
- FR-CAM-05: Only Published entries are eligible for voting.
- FR-CAM-06: Publish requires valid configuration and at least one published entry.
- FR-CAM-07: Lifecycle transitions follow section 7.

### Voting and results
- FR-VOTE-01: Voting requires authenticated active account.
- FR-VOTE-02: A voter can vote at most once for an entry in a campaign.
- FR-VOTE-03: A voter may vote for multiple different entries in one campaign.
- FR-VOTE-04: Campaign must be Active and now must satisfy StartAt <= now < EndAt.
- FR-VOTE-05: Entry must be Published and belong to the campaign.
- FR-VOTE-06: Votes are immutable in MVP.
- FR-VOTE-07: Leaderboard totals derive from persisted votes.
- FR-VOTE-08: Duplicate attempts return conflict and create no extra record.

### Audit/admin
- FR-ADMIN-01: Audit publish/pause/resume, role changes, account suspension, archival.
- FR-ADMIN-02: Platform Admin can inspect platform resources.
- FR-ADMIN-03: Organization users cannot access another organization's private management data.

## 7. Business rules

| ID | Rule |
|---|---|
| BR-01 | One user may cast one vote per entry per campaign. |
| BR-02 | User may vote for different entries in the same campaign. |
| BR-03 | Authentication is required. |
| BR-04 | Account must be active. |
| BR-05 | Campaign must be Active and StartAt <= current UTC < EndAt. |
| BR-06 | Entry must be Published and belong to the target campaign. |
| BR-07 | Vote is immutable in MVP. |
| BR-08 | Backend validation is mandatory; frontend state is not authoritative. |
| BR-09 | Database uniqueness is the final duplicate-vote guard. |
| BR-10 | Persist timestamps in UTC. |
| BR-11 | Organization actions require membership and appropriate role. |
| BR-12 | Votes are the source of truth for results. |
| BR-13 | Configuration changes after votes exist must not silently invalidate historical votes. |
| BR-14 | Campaign cannot resume after EndAt. |

### Campaign lifecycle
- Draft → Scheduled (published with future StartAt).
- Draft → Active (published within voting window).
- Scheduled → Active (when StartAt is reached).
- Scheduled/Active → Paused.
- Paused → Active only while before EndAt.
- Scheduled/Active/Paused → Ended.
- Ended → Archived.

Vote API checks timestamps and status on every request. A background job may synchronize displayed status, but correctness must not depend on it. Ended campaigns cannot reopen in MVP.

## 8. Non-functional requirements
- Security: authentication, authorization, validation, secure password hashing, secret management.
- Integrity: no duplicate votes under concurrent requests.
- Consistency: enforce campaign-entry-vote relationships in DB.
- Performance: keep demo queries responsive and measure before optimizing.
- Reliability: health endpoints and structured logs.
- Maintainability: modular boundaries and documented decisions.
- Testability: automated tests for critical rules.
- Usability: responsive layouts and clear states.
- Deployability: reproducible local setup and Docker.
- Observability: diagnose errors and important admin actions.

## 9. MVP acceptance criteria
- Organization can be created and managed.
- Owner can create campaign, categories, entries, and publish them.
- Visitor can discover campaigns and entries.
- Voter can register/sign in and vote under defined rules.
- Concurrent duplicate votes cannot create more than one vote record.
- Votes are rejected outside the allowed window or in paused/ended campaigns.
- Cross-organization access is prevented.
- Leaderboard matches persisted votes.
- Critical rules have automated tests.
- Public deployment, setup guide, and demo credentials exist.
- Docs distinguish implemented features from planned features.

## 10. Demo success measures
- Complete end-to-end campaign setup and voting.
- Zero duplicate vote records in concurrency tests.
- No cross-organization data leakage in authorization tests.
- Correct results for a known seeded dataset.
- Reproducible deployment and clear demo instructions.
