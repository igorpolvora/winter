# Architecture (v1)

Winter is a modular monolith (see [ADR 001](adr-001-modular-monolith.md)). The diagrams render automatically on GitHub.

## 1. System context

```mermaid
flowchart LR
    Customer([Customer / Admin]) --> SPA[React SPA]
    SPA -->|REST + JWT| API[Winter API<br/>Spring Boot modular monolith]
    API --> DB[(PostgreSQL)]
    API -->|Create Checkout Session| Stripe[Stripe<br/>test mode]
    Stripe -->|Webhooks| API
```

## 2. Modules and dependencies

```mermaid
flowchart TD
    Identity[Identity]
    Catalog[Catalog]
    Cart[Cart] --> Catalog
    Orders[Orders] --> Cart
    Orders --> Catalog
    Orders --> Inventory[Inventory]
    Orders --> Payments[Payments]
    Payments -. "events: PaymentSucceeded / PaymentFailed" .-> Orders
```

Rules:

- Dependencies point in one direction only. `Payments` never calls `Orders` directly; it publishes events that `Orders` listens to. This avoids a circular dependency.
- A module only uses another module's public interface, never its internal classes or tables.
- `Identity` provides authentication to the whole application (security filter) and is not called by business modules.

## 3. Internal structure of each module

```
com.winter.<module>/
├── api/             REST controllers and request/response objects
├── application/     use cases (business operations)
├── domain/          entities and business rules
└── infrastructure/ database access and external services
```

Business rules live in `domain`. Controllers stay thin and the database and Stripe details stay in `infrastructure`.

## 4. Checkout flow (the core problem)

```mermaid
sequenceDiagram
    actor C as Customer
    participant FE as React SPA
    participant OR as Orders
    participant INV as Inventory
    participant PAY as Payments
    participant ST as Stripe

    C->>FE: Click "Pay"
    FE->>OR: POST /orders (JWT)
    OR->>INV: Reserve stock (single DB transaction)
    INV-->>OR: Reserved
    OR->>PAY: Create payment for order
    PAY->>ST: Create Checkout Session
    ST-->>PAY: Session URL
    PAY-->>FE: Redirect URL
    FE->>ST: Customer pays on Stripe page
    ST->>PAY: Webhook checkout.session.completed
    PAY->>PAY: Verify signature, check STRIPE_EVENTS (idempotency)
    PAY-->>OR: Event PaymentSucceeded
    OR->>OR: Order status PAID
    OR->>INV: Confirm reservation
```

Failure path: if the payment fails, the Checkout Session expires or the order is canceled, the reservation is released and the stock becomes available again.

Key guarantees:

- **No overselling:** reservation uses optimistic locking on `INVENTORY.version`.
- **No duplicate processing:** each Stripe event id is stored once; repeated webhooks are ignored.
- **No trust in the browser:** an order is marked `PAID` only after the webhook, never because the customer was redirected back to the site.
- **Consistency:** creating the order and reserving stock happen in one database transaction.

## 5. Cross-cutting concerns

| Concern | Approach |
|---|---|
| Authentication | JWT issued by `Identity`, validated by a security filter |
| Authorization | Roles `CUSTOMER` and `ADMIN`, checked per endpoint |
| Errors | Standard error response format across the API |
| Database changes | Versioned migrations with Flyway |
| Tests | Unit tests for domain rules, integration tests with a real PostgreSQL (Testcontainers) |
| Delivery | Docker for local run, GitHub Actions for CI |