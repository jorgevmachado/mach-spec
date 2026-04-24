## ADDED Requirements

### Requirement: MyPokemon stores trainer-owned captured instances
The system SHALL store `MyPokemon` as a trainer-owned captured instance that is independent from Pokedex progression and allows multiple records for the same `(trainer_id, pokemon_id)`.

#### Scenario: Trainer can own duplicate species instances
- **WHEN** a trainer captures the same species multiple times
- **THEN** the system creates distinct `MyPokemon` records for each capture

#### Scenario: Capture persists instance progression and moves
- **WHEN** a `MyPokemon` record is created
- **THEN** the system initializes instance progression fields
- **AND** selects and persists the equipped moves for that captured instance

### Requirement: Capture updates species discovery when needed
The system SHALL mark the related Pokedex entry as discovered when a trainer captures a species that is not yet discovered.

#### Scenario: Capture discovers an undiscovered species
- **WHEN** a trainer captures a species whose Pokedex entry has `discovered = false`
- **THEN** the system updates that Pokedex entry to `discovered = true`
- **AND** sets `discovered_at`

#### Scenario: Capture preserves already discovered species state
- **WHEN** a trainer captures a species whose Pokedex entry already has `discovered = true`
- **THEN** the system creates the new `MyPokemon` without resetting the existing Pokedex progression

### Requirement: MyPokemon list and detail are trainer-scoped
The system SHALL expose protected `MyPokemon` list and detail endpoints by `id`, scoped to the authenticated trainer, with filtering for both nickname and related species name.

#### Scenario: MyPokemon list supports nickname filter
- **WHEN** an authenticated trainer requests the MyPokemon list with a nickname filter
- **THEN** the system filters by `nickname`
- **AND** returns only matching records for that trainer

#### Scenario: MyPokemon list supports species name filter
- **WHEN** an authenticated trainer requests the MyPokemon list with a species-name filter
- **THEN** the system filters by related `pokemon.name`
- **AND** returns only matching records for that trainer

#### Scenario: MyPokemon detail returns captured-instance data
- **WHEN** an authenticated trainer requests a MyPokemon detail by `id`
- **THEN** the response includes nickname, instance progression, equipped moves, primary types, and evolution timeline

### Requirement: MyPokemon nickname updates are validated
The system SHALL allow updating only the nickname through the protected update flow, and the nickname SHALL be required to have at least 3 characters.

#### Scenario: Valid nickname update succeeds
- **WHEN** an authenticated trainer updates a MyPokemon nickname with a non-empty value of at least 3 characters
- **THEN** the system persists the new nickname
- **AND** updates `updated_at`

#### Scenario: Invalid nickname update is rejected
- **WHEN** an authenticated trainer updates a MyPokemon nickname with an empty value or fewer than 3 characters
- **THEN** the system rejects the request

### Requirement: MyPokemon supports soft delete
The system SHALL support protected soft delete for trainer-owned MyPokemon records.

#### Scenario: MyPokemon soft delete marks deletion without removing the row
- **WHEN** an authenticated trainer soft deletes a MyPokemon entry
- **THEN** the system sets `deleted_at`
- **AND** does not physically remove the row from the table
