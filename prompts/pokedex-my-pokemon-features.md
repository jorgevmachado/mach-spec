# OpenSpec Prompt — Pokedex My Pokemon Features

## Context

This project uses a monorepo with two submodules:

- **mach-api** — Python 3.13, FastAPI, SQLAlchemy async, domain-driven architecture
- **mach-web** — Next.js 16, React 19, TypeScript, Tailwind CSS v4

---
## SPEC: mach-api
### 1. Database Models

#### 1.1 Model `Pokedex`
File location: `mach-api/src/models/pokedex.py`

Create a SQLAlchemy async model named `Pokedex` mapped to the table `pokedex`. with the following columns:

| Column            | Type    | Constraints                                                           |
|-------------------|---------|-----------------------------------------------------------------------|
| `id`              | UUID    | Primary key, default `uuid4`, not nullable                            |
| `hp`              | Integer | not nullable                                                          |
| `wins`            | Integer | not nullable                                                          |
| `level`           | Integer | not nullable                                                          |
| `losses`          | Integer | not nullable                                                          |
| `max_hp`          | Integer | not nullable                                                          |
| `battles`         | Integer | not nullable                                                          |
| `nickname`        | String  | not nullable                                                          |
| `speed`           | Integer | not nullable                                                          |
| `attack`          | Integer | not nullable                                                          |
| `defense`         | Integer | not nullable                                                          |
| `experience`      | Integer | not nullable                                                          |
| `special_attack`  | Integer | not nullable                                                          |
| `special_defense` | Integer | not nullable                                                          |
| `discovered`      | Boolean | Nullable, default `False`                                             |
| `formula`         | String  | not nullable                                                          |
| `discovered_at`   | DateTime | Nullable, default `None`, **only set on change discovered attribute** |
| `created_at`      | DateTime | Not nullable, default `utcnow`, **never updated after insert**        |
| `updated_at`      | DateTime | Nullable, default `None`, **only set on second or later updates**     |
| `deleted_at`      | DateTime | Nullable, default `None`, **only set when soft-delete is triggered**  |
| `pokemon_id`      | UUID | Foreign key → `pokemons.id`, Not nullable                             |
| `trainer_id`      | UUID | Foreign key → `trainers.id`, Not nullable                             |

**Relationships on `Pokedex`:**
- `pokemon`: One-to-one relationship with `Pokemon` model, accessed via `pokedex.pokemon`
- `trainer`: One-to-one relationship with `Trainer` model, accessed via `pokedex.trainer`

**Behavior rules for `Pokedex`:**
- `created_at` is set only at creation and must never be changed afterwards.
- `updated_at` starts as `None`. It must be set to the current UTC datetime only on the **second or later modification** of the record.
- `deleted_at` must only be set when a soft-delete operation is explicitly triggered. Soft-deleting must **not** physically remove the row.
- `discovered_at` is set only when `discovered` attribute is changed from `False` to `True`.

---

#### 1.2 Model `MyPokemon`
File location: `mach-api/src/models/my_pokemon.py`

Create a SQLAlchemy async model named `MyPokemon` mapped to the table `my_pokemons`. with the following columns:


| Column            | Type    | Constraints                                                           |
|-------------------|---------|-----------------------------------------------------------------------|
| `id`              | UUID    | Primary key, default `uuid4`, not nullable                            |
| `hp`              | Integer | not nullable                                                          |
| `wins`            | Integer | not nullable                                                          |
| `level`           | Integer | not nullable                                                          |
| `losses`          | Integer | not nullable                                                          |
| `max_hp`          | Integer | not nullable                                                          |
| `battles`         | Integer | not nullable                                                          |
| `nickname`        | String  | not nullable                                                          |
| `speed`           | Integer | not nullable                                                          |
| `attack`          | Integer | not nullable                                                          |
| `defense`         | Integer | not nullable                                                          |
| `experience`      | Integer | not nullable                                                          |
| `special_attack`  | Integer | not nullable                                                          |
| `special_defense` | Integer | not nullable                                                          |
| `formula`         | String  | not nullable                                                          |
| `capture_at`      | DateTime | not nullable, default `utcnow`, **never updated after insert**        |
| `created_at`      | DateTime | Not nullable, default `utcnow`, **never updated after insert**        |
| `updated_at`      | DateTime | Nullable, default `None`, **only set on second or later updates**     |
| `deleted_at`      | DateTime | Nullable, default `None`, **only set when soft-delete is triggered**  |
| `pokemon_id`      | UUID | Foreign key → `pokemons.id`, Not nullable                             |
| `trainer_id`      | UUID | Foreign key → `trainers.id`, Not nullable                             |

**Relationships on `MyPokemon`:**
- `pokemon`: One-to-one relationship with `Pokemon` model, accessed via `my_pokemon.pokemon`
- `trainer`: One-to-one relationship with `Trainer` model, accessed via `my_pokemon.trainer`

**Behavior rules for `MyPokemon`:**
- `capture_at` is set only when the record is created.
- `created_at` is set only at creation and must never be changed afterwards.
- `updated_at` starts as `None`. It must be set to the current UTC datetime only on the **second or later modification** of the record.
- `deleted_at` must only be set when a soft-delete operation is explicitly triggered. Soft-deleting must **not** physically remove the row.

---

#### 1.3 Model `Trainer`
File location: `mach-api/src/models/trainer.py`

Add Realationships to `Trainer` model:
- `pokedex`: One-to-many relationship with `Pokedex` model, accessed via `trainer.pokedex`
- `my_pokemons`: One-to-many relationship with `MyPokemon` model, accessed via `trainer.my_pokemons`

---

### 2. Alembic Migration

Generate a migration script for the `pokedex` and `my_pokemon` tables.

1. `pokedex`
2. `my_pokemon`
3. `trainer`