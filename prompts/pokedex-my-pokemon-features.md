# OpenSpec Prompt — Pokedex My Pokemon Features
## READ:
   [AGENTS SPEC](../AGENTS.md) 
   [README SPEC](../README.md) 
   [AGENTS API](../mach-api/AGENTS.md) 
   [README API](../mach-api/README.md)
   [AGENTS WEB](../mach-web/AGENTS.md)
   [README WEB](../mach-web/README.md)
    

## Context

This project uses a monorepo with two submodules:

- **mach-api** — Python 3.13, FastAPI, SQLAlchemy async, domain-driven architecture
- **mach-web** — Next.js 16, React 19, TypeScript, Tailwind CSS v4

## Objective
Create a **Pokedex List Page** in `mach-web` that is accessed when clicking the **Pokedex** button in the sidebar.
Create a **Pokedex Details Page** in `mach-web` that is accessed when clicking a Pokedex card **ONLY if its discovered is `true`**.
Create a **MyPokemon List Page** in `mach-web` that is accessed when clicking the **My Pokemon** button in the sidebar.
Create a **MyPokemon Details Page** in `mach-web` that is accessed when clicking a Pokemon card **ONLY if its status is `COMPLETE`**.

---
## SPEC: mach-api

### 1. Database Models

#### 1.1 Model `Pokedex`
File location: `mach-api/src/models/pokedex.py`

Create a SQLAlchemy async model named `Pokedex` mapped to the table `pokedex`. with the following columns:

| Column               | Type     | Constraints                                                           |
|----------------------|----------|-----------------------------------------------------------------------|
| `id`                 | UUID     | Primary key, default `uuid4`, not nullable                            |
| `hp`                 | Integer  | not nullable                                                          |
| `iv_hp`              | Integer  | not nullable                                                          |
| `ev_hp`              | Integer  | not nullable                                                          |
| `wins`               | Integer  | not nullable                                                          |
| `level`              | Integer  | not nullable                                                          |
| `losses`             | Integer  | not nullable                                                          |
| `max_hp`             | Integer  | not nullable                                                          |
| `battles`            | Integer  | not nullable                                                          |
| `nickname`           | String   | not nullable                                                          |
| `speed`              | Integer  | not nullable                                                          |
| `iv_speed`           | Integer  | not nullable                                                          |
| `ev_speed`           | Integer  | not nullable                                                          |
| `attack`             | Integer  | not nullable                                                          |
| `iv_attack`          | Integer  | not nullable                                                          |
| `ev_attack`          | Integer  | not nullable                                                          |
| `defense`            | Integer  | not nullable                                                          |
| `iv_defense`         | Integer  | not nullable                                                          |
| `ev_defense`         | Integer  | not nullable                                                          |
| `experience`         | Integer  | not nullable                                                          |
| `special_attack`     | Integer  | not nullable                                                          |
| `iv_special_attack`  | Integer  | not nullable                                                          |
| `ev_special_attack`  | Integer  | not nullable                                                          |
| `special_defense`    | Integer  | not nullable                                                          |
| `iv_special_defense` | Integer  | not nullable                                                          |
| `ev_special_defense` | Integer  | not nullable                                                          |
| `discovered`         | Boolean  | Nullable, default `False`                                             |
| `formula`            | String   | not nullable                                                          |
| `discovered_at`      | DateTime | Nullable, default `None`, **only set on change discovered attribute** |
| `created_at`         | DateTime | Not nullable, default `utcnow`, **never updated after insert**        |
| `updated_at`         | DateTime | Nullable, default `None`, **only set on second or later updates**     |
| `deleted_at`         | DateTime | Nullable, default `None`, **only set when soft-delete is triggered**  |
| `pokemon_id`         | UUID     | Foreign key → `pokemons.id`, Not nullable                             |
| `trainer_id`         | UUID     | Foreign key → `trainers.id`, Not nullable                             |

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


| Column               | Type     | Constraints                                                          |
|----------------------|----------|----------------------------------------------------------------------|
| `id`                 | UUID     | Primary key, default `uuid4`, not nullable                           |
| `hp`                 | Integer  | not nullable                                                         |
| `iv_hp`              | Integer  | not nullable                                                         |
| `ev_hp`              | Integer  | not nullable                                                         |
| `wins`               | Integer  | not nullable                                                         |
| `level`              | Integer  | not nullable                                                         |
| `losses`             | Integer  | not nullable                                                         |
| `max_hp`             | Integer  | not nullable                                                         |
| `battles`            | Integer  | not nullable                                                         |
| `nickname`           | String   | not nullable                                                         |
| `speed`              | Integer  | not nullable                                                         |
| `iv_speed`           | Integer  | not nullable                                                         |
| `ev_speed`           | Integer  | not nullable                                                         |
| `attack`             | Integer  | not nullable                                                         |
| `iv_attack`          | Integer  | not nullable                                                         |
| `ev_attack`          | Integer  | not nullable                                                         |
| `defense`            | Integer  | not nullable                                                         |
| `iv_defense`         | Integer  | not nullable                                                         |
| `ev_defense`         | Integer  | not nullable                                                         |
| `experience`         | Integer  | not nullable                                                         |
| `special_attack`     | Integer  | not nullable                                                         |
| `iv_special_attack`  | Integer  | not nullable                                                         |
| `ev_special_attack`  | Integer  | not nullable                                                         |
| `special_defense`    | Integer  | not nullable                                                         |
| `iv_special_defense` | Integer  | not nullable                                                         |
| `ev_special_defense` | Integer  | not nullable                                                         |
| `formula`            | String   | not nullable                                                         |
| `capture_at`         | DateTime | not nullable, default `utcnow`, **never updated after insert**       |
| `created_at`         | DateTime | Not nullable, default `utcnow`, **never updated after insert**       |
| `updated_at`         | DateTime | Nullable, default `None`, **only set on second or later updates**    |
| `deleted_at`         | DateTime | Nullable, default `None`, **only set when soft-delete is triggered** |
| `pokemon_id`         | UUID     | Foreign key → `pokemons.id`, Not nullable                            |
| `trainer_id`         | UUID     | Foreign key → `trainers.id`, Not nullable                            |

**Relationships on `MyPokemon`:**
- `pokemon`: One-to-one relationship with `Pokemon` model, accessed via `my_pokemon.pokemon`
- `trainer`: One-to-one relationship with `Trainer` model, accessed via `my_pokemon.trainer`
- `moves` → many-to-many relationship to `PokemonMove` via association table `my_pokemon_pokemon_moves`

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

---

### 3. Trainer
#### 3.1 Domain Schema
File location: `mach-api/src/domain/trainer/schema.py`

Should add in `InitializeTrainerResponse` the following fields:
- `pokemon_name: str | None = None`:

#### 3.2 Domain Repository `TrainerRepository`
File location: `mach-api/src/domain/trainer/repository.py`

This repository should populate the relations with the relationships using selectInLoad.

#### 3.3 Domain Service `TrainerService`
File location: `mach-api/src/domain/trainer/service.py`

Should add in `TrainerService.initialize` method this logic:
 - After create a new trainer
 - Verify if `Pokemon` is initialize em database with `pokemon_service.list`
 - Obtain the first Pokémon with the `pokemon_name` attribute. If the attribute is not set, obtain a random Pokémon, preferably one with the `COMPLETE` status. Only use those without the `COMPLETE` status if you don't have any with it.
 - Initialize a pokedex for the trainer with the obtained Pokémon, using the `pokedex_service.initialize` method. The pokedex should be initialized with the Pokémon's base stats and the trainer ID and first pokemon.
 - Create a `MyPokemon` for the trainer with the obtained Pokémon, using the `my_pokemon_service.capture` method. The `MyPokemon` should be initialized with the first pokemon and the trainer ID.
 - return Trainer with the pokedex and my_pokemon initialized.

#### 3.4 Domain Route `TrainerRoute`
File location: `mach-api/src/domain/trainer/route.py`

Don't touch this file.

### 4. Pokedex
#### 4.1 Domain Schema
File location: `mach-api/src/domain/pokedex/schema.py`

Create a Pydantic model `PokedexSchema` that represents a Pokedex.
Create a Pydantic model `PokedexInitializeSchema` with `trainer_id: int` and `pokemon: PokemonSchema` and `pokemons: list[PokemonSchema]`.
Create a Pydantic model `PokedexFilterPageSchema` that implements `FilterPage` from `mach-api/app/shared/schemas.py`. 
Create a Pydantic model `PokedexUpdateSchema` that represents a Pokedex with partial data.

#### 4.2 Domain Repository `PokedexRepository`
File location: `mach-api/src/domain/pokedex/repository.py`

Create a repository class `PokedexRepository` that implements BaseRepository[Pokedex] from `mach-api/app/core/repository/base.py`.

This repository should populate the model with Pokedex, the relations with the relationships using selectInload, and default_order_by with the value `discovered_at`.

#### 4.3 Domain Service `PokedexService`
File location: `mach-api/src/domain/pokedex/service.py`

Create a service class `PokedexService` that implements BaseService[PokedexRepository,Pokedex] from `mach-api/app/core/service/base.py`.

This service should implement the following methods:
- `initialize`: Initializes a pokedex for a trainer with a pokemon. It should receive a `PokedexInitializeSchema` and return a `PokedexSchema`. It should use the `PokedexRepository` to create the pokedex and return it.
Logic:
1. The program iterates through the matrix of `pokemons`, retrieving each item individually. Each item is then called an item.
2. Query `PokedexRepository.find_by(trainer_id=trainer_id)` If found
3. Verify if the item has `name` equal from attribute `pokemon.name` and attribute 'discovered' equal to `True`.
4. If yes, return it and continue the iterations.
5. If not, call `PokemonProgressionBusiness.initialize_stats` with the attribute `pokemon` to obtain the stats.
6. If the resutl of `PokemonProgressionBusiness.initialize_stats` with attribute `name` equal from attribute `pokemon.name` set attribute 'discovered' to `True`.
7. Create a new pokedex with the obtained stats and the trainer ID and the pokemon.
8. Return the pokedex and continue the iterations.

- `get(id: str) -> PokedexSchema | None` — returns a pokedex by id
- `list(page_filter: Annotated[PokedexFilterPageSchema, Query()] = None, trainer_id: str) -> CustomLimitOffsetPage[PokedexSchema] | list[PokedexSchema]` — returns a paginated list of pokedex or a list of pokedex with cached data, required filtered by trainer_id and optionally filtered by name (case-insensitive, partial match).
- `soft_delete(id: str)` — Soft delete a pokedex by id. Raise `HTTP 404 Not Found` if no pokedex is found with the given id. Set status to DELETED.
- `update(id: str, data: PokedexUpdateSchema)` — Update a pokedex by id. Raise `HTTP 404 Not Found` if no pokedex is found with the given id. Sets `updated_at` to the current UTC datetime if the pokedex is updated successfully.

#### 4.4 Domain Route `PokedexRoute`
File location: `mach-api/src/domain/pokedex/route.py`
Create a FastAPI router `PokedexRoute` with the following endpoints:

##### 4.4.1. `GET: /pokedex -> CustomLimitOffsetPage[PokedexSchema] | list[PokedexSchema]` — returns a paginated list of pokedex or a list of pokedex with cached data, optionally filtered by name (case-insensitive, partial match).
- **Method:** GET
- **Path:** `/pokedex`
- **Query Parameters:**
    - `page` (optional): Number of the page to retrieve (default: 1).
    - `limit` (optional): Number of items to return per page (default: 10).
    - `offset` (optional): Number of items to skip before starting to collect the result set (default: 0).
    - `order_by` (optional): Field to order the results by (default: "order").
- **Response:** `CustomLimitOffsetPage[PokedexSchema]` or `list[PokedexSchema]` with status `HTTP 200`
- **Auth required:** Yes
- **Description:** Returns a paginated list of pokedex or a list of pokedex with cached data,

##### 4.4.2. `GET: /pokedex/{param} -> PokedexSchema` — returns a pokedex by name or id.
- **Method:** GET
- **Path:** `/pokedex/{param}`
- **Path Parameters:** `param`: The name or ID of the pokedex to retrieve.
- **Response:** `PokedexSchema` with status `HTTP 200`
- **Auth required:** Yes
- **Description:** Returns a pokedex by name or id. Raises `HTTP 404 Not Found` if no pokedex is found with the given name or id.

##### 4.4.3. `DELETE: /pokedex/{id} -> str` — soft deletes a pokedex
- **Method:** DELETE
- **Path:** `/pokedexs/{id}`
- **Path Parameters:** `id`: The ID of the pokedex to soft delete.
- **Response:** `str` with status `HTTP 200`
- **Auth required:** Yes
- **Description:** Soft deletes a pokedex by ID. Raises `HTTP 404 Not Found` if no pokedex is found with the given ID. Sets the status of the pokedex to DELETED.

##### 4.4.4. `PATH OR PUT: /pokedex/{id} -> PokedexSchema}` — updates a pokedex
- **Method:** PATH OR PUT
- **Path:** `/pokedex/{id}`
- **Path Parameters:** `id`: The ID of the pokedex to update.
- **Request Body:** `PokedexUpdateSchema` with the fields to update.
- **Response:** `PokedexSchema` with status `HTTP 200`
- **Auth required:** Yes
- **Description:** Updates a pokedex by ID. Raises `HTTP 404 Not Found` if no pokedex is found with the given ID. Updates the fields of the pokedex with the provided data. Sets `updated_at` to the current UTC datetime if the pokedex is updated successfully.

#### 4.5 Router Registration
Register the `PokedexRoute` in `mach-api/app/main.py` under the `/pokedex` path.
```python
app.include_router(pokedex_router, prefix="/pokedex", tags=["Pokedex"])
```

### 5. MyPokemon
#### 5.1 Domain Schema
File location: `mach-api/src/domain/my_pokemon/schema.py`

Create a Pydantic model `MyPokemonSchema` that represents a MyPokemon.
Create a Pydantic model `MyPokemonCreateSchema` with `trainer_id: int` and `pokemon: PokemonSchema`.
Create a Pydantic model `MyPokemonFilterPageSchema` that implements `FilterPage` from `mach-api/app/shared/schemas.py`.
Create a Pydantic model `MyPokemonUpdateSchema` that represents a MyPokemon with partial data.

#### 5.2 Domain Repository `MyPokemonRepository`
File location: `mach-api/src/domain/my_pokemon/repository.py`

Create a repository class `MyPokemonRepository` that implements BaseRepository[Pokedex] from `mach-api/app/core/repository/base.py`.

This repository should populate the model with MyPokemon, the relations with the relationships using selectInload, and default_order_by with the value `captured_at`.

#### 5.3 Domain Service `MyPokemonService`
File location: `mach-api/src/domain/my_pokemon/service.py`

Create a service class `MyPokemonService` that implements BaseService[MyPokemonRepository,MyPokemon] from `mach-api/app/core/service/base.py`.
This service should implement the following methods:
- `create`: Captures a pokemon for a trainer. It should receive a `MyPokemonCreateSchema` and return a `MyPokemonSchema`. It should use the `MyPokemonRepository` to create the my_pokemon and return it.
  Logic:
  1. Query `MyPokemonRepository.find_by(trainer_id=trainer_id, pokemon_id=pokemon.id)` If found
  2. Verify if the item has `pokemon` equal from attribute `pokemon`.
  3. If yes, return it.
  4. If not, call `PokemonProgressionBusiness.initialize_stats` with the attribute `pokemon` to obtain the stats.
  5. Create a new MyPokemon with the obtained stats and the trainer ID and the pokemon with `MyPokemonRepository.save()`.
  6. Set moves call `_select_random_moves(moves=pokemon.moves)` with the attribute `pokemon.moves` to get the moves. 
  7. Set the moves to the MyPokemon with `MyPokemonRepository.update()`. 
  8. Return the MyPokemon and continue the iterations.
- `get(id: str) -> MyPokemonSchema | None` — returns a MyPokemon by id
- `list(page_filter: Annotated[MyPokemonFilterPageSchema, Query()] = None, trainer_id: str) -> CustomLimitOffsetPage[MyPokemonSchema] | list[MyPokemonSchema]` — returns a paginated list of MyPokemon or a list of MyPokemon with cached data, required filtered by trainer_id and optionally filtered by name (case-insensitive, partial match).
- `soft_delete(id: str)` — Soft delete a MyPokemon by id. Raise `HTTP 404 Not Found` if no MyPokemon is found with the given id. Set status to DELETED.
- `update(id: str, data: MyPokemonUpdateSchema)` — Update a MyPokemon by id. Raise `HTTP 404 Not Found` if no MyPokemon is found with the given id. Sets `updated_at` to the current UTC datetime if the MyPokemon is updated successfully.
- `_select_random_moves(moves: list[PokemonMoveSchema], count: int = 4) -> list[PokemonMoveSchema]`: Selects a random subset of moves from the given list of moves. The number of moves to select is determined by the `count` parameter, which defaults to 4. The method should return a list of randomly selected moves.
#### 5.4 Domain Route `MyPokemonRoute`
File location: `mach-api/src/domain/my_pokemon/route.py`
Create a FastAPI router `MyPokemonRoute` with the following endpoints:

##### 5.4.1. `GET: /pokedex -> CustomLimitOffsetPage[MyPokemonSchema] | list[MyPokemonSchema]` — returns a paginated list of my-pokemon or a list of my-pokemon with cached data, optionally filtered by name (case-insensitive, partial match).
- **Method:** GET
- **Path:** `/my-pokemon`
- **Query Parameters:**
    - `page` (optional): Number of the page to retrieve (default: 1).
    - `limit` (optional): Number of items to return per page (default: 10).
    - `offset` (optional): Number of items to skip before starting to collect the result set (default: 0).
    - `order_by` (optional): Field to order the results by (default: "order").
- **Response:** `CustomLimitOffsetPage[MyPokemonSchema]` or `list[MyPokemonSchema]` with status `HTTP 200`
- **Auth required:** Yes
- **Description:** Returns a paginated list of my-pokemon or a list of my-pokemon with cached data,

---

## Explore Notes

### 1. Modelo de domínio

**Pokedex**
- É por espécie e por treinador
- Chave lógica: `(trainer_id, pokemon_id)`
- Guarda progressão própria desde a inicialização
- Guarda dados de batalha/progressão:
  - level
  - IV/EV
  - wins/losses
  - demais stats definidos no modelo
- Não terá `nickname`
- Não terá moves próprios persistidos
- Pode exibir no detail os moves possíveis da espécie vindos de `pokemon.moves`
- Só o campo `discovered` controla visibilidade funcional do detail

**MyPokemon**
- É instância capturada
- Permite múltiplos registros do mesmo `(trainer_id, pokemon_id)`
- Guarda progressão própria, totalmente apartada da `Pokedex`
- Guarda:
  - nickname
  - level
  - IV/EV
  - wins/losses
  - stats
  - moves equipados
- Não terá campo `status`
- O gating do detail usa `my_pokemon.pokemon.status == COMPLETE`

### 2. Relação entre Pokedex e MyPokemon

Quando um Pokémon é capturado:
- cria `MyPokemon`
- se a espécie ainda não estiver descoberta, `Pokedex.discovered = true`
- se já estiver descoberta, não precisa alterar a entrada da `Pokedex`

A descoberta futura:
- terá um serviço próprio depois
- por enquanto, só acontece na inicialização do treinador para o starter

### 3. Inicialização do treinador

Na inicialização:
- deve existir uma lista de `Pokedex` com todos os Pokémon da base
- todos os itens da `Pokedex` já nascem com progressão inicial
- todos começam com `discovered = false`
- exceto o starter, que nasce com `discovered = true`
- deve criar apenas um `MyPokemon` inicial para o treinador
- tanto `Pokedex` quanto `MyPokemon` começam com progressão desde o início

A base de inicialização da `Pokedex`:
- usa stats base do catálogo para cada espécie
- não usa a mesma lógica de randomização/instância do `MyPokemon`

Quando uma espécie for descoberta depois:
- apenas altera `discovered = true`
- define `discovered_at`
- preserva a progressão já existente

### 4. Estratégia assíncrona da Pokedex

Foi assumida a estratégia com `BackgroundTasks`.

Fluxo:
- `POST /trainers/initialize`
  - cria trainer se necessário
  - cria `MyPokemon` inicial se necessário
  - dispara a carga da `Pokedex` em background
  - responde sem bloquear o processo inteiro

`Trainer` terá um novo campo:
- `pokedex_status`

Valores:
- `EMPTY`
- `INITIALIZING`
- `READY`
- `FAILED`

Regras:
- trainer novo começa com `pokedex_status = EMPTY`
- quando a carga inicia, vira `INITIALIZING`
- quando termina com sucesso, vira `READY`
- se falhar, vira `FAILED`

Retry:
- continua no mesmo `/trainers/initialize`
- se o trainer já existir:
  - só tenta reinicializar a `Pokedex` se `pokedex_status` for `EMPTY` ou `FAILED`
  - não recria o `MyPokemon`

Idempotência:
- se a entrada `(trainer_id, pokemon_id)` já existir na `Pokedex`, mantém
- se não existir, cria

Falha:
- a carga da `Pokedex` deve rodar em transação única
- em erro, faz rollback total da tentativa
- os parciais não devem ficar persistidos
- não usar soft delete para esse rollback técnico

### 5. Regras de UI para Pokedex

**Lista `/pokedex`**
- mostra todos os registros do treinador desde o início
- se `discovered = false`:
  - nome acinzentado
  - imagem padrão de pokebola
  - badge `Not Discovered`
  - sem acesso ao detail
- `pokemon.status` não participa da regra visual da `Pokedex`
- se `discovered = true`:
  - mostra nome real
  - mostra imagem real
  - mostra tipos principais em `<Badge />`
  - sem weakness/strength no card
  - pode mostrar também order, level, wins/losses, discovered_at

**Detail `/pokedex/[id]`**
- abre por `id`
- só abre quando `discovered = true`
- mostra:
  - imagem
  - nome
  - tipos
  - progressão completa
  - `discovered_at`
  - moves possíveis da espécie
  - evolução/timeline

### 6. Regras de UI para MyPokemon

**Lista `/my-pokemon`**
- detalhe abre por `id`
- campos essenciais do card:
  - nickname
  - `pokemon.name`
  - level
  - wins/losses
  - badges dos tipos
  - imagem real somente se `pokemon.status == COMPLETE`

**Detail `/my-pokemon/[id]`**
- título principal = `nickname`
- subtítulo = nome da espécie
- mostra:
  - imagem
  - tipos
  - progressão completa
  - moves equipados
  - evolução/timeline

Edição:
- permitir alteração somente de `nickname`
- via modal
- validação:
  - não pode ser vazio
  - mínimo de 3 caracteres

Moves:
- selecionados aleatoriamente no momento da captura
- persistidos como parte fixa da instância
- timeline de evolução é apenas informativa por enquanto

### 7. Regras de rotas/details

**Pokedex**
- detail por `id`
- só depende de `discovered = true`

**MyPokemon**
- detail por `id`
- depende de `my_pokemon.pokemon.status == COMPLETE`

**Pokemon / Pokedex / MyPokemon**
- todos os details devem mostrar evolução/timeline

### 8. UX de estados da Pokedex

Se `pokedex_status = INITIALIZING`:
- o frontend pode abrir `/pokedex`
- deve mostrar estado visual de preparação/carregamento

Se `pokedex_status = FAILED`:
- `/pokedex` mostra estado de erro
- com botão para chamar `/trainers/initialize` novamente

## Leitura geral

O desenho final ficou assim:

```text
Pokemon
  catálogo global compartilhado

Trainer
  possui estado da montagem da pokedex

Pokedex
  progresso por espécie para o treinador
  sem nickname
  sem moves equipados

MyPokemon
  criatura capturada, instanciada
  com nickname
  com moves equipados
  com progressão própria
```

##### 5.4.2. `POST: /my-pokemon -> MyPokemonSchema` — create a my-pokemon
- **Method:** POST
- **Path:** `/my-pokemon`
- **Request Body:** `MyPokemonCreateSchema` with the fields to create.
- **Response:** `MyPokemonSchema` with status `HTTP 201`.
- **Auth required:** Yes
- **Description:** Creates a new my-pokemon. Raises `HTTP 400 Bad Request` if the request body is invalid or if a my-pokemon with the same pokemon and trainer already exists.

##### 5.4.3. `GET: /my-pokemon/{param} -> MyPokemonSchema` — returns a my-pokemon by name or id.
- **Method:** GET
- **Path:** `/my-pokemon/{param}`
- **Path Parameters:** `param`: The name or ID of the my-pokemon to retrieve.
- **Response:** `MyPokemonSchema` with status `HTTP 200`
- **Auth required:** Yes
- **Description:** Returns a my-pokemon by name or id. Raises `HTTP 404 Not Found` if no my-pokemon is found with the given name or id.

##### 5.4.4. `DELETE: /my-pokemon/{id} -> str` — soft deletes a my-pokemon
- **Method:** DELETE
- **Path:** `/my-pokemons/{id}`
- **Path Parameters:** `id`: The ID of the my-pokemon to soft delete.
- **Response:** `str` with status `HTTP 200`
- **Auth required:** Yes
- **Description:** Soft deletes a my-pokemon by ID. Raises `HTTP 404 Not Found` if no my-pokemon is found with the given ID. Sets the status of the my-pokemon to DELETED.

##### 5.4.5. `PATH OR PUT: /my-pokemon/{id} -> PokedexSchema}` — updates a my-pokemon
- **Method:** PATH OR PUT
- **Path:** `/my-pokemon/{id}`
- **Path Parameters:** `id`: The ID of the my-pokemon to update.
- **Request Body:** `MyPokemonUpdateSchema` with the fields to update.
- **Response:** `MyPokemonSchema` with status `HTTP 200`
- **Auth required:** Yes
- **Description:** Updates a my-pokemon by ID. Raises `HTTP 404 Not Found` if no my-pokemon is found with the given ID. Updates the fields of the my-pokemon with the provided data. Sets `updated_at` to the current UTC datetime if the my-pokemon is updated successfully.

#### 5.5 Router Registration
Register the `MyPokemonRoute` in `mach-api/app/main.py` under the `/my-pokemon` path.
```python
app.include_router(my_pokemon_router, prefix="/my-pokemon", tags=["MyPokemon"])
```

### 6. Progression
#### 6.1 Domain Schema
File location: `mach-api/src/domain/progression/schema.py`

Create a Pydantic model `ProgressionInitializeSchema` with `pokemon: PokemonSchema`.
Create a Pydantic model `RegistrySchema` with:

| Column               | Type          |
|----------------------|---------------|
| `id`                 | UUID          |
| `hp`                 | Integer       |
| `iv_hp`              | Integer       |
| `ev_hp`              | Integer       |
| `wins`               | Integer       |
| `level`              | Integer       |
| `losses`             | Integer       |
| `max_hp`             | Integer       |
| `battles`            | Integer       |
| `nickname`           | String        |
| `speed`              | Integer       |
| `iv_speed`           | Integer       |
| `ev_speed`           | Integer       |
| `attack`             | Integer       |
| `iv_attack`          | Integer       |
| `ev_attack`          | Integer       |
| `defense`            | Integer       |
| `iv_defense`         | Integer       |
| `ev_defense`         | Integer       |
| `experience`         | Integer       |
| `special_attack`     | Integer       |
| `iv_special_attack`  | Integer       |
| `ev_special_attack`  | Integer       |
| `special_defense`    | Integer       |
| `iv_special_defense` | Integer       |
| `ev_special_defense` | Integer       |
| `formula`            | String        |
| `created_at`         | DateTime      |
| `updated_at`         | DateTime      |
| `deleted_at`         | DateTime      |
| `pokemon`            | PokemonSchema |

Create a Pydantic model `ProgressionSchema` that represents a Progression.

#### 6.2 Domain Business `ProgressionBusiness`
File location: `mach-api/src/domain/progression/business.py`

This business should implement the following methods:
- `initialize_stats`: Initializes the stats of a pokemon. It should receive a `ProgressionInitializeSchema` and return a `ProgressionSchema`. The stats should be initialized with the base stats of the pokemon and the level 1. The stats should be returned as a `ProgressionSchema`.
Logic: 
1. Verify if `pokemon` has status `COMPLETE`.
2. If not, return a default `ProgressionSchema` with the base stats of the pokemon and level 1.
3. If yes, Verify if `pokemon` has `growth_rate` attribute.
4. If not return a default `ProgressionSchema` with the base stats of the pokemon and level 1.
5. If yes, Generate `stats_evs` with `_initialize_evs` and `stats_ivs` with `_initialize_ivs`.
6. Call `_calculate_hp()` with `pokemon.hp`, `stats_ivs.hp`, `stats_evs.hp`, and level 1 to obtain the hp.
7. Call `_calculate_stats()` with `pokemon`, `stats_ivs`, `stats_evs`, and level 1 to obtain the attack, defense, speed, special_attack, and special_defense.
8. Return a `ProgressionSchema` with the base stats of the pokemon, the calculated stats, and level 1.
- `_calculate_hp(hp: int, iv_hp: int, ev_hp: int, level: int) -> int`: Calculates the hp of a pokemon with the formula `math.floor(((2 * hp + iv_hp + (ev_hp // 4)) * level) / 100) + level + 10`.
- `_calculate_stats(pokemon: PokemonSchema, stats_ivs: StatsBlock, stats_evs: StatsBlock, level: int) -> ProgressionSchema`: Calculate the stats attack, speed, defense, special_attack, special_defense, level and experience.
Logic:
1. Calculate attack attribute with `_calculate_stat()` with `level`, `stats_ivs.attack`, `stats_evs.attack`, `attack` 
2. Calculate defense attribute with `_calculate_stat()` with `level`, `stats_ivs.defense`, `stats_evs.defense`, `defense`
3. Calculate speed attribute with `_calculate_stat()` with `level`, `stats_ivs.speed`, `stats_evs.speed`, `speed`
4. Calculate special_attack attribute with `_calculate_stat()` with `level`, `stats_ivs.special_attack`, `stats_evs.special_attack`, `special_attack`
5. Calculate special_defense attribute with `_calculate_stat()` with `level`, `stats_ivs.special_defense`, `stats_evs.special_defense`, `special_defense`
6. Mount the `ProgressionSchema` with the base stats of the pokemon, the calculated stats, and the level.
- `_calculate_stat(level: int, iv: int, ev: int, base_stat: int) -> int`: Calculates the stat of a pokemon.
Logic:
1. Calculate the base_calc with the formula `2 * base_stat + iv_stat + (ev_stat // 4)`.
2. Calculate the scaled with the formula `(base_calc * level) / 100`
3. Calculate the stat with the formula `math.floor(scaled) + 5`
4. Return the final calculating with the formula `math.floor(stat * nature_multiple)`
- `_initialize_ivs()`: Initializes the ivs of a pokemon.
Logic:
1. Generate `hp` attribute with `random.randint(0, 31)`.
2. Generate `speed` attribute with `random.randint(0, 31)`.
3. Generate `attack` attribute with `random.randint(0, 31)`.
4. Generate `defense` attribute with `random.randint(0, 31)`.
5. Generate `special_attack` attribute with `random.randint(0, 31)`.
6. Generate `special_defense` attribute with `random.randint(0, 31)`.
7. Return a `StatsBlock` with the generated ivs.
- `_initialize_evs(max_total_ev: int = 510, max_ev_per_stat: int = 252)`: Initializes the ivs of a pokemon.
Logic:
1. set attribute `remaining` with the value `max_total_ev`.
2. Mount a default `StatsBlock`.
3. iterates through the matrix of `StatsBlock`, retrieving each item individually. Each item is then called an item.
4. if `remaining` is minor than 0, break the iterations.
5. if `remaining` is greater than 0, set `value` attribute with `random.randint(0, min(max_ev_per_stat, remaining))`
6. set `item.ev` attribute with `value`
7. rest `remaining` with `remaining - value`.
8. return the `StatsBlock` with the generated evs.
- `_calculate_experience(level: int,  base_experience: int, xp_multiplier: int = 7) -> int`: Calculates the experience of a pokemon with the formula `base_experience * max(1, level // xp_multiplier)`.
- `_calculate_level_from_experience(experience: int, current_level: int, formular: str) -> int`: Calculates the level of a pokemon based on the experience and the formula. The formula can be "SLOW", "MEDIUM_SLOW", "MEDIUM_FAST", or "FAST". The calculation is done with the following formulas:
Logic:
1. set `level` with `current_level` attribute.
2. initialize `while` with `True`.
3. set `experience_for_next_level` with result of function `calculate_by_formula(formula, level + 1)` from `mach-api/app/utils/number.py`.
4. if `experience_for_next_level` is greater than `experience`, break the loop and return the `level`.
5. if `experience_for_next_level` is minor than `experience`, increment `level` with 1 and continue the loop.
6. return the `level`.

### 7. Testing
#### 7.1. Create all unit tests for the domain layer
#### 7.2. Create all unit tests for the application layer
#### 7.3. Create all unit tests for the infrastructure layer
#### 7.4. Create all unit tests for the presentation layer
#### 7.5 Rules
- mach-api Use: pytest pytest-asyncio httpx
- Cover: Models Repositories Services Routers
- Critical Cases: Cache hit/miss, Completion pipeline, Recursive evolution (no loop), External API failure handling, Soft Delete

### 8. Architecture Rules
Follow strictly:

* route → service → repository → models
* No business logic in route
* Use schemas (Pydantic)
* Maintain consistency with domain structure
---

## SPEC: mach-web

### 1. Requirements
* The page must be:
    * Fully responsive
    * Visually rich
    * Reuse BaseService pattern
    * Follow same structure (state, loading, error)
    * Do NOT invent a new pattern
    * Keep consistency with existing codebase
    * Inspired by:
        * Pokedex UI
        * card-based layout
        * clean modern dashboards

### 2. UI/UX Guidelines

* Use TailwindCSS
* Use Design System components (`app/ds/`)
* Prefer composition
* Rounded cards
* Shadows
* Good spacing

### 3. Pokedex Structure
#### 3.1. Page
##### 3.1.1. List
File location: `mach-web/src/app/(protected)/pokedex/page.tsx`
Create the page based on the file called `mach-web/app/(protected)/pokemon/page.tsx`.
 * On click in Pokedex card:
   * if `item.discovered` is equal to `true` → redirect to `/pokedex/[name]`
   * if `item.discovered` is equal to `false` → do nothing
 * show Image in Pokedex card:
   * if `item.discovered` is equal to `true` → Show Image of pokemon
   * if `item.discovered` is equal to `false` → Show PokeBall image

##### 3.1.2. Detail
File location: `mach-web/src/app/(protected)/pokedex/[name]/page.tsx`
Create the page based on the file called `mach-web/app/(protected)/pokemon/[name]/page.tsx`.
###### 3.1.2.1. Header Section
Display:
* Pokemon Image (large): `item.pokemon.external_image || item.pokemon.image`
* Nickname: `item.nickname || item.pokemon.name || "Not Discovered"` 
* Name: `item.pokemon.name` : Place the attribute in parentheses with a smaller size; only display it if the `item.name` attribute exists and is not equal to `Nickname`. If it doesn't exist, don't display it.
* Order number: `item.pokemon.order`
* Short description (if available)
###### 3.1.2.2. Info Card
Display (if available):

* Height: `item.pokemon.height`
* Weight: `item.pokemon.weight`
* Habitat: `item.pokemon.habitat`
* Hatch Counter: `item.pokemon.capture_rate`

📌 IMPORTANT:
If any of these fields do not exist in the API, feel free to:

* Omit the field OR
* Replace with another relevant attribute
###### 3.1.2.3. Abilities: `item.pokemon.abilities`
* render as list of badges
* display the name of the ability
* display the description of the ability
* display the short description of the ability

###### 3.1.2.4. Moves: `item.moves`
* render as list of cards
* display the name of the move
* display the description of the move
* display the short description of the move

###### 3.1.2.5. Types: `item.pokemon.types`
* render as list of badges
* display the name of the type
* display the description of the type
* display the short description of the type
```ts
{
  name: string
  text_color: string
  background_color: string
}
```

Render using:

```tsx
<Badge
  style={{
    color: type.text_color,
    backgroundColor: type.background_color
  }}
>
  {type.name}
</Badge>
```
####### 3.1.2.5.1. weaknesses
* Same structure as types
* Also use `<Badge />` with style
####### 3.2.5.2. strengths
* Same structure as types
* Also use `<Badge />` with style

###### 3.1.2.6. Stats
Display:
* Level: `item.level`
* HP: `item.hp`
* Max HP: `item.hp`
* Experience: `item.experience`
* Attack: `item.attack`
* Defense: `item.defense`
* Speed: `item.speed`
* Special Attack: `item.special_attack`
* Special Defense: `item.special_defense`

UI suggestion:

* Progress bars
* Horizontal bars
* Clean visual indicators

###### 3.1.2.7. Evolutions (Timeline): `item.pokemon.evolutions`
Create a horizontal timeline:

* Show all evolutions
* Highlight current Pokemon
* Each item shows:

    * Image
    * Name

Example:

```
Bulbasaur → Ivysaur → Venusaur
```

#### 3.2. Api Pokedex
##### 3.2.1. List
File location: `mach-web/app/api/pokedex/route.ts`
Create a new API route that handles requests to `/api/pokedex` and returns a list of pokedex. This route should call the `PokedexService.list()` method to retrieve the data and return it in the response.
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
- Get type from searchParams.get('type')
- If type is not defined, set type to undefined
- If type is defined, set type to searchParams.get('type') and call `sanitizedParams` from `mach-web/app/utils/url.ts` to clean string 
- Get order_by from searchParams.get('order_by')
- If order_by is not defined, set order_by to undefined
- If order_by is defined, set order_by to searchParams.get('order_by') and call `sanitizedParams` from `mach-web/app/utils/url.ts` to clean string
- Get order from searchParams.get('order')
- If order is not defined, set order to undefined
- If order is defined, set order to searchParams.get('order') and call `sanitizedParams` from `mach-web/app/utils/url.ts` to clean string
- Get status from searchParams.get('status')
- If status is not defined, set status to undefined
- If status is defined, set status to searchParams.get('status') and call `sanitizedParams` from `mach-web/app/utils/url.ts` to clean string
- Call `PokedexService.list(page=page, limit=limit, name=name, order_by=order_by, order=order, status=status, type=type)` to get the list of pokemons
- Return the list of pokemons in the response with status `HTTP 200`
- If the API call is unsuccessful, return an error message in the response with status `HTTP 500` and a descriptive message.

##### 3.2.2. Detail
File location: `mach-web/app/api/pokedex/[name]/route.ts`
Create a new API route that handles requests to `/api/pokedex/{name}` and returns the details of a pokedex. This route should call the `PokedexService.detail()` method to retrieve the data and return it in the response.
Logic:
- Get session `getServerSession` from `mach-web/app/shared/lib/auth/server`
- if session.isAuthenticated is false or session.token is undefined return Response(status=401)
- Get id from params.id
- If id is not defined, return Response(status=400) with a descriptive message
- Call `PokedexService.detail(name=name)` to get the details of the pokedex
- If the pokedex is not found, return Response(status=404) with a descriptive message
- If the API call is successful, return the details of the pokedex in the response with status `HTTP 200`
- If the API call is unsuccessful, return an error message in the response with status `HTTP 500` and a descriptive message.

#### 3.3. Features Pokedex
##### 3.3.1. Service
File location: `mach-web/src/app/ui/features/pokedex/service/service.ts`
Create a service that calls the API route created in the previous step to get the list of pokedex and the details of a pokedex. This service should have two methods: `list` and `detail`.
Create based on the file called `mach-web/app/ui/features/pokemon/service/service.ts`.
- `list`: This method should call the API route for listing pokedex and return the data in a format that can be used by the UI components. It should handle any errors that may occur during the API call and return a descriptive error message if the call is unsuccessful.
- `detail`: This method should call the API route for getting the details of a pokedex and return the data in a format that can be used by the UI components. It should handle any errors that may occur during the API call and return a descriptive error message if the call is unsuccessful.

##### 3.3.2. list
###### 3.3.2.1. usePokedexList
File location: `mach-web/src/app/ui/features/pokedex/list/usePokedexList.ts`
Create a hook that uses the `useQuery` hook from `react-query` to fetch the list of pokedex from the service created in the previous step.
Create based on the file called `mach-web/app/ui/features/pokemon/list/usePokemonList.ts`.

###### 3.3.2.2. types
File location: `mach-web/src/app/ui/features/pokedex/list/types.ts`
Create a file that contains the types for the list of pokedex. This file should export the types that are used in the list of pokedex components.
Create based on the file called `mach-web/app/ui/features/pokemon/list/types.ts`.

##### 3.3.3. Detail
###### 3.3.3.1. usePokedexDetail
File location: `mach-web/src/app/ui/features/pokedex/detail/usePokedexDetail.ts`
Create a hook that uses the `useQuery` hook from `react-query` to fetch the details of a pokedex from the service created in the previous step.
Create based on the file called `mach-web/app/ui/features/pokemon/detail/usePokemonDetail.ts`.

###### 3.3.3.2. PokedexDetailPage
File location: `mach-web/src/app/ui/features/pokedex/detail/PokedexDetailPage.tsx`
Create a page that uses the `usePokedexDetail` hook created in the previous step to fetch the details of a pokedex and display them in the UI.
Create based on the file called `mach-web/app/ui/features/pokemon/detail/PokemonDetailPage.tsx`.

###### 3.3.3.3. types
File location: `mach-web/src/app/ui/features/pokedex/detail/types.ts`
Create a file that contains the types for the detail of pokedex.
Create based on the file called `mach-web/app/ui/features/pokemon/detail/types.ts`.

### 4. MyPokemon Structure
#### 4.1. Page
##### 4.1.1. List
File location: `mach-web/src/app/(protected)/my-pokemon/page.tsx`
Create the page based on the file called `mach-web/app/(protected)/pokemon/page.tsx`.
* On click in Pokedex card:
    * if `item.status` is equal to `COMPLETE` → redirect to `/my-pokemon/[name]`
    * if `item.status` is equal to `INCOMPLETE` → do nothing
* show Image in Pokedex card:
    * if `item.status` is equal to `COMPLETE` → Show Image of pokemon
    * if `item.status` is equal to `INCOMPLETE` → Show PokeBall image

##### 4.1.2. Detail
File location: `mach-web/src/app/(protected)/pokedex/[name]/page.tsx`
Create the page based on the file called `mach-web/app/(protected)/pokemon/[name]/page.tsx`.
###### 4.1.2.1. Header Section
Display:
* Pokemon Image (large): `item.pokemon.external_image || item.pokemon.image`
* Nickname: `item.nickname || item.pokemon.name || "Not Discovered"`
* Name: `item.pokemon.name` : Place the attribute in parentheses with a smaller size; only display it if the `item.name` attribute exists and is not equal to `Nickname`. If it doesn't exist, don't display it.
* Order number: `item.pokemon.order`
* Short description (if available)
###### 4.1.2.2. Info Card
Display (if available):

* Height: `item.pokemon.height`
* Weight: `item.pokemon.weight`
* Habitat: `item.pokemon.habitat`
* Hatch Counter: `item.pokemon.capture_rate`

📌 IMPORTANT:
If any of these fields do not exist in the API, feel free to:

* Omit the field OR
* Replace with another relevant attribute
###### 4.1.2.3. Abilities: `item.pokemon.abilities`
* render as list of badges
* display the name of the ability
* display the description of the ability
* display the short description of the ability

###### 4.1.2.4. Moves: `item.moves`
* render as list of cards
* display the name of the move
* display the description of the move
* display the short description of the move

###### 4.1.2.5. Types: `item.pokemon.types`
* render as list of badges
* display the name of the type
* display the description of the type
* display the short description of the type
```ts
{
  name: string
  text_color: string
  background_color: string
}
```

Render using:

```tsx
<Badge
  style={{
    color: type.text_color,
    backgroundColor: type.background_color
  }}
>
  {type.name}
</Badge>
```
####### 3.1.2.5.1. weaknesses
* Same structure as types
* Also use `<Badge />` with style
  ####### 3.2.5.2. strengths
* Same structure as types
* Also use `<Badge />` with style

###### 4.1.2.6. Stats
Display:
* Level: `item.level`
* HP: `item.hp`
* Max HP: `item.hp`
* Experience: `item.experience`
* Attack: `item.attack`
* Defense: `item.defense`
* Speed: `item.speed`
* Special Attack: `item.special_attack`
* Special Defense: `item.special_defense`

UI suggestion:

* Progress bars
* Horizontal bars
* Clean visual indicators

###### 4.1.2.7. Evolutions (Timeline): `item.pokemon.evolutions`
Create a horizontal timeline:

* Show all evolutions
* Highlight current Pokemon
* Each item shows:

    * Image
    * Name

Example:

```
Bulbasaur → Ivysaur → Venusaur
```

#### 4.2. Api Pokedex
##### 4.2.1. List
File location: `mach-web/app/api/my-pokemon/route.ts`
Create a new API route that handles requests to `/api/my-pokemon` and returns a list of my-pokemon. This route should call the `MyPokemonService.list()` method to retrieve the data and return it in the response.
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
- Get type from searchParams.get('type')
- If type is not defined, set type to undefined
- If type is defined, set type to searchParams.get('type') and call `sanitizedParams` from `mach-web/app/utils/url.ts` to clean string
- Get order_by from searchParams.get('order_by')
- If order_by is not defined, set order_by to undefined
- If order_by is defined, set order_by to searchParams.get('order_by') and call `sanitizedParams` from `mach-web/app/utils/url.ts` to clean string
- Get order from searchParams.get('order')
- If order is not defined, set order to undefined
- If order is defined, set order to searchParams.get('order') and call `sanitizedParams` from `mach-web/app/utils/url.ts` to clean string
- Get status from searchParams.get('status')
- If status is not defined, set status to undefined
- If status is defined, set status to searchParams.get('status') and call `sanitizedParams` from `mach-web/app/utils/url.ts` to clean string
- Call `MyPokemonService.list(page=page, limit=limit, name=name, order_by=order_by, order=order, status=status, type=type)` to get the list of my-pokemon
- Return the list of my-pokemon in the response with status `HTTP 200`
- If the API call is unsuccessful, return an error message in the response with status `HTTP 500` and a descriptive message.

##### 4.2.2. Detail
File location: `mach-web/app/api/my-pokemon/[name]/route.ts`
Create a new API route that handles requests to `/api/my-pokemon/{name}` and returns the details of a my-pokemon. This route should call the `MyPokemonService.detail()` method to retrieve the data and return it in the response.
Logic:
- Get session `getServerSession` from `mach-web/app/shared/lib/auth/server`
- if session.isAuthenticated is false or session.token is undefined return Response(status=401)
- Get id from params.id
- If id is not defined, return Response(status=400) with a descriptive message
- Call `MyPokemonService.detail(name=name)` to get the details of the my-pokemon
- If the my-pokemon is not found, return Response(status=404) with a descriptive message
- If the API call is successful, return the details of the my-pokemon in the response with status `HTTP 200`
- If the API call is unsuccessful, return an error message in the response with status `HTTP 500` and a descriptive message.

#### 4.3. Features MyPokemon
##### 4.3.1. Service
File location: `mach-web/src/app/ui/features/my-pokemon/service/service.ts`
Create a service that calls the API route created in the previous step to get the list of my-pokemon and the details of a my-pokemon. This service should have two methods: `list` and `detail`.
Create based on the file called `mach-web/app/ui/features/pokemon/service/service.ts`.
- `list`: This method should call the API route for listing my-pokemon and return the data in a format that can be used by the UI components. It should handle any errors that may occur during the API call and return a descriptive error message if the call is unsuccessful.
- `detail`: This method should call the API route for getting the details of a my-pokemon and return the data in a format that can be used by the UI components. It should handle any errors that may occur during the API call and return a descriptive error message if the call is unsuccessful.

##### 4.3.2. list
###### 4.3.2.1. useMyPokemonList
File location: `mach-web/src/app/ui/features/my-pokemon/list/useMyPokemonList.ts`
Create a hook that uses the `useQuery` hook from `react-query` to fetch the list of my-pokemon from the service created in the previous step.
Create based on the file called `mach-web/app/ui/features/pokemon/list/usePokemonList.ts`.

###### 4.3.2.2. types
File location: `mach-web/src/app/ui/features/my-pokemon/list/types.ts`
Create a file that contains the types for the list of my-pokemon. This file should export the types that are used in the list of my-pokemon components.
Create based on the file called `mach-web/app/ui/features/pokemon/list/types.ts`.

##### 4.3.3. Detail
###### 4.3.3.1. useMyPokemonDetail
File location: `mach-web/src/app/ui/features/my-pokemon/detail/usePokedexDetail.ts`
Create a hook that uses the `useQuery` hook from `react-query` to fetch the details of a my-pokemon from the service created in the previous step.
Create based on the file called `mach-web/app/ui/features/pokemon/detail/usePokemonDetail.ts`.

###### 4.3.3.2. MyPokemonDetailPage
File location: `mach-web/src/app/ui/features/my-pokemon/detail/MyPokemonDetailPage.tsx`
Create a page that uses the `useMyPokemonDetail` hook created in the previous step to fetch the details of a my-pokemon and display them in the UI.
Create based on the file called `mach-web/app/ui/features/pokemon/detail/PokemonDetailPage.tsx`.

###### 4.3.3.3. types
File location: `mach-web/src/app/ui/features/my-pokemon/detail/types.ts`
Create a file that contains the types for the detail of my-pokemon.
Create based on the file called `mach-web/app/ui/features/pokemon/detail/types.ts`.

---

### 5. Navigation Rules
1. If the user has attribute `status` different than `COMPLETE`, the user should not be able to see the `/my-pokemon` page, redirect to Home and inside sidebar.
2. If the user has attribute `status` different than `COMPLETE`, the user should not be able to see the `/pokedex` page, redirect to Home and inside sidebar.
3. If the user has attribute `status` different than `COMPLETE`, the user should not be able to see the `/my-pokemon/[name]` page, redirect to Home.
4. If the user has attribute `status` different than `COMPLETE`, the user should not be able to see the `/pokedex/[name]` page, redirect to Home.

### 6. Home
1. If the user has attribute `status` different than `COMPLETE` and click in `INITIALIZE` Button, add in `{/* Initialize Trainer Modal */}` one autocomplete with a list of pokemons from `/api/pokemon` filtered with `Capture Rate`. this input can be enabled only `Capture Rate` is different from `undefined`,

---

## Notes

* Follow existing patterns strictly
* Avoid creating new abstractions unnecessarily
* Keep UI clean and modern
* Prefer reuse to reinvention

---

## Coverage Rules
- Minimum:100%
- Focus on real behavior
- Avoid trivial tests
- Mock: API, REDIS

---
## General Rules
- Do not modify files unrelated to this spec.
- All async functions must use `async/await`.
- All database queries must use SQLAlchemy `AsyncSession`.
- No raw SQL — use SQLAlchemy ORM only.
- The Redis client must be injected as a FastAPI dependency.
- Error handling must return meaningful HTTP status codes and messages.
- Use Tailwind and custom components from `/ds`.
- Inject dependencies (Redis)
- Handle errors properly
- Do not touch auth modules
- Do not modify the `/ds` components
- Toda alteração de base de dados deve atualizar o redis

---
## Testing Rules

- Use pytest for API testing
- Use Jest and React Testing Library for web testing
- Cover all critical cases in both API and web testing
- Mock external dependencies (API, REDIS) in tests
- Ensure coverage meets minimum requirement
- Focus on real behavior and avoid trivial tests

---


## Proposal Decision
• ## O que foi decidido

  ### 1. Modelo de domínio

  Pokedex

  - É por espécie e por treinador
  - Chave lógica: (trainer_id, pokemon_id)
  - Guarda progressão própria desde a inicialização
  - Guarda dados de batalha/progressão:
      - level
      - IV/EV
      - wins/losses
      - demais stats definidos no modelo
  - Não terá nickname
  - Não terá moves próprios persistidos
  - Pode exibir no detail os moves possíveis da espécie vindos de pokemon.moves
  - Só o campo discovered controla visibilidade funcional do detail

  MyPokemon

  - É instância capturada
  - Permite múltiplos registros do mesmo (trainer_id, pokemon_id)
  - Guarda progressão própria, totalmente apartada da Pokedex
  - Guarda:
      - nickname
      - level
      - IV/EV
      - wins/losses
      - stats
      - moves equipados
  - Não terá campo status
  - O gating do detail usa my_pokemon.pokemon.status == COMPLETE

  ———
### 2. Relação entre Pokedex e MyPokemon

  Quando um Pokémon é capturado:

  - cria MyPokemon
  - se a espécie ainda não estiver descoberta, Pokedex.discovered = true
  - se já estiver descoberta, não precisa alterar a entrada da Pokedex

  A descoberta futura:

  - terá um serviço próprio depois
  - por enquanto, só acontece na inicialização do treinador para o starter

  ———
### 3. Inicialização do treinador

  Na inicialização:

  - deve existir uma lista de Pokedex com todos os Pokémon da base
  - todos os itens da Pokedex já nascem com progressão inicial
  - todos começam com discovered = false
  - exceto o starter, que nasce com discovered = true
  - deve criar apenas um MyPokemon inicial para o treinador
  - tanto Pokedex quanto MyPokemon começam com progressão desde o início

  A base de inicialização da Pokedex:

  - usa stats base do catálogo para cada espécie
  - não usa a mesma lógica de randomização/instância do MyPokemon

  Quando uma espécie for descoberta depois:

  - apenas altera discovered = true
  - define discovered_at
  - preserva a progressão já existente

  ———
### 4. Estratégia assíncrona da Pokedex

  Foi assumida a estratégia com BackgroundTasks.

  Fluxo:

  - POST /trainers/initialize
      - cria trainer se necessário
      - cria MyPokemon inicial se necessário
      - dispara a carga da Pokedex em background
      - responde sem bloquear o processo inteiro

  Trainer terá um novo campo:

  - pokedex_status

  Valores:

  - EMPTY
  - INITIALIZING
  - READY
  - FAILED

  Regras:

  - trainer novo começa com pokedex_status = EMPTY
  - quando a carga inicia, vira INITIALIZING
  - quando termina com sucesso, vira READY
  - se falhar, vira FAILED

  Retry:

  - continua no mesmo /trainers/initialize
  - se o trainer já existir:
      - só tenta reinicializar a Pokedex se pokedex_status for EMPTY ou FAILED
      - não recria o MyPokemon

  Idempotência:

  - se a entrada (trainer_id, pokemon_id) já existir na Pokedex, mantém
  - se não existir, cria

  Falha:

  - a carga da Pokedex deve rodar em transação única
  - em erro, faz rollback total da tentativa
  - os parciais não devem ficar persistidos
  - não usar soft delete para esse rollback técnico

  ———
### 5. Regras de UI para Pokedex

  Lista /pokedex

  - mostra todos os registros do treinador desde o início
  - se discovered = false:
      - nome acinzentado
      - imagem padrão de pokebola
      - badge Not Discovered
      - sem acesso ao detail
  - pokemon.status não participa da regra visual da Pokedex
  - se discovered = true:
      - mostra nome real
      - mostra imagem real
      - mostra tipos principais em <Badge />
      - sem weakness/strength no card
      - pode mostrar também order, level, wins/losses, discovered_at

  Detail /pokedex/[id]

  - abre por id
  - só abre quando discovered = true
  - mostra:
      - imagem
      - nome
      - tipos
      - progressão completa
      - discovered_at
      - moves possíveis da espécie
      - evolução/timeline

  ———

### 6. Regras de UI para MyPokemon

  Lista /my-pokemon

  - detalhe abre por id
  - campos essenciais do card:
      - nickname
      - pokemon.name
      - level
      - wins/losses
      - badges dos tipos
      - imagem real somente se pokemon.status == COMPLETE

  Detail /my-pokemon/[id]

  - título principal = nickname
  - subtítulo = nome da espécie
  - mostra:
      - imagem
      - tipos
      - progressão completa
      - moves equipados
      - evolução/timeline

  Edição:

  - permitir alteração somente de nickname
  - via modal
  - validação:
      - não pode ser vazio
      - mínimo de 3 caracteres

  Moves:

  - selecionados aleatoriamente no momento da captura
  - persistidos como parte fixa da instância
  - timeline de evolução é apenas informativa por enquanto

  ———
### 7. Regras de rotas/details

  Pokedex

  - detail por id
  - só depende de discovered = true

  MyPokemon

  - detail por id
  - depende de my_pokemon.pokemon.status == COMPLETE

  - todos os details devem mostrar evolução/timeline

  ———

  ### 8. UX de estados da Pokedex

  Se pokedex_status = INITIALIZING:

  - o frontend pode abrir /pokedex
  - deve mostrar estado visual de preparação/carregamento

  Se pokedex_status = FAILED:

  - /pokedex mostra estado de erro
  - com botão para chamar /trainers/initialize novamente

  ———

## Leitura geral

  O desenho final ficou assim:

  Pokemon
    catálogo global compartilhado

  Trainer
    possui estado da montagem da pokedex

  Pokedex
    progresso por espécie para o treinador
    sem nickname
    sem moves equipados

  MyPokemon
    criatura capturada, instanciada
    com nickname
    com moves equipados
    com progressão própria
