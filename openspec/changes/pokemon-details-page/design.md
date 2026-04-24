## Context

`mach-web` already has an authenticated `/pokemon` list page and a service abstraction for Pokemon API access. `mach-api` already exposes `GET /pokemon/{name_or_id}`, but the current detail payload is optimized for catalog completion rather than for direct UI consumption. The frontend prompt for the details page requires richer relationship data and guarded navigation based on Pokemon completion status.

## Goals / Non-Goals

**Goals:**
- Add a dedicated authenticated `/pokemon/[name]` details page.
- Allow navigation to that page only from Pokemon cards whose status is `COMPLETE`.
- Extend the Pokemon detail payload so the frontend can render moves, abilities, colorized types, weaknesses, strengths, growth rate, and evolutions without extra calls or frontend hardcoding.
- Keep the frontend aligned with the existing service and hook patterns.

**Non-Goals:**
- No change to the list pagination contract.
- No recursive response graph that embeds nested Pokemon relationships indefinitely.
- No new auth behavior or trainer coupling.
- No `/pokedex` or `/my-pokemon` work.

## Decisions

### 1. Details entry is guarded by Pokemon status
The list page will only navigate to `/pokemon/[name]` when the clicked Pokemon has status `COMPLETE`.

Rationale:
- The product intent is explicit in the prompt.
- Incomplete Pokemon do not guarantee a detail payload rich enough for the UI until completion happens.
- The list already exposes status, so the guard can be implemented without additional requests.

Alternatives considered:
- Allowing navigation for every Pokemon and relying on the detail endpoint to complete on demand. Rejected because the requested interaction explicitly uses status as the entry guard and would create a confusing difference between clickable and non-clickable cards only after navigation.

### 2. Detail responses use rich but non-recursive schemas
`GET /pokemon/{name_or_id}` will return a richer payload than the list endpoint, but every relationship will use a bounded schema that excludes nested Pokemon graphs and back-references.

Contract shape:
- `moves`: rich move objects without `pokemons`
- `abilities`: rich ability objects without `pokemons`
- `types`: rich type objects with `text_color`, `background_color`, `weaknesses`, and `strengths`
- `weaknesses` and `strengths`: type summaries with color fields
- `evolutions`: Pokemon summaries with display-safe scalar fields such as `id`, `name`, `order`, `status`, `image`, and `external_image`
- `growth_rate`: growth-rate detail without back-references

Rationale:
- The details page needs backend-owned type colors and relation data.
- The move and ability sections can reuse the already persisted domain fields instead of collapsing everything to `{ id, name }`.
- Bounded schemas prevent recursive serialization, oversized payloads, and ambiguous contracts.

Alternatives considered:
- Keeping the current minimal detail contract and forcing the frontend to derive colors or call additional endpoints. Rejected because it duplicates domain knowledge and fragments the page load.
- Returning full ORM-shaped objects for every relation. Rejected because it risks recursive payloads and unstable serialization behavior.

### 3. Frontend follows the existing service pattern
The details page will use a dedicated `usePokemonDetail` hook and the existing Pokemon service abstraction rather than introducing a separate data-loading pattern.

Rationale:
- The codebase already has a recognizable list retrieval pattern.
- Reusing the service layer keeps authenticated API access and error handling consistent.
- The prompt explicitly asks for the same pattern used by `usePokemonList`.

Alternatives considered:
- Fetching directly in the page component. Rejected because it bypasses the established client-service pattern already used in the feature.

## Risks / Trade-offs

- [Detail payloads can become too heavy] → Mitigate by returning rich but bounded schemas and avoiding recursive relations.
- [Frontend interaction may imply incompletes are broken] → Mitigate by making the card affordance clearly conditional on `COMPLETE` status.
- [Backend schema changes may affect existing detail consumers] → Mitigate by extending the contract compatibly and preserving existing scalar field names.

## Migration Plan

1. Add non-recursive detail schemas for Pokemon relationships in `mach-api`.
2. Update Pokemon detail loading/serialization to return enriched related data.
3. Add backend tests for enriched detail payloads and bounded evolution/type relations.
4. Add a frontend detail hook and authenticated `/pokemon/[name]` page.
5. Add guarded navigation from the protected `/pokemon` list page.
6. Validate backend tests plus frontend lint/build and detail-page tests.

## Open Questions

- Whether the details page should display species flavor text and category now or defer until the backend models those fields explicitly.
- Whether evolution timeline items should navigate to other complete Pokemon later, or remain display-only in this first version.
