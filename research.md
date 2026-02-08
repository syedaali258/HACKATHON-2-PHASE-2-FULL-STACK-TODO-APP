# Research: Backend API & Database Technology Decisions

**Feature**: Backend API & Database (Task Management Core)
**Date**: 2026-02-08
**Phase**: Phase 0 - Research & Technology Decisions

## Overview

This document consolidates research findings for implementing a secure, multi-user REST API with FastAPI, SQLModel, and Neon Serverless PostgreSQL. All decisions prioritize security, data isolation, and alignment with the project constitution.

---

## 1. FastAPI JWT Authentication Patterns

### Decision: Dependency Injection with `get_current_user`

**Implementation Pattern**:
```python
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from jose import JWTError, jwt

security = HTTPBearer()

async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(security)
) -> str:
    """Extract and verify JWT, return user_id"""
    token = credentials.credentials
    try:
        payload = jwt.decode(
            token,
            settings.BETTER_AUTH_SECRET,
            algorithms=[settings.JWT_ALGORITHM]
        )
        user_id: str = payload.get("sub")
        if user_id is None:
            raise HTTPException(status_code=401, detail="Invalid token")
        return user_id
    except JWTError:
        raise HTTPException(status_code=401, detail="Invalid token")
```

**Usage in Routes**:
```python
@router.get("/tasks")
async def list_tasks(
    user_id: str = Depends(get_current_user),
    db: Session = Depends(get_db)
):
    tasks = db.query(Task).filter(Task.user_id == user_id).all()
    return tasks
```

**Rationale**:
- FastAPI's dependency injection system automatically handles authentication
- Centralizes JWT verification logic (DRY principle)
- Automatically returns 401 for invalid/missing tokens
- Makes user_id available to all route handlers
- Easy to test by mocking the dependency

**Alternatives Considered**:
- **Middleware**: More complex, harder to test individual routes, less granular control
- **Manual verification in each route**: Violates DRY, error-prone, inconsistent error handling
- **Decorator pattern**: Less idiomatic in FastAPI, doesn't integrate with OpenAPI docs

**Implementation Guidance**:
1. Create `app/core/security.py` with JWT verification logic
2. Create `app/api/deps.py` with `get_current_user` dependency
3. Use `Depends(get_current_user)` in all protected route handlers
4. Configure HTTPBearer security scheme for OpenAPI documentation

---

## 2. SQLModel with Neon PostgreSQL

### Decision: Async SQLAlchemy Engine with Connection Pooling

**Connection Configuration**:
```python
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker

engine = create_async_engine(
    settings.DATABASE_URL,
    echo=False,
    pool_size=5,
    max_overflow=10,
    pool_recycle=3600,
    pool_pre_ping=True,
)

async_session = sessionmaker(
    engine, class_=AsyncSession, expire_on_commit=False
)
```

**Rationale**:
- **Async operations**: Improves concurrency for I/O-bound operations
- **Pool size (5-10)**: Balances connection reuse with resource efficiency in serverless
- **Max overflow (10)**: Handles traffic spikes without exhausting connections
- **Pool recycle (3600s)**: Prevents stale connections in Neon's serverless environment
- **Pool pre-ping (True)**: Verifies connection health before use, prevents errors

**Neon-Specific Considerations**:
- Neon serverless automatically scales compute, but connections are limited
- Connection pooling reduces overhead of establishing new connections
- Pre-ping is critical because Neon may recycle idle connections
- Use `postgresql+asyncpg` driver for best async performance

**Alternatives Considered**:
- **Synchronous SQLAlchemy**: Simpler but blocks on I/O, poor concurrency
- **No pooling**: High connection overhead, poor performance
- **Large pool (50+)**: Wastes resources, may exceed Neon connection limits

**Implementation Guidance**:
1. Install dependencies: `sqlalchemy[asyncio]`, `asyncpg`, `sqlmodel`
2. Create `app/database.py` with async engine and session factory
3. Use async context managers for database sessions
4. Implement `get_db` dependency for route handlers
5. Use `await` for all database operations

**Migration Strategy**:
- Use Alembic for schema migrations
- Configure Alembic for async operations
- Initial migration creates `tasks` table with indexes
- Ensure migrations preserve data isolation constraints (user_id NOT NULL)

---

## 3. Data Isolation Patterns

### Decision: Query-Level Filtering with User ID

**Pattern Implementation**:
```python
# Always filter by user_id from JWT
async def get_user_tasks(db: AsyncSession, user_id: str):
    result = await db.execute(
        select(Task).where(Task.user_id == user_id)
    )
    return result.scalars().all()

# Ownership verification for single task
async def get_user_task(db: AsyncSession, task_id: int, user_id: str):
    result = await db.execute(
        select(Task).where(
            Task.id == task_id,
            Task.user_id == user_id
        )
    )
    task = result.scalar_one_or_none()
    if task is None:
        raise HTTPException(status_code=404, detail="Task not found")
    return task
```

**Rationale**:
- **Query-level enforcement**: Data isolation guaranteed at database level
- **Explicit filtering**: Every query includes `user_id` filter
- **No implicit access**: Missing user_id filter causes query to fail
- **404 for unauthorized**: Returns "not found" instead of "forbidden" to prevent information leakage

**Index Optimization**:
```sql
CREATE INDEX idx_tasks_user_id ON tasks(user_id);
CREATE INDEX idx_tasks_user_id_created_at ON tasks(user_id, created_at DESC);
```

**Rationale for Indexes**:
- `user_id` index: Speeds up all user-scoped queries
- Composite index: Optimizes task list retrieval with sorting
- Neon benefits from indexes due to serverless compute scaling

**Preventing N+1 Queries**:
- Use `selectinload()` or `joinedload()` for relationships (if added later)
- Batch operations when possible
- Monitor query performance with logging

**Alternatives Considered**:
- **Row-level security (RLS)**: PostgreSQL feature, but adds complexity and requires database-level user context
- **Application-level filtering after fetch**: Insecure, data leakage risk
- **Separate databases per user**: Over-engineering, poor resource utilization

**Implementation Guidance**:
1. Always include `user_id` in WHERE clauses
2. Create database indexes on `user_id` columns
3. Return 404 (not 403) for unauthorized access to prevent enumeration
4. Write integration tests verifying data isolation
5. Use database transactions for consistency

---

## 4. API Error Handling

### Decision: FastAPI HTTPException with Standardized Format

**Error Response Format**:
```python
from fastapi import HTTPException

# Authentication error
raise HTTPException(
    status_code=401,
    detail="Invalid or expired token"
)

# Authorization error (ownership check failed)
raise HTTPException(
    status_code=404,  # Use 404 to prevent enumeration
    detail="Task not found"
)

# Validation error (handled automatically by Pydantic)
# Returns 422 with detailed validation errors

# Server error
raise HTTPException(
    status_code=500,
    detail="Internal server error"
)
```

**HTTP Status Code Conventions**:
- **200 OK**: Successful GET, PUT requests
- **201 Created**: Successful POST (resource created)
- **204 No Content**: Successful DELETE
- **400 Bad Request**: Malformed request (custom validation)
- **401 Unauthorized**: Missing or invalid authentication
- **403 Forbidden**: Authenticated but not authorized (avoid using, use 404 instead)
- **404 Not Found**: Resource doesn't exist OR user doesn't own it
- **422 Unprocessable Entity**: Pydantic validation errors
- **500 Internal Server Error**: Unexpected server errors

**Rationale**:
- **FastAPI HTTPException**: Built-in, consistent, integrates with OpenAPI
- **404 for ownership failures**: Prevents information leakage (can't enumerate other users' tasks)
- **Pydantic validation**: Automatic, detailed error messages for request validation
- **Consistent format**: All errors return `{"detail": "message"}` structure

**Validation Error Handling**:
```python
from pydantic import BaseModel, Field, validator

class TaskCreate(BaseModel):
    title: str = Field(..., min_length=1, max_length=200)
    description: str | None = Field(None, max_length=2000)
    is_completed: bool = False

    @validator('title')
    def title_not_empty(cls, v):
        if not v.strip():
            raise ValueError('Title cannot be empty or whitespace')
        return v.strip()
```

**Rationale for Pydantic Validation**:
- Automatic validation before route handler executes
- Returns 422 with detailed field-level errors
- Type coercion and conversion
- Self-documenting (appears in OpenAPI schema)

**Alternatives Considered**:
- **Custom error classes**: Over-engineering, FastAPI HTTPException sufficient
- **403 for ownership failures**: Reveals information about resource existence
- **Manual validation**: Error-prone, inconsistent, verbose

**Implementation Guidance**:
1. Use FastAPI HTTPException for all errors
2. Return 404 (not 403) for ownership check failures
3. Use Pydantic models with Field validators for request validation
4. Log errors for monitoring (but don't expose internal details to clients)
5. Use exception handlers for unexpected errors (500)

**Global Exception Handler** (optional):
```python
from fastapi import Request
from fastapi.responses import JSONResponse

@app.exception_handler(Exception)
async def global_exception_handler(request: Request, exc: Exception):
    # Log the error
    logger.error(f"Unexpected error: {exc}", exc_info=True)
    # Return generic error to client
    return JSONResponse(
        status_code=500,
        content={"detail": "Internal server error"}
    )
```

---

## Summary of Key Decisions

| Area | Decision | Rationale |
|------|----------|-----------|
| **Authentication** | FastAPI dependency injection with `get_current_user` | Centralized, testable, idiomatic FastAPI pattern |
| **Database** | Async SQLAlchemy with connection pooling (5-10 connections) | Optimized for Neon serverless, improves concurrency |
| **Data Isolation** | Query-level filtering with user_id in WHERE clauses | Guaranteed isolation at database level, prevents leakage |
| **Error Handling** | FastAPI HTTPException with 404 for ownership failures | Consistent format, prevents information enumeration |
| **Validation** | Pydantic models with Field validators | Automatic, detailed errors, self-documenting |
| **Indexes** | user_id and composite (user_id, created_at) indexes | Optimizes user-scoped queries, critical for performance |

---

## Implementation Checklist

- [ ] Install dependencies: fastapi, sqlmodel, asyncpg, python-jose, pydantic
- [ ] Configure async SQLAlchemy engine with connection pooling
- [ ] Implement `get_current_user` dependency for JWT verification
- [ ] Create Task SQLModel with user_id foreign key
- [ ] Add database indexes on user_id columns
- [ ] Implement query-level filtering in all database operations
- [ ] Use HTTPException with appropriate status codes
- [ ] Return 404 (not 403) for ownership check failures
- [ ] Write integration tests for data isolation
- [ ] Configure Alembic for database migrations

---

## References

- FastAPI Security: https://fastapi.tiangolo.com/tutorial/security/
- SQLModel Documentation: https://sqlmodel.tiangolo.com/
- Neon Serverless PostgreSQL: https://neon.tech/docs/
- JWT Best Practices: https://tools.ietf.org/html/rfc8725
- REST API Status Codes: https://www.rfc-editor.org/rfc/rfc7231
