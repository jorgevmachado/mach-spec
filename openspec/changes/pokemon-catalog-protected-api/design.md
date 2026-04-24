## Context

`mach-api` already contains authentication, trainer initialization, base repository/service abstractions, pagination support, and Pokemon-adjacent utility helpers, but it does not yet contain a Pokemon catalog domain. `mach-web` already has authenticated navigation and a basic Pokemon list client hook, so the missing piece is a protected backend catalog and a first frontend page that uses the existing pagination pattern.

This change introduces a new global Pokemon catalog sourced from PokeAPI. Access to the catalog must require a valid API token, but the Pokemon domain must not depend on `Trainer`, `trainer_id`, trainer history, or trainer-owned collection concepts. Related resources such as moves, abilities, types, and growth rates will be exposed in the backend now, while the frontend only consumes Pokemon list pagination in this phase.

## Goals / Non-Goals

**Goals:**
- Add a protected Pokemon catalog in `mach-api` with local persistence and synchronization from PokeAPI.
- Support paginated Pokemon listing filtered by `name` and `order` using the project’s existing pagination conventions.
- Support protected Pokemon detail retrieval by `name` or `id`, with lazy enrichment when a Pokemon is still incomplete.
- Model and expose related backend resources for moves, abilities, types, and growth rates.
- Enable the `mach-web` `/pokemon` page to consume the paginated listing without introducing a new frontend pagination contract.

**Non-Goals:**
- No `trainer_id` parameter, trainer history, or Pokemon-to-trainer relationship.
- No `/pokedex` or `/my-pokemon` implementation.
- No frontend consumption yet for moves, abilities, types, or growth rates.
- No battle mechanics or trainer-owned inventory flows.

## Decisions

### 1. Pokemon is a global catalog, not a trainer-owned resource
The backend will model `Pokemon` and related entities as global catalog data. Authentication is only an access gate through `get_current_user`, not a domain dependency.

Rationale:
- The user explicitly removed trainer coupling.
- The list and detail APIs are catalog operations, not trainer state mutations.
- This keeps repository and service signatures cleaner and makes cache keys independent of user-owned resources.

Alternatives considered:
- Reusing `trainer_id` only for access logging. Rejected because it still couples catalog reads to trainer state without product value in this phase.
- Creating a trainer-owned `Pokedex` now. Rejected as out of scope.

### 2. First request may bootstrap the local catalog from PokeAPI
`PokemonService` will support initial synchronization from PokeAPI so the local database becomes the primary read source. List endpoints will read from local persistence using standard pagination and filters.

Rationale:
- The prompt already assumes bootstrap and sync behavior.
- The frontend expects paginated reads that fit local query semantics better than paginating through PokeAPI on every request.
- A local catalog makes protected related-resource endpoints and future detail enrichment predictable.

Alternatives considered:
- Proxying all list traffic directly to PokeAPI. Rejected because it conflicts with local filtering, cache usage, and status-based completion flow.
- Requiring a manual admin sync before use. Rejected because it adds unnecessary setup friction for the first delivery.

### 3. Detail requests complete incomplete Pokemon lazily
Pokemon rows created during bootstrap start as `INCOMPLETE` with basic fields. A protected detail request will enrich that row from PokeAPI, persist relations, and mark it `COMPLETE`.

Rationale:
- It keeps initial sync lighter and faster.
- It preserves a clear distinction between list-safe minimal data and richer detail data.
- It allows moves, abilities, types, growth rates, and evolution data to be populated only when needed.

Alternatives considered:
- Fully hydrating all Pokemon during bootstrap. Rejected because it increases startup time, external API volume, and migration risk for the first iteration.

### 4. Related resources get their own protected endpoints now
Moves, abilities, types, and growth rates will have backend models, services, and protected routes in this change, even though the frontend will not consume them yet.

Rationale:
- The user explicitly chose option B.
- Exposing routes now makes the capability testable and avoids backend-only hidden scope.
- These resources are natural parts of the Pokemon catalog model and will be needed by detail or later frontend work.

Alternatives considered:
- Only preparing internal services without routes. Rejected because it leaves the capability partially delivered and harder to validate.

### 5. Frontend only adopts the existing paginated list pattern
The `/pokemon` page in `mach-web` will consume the same paginated response shape already used by the project’s list hooks and DS pagination helpers. Filters are limited to `name` and `order`.

Rationale:
- The project already has `usePaginatedList` and Pokemon list scaffolding.
- Reusing the existing contract reduces frontend churn and backend ambiguity.
- Limiting the first slice to list-only keeps the frontend delivery aligned with the requested scope.

Alternatives considered:
- Building detail pages and related-resource UI now. Rejected as unnecessary scope expansion.

## Risks / Trade-offs

- [Initial catalog bootstrap can be slow or fail on external API errors] → Mitigate with local persistence, descriptive `502 Bad Gateway` handling, and list sync logic that only fills missing entries after the initial load.
- [Route protection may accidentally leak trainer assumptions through copied patterns] → Mitigate by using auth only at the router boundary and keeping Pokemon services free of trainer parameters.
- [Related entities increase migration and modeling complexity] → Mitigate by separating global catalog models from trainer concepts and using explicit association tables.
- [Frontend and backend pagination parameters could diverge] → Mitigate by adhering to the project’s existing list pagination conventions and keeping frontend filters limited to `name` and `order`.
- [Lazy detail hydration can create inconsistent expectations between list and detail payloads] → Mitigate by treating `status` as the explicit indicator of completion and using the detail endpoint to perform enrichment.

## Migration Plan

1. Add Pokemon catalog models, related-resource models, and association tables.
2. Generate and apply Alembic migration in dependency-safe order.
3. Add external PokeAPI client and schemas.
4. Add repositories, services, and protected routes for Pokemon and related resources.
5. Update `mach-web` Pokemon list page to consume the protected paginated API.
6. Validate with backend tests, frontend lint/build, and end-to-end manual verification of authenticated listing.

Rollback strategy:
- Revert the deployed application code.
- Roll back the Alembic migration if needed before data is depended on elsewhere.
- Because this change introduces new tables rather than mutating trainer-owned data, rollback scope stays bounded to the new Pokemon catalog objects.

## Open Questions

- Whether the related-resource endpoints should support pagination immediately or return full local collections when first introduced.
- Whether `GET /pokemon/{name_or_id}` should trigger sync only for the requested Pokemon or also opportunistically fill missing related entities referenced by that Pokemon.
- Whether a background sync entrypoint will be needed later, or if request-driven sync is sufficient for the near term.
