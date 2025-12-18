# Testing

## Overview

This architecture enables thorough testing at every layer through dependency injection and clean separation of concerns.

```mermaid
graph TB
    subgraph "Testing Pyramid"
        E2E[E2E Tests]
        INT[Integration Tests]
        UNIT[Unit Tests]
    end
    
    E2E --> INT
    INT --> UNIT
    
    style UNIT fill:#90EE90
    style INT fill:#FFD700
    style E2E fill:#FF6B6B
```

## Test Structure

```
tests/
├── conftest.py              # Shared fixtures
├── unit/                    # Unit tests (fast, isolated)
│   ├── domain/
│   │   ├── test_entities.py
│   │   └── test_value_objects.py
│   └── application/
│       ├── test_create_user.py
│       └── test_list_users.py
├── integration/             # Integration tests (real DB)
│   ├── conftest.py
│   ├── test_repositories.py
│   └── test_api.py
└── e2e/                     # End-to-end tests
    └── test_user_flow.py
```

## Fixtures

```python
# tests/conftest.py
import pytest
from unittest.mock import AsyncMock, Mock
from uuid import uuid4

from app.domain.entities.user import User
from app.domain.enums.user_role import UserRole
from app.domain.value_objects.user_id import UserId
from app.domain.value_objects.username import Username
from app.domain.value_objects.password_hash import PasswordHash


@pytest.fixture
def user_id() -> UserId:
    return UserId(uuid4())


@pytest.fixture
def user(user_id: UserId) -> User:
    return User(
        id_=user_id,
        username=Username("testuser"),
        password_hash=PasswordHash(b"hashed"),
        role=UserRole.USER,
        is_active=True,
    )


@pytest.fixture
def admin_user() -> User:
    return User(
        id_=UserId(),
        username=Username("admin"),
        password_hash=PasswordHash(b"hashed"),
        role=UserRole.ADMIN,
        is_active=True,
    )


@pytest.fixture
def mock_user_repository():
    return Mock()


@pytest.fixture
def mock_unit_of_work():
    uow = AsyncMock()
    uow.commit = AsyncMock()
    uow.rollback = AsyncMock()
    return uow
```

## Unit Tests

### Domain Layer

```python
# tests/unit/domain/test_value_objects.py
import pytest

from app.domain.value_objects.username import Username
from app.domain.value_objects.email import Email
from app.domain.exceptions.user import InvalidUsernameError
from app.domain.exceptions.validation import InvalidEmailError


class TestUsername:
    def test_valid_username(self):
        username = Username("john_doe")
        assert username.value == "john_doe"
    
    def test_username_too_short(self):
        with pytest.raises(InvalidUsernameError, match="at least 3"):
            Username("ab")
    
    def test_username_too_long(self):
        with pytest.raises(InvalidUsernameError, match="at most 32"):
            Username("a" * 33)
    
    def test_username_invalid_start(self):
        with pytest.raises(InvalidUsernameError, match="start with a letter"):
            Username("123user")
    
    def test_username_equality(self):
        assert Username("test") == Username("test")
        assert Username("test") != Username("other")


class TestEmail:
    def test_valid_email(self):
        email = Email("user@example.com")
        assert email.value == "user@example.com"
    
    def test_email_normalized_lowercase(self):
        email = Email("User@EXAMPLE.com")
        assert email.value == "user@example.com"
    
    def test_invalid_email(self):
        with pytest.raises(InvalidEmailError):
            Email("invalid")
```

### Application Layer

```python
# tests/unit/application/test_create_user.py
import pytest
from unittest.mock import AsyncMock, Mock
from uuid import uuid4

from app.application.commands.create_user import (
    CreateUserInteractor,
    CreateUserRequest,
)
from app.application.common.exceptions.auth import AuthorizationError
from app.domain.entities.user import User
from app.domain.enums.user_role import UserRole
from app.domain.value_objects.user_id import UserId
from app.domain.value_objects.username import Username


@pytest.fixture
def deps():
    """Create mock dependencies."""
    return {
        "current_user_service": AsyncMock(),
        "user_service": AsyncMock(),
        "user_repository": Mock(),
        "unit_of_work": AsyncMock(),
    }


@pytest.fixture
def interactor(deps):
    return CreateUserInteractor(**deps)


class TestCreateUser:
    async def test_create_user_success(self, interactor, deps, admin_user):
        # Arrange
        deps["current_user_service"].get_current_user.return_value = admin_user
        
        created_user = User(
            id_=UserId(),
            username=Username("newuser"),
            password_hash=Mock(value=b"hash"),
            role=UserRole.USER,
        )
        deps["user_service"].create_user.return_value = created_user
        
        request = CreateUserRequest(
            username="newuser",
            password="password123",
            role=UserRole.USER,
        )
        
        # Act
        result = await interactor.execute(request)
        
        # Assert
        assert result["username"] == "newuser"
        deps["user_repository"].add.assert_called_once_with(created_user)
        deps["unit_of_work"].commit.assert_called_once()
    
    async def test_create_user_unauthorized(self, interactor, deps, user):
        # Arrange - regular user trying to create
        deps["current_user_service"].get_current_user.return_value = user
        
        request = CreateUserRequest(
            username="newuser",
            password="password123",
            role=UserRole.USER,
        )
        
        # Act & Assert
        with pytest.raises(AuthorizationError):
            await interactor.execute(request)
    
    async def test_create_admin_requires_super_admin(self, interactor, deps, admin_user):
        # Arrange - admin trying to create admin
        deps["current_user_service"].get_current_user.return_value = admin_user
        
        request = CreateUserRequest(
            username="newadmin",
            password="password123",
            role=UserRole.ADMIN,  # Trying to create admin
        )
        
        # Act & Assert
        with pytest.raises(AuthorizationError, match="super admin"):
            await interactor.execute(request)
```

## Integration Tests

```python
# tests/integration/conftest.py
import pytest
import asyncio
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession

from app.infrastructure.persistence.mappings.all import map_all_tables
from app.infrastructure.persistence.mappings.registry import mapper_registry


@pytest.fixture(scope="session")
def event_loop():
    loop = asyncio.get_event_loop_policy().new_event_loop()
    yield loop
    loop.close()


@pytest.fixture(scope="session")
async def engine():
    """Create test database engine."""
    engine = create_async_engine(
        "postgresql+psycopg://test:test@localhost:5432/test",
        echo=True,
    )
    
    map_all_tables()
    
    async with engine.begin() as conn:
        await conn.run_sync(mapper_registry.metadata.create_all)
    
    yield engine
    
    async with engine.begin() as conn:
        await conn.run_sync(mapper_registry.metadata.drop_all)
    
    await engine.dispose()


@pytest.fixture
async def session(engine):
    """Create test session with rollback."""
    async with AsyncSession(engine) as session:
        async with session.begin():
            yield session
            await session.rollback()
```

```python
# tests/integration/test_repositories.py
import pytest
from uuid import uuid4

from app.domain.entities.user import User
from app.domain.enums.user_role import UserRole
from app.domain.value_objects.user_id import UserId
from app.domain.value_objects.username import Username
from app.domain.value_objects.password_hash import PasswordHash
from app.infrastructure.persistence.repositories.user_repository import (
    SQLAlchemyUserRepository,
)


@pytest.fixture
def user_repo(session):
    return SQLAlchemyUserRepository(session)


class TestUserRepository:
    async def test_add_and_get_user(self, user_repo, session):
        # Arrange
        user = User(
            id_=UserId(),
            username=Username("testuser"),
            password_hash=PasswordHash(b"hash"),
            role=UserRole.USER,
            is_active=True,
        )
        
        # Act
        user_repo.add(user)
        await session.flush()
        
        found = await user_repo.get_by_id(user.id_)
        
        # Assert
        assert found is not None
        assert found.username.value == "testuser"
    
    async def test_get_by_username(self, user_repo, session):
        # Arrange
        user = User(
            id_=UserId(),
            username=Username("findme"),
            password_hash=PasswordHash(b"hash"),
            role=UserRole.USER,
        )
        user_repo.add(user)
        await session.flush()
        
        # Act
        found = await user_repo.get_by_username(Username("findme"))
        
        # Assert
        assert found is not None
        assert found.id_ == user.id_
```

## API Tests

```python
# tests/integration/test_api.py
import pytest
from httpx import AsyncClient
from fastapi import status

from app.setup.app_factory import create_app


@pytest.fixture
async def client():
    app = create_app()
    async with AsyncClient(app=app, base_url="http://test") as client:
        yield client


class TestUsersAPI:
    async def test_list_users_unauthorized(self, client):
        response = await client.get("/api/v1/users")
        assert response.status_code == status.HTTP_401_UNAUTHORIZED
    
    async def test_create_user(self, client, auth_headers):
        response = await client.post(
            "/api/v1/users",
            json={
                "username": "newuser",
                "password": "password123",
                "role": "USER",
            },
            headers=auth_headers,
        )
        assert response.status_code == status.HTTP_201_CREATED
        data = response.json()
        assert data["username"] == "newuser"
```

## Running Tests

```bash
# Run all tests
pytest

# Run with coverage
pytest --cov=app --cov-report=html

# Run specific test file
pytest tests/unit/application/test_create_user.py

# Run with verbose output
pytest -v

# Run only unit tests
pytest tests/unit/

# Run with parallel execution
pytest -n auto
```

## pytest.ini

```ini
[pytest]
asyncio_mode = auto
testpaths = tests
python_files = test_*.py
python_functions = test_*
addopts = -v --tb=short
filterwarnings =
    ignore::DeprecationWarning
```

---

**Previous**: [Redis](12-redis.md) | **Next**: [Code Quality](14-code-quality.md)
