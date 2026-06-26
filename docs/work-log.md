# Work Log

One entry per session. Newest last. What was done, decisions, open items, next step.

## Session 01 — 2026-06-25 — Stages 0–3 complete; Stage 4 started
- **Done:** Drove the whole pipeline through design under a user **autonomy grant** ("do all stages").
  - **Stage 0** — idea + scope agreed; workspace seeded; references catalogued (`docs/references.md`):
    user's `system-design-components` methodology (reuse `auth`+`rate-limiting`), `examples/vip_backend`
    (Laravel modular-monolith + package shortlist), AEKL (Spring Boot+Angular → future B3). Gate signed.
  - **Stage 1** — `spec.md` frozen: Catalog/Cart/Ordering/Inventory/Admin FRs + reused Auth, NFR-1..14,
    C-1..6, AC-1..17. Spec Freeze signed.
  - **Stage 2** — `system-design.md` (8 modules, ports, domain/ER, atomic checkout flow, stack);
    `contract/openapi.yaml` (OpenAPI 3.1, 24 paths/30 schemas, validated) + `contract/error-catalog.md`.
    Design Freeze signed.
  - **Stage 3** — `process/blueprints/v1-blueprint.md` (4 sections, SG-0..SG-10).
  - **Stage 4** — toolchain verified. **SG-0 ✓** (Laravel 13.17, packages, nwidart modules, Postgres:5440
    + Redis:6380 docker, Pest on `estore_test`, baseline green). **SG-1 ✓** Core error kernel
    (DomainException + Common/*, RFC 9457 ProblemDetails renderer, central handler, catalog-conformance
    test) — suite green 9/9, Pint clean.
- **Decisions added:** D11–D13, D14 cart, D15 inventory-atomic, D16 categories M:N, D17 RFC 9457 errors,
  D18 error-code scheme. D7/D8/D9/D10 locked.
- **Deviations:** `Money` helper moved SG-1 → SG-4. No Docker for dev (D19, host Postgres `deployer`).
  Per-code git repos (D20: estore-laravel own repo, root local-only). Reset uses a token-keyed
  `password_resets` table (contract reset is token-only).
- **SG-2 Identity ✓** (suite 30/30): register/login/logout/me + forgot/reset/change via Sanctum; opaque
  account ids; generic+timing-safe auth; hashed single-use out-of-band reset tokens; audit log.
- **Skills:** user added stack skills under `estore/.claude/skills/` (laravel-best-practices,
  laravel-module, laravel-permission-development, laravel-query-builder, medialibrary-development,
  configuring-horizon); fixed `laravel-module` missing frontmatter.
- **Next step:** SG-3 Access (RBAC roles/policies, seed roles) → SG-4 Catalog → … → SG-9 → frontend.

## Session 02 — 2026-06-25 — Backend domain complete (SG-3 … SG-8)
- **Done:** Built the full backend domain across 8 modules, all TDD, suite **124/124 green**:
  - **SG-3 Access** — RBAC enums, RoleSeeder (customer/admin + manage-* perms), EnsurePermission middleware.
  - **SG-4 Catalog** — Money; products/categories (M:N); public browse/filter/sort (spatie QB)/detail; admin
    CRUD; product images (medialibrary, contract amended D21).
  - **SG-5 Pricing** — Offer + discount-strategy registry (OCP); effective-price service; admin offer CRUD;
    storefront discounts (AC-2).
  - **SG-6 Inventory** — stock keyed by product public id; atomic oversell-safe decrement/restock; stock on
    product views; admin set-stock.
  - **SG-7 Cart** — guest+customer carts (X-Cart-Token), stock-validated, effective pricing; merge-on-login
    via UserLoggedIn event.
  - **SG-8 Ordering** — atomic checkout (AC-6/AC-7), price snapshot, order history/detail (own-only 404),
    admin transitions + restock-on-cancel.
- **Decisions:** D21 (contract amend: product image upload). Infra: no Docker dev, host Postgres `deployer`.
- **Conventions discovered:** brick/math `RoundingMode::HalfUp` (PascalCase); spatie `allowedSorts` is
  variadic; nwidart `database/seeders` PSR-4; deleting a module's `routes/api.php` breaks boot (keep minimal).
- **Next step:** SG-9 conformance (Spectator/rate-limit/seeders) → Stage 5 acceptance → Nuxt frontend (SG-10).
