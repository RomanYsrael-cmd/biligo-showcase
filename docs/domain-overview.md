# Domain overview

Biligo models a marketplace as several cooperating business domains with explicit ownership. The boundaries below are conceptual summaries of the private system; they are not a canonical schema or API specification.

## Domain map at a glance

| Domain | Owns conceptually | Current status |
| --- | --- | --- |
| Identity and Access | User identity, credentials, sessions, memberships, permissions, and assurance | Core implemented |
| Buyer Profile | Buyer-owned profile and address book | Core implemented |
| Seller, Store, and Compliance | Seller accounts, stores, operating state, and participation/compliance metadata | Core implemented; broader compliance workflows remain |
| Catalog | Categories, products, variants, moderation, publication eligibility, and current marketplace content | Core implemented; discovery/search and media work remain |
| Inventory | Stock availability, reservations, commits, releases, adjustments, and movement history | Core implemented |
| Cart and Checkout | Buyer cart, validation evidence, serviceability, delivery quote, payment selection, and conversion coordination | Core implemented |
| Commerce and Orders | Purchase, Store-scoped Seller Orders, line items, transaction-time snapshots, and commercial lifecycle | Core implemented for construction and conversion |
| Payments and COD | Payment intent/evidence, provider operations, confirmation/reconciliation, and future COD obligation/custody | Online-payment boundary implemented; COD remains future work |
| Financial Ledger | Append-only balanced marketplace subledger and financial posting evidence | Posting substrate implemented |
| Seller Settlement and Payout | Seller earning entitlement, holds, availability, settlement decisions, payout reservations, and payout execution | Entitlement substrate implemented; full workflow deferred |
| Delivery and Fulfillment | Serviceability, package/shipment truth, dispatch, assignments, attempts, custody, and reverse movement | Serviceability/quote core implemented; physical fulfillment deferred |
| After-sales | Returns, remedies, refunds, disputes, incidents, evidence, and liability decisions | Designed; not complete |
| Notifications, Audit, and Reporting | Side effects, durable security/business evidence, and derived read models | Architectural boundaries exist; broad operational surfaces remain |

## Ownership principles

### Identity is not a business persona

One user identity may hold compatible buyer, seller, store, rider, or staff relationships. Authentication proves the identity; authorization evaluates the relationship, resource scope, permission, assurance, and current business state. A successful login does not imply access to every store, purchase, case, or financial action.

### Seller Account is not Store

A Seller Account represents the seller relationship with the marketplace. A Store is an operating and catalog boundary within that relationship. Store-scoped permissions, catalog content, inventory, delivery rules, and Seller Orders retain their store context.

### Catalog is not transaction history

Catalog content is mutable current marketplace content. Commerce captures transaction-time product, variant, price, rule, charge, store, and address evidence so an old purchase does not change when current catalog data changes.

### Inventory is authoritative for availability

Inventory owns stock availability and reservation state. The system uses database-backed guarded operations so simultaneous buyers cannot both claim the same final stock. Movement history is evidence; it is not a second write path.

### Commerce is not fulfillment

Commerce owns the commercial purchase relationship. Fulfillment owns physical shipment and custody truth. A Seller Order can summarize physical progress, but it does not replace the operational history of packages, attempts, assignments, or custody.

### Payment is not Ledger, and Ledger is not settlement

Payment records describe how a purchase is funded and what provider evidence exists. The financial Ledger records the accounting consequences of approved business effects. Seller settlement decides when an entitlement may be released, held, reserved, or paid. These are related but separate responsibilities.

### COD has multiple truths

The design separates the buyer's COD obligation, delivery collection, rider custody, remittance, reconciliation, and seller settlement. The current showcase does not publish the detailed state machine or posting rules, and the complete COD workflow remains deferred.

## Conceptual relationships

The normal forward relationship is:

1. Catalog and Seller/Store provide current sellable content.
2. Inventory provides availability evidence.
3. Cart and Checkout validate buyer intent, address, serviceability, delivery quote, and payment selection.
4. Commerce creates a Purchase, one Seller Order per Store, and immutable order-time facts.
5. Payments creates durable funding intent and later applies verified provider evidence.
6. Inventory, Ledger, and Seller earnings consume approved effects through their owning boundaries.
7. Fulfillment will own the physical shipment lifecycle once that module is implemented.
8. After-sales and settlement will apply controlled financial or operational consequences without rewriting historical facts.

## Read models and projections

Cross-domain screens may compose read models, summaries, or projections. Those views are convenience representations. They do not replace the owning domain's authoritative state, and they are not permitted to become an unreviewed alternate write path.

## Scope of this document

The public domain summary intentionally excludes entity fields, relationship keys, database constraints, migration history, event payloads, exact lifecycle values, authorization matrices, and financial posting algorithms. The goal is to explain ownership and relationships without disclosing enough information to reproduce the production design.
