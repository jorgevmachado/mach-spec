# OpenSpec Prompt — Auth & Trainer Feature

## Context

This project uses a monorepo with two submodules:

- **mach-api** — Python 3.13, FastAPI, SQLAlchemy async, domain-driven architecture
- **mach-web** — Next.js 16, React 19, TypeScript, Tailwind CSS v4

---

## SPEC: mach-api

### 1. Database Models

#### 1.1 Model: `User`

File location: `mach-api/app/models/user.py`

Create a SQLAlchemy async model named `User` mapped to the table `users` with the following columns:

| Column | Type | Constraints |
|---|---|---|
| `id` | UUID | Primary key, default `uuid4`, not nullable |
| `name` | String | Not nullable |
| `email` | String | Unique, not nullable |
| `username` | String | Unique, not nullable |
| `gender` | Enum(`GenderEnum`) | Not nullable |
| `status` | Enum(`StatusEnum`) | Not nullable, default `INCOMPLETE` |
| `password` | String | Not nullable — must always be stored hashed |
| `date_of_birth` | DateTime | Not nullable |
| `total_authentications` | Integer | Nullable, default `0` |
| `authentication_success` | Integer | Nullable, default `0` |
| `authentication_failures` | Integer | Nullable, default `0` |
| `last_authentication_at` | DateTime | Nullable, default `None` |
| `created_at` | DateTime | Not nullable, default `utcnow`, **never updated after insert** |
| `updated_at` | DateTime | Nullable, default `None`, **only set on second or later updates** |
| `deleted_at` | DateTime | Nullable, default `None`, **only set when soft-delete is triggered** |

Create two Python `Enum` classes in the same file or in `mach-api/app/models/enums.py`:

```python
class GenderEnum(str, Enum):
    MALE = "MALE"
    FEMALE = "FEMALE"
    OTHER = "OTHER"

class StatusEnum(str, Enum):
    ACTIVE = "ACTIVE"
    COMPLETE = "COMPLETE"
    INACTIVE = "INACTIVE"
    INCOMPLETE = "INCOMPLETE"
```

**Behavior rules for `User`:**
- `created_at` is set only at creation and must never be changed afterwards.
- `updated_at` starts as `None`. It must be set to the current UTC datetime only on the **second or later modification** of the record.
- `deleted_at` must only be set when a soft-delete operation is explicitly triggered. Soft-deleting a user must **not** physically remove the row.
- `password` must **never** be stored in plain text. Always use `from app.core.security import get_password_hash` before persisting.

---

#### 1.2 Model: `Trainer`

File location: `mach-api/app/models/trainer.py`

Create a SQLAlchemy async model named `Trainer` mapped to the table `trainers` with the following columns:

| Column | Type | Constraints |
|---|---|---|
| `id` | UUID | Primary key, default `uuid4`, not nullable |
| `user_id` | UUID | Foreign key → `users.id`, not nullable, unique (one trainer per user) |
| `capture_rate` | Integer | Not nullable |
| `pokeballs` | Integer | Not nullable |
| `created_at` | DateTime | Not nullable, default `utcnow`, **never updated after insert** |
| `updated_at` | DateTime | Nullable, default `None`, **only set on second or later updates** |
| `deleted_at` | DateTime | Nullable, default `None`, **only set when soft-delete is triggered** |

Define a relationship `user` on `Trainer` pointing to `User`.

**Behavior rules for `Trainer`:**
- Same `created_at`, `updated_at`, `deleted_at` rules as `User`.
- One user can only have one trainer record (enforce via unique constraint on `user_id`).

---

### 2. Alembic Migration

Generate an Alembic migration file that creates both `users` and `trainers` tables in order (users first, then trainers). Include the enum types creation for `GenderEnum` and `StatusEnum`.

---

### 3. Schemas (Pydantic)

File location: `mach-api/app/schemas/auth.py` and `mach-api/app/schemas/trainer.py`

#### 3.1 `RegisterRequest`
Fields:
- `name: str`
- `email: EmailStr`
- `username: str`
- `gender: GenderEnum`
- `date_of_birth: datetime`
- `password: str` — minimum 8 characters, validated with a Pydantic validator

#### 3.2 `RegisterResponse`
Fields:
- `id: UUID`
- `name: str`
- `email: str`
- `username: str`
- `status: StatusEnum`
- `created_at: datetime`

#### 3.3 `LoginRequest`
Fields:
- `credential: str` — accepts either `email` or `username`
- `password: str`

#### 3.4 `LoginResponse`
Fields:
- `access_token: str`
- `token_type: str` — always `"bearer"`

#### 3.5 `InitializeTrainerRequest`
Fields:
- `pokeballs: int`
- `capture_rate: int`

#### 3.6 `InitializeTrainerResponse`
Fields:
- `id: UUID`
- `user_id: UUID`
- `pokeballs: int`
- `capture_rate: int`
- `created_at: datetime`

---

### 4. Repositories

File location: `mach-api/app/repositories/user_repository.py` and `mach-api/app/repositories/trainer_repository.py`

#### 4.1 `UserRepository`
Async methods:
- `get_by_email(email: str) -> User | None`
- `get_by_username(username: str) -> User | None`
- `get_by_email_or_username(credential: str) -> User | None` — queries by email OR username
- `create(data: dict) -> User`
- `update_auth_success(user_id: UUID) -> None` — increments `total_authentications` and `authentication_success`, sets `last_authentication_at = utcnow()`; applies `updated_at` rule
- `update_auth_failure(user_id: UUID) -> None` — increments `total_authentications` and `authentication_failures`; applies `updated_at` rule
- `update_status(user_id: UUID, status: StatusEnum) -> None` — updates `status`; applies `updated_at` rule
- `soft_delete(user_id: UUID) -> None` — sets `deleted_at = utcnow()`

#### 4.2 `TrainerRepository`
Async methods:
- `get_by_user_id(user_id: UUID) -> Trainer | None`
- `create(data: dict) -> Trainer`

---

### 5. Services

File location: `mach-api/app/services/auth_service.py` and `mach-api/app/services/trainer_service.py`

#### 5.1 `AuthService`

**`register(data: RegisterRequest) -> User`**
1. Check if `email` already exists via `UserRepository.get_by_email`. If yes, raise `HTTP 409 Conflict` with message `"Email already registered"`.
2. Check if `username` already exists via `UserRepository.get_by_username`. If yes, raise `HTTP 409 Conflict` with message `"Username already taken"`.
3. Hash the password using `from app.core.security import get_password_hash`.
4. Set `status = StatusEnum.INCOMPLETE`.
5. Persist via `UserRepository.create` and return the created user.

**`login(data: LoginRequest) -> dict`**
1. Find user by `credential` using `UserRepository.get_by_email_or_username`.
2. If no user found, raise `HTTP 401 Unauthorized` with message `"Invalid credentials"`.
3. Verify password using `from app.core.security import verify_password`. If mismatch:
   - Call `UserRepository.update_auth_failure(user.id)`.
   - Raise `HTTP 401 Unauthorized` with message `"Invalid credentials"`.
4. On success:
   - Call `UserRepository.update_auth_success(user.id)`.
   - Generate JWT token using `from app.core.security import create_access_token` passing `{"sub": str(user.id)}`.
   - Return `{"access_token": token, "token_type": "bearer"}`.

#### 5.2 `TrainerService`

**`initialize(user_id: UUID, data: InitializeTrainerRequest) -> Trainer`**
1. Check if a trainer already exists for this `user_id` via `TrainerRepository.get_by_user_id`. If yes, raise `HTTP 409 Conflict` with message `"Trainer already initialized"`.
2. Create trainer record via `TrainerRepository.create`.
3. Update user status to `StatusEnum.COMPLETE` via `UserRepository.update_status`.
4. Return the created trainer.

---

### 6. Routers

#### 6.1 `/auth/register` — POST

File: `mach-api/app/routers/auth.py`

- **Method:** POST  
- **Path:** `/auth/register`  
- **Request body:** `RegisterRequest`  
- **Response:** `RegisterResponse` with status `HTTP 201`  
- **Auth required:** No  
- Delegates to `AuthService.register`

#### 6.2 `/auth/login` — POST

- **Method:** POST  
- **Path:** `/auth/login`  
- **Request body:** `LoginRequest`  
- **Response:** `LoginResponse` with status `HTTP 200`  
- **Auth required:** No  
- Delegates to `AuthService.login`

#### 6.3 `/trainers/initialize` — POST

File: `mach-api/app/routers/trainers.py`

- **Method:** POST  
- **Path:** `/trainers/initialize`  
- **Request body:** `InitializeTrainerRequest`  
- **Response:** `InitializeTrainerResponse` with status `HTTP 201`  
- **Auth required:** Yes — extract `user_id` from the JWT token via a dependency `get_current_user`  
- Delegates to `TrainerService.initialize(user_id, data)`

The `get_current_user` dependency must:
1. Read the `Authorization: Bearer <token>` header.
2. Decode the JWT using the existing `app.core.security` utilities.
3. Return the `user_id` (UUID) from the token payload `sub` field.
4. Raise `HTTP 401 Unauthorized` if the token is missing, invalid, or expired.

---

### 7. Router Registration

Register both routers in `mach-api/app/main.py` (or wherever the FastAPI app is instantiated):

```python
app.include_router(auth_router, prefix="/auth", tags=["Auth"])
app.include_router(trainers_router, prefix="/trainers", tags=["Trainers"])
```

---

---

## SPEC: mach-web

### 8. API Client

File location: `mach-web/src/services/api.ts`

Create a typed API client using `fetch` (or `axios` if already present) with the following functions:

```ts
registerUser(data: RegisterPayload): Promise<RegisterResponse>
loginUser(data: LoginPayload): Promise<LoginResponse>
initializeTrainer(data: InitializeTrainerPayload, token: string): Promise<InitializeTrainerResponse>
```

Base URL must read from `process.env.NEXT_PUBLIC_API_URL`.

All responses must be typed. Define the following TypeScript types in `mach-web/src/types/auth.ts`:

```ts
type GenderEnum = "MALE" | "FEMALE" | "OTHER"
type StatusEnum = "ACTIVE" | "COMPLETE" | "INACTIVE" | "INCOMPLETE"

interface RegisterPayload {
  name: string
  email: string
  username: string
  gender: GenderEnum
  date_of_birth: string // ISO 8601
  password: string
}

interface RegisterResponse {
  id: string
  name: string
  email: string
  username: string
  status: StatusEnum
  created_at: string
}

interface LoginPayload {
  credential: string
  password: string
}

interface LoginResponse {
  access_token: string
  token_type: string
}

interface InitializeTrainerPayload {
  pokeballs: number
  capture_rate: number
}

interface InitializeTrainerResponse {
  id: string
  user_id: string
  pokeballs: number
  capture_rate: number
  created_at: string
}

interface AuthUser {
  id: string
  name: string
  email: string
  username: string
  status: StatusEnum
}
```

---

### 9. Auth State Management

File: `mach-web/src/context/AuthContext.tsx`

Create a React Context with:
- `user: AuthUser | null`
- `token: string | null`
- `login(token: string, user: AuthUser): void` — stores token in `localStorage` as `"mach_token"` and user as `"mach_user"`
- `logout(): void` — clears localStorage and resets state
- `isAuthenticated: boolean`

The context provider must rehydrate from `localStorage` on mount.

---

### 10. Route Guard

File: `mach-web/src/components/ProtectedRoute.tsx`

A component that:
- Reads `isAuthenticated` from `AuthContext`
- If not authenticated, redirects to `/login` using Next.js `router.replace`
- If authenticated, renders `children`

---

### 11. Pages

#### 11.1 Register Page — `/register`

File: `mach-web/src/app/register/page.tsx`

Form fields:
- `name` (text input)
- `email` (email input)
- `username` (text input)
- `gender` (select: MALE, FEMALE, OTHER)
- `date_of_birth` (date input)
- `password` (password input)
- `confirmPassword` (password input — validated client-side, must match `password`)

Behavior:
1. On submit, validate all fields are filled and passwords match.
2. Call `registerUser(payload)` from the API client.
3. On **success (201)**: redirect to `/login` and display a success toast/message `"Account created! Please log in."`.
4. On **error**: display the error message returned by the API (e.g. `"Email already registered"`).

Style: use Tailwind CSS v4. Design a clean, centered card layout. Include a link to `/login` at the bottom (`"Already have an account? Sign in"`).

---

#### 11.2 Login Page — `/login`

File: `mach-web/src/app/login/page.tsx`

Form fields:
- `credential` (text input — label: `"Email or Username"`)
- `password` (password input)

Behavior:
1. Call `loginUser(payload)` from the API client.
2. On **success**: decode the JWT or store it, call `AuthContext.login(token, user)`.
   - The user data must be extracted from the JWT payload (decode the base64 middle part) or fetched from a `/users/me` endpoint if it exists. If neither is available, persist only the token and derive `AuthUser` from the register response stored in session/localStorage temporarily.
3. Redirect to `/home` after successful login.
4. On **error**: show inline error `"Invalid credentials"`.

Style: same card layout as register. Include a link to `/register` (`"Don't have an account? Sign up"`).

---

#### 11.3 Home Page — `/home`

File: `mach-web/src/app/home/page.tsx`

Wrap with `ProtectedRoute`.

Display:
- User's `name`, `email`, `username`, `status`
- If `status === "INCOMPLETE"`: show a prominent button labeled **"INITIALIZE"**

**INITIALIZE button behavior:**
1. Open a modal or inline form asking for:
   - `pokeballs` (number input, required)
   - `capture_rate` (number input, required)
2. On confirm, call `initializeTrainer(payload, token)` with the token from `AuthContext`.
3. On **success**: update the user's `status` in `AuthContext` to `"COMPLETE"` and hide the button.
4. On **error**: show the error message.

Style: Tailwind CSS v4. Card-based layout with user avatar placeholder. Status badge with color coding:
- `INCOMPLETE` → yellow/amber
- `COMPLETE` → green
- `ACTIVE` → blue
- `INACTIVE` → gray

---

### 12. Navigation / Routing

Ensure the following routes exist in the Next.js App Router:

| Path | Component | Auth required |
|---|---|---|
| `/register` | Register page | No |
| `/login` | Login page | No |
| `/home` | Home page | **Yes** |
| `/` | Redirect to `/login` | No |

---

### 13. Environment Variables

Add to `mach-web/.env.local.example`:

```
NEXT_PUBLIC_API_URL=http://localhost:8000
```

---

## General Rules

- Follow existing code conventions and folder structure in each submodule.
- Do not modify files unrelated to this spec.
- All async functions must use `async/await`.
- All database queries must be async (SQLAlchemy `AsyncSession`).
- No raw SQL — use SQLAlchemy ORM only.
- Error handling must return meaningful HTTP status codes and messages.
- The JWT secret and algorithm must be read from environment variables via the existing `app.core.config` or `app.core.security` setup — do not hardcode secrets.
- TypeScript strict mode must be respected in `mach-web`.