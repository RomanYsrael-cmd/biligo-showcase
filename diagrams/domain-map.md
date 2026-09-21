# Domain map

The domain map distinguishes ownership and conceptual relationships. It is not a canonical ERD, dependency graph, or list of deployable services.

~~~mermaid
flowchart TB
    subgraph Access[Identity and access]
        IAM[Identity and authorization]
        BuyerProfile[Buyer profile and address]
        IAM --> BuyerProfile
    end

    subgraph Commerce[Commerce domains]
        Seller[Seller and Store]
        Catalog[Catalog]
        Inventory[Inventory]
        Cart[Cart and Checkout]
        Orders[Commerce and Orders]
        Seller --> Catalog
        Catalog --> Inventory
        BuyerProfile --> Cart
        Seller --> Cart
        Catalog --> Cart
        Inventory --> Cart
        Cart --> Orders
    end

    subgraph Money[Money domains]
        Payments[Payments]
        Ledger[Financial Ledger]
        Earnings[Seller earnings]
        Settlement[Settlement and Payout]
        Orders --> Payments
        Payments --> Ledger
        Ledger --> Earnings
        Earnings --> Settlement
    end

    subgraph Operations[Operational domains]
        Delivery[Serviceability and delivery quote]
        Fulfillment[Fulfillment and dispatch]
        Rider[Rider operations]
        AfterSales[After-sales]
        Admin[Admin, audit, and reporting]
        Cart --> Delivery
        Orders --> Delivery
        Orders --> Fulfillment
        Fulfillment --> Rider
        Orders --> AfterSales
        AfterSales --> Payments
        AfterSales --> Settlement
        Admin -. controlled commands .-> Seller
        Admin -. controlled commands .-> Payments
        Admin -. controlled commands .-> Settlement
        Admin -. controlled commands .-> Fulfillment
    end

    Reliability[Idempotency, Inbox, Outbox, durable work]
    Orders -. durable facts .-> Reliability
    Payments -. durable facts .-> Reliability
    Inventory -. durable facts .-> Reliability
    Fulfillment -. future durable facts .-> Reliability
~~~

## Reading the map

- Arrows show conceptual collaboration, not direct table writes.
- Each authoritative business state has one logical owner.
- The dashed links indicate durable-fact or controlled-command relationships.
- Serviceability and quote evidence supports checkout but does not own physical shipment truth.
- Payment and Ledger are related but distinct; Seller settlement and payout are later consumers of approved financial facts.

## Implementation status

Identity, buyer profile/address, seller/store, catalog, inventory, cart/checkout, commerce conversion, payment boundaries, delivery quoting, the Ledger substrate, and Seller earning entitlement are implemented to varying core levels in the private backend. Fulfillment, Rider execution, COD, after-sales orchestration, complete settlement/payout, and frontend applications remain incomplete or deferred.
