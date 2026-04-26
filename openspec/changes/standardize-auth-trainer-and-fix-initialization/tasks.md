## 1. Schema and contract standardization

- [x] 1.1 Rename `auth` and `trainer` schema classes to the shared `*Schema` convention and update all imports, dependency annotations, and route response models.
- [x] 1.2 Review related schema names across the API for consistency with existing `pokemon`, `pokedex`, and `my_pokemon` naming patterns.

## 2. Domain architecture alignment

- [x] 2.1 Refactor `auth` routes and services so route handlers delegate through service contracts instead of returning repository-backed data directly.
- [x] 2.2 Refactor `trainer` routes and services so `/me` and initialization behavior follow the same route/service/repository responsibility split used by `pokemon`.
- [x] 2.3 Split the combined auth/trainer tests into `tests/app/domain/auth/` and `tests/app/domain/trainer/` with separate service and route coverage.

## 3. Trainer initialization fix

- [x] 3.1 Remove `BackgroundTasks` from `POST /trainers/initialize` and the detached initialization helper that creates its own database session.
- [x] 3.2 Implement a deterministic trainer bootstrap flow that creates or reuses the trainer, captures the starter, initializes the Pokedex, and applies explicit `INITIALIZING` and `READY` status transitions.
- [x] 3.3 Ensure initialization failures surface clear errors and do not return misleading `pokedex_status` values.

## 4. Verification

- [x] 4.1 Add or update API tests covering successful first initialization, retry behavior for `EMPTY` and `FAILED`, and failure propagation from Pokedex initialization.
- [x] 4.2 Add or update API tests covering `/auth/me`, `/trainers/me`, and schema rename regressions after the refactor.
- [x] 4.3 Run `make test` in `mach-api` and resolve regressions introduced by the refactor.
