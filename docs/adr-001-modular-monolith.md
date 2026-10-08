# ADR 001: Modular monolith architecture

- **Status:** Accepted
- **Date:** 2026-10-08

## Context

Winter needs several distinct capabilities: authentication, catalog, cart, orders, payments and inventory. The project is developed by a single person, with a focus on solving hard problems (payments, stock concurrency, order lifecycle) and on keeping the architecture clear and well documented.

A decision is needed on how to structure the application: as one deployable unit or as several independent services.

## Decision

Winter will be built as a **modular monolith**: a single deployable application, internally divided into modules (Identity, Catalog, Cart, Orders, Payments, Inventory).

Rules for the modules:

- Each module owns its own logic and data access.
- Modules communicate through clearly defined interfaces, not by reaching into each other's internals.
- Dependencies between modules are kept to a minimum and documented.

## Alternatives considered

**Microservices.** Each capability would be a separate service with its own deployment and database.

Rejected because:

- It adds large operational complexity (service communication, distributed transactions, deployment, monitoring) with no real benefit at this scale.
- Distributed transactions would make problems like stock and payment consistency much harder, shifting the focus away from the business problems the project wants to show.
- A single developer would spend most of the time on infrastructure instead of on the domain.

**Unstructured monolith.** One application with no internal boundaries.

Rejected because it becomes hard to maintain and does not demonstrate architectural care.

## Consequences

**Positive**

- Simpler development, testing and deployment (one application, one database).
- Business operations that must be atomic, such as creating an order and reserving stock, can use a single database transaction.
- Clear module boundaries keep the code organized and make it possible to extract a module into a separate service later, if needed.

**Negative**

- All modules are deployed and scaled together.
- Module boundaries depend on discipline, since nothing physically prevents one module from calling another's internals. This will be mitigated with conventions and, where possible, automated checks.

## Future evolution

The planned evolution to a multi-vendor marketplace will add new modules (for example, Sellers and Payouts) without requiring a change of architecture.