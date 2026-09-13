# ADR-0001: Repurpose Greenfield4 as the product repository

**Date:** 2026-09-13
**Status:** accepted
**Deciders:** Product owner (anthracite-labs), recorded through Issue #4

## Context

Greenfield4 began as an App-Factory repository whose default documented path is to create a fresh application repository and run `scripts/init-project.sh` before product discovery. During this session the product owner explicitly directed that Greenfield4 itself should become the product repository and that the discovery work already completed should be carried into it.

The repository’s lifecycle guard can represent `PROJECT_PHASE=discovery` while keeping `ALLOW_APP_STACK=0` and an empty `STACK_DECISION_ADR`. The earlier direct factory-to-discovery PR was closed after a bootstrap audit because the exception had not been made explicit; Issue #4 now records the deliberate override and its acceptance criteria.

## Decision

Repurpose Greenfield4 itself from the reusable factory role into this product repository and transition it to the `discovery` phase through a reviewed PR. Retain the App-Factory engineering foundation, provenance, ECC adapter, verification gate, and no-stack guard unchanged.

The default “create a fresh repository from the factory” path remains the normal App-Factory guidance, but this repository is an intentional exception for this project.

## Alternatives considered

### Alternative: Create a fresh application repository from Greenfield4

- **Pros:** Follows `docs/FACTORY.md` and `docs/ROADMAP.md` exactly; preserves Greenfield4 as a reusable factory source with a clean separation of histories.
- **Cons:** Splits the active work into a second repository and requires migration/administrative setup before continuing discovery.
- **Why not:** The product owner explicitly chose to continue in Greenfield4 itself and requested the existing discovery work be carried forward here.

### Alternative: Keep Greenfield4 in `factory` while continuing discovery only in chat/issues

- **Pros:** Requires no lifecycle change.
- **Cons:** Violates the lifecycle meaning of `factory`, leaves product work outside the canonical repository state, and creates repeated context drift.
- **Why not:** Product discovery belongs in a repository whose committed lifecycle says `discovery`.

## Consequences

### Positive

- The repository’s committed lifecycle now matches the work actually being performed.
- Product decisions can live in `docs/PRODUCT.md`, `docs/DOMAIN.md`, and project memory instead of relying on chat history.
- The existing App-Factory engineering controls remain available without weakening the no-stack guard.
- Future sessions can resolve the intentional exception by reading this ADR instead of rediscovering the same conflict.

### Negative

- Greenfield4 is no longer treated as a pristine reusable factory repository for this project’s future history.
- `docs/FACTORY.md` still describes the default instantiation path, so readers must understand that ADR-0001 records an explicit exception for this repository.
- Product and foundation history now coexist in one repository.

### Follow-ups

- Finish product naming; `PROJECT_NAME` and `PROJECT_SLUG` stay blank until that decision is made.
- Complete discovery and review the product definition before moving to `architecture`.
- Evaluate the Rust-core/switchable-UI preference only during architecture; this ADR does not accept an application stack.
- Select the first two smart-TV ecosystems using the evidence requirements in `docs/PRODUCT.md`.
