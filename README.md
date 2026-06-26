# estore — Multi-Language E-Commerce Comparison Study

This is the umbrella repository for the estore project. The goal is straightforward: build one
e-commerce platform against a fixed API definition, then rebuild the same backend in several
languages and compare how each implementation works out. The frontend stays the same throughout
and runs against whichever backend is up.

The application code lives in separate repositories, listed under [Repositories](#repositories)
below. What this repository holds is the material they all share: the API definition and the
project documentation.

## The shared contract

The whole project is organised around one OpenAPI definition (`contract/openapi.yaml`) and a
companion list of error codes (`contract/error-catalog.md`). We write these by hand and treat
them as the reference. They are never generated from a backend.

Each part of the system checks itself against that definition:

- The backends run contract tests. The Laravel backend uses
  [Spectator](https://github.com/hotmeteor/spectator) to verify that every request and response
  matches the schema, and a set of tests to verify that every error code it can throw is listed
  in the error catalog.
- The frontend generates its TypeScript types directly from the contract (`pnpm gen:api`), so
  the API client will not compile if it drifts from the agreed shapes.

When the contract changes, you re-run the backend tests and regenerate the frontend types, and
any mismatch shows up straight away. This is what keeps the comparison honest: every
implementation is measured against the same definition rather than against each other.

## Repositories

The code lives in these repositories:

| Repository | Role | Stack | Status |
|---|---|---|---|
| [`estore-laravel`](https://github.com/aristonis/estore-laravel) | Backend API | Laravel 13 modular monolith, PHP 8.4, PostgreSQL | Feature-complete (140 tests) |
| [`estore-nuxt`](https://github.com/aristonis/estore-nuxt) | Frontend | Nuxt 4 / Vue 3 SSR, Pinia, pnpm | Feature-complete |
| `estore-springboot` | Second backend | Spring Boot, Java | Planned (B3) |
| `estore-dotnet` | Third backend | .NET, C# | Planned (B4) |

## Layout

```
estore/
├── contract/
│   ├── openapi.yaml        # the API definition
│   └── error-catalog.md    # error codes (module.ErrorName), one 100-block per module
├── docs/
│   ├── decision-log.md     # the decisions taken, with reasoning
│   ├── requirements/       # spec.md: problem, requirements, acceptance criteria
│   └── architecture/       # system design (eight-module modular monolith)
├── LICENSE
└── README.md
```

## Scope (v1)

Catalog, cart, checkout (no online payment), accounts and authentication, and an admin area for
catalog and inventory. Three roles: guest, customer, and admin. Data is stored in PostgreSQL.

For the reasoning behind the layout and the decisions, see
[`docs/decision-log.md`](docs/decision-log.md). The full specification is in
[`docs/requirements/spec.md`](docs/requirements/spec.md).
