# Stage Gate 0 — Idea Intake & Setup

**Status:** ✅ PASSED — 2026-06-25 (user: "yes").

## Criteria
- [x] Idea restated and confirmed by the user.
- [x] Scope (in/out) agreed; non-goals named.
- [x] Actors agreed (Guest / Customer / Admin).
- [x] Persistence/env baseline agreed (PostgreSQL + Docker).
- [x] Decision log seeded (D1–D13); backlog seeded (B1–B8).
- [x] Workspace skeleton created (`.progress/`, `docs/`, `process/`, `memory/`, `contract/`, `temp/`).
- [x] Reference library catalogued (`docs/references.md`) and folded into decisions (D11–D13).

## Agreed problem statement
Build **estore**, a monorepo e-commerce platform defined by one frozen, root-owned OpenAPI 3
contract, so the same backend feature set can be re-implemented in multiple languages (Laravel now;
Spring Boot, .NET later) and compared fairly, while one Nuxt 3 frontend works against any backend
unchanged. **v1 builds only `estore-laravel` (modular-monolith API) + `estore-nuxt`**, on
PostgreSQL + Docker: browse catalog → cart → checkout (order recorded, **no online payment**), with
Guest / Customer / Admin actors and admin product/inventory management.

## Open items carried forward (do not block Stage 1)
- D9 — Nuxt rendering mode (SPA vs SSR) → Stage 2.
- D7 — exact module boundaries → Stage 2.
- D10 — depth of "offers/promotions" → confirm in Stage 1 spec.

## Next
Stage 1 — Requirements: build `docs/requirements/spec.md` in the user's user-stories + NFR format.
Auth requirements reuse the canonical `auth` component (D12) rather than being re-derived.
