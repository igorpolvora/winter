# Data model (v1)

Relational model for PostgreSQL. The diagram renders automatically on GitHub.

```mermaid
erDiagram
    USERS ||--o| CARTS : has
    USERS ||--o{ ORDERS : places
    CATEGORIES ||--o{ PRODUCTS : groups
    PRODUCTS ||--|| INVENTORY : tracks
    CARTS ||--o{ CART_ITEMS : contains
    PRODUCTS ||--o{ CART_ITEMS : "added as"
    ORDERS ||--|{ ORDER_ITEMS : contains
    PRODUCTS ||--o{ ORDER_ITEMS : "sold as"
    ORDERS ||--o{ PAYMENTS : "paid by"

    USERS {
        uuid id PK
        string email UK
        string password_hash
        string role "CUSTOMER | ADMIN"
        timestamp created_at
    }
    CATEGORIES {
        uuid id PK
        string name UK
    }
    PRODUCTS {
        uuid id PK
        uuid category_id FK
        string name
        string description
        bigint price_cents
        string currency
        boolean active
        timestamp created_at
    }
    INVENTORY {
        uuid product_id PK, FK
        int available_quantity
        int reserved_quantity
        int version "optimistic locking"
    }
    CARTS {
        uuid id PK
        uuid user_id FK
        timestamp updated_at
    }
    CART_ITEMS {
        uuid id PK
        uuid cart_id FK
        uuid product_id FK
        int quantity
    }
    ORDERS {
        uuid id PK
        uuid user_id FK
        string status "see order lifecycle"
        bigint total_cents
        string currency
        timestamp created_at
    }
    ORDER_ITEMS {
        uuid id PK
        uuid order_id FK
        uuid product_id FK
        string product_name_snapshot
        bigint unit_price_cents_snapshot
        int quantity
    }
    PAYMENTS {
        uuid id PK
        uuid order_id FK
        string stripe_session_id UK
        string stripe_payment_intent_id
        string status "PENDING | SUCCEEDED | FAILED"
        bigint amount_cents
        timestamp created_at
    }
    STRIPE_EVENTS {
        string event_id PK "Stripe event id, makes webhooks idempotent"
        string type
        timestamp processed_at
    }
```

## Order lifecycle

```
CREATED -> PAID -> SHIPPED -> DELIVERED
   |
   +--> CANCELED   (allowed from CREATED or PAID)
```

Only these transitions are valid. Any other transition is rejected by the domain rules.

## Design decisions

1. **Money as integer cents** (`price_cents`), never floating point. Avoids rounding errors.
2. **Price and name snapshot in `ORDER_ITEMS`.** Changing a product later must not change past orders.
3. **Separate `INVENTORY` table with `available` and `reserved`.** Stock is reserved when the order is created and confirmed or released depending on the payment result. The `version` column supports optimistic locking, which prevents overselling under concurrent purchases.
4. **`STRIPE_EVENTS` table.** Each webhook event is stored by its Stripe id. If the same event arrives twice, the second one is ignored (idempotency).
5. **UUIDs as primary keys.** Ids are not guessable or sequential, which is safer to expose in a public API.
6. **`PAYMENTS` separate from `ORDERS`.** An order can have more than one payment attempt (for example, a failed card followed by a successful one).
7. **Passwords stored only as hashes** (`password_hash`), never in plain text.