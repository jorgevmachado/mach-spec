## Why

The project already exposes trainer initialization and Pokemon catalog flows, but it still has no trainer-owned collection model for species progress (`Pokedex`) and captured instances (`MyPokemon`). These features need to land together because trainer bootstrap, async Pokedex materialization, capture semantics, and the protected web pages all depend on the same domain rules.

## What Changes

- Add trainer-owned `Pokedex` records with per-species progression, discovery state, and protected list/detail APIs.
- Add trainer-owned `MyPokemon` records with independent progression, equipped moves, protected list/detail APIs, and nickname updates.
- Extend trainer initialization so it creates the starter `MyPokemon`, kicks off async Pokedex materialization, and exposes `pokedex_status` lifecycle states for frontend UX.
- Add protected `/pokedex` and `/my-pokemon` list and detail pages, including discovery gating, catalog-complete gating, and retry/preparation states.
- Keep Pokedex and MyPokemon intentionally separate: Pokedex tracks species progress, while MyPokemon tracks captured instances and editable nicknames.

## Capabilities

### New Capabilities
- `trainer-initialize-api`: Trainer bootstrap with starter selection, async Pokedex initialization, `pokedex_status`, and retry semantics through the existing initialize endpoint.
- `pokedex-api`: Trainer-scoped species progress model and protected endpoints for listing, reading, updating, and soft deleting Pokedex entries.
- `my-pokemon-api`: Trainer-scoped captured Pokemon model and protected endpoints for listing, reading, updating nickname, soft deleting, and capture-time move persistence.
- `pokedex-web`: Protected Pokedex list and detail experience with discovery gating, loading/error states tied to `pokedex_status`, and species-level progression rendering.
- `my-pokemon-web`: Protected My Pokemon list and detail experience with instance-level progression, equipped moves, nickname editing, and catalog-complete detail gating.

### Modified Capabilities
- None.

## Impact

- Affected backend areas: `mach-api/app/domain/trainer`, new `pokedex` and `my_pokemon` domains, SQLAlchemy models, migrations, progression logic, and protected route tests.
- Affected frontend areas: protected navigation targets, new `/pokedex` and `/my-pokemon` routes, data hooks/services, modal editing flow, and component tests.
- Operational impact: trainer initialization now includes a background task lifecycle and explicit readiness/error status for Pokedex materialization.
