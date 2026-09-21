# System context

This diagram shows Biligo's public-level system boundary and the categories of external actors and providers it is designed to work with. It intentionally uses provider categories instead of account names, URLs, payloads, or network details.

~~~mermaid
flowchart LR
    Guest[Guest / Buyer]
    Seller[Seller / Store staff]
    Rider[Rider / Logistics operations]
    Staff[Support / Finance / Compliance / Operations]

    Biligo[Biligo platform]

    Payment[Payment provider]
    Notify[Notification providers]
    Maps[Mapping and routing provider]
    Storage[Object and evidence storage]
    Courier[Future courier integrations]

    Guest --> Biligo
    Seller --> Biligo
    Rider --> Biligo
    Staff --> Biligo

    Biligo <--> Payment
    Biligo --> Notify
    Biligo <--> Maps
    Biligo <--> Storage
    Biligo <--> Courier
~~~

## Boundary notes

- External providers return evidence through adapters; they do not directly mutate Biligo business state.
- Binary media and private evidence are handled separately from transactional relational records.
- Rider and courier capabilities are future operational integrations in the current implementation status.
- The diagram is conceptual and does not describe deployment topology, trust configuration, provider contracts, or internal endpoints.
