# Project setup and workflow

## Repository structure

```
winter/
├── README.md
├── docker-compose.yml        # local PostgreSQL
├── .env.example              # variable names only, no real secrets
├── .gitignore
├── .github/
│   └── workflows/
│       └── ci.yml            # build and test on every push (added in the CI phase)
├── docs/                     # architecture, data model, API design, ADRs
├── backend/                  # Java 21 + Spring Boot
└── frontend/                 # React + TypeScript + Vite
```

A single repository (monorepo) keeps code and documentation together, which makes the project easy to review.

## Environment variables

`.env.example` lists the variables the project needs. The real `.env` is in `.gitignore` and is never committed.

```
DB_USER=winter
DB_PASSWORD=change_me
JWT_SECRET=change_me
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
```

## Git workflow

- `main` is always in a working state.
- Work happens in short branches: `feature/<name>`, `fix/<name>`, `docs/<name>`.
- Changes enter `main` through pull requests, even working alone. This shows the process and creates a readable history.
- Commit messages follow Conventional Commits:
  - `feat: add product listing endpoint`
  - `fix: prevent negative stock on reservation`
  - `docs: add data model diagram`
  - `test: cover order status transitions`
  - `chore: add docker-compose for local database`

## Milestones (GitHub)

Each milestone matches a roadmap phase in the README. Each phase is broken into small issues.

| Milestone | Example issues |
|---|---|
| 0. Setup and documentation | Repository structure, docker-compose, README, ADR 001, data model, API design |
| 1. Backend foundation | Spring Boot project, Flyway migrations, product CRUD, standard error handling |
| 2. Authentication | Register, login, JWT, refresh token, role-based access |
| 3. Cart and orders | Cart endpoints, order creation, order status rules, idempotency key |
| 4. Stripe | Checkout Session, webhook signature check, `STRIPE_EVENTS`, expiration handling |
| 5. Frontend | Catalog page, product page, cart, login, checkout, order history |
| 6. Quality and delivery | Concurrency tests, Testcontainers, CI pipeline, deployment, final docs |

## Definition of done (for any issue)

1. The behavior works as described in `docs/`.
2. Tests cover the business rules involved.
3. Documentation is updated if the design changed.
4. The change entered `main` through a pull request.