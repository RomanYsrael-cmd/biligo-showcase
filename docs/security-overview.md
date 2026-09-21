# Security overview

This page describes Biligo's defensive security posture at a portfolio level. It is not a threat model, penetration-test report, or implementation guide.

## Identity and authentication

The backend uses one canonical user identity. Buyer, seller, store, rider, and staff capabilities are represented through domain relationships and scoped memberships rather than separate credential stores.

The current identity core includes:

- memory-hard password verification;
- revocable server-side sessions;
- server-derived authentication context;
- account and session lifecycle checks;
- generic failure behavior for sensitive authentication paths; and
- browser request protections appropriate to a first-party application.

Authentication is deliberately separate from authorization and business-state validation. Being signed in does not grant access to a particular Store, Purchase, case, fulfillment record, payout, or administrative capability.

## Authorization

Authorization follows a default-deny model. A decision considers the authenticated user, permission, resource relationship or ownership scope, assurance level, risk conditions, and the current business state.

Controls are enforced server-side. Hiding a button or route in a future client does not replace the policy check. Privileged operations are expected to use stronger assurance and durable audit evidence, and platform administration must invoke the owning domain rules rather than edit business state directly.

The public repository intentionally does not publish the role catalog, permission names, scope identifiers, session parameters, authorization decision code, or sensitive action matrix.

## Input and state validation

Commands validate both input shape and the current lifecycle state of the resource. The server derives sensitive identity, ownership, pricing, delivery, payment, and financial values from authoritative state instead of trusting client-provided copies.

Duplicate-sensitive actions use idempotency and conflict detection. Database-backed integrity and transaction boundaries protect money, stock, historical snapshots, and durable work from partial updates or races.

## Secrets and configuration

Credentials and provider secrets are supplied through approved configuration mechanisms and are not committed to source control. Environments are intended to use separate secrets, data stores, provider credentials, and access controls.

This showcase contains no .env files, connection strings, passwords, API keys, tokens, certificates, signing material, private hostnames, internal URLs, IP addresses, or production configuration.

## Provider and webhook boundaries

External providers supply evidence; they do not directly own Biligo's order, payment, inventory, Ledger, settlement, or fulfillment state. Provider-specific calls sit behind adapters, and inbound evidence passes through verification, deduplication, normalization, and an owning-domain transition.

An uncertain remote result is kept distinct from confirmed failure. Reconciliation is used before an unsafe retry or a financial conclusion. Provider credentials, webhook URLs, signature algorithms, payload schemas, and replay-sensitive details are deliberately omitted.

## Data minimization and historical evidence

The architecture aims to keep personally identifiable information within the domain that needs it. Cross-domain consumers receive the minimum reference or snapshot required for their responsibility. Binary evidence is kept outside the relational record by default, with access mediated by application authorization.

Historical commercial and financial facts are preserved as evidence. Corrections are represented as later business or financial effects rather than silent edits to posted history.

## Auditability and operations

Security-sensitive and privileged actions are designed to produce durable audit metadata separate from ordinary operational logs. Logs are for operating the system; audit evidence explains who performed a sensitive action, on what resource, and why, without copying secrets or unnecessary personal data.

Operational controls also include health/readiness checks, structured telemetry, controlled background processing, backup/recovery expectations, and the ability to pause unsafe transaction paths when critical dependencies cannot be trusted.

## Status and limitations

The identity, authorization, transactional-integrity, and provider-boundary foundations are implemented in the private backend. Broader MFA/recovery workflows, full frontend delivery, production secret activation, and complete operational security readiness remain project work.

The public documents do not claim that a portfolio repository is a security certification or that all production controls are complete. Manual review is still required before a public release or production launch.
