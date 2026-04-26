## ADDED Requirements

### Requirement: Auth and trainer domains follow the reference layering
The system SHALL structure the `auth` and `trainer` domains using the same responsibility boundaries as the `pokemon` domain, where routes delegate use cases, services orchestrate domain logic, repositories own persistence access, and schemas define transport contracts.

#### Scenario: Route delegates to service
- **WHEN** an `auth` or `trainer` API route handles a request
- **THEN** it MUST delegate the use case to a domain service instead of executing business or repository logic directly

#### Scenario: Repository access stays behind domain service
- **WHEN** a route needs trainer or auth data beyond authentication dependency injection
- **THEN** the route MUST obtain that data through a domain service contract rather than calling repository methods directly

### Requirement: Domain modules keep infrastructure concerns out of orchestration helpers
The system SHALL keep infrastructure-specific concerns such as ad-hoc session creation and request-lifecycle task wiring out of domain orchestration helpers inside `auth` and `trainer`.

#### Scenario: Trainer initialization helper behavior
- **WHEN** trainer bootstrap requires Pokedex initialization or status updates
- **THEN** the orchestration MUST occur through service-owned dependencies in the active application flow rather than through detached helper functions that construct their own database session

### Requirement: Auth and trainer tests follow domain-oriented layout
The system SHALL keep `auth` and `trainer` tests in dedicated domain directories with route- and service-focused coverage aligned to repository conventions.

#### Scenario: Adding auth or trainer regression coverage
- **WHEN** new tests are created or adjusted for `auth` or `trainer`
- **THEN** they MUST live under `tests/app/domain/auth/` or `tests/app/domain/trainer/` instead of a combined cross-domain file
