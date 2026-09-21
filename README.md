# Biligo

## Production-oriented marketplace engineering case study

Biligo is a Philippine-focused multi-vendor e-commerce platform designed around the hard parts of marketplace systems: multiple sellers and stores, inventory contention, checkout conversion, online payments, financial evidence, delivery serviceability, and operational reconciliation. The private system is being built as an intentionally bounded, transaction-aware platform rather than as a tutorial CRUD application.

This repository is a public portfolio/showcase repository. It contains sanitized product, architecture, domain, security, testing, and deployment documentation. The production source code and proprietary implementation remain private and are not included here.

## What this showcases

- Explicit domain ownership inside a modular monolith.
- Server-owned checkout and order conversion with immutable transaction evidence.
- Concurrency-safe inventory reservation and release/commit boundaries.
- Idempotent duplicate-sensitive commands with durable Inbox and transactional Outbox primitives.
- Provider integrations designed around durable intent, uncertain outcomes, and reconciliation.
- An append-only, balanced marketplace financial subledger rather than mutable balance fields.
- Central identity with scoped authorization and business-state validation.
- PostgreSQL-backed integration and concurrency verification.

The emphasis is on the engineering problems, boundaries, and principles. This repository deliberately does not publish production source, complete schemas, API contracts, provider payloads, or infrastructure details that would make the private system reconstructable.

## At a glance

| Area | Portfolio-level summary |
| --- | --- |
| Product | Multi-vendor marketplace for buyers, sellers, stores, and platform operations |
| Architecture | Modular monolith with separate API, worker, and reconciliation process roles |
| Transactional authority | PostgreSQL-backed relational persistence |
| Backend | .NET 10, C# 14, ASP.NET Core, REST/JSON, OpenAPI |
| Data access | Entity Framework Core and Npgsql, with reviewed PostgreSQL-specific operations where integrity requires it |
| Verification | xUnit, WebApplicationFactory, Testcontainers for .NET, architecture/structural tests, GitHub Actions |
| Public repository scope | Sanitized engineering case study; no production source code |

## High-level architecture

Client surfaces call an application/API boundary. The application coordinates domain modules such as identity, catalog, inventory, checkout, commerce, payments, ledger, and delivery. PostgreSQL remains the authoritative transactional store; durable work and integration evidence are processed by worker/reconciliation roles. External payment, messaging, mapping, storage, and future logistics providers are isolated behind adapters.

See [Architecture overview](docs/architecture-overview.md) and the [system-context diagram](diagrams/system-context.md).

## Domain overview

Biligo separates commercial, financial, and physical-delivery truth:

- Identity and access governs authentication, scoped permissions, and assurance.
- Seller and Store management owns marketplace participation and store operations.
- Catalog and Inventory manage sellable content and stock availability.
- Cart, Checkout, and Commerce turn validated buyer intent into durable purchase facts.
- Payments and the financial Ledger record money movement and evidence separately from fulfillment.
- Delivery serviceability and quoting provide checkout readiness without becoming shipment truth.
- Fulfillment, dispatch, Rider operations, COD, after-sales, and full settlement/payout workflows are designed boundaries with implementation status called out explicitly.

See [Domain overview](docs/domain-overview.md) for the conceptual relationships and current status.

## Technology stack

The current private repository verifies the backend stack shown below. Frontend and operations technologies are selected as the target platform baseline, but their complete production implementation is not represented as finished in this showcase.

| Layer | Current or selected baseline |
| --- | --- |
| Backend runtime | .NET 10 / C# 14 / ASP.NET Core |
| API | REST/JSON with generated OpenAPI |
| Database | PostgreSQL 18 |
| Persistence | Entity Framework Core 10 / Npgsql 10 |
| Async work | PostgreSQL transactional Outbox, durable work records, .NET workers |
| Frontend baseline | Next.js, React, TypeScript, Tailwind CSS |
| Rider baseline | Installable PWA with offline-pending behavior |
| Verification | xUnit, WebApplicationFactory, Testcontainers for .NET, architecture tests |
| Delivery and operations baseline | Containerized Linux deployment, controlled CI/CD, private object storage, backup/recovery, structured telemetry |

## Testing and quality

The private repository uses several complementary verification layers:

- unit tests for policies, lifecycle rules, and application behavior;
- API integration tests through the ASP.NET test host;
- real PostgreSQL integration tests through Testcontainers;
- architecture tests for repository structure and module write ownership;
- migration/bootstrap verification against a fresh PostgreSQL instance;
- concurrency, duplicate-delivery, idempotency, rollback, and provider-uncertainty scenarios;
- CI restore, release build, and test validation using pinned dependencies and locked restore.

The showcase intentionally avoids a fixed test-count claim because the suite is an active implementation artifact and changes as the platform grows.

## Security principles

The security baseline is defensive and server-enforced: one canonical identity, default-deny authorization, resource scope, revocable sessions, stronger assurance for sensitive actions, input and state validation, secret separation, minimized sensitive data, authenticated provider evidence, and durable auditability where appropriate.

See [Security overview](docs/security-overview.md). Credentials, tokens, connection strings, private addresses, internal URLs, exact session details, provider verification material, and security-sensitive implementation rules are intentionally omitted.

## Deployment and operations

The deployment design separates request-serving API processes from background and reconciliation workers, keeps authoritative state in durable services, and treats health/readiness, structured logs, metrics, traces, backups, restore testing, and operational alerts as production dependencies. CI/CD is designed around reproducible builds and controlled promotion.

The repository currently has backend health/readiness and CI foundations. Full frontend delivery, broad production deployment, telemetry operations, recovery exercises, and several provider activation steps remain project work rather than claims of completion.

See [Deployment overview](docs/deployment-overview.md).

## Current project status

### Implemented backend capabilities

- Backend foundation, health/readiness checks, and PostgreSQL migrations.
- Idempotency, Inbox deduplication, and transactional Outbox processing primitives.
- Central identity, session authentication, scoped authorization, and buyer profile/address ownership.
- Seller Account and Store core, Catalog lifecycle/moderation/publication reads, and Store-scoped inventory management.
- Buyer cart, checkout-session validation, serviceability and delivery-quote evidence, payment-method selection, and authenticated checkout conversion.
- Durable commerce Purchase/Seller Order construction with transaction-time snapshots.
- Payment-intent lifecycle, provider-operation evidence, confirmation/funding and reconciliation boundaries, including a hosted online-payment adapter boundary.
- Append-only financial Ledger posting substrate and Seller earning entitlement substrate.

### Explicitly incomplete or deferred

- Complete buyer discovery/search experience and frontend applications.
- Fulfillment, dispatch, Rider execution, and physical shipment operations.
- COD collection/remittance workflow.
- Returns, refunds, after-sales, disputes, and chargebacks orchestration.
- Full Seller settlement/holds/payout execution.
- Broad production deployment, telemetry operations, and recovery readiness.

This status is a portfolio snapshot, not a promise that every designed domain is production-complete.

## Documentation

- [Product overview](docs/product-overview.md)
- [Architecture overview](docs/architecture-overview.md)
- [Domain overview](docs/domain-overview.md)
- [Engineering decisions](docs/engineering-decisions.md)
- [Testing strategy](docs/testing-strategy.md)
- [Security overview](docs/security-overview.md)
- [Deployment overview](docs/deployment-overview.md)
- [System context diagram](diagrams/system-context.md)
- [Domain map](diagrams/domain-map.md)
- [Repository notice](NOTICE.md)

## Scope and confidentiality

This showcase explains engineering intent at a level suitable for a public portfolio. It does not provide a canonical ERD, migration SQL, complete API specification, endpoint inventory, event contracts, financial posting algorithms, fraud/abuse logic, authorization internals, credentials, production topology, customer data, or operational runbooks.

The production source code and proprietary implementation of Biligo are maintained separately.
