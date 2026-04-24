## ADDED Requirements

### Requirement: Pokedex stores trainer-scoped species progression
The system SHALL store one `Pokedex` record per `(trainer_id, pokemon_id)` and each record SHALL maintain species-level progression fields, battle counters, discovery state, and soft-delete support.

#### Scenario: Initializer creates one species entry per trainer
- **WHEN** Pokedex initialization runs for a trainer
- **THEN** the system creates at most one Pokedex record for each `(trainer_id, pokemon_id)` pair
- **AND** each record stores initialized progression values derived from the catalog species base stats

#### Scenario: Discovery timestamp is recorded once discovered
- **WHEN** a Pokedex record changes from `discovered = false` to `discovered = true`
- **THEN** the system sets `discovered_at`
- **AND** preserves the previously initialized progression fields

### Requirement: Pokedex list is trainer-scoped and discovery-aware
The system SHALL expose a protected Pokedex listing endpoint that returns only the authenticated trainer's Pokedex entries, supports filtering by species name, and includes enough data for discovered and undiscovered card states.

#### Scenario: Trainer receives only own Pokedex entries
- **WHEN** an authenticated trainer requests the Pokedex list
- **THEN** the response contains only records belonging to that trainer
- **AND** includes discovery state for each entry

#### Scenario: Name filter matches catalog species name
- **WHEN** an authenticated trainer requests the Pokedex list with a name filter
- **THEN** the system filters by the related `pokemon.name`
- **AND** applies the filter case-insensitively

### Requirement: Pokedex detail is gated by discovery
The system SHALL expose a protected Pokedex detail endpoint by Pokedex `id`, and the endpoint SHALL return the trainer-scoped Pokedex detail only when the entry is discovered.

#### Scenario: Discovered entry detail is returned
- **WHEN** an authenticated trainer requests a Pokedex detail by `id` for an entry with `discovered = true`
- **THEN** the response includes species progression, primary types, evolution timeline, and species-possible moves

#### Scenario: Undiscovered entry detail is blocked
- **WHEN** an authenticated trainer requests a Pokedex detail by `id` for an entry with `discovered = false`
- **THEN** the system rejects access to that detail

### Requirement: Pokedex supports update and soft delete
The system SHALL allow protected update and soft-delete operations on trainer-owned Pokedex entries.

#### Scenario: Pokedex update sets update timestamp
- **WHEN** an authenticated trainer updates a Pokedex entry
- **THEN** the system persists the changed fields
- **AND** sets `updated_at` according to the domain update rules

#### Scenario: Pokedex soft delete marks deletion without removing the row
- **WHEN** an authenticated trainer soft deletes a Pokedex entry
- **THEN** the system sets `deleted_at`
- **AND** does not physically remove the row from the table
