## Why

The repository already has early Pokemon-related utilities, frontend navigation, and list client scaffolding, but the backend catalog and protected endpoints do not exist yet. This change establishes a coherent first delivery for authenticated Pokemon browsing without introducing trainer-owned collections, pokedex flows, or other premature coupling.

## What Changes

- Add a protected Pokemon catalog API backed by local persistence and synchronized from PokeAPI.
- Introduce Pokemon domain models, related entities, association tables, and migrations in `mach-api`.
- Add async external API client schemas and integration logic for Pokemon, moves, abilities, types, growth rates, and evolutions.
- Expose protected routes for:
  - paginated Pokemon listing with filtering by `name` and `order`
  - Pokemon detail by `name` or `id`
  - related resource listing for moves, abilities, types, and growth rates
- Require API token authentication for all Pokemon routes only to validate that the caller is a registered user.
- Remove `trainer_id` and any trainer-history or trainer-coupled behavior from Pokemon service contracts.
- Deliver the first frontend slice as a paginated `/pokemon` listing page using the project’s existing list pagination pattern.
- Keep `/pokedex` and `/my-pokemon` out of scope for this change.

## Capabilities

### New Capabilities
- `pokemon-catalog-api`: Protected Pokemon catalog endpoints, local persistence, pagination, filtering, detail retrieval, and sync from PokeAPI.
- `pokemon-related-resources-api`: Protected endpoints and backend support for Pokemon moves, abilities, types, and growth rates.
- `pokemon-list-web`: Authenticated frontend Pokemon list page with paginated browsing and filters for `name` and `order`.

### Modified Capabilities

None.

## Impact

- Affected backend areas: `mach-api/app/models`, `mach-api/app/domain`, `mach-api/app/infrastructure`, routes, migrations, cache usage, and tests.
- Affected frontend areas: protected Pokemon page, Pokemon service types, and list integration in `mach-web`.
- External dependency: `https://pokeapi.co` via async `httpx` client.
- Authentication impact: Pokemon routes require existing token-based user authentication, but no trainer-specific dependency is introduced.
