## Context

The API already has a mature reference domain in `app/domain/pokemon`, where routes are thin, repositories stay focused on persistence, services own orchestration, and schemas follow a stable naming pattern. The `auth` and `trainer` domains have the same top-level files, but they do not follow the same level of separation: routes still reach directly into repositories, schema names drift between `Request`/`Response` and `*Schema`, and trainer initialization mixes domain orchestration with request-lifecycle infrastructure.

The highest-risk path is `POST /trainers/initialize` in `app/domain/trainer/service.py`. Today it can enqueue Pokedex initialization through FastAPI `BackgroundTasks`, create a second session inside a helper, and collapse any failure into `FAILED` without preserving a clear error boundary. Existing tests pass because they only validate service-level enqueueing, not the real background execution path.

## Goals / Non-Goals

**Goals:**
- Align `auth` and `trainer` with the layering and dependency boundaries used by `pokemon`.
- Establish one schema naming convention across the API and update imports accordingly.
- Make `POST /trainers/initialize` deterministic: successful initialization returns coherent state, failed initialization raises explicit errors, and `pokedex_status` reflects the true result.
- Keep the implementation simple enough to fit the current FastAPI async architecture without introducing new infrastructure.
- Reorganize tests to follow the per-domain layout expected by the repository conventions.

**Non-Goals:**
- Redesign the `pokemon`, `pokedex`, or `my_pokemon` domains beyond what is needed to support the initialization fix.
- Introduce a distributed queue, worker process, or event-driven orchestration framework.
- Change the public route shape of `/auth/*` or `/trainers/*` beyond response model naming and corrected initialization semantics.
- Create a brand-new architectural pattern separate from the one already present in `pokemon`.

## Decisions

### Use `pokemon` as the architectural reference, not just as inspiration
`auth` and `trainer` will be refactored to match the route/service/repository/schema separation already present in `pokemon`.

Rationale:
- The repository already documents that structure as the desired standard.
- Reusing the existing pattern reduces debate and keeps future domains consistent.

Alternative considered:
- Preserve the current files and apply only minimal bug fixes. Rejected because it would leave the structural drift in place and make the initialization bug easier to reintroduce.

### Standardize schema names around `*Schema`
Schema classes will follow a consistent pattern such as `RegisterSchema`, `LoginSchema`, `TrainerInitializeSchema`, `TrainerInitializeResultSchema`, `TrainerMeSchema`, `*DetailSchema`, `*UpdateSchema`, and `*FilterPageSchema`.

Rationale:
- `pokemon`, `pokedex`, and `my_pokemon` already establish `*Schema` as the dominant convention.
- This removes ambiguity between transport contracts and implementation classes.

Alternative considered:
- Keep `Request`/`Response` names in `auth` and `trainer`. Rejected because it preserves inconsistency and forces special cases in imports and mental models.

### Remove `BackgroundTasks` from trainer initialization
`POST /trainers/initialize` will execute trainer bootstrap and Pokedex initialization inside the service flow rather than enqueueing background work.

Rationale:
- The current task-based flow hides failures and depends on a second session detached from the request flow.
- The initialization logic is important enough that the caller should receive a real success or failure result, not an optimistic acknowledgment followed by silent fallback to `FAILED`.
- This is the simplest path that satisfies the prompt without introducing extra infrastructure.

Alternative considered:
- Keep `BackgroundTasks` and improve logging or retries. Rejected because observability would improve, but the endpoint would still be nondeterministic and harder to validate.
- Replace `BackgroundTasks` with `asyncio.create_task`. Rejected because it keeps the same lifecycle ambiguity with weaker guarantees.

### Keep trainer initialization orchestration in `TrainerService`
`TrainerService` remains the orchestration point for trainer bootstrap, but infrastructure-specific helpers such as ad-hoc session creation will be removed. `PokedexService` continues to own Pokedex materialization, while `TrainerService` coordinates calls and status transitions.

Rationale:
- This preserves domain boundaries already implied by the current modules.
- It avoids duplicating initialization rules across route handlers or repositories.

Alternative considered:
- Move all bootstrap behavior into `PokedexService`. Rejected because trainer creation, user status updates, and starter capture are not Pokedex responsibilities.

### Make status transitions explicit and failure-first
The flow will set `INITIALIZING` before dependent work starts, set `READY` only after Pokedex creation succeeds, and surface exceptions clearly when initialization fails. If failure persistence is retained, it must happen in the same controlled service flow rather than a detached task.

Rationale:
- The current status lifecycle exists for a reason; the issue is reliability, not the existence of the enum.
- Explicit transitions make tests and frontend behavior straightforward.

Alternative considered:
- Remove `pokedex_status` entirely and infer readiness from related rows. Rejected because the frontend already relies on explicit states and the repository uses those semantics.

### Split `auth` and `trainer` tests by domain
The combined `tests/app/domain/test_auth_and_trainer.py` file will be replaced by per-domain route/service tests.

Rationale:
- It matches the repository’s documented testing structure.
- It isolates failures and keeps architectural ownership clear.

Alternative considered:
- Keep the combined test file and append new cases. Rejected because it preserves the same structural inconsistency the change is meant to remove.

## Risks / Trade-offs

- [Synchronous initialization increases request time] → Keep the orchestration simple, reuse existing services, and accept a slower but deterministic bootstrap path over a faster opaque one.
- [Schema renames can break imports broadly] → Apply a full import sweep and update response models together with the schema classes.
- [Refactoring route/service boundaries can change serialization behavior] → Add focused route tests for `/auth/me`, `/trainers/me`, and `/trainers/initialize`.
- [Existing frontend assumptions may expect optimistic success] → Preserve the endpoint shape while returning coherent final status and explicit errors.

## Migration Plan

1. Rename and reorganize schema classes in `auth` and `trainer`, then update imports and route response models.
2. Refactor `auth` and `trainer` route/service boundaries to match the `pokemon` layering pattern.
3. Remove `BackgroundTasks` from trainer initialization and execute Pokedex materialization in the service flow with explicit status transitions.
4. Adjust eager loading and persistence helpers needed by the new initialization path.
5. Split the current combined auth/trainer tests into per-domain service and route files and add coverage for initialization success and failure.

Rollback strategy:
- Revert the domain refactor and restore current schema names if import churn causes unacceptable regressions.
- Revert the initialization flow as one unit if the synchronous path proves unstable, rather than partially restoring background execution.

## Open Questions

- Whether the final naming pattern should preserve `Request`/`Response` for external transport classes or fully converge on `*Schema` everywhere. The proposal assumes full convergence because it matches the broader codebase more closely.
