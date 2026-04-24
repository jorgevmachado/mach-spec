## 1. Database and domain modeling

- [x] 1.1 Add `pokedex_status` to `Trainer` and create the new `Pokedex`, `MyPokemon`, and `my_pokemon_pokemon_moves` persistence structures with the required constraints and soft-delete fields.
- [x] 1.2 Generate and review the Alembic migration for trainer, Pokedex, MyPokemon, and move-association changes.
- [x] 1.3 Add domain schemas for trainer initialize response, Pokedex, and MyPokemon list/detail/update contracts.

## 2. Trainer bootstrap flow

- [x] 2.1 Extend `TrainerService.initialize` to select a starter species, create the starter `MyPokemon`, and drive `pokedex_status` transitions.
- [x] 2.2 Implement the background Pokedex initialization service flow with full-transaction rollback on failure and `READY`/`FAILED` status updates.
- [x] 2.3 Update the protected trainer initialize route wiring so it can trigger bootstrap and retry without recreating starter data.

## 3. Pokedex API

- [x] 3.1 Implement the `Pokedex` repository and service with trainer scoping, name filtering by `pokemon.name`, discovery gating, and soft delete.
- [x] 3.2 Add protected `Pokedex` list, detail, update, and delete routes wired into the API router tree.
- [x] 3.3 Add API tests covering initialization outcomes, discovery gating, filtering, retry behavior, and soft delete.

## 4. MyPokemon API

- [x] 4.1 Implement the `MyPokemon` repository and service with capture-time progression, equipped move persistence, and discovery updates for related Pokedex entries.
- [x] 4.2 Add protected `MyPokemon` list, detail, nickname update, and delete routes with trainer scoping and filter support for nickname and species name.
- [x] 4.3 Add API tests covering duplicate species captures, nickname validation, detail retrieval, filtering, and soft delete.

## 5. Protected web experience

- [x] 5.1 Add protected `/pokedex` and `/pokedex/[id]` pages, hooks, and components for loading, failed-retry, undiscovered, and discovered states.
- [x] 5.2 Add protected `/my-pokemon` and `/my-pokemon/[id]` pages, hooks, and components for list/detail rendering and guarded navigation.
- [x] 5.3 Implement the MyPokemon nickname edit modal with client validation aligned to the API contract.

## 6. Verification

- [x] 6.1 Add or update frontend tests for Pokedex states, MyPokemon gating, and nickname editing behavior.
- [x] 6.2 Run `make test` in `mach-api` and resolve any regressions.
- [x] 6.3 Run `npm run lint` and `npm run build` in `mach-web` and resolve any regressions.
