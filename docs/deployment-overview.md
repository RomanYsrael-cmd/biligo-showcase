# Deployment overview

Biligo's deployment design is intentionally simple at the application level and disciplined about durable state. The target is a containerized Linux deployment in which API, background-worker, and reconciliation roles can scale or restart independently while sharing the same domain rules and authoritative persistence.

## Logical runtime

| Component | Responsibility | Status |
| --- | --- | --- |
| API process | Request handling, authentication, authorization, queries, and domain commands | Backend foundation implemented |
| Background worker | Durable post-commit work, provider follow-up, and safe retry processing | Worker foundation and payment-related work exist; broader workflows remain |
| Reconciliation worker | Provider, payment, and later financial exception follow-up | Boundary and selected payment reconciliation are implemented; full operations remain |
| PostgreSQL | Authoritative transactional state, integrity, and durable work evidence | Implemented backend authority |
| Object storage | Product media and private evidence blobs | Selected architecture baseline; application surfaces remain incomplete |
| Cache or ephemeral coordination | Non-authoritative acceleration or rate limiting | Optional baseline, never transaction truth |
| Telemetry and alerting | Health, logs, metrics, traces, business-integrity signals, and operational alerts | Health/readiness and CI foundations exist; broad production operations remain |

These are logical roles rather than a public infrastructure diagram. Exact hosts, networks, ingress configuration, firewall rules, storage namespaces, and credentials are not part of the showcase.

## Release and CI/CD principles

The private repository verifies a reproducible backend build through GitHub Actions:

- SDK and dependency versions are selected in repository metadata;
- dependency restore runs in locked mode;
- the solution is built in Release configuration;
- the complete backend test suite runs in CI; and
- test results are retained for review.

The target release flow uses immutable build artifacts and controlled promotion. Database changes are versioned and reviewed; application startup is not treated as an excuse to make uncontrolled schema changes.

## Health and readiness

The backend foundation includes process liveness and database readiness checks. A process can be alive while the application is not ready to accept transactional work, so readiness is evaluated separately from process health.

Future operational readiness also depends on worker backlog visibility, provider/reconciliation health, database capacity, critical business-integrity signals, and recovery evidence.

## Recovery and data protection

The design treats recovery as part of production correctness:

- authoritative PostgreSQL data requires backups and point-in-time recovery capability appropriate to its risk;
- binary evidence requires storage independent enough to survive application-host failure;
- recovery copies and telemetry should not depend on the same failure domain as the application;
- restore exercises must verify both data recovery and the ability to reconcile provider and financial state; and
- production secrets and recovery access must remain separately protected.

The portfolio description does not publish backup locations, retention schedules, recovery credentials, exact recovery targets, or provider account details.

## Failure posture

- API instances are replaceable because authoritative state is not held in process memory or local disks.
- Workers use durable work and retry state so interruption does not silently discard a committed obligation.
- Provider calls occur outside long-lived database transactions.
- Unknown provider outcomes remain visible until reconciled.
- Cache failure may degrade performance but must not erase business truth.
- A controlled operator can pause unsafe payment or order paths while a critical dependency is unavailable or ambiguous.

## Current implementation status

The repository has a backend CI foundation, health/readiness behavior, PostgreSQL migrations, worker hosting, and selected payment reconciliation work. It is not a claim that the full consumer frontend, logistics runtime, broad production deployment, telemetry stack, disaster-recovery exercises, or every provider activation step is complete.

The deployment architecture also acknowledges that an early constrained pilot may have a smaller fault domain than broad production. That exception requires explicit limits, independent recovery material, external monitoring, tested restore, and a documented path to stronger resilience.

## Deliberate omissions

This document omits production hostnames, domains, IPs, ports, SSH configuration, firewall rules, internal network layouts, compose files, environment files, registry paths, provider account identifiers, credentials, and operator runbooks. They are not needed to understand the engineering approach and should remain private.
