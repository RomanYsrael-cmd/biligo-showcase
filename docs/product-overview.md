# Product overview

Biligo is a Philippine-focused multi-vendor marketplace platform. Its product scope brings together buyer commerce, seller and store operations, inventory, checkout, online payment flows, marketplace financial evidence, delivery serviceability, and the operational capabilities needed for a later fulfillment and Rider experience.

The project is intentionally treated as a transaction platform. A marketplace is not only a catalog and a set of CRUD screens: it must preserve what a buyer agreed to, avoid overselling under contention, distinguish payment evidence from provider callbacks, explain where money is owed, and keep physical-delivery state separate from commercial state.

## Product surfaces

The design serves several actors through a shared domain platform:

| Actor or surface | Responsibility |
| --- | --- |
| Guest and Buyer | Discover products, manage a cart, validate checkout, place purchases, and view buyer-owned information |
| Seller and Store staff | Manage stores, catalog content, moderation state, inventory, and seller-facing operational work |
| Rider and logistics operations | Receive future fulfillment, dispatch, delivery-attempt, custody, and COD responsibilities |
| Support, Finance, Compliance, and Operations | Perform controlled administrative and exception workflows through the owning domain rules |
| External providers | Supply payment, messaging, mapping, storage, and future logistics evidence through adapters |

Not every surface is complete. The current private repository has substantial backend foundations and transaction cores; frontend applications and physical fulfillment execution remain project work.

## What the current backend proves

The implemented backend demonstrates:

- a central identity and authorization authority rather than separate credential stores per persona;
- seller/store, catalog, publication, buyer profile, address, cart, inventory, checkout-session, and delivery-quote boundaries;
- server-owned checkout conversion that builds durable purchase and seller-order facts from persisted evidence;
- payment-intent, provider-operation, confirmation, evidence, status-lookup, expiration, and reconciliation boundaries;
- a financial Ledger posting substrate and immutable Seller earning entitlement substrate;
- PostgreSQL-backed reliability primitives for idempotency, duplicate integration evidence, and durable post-commit work.

These capabilities are deliberately described as boundaries and foundations. They are not presented as proof that the entire marketplace, all clients, or all operational workflows are finished.

## Conceptual buyer flow

1. Catalog and Inventory provide current sellable content and availability evidence.
2. The Buyer Cart and Checkout boundaries validate ownership, current prices, stock, address, serviceability, delivery quote, and an allowed payment method.
3. A server-owned conversion creates the durable commercial graph: one buyer-level Purchase, Store-scoped Seller Orders, transaction-time item snapshots, reservations, and a Payment Intent.
4. Only after durable local state exists does the system interact with an external payment provider.
5. Provider evidence is verified, deduplicated, and reconciled before it changes Biligo-owned payment and financial state.
6. Financial consequences are represented as Ledger and entitlement facts. Future fulfillment and settlement workflows consume those facts without rewriting them.

The flow is intentionally conceptual. The public repository does not disclose endpoint paths, request/response shapes, table definitions, or provider payloads.

## Current scope snapshot

| Capability | Status in the private system | Portfolio interpretation |
| --- | --- | --- |
| Backend foundation and reliability | Implemented | Health/readiness, migrations, idempotency, Inbox, Outbox, and durable work foundations |
| Identity and authorization | Core implemented | Central identity, revocable sessions, scoped permissions, and server-side policy evaluation |
| Seller, Store, Catalog, and Inventory | Core implemented | Store-owned catalog and stock boundaries with publication and reservation integrity |
| Buyer profile, address, and cart | Core implemented | Authenticated self-service and multi-store cart foundations |
| Checkout, serviceability, and quote | Core implemented | Persisted validation evidence, quote validity, payment selection, and conversion boundary |
| Payments | Core and provider boundary implemented | Durable intents, evidence, confirmation/reconciliation mechanisms, and a hosted adapter boundary |
| Financial Ledger and Seller earnings | Substrates implemented | Append-only posting and entitlement foundations; complete operational settlement is later |
| Fulfillment, dispatch, Rider execution | Designed, not complete | Physical shipment truth and custody workflows remain future work |
| COD, returns, refunds, disputes | Designed, not complete | The public showcase does not imply these workflows are live |
| Frontend applications | Not complete | Buyer, Seller, Admin, and Rider client delivery remains future work |
| Broad production operations | Partial foundation | CI and health/readiness exist; deployment, telemetry, and recovery operations continue to mature |

## What this is not

Biligo is not being presented here as a finished consumer marketplace, a reference implementation, or an open-source framework. The showcase is a technical case study of a private product and its engineering approach.

The following are intentionally absent:

- production source code or copied implementation fragments;
- canonical database diagrams, migrations, indexes, triggers, or constraints;
- complete API or event specifications;
- financial posting, pricing, payment, payout, logistics, or fraud algorithms;
- customer, seller, rider, or operational data;
- credentials, provider identifiers, private addresses, internal URLs, and infrastructure configuration.
