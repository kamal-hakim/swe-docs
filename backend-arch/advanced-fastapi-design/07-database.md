# Database

## Overview

This document covers PostgreSQL configuration, connection management, migrations with Alembic, and database best practices.

## PostgreSQL Configuration

### Docker Setup

```yaml
# docker-compose.yml
services:
  postgres:
    image: postgres:16-alpine
    container_name: app_postgres
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./scripts/init-db.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:
```

### Connection String Format

```python
# Async connection string for SQLAlchemy
DATABASE_URL = "postgresql+psycopg://user:password@localhost:5432/dbname"

# Components:
# - Driver: psycopg (async driver for PostgreSQL)
# - Format: postgresql+psycopg://{user}:{password}@{host}:{port}/{database}
```

### Settings Configuration

```python
# src/app/setup/config/settings.py
from pydantic import Field, computed_field
from pydantic_settings import BaseSettings


class DatabaseSettings(BaseSettings):
    """Database configuration."""
    
    host: str = Field(default="localhost")
    port: int = Field(default=5432)
    user: str = Field(default="postgres")
    password: str = Field(default="postgres")
    database: str = Field(default="app")
    
    # Connection pool settings
    pool_size: int = Field(default=10)
    max_overflow: int = Field(default=20)
    pool_timeout: int = Field(default=30)
    pool_recycle: int = Field(default=1800)  # 30 minutes
    
    # Query settings
    echo: bool = Field(default=False)
    echo_pool: bool = Field(default=False)
    
    @computed_field
    @property
    def url(self) -> str:
        """Build async database URL."""
        return (
            f"postgresql+psycopg://{self.user}:{self.password}"
            f"@{self.host}:{self.port}/{self.database}"
        )
    
    @computed_field
    @property
    def sync_url(self) -> str:
        """Build sync database URL for Alembic."""
        return (
            f"postgresql://{self.user}:{self.password}"
            f"@{self.host}:{self.port}/{self.database}"
        )
```

## Connection Management

### Async Engine and Session Factory

```python
# src/app/infrastructure/persistence/connection.py
from contextlib import asynccontextmanager
from typing import AsyncGenerator

from sqlalchemy.ext.asyncio import (
    AsyncEngine,
    AsyncSession,
    async_sessionmaker,
    create_async_engine,
)

from app.setup.config.settings import DatabaseSettings


def create_engine(settings: DatabaseSettings) -> AsyncEngine:
    """Create SQLAlchemy async engine with connection pooling."""
    return create_async_engine(
        settings.url,
        echo=settings.echo,
        echo_pool=settings.echo_pool,
        pool_size=settings.pool_size,
        max_overflow=settings.max_overflow,
        pool_timeout=settings.pool_timeout,
        pool_recycle=settings.pool_recycle,
        pool_pre_ping=True,  # Verify connections before use
    )


def create_session_factory(
    engine: AsyncEngine,
) -> async_sessionmaker[AsyncSession]:
    """Create session factory with proper settings."""
    return async_sessionmaker(
        bind=engine,
        class_=AsyncSession,
        expire_on_commit=False,  # Don't expire objects after commit
        autoflush=False,  # Control flushing explicitly
    )


@asynccontextmanager
async def get_session(
    session_factory: async_sessionmaker[AsyncSession],
) -> AsyncGenerator[AsyncSession, None]:
    """Get a database session with automatic cleanup."""
    session = session_factory()
    try:
        yield session
    finally:
        await session.close()
```

### Connection Pool Diagram

```mermaid
graph TB
    subgraph "Application"
        R1[Request 1]
        R2[Request 2]
        R3[Request N]
    end
    
    subgraph "Connection Pool"
        P[Pool Manager]
        C1[Connection 1]
        C2[Connection 2]
        C3[Connection N]
        O[Overflow Connections]
    end
    
    subgraph "PostgreSQL"
        DB[(Database)]
    end
    
    R1 --> P
    R2 --> P
    R3 --> P
    
    P --> C1
    P --> C2
    P --> C3
    P -.->|on demand| O
    
    C1 --> DB
    C2 --> DB
    C3 --> DB
    O --> DB
```

## Alembic Migrations

### Setup

```ini
# alembic.ini
[alembic]
script_location = src/app/infrastructure/persistence/alembic
prepend_sys_path = .
version_path_separator = os

[post_write_hooks]
hooks = ruff
ruff.type = exec
ruff.executable = ruff
ruff.options = format REVISION_SCRIPT_FILENAME
```

### Environment Configuration

```python
# src/app/infrastructure/persistence/alembic/env.py
import asyncio
from logging.config import fileConfig

from alembic import context
from sqlalchemy import pool
from sqlalchemy.ext.asyncio import async_engine_from_config

from app.setup.config.settings import Settings
from app.infrastructure.persistence.mappings.registry import mapper_registry
from app.infrastructure.persistence.mappings.all import map_all_tables

# Map all tables to ensure metadata is populated
map_all_tables()

config = context.config

if config.config_file_name is not None:
    fileConfig(config.config_file_name)

target_metadata = mapper_registry.metadata


def get_url() -> str:
    """Get database URL from settings."""
    settings = Settings()
    return settings.database.sync_url


def run_migrations_offline() -> None:
    """Run migrations in 'offline' mode."""
    url = get_url()
    context.configure(
        url=url,
        target_metadata=target_metadata,
        literal_binds=True,
        dialect_opts={"paramstyle": "named"},
        compare_type=True,
        compare_server_default=True,
    )

    with context.begin_transaction():
        context.run_migrations()


def do_run_migrations(connection) -> None:
    """Run migrations with connection."""
    context.configure(
        connection=connection,
        target_metadata=target_metadata,
        compare_type=True,
        compare_server_default=True,
    )

    with context.begin_transaction():
        context.run_migrations()


async def run_async_migrations() -> None:
    """Run migrations in async mode."""
    configuration = config.get_section(config.config_ini_section) or {}
    configuration["sqlalchemy.url"] = get_url()
    
    connectable = async_engine_from_config(
        configuration,
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,
    )

    async with connectable.connect() as connection:
        await connection.run_sync(do_run_migrations)

    await connectable.dispose()


def run_migrations_online() -> None:
    """Run migrations in 'online' mode."""
    asyncio.run(run_async_migrations())


if context.is_offline_mode():
    run_migrations_offline()
else:
    run_migrations_online()
```

### Common Commands

```bash
# Create a new migration
alembic revision --autogenerate -m "Add users table"

# Apply all pending migrations
alembic upgrade head

# Rollback one migration
alembic downgrade -1

# Show current revision
alembic current

# Show migration history
alembic history --verbose

# Generate SQL without applying
alembic upgrade head --sql
```

### Migration Script Example

```python
# src/app/infrastructure/persistence/alembic/versions/001_create_users.py
"""Create users table.

Revision ID: 001
Revises: 
Create Date: 2024-01-01 00:00:00.000000
"""
from typing import Sequence, Union

from alembic import op
import sqlalchemy as sa

revision: str = '001'
down_revision: Union[str, None] = None
branch_labels: Union[str, Sequence[str], None] = None
depends_on: Union[str, Sequence[str], None] = None


def upgrade() -> None:
    # Create enum type
    op.execute("CREATE TYPE userrole AS ENUM ('USER', 'ADMIN', 'SUPER_ADMIN')")
    
    # Create users table
    op.create_table(
        'users',
        sa.Column('id', sa.UUID(), nullable=False),
        sa.Column('username', sa.String(32), nullable=False),
        sa.Column('password_hash', sa.LargeBinary(), nullable=False),
        sa.Column(
            'role',
            sa.Enum('USER', 'ADMIN', 'SUPER_ADMIN', name='userrole'),
            nullable=False,
            server_default='USER',
        ),
        sa.Column('is_active', sa.Boolean(), nullable=False, server_default='true'),
        sa.Column('created_at', sa.DateTime(timezone=True), server_default=sa.func.now()),
        sa.Column('updated_at', sa.DateTime(timezone=True), onupdate=sa.func.now()),
        sa.PrimaryKeyConstraint('id'),
        sa.UniqueConstraint('username'),
    )
    
    # Create index
    op.create_index('ix_users_username', 'users', ['username'])


def downgrade() -> None:
    op.drop_index('ix_users_username')
    op.drop_table('users')
    op.execute("DROP TYPE userrole")
```

## Database Best Practices

| Practice | Description |
|----------|-------------|
| **Use UUIDs** | Prefer UUID v7 for primary keys (time-ordered) |
| **Soft Deletes** | Use `is_active` flag instead of hard deletes |
| **Timestamps** | Always include `created_at` and `updated_at` |
| **Indexes** | Create indexes for frequently queried columns |
| **Constraints** | Enforce data integrity at database level |
| **Connection Pooling** | Configure pool size based on workload |

## Pre-Start Script

```python
# src/app/infrastructure/persistence/pre_start.py
import asyncio
import logging
from tenacity import after_log, before_log, retry, stop_after_attempt, wait_fixed

from sqlalchemy import select
from sqlalchemy.ext.asyncio import AsyncSession

from app.infrastructure.persistence.connection import create_engine, create_session_factory
from app.setup.config.settings import Settings

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

max_tries = 60 * 5  # 5 minutes
wait_seconds = 1


@retry(
    stop=stop_after_attempt(max_tries),
    wait=wait_fixed(wait_seconds),
    before=before_log(logger, logging.INFO),
    after=after_log(logger, logging.WARN),
)
async def check_db_ready() -> None:
    """Wait for database to be ready."""
    settings = Settings()
    engine = create_engine(settings.database)
    session_factory = create_session_factory(engine)
    
    async with session_factory() as session:
        await session.execute(select(1))
    
    await engine.dispose()
    logger.info("Database is ready!")


if __name__ == "__main__":
    asyncio.run(check_db_ready())
```

---

**Previous**: [Presentation Layer](06-presentation-layer.md) | **Next**: [ORM](08-orm.md)
