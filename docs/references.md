# Reference Library — estore

External references the user pointed us to. Root: `~/MyFile/programming/softwareEnginering/`.
Consult these before reaching for memory/web on the topics they cover.

## 1. `system-design-components/` — ⭐ the design methodology we adopt
The user's own "design backend components before building them" repo. Design-only (Markdown +
Mermaid + language-agnostic pseudocode). Each component: a level-independent **`base/`** contract,
then realized at three decoupling levels — **modular-monolith → module → microservice**.

**`base/` file set (the template we mirror for every estore module):**
`context.md` · `user-stories.md` (stories + NFR table) · `use-case.md` · `crc-cards.md` ·
`domain-model.md` (class diagram) · `data-model.md` (logical ER) · `glossary.md`.
Each level adds: `architecture.md` · `sequence.md` · `pseudocode.md` · `tradeoffs.md`.

**Already complete — reuse directly (our architecture = modular-monolith, D7/D12):**
- **`auth/`** — register / login / logout / forgot / reset / change. One verify path + per-channel
  credential-issuer **registry** (OCP); generalized `findByIdentifier`; single `ACCOUNT` ("admin"
  is an authorization concern, not auth); opaque non-enumerable ids; generic + timing-safe (no
  enumeration); reset tokens hashed/single-use/out-of-band; revoke-all on reset, revoke-others on
  change; audit log; per-identity + per-IP rate limiting. **Maps to our Identity/Auth module.**
- **`rate-limiting/`** — thresholds & buckets (per-identity + per-IP). **Maps to our rate-limiting NFR.**

→ Stage 2 plan: plug in `auth` + `rate-limiting` as-is; author Catalog / Cart / Ordering / Inventory
  in the identical `base/` + `modular-monolith/` format.

## 2. `examples/vip_backend/` — ⭐ primary Laravel structural + package reference
Real **Laravel 13 modular monolith** via `nwidart/laravel-modules`. Modules incl. Admin,
Authorization, User, Products, Promo/PromoItem, Sales, Notifications, Core. Per-module layout:
`app/{Models,Services,Policies,Observers,Providers,Console,Support}` · `config` ·
`database/{migrations,factories,seeders}` · `routes` · `resources/views` · `tests/{Feature,Unit}` · `docs`.
**Package shortlist it proves out (→ D8 approval list in Stage 2):** `nwidart/laravel-modules`,
`laravel/sanctum`, `spatie/laravel-permission`, `spatie/laravel-query-builder`,
`spatie/laravel-medialibrary`, `filament/filament` (admin), `dedoc/scramble` (OpenAPI),
`archtechx/enums`, `f9webltd/laravel-api-response-helpers`, Octane/Horizon (perf/queues).

## 3. `Arab-Enterprise-Kit-Lite/` — Spring Boot + Angular enterprise starter
Java/Spring Boot backend (`pom.xml`, `src/`) + **Angular 21 + PrimeNG** frontend, Docker Compose
(dev + prod), nginx, deploy scripts, numbered `Documentation/` set. **Not** a Laravel/Nuxt source.
→ Reference for the **future Spring Boot sibling (backlog B3)** + Docker/deploy + doc-structure template.

## 4. `examples/laravel-authentication`, `examples/laravel-support-tickets`
Focused Laravel 12/13 references: Sanctum-based auth package; a domain module built as a composer
package (`spatie/laravel-package-tools`). Concrete implementation patterns.

## 5. `system-design-101/` — ByteByteGo education repo
General system-design diagrams/articles. Background only.

---
**Local reference also available** (per global CLAUDE.md): `~/MyFile/programming/referances/laravel_docs/`
(authoritative Laravel docs) and `desingPattern/` (GoF patterns + refactoring). Grep these first for
Laravel/pattern questions.
