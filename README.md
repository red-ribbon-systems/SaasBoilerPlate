# SaasBoilerPlate

Lightweight starting point for a B2B multi-tenant SaaS on .NET 8. Focus: clean layering, clear extension points for tenant resolution, data isolation, billing, and background processing.

## Goals
- Opinionated project structure
- Clean Domain (no EF / infrastructure leakage)
- Central multi-tenancy seam (middleware + `ITenantContext`)
- Evolvable data isolation: single DB -> per-tenant DB / shard
- Support CQRS-style Application layer (commands / queries) later
- High testability (unit + integration isolation)

## Project Structure
- `Api` ASP.NET Core Web API (HTTP endpoints, auth, tenant resolution)
- `Application` Use cases, orchestration, validation, interfaces (MediatR later)
- `Domain` Entities, value objects, domain events, business rules (pure C#)
- `Infrastructure` EF Core persistence, external services (email, cache, messaging), DI wiring
- `Shared` Cross-cutting abstractions (`ITenantContext`, `IClock`, result / error primitives)
- `Tests.Unit` Fast tests for Domain & Application
- `Testing.Integration` End-to-end (HTTP + DB) tests (later: Testcontainers)

Optional (future): `Workers` (background jobs), `Migrations` (design-time), `Identity` (auth server).

## Dependency Flow
Api -> Application -> Domain
Infrastructure -> (Application + Domain)
Shared -> referenced where needed
Tests -> target specific layers only

## Planned Multitenancy Approach
1. Tenant resolution (host/path/header) middleware creating a scoped `TenantContext`.
2. Initial persistence: single database with `TenantId` discriminator + global query filters.
3. Upgrade path: dynamic connection selection for per-tenant DBs (migrations fan-out service).
4. Feature gating: cache of tenant plan + `IFeatureGate` interface.
5. Usage metering: domain events -> outbox -> background processor (future `Workers`).

## Getting Started
Restore & build:
```
dotnet restore
dotnet build
```
Run API:
```
dotnet run --project API/Api.csproj
```
Run unit tests:
```
dotnet test Tests.Unit/Tests.Unit.csproj
```

## Next Steps (Implementation Backlog)
1. Add `ITenantContext` + stub tenant resolution middleware.
2. Introduce first Domain aggregate (`Tenant`).
3. Add Application command/query abstractions (MediatR) + validation (FluentValidation).
4. Add EF Core to Infrastructure + initial migration (catalog + tenant tables).
5. Implement global query filter for `TenantId`.
6. Integration test harness (Testcontainers) for DB isolation.
7. Feature flags table + simple cache / `IFeatureGate` service.
8. Structured logging enrichment (TenantId, CorrelationId).

## Suggested NuGet Packages
- MediatR (application messaging)
- FluentValidation (request validation)
- EF Core + provider (e.g. `Microsoft.EntityFrameworkCore.SqlServer`)
- OpenIddict or external IdP client (auth)
- Scrutor (assembly scanning) optional
- Testcontainers (integration tests)
- Serilog + enrichers (structured logging)

## Testing Philosophy
- Domain: pure, deterministic unit tests (no mocks for value objects / entities).
- Application: handler tests with in-memory fakes first; later real DB for critical workflows.
- Integration: real HTTP + real database (container) verifying multi-tenant boundaries & filters.

## Roadmap
- [ ] Tenant model + provisioning endpoint
- [ ] Multi-tenant middleware skeleton
- [ ] Persistence + migrations
- [ ] Feature flags + plan enforcement
- [ ] Usage metering & outbox
- [ ] Background worker service
- [ ] Billing integration
- [ ] Observability (metrics, tracing, dashboards)

## Contributing Guidelines
- Keep Domain ignorant of infrastructure and UI concerns.
- Avoid static singletons; prefer DI & interfaces in `Shared`.
- Favor vertical slice growth in Application (feature folders) as it scales.
- Use domain events for cross-cutting reactions (email, billing) via outbox pattern.

## License
Add a license (MIT/Apache-2.0) before publishing.

---
Early focus: establish consistent patterns before adding complexity. Extend only when a real feature requires it.