# estore — Requirements Specification (v1)

**Status:** DRAFT — proposed for **Spec Freeze**. Format follows the `system-design-components`
user-story + NFR style (D11). Auth requirements are **reused** from the canonical `auth` component
(D12), not re-derived here.

Conventions: story = *As a … I want … so that …*; acceptance = *Given / When / Then*.
Functional reqs grouped by module; each `FR-x` is one observable behavior.

---

## 1. Problem
There is no single, contract-driven e-commerce codebase that (a) ships a working store and
(b) serves as the fair baseline for re-implementing the *same* backend across Laravel / Spring Boot /
.NET. Each language would otherwise drift in features and API shape, making comparison meaningless.

## 2. Goal
A working v1 store — **browse → cart → checkout (order placed, no payment)** — served by
`estore-laravel` (modular monolith) and driven by `estore-nuxt`, with **every endpoint conforming to
the root OpenAPI 3 contract** so any future backend is a drop-in replacement.

## 3. Actors
| Actor | Description |
|---|---|
| **Guest** | Anonymous visitor. Browses catalog, builds a cart (anonymous cart token). Cannot place an order. |
| **Customer** | Registered user. Everything a guest can do, plus place orders and view own order history. |
| **Admin** | Manages products, categories, offers, stock, and all orders. Authenticates identically to others; "admin" is an **authorization role** (RBAC), not a separate auth flow. |

---

## 4. Functional Requirements

### 4.0 Identity / Auth — REUSED (see `system-design-components/auth`)
- **FR-AU** — Register / login / logout / forgot / reset / change password per the canonical `auth`
  component **US-1..US-6** and **NFR-1..NFR-14**. Two channels: browser session + API bearer
  (Sanctum). On register, the account is granted the **Customer** role; **Admin** is seeded.
  *This spec does not restate those stories; it incorporates them by reference.*

### 4.1 Catalog (Guest + Customer)
- **FR-C1 Browse products** — *As a guest, I want a paginated product list so I can shop.*
  - Given published products, when I list, then I get name, base price, **effective price** (after any
    active offer), thumbnail, category names, and stock availability (in/out of stock).
  - Unpublished/draft products are never listed publicly.
- **FR-C2 Filter & sort** — *As a guest, I want to filter and sort so I can find products.*
  - Given the catalog, when I filter by category and/or price range and sort by price or newest,
    then the list reflects it. (Allowed filters/sorts are an explicit allow-list — no arbitrary fields.)
- **FR-C3 Product detail** — *As a guest, I want a product page.*
  - Given a product's opaque id/slug, when I view it, then I get full description, image set, base +
    effective price, active offer (if any), stock availability, and its categories.
  - An unknown/unpublished id returns **404**.
- **FR-C4 Categories** — *As a guest, I want to browse by category.*
  - Given flat categories, when I list them or open one, then I see the category and its products
    (a product may appear in several categories — many-to-many).
- **FR-C5 Offers (effective price)** — *As a guest, I want to see discounts.*
  - Given a product with an **active** offer (now within its date window), when I view it, then the
    **effective price** = base price reduced by the best applicable active offer, with an offer badge.
  - Given no active offer, then effective price = base price.
  - Offer discount type is **percentage or fixed-amount**, resolved via a **registry** (a new type is a
    registration insert, never an edit — OCP).

### 4.2 Cart (Guest + Customer)
- **FR-K1 Add to cart** — *As a guest or customer, I want to add a product with a quantity.*
  - Given an in-stock product and qty ≤ available, when I add it, then a cart line is created/updated.
  - Given qty > available stock, when I add, then I get a **stock-availability** error and the cart is
    unchanged (fail loud).
  - A guest cart is keyed by an **anonymous cart token**; a customer cart is keyed by account.
- **FR-K2 View cart** — line items (product, qty, **effective unit price**, line total) and cart total.
- **FR-K3 Update cart** — change a line's quantity or remove it; quantity changes **re-validate stock**.
- **FR-K4 Merge on login** — *As a returning customer, I want my guest cart kept.*
  - Given a guest cart and a login, when I authenticate, then the guest cart **merges** into my account
    cart (summing quantities, re-validated against stock); the anonymous token is retired.

### 4.3 Ordering / Checkout (Customer only)
- **FR-O1 Place order** — *As a customer, I want to turn my cart into an order.*
  - Given a **logged-in** customer with a non-empty cart and a shipping/contact address, when I check
    out, then — **in one DB transaction** — stock is re-validated and **atomically decremented**, an
    **Order** is created with a **price snapshot** per line (price at order time), `payment_status =
    pending`, `order_status = placed`, and the cart is emptied.
  - Given insufficient stock for any line at checkout time, then the whole order **fails loud** and
    nothing is decremented (all-or-nothing).
  - Given a guest (not logged in), then checkout is rejected — login required (D14).
- **FR-O2 Order confirmation** — returns the order with an **opaque order number**, line snapshot,
  totals, and statuses.
- **FR-O3 Order history** — *As a customer, I want to see my past orders.* Returns only **my** orders.
- **FR-O4 Order detail** — *As a customer, I want one order's detail.* My own only; another user's or
  unknown order returns **404** (scope-before-authorize, not 403).

### 4.4 Inventory
- **FR-I1 Stock tracking** — each product carries a stock quantity; availability is derived from it.
- **FR-I2 Atomic decrement** — stock decremented inside the order transaction (FR-O1); concurrent
  checkouts never oversell.
- **FR-I3 Restock on cancel** — cancelling an order restocks its lines atomically (FR-A4).

### 4.5 Admin (RBAC-gated)
- **FR-A1 Products** — admin CRUD products: name, description, base price, images, categories, stock,
  published flag.
- **FR-A2 Categories** — admin CRUD flat categories.
- **FR-A3 Offers** — admin CRUD offers: discount type + value, date window (`starts_at`/`ends_at`),
  linked products, active flag.
- **FR-A4 Orders** — admin lists all orders, views any order, and transitions `order_status`
  (`placed → fulfilled`, `placed → cancelled`); **cancel restocks** atomically (FR-I3).
- **FR-A5 Authorization** — every admin endpoint rejects non-admins. Capability checks → **403**;
  resource-ownership checks → **404** (scope-before-authorize). Roles: guest, customer, admin (RBAC).

---

## 5. Non-Functional Requirements
| ID | Category | Requirement |
|---|---|---|
| NFR-1 | Contract | Every endpoint conforms to the root **OpenAPI 3** contract (request + response shapes). The contract is the single source of truth; impl is validated against it. |
| NFR-2 | Security | Public ids **opaque & non-enumerable** (orders, products); never expose raw incremental ids; reject reversible/zero-padded ids. |
| NFR-3 | Correctness | **Money is exact decimal**, never float — prices, discounts, line totals, order totals. |
| NFR-4 | Integrity | Order placement + stock decrement is **one atomic DB transaction**; no overselling under concurrency. |
| NFR-5 | Security (auth) | Per the `auth` component NFR-1..NFR-14 (slow-KDF hashing, generic+timing-safe/no-enumeration, hashed revocable tokens, audit log, config-driven policy). |
| NFR-6 | Security | **Rate limiting** on public + auth endpoints, **per-identity + per-IP**, per the `rate-limiting` component. |
| NFR-7 | Security | All input validated at the boundary (FormRequests / typed DTOs). |
| NFR-8 | Security | Authorization enforced on **every** protected route (RBAC); scope-before-authorize. |
| NFR-9 | Performance | p95 < 200 ms for auth; p95 < 300 ms for catalog/cart/order under normal load. |
| NFR-10 | Ops | Runs via **Docker Compose** on **PostgreSQL**; reproducible env; all config via env vars, validated at startup. |
| NFR-11 | Security / Contract | All error responses use **one consistent, contract-defined shape** — **RFC 9457 Problem Details** (`application/problem+json`; successor to RFC 7807), portable to Spring/.NET. Extended with two stable members: **`code`** (numeric, block-per-module) and **`code_name`** (`module.ErrorName`); `type` is a stable URI derived from `code_name`. Errors never leak internals/stack traces; unexpected errors map to a generic 500. |
| NFR-12 | Architecture | **Modular monolith**: sealed module boundaries — cross-module calls go service-to-service, never reaching into another module's tables/models. |
| NFR-13 | Quality | Test coverage ≥ **80%** on logic (unit + integration + E2E); wiring/`main` may be lower, covered by acceptance run. |
| NFR-14 | Architecture | Every error path flows through a **typed domain-exception hierarchy** (base app exception → specific domain exceptions, each declaring HTTP status + numeric `code` + `code_name` (`module.ErrorName`) + safe message), rendered by the **framework-native central exception handler** — not per-controller `try/catch`. Framework exceptions (validation, auth, not-found) are mapped centrally to the same NFR-11 shape. All codes registered in a single **error catalog** (source of truth). |

---

## 6. Constraints
| ID | Constraint |
|---|---|
| C-1 | Backend = **Laravel (PHP 8.3+)**, modular monolith via `nwidart/laravel-modules` (D13). |
| C-2 | Frontend = **Nuxt 3 / Vue 3** (rendering mode decided in Stage 2 — D9). |
| C-3 | Persistence = **PostgreSQL** in Docker Compose (D6). |
| C-4 | Root-owned **OpenAPI 3 contract** is the single source of truth (D2). |
| C-5 | Prefer mature Laravel ecosystem packages (D8); **any new dependency needs explicit approval**. |
| C-6 | **No online payment** integration in v1 (D4). |

---

## 7. Scope
**In:** catalog (products, flat many-to-many categories, offers), cart (guest + merge), checkout →
order (no payment), inventory (atomic stock), accounts/auth (reused), admin (products / categories /
offers / stock / orders), RBAC.

**Out (→ backlog):** online payments (B1), coupon **codes** / promotion-rules engine (B2),
Spring Boot & .NET impls (B3/B4), search (B5), reviews/wishlist (B6), shipping/tax/multi-currency
(B7), multi-vendor (B8), nested categories, email verification / 2FA / social login (per `auth` deferrals).

---

## 8. Acceptance Criteria
Each is human-checkable and traces to FR/NFR.

| ID | Criterion | Traces |
|---|---|---|
| AC-1 | A guest can list, filter by category + price, sort, and open a product detail page; unpublished products never appear; unknown id → 404. | FR-C1..C3 |
| AC-2 | A product with an active offer shows the discounted effective price + badge; outside the date window it shows base price. | FR-C5 |
| AC-3 | A product belongs to multiple categories and appears under each. | FR-C4 |
| AC-4 | A guest adds items to a cart; adding qty beyond stock is rejected with a stock error. | FR-K1 |
| AC-5 | After login, the guest cart merges into the customer's cart with quantities summed and re-validated. | FR-K4 |
| AC-6 | A logged-in customer checks out a valid cart → an order is created with a price snapshot, `pending` payment, `placed` status; stock is decremented; cart emptied. A guest is blocked from checkout. | FR-O1, FR-I2, D14 |
| AC-7 | Two concurrent checkouts of the last unit → exactly one succeeds; the other fails loud; stock never goes negative. | NFR-4, FR-I2 |
| AC-8 | A customer sees only their own orders; requesting another's order → 404. | FR-O3, FR-O4, NFR-8 |
| AC-9 | An admin creates/edits/deletes products, categories, and offers; sets stock. | FR-A1..A3, FR-I1 |
| AC-10 | An admin cancels an order → its stock is restocked atomically. | FR-A4, FR-I3 |
| AC-11 | A non-admin calling an admin endpoint is rejected (403 capability / 404 ownership). | FR-A5, NFR-8 |
| AC-12 | Register/login/logout/reset/change all behave per the `auth` component's acceptance criteria. | FR-AU |
| AC-13 | Every implemented endpoint validates against the root OpenAPI contract (request + response). | NFR-1 |
| AC-14 | All money values are exact decimal (no float drift in totals/discounts). | NFR-3 |
| AC-15 | Auth + public endpoints are rate-limited per-identity and per-IP. | NFR-6 |
| AC-16 | The stack boots from `docker compose up` against PostgreSQL with env-based config. | NFR-10, C-3 |
| AC-17 | A domain error (e.g. insufficient stock), a validation error, an auth error, and an unknown route all return the **same RFC 9457 shape** carrying the right `status` + numeric `code` + `code_name` (`module.ErrorName`), leaking no internals; the path goes through the typed exception + central handler; every code exists in the error catalog. | NFR-11, NFR-14 |
