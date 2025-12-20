# Ticket: BE-001 - Initial Project Setup and Folder Structure

**Epic**: Foundation & Infrastructure
**Type**: Task
**Priority**: Highest

## 📝 Story
As a backend developer, I need a robust project scaffold that follows our defined Clean Architecture so that I can start implementing features in an organized manner.

## ✅ Acceptance Criteria
- [ ] Project root created with `src/app`, `tests`, `scripts`, `config`.
- [ ] Source directory structure matches "Advanced FastAPI Design" (Domain, Application, Infrastructure, Presentation).
- [ ] Dependency management setup with `pyproject.toml` (using uv or poetry).
- [ ] Pre-commit hooks configured (formatting, linting).
- [ ] Docker compose file for local development (Postgres + Redis).

## 🛠️ Solution Approach

### Folder Structure
Implement the following structure exactly:

```
src/app/
├── domain/          # Pure python, no external dependencies
├── application/     # Orchestration, Interactors, Ports
├── infrastructure/  # Database, Adapters, External Libs
├── presentation/    # FastAPI, HTTP handlers
└── setup/           # DI Container, Config
```

### Dependencies
Add the following core libraries:
- `fastapi`, `uvicorn` (Web)
- `sqlalchemy[asyncio]`, `asyncpg`, `alembic` (DB)
- `pydantic-settings` (Config)
- `dishka` (Dependency Injection)

### Best Practices Explained
> **Clean Architecture**: We strictly separate layers to ensure the core business logic (Domain/Application) is independent of frameworks (FastAPI) and drivers (SQLAlchemy). Be careful not to import `infrastructure` code into `domain`.

## 🔗 References
- [Project Structure](../docs/backend/02-project-structure.md)
