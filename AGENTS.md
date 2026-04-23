# mach-specs

## Project Snapshot
Umbrella repository containing two git submodules:
- **mach-api** – Python 3.13, FastAPI, SQLAlchemy async, domain-driven architecture
- **mach-web** – Next.js 16, React 19, TypeScript, Tailwind CSS v4

Each submodule has its own git history, dependencies, and AGENTS.md. Always work inside
the submodule directory; do not mix API and web concerns.

## Root Setup Commands
```bash
# Clone with submodules
git clone --recurse-submodules https://github.com/jorgevmachado/mach-specs.git

# Update submodules after pull
git submodule update --init --recursive

# Open API submodule
cd mach-api && poetry install

# Open web submodule
cd mach-web && npm install
```

## Universal Conventions
- **Commits**: Conventional Commits format (`feat:`, `fix:`, `chore:`, `refactor:`, `test:`)
- **Secrets**: Never commit `.env` files, tokens, or credentials — use environment variables
- **Branches**: Feature branches off `main`; open PRs with clear description and passing CI
- **Submodule changes**: Commit inside the submodule first, then update the pointer in root

## Security & Secrets
- API secrets via `pydantic-settings` (reads `.env`); never hardcode `SECRET_KEY`, DB URLs, or Redis URLs
- Web secrets via `process.env.*`; client-side vars must use `NEXT_PUBLIC_` prefix
- Auth cookies are `httpOnly`, `secure` in production — see `mach-web/app/shared/lib/auth/session/session.ts`

## JIT Index
### Submodules
- API backend: `mach-api/` → [see mach-api/AGENTS.md](mach-api/AGENTS.md)
- Web frontend: `mach-web/` → [see mach-web/AGENTS.md](mach-web/AGENTS.md)

### Quick Find Commands
```bash
# Find a Python function/class
rg -n "def function_name\|class ClassName" mach-api/app/

# Find a React component export
rg -n "export (default )?function\|export const" mach-web/app/ds/

# Find API routes
rg -n "@app\.(get|post|put|delete|patch)" mach-api/app/

# Find auth-related code
rg -n "auth\|jwt\|token" mach-api/app/core/security/ mach-web/app/shared/lib/auth/
```

## Definition of Done
- API: `make test` passes (lint + pytest with coverage) and Alembic migration created if schema changed
- Web: `npm run lint` and `npm run build` pass, `.spec.tsx` tests updated for changed components
- No secrets committed; no `any` in TypeScript; no business logic in routes/components
