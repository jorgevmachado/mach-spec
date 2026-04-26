## ADDED Requirements

### Requirement: API schema classes use a single naming convention
The system SHALL use a consistent schema naming convention across API domains based on `*Schema` suffixes for transport and filter contracts.

#### Scenario: Declaring a new schema class
- **WHEN** a domain introduces a new request, response, detail, update, list, or filter contract
- **THEN** the class name MUST follow the shared `*Schema` convention used by the mature API domains

### Requirement: Renamed schemas preserve route contract wiring
The system SHALL update response models, dependency annotations, and imports wherever a schema is renamed so that route contracts remain valid after the naming standardization.

#### Scenario: Schema rename in auth or trainer
- **WHEN** a schema in `auth` or `trainer` is renamed to match the shared convention
- **THEN** every route, service, and import site using that contract MUST be updated in the same change

### Requirement: Schema names avoid ambiguous generic transport labels
The system SHALL avoid domain-local naming patterns that depend on ambiguous `Request` and `Response` labels when an equivalent `*Schema` form can describe the contract more precisely.

#### Scenario: Exposing trainer initialization contracts
- **WHEN** the API exposes the initialize trainer input or output model
- **THEN** the schema names MUST communicate the contract purpose explicitly and align with the same suffix pattern used elsewhere in the API
