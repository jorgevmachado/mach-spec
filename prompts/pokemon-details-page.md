# OpenSpec Prompt — Pokemon Details Page

## Context

This project uses a monorepo with two submodules:

* **mach-api** — Python 3.13, FastAPI, SQLAlchemy async, DDD architecture
* **mach-web** — Next.js 16, React 19, TypeScript, Tailwind CSS

There is already:

* A **paginated list page** of Pokemons in `mach-web`

* Each card shows:

    * Image
    * Order number
    * Status

* An existing API endpoint:

```
GET /pokemon/{name}
```

* A hook already implemented for list:

```
mach-web/app/ui/features/pokemon/list/usePokemonList.ts
```

---

## Objective

Create a **Pokemon Details Page** in `mach-web` that is accessed when clicking a Pokemon card **ONLY if its status is `COMPLETE`**.

---

## Navigation Behavior

* On click in Pokemon card:

    * If `status !== COMPLETE` → do nothing
    * If `status === COMPLETE` → navigate to:

```
/pokemon/[name]
```

---

# SPEC — mach-web (Frontend)

## 1. Page Creation

Create a new route:

```
app/pokemon/[name]/page.tsx
```

---

## 2. Data Fetching Pattern (IMPORTANT)

Follow the SAME pattern used in:

```
usePokemonList.ts
```

Create a new hook:

```
usePokemonDetail.ts
```

Location suggestion:

```
mach-web/app/ui/features/pokemon/detail/usePokemonDetail.ts
```

### Requirements:

* Reuse BaseService pattern
* Follow same structure (state, loading, error)
* Do NOT invent a new pattern
* Keep consistency with existing codebase

---

## 3. Layout Requirements

The page must be:

* Fully responsive
* Visually rich
* Inspired by:

    * Pokedex UI
    * Card-based layout
    * Clean modern dashboards

---

## 4. Sections of the Page

---

### 4.1 Header Section

Display:

* Pokemon image (large)
* Name
* Order number
* Short description (if available)

---

### 4.2 Info Card

Display (if available):

* Altura (height)
* Peso (weight)
* Categoria (category)
* Hatch Counter

📌 IMPORTANT:
If any of these fields do not exist in the API, feel free to:

* Omit the field OR
* Replace with another relevant attribute

---

### 4.3 Habilidades

* Render as list or badges

---

### 4.4 Moves

* Render as list or badges

---

### 4.5 Tipo (IMPORTANT)

Types come as objects:

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

---

### 4.5 Fraquezas

* Same structure as types
* Also use `<Badge />` with style

---

### 4.6 Habitats

* Render as tags or list
* Optional section if exists

---

### 4.7 Estatísticas

Display:

* HP
* Attack
* Defense
* Speed
* Special Attack
* Special Defense

UI suggestion:

* Progress bars
* Horizontal bars
* Clean visual indicators

---

### 4.8 Evoluções (Timeline)

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

---

## 5. UI/UX Guidelines

* Use TailwindCSS
* Use Design System components (`app/ds/`)
* Prefer composition
* Rounded cards
* Shadows
* Good spacing

---

# SPEC — mach-api (Backend Adjustments)

## 1. Response Contract (IMPORTANT)

Below is ONLY an example response:

⚠️ DO NOT assume exact field names
⚠️ Use the SAME naming that already exists in the API
⚠️ Adapt only if necessary

```json
{
  "id": "d1265838-3fc3-4dee-9ad4-c6406029b85a",
  "name": "bulbasaur",
  "order": 1,
  "status": "COMPLETE",
  "external_image": "https://www.pokemon.com/static-assets/content-assets/cms2/img/pokedex/detail/001.png",
  "hp": 45,
  "image": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/1.png",
  "speed": 45,
  "height": 7,
  "weight": 69,
  "attack": 49,
  "defense": 49,
  "habitat": "grassland",
  "is_baby": false,
  "shape_url": "https://pokeapi.co/api/v2/pokemon-shape/8/",
  "shape_name": "quadruped",
  "is_mythical": false,
  "gender_rate": 1,
  "is_legendary": false,
  "capture_rate": 45,
  "hatch_counter": 20,
  "base_happiness": 70,
  "special_attack": 65,
  "base_experience": 64,
  "special_defense": 65,
  "evolution_chain": "https://pokeapi.co/api/v2/evolution-chain/1/",
  "evolves_from_species": null,
  "has_gender_differences": false,
  "growth_rate": {
    "id": "d1fadbac-e986-4f4e-b693-d539735003c2",
    "name": "medium-slow"
  },
  "moves": [
    {
      "id": "364b0803-566a-44f7-ba77-7128c76d3e84",
      "name": "razor-wind"
    },
    {
      "id": "81aa65d0-f3dc-4b64-a43d-ac827c1c153a",
      "name": "swords-dance"
    },
    {
      "id": "0b1d5488-d2d7-476b-a5a2-692a40e67174",
      "name": "cut"
    },
    {
      "id": "46365c2a-4b13-4092-92f6-5da832445a7c",
      "name": "bind"
    },
    {
      "id": "b2165e95-300c-4c7b-9b44-3d13414bf81b",
      "name": "vine-whip"
    },
    {
      "id": "c3fda680-d8a6-48b3-b9ac-c2c06174837d",
      "name": "headbutt"
    },
    {
      "id": "854e1f53-be66-41ef-a05e-a34762f6a201",
      "name": "tackle"
    },
    {
      "id": "1bc0b4c2-6d7b-4b1e-8240-b4693e4c5ae6",
      "name": "body-slam"
    },
    {
      "id": "a9de48ca-0e75-4e65-bd36-a3b1c8cfbca5",
      "name": "take-down"
    },
    {
      "id": "141020d2-a06c-44f8-9084-604a8fc635c3",
      "name": "double-edge"
    },
    {
      "id": "4c9466c6-13ba-49bc-b13a-c3cd35278144",
      "name": "growl"
    },
    {
      "id": "1cb0b954-0735-477b-afb9-dbd1b30c5a00",
      "name": "strength"
    },
    {
      "id": "fc7bf161-d2e3-4f24-93a0-0471fbe0485c",
      "name": "mega-drain"
    },
    {
      "id": "7a398074-dffd-4c84-b731-72934ff767b6",
      "name": "leech-seed"
    },
    {
      "id": "7cdb3260-10d6-4bcf-bca6-c383911cf5b3",
      "name": "growth"
    },
    {
      "id": "965957ba-7070-447f-bf12-358af4117d65",
      "name": "razor-leaf"
    },
    {
      "id": "b86d0e2f-775a-4b4b-9dff-4bb693b41749",
      "name": "solar-beam"
    },
    {
      "id": "3a73268d-e47f-4cf9-b35a-084845bd5bec",
      "name": "poison-powder"
    },
    {
      "id": "014abf47-e9d4-4c18-9fda-d40b1af96c14",
      "name": "sleep-powder"
    },
    {
      "id": "09db78ca-63cd-46ed-a45d-10eb3bba9425",
      "name": "petal-dance"
    }
  ],
  "abilities": [
    {
      "id": "753b002c-6320-477d-97c0-f8b19455cb0c",
      "name": "overgrow"
    },
    {
      "id": "343f2291-5303-401c-8b50-b7286ec2bfa6",
      "name": "chlorophyll"
    }
  ],
  "types": [
    {
      "id": "d77ab6be-8bda-43e6-91a9-1db7bbc67430",
      "name": "grass"
    },
    {
      "id": "ce22c47a-8062-4ff6-bdb3-2c9cdbbc2d56",
      "name": "poison"
    }
  ],
  "evolutions": [
    {
      "id": "d70a6232-0782-4ce8-b2b9-72b9506ecd6d",
      "name": "ivysaur"
    },
    {
      "id": "7b1337ad-784b-4007-944a-65230a8ca537",
      "name": "venusaur"
    }
  ],
  "created_at": "2026-04-23T22:38:08.560237Z",
  "updated_at": "2026-04-24T14:37:39.952728Z",
  "deleted_at": null
}
```

---

## 2. Flexibility Rule

If any field:

* Does NOT exist → ignore or replace
* Exists with different naming → KEEP original naming

---

## 3. Architecture Rules

Follow strictly:

* route → service → repository → models
* No business logic in route
* Use schemas (Pydantic)
* Maintain consistency with domain structure

---

## Definition of Done

### Frontend

* [ ] Page created
* [ ] Navigation works only for COMPLETE
* [ ] Hook `usePokemonDetail` implemented
* [ ] UI responsive
* [ ] Types rendered with styled Badge
* [ ] Evolution timeline implemented
* [ ] Unit tests
* [ ] 100% code coverage

### Backend

* [ ] Endpoint returns sufficient data
* [ ] No breaking changes
* [ ] Naming preserved
* [ ] Schema updated if needed
* [ ] Unit tests
* [ ] 100% code coverage
---

## Notes

* Follow existing patterns strictly
* Avoid creating new abstractions unnecessarily
* Keep UI clean and modern
* Prefer reuse over reinvention

---

---

## Coverage Rules
- Minimum:100%
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
- Toda alteração de base de dados deve atualizar o redis

## Testing Rules

- Use pytest for API testing
- Use Jest and React Testing Library for web testing
- Cover all critical cases in both API and web testing
- Mock external dependencies (API, REDIS) in tests
- Ensure coverage meets minimum requirement
- Focus on real behavior and avoid trivial tests
