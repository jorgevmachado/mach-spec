## 1. Backend Catalog Models And Migration

- [x] 1.1 Add Pokemon catalog models in `mach-api/app/models` for `Pokemon`, `PokemonMove`, `PokemonAbility`, `PokemonType`, and `PokemonGrowthRate`
- [x] 1.2 Add Pokemon association tables for moves, abilities, types, type relations, and evolutions
- [x] 1.3 Export new model types and enums where needed so the new domain composes with the existing model package
- [x] 1.4 Generate and review the Alembic migration that creates Pokemon catalog tables and association tables in dependency-safe order

## 2. External PokeAPI Integration

- [x] 2.1 Add external API schemas in `mach-api/app/infrastructure/external_api/schemas.py` for Pokemon list, Pokemon detail, species, moves, types, growth rates, and evolution chains
- [x] 2.2 Implement `PokeApiClient` with async `httpx` calls and descriptive `502 Bad Gateway` error handling
- [x] 2.3 Reuse existing helper utilities to derive `order` and `external_image` values during catalog synchronization

## 3. Pokemon Domain And Protected Routes

- [x] 3.1 Add `PokemonRepository` with eager-loading configuration for Pokemon relations and default ordering by `order`
- [x] 3.2 Add Pokemon schemas for list/detail/filter use cases using the existing pagination/filter conventions
- [x] 3.3 Implement `PokemonService` for `total`, `initialize_list`, `list_sync`, `list`, `list_cached`, `get`, and `complete_pokemon`
- [x] 3.4 Remove `trainer_id` and any trainer-coupled behavior from Pokemon service contracts, cache flows, and route signatures
- [x] 3.5 Add protected Pokemon routes for paginated listing and detail retrieval using `get_current_user`

## 4. Related Resource APIs

- [x] 4.1 Add repositories, schemas, and services for Pokemon moves, abilities, types, and growth rates backed by local persistence
- [x] 4.2 Add protected routes for `/pokemon/moves`, `/pokemon/abilities`, `/pokemon/types`, and `/pokemon/growth-rates`
- [x] 4.3 Ensure related-resource APIs remain trainer-independent and rely only on authentication for access control

## 5. Frontend Pokemon Listing

- [x] 5.1 Update the protected `/pokemon` page in `mach-web` to render a paginated Pokemon list instead of the current placeholder
- [x] 5.2 Align frontend Pokemon types and service usage with the protected paginated backend contract
- [x] 5.3 Wire existing list filters for `name` and `order` into the `/pokemon` page without consuming related-resource endpoints

## 6. Verification

- [x] 6.1 Add or update backend tests for model behavior, repository filtering, protected routes, sync logic, and error handling
- [x] 6.2 Add or update frontend tests for Pokemon list filters and paginated rendering where the current frontend test layout supports it
- [ ] 6.3 Run backend validation with `make test` in `mach-api`
- [ ] 6.4 Run frontend validation with `npm run lint` and `npm run build` in `mach-web`
