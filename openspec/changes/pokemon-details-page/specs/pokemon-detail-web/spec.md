## ADDED Requirements

### Requirement: Authenticated Pokemon Details Page
The web application SHALL provide an authenticated `/pokemon/[name]` page that consumes the protected Pokemon detail API using the same service pattern already used by the Pokemon listing flow.

#### Scenario: User opens a Pokemon details page
- **WHEN** an authenticated user navigates to `/pokemon/[name]`
- **THEN** the page requests the protected Pokemon detail endpoint through the frontend service layer
- **AND** renders the returned Pokemon header, stats, moves, abilities, types, weaknesses, strengths, and evolutions

#### Scenario: Type badges use backend-provided colors
- **WHEN** the details page renders Pokemon types, weaknesses, or strengths
- **THEN** each badge uses the `text_color` and `background_color` values provided by the API response
