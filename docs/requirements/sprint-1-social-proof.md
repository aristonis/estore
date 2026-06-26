# estore — Sprint 1 Spec: Social Proof

**Status:** ✅ **FROZEN** (Spec Freeze passed 2026-06-26; OQ-1/2/3 all resolved below).
**Sprint scope:** Reviews & Ratings + Wishlist + Real related-products.
**Amends:** the v1 [`spec.md`](./spec.md) Scope §7 — pulls **B6 (reviews / wishlist)** from "Out → backlog"
into "In". Recommendations engine and guest wishlist stay out.

This spec is a **delta** to the v1 spec. Actors, NFRs, and conventions are **reused by reference**;
only what is new or changed is restated here. Story = *As a … I want … so that …*;
acceptance = *Given / When / Then*.

---

## 1. Problem
A v1 product page shows price, stock, and images but **no social proof** — a shopper cannot see what
other buyers thought, cannot save a product for later, and the "You may also like" strip is random
(pulled from `/products` with no relation to the current product). This hurts trust and return visits.

## 2. Goal
Make the storefront trustworthy and sticky by adding **customer reviews + star ratings**, a per-customer
**wishlist**, and **genuinely related** product suggestions — each a contract-conforming slice that
respects the modular-monolith boundaries (Reviews and Wishlist are **new self-contained modules**;
neither reaches into another module's tables).

## 3. Actors (reused — see v1 spec §3)
| Actor | New capabilities in this sprint |
|---|---|
| **Guest** | Reads published reviews + average rating; sees related products. **No** writing reviews, **no** wishlist. |
| **Customer** | Writes / edits / deletes **own** review (one per product); manages a personal wishlist. |
| **Admin** | Moderates reviews (hide / unhide / delete) via the existing RBAC `manage-*` pattern. |

---

## 4. Functional Requirements

### 4.1 Reviews & Ratings — new `Reviews` module
- **FR-R1 Read reviews + dual rating** — *As a guest, I want to read a product's reviews and compare
  what everyone says vs what real buyers say.*
  - Given a published product, when I open it, then I see **two star averages shown together**: the
    **overall** average (all published reviews) and the **verified-buyers-only** average (published
    reviews by customers who purchased it), each with its own **count**, as 1–5 stars (1 decimal).
  - Then a **paginated** list of published reviews — each with rating, optional title/body, reviewer
    display name, date, and a **"Verified purchase"** badge when applicable — optionally filterable to
    **verified only** (`?verified=true`).
  - Catalog **list + detail** responses carry `average_rating` + `review_count` **and**
    `verified_average_rating` + `verified_review_count` (each `null`/`0` when none), so cards and the
    product page can show both. (Contract amendment to the product schema.)
- **FR-R2 Write a review** — *As a customer, I want to rate and review a product.*
  - Given a logged-in customer, when I submit a **rating 1–5** (required) with an optional title
    (≤ 120 chars) and body (≤ 2000 chars), then a review is created **published** and the product's
    aggregate updates.
  - A customer may have **at most one** review per product; a second submit is rejected
    (`reviews.AlreadyReviewed`).
  - **Verified-purchase flag:** at submit (and on edit) the **Ordering service** is asked
    (service-to-service) whether this customer has an order containing this product; the boolean is
    **snapshotted on the review** so both aggregates stay pure within-module SQL. Purchase is **not
    required** to post a review — it only drives the badge and the verified-buyers average
    *(OQ-1 resolved)*.
  - Guests are rejected — login required.
- **FR-R3 Edit / delete own review** — *As a customer, I want to change or remove my review.*
  - Given my own review, when I update the rating/title/body or delete it, then it changes / disappears
    and the aggregate recomputes. Another customer's review → **404** (scope-before-authorize).
- **FR-R4 Moderate reviews** — *As an admin, I want to remove abusive reviews.*
  - Given any review, when I **hide** it (status `published → hidden`), then it is excluded from public
    lists and aggregates but retained; I can **unhide** or **delete** it. Gated `manage-reviews`.
  - Moderation model: reviews **auto-publish**, admin removes after the fact — no pre-approval queue
    *(see OQ-2)*.

### 4.2 Wishlist — new `Wishlist` module
- **FR-W1 View wishlist** — *As a customer, I want to see products I saved.*
  - Given a logged-in customer, when I open my wishlist, then I get my saved products with their current
    catalog presentation (effective price, stock, thumbnail) — resolved via the **Catalog service**.
- **FR-W2 Add to wishlist** — *As a customer, I want to save a product for later.*
  - Given a product, when I add it, then it appears in my wishlist. Adding the **same product twice is
    idempotent** (no duplicate, no error). Unknown product → 404.
- **FR-W3 Remove from wishlist** — removing a saved product takes it off the list.
- **FR-W4 Customers only** — wishlist requires auth; **no guest wishlist** in v1 *(see OQ-3)*. Moving an
  item to the cart reuses the **existing cart endpoints** — no new backend.

### 4.3 Related products (Catalog enhancement)
- **FR-X1 Related products** — *As a shopper, I want suggestions actually related to what I'm viewing.*
  - Given a product, when I request related items, then I get up to **4** other **published** products
    that **share at least one category**, excluding the product itself; fewer (or none) if not enough
    exist. Served by a new `GET /products/{publicId}/related`; the product detail page uses it instead
    of the random strip.

---

## 5. Non-Functional Requirements
All v1 NFRs (NFR-1..NFR-14) **apply unchanged** — contract conformance, opaque ids, exact-decimal money,
RBAC + scope-before-authorize, boundary validation, RFC 9457 errors via the typed-exception + central
handler, sealed module boundaries, ≥ 80% coverage. Sprint-specific emphases:

| ID | Category | Requirement |
|---|---|---|
| NFR-S1 | Security (XSS) | Review title/body are **user-generated content**: stored raw, rendered as **text not HTML** on the frontend (no `v-html`); length-capped + validated at the boundary. |
| NFR-S2 | Security (abuse) | Review submission is **rate-limited** (reuse the per-identity + per-IP buckets); one-review-per-product enforced server-side. |
| NFR-S3 | Architecture | `Reviews` and `Wishlist` are **new sealed modules**; cross-module reads (product, purchase) go **service-to-service** (Catalog, Ordering) by `public_id` — never touching another module's models. New error blocks: **Reviews 1800–18xx**, **Wishlist 1900–19xx**, registered in the error catalog. |
| NFR-S4 | Correctness | **Both** averages (overall + verified-only) are computed from **published** reviews only and stay consistent after create / edit / delete / hide (recomputed, not drifting); the verified average counts only reviews flagged verified-purchase. |

---

## 6. Constraints
Inherits v1 §6 (Laravel modular monolith, Nuxt, PostgreSQL, root OpenAPI contract, new dependency needs
approval). Sprint adds **no new dependency** (reviews/wishlist/related are pure domain + existing stack).

---

## 7. Scope
**In:** Reviews & ratings (read/write/edit/delete own + admin moderation), per-customer wishlist
(add/remove/list, customers only), real category-based related products, product-aggregate fields on
catalog responses, contract amendments + conformance, storefront + admin UI.

**Out (→ backlog):** recommendation **engine** / personalization (stays B6), **guest** wishlist,
review images / replies / helpful-votes, review-only-if-verified-purchase enforcement (badge only this
sprint), wishlist sharing.

---

## 8. Open Questions — ALL RESOLVED (Spec Freeze 2026-06-26)
| ID | Question | Recommended default |
|---|---|---|
| **OQ-1** | *(RESOLVED 2026-06-26)* Purchase-gate reviews, or open to all? | **Open to all logged-in customers; show TWO ratings together — overall (all reviewers) + verified-buyers-only — plus a per-review verified badge. Stars 1–5.** |
| **OQ-2** | *(RESOLVED)* Auto-publish or pre-moderation queue? | **Auto-publish** + admin hide/delete (status `published`↔`hidden`, plus delete). No pending queue. |
| **OQ-3** | *(RESOLVED)* Wishlist for guests too? | **Customers only** in v1. Guest wishlist → backlog. |

---

## 9. Acceptance Criteria
| ID | Criterion | Traces |
|---|---|---|
| AC-S1 | A guest opens a product and sees BOTH the overall average and the verified-buyers-only average (each with its count + stars), then a paginated review list where verified-purchase reviews show a badge; the list can be filtered to verified-only. | FR-R1 |
| AC-S2 | Catalog list + detail responses include `average_rating`/`review_count` AND `verified_average_rating`/`verified_review_count`; cards render the overall stars. | FR-R1 |
| AC-S3 | A logged-in customer posts a rating 1–5 (+ optional title/body); it appears immediately and the average updates. A guest is blocked. | FR-R2 |
| AC-S4 | A second review by the same customer on the same product is rejected (`reviews.AlreadyReviewed`). | FR-R2 |
| AC-S5 | A customer edits then deletes their own review; the aggregate recomputes. Editing/deleting another customer's review → 404. | FR-R3 |
| AC-S6 | An admin hides a review → it vanishes from public lists and the average; unhide restores it; delete removes it. Non-admin → rejected. | FR-R4, NFR-8 |
| AC-S7 | A customer adds a product to their wishlist, sees it listed with live price/stock, and removes it; adding the same product twice is idempotent; guest wishlist access is rejected. | FR-W1..W4 |
| AC-S8 | A product detail page's "related" strip shows ≤ 4 published products sharing a category, never the product itself. | FR-X1 |
| AC-S9 | Review title/body containing HTML is rendered as inert text (no script execution); over-length input is rejected at the boundary. | NFR-S1 |
| AC-S10 | Every new endpoint validates against the root OpenAPI contract (request + response); errors use the RFC 9457 shape with Reviews/Wishlist codes present in the catalog. | NFR-1, NFR-11, NFR-S3 |
