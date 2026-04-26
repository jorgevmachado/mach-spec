## ADDED Requirements

### Requirement: Trainer initialization completes through a deterministic service flow
The system SHALL execute trainer bootstrap, starter capture, Pokedex initialization, and status transitions through a single predictable service workflow for `POST /trainers/initialize`.

#### Scenario: Successful first initialization
- **WHEN** an authenticated user without a trainer calls `POST /trainers/initialize` with valid bootstrap data
- **THEN** the system MUST create the trainer, capture the starter Pokemon, initialize the trainer Pokedex, update the user status, and return a coherent trainer initialization result

### Requirement: Pokedex status reflects the true initialization outcome
The system SHALL set `pokedex_status` based on the real result of initialization work rather than an optimistic response followed by detached background execution.

#### Scenario: Successful initialization status
- **WHEN** trainer bootstrap and Pokedex materialization complete successfully
- **THEN** the returned and persisted `pokedex_status` MUST be `READY`

#### Scenario: Failed initialization status
- **WHEN** trainer bootstrap or Pokedex materialization fails before completion
- **THEN** the system MUST surface a clear error and MUST NOT report `READY` for that attempt

### Requirement: Initialization failures are not hidden behind background tasks
The system SHALL not rely on FastAPI `BackgroundTasks` to perform the Pokedex initialization for `POST /trainers/initialize`.

#### Scenario: Request lifecycle for trainer initialization
- **WHEN** the initialize endpoint is invoked
- **THEN** the caller MUST receive the final success or failure result from the initialization flow within the same request lifecycle

### Requirement: Existing trainers re-enter initialization predictably
The system SHALL handle existing trainers with non-ready Pokedex state without duplicating trainer ownership data or returning misleading success states.

#### Scenario: Retry for trainer with failed or empty Pokedex
- **WHEN** an existing trainer with `pokedex_status` of `EMPTY` or `FAILED` calls `POST /trainers/initialize`
- **THEN** the system MUST retry the missing initialization work and return a status that reflects the actual outcome of that retry
