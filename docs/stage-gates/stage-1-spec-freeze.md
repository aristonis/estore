# Stage Gate 1 — Spec Freeze

**Status:** ✅ PASSED — 2026-06-25 (user approved scope + "go next").

## Criteria
- [x] Every FR is a single observable behavior; Auth reused by reference (FR-AU).
- [x] Every NFR measurable (NFR-1..14).
- [x] Constraints explicit (C-1..6); Scope in/out named.
- [x] Every AC (AC-1..17) traces to an FR/NFR and is human-checkable.
- [x] No open requirement-level questions (D9 Nuxt mode + D7 module boundaries deferred to Stage 2 by agreement).

## Frozen artifact
`docs/requirements/spec.md` — Catalog (FR-C1..5), Cart (FR-K1..4), Ordering (FR-O1..4),
Inventory (FR-I1..3), Admin (FR-A1..5), Auth (FR-AU, reused). NFR-1..14. AC-1..17.

## Key locked decisions since draft
- D17 — typed exception hierarchy + framework-native central handler; **RFC 9457 Problem Details**.
- D18 — error `code` (numeric, block-per-module) + `code_name` (`module.ErrorName`); error catalog = source of truth.

## After freeze
Spec changes go through the decision log. Next: **Stage 2 — System Design** (state set accordingly).
