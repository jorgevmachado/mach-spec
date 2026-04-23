# OpenSpec Prompt — Auth & Trainer Feature

## Context

This project uses a monorepo with two submodules:

- **mach-api** — Python 3.13, FastAPI, SQLAlchemy async, domain-driven architecture
- **mach-web** — Next.js 16, React 19, TypeScript, Tailwind CSS v4

---
## SPEC: mach-api

### 1. Database Models

#### 1.1 Model: `Pokemon`
File location: `mach-api/app/models/pokemon.py`

Create a SQLAlchemy async model named `Pokemon` mapped to the table `pokemons` with the following columns:

| Column | Type | Constraints |
|---|---|---|
| `id` | UUID | Primary key, default `uuid4`, not nullable |
| `name` | String | Unique, not nullable |
| `order` | Integer | Not nullable |
| `status` | Enum(`StatusEnum`) | Not nullable |
| `external_image` | String | Not nullable |
| `hp` | Integer | Nullable, default `0` |
| `image` | String | Nullable |
| `speed` | Integer | Nullable, default `0` |
| `height` | Integer | Nullable, default `0` |
| `weight` | Integer | Nullable, default `0` |
| `attack` | Integer | Nullable, default `0` |
| `defense` | Integer | Nullable, default `0` |
| `habitat` | String | Nullable, default `None` |
| `is_baby` | Boolean | Nullable, default `False` |
| `shape_url` | String | Nullable |
| `shape_name` | String | Nullable |
| `is_mythical` | Boolean | Nullable, default `False` |
| `gender_rate` | Integer | Nullable, default `0` |
| `is_legendary` | Boolean | Nullable, default `False` |
| `capture_rate` | Integer | Nullable, default `0` |
| `hatch_counter` | Integer | Nullable, default `0` |
| `base_happiness` | Integer | Nullable, default `0` |
| `special_attack` | Integer | Nullable, default `0` |
| `base_experience` | Integer | Nullable, default `0` |
| `special_defense` | Integer | Nullable, default `0` |
| `evolution_chain` | String | Nullable |
| `evolves_from_species` | String | Nullable |
| `has_gender_differences` | Boolean | Nullable, default `False` |
| `growth_rate_id` | UUID | Foreign key → `pokemon_growth_rates.id`, Nullable |
| `created_at` | DateTime | Not nullable, default `utcnow`, **never updated after insert** |
| `updated_at` | DateTime | Nullable, default `None`, **only set on second or later updates** |
| `deleted_at` | DateTime | Nullable, default `None`, **only set when soft-delete is triggered** |

**Reuse the `Enum` class in `mach-api/app/models/enums.py` by consuming the `StatusEnum`**.
**Relationships on `Pokemon`:**
- `growth_rate` → many-to-one relationship to `PokemonGrowthRate` (optional, nullable FK)
- `moves` → many-to-many relationship to `PokemonMove` via association table `pokemon_pokemon_moves`
- `abilities` → many-to-many relationship to `PokemonAbility` via association table `pokemon_pokemon_abilities`
- `types` → many-to-many relationship to `PokemonType` via association table `pokemon_pokemon_types`
- `evolutions` → many-to-many self-referential relationship on `pokemons` via association table `pokemon_evolutions`. Both sides must be accessible (e.g. `pokemon.evolutions` lists Pokémon this one evolves into)

**Behavior rules for `Pokemon`:**
- `created_at` is set only at creation and must never be changed afterwards.
- `updated_at` starts as `None`. It must be set to the current UTC datetime only on the **second or later modification** of the record.
- `deleted_at` must only be set when a soft-delete operation is explicitly triggered. Soft-deleting must **not** physically remove the row.

---

#### 1.2 Model: `PokemonGrowthRate`

File location: `mach-api/app/models/pokemon_growth_rate.py`

Create a SQLAlchemy async model named `PokemonGrowthRate` mapped to the table `pokemon_growth_rates`:

| Column | Type | Constraints |
|---|---|---|
| `id` | UUID | Primary key, default `uuid4`, not nullable |
| `url` | String | Not nullable |
| `name` | String | Unique, not nullable |
| `formula` | String | Not nullable |
| `description` | String | Not nullable |
| `created_at` | DateTime | Not nullable, default `utcnow`, never updated after insert |
| `updated_at` | DateTime | Nullable, default `None`, only set on second or later updates |
| `deleted_at` | DateTime | Nullable, default `None`, only set on soft-delete |

**Relationships on `PokemonGrowthRate`:**
- `pokemons` → back-populates from `Pokemon.growth_rate`

---

#### 1.3 Model: `PokemonMove`

File location: `mach-api/app/models/pokemon_move.py`

Create a SQLAlchemy async model named `PokemonMove` mapped to the table `pokemon_moves`:

| Column | Type | Constraints |
|---|---|---|
| `id` | UUID | Primary key, default `uuid4`, not nullable |
| `pp` | Integer | Not nullable |
| `url` | String | Not nullable |
| `type` | String | Not nullable |
| `name` | String | Unique, not nullable |
| `order` | Integer | Not nullable |
| `power` | Integer | Not nullable |
| `target` | String | Not nullable |
| `effect` | String | Not nullable |
| `priority` | Integer | Not nullable |
| `accuracy` | Integer | Not nullable |
| `short_effect` | String | Not nullable |
| `damage_class` | String | Not nullable |
| `effect_chance` | Integer | Nullable |
| `created_at` | DateTime | Not nullable, default `utcnow`, never updated after insert |
| `updated_at` | DateTime | Nullable, default `None`, only set on second or later updates |
| `deleted_at` | DateTime | Nullable, default `None`, only set on soft-delete |

**Relationships on `PokemonMove`:**
- `pokemons` → many-to-many back-reference via `pokemon_pokemon_moves`

---

#### 1.4 Model: `PokemonAbility`

File location: `mach-api/app/models/pokemon_ability.py`

Create a SQLAlchemy async model named `PokemonAbility` mapped to the table `pokemon_abilities`:

| Column | Type | Constraints |
|---|---|---|
| `id` | UUID | Primary key, default `uuid4`, not nullable |
| `url` | String | Not nullable |
| `order` | Integer | Not nullable |
| `name` | String | Unique, not nullable |
| `slot` | Integer | Not nullable |
| `is_hidden` | Boolean | Not nullable |
| `created_at` | DateTime | Not nullable, default `utcnow`, never updated after insert |
| `updated_at` | DateTime | Nullable, default `None`, only set on second or later updates |
| `deleted_at` | DateTime | Nullable, default `None`, only set on soft-delete |

**Relationships on `PokemonAbility`:**
- `pokemons` → many-to-many back-reference via `pokemon_pokemon_abilities`

---

#### 1.5 Model: `PokemonType`

File location: `mach-api/app/models/pokemon_type.py`

Create a SQLAlchemy async model named `PokemonType` mapped to the table `pokemon_types`:

| Column | Type | Constraints |
|---|---|---|
| `id` | UUID | Primary key, default `uuid4`, not nullable |
| `url` | String | Not nullable |
| `order` | Integer | Not nullable |
| `name` | String | Unique, not nullable |
| `text_color` | String | Not nullable |
| `background_color` | String | Not nullable |
| `created_at` | DateTime | Not nullable, default `utcnow`, never updated after insert |
| `updated_at` | DateTime | Nullable, default `None`, only set on second or later updates |
| `deleted_at` | DateTime | Nullable, default `None`, only set on soft-delete |

**Relationships on `PokemonType`:**
- `pokemons` → many-to-many back-reference via `pokemon_pokemon_types`
- `weaknesses` → many-to-many self-referential via association table `pokemon_type_weaknesses` (types that deal double or half damage TO this type)
- `strengths` → many-to-many self-referential via association table `pokemon_type_strengths` (types this type deals double or half damage TO)

**Color mapping rule for `text_color` and `background_color`:**

When creating a `PokemonType`, generate colors deterministically based on the type `name` following this palette:

| name | background_color | text_color |
|---|---|---|
|ice | #fff | #51c4e7 |
|bug| #b5d7a7 | #482d53 |
|fire| #fff | #fd7d24 |
|rock| #fff | #a38c21 |
|dark| #fff | #707070 |
|steel| #fff | #9eb7b8 |
|ghost| #fff | #7b62a3 |
|fairy| #cb3fa0 | #c8a2c8 |
|water| #6890F0 | #FFFFFF |
| grass | #78C850 | #FFFFFF |
|normal| #A8A878 | #FFFFFF |
|dragon| #fff | #FF8C00 |
|poison| #fff | #b97fc9 |
|flying| #424242 | #3dc7ef |
|ground| #f5f5f5 | #bc5e00 |
|psychic| #fff | #f366b9 |
| electric | #F8D030 | #212121 |
|fighting| #fff | #d56723 |
| (default) | #68A090 | #FFFFFF |

---

### 2. Association Tables

File location: `mach-api/app/models/associations.py`

Define the following SQLAlchemy `Table` objects for many-to-many relationships:

```python
pokemon_pokemon_moves = Table(
    "pokemon_pokemon_moves",
    Base.metadata,
    Column("pokemon_id", UUID, ForeignKey("pokemons.id"), primary_key=True),
    Column("pokemon_move_id", UUID, ForeignKey("pokemon_moves.id"), primary_key=True),
)

pokemon_pokemon_abilities = Table(
    "pokemon_pokemon_abilities",
    Base.metadata,
    Column("pokemon_id", UUID, ForeignKey("pokemons.id"), primary_key=True),
    Column("pokemon_ability_id", UUID, ForeignKey("pokemon_abilities.id"), primary_key=True),
)

pokemon_pokemon_types = Table(
    "pokemon_pokemon_types",
    Base.metadata,
    Column("pokemon_id", UUID, ForeignKey("pokemons.id"), primary_key=True),
    Column("pokemon_type_id", UUID, ForeignKey("pokemon_types.id"), primary_key=True),
)

pokemon_type_weaknesses = Table(
    "pokemon_type_weaknesses",
    Base.metadata,
    Column("type_id", UUID, ForeignKey("pokemon_types.id"), primary_key=True),
    Column("weak_against_id", UUID, ForeignKey("pokemon_types.id"), primary_key=True),
)

pokemon_type_strengths = Table(
    "pokemon_type_strengths",
    Base.metadata,
    Column("type_id", UUID, ForeignKey("pokemon_types.id"), primary_key=True),
    Column("strong_against_id", UUID, ForeignKey("pokemon_types.id"), primary_key=True),
)

pokemon_evolutions = Table(
    "pokemon_evolutions",
    Base.metadata,
    Column("pokemon_id", UUID, ForeignKey("pokemons.id"), primary_key=True),
    Column("evolution_id", UUID, ForeignKey("pokemons.id"), primary_key=True),
)
```

---

### 3. Alembic Migration

Generate an Alembic migration that creates the tables in this order:

1. `pokemon_growth_rates`
2. `pokemon_moves`
3. `pokemon_abilities`
4. `pokemon_types`
5. `pokemons`
6. All association tables

---

### 4. External API Client

File location: `mach-api/app/infrastructure/external_api/pokeapi_client.py`

Create a dedicated async HTTP client class `PokeApiClient` responsible for all calls to `https://pokeapi.co`. This class must be injected via FastAPI dependency injection and must NOT be mixed into domain services or repositories. Methods:

This service should implement the following methods:
- `list_pokemon(offset: int = 0, limit: int = 1350) -> list[PokemonExternalBaseSchema]` — returns the list of pokemons from the external API with pagination support. The default limit should be set to 1350 to retrieve all pokemons in one call.
- `get_pokemon(name: str) -> PokemonExternalSchema` — returns the pokemon details by name from the external API.
- `get_pokemon_species(name_or_id: str) -> PokemonExternalSpecieSchema` — returns the pokemon species details by name or id from the external API.
- `get_move(order: int) -> PokemonExternalMoveSchema` — returns the pokemon move details by order from the external API.
- `get_type(order: int) -> PokemonExternalTypeSchema` — returns the pokemon type details by order from the external API.
- `get_growth_rate(order: int) -> PokemonExternalGrowthRateSchema` — returns the pokemon growth rate details by order from the external API.
- `get_evolution_chain(url: str) -> PokemonExternalEvolutionSchema` — returns the pokemon evolution chain details by url from the external API.
- `get_pokemon_by_name(pokemon: PokemonSchema) -> PokemonByNameResponseSchema` — returns the pokemon details by name from the external API.
- - Map `PokemonByNameResponseSchema` to `{ pokemon: PokemonSchema, types: list[PokemonTypeResponseSchema], moves: list[PokemonMoveResponseSchema], abilities: list[PokemonAbilityResponseSchema], growth_rate: PokemonGrowthRateResponseSchema }` 
- - Logic:
    1. Call `PokeApiClient.get_pokemon(name_or_id: str)` to get the pokemon details.
    2. If the pokemon is not found, return `PokemonByNameResponseSchema` with `{ pokemon: pokemon, types: [], moves: [], abilities: [], growth_rate: None }`
    3. If the pokemon is found, extract the attributes from the response and map them to the pokemon model fields.
    4. If the pokemon is found, mount `PokemonByNameResponseSchema.pokemon` pokemon with:
       - `id=pokemon.id`
       - `url=pokemon.url`
       - `name=pokemon.name`
       - `order=pokemon.order`
       - `status=StatusEnum.INCOMPLETE`
       - `external_image=pokemon.external_image`
       - `hp=attributes.hp`
       - `image=PokemonExternalBusiness.ensure_image(pokemon_name_response.sprites)`
       - `speed=attributes.speed`
       - `height=attributes.height`
       - `weight=attributes.weight`
       - `attack=attributes.attack`
       - `defense=attributes.defense`
       - `habitat=None`
       - `is_baby=False`
       - `shape_url=None`
       - `shape_name=None`
       - `is_mythical=False`
       - `gender_rate=None`
       - `is_legendary=False`
       - `capture_rate=None`
       - `hatch_counter=None`
       - `base_happiness=None`
       - `special_attack=attributes.special_attack`
       - `base_experience=attributes.base_experience`
       - `special_defense=attributes.special_defense`
       - `evolution_chain_url=None`
       - `evolves_from_species=None`
       - `has_gender_differences=False`
       - `created_at=pokemon.created_at`
       - `updated_at=pokemon.updated_at`
       - `deleted_at=pokemon.deleted_at`
    5. If the pokemon is found, mount `PokemonByNameResponseSchema.types` types with pokemon_name_response.types
    6. If the pokemon is found, mount `PokemonByNameResponseSchema.moves` moves with pokemon_name_response.moves
    7. If the pokemon is found, mount `PokemonByNameResponseSchema.abilities` abilities with pokemon_name_response.abilities
    8. Call `PokeApiClient.get_pokemon_species(name_or_id=pokemon.species.name_or_id)` to get the pokemon species details.
    9. If the pokemon species is not found, return `PokemonByNameResponseSchema` with new pokemon attributes.
    10. If the pokemon species is found, update the pokemon attributes with:
        - `id=pokemon.id`
        - `url=pokemon.url`
        - `name=pokemon.name`
        - `order=pokemon.order`
        - `status=StatusEnum.INCOMPLETE`
        - `external_image=pokemon.external_image`
        - `hp=attributes.hp`
        - `image=PokemonExternalBusiness.ensure_image(pokemon_name_response.sprites)`
        - `speed=attributes.speed`
        - `height=attributes.height`
        - `weight=attributes.weight`
        - `attack=attributes.attack`
        - `defense=attributes.defense`
        - `habitat=specie_attributes.habitat`
        - `is_baby=specie_attributes.is_baby`
        - `shape_url=specie_attributes.shape_url`
        - `shape_name=specie_attributes.shape_name`
        - `is_mythical=specie_attributes.is_mythical`
        - `gender_rate=specie_attributes.gender_rate`
        - `is_legendary=specie_attributes.is_legendary`
        - `capture_rate=specie_attributes.capture_rate`
        - `hatch_counter=specie_attributes.hatch_counter`
        - `base_happiness=specie_attributes.base_happiness`
        - `special_attack=attributes.special_attack`
        - `base_experience=attributes.base_experience`
        - `special_defense=attributes.special_defense`
        - `evolution_chain_url=specie_attributes.evolution_chain_url`
        - `evolves_from_species=specie_attributes.evolves_from_species`
        - `has_gender_differences=specie_attributes.has_gender_differences`
        - `created_at=pokemon.created_at`
        - `updated_at=pokemon.updated_at`
        - `deleted_at=pokemon.deleted_at`
    11. If the pokemon species is found, mount `PokemonByNameResponseSchema.growth_rate` with pokemon_species_attributes.growth_rate
    12. return `PokemonByNameResponseSchema`.    

All methods must use `httpx.AsyncClient`. Raise `HTTP 502 Bad Gateway` with a descriptive message if the external API returns a non-2xx response.
#### 4.1. External API Client Schema:
File location: `mach-api/app/infrastructure/external_api/schemas.py`
 1. Create a Pydantic model `PokemonExternalBaseSchema` that represents a Pokemon from the external API.
    - `url: str`
    - `order: int`
    - `name: str`
    - `external_image: str`     
 2. Create a Pydantic model `PokemonExternalBaseTypeSchemaResponse` that represents a Pokemon type from the external API.
    - `type: { name: str, url: str }`
    - `slot: int`
 3. Create a Pydantic model `PokemonExternalBaseMoveSchemaResponse` that represents a Pokemon move from the external API.
    - `move: { name: str, url: str }` 
 4. Create a Pydantic model `PokemonExternalBaseAbilitySchemaResponse` that represents a Pokemon ability from the external API.
    - `slot: int`
    - `ability: { name: str, url: str }`
    - `is_hidden: bool` 
 5. Create a Pydantic model `PokemonExternalBaseStatSchemaResponse` that represents a Pokemon ability from the external API. 
    - `stat: { name: str, url: str }`
    - `base_stat: int`
 6. Create a Pydantic model `PokemonExternalBaseSpritesSchemaResponse` that represents a Pokemon ability from the external API.
    - `back_gray: Optional[str] = None`
    - `front_gray: Optional[str] = None`
    - `back_shiny: Optional[str] = None`
    - `front_shiny: Optional[str] = None`
    - `back_female: Optional[str] = None`
    - `front_female: Optional[str] = None`
    - `back_default: Optional[str] = None`
    - `front_default: Optional[str] = None`
    - `back_transparent: Optional[str] = None`
    - `front_transparent: Optional[str] = None`
    - `back_shiny_female: Optional[str] = None`
    - `front_shiny_female: Optional[str] = None`
    - `back_shiny_transparent: Optional[str] = None`
    - `front_shiny_transparent: Optional[str] = None`
 7. Create a Pydantic model `PokemonExternalSchema` that represents a Pokemon from the external API.
    - `name: str`
    - `order: int`
    - `types: list[PokemonExternalBaseTypeSchemaResponse]`
    - `moves: list[PokemonExternalBaseMoveSchemaResponse]`
    - `stats: list[PokemonExternalBaseStatSchemaResponse]`
    - `height: int`
    - `weight: int`
    - `sprites: PokemonExternalBaseSpritesSchemaResponse`
    - `abilities: list[PokemonExternalBaseAbilitySchemaResponse]`
    - `base_experience: int`
 8. Create a Pydantic model `PokemonExternalSpecieSchema` that represents a Pokemon species from the external API.
    - `name: str`
    - `shape: Optional[{ name: str, url: str}] = None`
    - `habitat: Optional[{ name: str, url: str}] = None`
    - `is_baby: bool`
    - `growth_rate: Optional[{ name: str, url: str}] = None`
    - `gender_rate: int`
    - `is_mythical: bool`
    - `capture_rate: int`
    - `is_legendary: bool`
    - `hatch_counter: int`
    - `base_happiness: int`
    - `evolution_chain: Optional[{ name: str, url: str}] = None`
    - `evolves_from_species: Optional[{ name: str, url: str}] = None`
    - `has_gender_differences: bool`     
 9. Create a Pydantic model `PokemonExternalMoveSchemaResponse` that represents a Pokemon move from the external API.
    - `type: { name: str, url: str }`
    - `name: str`
    - `order: int`
    - `power: int`
    - `target: { name: str }`
    - `effect_entries: { flavor_text: str, language: { name: str }, short_effect: str }`
    - `damage_class: { name: str }`
    - `effect_chance: int | None` 
 10. Create a Pydantic model `PokemonExternalGrowthRateSchemaResponse` that represents a Pokemon Growth Rate from the external API.
     - `id: int`
     - `name: str`
     - `formula: str`
     - `levels: list[{ level: int, experience: int }]`
     - `descriptions: list[{ description: str, language: { name: str, url: str }}]`
 11. Create a Pydantic model `PokemonExternalTypeSchemaResponse` that represents a Pokemon type from the external API.
     - `id: int`
     - `moves: list[{name: str, url: str }]`
     - `names: list[{ name: str, language: { name: str, url: str}}]`
     - `generation: { name: str, url: str }`
     - `game_indices: list[{ game_index: int, generation: { name: str, url: str }}]`
     - `damage_relations: { double_damage_from: { name: str, url: str }, double_damage_to: { name: str, url: str }, half_damage_from: { name: str, url: str }, half_damage_to: { name: str, url: str },} }`    
     - `move_damage_class: Optional[{ name: str, url: str }] = None` 
---

---

### 6. Utility: `external_image` Generator
Use the helper function ensure_order_number from file location: `mach-api/app/shared/utils/number.py`
Use the helper function ensure_external_image from file location: `mach-api/app/shared/utils/image.py`
---

---

### 7. Pokémon
#### 7.1 Domain Repository: `PokemonRepository`
File location: `mach-api/app/domain/pokemon/repository.py`
Create a repository class `PokemonRepository` that implements BaseRepository[Pokemon] from `mach-api/app/core/repository/base.py`.

This repository should populate the model with Pokemon, the relations with the relationships using selectInload, and default_order_by with the value 'order'.

#### 7.2. Domain Schema: 
File location: `mach-api/app/domain/pokemon/schema.py`
Create a Pydantic model `PokemonSchema` that represents a Pokemon.
Create a Pydantic model `PokemonFilterPageSchema` that implements `FilterPage` from `mach-api/app/shared/schemas.py`.

#### 7.3. Domain Service: `PokemonService`
File location: `mach-api/app/domain/pokemon/service.py`
Create a service class `PokemonService` that implements BaseService[PokemonRepository,Pokemon] from `mach-api/app/core/service/base.py`.

Async Methods:
- `total() -> int` — returns total pokemon count
- `initialize_list(total: int = 0, external_total: int = 1350)`
Logic: 
1. Call `PokeApiClient.list_pokemon(offset=0, limit=external_total)` to get the list of all pokemons from the external API.
2. If total is 0, For each result in the list,
   - Extract `order` from `url` using `ensure_order_number` from `mach-api/app/utils/number.py`
   - Generate `external_image` using `ensure_external_image` from `mach-api/app/shared/utils/image.py`
   - Create via `PokemonRepository.create()`
     - `name` → result name
     - `order` → extracted order
     - `status` → `StatusEnum.INCOMPLETE`
     - `external_image` → generated URL
3. Re-query `PokemonRepository.list_all()` and build response list.
4. If total is not 0 and total < external_total. For each result in the list,
   - Extract `order` from `url` using `ensure_order_number` from `mach-api/app/utils/number.py`
   - Generate `external_image` using `ensure_external_image` from `mach-api/app/shared/utils/image.py`
   - Check if pokemon with the same order exists in the database via `PokemonRepository.find_by(order=order)`
   - If not exist, Create via `PokemonRepository.create()`
     - `name` → result name
     - `order` → extracted order
     - `status` → `StatusEnum.INCOMPLETE`
     - `external_image` → generated URL
5. Re-query `PokemonRepository.list_all()` and build response list.
 - `list_sync() -> bool` — make sure the database is up to date with the external API
   Logic:
   1. Check Redis cache with key `pokemon:meta`
   2. If cached, return False.
   3. If not cached, call `total()` and `PokeApiClient.total()` and set meta cache.
   4. If total `0`, call `initialize_list()` and return True.
   5. If total `< 1350`, call `initialize_list()` and call `total()` set new meta cache and return True.
   
 - `list(page_filter: Annotated[PokemonFilterPage, Query()] = None, user_request: str | None = None, trainer_id: str | None = None)`. — returns a paginated list of pokemons or a list of pokemons, optionally filtered by name (case-insensitive, partial match). If `user_request` and `trainer_id` are provided, log the request in the trainer's history (see TrainerService requirements for details on how to do this).
 - `list_cached(page_filter: Annotated[PokemonFilterPage, Query()] = None, user_request: str | None = None, trainer_id: str | None = None)`. — returns a paginated list of pokemons or a list of pokemons with cached data.
  Logic:
  1. Check Redis cache with key `"pokemon:list"` or `"pokemon:list:status=COMPLETE"`
  2. If cached, return the cached data.
  3. If not cached, query `list`.
- `get(name_or_id: str) -> Pokemon` — returns a pokemon by name or id. Raise `HTTP 404 Not Found` if no pokemon is found with the given name or id. if status is INCOMPLETE call `complete_pokemon(name: str)` else return the pokemon.
   Logic:
   - Check if pokemon exists in the database via `PokemonRepository.find_by(name=name)`
   - If pokemon exists and status is COMPLETE return it.
   - If pokemon does not exist, raise `HTTP 404 Not Found`
   - If pokemon exists and status is INCOMPLETE call `complete_pokemon`, return the result and set status to COMPLETE. If `complete_pokemon` fails, raise `HTTP 502 Bad Gateway` with a descriptive message.
 - `complete_pokemon(name: str, with_evolutions: bool = True) -> Pokemon` — completes a pokemon by name.
 Logic:
  1. Fetch the pokemon data from the external API using `PokeApiClient.get_pokemon(name: str)`.
  2. Verify moves call `add_moves(moves: list[PokemonMoveResponseSchema])` and return a list of added moves.
  3. Verify abilities call `add_abilities(abilities: list[PokemonAbilityResponseSchema])` and return a list of added abilities.
  4. Verify types call `add_types(types: list[PokemonTypeResponseSchema])` and return a list of added types.
  5. Verify growth_rate call `add_growth_rate(growth_rate: PokemonGrowthRateResponseSchema)` and return the added growth rate.
  6. Verify evolutions call `add_evolutions(evolution_chain_url: str)` and return a list of pokemons.
  7. Update the pokemon database record with the complete data and set status to COMPLETE.
- - If successful `PokeApiClient.get_pokemon(name: str)` Verify moves, abilities, types, growth_rate and evolutions by `add_moves(moves: list[PokemonMove])` and `add_types(types: list[PokemonType])` and `add_abilities(abilities: list[PokemonAbility])` and `add_growth_rate(growth_rate: PokemonGrowthRate)` and `add_evolutions(evolution_chain_url: str)` and Update Pokemon database record with the complete data and set status to COMPLETE. If any of the above steps fail, raise `HTTP 502 Bad Gateway` with a descriptive message.
- `add_moves(moves: list[PokemonMove])` — Call `PokemonMoveService.add_many(moves)` and return a list of added moves.
- `add_abilities(abilities: list[PokemonAbility])` — Call `PokemonAbilityService.add_many(abilities)` and return a list of added abilities.
- `add_types(types: list[PokemonType])` — Call `PokemonTypeService.add_many(types)` and return a list of added types.
- `add_growth_rate(growth_rate: PokemonGrowthRate)` — Call `PokemonGrowthRateService.add(growth_rate)` and return the added growth rate.
- `add_evolutions(evolution_chain_url: str)` — Call `PokeApiClient.get_evolutions(evolution_chain_url)` and call `complete_pokemon(name: str, with_evolutions: bool = False)` and return a list of pokemons.
- `soft_delete(id: str)` — Soft delete a pokemon by id. Raise `HTTP 404 Not Found` if no pokemon is found with the given id. Set status to DELETED.

#### 7.4. Domain Route: `PokemonRoute`
File location: `mach-api/app/domain/pokemon/route.py`
Create a FastAPI router `PokemonRoute` with the following endpoints:
##### 8.5.1. `GET: /pokemon -> CustomLimitOffsetPage[PokemonSchema] | list[PokemonSchema]` — returns a paginated list of pokemon or a list of pokemon with cached data, optionally filtered by name (case-insensitive, partial match).
- **Method:** GET
- **Path:** `/pokemon`
- **Query Parameters:**
    - `page` (optional): Number of the page to retrieve (default: 1).
    - `limit` (optional): Number of items to return per page (default: 10).
    - `offset` (optional): Number of items to skip before starting to collect the result set (default: 0).
    - `order_by` (optional): Field to order the results by (default: "order").
- **Response:** `CustomLimitOffsetPage[PokemonSchema]` or `list[PokemonSchema]` with status `HTTP 200`
- **Auth required:** Yes 
- **Description:** Returns a paginated list of pokemon or a list of pokemon with cached data,

##### 8.5.2. `GET: /pokemon/{param} -> PokemonSchema` — returns a pokemon by name.
- **Method:** GET
- **Path:** `/pokemon/{param}`
- **Path Parameters:** `param`: The name of the pokemon to retrieve.
- **Response:** `PokemonSchema` with status `HTTP 200`
- **Auth required:** Yes 
- **Description:** Returns a pokemon by name or id. Raises `HTTP 404 Not Found` if no pokemon is found with the given name.

##### 8.5.3. `DELETE: /pokemon/{id} -> str` — soft deletes a pokemon
- **Method:** DELETE
- **Path:** `/pokemon/{id}`
- **Path Parameters:** `id`: The ID of the pokemon to soft delete.
- **Response:** `str` with status `HTTP 200`
- **Auth required:** Yes 
- **Description:** Soft deletes a pokemon by ID. Raises `HTTP 404 Not Found` if no pokemon is found with the given ID. Sets the status of the pokemon to DELETED.

---

#### 7.5. Router Registration
Register the `PokemonRoute` in `mach-api/app/main.py` under the `/pokemon` path.
```python
app.include_router(pokemon_router, prefix="/pokemon", tags=["Pokemon"])
```
---

---

### 8. PokemonMove
#### 8.1 Domain Repository: `PokemonMoveRepository`
File location: `mach-api/app/domain/pokemon_move/repository.py`
Create a repository class `PokemonMoveRepository` that implements BaseRepository[PokemonMove] from `mach-api/app/core/repository/base.py`.

This repository should populate the model with PokemonMove, the relations with the relationships using selectInload, and default_order_by with the value 'order'.

#### 8.2. Domain Schema:
File location: `mach-api/app/domain/pokemon_move/schema.py`
Create a Pydantic model `PokemonMoveSchema` that represents a PokemonMove.
Create a Pydantic model `PokemonMoveFilterPageSchema` that implements `FilterPage` from `mach-api/app/shared/schemas.py`.
Create a Pydantic model `EffectEntry`  with effect: str and short_effect: str;

#### 8.3. Domain Service: `PokemonMoveService`
File location: `mach-api/app/domain/pokemon_move/service.py`
Create a service class `PokemonMoveService` that implements BaseService[PokemonMoveRepository,PokemonMove] from `mach-api/app/core/service/base.py`.

This service should implement the following methods:
- `add_moves(moves: list[PokemonMoveResponseSchema]) -> list[PokemonMove] | None` — search in database for existing moves and return a list of added moves. If a move already exists, return the existing move. If no move is found, call `PokeApiClient.get_pokemon_moves(order: int)` if service succesffuly call `PokemonMoveBusiness.ensure_effect_message(effect_entries)` and merge result with props and save in database.
**`PokemonMoveResponseSchema` has the shape: `move: { name: str, url: str }`**
Logic:
1. The program iterates through the matrix, retrieving each item individually. Each item is then called an item.
2. Extract `order` from `item['url']` using `ensure_order_number` from `mach-api/app/utils/number.py`
3. Query `PokemonMoveRepository.find_by(order=order)` If found, return it and continue the iterations.
4. If not found, call `PokeApiClient.get_move(order: int)`. 
5. Map the external API response to the model fields:
    - `pp` → `data["pp"]`
    - `url` → `move_data["url"]`
    - `type` → `data["type"]["name"]`
    - `name` → `data["name"]`
    - `order` → extracted from url
    - `power` → `data["power"] or 0`
    - `target` → `data["target"]["name"]`
    - `effect` → first English entry in `data["effect_entries"]` where `language.name == "en"`, field `effect`; fallback to `""`
    - `priority` → `data["priority"]`
    - `accuracy` → `data["accuracy"] or 0`
    - `short_effect` → first English entry in `data["effect_entries"]` where `language.name == "en"`, field `short_effect`; fallback to `""`
    - `damage_class` → `data["damage_class"]["name"]`
    - `effect_chance` → `data["effect_chance"]` (nullable) 
6. If the API call is successful, extract the `effect_entries` from the response and call `PokemonMoveBusiness.ensure_effect_message(effect_entries)` to get the effect message. Merge the effect message with the other properties of the move and save it in the database using `PokemonMoveRepository.save()`. Return the newly added move.
- `get(id: str) -> PokemonMoveSchema | None` — returns a pokemon move by id
- `list(page_filter: Annotated[PokemonMoveFilterPageSchema, Query()] = None) -> CustomLimitOffsetPage[PokemonMoveSchema] | list[PokemonMoveSchema]` — returns a paginated list of pokemon moves or a list of pokemon moves with cached data, optionally filtered by name (case-insensitive, partial match).
- `soft_delete(id: str)` — Soft delete a pokemon move by id. Raise `HTTP 404 Not Found` if no pokemon move is found with the given id. Set status to DELETED.

#### 8.4. Domain Business: `PokemonMoveBusiness`
File location: `mach-api/app/domain/pokemon_move/business.py`:
Create a class `PokemonMoveBusiness` with the following method:
- `effect_entries(effect_entries: list[dict]) -> EffectEntry` — receives the `effect_entries` list from the external API response and returns a string with the effect message in English. If no English entry is found, return the first entry in any language. If the list is empty, return "No effect information available".
- `select_random_moves(available_moves: Sequence[PokemonMove],max_moves: int = 4) -> list[PokemonMove]` — returns a list of random moves from the available moves, up to a maximum of `max_moves`.

#### 8.5. Domain Route: `PokemonMoveRoute`
File location: `mach-api/app/domain/pokemon_move/route.py`
Create a FastAPI router `PokemonMoveRoute` with the following endpoints:

##### 8.5.1. `GET: /pokemon-move -> CustomLimitOffsetPage[PokemonMoveSchema] | list[PokemonMoveSchema]` — returns a paginated list of pokemon moves or a list of pokemon moves with cached data, optionally filtered by name (case-insensitive, partial match).
- **Method:** GET
- **Path:** `/pokemon-move`
- **Query Parameters:** 
  - `page` (optional): Number of the page to retrieve (default: 1).
  - `limit` (optional): Number of items to return per page (default: 10).
  - `offset` (optional): Number of items to skip before starting to collect the result set (default: 0).
  - `order_by` (optional): Field to order the results by (default: "order").
- **Response:** `CustomLimitOffsetPage[PokemonMoveSchema]` or `list[PokemonMoveSchema]` with status `HTTP 200`
- **Auth required:** Yes 
- **Description:** Returns a paginated list of pokemon moves or a list of pokemon moves with cached data,

##### 8.5.2. `GET: /pokemon-move/{param} -> PokemonMoveSchema` — returns a pokemon move by name or id.
- **Method:** GET
- **Path:** `/pokemon-move/{param}`
- **Path Parameters:** `param`: The name or ID of the pokemon move to retrieve.
- **Response:** `PokemonMoveSchema` with status `HTTP 200`
- **Auth required:** Yes 
- **Description:** Returns a pokemon move by name or id. Raises `HTTP 404 Not Found` if no pokemon move is found with the given name or id.

##### 8.5.3. `DELETE: /pokemon-move/{id} -> str` — soft deletes a pokemon move  
- **Method:** DELETE
- **Path:** `/pokemon-moves/{id}`
- **Path Parameters:** `id`: The ID of the pokemon move to soft delete.
- **Response:** `str` with status `HTTP 200`
- **Auth required:** Yes 
- **Description:** Soft deletes a pokemon move by ID. Raises `HTTP 404 Not Found` if no pokemon move is found with the given ID. Sets the status of the pokemon move to DELETED.

---

#### 8.6. Router Registration
Register the `PokemonMoveRoute` in `mach-api/app/main.py` under the `/pokemon-move` path.
```python
app.include_router(pokemon_move_router, prefix="/pokemon-move", tags=["PokemonMove"])
```
---

---

### 9. PokemonAbility
#### 9.1 Domain Repository: `PokemonAbilityRepository`
File location: `mach-api/app/domain/pokemon_ability/repository.py`
Create a repository class `PokemonAbilityRepository` that implements BaseRepository[PokemonAbility] from `mach-api/app/core/repository/base.py`.

This repository should populate the model with PokemonAbility, the relations with the relationships using selectInload, and default_order_by with the value 'order'.

#### 9.2. Domain Schema:
File location: `mach-api/app/domain/pokemon_ability/schema.py`
Create a Pydantic model `PokemonAbilitySchema` that represents a PokemonAbility.
Create a Pydantic model `PokemonAbilityFilterPageSchema` that implements `FilterPage` from `mach-api/app/shared/schemas.py`.

#### 9.3. Domain Service: `PokemonAbilityService`
File location: `mach-api/app/domain/pokemon_ability/service.py`
Create a service class `PokemonAbilityService` that implements BaseService[PokemonAbilityRepository,PokemonAbility] from `mach-api/app/core/service/base.py`.

This service should implement the following methods:
- `add_abilities(abilities: list[PokemonAbilityResponseSchema]) -> list[PokemonAbilitySchema] | None` — search in database for existing abilities and return a list of added abilities.
**`PokemonAbilityResponseSchema` has the shape: `ability: { name: str, url: str }, is_hidden: bool, slot: int`**
Logic:
1. The program iterates through the matrix, retrieving each item individually. Each item is then called an item.
2. Extract `order` from `item['ability']['url']` using `ensure_order_number` from `mach-api/app/utils/number.py`
3. Query `PokemonAbilityRepository.find_by(order=order)` If found, return it and continue the iterations.
4. If not found, create with:
   - `url` → `ability_data["ability"]["url"]`
   - `order` → extracted from url
   - `name` → `ability_data["ability"]["name"]`
   - `slot` → `ability_data["slot"]`
   - `is_hidden` → `ability_data["is_hidden"]`
5. Persist via `PokemonAbilityRepository.save` and return.
- `get(param: str) -> PokemonAbilitySchema | None` — returns a pokemon ability by id or name
- `list(page_filter: Annotated[PokemonAbilityFilterPageSchema, Query()] = None) -> CustomLimitOffsetPage[PokemonAbilitySchema] | list[PokemonAbilitySchema]` — returns a paginated list of pokemon moves or a list of pokemon moves with cached data, optionally filtered by name (case-insensitive, partial match).
- `soft_delete(id: str)` — Soft delete a pokemon ability by id. Raise `HTTP 404 Not Found` if no pokemon ability is found with the given id. Set status to DELETED.

#### 9.4. Domain Route:
File location: `mach-api/app/domain/pokemon_ability/route.py`
Create a FastAPI router `PokemonAbilityRoute` with the following endpoints:
##### 9.4.1. `GET: /pokemon-ability -> CustomLimitOffsetPage[PokemonAbilitySchema] | list[PokemonAbilitySchema]` — returns a paginated list of pokemon moves or a list of pokemon moves with cached data, optionally filtered by name (case-insensitive, partial match).
- **Method:** GET
- **Path:** `/pokemon-ability`
- **Query Parameters:**
    - `page` (optional): Number of the page to retrieve (default: 1).
    - `limit` (optional): Number of items to return per page (default: 10).
    - `offset` (optional): Number of items to skip before starting to collect the result set (default: 0).
    - `order_by` (optional): Field to order the results by (default: "order").
- **Response:** `CustomLimitOffsetPage[PokemonAbilitySchema]` or `list[PokemonAbilitySchema]` with status `HTTP 200`
- **Auth required:** Yes 
- **Description:** Returns a paginated list of pokemon abilities or a list of pokemon abilities with cached data,

##### 9.4.2. `GET: /pokemon-ability/{param} -> PokemonAbilitySchema` — returns a pokemon move by name or id.
- **Method:** GET
- **Path:** `/pokemon-ability/{param}`
- **Path Parameters:** `param`: The name or ID of the pokemon move to retrieve.
- **Response:** `PokemonAbilitySchema` with status `HTTP 200`
- **Auth required:** Yes 
- **Description:** Returns a pokemon ability by name or id. Raises `HTTP 404 Not Found` if no pokemon ability is found with the given name or id.

##### 9.4.3. `DELETE: /pokemon-ability/{id} -> str` — soft deletes a pokemon move
- **Method:** DELETE
- **Path:** `/pokemon-ability/{id}`
- **Path Parameters:** `id`: The ID of the pokemon move to soft delete.
- **Response:** `str` with status `HTTP 200`
- **Auth required:** Yes 
- **Description:** Soft deletes a pokemon ability by ID. Raises `HTTP 404 Not Found` if no pokemon ability is found with the given ID. Sets the status of the pokemon ability to DELETED.

---

#### 9.5. Router Registration
Register the `PokemonAbilityRoute` in `mach-api/app/main.py` under the `/pokemon-ability` path.
```python
app.include_router(pokemon_ability_router, prefix="/pokemon-ability", tags=["PokemonAbility"])
```
---

---

### 10. PokemonType
#### 10.1 Domain Repository: `PokemonTypeRepository`
File location: `mach-api/app/domain/pokemon_type/repository.py`
Create a repository class `PokemonTypeRepository` that implements BaseRepository[PokemonType] from `mach-api/app/core/repository/base.py`.

This repository should populate the model with PokemonType, the relations with the relationships using selectInload, and default_order_by with the value 'order'.

#### 10.2. Domain Schema:
File location: `mach-api/app/domain/pokemon_type/schema.py`
Create a Pydantic model `PokemonTypeSchema` that represents a PokemonType.
Create a Pydantic model `PokemonTypeFilterPageSchema` that implements `FilterPage` from `mach-api/app/shared/schemas.py`.

#### 10.3. Domain Service: `PokemonTypeService`
File location: `mach-api/app/domain/pokemon_type/service.py`
Create a service class `PokemonTypeService` that implements BaseService[PokemonTypeRepository,PokemonType] from `mach-api/app/core/service/base.py`.

This service should implement the following methods:
- `add_types(types: list[PokemonTypeResponseSchema]) -> list[PokemonTypeSchema] | None` — search in database for existing types if not exist call `PokeApiClient.get_pokemon_type(order: int)` and call `PokemonTypeBusiness.ensure_colors(pokemon_type_name: str)` and call `add_relations(pokemon_type: PokemonType)` and return a list of added types.
**`PokemonTypeResponseSchema` has the shape: `type: { name: str, url: str }, slot: int,`**
Logic: 
1. The program iterates through the matrix, retrieving each item individually. Each item is then called an item.
2. Extract `order` from `item['type']['url']` using `ensure_order_number` from `mach-api/app/utils/number.py`
3. Query `PokemonTypeRepository.find_by(order=order)` If found, return it and continue the iterations.
4. If not found, call `PokeApiClient.get_type(order: int)`. If successful, extract the `name` from the response and call `PokemonTypeBusiness.ensure_colors(pokemon_type_name: str)` to get the `background_color` and `text_color` for the type. Create a new `PokemonType` instance with the retrieved data and call `add_relations(pokemon_type: PokemonType)` to handle the damage relations.
5. Finally, save the new type in the database using `PokemonTypeRepository.save()` and return it 
- `add_relations(pokemon_type: PokemonType)` — search in database 
- - if exist call `PokemonTypeBusiness.ensure_damage_relations(damage_relations)` and return list of weakness and strengths
- - if not exist save in database and call `PokemonTypeBusiness.ensure_damage_relations(damage_relations)` and return list of weakness and strengths
- `get(param: str) -> PokemonAbilitySchema | None` — returns a pokemon type by id or name
- `list(page_filter: Annotated[PokemonTypeFilterPageSchema, Query()] = None) -> CustomLimitOffsetPage[PokemonTypeSchema] | list[PokemonTypeSchema]` — returns a paginated list of pokemon types or a list of pokemon types with cached data, optionally filtered by name (case-insensitive, partial match).
- `soft_delete(id: str)` — Soft delete a pokemon type by id. Raise `HTTP 404 Not Found` if no pokemon type is found with the given id. Set status to DELETED.

#### 10.4. Domain Business: `PokemonTypeBusiness`
File location: `mach-api/app/domain/pokemon_type/business.py`:
Create a class `PokemonTypeBusiness` with the following methods:
- `ensure_colors(pokemon_type_name: str) -> tuple[str, str]` — returns a tuple with the background color and text color for a given pokemon type name following the color mapping rule defined in the SPEC. If the type name is not in the mapping, return the default colors.
- `ensure_damage_relations(damage_relations: dict) -> tuple[list[PokemonType], list[PokemonType]]` — receives the `damage_relations` dict from the external API response and returns a tuple with two lists: the first list contains the weakness types (double or half damage TO this type) and the second list contains the strength types (double or half damage this type deals TO other types). For each type in the damage relations, check if it exists in the database. If it exists, use the existing type. If it does not exist, call `add_types` to add it to the database and use the newly added type.

#### 10.5. Domain Route:
File location: `mach-api/app/domain/pokemon_type/route.py`
Create a FastAPI router `PokemonTypeRoute` with the following endpoints:
##### 10.5.1. `GET: /pokemon-type -> CustomLimitOffsetPage[PokemonTypeSchema] | list[PokemonTypeSchema]` — returns a paginated list of pokemon moves or a list of pokemon moves with cached data, optionally filtered by name (case-insensitive, partial match).
- **Method:** GET
- **Path:** `/pokemon-type`
- **Query Parameters:**
    - `page` (optional): Number of the page to retrieve (default: 1).
    - `limit` (optional): Number of items to return per page (default: 10).
    - `offset` (optional): Number of items to skip before starting to collect the result set (default: 0).
    - `order_by` (optional): Field to order the results by (default: "order").
- **Response:** `CustomLimitOffsetPage[PokemonTypeSchema]` or `list[PokemonTypeSchema]` with status `HTTP 200`
- **Auth required:** Yes 
- **Description:** Returns a paginated list of pokemon types or a list of pokemon types with cached data,

##### 10.5.2. `GET: /pokemon-type/{param} -> PokemonTypeSchema` — returns a pokemon move by name or id.
- **Method:** GET
- **Path:** `/pokemon-type/{param}`
- **Path Parameters:** `param`: The name or ID of the pokemon move to retrieve.
- **Response:** `PokemonTypeSchema` with status `HTTP 200`
- **Auth required:** Yes 
- **Description:** Returns a pokemon type by name or id. Raises `HTTP 404 Not Found` if no pokemon move is found with the given name or id.

##### 10.5.3. `DELETE: /pokemon-type/{id} -> str` — soft deletes a pokemon move
- **Method:** DELETE
- **Path:** `/pokemon-type/{id}`
- **Path Parameters:** `id`: The ID of the pokemon move to soft delete.
- **Response:** `str` with status `HTTP 200`
- **Auth required:** Yes 
- **Description:** Soft deletes a pokemon type by ID. Raises `HTTP 404 Not Found` if no pokemon type is found with the given ID. Sets the status of the pokemon type to DELETED.

---
#### 10.6. Router Registration
Register the `PokemonTypeRoute` in `mach-api/app/main.py` under the `/pokemon-type` path.
```python
app.include_router(pokemon_type_router, prefix="/pokemon-type", tags=["PokemonType"])
```
---

---

### 11. PokemonGrowthRate
#### 11.1 Domain Repository: `PokemonGrowthRateRepository`
File location: `mach-api/app/domain/pokemon_growth_rate/repository.py`
Create a repository class `PokemonGrowthRateRepository` that implements BaseRepository[PokemonGrowthRate] from `mach-api/app/core/repository/base.py`.

This repository should populate the model with PokemonGrowthRate, the relations with the relationships using selectInload, and default_order_by with the value 'order'.

#### 11.2. Domain Schema:
File location: `mach-api/app/domain/pokemon_growth_rate/schema.py`
Create a Pydantic model `PokemonGrowthRateSchema` that represents a PokemonGrowthRate.
Create a Pydantic model `PokemonGrowthRateFilterPageSchema` that implements `FilterPage` from `mach-api/app/shared/schemas.py`.

#### 11.3. Domain Service: `PokemonGrowthRateService`
File location: `mach-api/app/domain/pokemon_growth_rate/service.py`
Create a service class `PokemonGrowthRateService` that implements BaseService[PokemonGrowthRateRepository,PokemonGrowthRate] from `mach-api/app/core/service/base.py`.

This service should include methods for retrieving, creating, updating, and deleting PokemonGrowthRate entities, as well as any additional business logic related to growth rates.
- `add(growth_rate: PokemonGrowthRateResponseSchema) -> PokemonGrowthRate` — search in database for existing growth rate if not exist call `PokeApiClient.get_pokemon_growth_rate(order: int)` and call `PokemonGrowthRateBusiness.ensure_description_message(descriptions: list[PokemonExternalGrowthRateDescriptionSchemaResponse])` save in database and return the added growth rate.
**`PokemonGrowthRateResponseSchema` has the shape: `name: str, url: str, formula: str, descriptions: list[ { description: str, language: { name: str } } ]`**
Logic:
1. Extract `order` from `growth_rate['url']` using `ensure_order_number` from `mach-api/app/utils/number.py`
2. Query `PokemonGrowthRateRepository.find_by(order=order)` If found, return it.
3. If not found, call `PokeApiClient.get_growth_rate(order: int)`.
4. Map fields:
  - `url` → `growth_rate_data["url"]`
  - `name` → `data["name"]`
  - `formula` → `data["formula"]`
  - `description` → first English entry in `data["descriptions"]` where `language.name == "en"`, field `description`; fallback to `""`
5. If successful, extract the `descriptions` from the response and call `PokemonGrowthRateBusiness.ensure_description_message(descriptions: list[PokemonExternalGrowthRateDescriptionSchemaResponse])` to get the description message. Create a new `PokemonGrowthRate` instance with the retrieved data and save it in the database using `PokemonGrowthRateRepository.save()`. Return the newly added growth rate.
6. If the API call is successful, extract the `descriptions` from the response and call `PokemonGrowthRateBusiness.ensure_description_message(descriptions: list[PokemonExternalGrowthRateDescriptionSchemaResponse])` to get the description message. Merge the description message with the other properties of the growth rate and save it in the database using `PokemonGrowthRateRepository.save()`. Return the newly added growth rate.
   - `get(param: str) -> PokemonGrowthRateSchema | None` — returns a pokemon growth rate by id or name
   - `list(page_filter: Annotated[PokemonGrowthRateFilterPageSchema, Query()] = None) -> CustomLimitOffsetPage[PokemonGrowthRateSchema] | list[PokemonGrowthRateSchema]` — returns a paginated list of pokemon growth rates or a list of pokemon growth rates with cached data, optionally filtered by name (case-insensitive, partial match).
   - `soft_delete(id: str)` — Soft delete a pokemon growth rate by id. Raise `HTTP 404 Not Found` if no pokemon growth rate is found with the given id. Set status to DELETED.

#### 11.4. Domain Business: `PokemonGrowthRateBusiness`
File location: `mach-api/app/domain/pokemon_growth_rate/business.py`:
Create a class `PokemonGrowthRateBusiness` with the following method:
- `ensure_description_message(descriptions: list[PokemonExternalGrowthRateDescriptionSchemaResponse]) -> str` — receives the `descriptions` list from the external API response and returns a string with the description message in English. If no English entry is found, return the first entry in any language. If the list is empty, return "No description information available".

#### 11.5. Domain Route:
File location: `mach-api/app/domain/pokemon_growth_rate/route.py`
Create a FastAPI router `PokemonGrowthRateRoute` with the following endpoints:
##### 11.5.1. `GET: /pokemon-growth-rate -> CustomLimitOffsetPage[PokemonGrowthRateSchema] | list[PokemonGrowthRateSchema]` — returns a paginated list of pokemon moves or a list of pokemon moves with cached data, optionally filtered by name (case-insensitive, partial match).
- **Method:** GET
- **Path:** `/pokemon-growth-rate`
- **Query Parameters:**
    - `page` (optional): Number of the page to retrieve (default: 1).
    - `limit` (optional): Number of items to return per page (default: 10).
    - `offset` (optional): Number of items to skip before starting to collect the result set (default: 0).
    - `order_by` (optional): Field to order the results by (default: "order").
- **Response:** `CustomLimitOffsetPage[PokemonGrowthRateSchema]` or `list[PokemonGrowthRateSchema]` with status `HTTP 200`
- **Auth required:** Yes 
- **Description:** Returns a paginated list of pokemon growth-rates or a list of pokemon growth-rates with cached data,

##### 11.5.2. `GET: /pokemon-growth-rate/{param} -> PokemonGrowthRateSchema` — returns a pokemon move by name or id.
- **Method:** GET
- **Path:** `/pokemon-growth-rate/{param}`
- **Path Parameters:** `param`: The name or ID of the pokemon move to retrieve.
- **Response:** `PokemonGrowthRateSchema` with status `HTTP 200`
- **Auth required:** Yes
- **Description:** Returns a pokemon growth_rate by name or id. Raises `HTTP 404 Not Found` if no pokemon move is found with the given name or id.

##### 11.5.3. `DELETE: /pokemon-growth-rate/{id} -> str` — soft deletes a pokemon move
- **Method:** DELETE
- **Path:** `/pokemon-growth-rate/{id}`
- **Path Parameters:** `id`: The ID of the pokemon move to soft delete.
- **Response:** `str` with status `HTTP 200`
- **Auth required:** Yes
- **Description:** Soft deletes a pokemon growth_rate by ID. Raises `HTTP 404 Not Found` if no pokemon growth_rate is found with the given ID. Sets the status of the pokemon growth_rate to DELETED.

---

#### 10.6. Router Registration
Register the `PokemonGrowthRateRoute` in `mach-api/app/main.py` under the `/pokemon-growth-rate` path.
```python
app.include_router(pokemon_growth_rate_router, prefix="/pokemon-growth-rate", tags=["PokemonGrowthRate"])
```
---

---

### 12. Testing
#### 12.1. Create all unit tests for the domain layer
#### 12.2. Create all unit tests for the application layer
#### 12.3. Create all unit tests for the infrastructure layer
#### 12.4. Create all unit tests for the presentation layer
#### 12.5 Rules
- mach-api Use: pytest pytest-asyncio httpx
- Cover: Models Repositories Services Routers
- Critical Cases: Cache hit/miss, Completion pipeline, Recursive evolution (no loop), External API failure handling, Soft Delete

## SPEC: mach-web

---
### 1. Page (pokemon)
File location: `mach-web/app/(protected)/pokemon/page.tsx`
Create a new page component `PokemonPage` that imports the `PokemonList` component from `mach-web/app/ui/features/list/pokemon-list.tsx`.

### 2. api/pokemon
#### 2.1 List
File location: `mach-web/app/api/pokemon/route.ts`
Create a new API route that handles requests to `/api/pokemon` and returns a list of pokemons. This route should call the `PokemonService.list()` method to retrieve the data and return it in the response.
Logic: 
- Get searchParams from new URL(request.url).searchParams
- Get session `getServerSession` from `mach-web/app/shared/lib/auth/server`
- if session.isAuthenticated is false or session.token is undefined return Response(status=401)
- Get searchParamsPage from searchParams.get('page')
- If searchParamsPage is not defined, set page to undefined
- If searchParamsPage is defined, set page with call `toPositiveInteger` from `mach-web/app/utils/number.ts` to convert it to a positive integer 
- Get searchParamsLimit from searchParams.get('limit')
- If searchParamsLimit is not defined, set limit to undefined
- If searchParamsLimit is defined, set limit with call `toPositiveInteger` from `mach-web/app/utils/number.ts` to convert it to a positive integer 
- Get name from searchParams.get('name')
- If name is not defined, set name to undefined
- If name is defined, set name to searchParams.get('name') and call `sanitizedParams` from `mach-web/app/utils/url.ts` to clean string
- Get order_by from searchParams.get('order_by')
- If order_by is not defined, set order_by to undefined
- If order_by is defined, set order_by to searchParams.get('order_by') and call `sanitizedParams` from `mach-web/app/utils/url.ts` to clean string
- Get order from searchParams.get('order')
- If order is not defined, set order to undefined
- If order is defined, set order to searchParams.get('order') and call `sanitizedParams` from `mach-web/app/utils/url.ts` to clean string
- Get status from searchParams.get('status')
- If status is not defined, set status to undefined
- If status is defined, set status to searchParams.get('status') and call `sanitizedParams` from `mach-web/app/utils/url.ts` to clean string
- Call `PokemonService.list(page=page, limit=limit, name=name, order_by=order_by, order=order, status=status)` to get the list of pokemons
- Return the list of pokemons in the response with status `HTTP 200`
- If the API call is unsuccessful, return an error message in the response with status `HTTP 500` and a descriptive message.
#### 2.2 Get by name
File location: `mach-web/app/api/pokemon/[name]/route.ts`
Create a new API route that handles requests to `/api/pokemon/{name}` and returns a pokemon by name. This route should call the `PokemonService.get_by_name(name: str)` method to retrieve the data and return it in the response.
Logic:
- Get session `getServerSession` from `mach-web/app/shared/lib/auth/server`
- if session.isAuthenticated is false or session.token is undefined return Response(status=401)
- Get name from params.name
- If name is not defined, return an error message in the response with status `HTTP 400` and a descriptive message.
- If name is defined, set name to params.name and call `sanitizedParams` from `mach-web/app/utils/url.ts` to clean string
- Call `PokemonService.get_by_name(name=name)` to get the pokemon by name
- If the pokemon is found, return the pokemon in the response with status `HTTP 200`
- If no pokemon is found with the given name, return an error message in the response with status `HTTP 404` and a descriptive message.
- If the API call is unsuccessful, return an error message in the response with status `HTTP 500` and a descriptive message.



### 3. UI
#### 3.1 PokemonCard
File location: `mach-web/app/ui/components/pokemon-card.tsx`
Create a new component `PokemonCard` that renders a card for a single pokemon. The card should display the pokemon's name, types, and a thumbnail image. The card should be styled using Tailwind CSS and should be responsive to different screen sizes. The card should also include a link that navigates to the pokemon's detail page at `/pokemon/{name}` when clicked.
- This component should be a responsive card,
- where at the top there should be an image using componet `Image` from `mach-web/app/ds/components/image`,
- If status is COMPLETE show Image with pokemon image,
- If status is INCOMPLETE show a default Image with Pokeball,
- below the order number of the Pokémon using function `formatNumberPrefix` from `mach-web/app/ui/utils/formatNumberPrefix.ts`
- And the name of the Pokémon, if status is INCOMPLETE show with gray color if status is COMPLETE show with black 
- If status is COMPLETE show badges using `<Badge>` from `mach-web/app/ds/components/badge` indicating the types.
- If status is INCOMPLETE show badges using `<Badge>` from `mach-web/app/ds/components/badge` indicating the status INCOMPLETE in gray color.
If the status is incomplete, instead of loading the image, it should load an image of a Poké Ball, and in place of the badges, it should only display an INCOMPLETE badge.

#### 3.1 usePokemonList
File location: `mach-web/app/ui/features/list/usePokemonList.ts`
Create a new hook `usePokemonList` that returns the data and functions to fetch the list of pokemons.
This hook should build with `usePaginatedList` from `mach-web/app/ui/hooks/list/usePaginatedList.ts`.

#### 3.2 PokemonList
File location: `mach-web/app/ui/features/list/pokemon-list.tsx`
Create a new component `PokemonList` that uses the `usePokemonList` hook to display a list of pokemons. This component should include pagination controls and a search input to filter pokemons by name. The search input should update the query parameters in the URL to reflect the current search term. The component should also handle loading and error states appropriately, displaying a loading indicator while data is being fetched and an error message if the API call fails. Each pokemon item in the list should display the pokemon's name, types, and a thumbnail image. Clicking on a pokemon item should navigate to the pokemon's detail page at `/pokemon/{name}`.
Implement in this component `Filters` from `mach-web/app/ds/components/filters.tsx`
Implement in this component `Pagination` from `mach-web/app/ds/components/pagination.tsx`
Implement in this component `PokemonCard` from `mach-web/app/ui/components/pokemon-card.tsx`

### 4. Testing
#### 4.1 Create all unit tests for the components
- Create unit tests for the `PokemonCard` component.
- Create unit tests for the `usePokemonList` hook.
- Create unit tests for the `PokemonList` component.
- Create unit tests for the `Filters` component.
- Create unit tests for the `Pagination` component.
#### 4.2 Rules
- mach-web Use: Jest, React Testing Library
- Cover: Components, Pagination, Navigation Rules, Image fallback, API mocked

---

## Coverage Rules
- Minimum:95%
- Focus on real behavior
- Avoid trivial tests
- Mock: API, REDIS

## General Rules
- Do not modify files unrelated to this spec.
- All async functions must use `async/await`.
- All database queries must use SQLAlchemy `AsyncSession`.
- No raw SQL — use SQLAlchemy ORM only.
- The Redis client must be injected as a FastAPI dependency.
- The `PokeApiClient` must be injected as a FastAPI dependency.
- Error handling must return meaningful HTTP status codes and messages.
- Do not modify the `User` or `Trainer` models or any auth-related files.
- Use Tailwind and custom components from `/ds`.
- Inject dependencies (Redis + PokeAPI)
- Handle errors properly
- Prevent infinite recursion in evolution chain
- Do not touch auth modules
- Do not modify the `/ds` components

## Testing Rules

- Use pytest for API testing
- Use Jest and React Testing Library for web testing
- Cover all critical cases in both API and web testing
- Mock external dependencies (API, REDIS) in tests
- Ensure coverage meets minimum requirement
- Focus on real behavior and avoid trivial tests