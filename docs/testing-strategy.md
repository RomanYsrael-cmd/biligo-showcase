# Testing strategy

Biligo treats testing as part of the architecture. The private repository combines fast feedback for local rules with real PostgreSQL verification for the invariants that matter under contention, retry, rollback, and provider uncertainty.

## Verification layers

| Layer | What it protects | Representative coverage |
| --- | --- | --- |
| Unit and application tests | Small policies, lifecycle transitions, validation, and worker behavior | Payment-method policy, inventory rules, expiration decisions, provider adapter behavior |
| API integration tests | Authentication, authorization, CSRF, problem mapping, and application composition | Buyer, seller/store, catalog, inventory, checkout, payment, and webhook-facing boundaries |
| PostgreSQL integration tests | Persistence integrity and transaction behavior | Migrations, idempotency, duplicate evidence, inventory contention, checkout conversion, Ledger posting, and financial entitlement |
| Architecture tests | Module structure and ownership guardrails | Repository layout, allowed project relationships, and module write ownership |
| CI verification | Reproducible build and test entry point | Pinned SDK, locked dependency restore, release build, full test run, and test-result artifacts |

The implementation uses xUnit, the ASP.NET Core test host through WebApplicationFactory, and Testcontainers for .NET with PostgreSQL. Critical persistence behavior is not validated only against an in-memory substitute.

## Invariants tested in depth

### Duplicate and retry behavior

Duplicate command attempts, duplicate provider evidence, same-meaning replays, and conflicting reuse of an operation identity are expected behaviors. Tests verify that they converge safely or return a visible conflict rather than applying a second effect.

### Concurrency

Real PostgreSQL tests exercise competing inventory reservations, terminal reservation races, payment-intent contention, checkout conversion races, financial posting races, and competing entitlement movements. The goal is to prove the invariant at the database boundary, not just to demonstrate that a single-threaded service path works.

### Transaction atomicity

Failure-injection tests verify that a failed operation does not leave behind only part of its graph. Depending on the operation, the rollback boundary includes business state, reservations, idempotency state, Outbox facts, and financial evidence.

### Migrations and bootstrap

Fresh-database tests apply the migration path, check that the resulting model is current, and exercise the infrastructure required by integration tests. Migration validation is kept separate from ordinary unit coverage because a model can compile while a real database rejects the shape or behavior.

### Provider uncertainty

Payment-provider initiation, evidence intake, status lookup, expiration, reconciliation, and late/duplicate outcomes are tested as durable state transitions. A timeout is not automatically treated as a safe retry, and a browser or provider callback is not accepted without the appropriate verification boundary.

### Architecture enforcement

Structural tests make selected repository and module rules executable. They help detect accidental cross-module persistence writes and preserve the intended direction of dependencies as implementation grows.

## CI quality gate

The private repository has a GitHub Actions workflow that:

1. checks out the repository;
2. installs the SDK selected by repository metadata;
3. restores local tools;
4. restores dependencies in locked mode;
5. builds the solution in Release configuration; and
6. runs the backend test suite and preserves test-result artifacts.

The integration suite requires a container runtime because it starts PostgreSQL for real database coverage.

## Why no test count is published

This is an active implementation repository. A fixed number of tests would become stale quickly and would say less than the scenarios covered. The showcase therefore describes the verification model and the kinds of failure it exercises rather than presenting a vanity metric.

## Public-safety boundary

This document does not copy test source, fixtures, generated migration SQL, connection strings, provider payloads, private repository paths, logs, or customer data. The intent is to show how correctness is approached, not to provide a production test harness or reveal internal contracts.
