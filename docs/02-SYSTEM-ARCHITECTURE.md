# System Architecture — Let'sVote

- Status: Planned baseline
- Style: Modular Monolith with Clean Architecture dependency direction

## 1. Architecture decisions

Use a Modular Monolith. The first release is one deployable backend and one frontend. Modules are logical boundaries, not independently deployed services. This keeps boundaries and testability without distributed-system overhead.

## 2. High-level context

~~~mermaid
flowchart LR
  User[Visitor / Voter / Organizer / Admin] --> Web[React Web App]
  Web --> API[ASP.NET Core Web API]
  API --> DB[(PostgreSQL)]
  API --> Storage[Object Storage]
  API --> Logs[Structured Logs]
~~~

## 3. Technology stack

| Layer | Planned technology | Reason |
|---|---|---|
| Frontend | React + TypeScript | Component-based web experience |
| Backend | C# / ASP.NET Core Web API | Primary backend learning and CV focus |
| ORM | Entity Framework Core | Migrations, LINQ, relational mapping |
| Database | PostgreSQL | Constraints and transactions |
| Authentication | ASP.NET Core Identity | User lifecycle and password hashing |
| API docs | OpenAPI / Swagger | Contract visibility |
| Tests | xUnit and PostgreSQL integration tests | Verify rules and persistence |
| Local environment | Docker Compose | Reproducible setup |
| CI | GitHub Actions | Restore, build, test |
| Media | S3-compatible object storage | Keep binary files outside relational DB |

Pin supported stable versions when implementation begins. Avoid preview dependencies without a clear reason.

## 4. Suggested backend solution

~~~text
src/
  LetsVote.Api/
    Controllers/
    Middleware/
    Extensions/
    Contracts/
  LetsVote.Application/
    Abstractions/
    Common/
    Features/
      Authentication/
      Organizations/
      Campaigns/
      Categories/
      Entries/
      Voting/
      Results/
      Administration/
  LetsVote.Domain/
    Entities/
    Enums/
    Events/
    Exceptions/
    Rules/
  LetsVote.Infrastructure/
    Persistence/
      Configurations/
      Migrations/
    Identity/
    Storage/
    Time/
    Observability/
tests/
  LetsVote.UnitTests/
  LetsVote.IntegrationTests/
~~~

Dependency direction:

~~~text
Api ------------> Application
Infrastructure -> Application + Domain
Application ----> Domain
Domain ---------> no project dependencies
~~~

Domain contains business concepts, not EF Core/HTTP types. Application orchestrates use cases and defines interfaces. Infrastructure implements persistence and external concerns. API handles HTTP, authentication middleware, contracts, and dependency composition. Avoid abstractions without a concrete use case.

## 5. Logical modules

| Module | Responsibility |
|---|---|
| Identity | Registration, login, logout, profile, account status |
| Organizations | Organization lifecycle and membership |
| Campaigns | Campaign configuration and lifecycle |
| Categories | Campaign category management |
| Entries | Competition entries and media metadata |
| Voting | Eligibility, duplicate prevention, persistence |
| Results | Aggregated totals and leaderboard |
| Administration | Platform moderation |
| Audit | Important administrative action records |

Modules share PostgreSQL in MVP. Cross-module writes must be explicit and transactional where needed.

## 6. Vote request sequence

~~~mermaid
sequenceDiagram
  participant V as Voter
  participant API as ASP.NET Core API
  participant App as Voting Use Case
  participant DB as PostgreSQL
  V->>API: POST vote
  API->>API: Authenticate and validate
  API->>App: Execute with principal user ID
  App->>DB: Check campaign and entry eligibility
  App->>DB: Insert vote in transaction
  DB-->>App: Insert or unique constraint violation
  App-->>API: Created or conflict
  API-->>V: 201 Created / 409 Conflict
~~~

The database unique constraint must handle the race where duplicate requests pass application checks simultaneously.

## 7. Backend and storage boundaries
- PostgreSQL stores relational data and metadata.
- Object storage stores uploaded images; DB stores object key and metadata.
- Do not store uploaded binary files in relational tables for MVP.
- Validate size, allowed content type, and file signature where practical.
- A local filesystem adapter may be used in development behind an abstraction.

## 8. Deployment topology
- React static frontend on a static hosting provider.
- ASP.NET Core API as a container.
- PostgreSQL managed service or container on a suitable host.
- Object storage for images.
- HTTPS at public edge.
- GitHub Actions CI; add automated deployment after manual deployment is stable.

Choose low-cost maintainable infrastructure; document limitations rather than designing for hypothetical scale.

## 9. Observability
- Structured logs with correlation/trace identifiers.
- Liveness and readiness health endpoints.
- Never log passwords, tokens, or authentication secrets.
- Track API errors and database connectivity.
- Audit high-impact administrative actions.
- Add metrics/load tests after core correctness.

## 10. Architecture quality gates
- No direct DB access from controllers.
- No UserId accepted from request body for vote ownership.
- No business rule enforced only in React.
- No cross-organization access based only on possession of an ID.
- No duplicate prevention relying only on pre-insert query.
- No microservices, broker, or distributed cache in MVP without measured need.
