## Context

The repository already has authenticated trainer initialization and a protected Pokemon catalog, but it does not yet model trainer-owned species progress or captured instances. The new feature crosses both submodules: the API must create and manage `Pokedex` and `MyPokemon` records, and the web app must expose protected list/detail pages that reflect discovery state, catalog completeness, and trainer bootstrap progress.

The highest-coupling point is `POST /trainers/initialize`. That endpoint currently creates only the trainer record and rejects repeat calls. The new workflow must create the starter instance immediately, materialize a full trainer-scoped Pokedex asynchronously, and expose enough state for the frontend to render loading and retry states without inventing its own polling contract.

## Goals / Non-Goals

**Goals:**
- Introduce a trainer-scoped `Pokedex` aggregate for species-level progression and discovery state.
- Introduce a trainer-scoped `MyPokemon` aggregate for captured-instance progression, nickname, and equipped moves.
- Extend trainer bootstrap with a starter selection flow plus background Pokedex materialization and explicit `pokedex_status` lifecycle.
- Define protected list/detail APIs and matching web pages for both `Pokedex` and `MyPokemon`.
- Keep Pokedex and MyPokemon semantically separate so future battle/capture work has a clear source of truth.

**Non-Goals:**
- Implement the future standalone discovery service that marks arbitrary species as discovered outside trainer initialization.
- Add move-management UX beyond showing persisted moves on `MyPokemon` and species-possible moves on `Pokedex`.
- Introduce a distributed job queue in this change.
- Define battle execution rules or party-management concepts.

## Decisions

### Separate species progress from captured instances
`Pokedex` and `MyPokemon` will both store progression fields, but they represent different domain objects:
- `Pokedex` is unique per `(trainer_id, pokemon_id)` and tracks species progress plus `discovered`.
- `MyPokemon` represents a concrete captured creature, allows duplicates for the same species, and owns `nickname` plus equipped moves.

This avoids the ambiguity that would appear if a trainer had multiple captured instances of the same species but only one shared nickname or move set.

Alternative considered:
- Use `Pokedex` as the only progression store and make `MyPokemon` a thin reference layer. Rejected because it collapses species-level and instance-level concepts and makes capture/battle logic ambiguous.

### Use explicit `pokedex_status` on `Trainer`
`Trainer` will gain `pokedex_status` with `EMPTY`, `INITIALIZING`, `READY`, and `FAILED`.

Rationale:
- `null` is too ambiguous for frontend and retry logic.
- The web app needs a first-class contract for `/pokedex` preparation and retry states.
- The initialize endpoint needs deterministic idempotent behavior when called again.

Alternative considered:
- Infer readiness from the count of `Pokedex` rows. Rejected because partial data and retry semantics become harder to reason about.

### Materialize the Pokedex with FastAPI `BackgroundTasks`
The first version will schedule full Pokedex creation through `BackgroundTasks` after trainer initialization returns.

Rationale:
- Keeps `POST /trainers/initialize` responsive.
- Avoids adding Celery/RQ/worker infrastructure prematurely.
- Provides a clean path to a real queue later because the materialization logic stays inside a dedicated service method.

Alternative considered:
- Synchronous initialization. Rejected because it risks keeping an expensive bootstrap path forever.
- `asyncio.create_task(...)`. Rejected because lifecycle and error handling are looser than `BackgroundTasks`.

### Retry through the existing initialize endpoint
`POST /trainers/initialize` remains the only entry point for bootstrap and retry:
- New trainer: create trainer, choose starter, create starter `MyPokemon`, set `pokedex_status`, enqueue Pokedex initialization.
- Existing trainer: if `pokedex_status` is `EMPTY` or `FAILED`, enqueue only Pokedex initialization and do not recreate `MyPokemon`.

This keeps frontend wiring simple and avoids duplicating authorization, starter-selection, and retry policies across multiple routes.

Alternative considered:
- Add a dedicated `POST /trainers/{id}/initialize-pokedex`. Rejected to keep bootstrap semantics in one place for now.

### Make failed Pokedex initialization atomic per attempt
The background initialization will run in one transaction. If anything fails, the attempt rolls back completely and the trainer moves to `FAILED`.

Rationale:
- No partial Pokedex rows leak into user-visible state.
- Retry starts from a clean attempt.
- No extra `initialization_batch_id` field is needed on `Pokedex`.

Alternative considered:
- Persist partial rows and patch missing species later. Rejected because it complicates UX and makes readiness ambiguous.

### Preserve idempotency on successful retries
The initializer will guarantee that every base Pokemon has a corresponding `(trainer_id, pokemon_id)` `Pokedex` row, creating only missing rows and preserving existing valid rows when a retry is needed.

Rationale:
- Supports rerunning bootstrap safely after an interrupted or failed flow.
- Avoids accidental destruction of user progress if a later retry occurs after partial manual updates in future changes.

### Keep Pokedex gating independent from catalog completeness
`Pokedex` visibility rules depend only on `discovered`, not on `pokemon.status`.

Rationale:
- The Pokedex is framed as trainer knowledge, not catalog curation completeness.
- The user explicitly wants undiscovered items hidden the same way regardless of the underlying Pokemon status.

By contrast, `MyPokemon` detail gating depends on `my_pokemon.pokemon.status == COMPLETE`, because that detail page relies on the richer catalog-backed detail payload.

### Keep move ownership asymmetric
`MyPokemon` persists equipped moves selected at capture time. `Pokedex` does not persist its own move set and may only render possible species moves via the related `Pokemon`.

Rationale:
- Moves belong to a captured instance for battle usage.
- A species-level moveset in `Pokedex` would be ambiguous once multiple captured instances exist.

## Risks / Trade-offs

- [Background task reliability in a single API process] → Keep the initialization logic isolated behind a service boundary so it can move to a durable queue later without changing the API contract.
- [Heavy trainer bootstrap when the Pokemon catalog grows] → Keep the background path off the request-response critical path and make the initializer idempotent.
- [Two progression stores may drift semantically] → Define clear ownership: Pokedex is per-species, MyPokemon is per-instance, and nickname/moves stay out of Pokedex.
- [Retry complexity around existing trainers] → Centralize the decision in `TrainerService.initialize` using `pokedex_status` rather than distributed route logic.
- [Frontend confusion during `INITIALIZING` or `FAILED`] → Expose explicit states and require dedicated `/pokedex` loading/error UI with a retry action.

## Migration Plan

1. Add database support for `pokedex`, `my_pokemons`, trainer `pokedex_status`, and the `my_pokemon` to `pokemon_move` association table.
2. Introduce API models, repositories, and services for trainer bootstrap, Pokedex, and MyPokemon.
3. Extend the protected trainer initialize route to return the richer bootstrap response and enqueue background initialization.
4. Add protected Pokedex and MyPokemon endpoints plus route tests.
5. Add protected web pages, hooks, and modal update flow.
6. Roll out with the UI honoring `pokedex_status` so trainers created before the deployment can safely retry initialization.

Rollback strategy:
- Revert the web routes first if needed.
- Revert API handlers next.
- Keep schema rollback explicit through a follow-up migration if the feature has not yet been populated in production data.

## Open Questions

- None for this proposal. The product decisions required for the first implementation pass were resolved during exploration.
