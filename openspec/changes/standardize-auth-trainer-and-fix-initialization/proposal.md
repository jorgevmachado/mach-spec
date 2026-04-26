## Why

The `trainer` and `auth` domains drifted from the architectural conventions already established by `pokemon`, which makes the API harder to maintain and obscures responsibilities across routes, services, repositories, and schemas. At the same time, `POST /trainers/initialize` currently reports `pokedex_status=FAILED` in cases that should complete successfully, and the current `BackgroundTasks` flow hides the real cause of failure instead of providing a predictable initialization contract.

## What Changes

- Standardize the `auth` and `trainer` domains to follow the same layered structure, naming rules, and dependency boundaries already used by `pokemon`.
- Rename and reorganize schema classes across the API so that request, response, detail, list, update, and filter contracts follow a single naming convention.
- Refactor trainer initialization so that starter capture, Pokedex materialization, and `pokedex_status` transitions are executed through a predictable service workflow with explicit error handling.
- Remove the current `BackgroundTasks` dependency from `POST /trainers/initialize` and replace it with a simpler initialization path that does not hide failures.
- Reorganize tests so `auth` and `trainer` coverage follows the domain-oriented test layout used by the repository conventions.

## Capabilities

### New Capabilities
- `domain-architecture-standardization`: Structural requirements for `auth` and `trainer` so both domains follow the same route/service/repository/schema responsibilities and test layout conventions as the mature `pokemon` domain.
- `schema-naming-consistency`: Consistent naming rules for API schema classes and imports across domains, including trainer and auth contracts.
- `trainer-initialize-api`: Predictable trainer initialization behavior for starter creation, Pokedex initialization, status transitions, retry handling, and explicit error reporting through the existing initialize endpoint.

### Modified Capabilities

None.

## Impact

- Affected backend areas: `mach-api/app/domain/auth`, `mach-api/app/domain/trainer`, `mach-api/app/domain/pokedex`, shared imports, and protected route wiring.
- Affected tests: `mach-api/tests/app/domain/auth`, `mach-api/tests/app/domain/trainer`, and current combined auth/trainer coverage.
- API impact: `POST /trainers/initialize` keeps the same endpoint but changes execution semantics to be synchronous and deterministic.
- Operational impact: initialization failures become observable and actionable instead of being hidden behind a background task lifecycle.
