# Winter

A full stack e-commerce platform built as a portfolio project, focused on solving the hard problems of online retail: secure authentication, payments, consistent stock and reliable order handling.

> **Status:** in design. This README describes the planned scope. Items are checked off in the [roadmap](#roadmap) as they are built.

## About

Winter is a single-store e-commerce application inspired by large marketplaces. The goal is not to have many screens, but to show how real engineering problems are solved: payment flows with external providers, concurrent stock updates and well-defined business rules.

It is a prototype for demonstration only. Payments run in Stripe **test mode**, and no real money is involved.

## Scope (v1)

- User registration and login (JWT-based authentication, customer and admin roles)
- Product catalog with search, filters and pagination
- Shopping cart
- Order creation and order lifecycle (created, paid, shipped, delivered, canceled)
- Payment with Stripe Checkout, confirmed through webhooks
- Stock control that prevents overselling
- Admin area to manage products and orders

## Out of scope (v1)

- Multi-vendor marketplace (seller accounts, commissions, payouts via Stripe Connect). Planned as a future version; the architecture is designed to allow it.
- Real payments, shipping carrier integrations and tax calculation

## Tech stack (planned)

| Layer | Technology |
|---|---|
| Backend | Java 21, Spring Boot |
| Security | Spring Security, JWT |
| Database | PostgreSQL, Flyway (migrations) |
| Frontend | React, TypeScript, Vite |
| Payments | Stripe (test mode) |
| Tests | JUnit 5, Testcontainers |
| Infrastructure | Docker, GitHub Actions |
| API docs | OpenAPI / Swagger |

All tools and services used in this project have free options.

## Architecture

Winter is a **modular monolith**: one deployable application, organized internally into independent modules with clear boundaries.

| Module | Responsibility |
|---|---|
| Identity | Registration, login, roles |
| Catalog | Products, categories, search |
| Cart | Shopping cart management |
| Orders | Order creation and lifecycle |
| Payments | Stripe integration and webhooks |
| Inventory | Stock reservation and control |

The reasoning behind this decision is recorded in [ADR 001](docs/adr-001-modular-monolith.md).

## Key engineering challenges

1. **Payments and webhooks:** handling asynchronous events from Stripe without creating duplicate orders or charges (idempotency).
2. **Stock concurrency:** two customers buying the last unit at the same time must not result in overselling.
3. **Order state machine:** valid transitions between order states, enforced by the domain rules.
4. **Transactions:** operations that must succeed or fail as a whole (all or nothing).
5. **Security:** authentication, authorization and protection of routes and data.

## Roadmap

- [ ] Project setup and documentation
- [ ] Backend foundation: REST API, database, product CRUD
- [ ] Authentication and authorization
- [ ] Cart and orders
- [ ] Stripe integration and webhooks
- [ ] React frontend
- [ ] Concurrent stock handling, tests, CI and deployment

## Documentation

- [ADR 001: Modular monolith](docs/adr-001-modular-monolith.md)

More documents (architecture diagrams, data model, API reference) will be added as the project evolves.

## Running locally

Instructions will be added once the first version of the application is available.

## Author

Igor, Software Engineering.
