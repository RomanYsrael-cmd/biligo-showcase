# Architecture overview

Biligo starts as a modular monolith with strict domain ownership. One application codebase can run as an API process, background workers, reconciliation/scheduled workers, and controlled administrative jobs. These are process roles, not a collection of independently deployed microservices.

The choice reflects the product's early transactional coupling: checkout, inventory, commerce, payment evidence, and financial effects must remain coherent while the product and team are still evolving. The codebase is divided by business ownership so that future extraction remains possible when measurable scaling, isolation, or team-ownership needs justify it.

## Logical architecture

~~~mermaid
flowchart LR
    Clients[Buyer, Seller, Rider, Staff clients]
    App[Application and API boundary]

    subgraph Modules[Domain modules]
        IAM[Identity and access]
        Commerce[Catalog, Inventory, Cart, Checkout, Commerce]
        Money[Payments, Ledger, Seller earnings]
        Operations[Delivery, Fulfillment, After-sales, Admin]
    end

    subgraph Durable[Durable platform services]
        DB[(PostgreSQL transactional authority)]
        Reliability[Idempotency, Inbox, Outbox, durable work]
        Objects[Private object and evidence storage]
    end

    Workers[Workers and reconciliation]
    Adapters[Provider adapters]
    Providers[Payment, messaging, mapping, storage, future logistics providers]
    Observability[Health, logs, metrics, traces, alerts]

    Clients --> App
    App --> IAM
    App --> Commerce
    App --> Money
    App --> Operations
    Modules --> DB
    Modules --> Reliability
    Modules --> Objects
    Reliability --> Workers
    Workers --> Adapters
    Adapters <--> Providers
    App --> Observability
    Workers --> Observability
    DB --> Observability
~~~

The diagram shows logical boundaries. It does not imply separate services, reveal deployable host topology, or define a public API.

## Boundary responsibilities

### Application and API boundary

The API accepts actor intent and turns it into commands or queries. It does not become an alternate source of truth. Protected commands derive identity and scope from server-side authentication context, validate current business state, and delegate writes to the owning domain.

### Domain modules

Each authoritative business state has one logical owner. Other modules use narrow application contracts, immutable references, derived read models, or durable facts. Administrative tools orchestrate domain commands; they are not a universal database bypass.

### PostgreSQL

PostgreSQL is the authoritative transactional store for the implemented backend. It provides the local transaction boundary, persistence integrity, concurrency controls, and durable evidence needed for duplicate-sensitive operations. The public documentation intentionally does not describe the canonical schema or its physical enforcement details.

### Reliability primitives

Idempotency records prevent a retried command from applying a different effect. Inbox records make duplicate integration evidence visible and replay-safe. Outbox records commit with the owning state so workers can process durable facts after the transaction succeeds. Processing assumes at-least-once delivery and makes the effect safe to repeat.

### Workers and reconciliation

Background processes claim durable work, publish or apply post-commit facts, expire safe work, and reconcile ambiguous provider outcomes. A worker is held to the same domain authorization and business-state rules as an interactive command.

### External adapters

Provider-specific SDKs and payloads are kept outside domain logic. The adapter boundary translates external evidence into a Biligo-owned result; the owning domain then decides whether and how its state changes. A provider never directly mutates an order, inventory, Ledger, or settlement record.

## Consistency strategy

Biligo uses strong local consistency for business changes that must commit together. Checkout conversion, for example, validates current evidence and persists the commercial graph, reservations, payment intent, and durable follow-up facts as one local transaction boundary.

Remote calls are not included inside a long-held database transaction. The system persists intent first, performs the remote operation, records verified evidence, and reconciles until the external and internal views agree. An unknown provider outcome is treated as a state requiring evidence, not as permission to blindly retry.

Historical transaction facts are preserved as snapshots. Current Catalog, Store, Buyer, or configuration records may change later; the purchase record remains explainable from the evidence captured at conversion time.

## Runtime responsibilities

The same application artifact can support:

- request-serving API processes;
- asynchronous workers for durable work and integration facts;
- scheduled/reconciliation workers for provider and financial follow-up;
- controlled administrative maintenance jobs.

This separation makes the runtime independently scalable without prematurely introducing network-distributed domain transactions.

## Failure and degradation posture

- Database unavailability stops success claims for transactional writes.
- Provider timeouts produce pending or unknown outcomes that require reconciliation.
- Notification failure does not roll back an already committed business transaction.
- Worker interruption leaves durable work available for later recovery.
- Cache loss cannot lose authoritative orders, money, inventory, or audit evidence.
- Evidence-dependent actions do not claim success unless the evidence was durably stored.

## Public boundary

This repository intentionally omits the canonical ERD, migration SQL, exact indexes and triggers, endpoint inventory, request/response payloads, event contracts, deployment hostnames, network layout, credentials, and operational runbooks. Those details are unnecessary to evaluate the architecture and would make the private implementation easier to reconstruct.
