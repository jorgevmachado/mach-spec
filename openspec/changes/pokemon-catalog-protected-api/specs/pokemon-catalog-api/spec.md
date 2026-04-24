## ADDED Requirements

### Requirement: Protected Pokemon Catalog Listing
The system SHALL expose a protected Pokemon catalog listing endpoint that requires a valid API token and returns paginated results from the local database using the project’s standard pagination contract.

#### Scenario: Authenticated user lists Pokemon
- **WHEN** an authenticated client requests the Pokemon listing endpoint
- **THEN** the system returns a paginated response containing Pokemon items and pagination metadata

#### Scenario: Unauthenticated client lists Pokemon
- **WHEN** a client without a valid API token requests the Pokemon listing endpoint
- **THEN** the system rejects the request according to the existing authentication behavior

### Requirement: Pokemon Listing Filters
The system SHALL allow Pokemon listing results to be filtered by `name` and `order` without requiring any `trainer_id` or trainer-scoped parameter.

#### Scenario: Filter Pokemon by partial name
- **WHEN** an authenticated client requests the listing endpoint with a `name` filter
- **THEN** the system returns only Pokemon whose stored names match the filter criteria

#### Scenario: Filter Pokemon by order
- **WHEN** an authenticated client requests the listing endpoint with an `order` filter
- **THEN** the system returns only Pokemon whose stored order matches the provided value

#### Scenario: Trainer parameter is not part of the contract
- **WHEN** the Pokemon catalog capability is implemented
- **THEN** its listing and caching behavior do not require `trainer_id` or trainer-history side effects

### Requirement: Local Catalog Synchronization
The system SHALL maintain a local Pokemon catalog synchronized from PokeAPI and use local persistence as the source of truth for protected listing operations.

#### Scenario: Empty local catalog is initialized
- **WHEN** the local Pokemon catalog is empty and synchronization is triggered
- **THEN** the system fetches the external Pokemon list, persists base Pokemon records locally, and makes them available for listing

#### Scenario: Incomplete local catalog is filled
- **WHEN** the local Pokemon count is lower than the configured external total and synchronization is triggered
- **THEN** the system adds missing Pokemon records without duplicating existing entries

### Requirement: Protected Pokemon Detail Retrieval
The system SHALL expose a protected Pokemon detail endpoint that resolves a Pokemon by `name` or `id` from the local catalog.

#### Scenario: Fetch Pokemon detail by name
- **WHEN** an authenticated client requests Pokemon detail using a stored Pokemon name
- **THEN** the system returns that Pokemon from the local catalog

#### Scenario: Fetch Pokemon detail by id
- **WHEN** an authenticated client requests Pokemon detail using a stored Pokemon id
- **THEN** the system returns that Pokemon from the local catalog

#### Scenario: Pokemon detail is not found
- **WHEN** an authenticated client requests a Pokemon that does not exist in the local catalog
- **THEN** the system returns `404 Not Found`

### Requirement: Lazy Completion of Incomplete Pokemon
The system SHALL enrich an incomplete Pokemon from PokeAPI on demand when its protected detail endpoint is requested.

#### Scenario: Detail request completes incomplete Pokemon
- **WHEN** an authenticated client requests detail for a Pokemon whose status is `INCOMPLETE`
- **THEN** the system fetches external data, persists the enriched Pokemon and its relations, and returns the completed Pokemon

#### Scenario: External enrichment fails
- **WHEN** the system cannot retrieve or map required external data during completion
- **THEN** the system returns `502 Bad Gateway` with a descriptive error message
