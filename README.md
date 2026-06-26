# estore — Multi-Language E-Commerce Comparison Study

The **umbrella repository** for a study that builds one e-commerce platform against **one
frozen API contract**, then re-implements the **same backend** in several languages to compare
them fairly. One frontend talks to any backend, unchanged.

This repo is the **single source of truth** for the cross-cutting artifacts — the API contract
and the project docs. The application code lives in separate, per-language repositories
(see [Repositories](#repositories)).

## Why a shared contract

Every backend conforms to the same root-owned **OpenAPI 3 contract** (`contract/openapi.yaml`)
and **error catalog** (`contract/error-catalog.md`). Nothing is generated *from* a backend; the
contract is hand-authored and authoritative (decisions D2 / D18).

- **Backends** are held to the contract by **contract tests** — e.g. the Laravel suite uses
  [Spectator](https://github.com/hotmeteor/spectator) to assert every request/response matches
  the schema, and per-module tests assert every thrown error code is documented in
  `error-catalog.md`.
- **The frontend** generates its TypeScript types from the contract
  (`openapi-typescript` → `pnpm gen:api`), so the typed API client is compile-checked against it.

Change the contract → re-run the backend contract tests and `pnpm gen:api` on the frontend, and
every drift surfaces immediately. This is what makes the language comparison fair: the contract,
not any one implementation, is the fixed point.

## Repositories

The code is **polyrepo** — each implementation is its own repository. They are linked here and
will be wired as git **submodules** once published.

| Repo | Role | Stack | Status |
|---|---|---|---|
| [`estore-laravel`](https://github.com/aristonis/estore-laravel) | Backend API | Laravel 13 modular monolith · PHP 8.4 · PostgreSQL | Feature-complete (140 tests) |
| [`estore-nuxt`](https://github.com/aristonis/estore-nuxt) | Frontend | Nuxt 4 / Vue 3 SSR · Pinia · pnpm | Feature-complete |
| `estore-springboot` | Backend (2nd impl) | Spring Boot · Java | Planned (B3) |
| `estore-dotnet` | Backend (3rd impl) | .NET · C# | Planned (B4) |

> The `estore-*` repos above are separate repositories (not yet pushed to remotes). Until they
> are, they live beside this repo in the working tree and consume the contract by relative path
> (`../estore/contract/openapi.yaml`).

## Layout

```
estore/                     # this umbrella repo (aristonis/estore)
├── contract/               # ⭐ Root-owned OpenAPI 3 contract — single source of truth (D2)
│   ├── openapi.yaml        #    the API contract
│   └── error-catalog.md    #    stable error codes (module.ErrorName), one 100-block per module (D18)
├── docs/
│   ├── decision-log.md     #    every locked decision (D1…) with rationale
│   ├── requirements/       #    spec.md (problem, FR/NFR, acceptance criteria)
│   ├── architecture/       #    system design (8-module modular monolith)
│   ├── stage-gates/        #    dev-workflow gate records
│   ├── references.md       #    reference catalog
│   └── work-log.md         #    chronological build log
├── LICENSE
└── README.md
```

## v1 scope

Catalog · Cart · Checkout (no online payment) · Accounts/Auth · Admin/Inventory.
Actors: Guest, Customer, Admin. Persistence: PostgreSQL.

See [`docs/decision-log.md`](docs/decision-log.md) for the decisions behind this layout, and
[`docs/requirements/spec.md`](docs/requirements/spec.md) for the full specification.
