## 1. Backend Detail Contract

- [x] 1.1 Add bounded Pydantic schemas for Pokemon detail relationships in `mach-api`
- [x] 1.2 Return rich move and ability objects without Pokemon back-references
- [x] 1.3 Return rich type objects with `text_color`, `background_color`, `weaknesses`, and `strengths`
- [x] 1.4 Return evolutions as bounded Pokemon summaries with display-safe scalar fields

## 2. Frontend Details Experience

- [x] 2.1 Add guarded navigation from `COMPLETE` Pokemon cards on `/pokemon`
- [x] 2.2 Add a `usePokemonDetail` hook following the existing Pokemon service pattern
- [x] 2.3 Add the authenticated `/pokemon/[name]` page
- [x] 2.4 Render header, info, moves, abilities, types, weaknesses, strengths, stats, and evolution timeline sections

## 3. Verification

- [x] 3.1 Add or update backend tests for enriched Pokemon detail serialization
- [x] 3.2 Add or update frontend tests for guarded navigation and details rendering
- [x] 3.3 Run backend validation with `make test` in `mach-api`
- [x] 3.4 Run frontend validation with `npm run lint` and `npm run build` in `mach-web`
