# Ticket: BE-002 - Database Configuration and Migration Setup

**Epic**: Foundation & Infrastructure
**Type**: Task
**Priority**: High
**Dependencies**: BE-001

## 📝 Story
As a developer, I need a configured database connection and migration system so that I can persist application data and manage schema changes over time.

## ✅ Acceptance Criteria
- [ ] Async SQLAlchemy engine configured in `src/app/infrastructure/persistence/connection.py`.
- [ ] Alembic initialized for async migrations.
- [ ] `Makefile` or script created to run migrations (`make migrate`).
- [ ] Health check endpoint (`GET /health`) connects to DB to verify status.

## 🛠️ Solution Approach

### Infrastructure Layer
1. **Connection**: Create `get_session` factory using `sqlalchemy.ext.asyncio`.
2. **Configuration**: Load DB credentials from `Settings` (created in BE-001).

### Migration Workflow
We use Alembic for version control of our schema.
- **Env.py**: Configure `env.py` to run migrations in an async context.
- **Metadata**: Ensure `target_metadata` imports all your ORM mappings (even if empty for now) so autogenerate works later.

```python
# src/app/infrastructure/persistence/connection.py
async_engine = create_async_engine(settings.postgres.dsn)
async_session_maker = async_sessionmaker(async_engine, expire_on_commit=False)
```

### Best Practices Explained
> **AsyncIO**: We use async drivers (`asyncpg`) to ensure our API remains non-blocking under load.
> **Externalizing Config**: Database URLs must never be hardcoded. Use Pydantic Settings.
