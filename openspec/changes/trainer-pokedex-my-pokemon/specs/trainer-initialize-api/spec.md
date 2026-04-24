## ADDED Requirements

### Requirement: Trainer initialize bootstraps starter and Pokedex lifecycle
The system SHALL use the existing protected trainer initialize endpoint to bootstrap trainer-owned Pokemon progress. The endpoint SHALL create the trainer when absent, choose a starter species, create exactly one starter `MyPokemon`, set trainer `pokedex_status`, and start asynchronous Pokedex materialization.

#### Scenario: New trainer initialize starts bootstrap
- **WHEN** an authenticated user without a trainer calls the initialize endpoint
- **THEN** the system creates the trainer record
- **AND** creates exactly one starter `MyPokemon` for that trainer
- **AND** sets `pokedex_status` to `INITIALIZING`
- **AND** schedules full Pokedex initialization in the background

#### Scenario: Starter discovery is applied during bootstrap
- **WHEN** the background Pokedex initialization materializes entries for a new trainer
- **THEN** every Pokemon species in the catalog receives a trainer-scoped Pokedex entry
- **AND** only the starter species is marked `discovered = true`
- **AND** all other species are created with `discovered = false`

### Requirement: Trainer initialize exposes Pokedex lifecycle states
The system SHALL persist trainer `pokedex_status` with the values `EMPTY`, `INITIALIZING`, `READY`, and `FAILED`, and SHALL expose that state in the trainer initialize contract so the web application can render bootstrap progress and retry states.

#### Scenario: Successful background initialization marks readiness
- **WHEN** the background Pokedex initialization completes successfully
- **THEN** the trainer `pokedex_status` becomes `READY`

#### Scenario: Failed background initialization marks failure
- **WHEN** the background Pokedex initialization fails
- **THEN** the trainer `pokedex_status` becomes `FAILED`
- **AND** the failed attempt does not leave partial persisted Pokedex rows

### Requirement: Trainer initialize supports idempotent retry
The system SHALL allow the existing initialize endpoint to retry only the Pokedex initialization for an existing trainer whose `pokedex_status` is `EMPTY` or `FAILED`, without recreating the trainer or starter `MyPokemon`.

#### Scenario: Retry restarts failed Pokedex initialization
- **WHEN** an authenticated user with an existing trainer and `pokedex_status = FAILED` calls the initialize endpoint
- **THEN** the system does not create another trainer
- **AND** does not create another starter `MyPokemon`
- **AND** sets `pokedex_status` to `INITIALIZING`
- **AND** schedules Pokedex initialization again

#### Scenario: Ready trainer is not reinitialized
- **WHEN** an authenticated user with an existing trainer and `pokedex_status = READY` calls the initialize endpoint
- **THEN** the system does not reinitialize the Pokedex
- **AND** does not create additional starter `MyPokemon` records
