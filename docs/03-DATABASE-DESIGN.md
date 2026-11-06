# Database Design — Let'sVote

- Status: Initial logical model
- Database: PostgreSQL
- ORM: Entity Framework Core, code-first

## 1. Conventions
- UUID primary keys.
- PostgreSQL timestamptz for UTC timestamps.
- Explicit EF Core entity configurations.
- Foreign keys and unique constraints for enforceable invariants.
- Commit migrations to source control.
- Restrictive delete behavior for records referenced by votes/audit history.
- Archive/status fields instead of destructive cascading deletes for important records.

## 2. Core entities

### Users
Use ASP.NET Core Identity user with required profile/account fields:
- Id (UUID), Email, normalized email, DisplayName.
- Status (Active, Suspended, Disabled).
- CreatedAtUtc, UpdatedAtUtc.

### Organizations
- Id, Name, Slug, Description (nullable), LogoObjectKey (nullable), Status.
- CreatedAtUtc, UpdatedAtUtc.
- Recommended: globally unique slug.

### OrganizationMembers
- OrganizationId, UserId, Role (Owner, Organizer), JoinedAtUtc.
- Unique key: (OrganizationId, UserId).
- **Single source of truth for ownership:** Owner is a role in OrganizationMembers. Do not also store Organization.OwnerId without an explicit synchronization design.

### Campaigns
- Id, OrganizationId, Title, Slug, Description, BannerObjectKey.
- StartAtUtc, EndAtUtc, Status (Draft, Scheduled, Active, Paused, Ended, Archived).
- ResultVisibility (Hidden, PublicDuringCampaign, PublicAfterEnd).
- CreatedByUserId, CreatedAtUtc, UpdatedAtUtc, PublishedAtUtc.
- Unique: (OrganizationId, Slug).

### Categories
- Id, CampaignId, Name, Description, DisplayOrder, CreatedAtUtc.
- Unique (CampaignId, Name) where appropriate.

### Entries
- Id, CampaignId, CategoryId (nullable), Name, Slug, ShortDescription, Description.
- CoverObjectKey, Status (Draft, Published, Unpublished).
- CreatedByUserId, CreatedAtUtc, UpdatedAtUtc, PublishedAtUtc.
- Unique (CampaignId, Slug).
- If CategoryId exists, it must refer to a category in the same campaign. Enforce through composite FK or equivalent DB constraint.

### EntryMembers (optional)
Normalize multiple named participants only if required. Otherwise defer and use simple descriptive text for MVP.

### Votes
- Id, CampaignId, EntryId, UserId, CreatedAtUtc.
- Critical unique constraint: (CampaignId, EntryId, UserId).
- CampaignId is included so a composite FK can ensure Vote.CampaignId matches Entry.CampaignId.

### AuditLogs
- Id, ActorUserId (nullable for system events), OrganizationId (nullable).
- Action, ResourceType, ResourceId, OccurredAtUtc.
- Metadata (JSONB, allowlisted non-sensitive fields), CorrelationId (nullable).
- Append-only from the application's perspective.

## 3. Relationship overview

~~~mermaid
erDiagram
  USERS ||--o{ ORGANIZATION_MEMBERS : joins
  ORGANIZATIONS ||--o{ ORGANIZATION_MEMBERS : has
  ORGANIZATIONS ||--o{ CAMPAIGNS : owns
  CAMPAIGNS ||--o{ CATEGORIES : contains
  CAMPAIGNS ||--o{ ENTRIES : contains
  CATEGORIES o|--o{ ENTRIES : classifies
  USERS ||--o{ VOTES : casts
  CAMPAIGNS ||--o{ VOTES : receives
  ENTRIES ||--o{ VOTES : receives
  USERS ||--o{ AUDIT_LOGS : performs
~~~

## 4. Critical constraints and indexes

| Table | Constraint/index | Purpose |
|---|---|---|
| OrganizationMembers | UNIQUE (OrganizationId, UserId) | One membership per user/org |
| Campaigns | UNIQUE (OrganizationId, Slug) | Stable campaign URL |
| Entries | UNIQUE (CampaignId, Slug) | Stable entry URL |
| Votes | UNIQUE (CampaignId, EntryId, UserId) | Concurrency-safe vote rule |
| Votes | INDEX (CampaignId, EntryId) | Aggregate votes |
| Votes | INDEX (CampaignId, UserId) | User voting history |
| Campaigns | INDEX (Status, StartAtUtc, EndAtUtc) | Discovery/lifecycle |
| Entries | INDEX (CampaignId, Status, CategoryId) | Entry filtering |
| AuditLogs | INDEX (OrganizationId, OccurredAtUtc) | Audit history |

## 5. Vote integrity
1. Resolve UserId from authenticated principal.
2. Load campaign and entry; verify relationship.
3. Verify account, campaign status/time, and entry publication.
4. Insert vote in transaction.
5. Let unique constraint arbitrate concurrent duplicates.
6. Translate unique-constraint violation to stable HTTP 409.

A SELECT-then-INSERT check alone is insufficient.

## 6. Vote counts
Votes are the source of truth. Compute totals with SQL aggregation for MVP. Do not maintain a manually incremented VoteCount until there is measured need and a reconciliation strategy.

## 7. Delete and retention
- Do not hard-delete campaigns/entries after votes exist.
- Prefer status changes and archival.
- Do not cascade-delete votes.
- Account deletion/anonymization policy is an open privacy/product decision.
- Define retention before public production use.

## 8. Open schema decisions
- Global or organization-scoped organization slugs (recommend global).
- One category or multiple categories per entry (recommend one optional category).
- Normalized EntryMembers or descriptive text.
- Account deletion/anonymization policy.
- Result visibility modes.
