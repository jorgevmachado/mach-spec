## MODIFIED Requirements

### Requirement: Protected Pokemon Detail Retrieval
The system SHALL expose a protected Pokemon detail endpoint that resolves a Pokemon by `name` or `id` from the local catalog and returns the enriched non-recursive detail contract required by the details experience.

#### Scenario: Detail includes rich move and ability data
- **WHEN** an authenticated client requests Pokemon detail for a completed Pokemon
- **THEN** the response includes move and ability objects with their own domain fields
- **AND** those related objects do not embed `pokemons` or other back-references

#### Scenario: Detail includes color-capable types with strengths and weaknesses
- **WHEN** an authenticated client requests Pokemon detail for a completed Pokemon
- **THEN** the response includes Pokemon type objects with `name`, `text_color`, and `background_color`
- **AND** each type includes `weaknesses` and `strengths` as non-recursive type summaries with the same color-capable fields

#### Scenario: Detail includes bounded evolution data
- **WHEN** an authenticated client requests Pokemon detail for a completed Pokemon
- **THEN** the response includes each related evolution as a bounded Pokemon summary with scalar display fields such as `id`, `name`, `order`, `status`, `image`, and `external_image`
- **AND** the evolution objects do not embed nested moves, abilities, types, or evolutions

### Requirement: Lazy Completion of Incomplete Pokemon
The system SHALL enrich an incomplete Pokemon from PokeAPI on demand when its protected detail endpoint is requested.

#### Scenario: Completion persists enriched relationships for detail usage
- **WHEN** an incomplete Pokemon is completed during a detail request
- **THEN** the system persists the related moves, abilities, types, strengths, weaknesses, growth rate, and evolution summaries required by the detail contract
