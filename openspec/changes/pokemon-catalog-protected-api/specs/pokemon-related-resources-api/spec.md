## ADDED Requirements

### Requirement: Protected Pokemon Moves Endpoint
The system SHALL expose a protected endpoint for Pokemon moves backed by locally persisted move data.

#### Scenario: Authenticated user lists moves
- **WHEN** an authenticated client requests the Pokemon moves endpoint
- **THEN** the system returns move data from the local database

#### Scenario: Unauthenticated user lists moves
- **WHEN** a client without a valid API token requests the Pokemon moves endpoint
- **THEN** the system rejects the request according to the existing authentication behavior

### Requirement: Protected Pokemon Abilities Endpoint
The system SHALL expose a protected endpoint for Pokemon abilities backed by locally persisted ability data.

#### Scenario: Authenticated user lists abilities
- **WHEN** an authenticated client requests the Pokemon abilities endpoint
- **THEN** the system returns ability data from the local database

### Requirement: Protected Pokemon Types Endpoint
The system SHALL expose a protected endpoint for Pokemon types backed by locally persisted type data, including their configured color mapping and type relations.

#### Scenario: Authenticated user lists types
- **WHEN** an authenticated client requests the Pokemon types endpoint
- **THEN** the system returns type data from the local database

### Requirement: Protected Pokemon Growth Rates Endpoint
The system SHALL expose a protected endpoint for Pokemon growth rates backed by locally persisted growth rate data.

#### Scenario: Authenticated user lists growth rates
- **WHEN** an authenticated client requests the Pokemon growth rates endpoint
- **THEN** the system returns growth rate data from the local database

### Requirement: Related Pokemon Resources Are Trainer-Independent
The system SHALL manage Pokemon moves, abilities, types, and growth rates without requiring trainer-specific parameters or relationships.

#### Scenario: Related resources do not require trainer scope
- **WHEN** related-resource endpoints are called
- **THEN** the system resolves their data without `trainer_id`, trainer history, or Pokemon-to-trainer coupling
