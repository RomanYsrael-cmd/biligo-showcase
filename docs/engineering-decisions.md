# Engineering decisions

Biligo's interesting engineering work is concentrated at the boundaries where marketplace state can become inconsistent: contention, retries, provider ambiguity, historical evidence, money movement, and cross-domain ownership. The decisions below summarize the design intent without publishing implementation recipes.

## 1. Modular monolith before service extraction

**Problem:** Checkout, Inventory, Commerce, Payments, and financial effects are tightly coupled while the product is still establishing its workflows.

**Decision:** Keep one deployable application codebase with explicit domain ownership and separate API, worker, and reconciliation process roles.

**Why it matters:** The platform can use strong local transaction boundaries and simple operational coordination while still preserving recognizable module contracts for future extraction. Service boundaries are earned by measured scaling, isolation, regulatory, or ownership needs rather than assumed from the start.

## 2. PostgreSQL is transactional authority

**Problem:** Marketplace correctness cannot depend on a collection of eventually consistent caches or mutable application-only counters.

**Decision:** Use PostgreSQL as the authoritative store for implemented transactional state, with database-backed integrity and concurrency enforcement where the application model alone is insufficient.

**Why it matters:** Order, inventory, payment, and financial state have one durable source of truth. Read caches and projections can improve access patterns, but losing one must not lose money, stock, or audit evidence.

## 3. Historical facts are snapshots

**Problem:** Current catalog, address, store, pricing, or policy data can change after a buyer has placed an order.

**Decision:** Capture the transaction-time facts needed to explain the purchase, quote, allocation, and financial context. Preserve them as historical evidence instead of recomputing from mutable master data.

**Why it matters:** A past transaction remains explainable and auditable without freezing the current catalog or leaking the full production data model.

## 4. Checkout conversion is a server-owned transaction

**Problem:** A client-supplied total, stale stock view, or repeated submit could create an order that does not match authoritative facts.

**Decision:** The server revalidates current evidence and converts a ready checkout session into durable commercial, reservation, and payment-intent facts. The caller supplies intent, not authoritative prices, delivery amounts, ownership, or payment state.

**Why it matters:** The conversion boundary makes replay, rollback, and conflict behavior explicit. The public showcase does not disclose the endpoint, payload, lock order, or schema used to implement it.

## 5. Duplicate-sensitive actions are idempotent

**Problem:** Browsers retry, workers restart, callbacks duplicate, and network failures make it unsafe to assume one request or one provider notification.

**Decision:** Use stable operation identity, request-fingerprint conflict detection, durable idempotency state, and replay-safe domain effects for commands that can create money, stock, or order consequences.

**Why it matters:** A retry with the same meaning converges to the original result; a retry with different material input is visible as a conflict instead of silently changing history.

## 6. Inbox and Outbox protect the local-to-remote boundary

**Problem:** A process can commit a database change and crash before publishing a follow-up fact, or receive the same provider event more than once.

**Decision:** Persist Outbox facts with the owning transaction, deduplicate inbound evidence through an Inbox boundary, and let workers process durable work at least once with safe completion/retry behavior.

**Why it matters:** Asynchronous work becomes recoverable without pretending that distributed exactly-once delivery exists. The showcase omits the envelope shape, claim query, lease details, and event contracts.

## 7. Provider uncertainty is explicit

**Problem:** A timeout after a payment or payout request does not prove failure; blindly retrying may duplicate a real external operation.

**Decision:** Persist external intent before the call, use stable provider-operation identity, distinguish confirmed failure from unknown outcome, and reconcile through authenticated evidence before deciding whether a new attempt is safe.

**Why it matters:** Provider integrations do not get to rewrite Biligo state directly, and a browser success page is not treated as financial proof. Provider credentials, webhook URLs, signatures, and payloads are deliberately absent from this repository.

## 8. Financial facts are append-only and balanced

**Problem:** Mutable seller balances and ad hoc adjustments are difficult to explain, reverse, and reconcile.

**Decision:** Use an append-only, balanced marketplace subledger with explicit money representation, logical effect identity, and reversal/compensation rather than editing posted history.

**Why it matters:** Financial outcomes remain traceable to a source effect, while settlement and payout can evolve independently. The showcase does not publish account catalogs, posting templates, allocation algorithms, or exact ledger schema.

## 9. Payment, fulfillment, and settlement remain separate

**Problem:** A payment confirmation, a delivered package, a COD remittance, a seller entitlement, and a successful payout are different facts that do not necessarily happen at the same time.

**Decision:** Keep Payment, Ledger, Fulfillment, After-sales, Settlement, and Payout as distinct owning boundaries connected by approved commands and durable facts.

**Why it matters:** A single generic status field cannot represent the lifecycle safely. The design can express delayed, reversed, disputed, held, or reconciled outcomes without rewriting unrelated state.

## 10. Delivery quotes have a lifecycle

**Problem:** Serviceability and delivery price are time-sensitive evidence, not permanent properties of a cart.

**Decision:** Evaluate stores against authoritative serviceability configuration, persist quote evidence, expose a validity window, and require current evidence again at the conversion boundary.

**Why it matters:** Checkout does not silently reuse an expired or stale quote. The public description intentionally omits zone data, rate formulas, package rules, identifiers, and persistence constraints.

## 11. Authorization is scoped and server-enforced

**Problem:** Role names alone do not determine whether a person may access a particular Store, Buyer profile, case, or financial operation.

**Decision:** Centralize identity, keep sessions revocable, evaluate permission plus resource scope plus business state on the server, and reserve stronger assurance for sensitive actions.

**Why it matters:** UI visibility is not security, and privileged administration is not a bypass around domain ownership. Exact role catalogs, scope identifiers, session parameters, and authorization implementation details remain private.

## 12. Architecture rules are tested

**Problem:** A modular monolith can gradually collapse into cross-module table writes and hidden coupling if boundaries exist only in prose.

**Decision:** Maintain structural/architecture tests alongside unit and integration tests, including checks for repository layout and write ownership, and verify critical invariants against real PostgreSQL.

**Why it matters:** The architecture is treated as an executable quality constraint, not only a diagram. Test source, fixtures, generated SQL, and environment-specific setup are not copied into this repository.

## Deliberate non-decisions

The private design does not require microservices, Kubernetes, event sourcing, a dedicated broker, a document-first authoritative store, or a complete frontend before the backend transaction foundations are sound. Those choices remain subject to evidence and later review.
