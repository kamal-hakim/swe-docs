# Presentation Layer

## Overview

The Presentation layer handles external requests, validates inputs, and routes them to the Application layer. In this architecture, it primarily consists of FastAPI controllers, Pydantic schemas, and error handlers.

```mermaid
graph TB
    subgraph "Presentation Layer"
        subgraph HTTP["HTTP Interface"]
            C[Controllers]
            S[Schemas]
            M[Middleware]
            E[Error Handlers]
        end
        
        subgraph Router["FastAPI Routers"]
            R1[/api/v1/auth]
            R2[/api/v1/users]
            R3[/api/v1/orders]
        end
    end
    
    C --> R1
    C --> R2
    C --> R3
    C --> S
    C --> E
```

## Controllers

Controllers are thin wrappers that validate input, call interactors, and format responses.

### Basic Controller Pattern

```python
# src/app/presentation/http/controllers/users.py
from uuid import UUID

from dishka.integrations.fastapi import DishkaRoute, FromDishka
from fastapi import APIRouter, status

from app.application.commands.create_user import (
    CreateUserInteractor,
    CreateUserRequest,
    CreateUserResponse,
)
from app.application.queries.list_users import (
    ListUsersQuery,
    ListUsersRequest,
    ListUsersResponse,
)
from app.presentation.http.schemas.user import (
    UserCreateSchema,
    UserPublicSchema,
    UserListSchema,
)

router = APIRouter(prefix="/users", tags=["Users"], route_class=DishkaRoute)


@router.post(
    "/",
    response_model=UserPublicSchema,
    status_code=status.HTTP_201_CREATED,
    summary="Create a new user",
    description="Create a new user account. Requires admin privileges.",
)
async def create_user(
    body: UserCreateSchema,
    interactor: FromDishka[CreateUserInteractor],
) -> UserPublicSchema:
    """Create a new user."""
    request = CreateUserRequest(
        username=body.username,
        password=body.password,
        role=body.role,
    )
    result = await interactor.execute(request)
    return UserPublicSchema(
        id=result["id"],
        username=result["username"],
    )


@router.get(
    "/",
    response_model=UserListSchema,
    summary="List all users",
    description="Get a paginated list of users. Requires admin privileges.",
)
async def list_users(
    page: int = 1,
    page_size: int = 20,
    is_active: bool | None = None,
    query: FromDishka[ListUsersQuery],
) -> UserListSchema:
    """List users with pagination."""
    request = ListUsersRequest(
        page=page,
        page_size=page_size,
        is_active=is_active,
    )
    result = await query.execute(request)
    return UserListSchema(
        users=[
            UserPublicSchema(id=u["id"], username=u["username"])
            for u in result["users"]
        ],
        total=result["total"],
        page=result["page"],
        page_size=result["page_size"],
    )


@router.get(
    "/{user_id}",
    response_model=UserPublicSchema,
    summary="Get user by ID",
)
async def get_user(
    user_id: UUID,
    query: FromDishka[GetUserQuery],
) -> UserPublicSchema:
    """Get a specific user by ID."""
    result = await query.execute(user_id)
    if not result:
        raise HTTPException(status_code=404, detail="User not found")
    return UserPublicSchema(**result)
```

### Authentication Controller

```python
# src/app/presentation/http/controllers/auth.py
from fastapi import APIRouter, Response, status
from dishka.integrations.fastapi import DishkaRoute, FromDishka

from app.infrastructure.auth.handlers.login import LoginHandler, LoginRequest
from app.infrastructure.auth.handlers.signup import SignupHandler, SignupRequest
from app.infrastructure.auth.handlers.logout import LogoutHandler
from app.presentation.http.schemas.auth import (
    LoginSchema,
    SignupSchema,
    TokenSchema,
)

router = APIRouter(prefix="/auth", tags=["Authentication"], route_class=DishkaRoute)


@router.post(
    "/signup",
    status_code=status.HTTP_201_CREATED,
    summary="Register a new account",
)
async def signup(
    body: SignupSchema,
    handler: FromDishka[SignupHandler],
) -> dict:
    """Register a new user account."""
    request = SignupRequest(
        username=body.username,
        password=body.password,
    )
    await handler.execute(request)
    return {"message": "Account created successfully"}


@router.post(
    "/login",
    response_model=TokenSchema,
    summary="Login and get access token",
)
async def login(
    body: LoginSchema,
    response: Response,
    handler: FromDishka[LoginHandler],
) -> TokenSchema:
    """Authenticate and receive access token."""
    request = LoginRequest(
        username=body.username,
        password=body.password,
    )
    result = await handler.execute(request)
    
    # Set token in HTTP-only cookie
    response.set_cookie(
        key="access_token",
        value=result.token,
        httponly=True,
        secure=True,
        samesite="lax",
        max_age=result.expires_in,
    )
    
    return TokenSchema(
        access_token=result.token,
        token_type="bearer",
        expires_in=result.expires_in,
    )


@router.delete(
    "/logout",
    status_code=status.HTTP_204_NO_CONTENT,
    summary="Logout and invalidate session",
)
async def logout(
    response: Response,
    handler: FromDishka[LogoutHandler],
) -> None:
    """Logout and invalidate current session."""
    await handler.execute()
    response.delete_cookie("access_token")
```

## Pydantic Schemas

Schemas handle request validation and response serialization.

```python
# src/app/presentation/http/schemas/base.py
from pydantic import BaseModel, ConfigDict


class BaseSchema(BaseModel):
    """Base schema with common configuration."""
    
    model_config = ConfigDict(
        from_attributes=True,
        populate_by_name=True,
        str_strip_whitespace=True,
    )


class PaginatedResponse(BaseSchema):
    """Base paginated response."""
    total: int
    page: int
    page_size: int
```

```python
# src/app/presentation/http/schemas/user.py
from uuid import UUID

from pydantic import Field

from app.domain.enums.user_role import UserRole
from app.presentation.http.schemas.base import BaseSchema, PaginatedResponse


class UserCreateSchema(BaseSchema):
    """Request schema for creating a user."""
    
    username: str = Field(
        ...,
        min_length=3,
        max_length=32,
        pattern=r"^[a-zA-Z][a-zA-Z0-9_-]*$",
        description="Username (letters, numbers, underscores, hyphens)",
        examples=["john_doe"],
    )
    password: str = Field(
        ...,
        min_length=8,
        max_length=128,
        description="Password (min 8 characters)",
    )
    role: UserRole = Field(
        default=UserRole.USER,
        description="User role",
    )


class UserPublicSchema(BaseSchema):
    """Public user representation."""
    
    id: UUID
    username: str
    role: str | None = None
    is_active: bool | None = None


class UserListSchema(PaginatedResponse):
    """Paginated user list response."""
    
    users: list[UserPublicSchema]
```

```python
# src/app/presentation/http/schemas/auth.py
from pydantic import Field

from app.presentation.http.schemas.base import BaseSchema


class LoginSchema(BaseSchema):
    """Login request schema."""
    
    username: str = Field(..., min_length=1)
    password: str = Field(..., min_length=1)


class SignupSchema(BaseSchema):
    """Signup request schema."""
    
    username: str = Field(
        ...,
        min_length=3,
        max_length=32,
        pattern=r"^[a-zA-Z][a-zA-Z0-9_-]*$",
    )
    password: str = Field(..., min_length=8, max_length=128)


class TokenSchema(BaseSchema):
    """Token response schema."""
    
    access_token: str
    token_type: str = "bearer"
    expires_in: int
```

## Error Handling

```python
# src/app/presentation/http/errors/handlers.py
from fastapi import FastAPI, Request, status
from fastapi.responses import JSONResponse

from app.application.common.exceptions.auth import (
    AuthenticationError,
    AuthorizationError,
)
from app.application.common.exceptions.base import NotFoundError
from app.domain.exceptions.base import ValidationError, BusinessRuleViolationError


def setup_error_handlers(app: FastAPI) -> None:
    """Register exception handlers."""
    
    @app.exception_handler(ValidationError)
    async def validation_error_handler(
        request: Request,
        exc: ValidationError,
    ) -> JSONResponse:
        return JSONResponse(
            status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
            content={
                "error": "validation_error",
                "message": str(exc),
            },
        )
    
    @app.exception_handler(BusinessRuleViolationError)
    async def business_rule_handler(
        request: Request,
        exc: BusinessRuleViolationError,
    ) -> JSONResponse:
        return JSONResponse(
            status_code=status.HTTP_409_CONFLICT,
            content={
                "error": "business_rule_violation",
                "message": str(exc),
            },
        )
    
    @app.exception_handler(AuthenticationError)
    async def auth_error_handler(
        request: Request,
        exc: AuthenticationError,
    ) -> JSONResponse:
        return JSONResponse(
            status_code=status.HTTP_401_UNAUTHORIZED,
            content={
                "error": "authentication_error",
                "message": str(exc),
            },
            headers={"WWW-Authenticate": "Bearer"},
        )
    
    @app.exception_handler(AuthorizationError)
    async def authz_error_handler(
        request: Request,
        exc: AuthorizationError,
    ) -> JSONResponse:
        return JSONResponse(
            status_code=status.HTTP_403_FORBIDDEN,
            content={
                "error": "authorization_error",
                "message": str(exc),
            },
        )
    
    @app.exception_handler(NotFoundError)
    async def not_found_handler(
        request: Request,
        exc: NotFoundError,
    ) -> JSONResponse:
        return JSONResponse(
            status_code=status.HTTP_404_NOT_FOUND,
            content={
                "error": "not_found",
                "message": str(exc),
            },
        )
```

## Middleware

```python
# src/app/presentation/http/middleware/request_id.py
import uuid
from contextvars import ContextVar

from starlette.middleware.base import BaseHTTPMiddleware
from starlette.requests import Request
from starlette.responses import Response

request_id_ctx: ContextVar[str] = ContextVar("request_id", default="")


class RequestIdMiddleware(BaseHTTPMiddleware):
    """Add unique request ID to each request."""
    
    async def dispatch(self, request: Request, call_next) -> Response:
        request_id = request.headers.get("X-Request-ID", str(uuid.uuid4()))
        request_id_ctx.set(request_id)
        
        response = await call_next(request)
        response.headers["X-Request-ID"] = request_id
        return response
```

## FastAPI App Factory

```python
# src/app/presentation/http/app.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

from app.presentation.http.controllers import auth, users, health
from app.presentation.http.errors.handlers import setup_error_handlers
from app.presentation.http.middleware.request_id import RequestIdMiddleware
from app.setup.config.settings import Settings


def create_fastapi_app(settings: Settings) -> FastAPI:
    """Create and configure FastAPI application."""
    
    app = FastAPI(
        title=settings.app.name,
        version=settings.app.version,
        debug=settings.app.debug,
        docs_url="/docs" if settings.app.debug else None,
        redoc_url="/redoc" if settings.app.debug else None,
    )
    
    # Middleware
    app.add_middleware(RequestIdMiddleware)
    app.add_middleware(
        CORSMiddleware,
        allow_origins=settings.cors.origins,
        allow_credentials=True,
        allow_methods=["*"],
        allow_headers=["*"],
    )
    
    # Error handlers
    setup_error_handlers(app)
    
    # Routes
    app.include_router(health.router, prefix="/api/v1")
    app.include_router(auth.router, prefix="/api/v1")
    app.include_router(users.router, prefix="/api/v1")
    
    return app
```

---

**Previous**: [Infrastructure Layer](05-infrastructure-layer.md) | **Next**: [Database](07-database.md)
