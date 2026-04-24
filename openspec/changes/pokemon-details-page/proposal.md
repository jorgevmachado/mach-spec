## Why

The Pokemon catalog change established protected list and detail endpoints plus the first `/pokemon` listing page, but it intentionally stopped short of a dedicated details experience. The current detail contract also returns only minimal relationship objects, which is not sufficient for a rich details screen that needs colored types, strengths, weaknesses, richer move and ability data, and bounded evolution display data.

## What Changes

- Extend the protected Pokemon detail contract with enriched but non-recursive relationship objects.
- Add guarded navigation from `/pokemon` cards so only Pokemon with status `COMPLETE` open a details page.
- Add an authenticated `/pokemon/[name]` details page in `mach-web`.
- Add a `usePokemonDetail` hook that follows the same frontend service pattern already used for list retrieval.
- Keep the details contract bounded by using summary/detail schemas that exclude recursive back-references.

## Capabilities

### New Capabilities
- `pokemon-detail-web`: Authenticated Pokemon details page with guarded entry from the Pokemon list and rich rendering of the protected detail response.

### Modified Capabilities
- `pokemon-catalog-api`: Protected Pokemon detail retrieval now returns richer non-recursive related objects needed by the frontend details experience.
- `pokemon-list-web`: The authenticated Pokemon list now supports guarded navigation to details for `COMPLETE` entries.

## Impact

- Affected backend areas: `mach-api/app/domain/pokemon`, related schemas, repository loading, and detail-route tests.
- Affected frontend areas: protected Pokemon list page interactions, new protected detail route, Pokemon service types, and detail tests.
- No auth scope change: the feature continues to rely on existing token authentication.
