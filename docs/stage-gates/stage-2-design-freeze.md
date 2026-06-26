# Stage Gate 2 — Design Freeze

**Status:** ✅ PASSED — 2026-06-25 (under the autonomy grant; design + contract complete and self-consistent).

## Criteria
- [x] Design covers **every FR** — each maps to a module facade method (`system-design.md` §3/§8).
- [x] **Every NFR has a mechanism** (`system-design.md` §6): contract+Spectator, money=NUMERIC↔brick,
      atomic guarded decrement, opaque ids, RBAC scope-before-authorize, Redis rate-limit, Problem Details.
- [x] **Stack chosen with reasons** and packages approved (D8); Nuxt SSR (D9); 8 modules (D7).
- [x] **Ports** let adapters be built and faked independently (`system-design.md` §3).
- [x] **Root contract authored** — `contract/openapi.yaml` (OpenAPI 3.1, 24 paths, 30 schemas, refs resolve).
- [x] **Error catalog authored** — `contract/error-catalog.md` (codes per module block).

## Frozen artifacts
- `docs/architecture/system-design.md`
- `contract/openapi.yaml` (the source of truth — D2)
- `contract/error-catalog.md`

## Carried decisions
D7 (8 modules), D8 (packages), D9 (Nuxt SSR), D17/D18 (errors). Money = `NUMERIC(12,2)` + currency ↔
`brick/Money`. Public ids = UUIDv4 + opaque `order_number`.

## Next
Stage 3 — Blueprint (`process/blueprints/v1-blueprint.md`), then Stage 4 execution (TDD, step-group by step-group).
Contract changes after this freeze go through the decision log.
