# estore — Error Catalog (source of truth)

Every error the API can return. Each carries a stable numeric **`code`** and a **`code_name`**
(`module.ErrorName`), rendered in the RFC 9457 Problem Details body (D17/D18, NFR-11/14).
`type` URI = `https://errors.estore.dev/{module}/{ErrorName}`.

**Numbering = one 100-block per module.** Adding an error = the next free number in that block
(OCP — never renumber existing codes). Adding a module = the next free block.

| code | code_name | HTTP | When |
|------|-----------|------|------|
| **common 1000–1099** | | | cross-cutting (Core) |
| 1000 | common.ValidationFailed | 422 | Boundary validation failed (field errors in `errors`). |
| 1001 | common.RateLimited | 429 | Rate-limit window exceeded (per-identity or per-IP). |
| 1002 | common.Unauthenticated | 401 | Authentication required / missing or invalid token. |
| 1003 | common.Forbidden | 403 | Authenticated but lacks the capability (non-ownership). |
| 1004 | common.NotFound | 404 | Unknown route or generic resource miss. |
| 1005 | common.InternalError | 500 | Unexpected error (no internals leaked). |
| **identity 1100–1199** | | | auth (reused component) |
| 1100 | identity.IdentifierInUse | 409 | Register with an already-registered identifier. |
| 1101 | identity.InvalidCredentials | 401 | Login failed — generic, timing-safe, no enumeration. |
| 1102 | identity.InvalidResetToken | 400 | Reset token unknown/expired/used — generic. |
| 1103 | identity.PasswordPolicyViolation | 422 | New password fails the configured policy. |
| 1104 | identity.CurrentPasswordMismatch | 401 | change-password: current password wrong (generic, timing-safe). |
| 1105 | identity.SamePassword | 422 | New password equals the current one. |
| **access 1200–1299** | | | RBAC |
| 1200 | access.Forbidden | 403 | Role/permission denies the action (admin-only endpoint). |
| **catalog 1300–1399** | | | products / categories |
| 1300 | catalog.ProductNotFound | 404 | Unknown or unpublished product (public view). |
| 1301 | catalog.CategoryNotFound | 404 | Unknown category. |
| 1303 | catalog.SlugConflict | 409 | Product/category slug collision on create. |
| **pricing 1400–1499** | | | offers |
| 1400 | pricing.OfferNotFound | 404 | Unknown offer. |
| 1401 | pricing.InvalidDiscount | 422 | e.g. percentage > 100, or negative value. |
| 1402 | pricing.InvalidOfferWindow | 422 | `ends_at` ≤ `starts_at`. |
| 1403 | pricing.UnknownDiscountType | 422 | Discount type not in the registry. |
| **inventory 1500–1599** | | | stock |
| 1501 | inventory.InsufficientStock | 409 | Requested qty exceeds available (add-to-cart / checkout). |
| 1502 | inventory.NegativeQuantity | 422 | Stock set to a negative value. |
| **cart 1600–1699** | | | cart |
| 1601 | cart.ItemNotFound | 404 | Updating/removing a line not in the cart. |
| **ordering 1700–1799** | | | orders / checkout |
| 1700 | ordering.OrderNotFound | 404 | Unknown order, or not owned by the caller (scope-before-authorize). |
| 1701 | ordering.CartEmpty | 409 | Checkout with an empty cart. |
| 1702 | ordering.IllegalTransition | 409 | Status transition not allowed (e.g. fulfil a cancelled order). |

**Implementation note:** each row is one subclass of `Core\Exception\DomainException` declaring
`status()`, `code()`, `codeName()`; the central handler reads these to render Problem Details.
Framework exceptions map to the `common.*` rows. A test asserts every thrown `code_name` exists here.
