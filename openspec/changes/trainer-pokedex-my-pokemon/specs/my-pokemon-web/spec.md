## ADDED Requirements

### Requirement: Protected MyPokemon list page
The web application SHALL provide an authenticated `/my-pokemon` page that renders trainer-owned captured instances with nickname, species identity, progression summary, and type badges.

#### Scenario: MyPokemon card shows required summary data
- **WHEN** the MyPokemon list page renders a captured instance
- **THEN** the card shows `nickname`, related species name, level, wins, losses, and primary type badges

#### Scenario: Image rendering respects catalog completeness
- **WHEN** the MyPokemon list page renders a captured instance whose related `pokemon.status` is not `COMPLETE`
- **THEN** the card does not require the full real detail presentation needed for a complete species

### Requirement: MyPokemon detail route is gated by catalog completeness
The web application SHALL provide an authenticated `/my-pokemon/[id]` page that opens by MyPokemon `id` only when the related species status is `COMPLETE`.

#### Scenario: Complete species detail opens
- **WHEN** an authenticated user navigates from a MyPokemon card whose related `pokemon.status` is `COMPLETE`
- **THEN** the application loads the protected MyPokemon detail by `id`
- **AND** renders nickname as the main title, species name as subtitle, instance progression, equipped moves, types, and evolution timeline

#### Scenario: Incomplete species card does not open detail
- **WHEN** an authenticated user interacts with a MyPokemon card whose related `pokemon.status` is not `COMPLETE`
- **THEN** the application does not navigate to the detail page

### Requirement: MyPokemon nickname can be edited from the detail page
The web application SHALL allow nickname updates through a modal on the MyPokemon detail page and SHALL enforce the same minimum validation used by the API.

#### Scenario: Valid nickname update flows through modal
- **WHEN** an authenticated user submits a nickname with at least 3 characters from the modal
- **THEN** the application sends the protected update request
- **AND** refreshes the rendered nickname after success

#### Scenario: Invalid nickname is blocked in the UI
- **WHEN** an authenticated user submits an empty nickname or a value shorter than 3 characters from the modal
- **THEN** the application blocks submission and shows a validation error
