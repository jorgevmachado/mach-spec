## ADDED Requirements

### Requirement: Protected Pokedex list page
The web application SHALL provide an authenticated `/pokedex` page that renders every Pokedex entry for the current trainer and reflects both discovery state and trainer `pokedex_status`.

#### Scenario: Undiscovered entries stay visually hidden
- **WHEN** the Pokedex list page renders an entry with `discovered = false`
- **THEN** the card shows a grayed species name
- **AND** renders a Pokeball placeholder instead of the real image
- **AND** shows a `Not Discovered` badge

#### Scenario: Discovered entries show species data
- **WHEN** the Pokedex list page renders an entry with `discovered = true`
- **THEN** the card shows the real species image and name
- **AND** renders the primary types as badges
- **AND** may show progression summary fields such as level, wins, losses, or discovered date

### Requirement: Pokedex list reacts to trainer Pokedex lifecycle
The web application SHALL render dedicated states for `INITIALIZING` and `FAILED` using trainer `pokedex_status`.

#### Scenario: Preparing state is shown while background initialization runs
- **WHEN** the current trainer has `pokedex_status = INITIALIZING`
- **THEN** the `/pokedex` page remains accessible
- **AND** shows a preparation/loading state instead of pretending the Pokedex is ready

#### Scenario: Retry action is shown after failed initialization
- **WHEN** the current trainer has `pokedex_status = FAILED`
- **THEN** the `/pokedex` page shows an error state
- **AND** exposes an action that calls the existing trainer initialize endpoint again

### Requirement: Pokedex detail route is discovery-gated
The web application SHALL provide an authenticated `/pokedex/[id]` page that opens only for discovered entries.

#### Scenario: User opens discovered Pokedex detail
- **WHEN** an authenticated user navigates from a discovered Pokedex card
- **THEN** the application loads the protected Pokedex detail by `id`
- **AND** renders image, name, primary types, progression, possible moves, and evolution timeline

#### Scenario: Undiscovered card does not navigate
- **WHEN** an authenticated user interacts with an undiscovered Pokedex card
- **THEN** the application does not navigate to the detail page
