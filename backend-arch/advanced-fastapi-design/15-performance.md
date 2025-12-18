# Performance

## Overview

Performance optimization in this architecture focuses on async patterns, database optimization, caching, and efficient serialization.

## Async Best Practices

### Use Async All the Way

```python
# ✅ Good - fully async
async def get_user(user_id: UUID) -> User:
    user = await repository.get_by_id(user_id)
    return user

# ❌ Bad - blocking in async context
async def get_user(user_id: UUID) -> User:
    user = sync_repository.get_by_id(user_id)  # Blocks event loop!
    return user
```

### Run Blocking Code in Thread Pool

```python
import asyncio
from concurrent.futures import ThreadPoolExecutor

executor = ThreadPoolExecutor(max_workers=4)

async def hash_password(password: str) -> bytes:
    """CPU-intensive bcrypt - run in thread pool."""
    loop = asyncio.get_running_loop()
    return await loop.run_in_executor(
        executor,
        lambda: bcrypt.hashpw(password.encode(), bcrypt.gensalt(12)),
    )
```

### Concurrent Gathering

```python
async def get_dashboard_data(user_id: UUID) -> dict:
    """Fetch multiple resources concurrently."""
    user, orders, notifications = await asyncio.gather(
        user_service.get_user(user_id),
        order_service.get_recent_orders(user_id, limit=5),
        notification_service.get_unread(user_id),
    )
    return {
        "user": user,
        "orders": orders,
        "notifications": notifications,
    }
```

## Database Optimization

### Connection Pooling

```python
engine = create_async_engine(
    database_url,
    pool_size=10,        # Base pool size
    max_overflow=20,     # Extra connections if needed
    pool_timeout=30,     # Wait time for connection
    pool_recycle=1800,   # Recycle connections after 30 min
    pool_pre_ping=True,  # Validate connections
)
```

### Query Optimization

```python
# Select only needed columns
stmt = select(
    users_table.c.id,
    users_table.c.username,
).where(users_table.c.is_active == True)

# Use indexes
# CREATE INDEX ix_users_username ON users(username);

# Pagination with keyset (more efficient than OFFSET)
stmt = (
    select(User)
    .where(users_table.c.id > last_seen_id)
    .order_by(users_table.c.id)
    .limit(page_size)
)
```

### Eager Loading

```python
from sqlalchemy.orm import selectinload, joinedload

# Load relationships in single query
stmt = (
    select(Order)
    .options(selectinload(Order.items))
    .where(orders_table.c.user_id == user_id)
)
```

## Caching Strategies

| Strategy | Description | TTL | Use Case |
|----------|-------------|-----|----------|
| **Read-through** | Check cache, load if miss | 5-15 min | User profiles |
| **Write-through** | Update cache on write | N/A | Session data |
| **Cache-aside** | App manages cache | Varies | Query results |
| **Invalidation** | Delete on mutation | N/A | Related data |

```python
# Cache-aside pattern
async def get_user(user_id: UUID) -> User:
    cache_key = f"user:{user_id}"
    
    # Check cache
    cached = await cache.get(cache_key)
    if cached:
        return User(**cached)
    
    # Load from DB
    user = await repository.get_by_id(user_id)
    
    # Store in cache
    await cache.set(cache_key, user.to_dict(), ttl=600)
    
    return user
```

## Response Optimization

### Use TypedDict for Response DTOs

```python
# TypedDict is ~2x faster than dataclass
class UserResponse(TypedDict):
    id: str
    username: str
    role: str

# vs dataclass (slower creation)
@dataclass
class UserResponse:
    id: str
    username: str
    role: str
```

### ORJSON for Fast Serialization

```python
# In FastAPI
from fastapi.responses import ORJSONResponse

app = FastAPI(default_response_class=ORJSONResponse)
```

## Async Task Offloading

```python
# Offload heavy work to Celery
class CreateReportInteractor:
    async def execute(self, request: ReportRequest) -> dict:
        # Don't block - queue the task
        task_id = self._task_queue.enqueue(
            "generate_report",
            str(request.user_id),
            request.date_range,
        )
        return {"task_id": task_id, "status": "queued"}
```

## Profiling

```python
# pip install line-profiler
# kernprof -l -v script.py

from line_profiler import profile

@profile
def expensive_function():
    # ... code to profile
    pass
```

## Key Metrics

| Metric | Target | Tool |
|--------|--------|------|
| Response time (p95) | < 100ms | Prometheus |
| Database queries/request | < 5 | Debug middleware |
| Memory usage | Stable | Prometheus |
| Connection pool usage | < 80% | SQLAlchemy events |

---

**Previous**: [Code Quality](14-code-quality.md) | **Next**: [Deployment](16-deployment.md)
