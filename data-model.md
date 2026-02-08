# Data Model: Backend API & Database (Task Management Core)

**Feature**: Backend API & Database (Task Management Core)
**Date**: 2026-02-08
**Phase**: Phase 1 - Design & Contracts

## Overview

This document defines the data model for the task management system. The model enforces multi-user data isolation through user ownership and supports all CRUD operations defined in the feature specification.

---

## Entity: Task

### Description

Represents a single todo item that a user needs to track. Each task is owned by exactly one user and can only be accessed, modified, or deleted by that user.

### SQLModel Schema

```python
from sqlmodel import SQLModel, Field
from datetime import datetime
from typing import Optional

class Task(SQLModel, table=True):
    """Task entity with user ownership for data isolation"""

    __tablename__ = "tasks"

    # Primary key
    id: Optional[int] = Field(default=None, primary_key=True)

    # User ownership (foreign key to users table)
    user_id: str = Field(
        nullable=False,
        index=True,
        description="ID of the user who owns this task"
    )

    # Task content
    title: str = Field(
        max_length=200,
        nullable=False,
        description="Task title (required)"
    )

    description: Optional[str] = Field(
        default=None,
        max_length=2000,
        description="Optional task description"
    )

    # Task state
    is_completed: bool = Field(
        default=False,
        nullable=False,
        description="Whether the task is completed"
    )

    # Timestamps
    created_at: datetime = Field(
        default_factory=datetime.utcnow,
        nullable=False,
        description="When the task was created"
    )

    updated_at: datetime = Field(
        default_factory=datetime.utcnow,
        nullable=False,
        description="When the task was last updated"
    )
```

### Field Specifications

| Field | Type | Constraints | Default | Description |
|-------|------|-------------|---------|-------------|
| `id` | Integer | PRIMARY KEY, AUTO INCREMENT | Auto-generated | Unique task identifier |
| `user_id` | String | NOT NULL, INDEXED | - | Owner's user ID from JWT |
| `title` | String | NOT NULL, MAX 200 chars | - | Task title (required) |
| `description` | String | NULLABLE, MAX 2000 chars | NULL | Optional task description |
| `is_completed` | Boolean | NOT NULL | False | Completion status |
| `created_at` | Timestamp | NOT NULL | Current UTC time | Creation timestamp |
| `updated_at` | Timestamp | NOT NULL | Current UTC time | Last update timestamp |

### Indexes

```sql
-- Primary key index (automatic)
CREATE UNIQUE INDEX pk_tasks ON tasks(id);

-- User ownership index (for filtering)
CREATE INDEX idx_tasks_user_id ON tasks(user_id);

-- Composite index for sorted task lists
CREATE INDEX idx_tasks_user_id_created_at ON tasks(user_id, created_at DESC);
```

**Index Rationale**:
- `idx_tasks_user_id`: Optimizes all user-scoped queries (list, count)
- `idx_tasks_user_id_created_at`: Optimizes task list retrieval with default sorting (newest first)

### Constraints

- **NOT NULL on user_id**: Every task must have an owner
- **Foreign key to users**: Ensures referential integrity (if users table exists)
- **Length limits**: Prevents database bloat and ensures reasonable data sizes
- **Timestamp defaults**: Automatically tracks creation and modification times

---

## Pydantic Schemas (Request/Response)

### TaskCreate (Request)

```python
from pydantic import BaseModel, Field, validator

class TaskCreate(BaseModel):
    """Schema for creating a new task"""

    title: str = Field(
        ...,
        min_length=1,
        max_length=200,
        description="Task title (required)"
    )

    description: str | None = Field(
        None,
        max_length=2000,
        description="Optional task description"
    )

    is_completed: bool = Field(
        default=False,
        description="Initial completion status"
    )

    @validator('title')
    def title_not_empty(cls, v):
        """Ensure title is not just whitespace"""
        if not v.strip():
            raise ValueError('Title cannot be empty or whitespace')
        return v.strip()

    class Config:
        schema_extra = {
            "example": {
                "title": "Buy groceries",
                "description": "Milk, eggs, bread",
                "is_completed": False
            }
        }
```

### TaskUpdate (Request)

```python
class TaskUpdate(BaseModel):
    """Schema for updating an existing task"""

    title: str | None = Field(
        None,
        min_length=1,
        max_length=200,
        description="Updated task title"
    )

    description: str | None = Field(
        None,
        max_length=2000,
        description="Updated task description"
    )

    is_completed: bool | None = Field(
        None,
        description="Updated completion status"
    )

    @validator('title')
    def title_not_empty(cls, v):
        """Ensure title is not just whitespace if provided"""
        if v is not None and not v.strip():
            raise ValueError('Title cannot be empty or whitespace')
        return v.strip() if v else v

    class Config:
        schema_extra = {
            "example": {
                "title": "Buy groceries and cook dinner",
                "is_completed": True
            }
        }
```

### TaskResponse (Response)

```python
from datetime import datetime

class TaskResponse(BaseModel):
    """Schema for task responses"""

    id: int
    user_id: str
    title: str
    description: str | None
    is_completed: bool
    created_at: datetime
    updated_at: datetime

    class Config:
        orm_mode = True  # Enable ORM model conversion
        schema_extra = {
            "example": {
                "id": 1,
                "user_id": "user_123",
                "title": "Buy groceries",
                "description": "Milk, eggs, bread",
                "is_completed": False,
                "created_at": "2026-02-08T10:30:00Z",
                "updated_at": "2026-02-08T10:30:00Z"
            }
        }
```

---

## Relationships

### Task → User (Many-to-One)

- Each task belongs to exactly one user
- User is identified by `user_id` field
- User information comes from authentication system (not stored in this database)

**Note**: The User entity is managed by the authentication system (Better Auth). This backend only stores the user_id reference for ownership tracking.

---

## Data Validation Rules

### Title Validation
- **Required**: Cannot be null or empty
- **Length**: 1-200 characters
- **Whitespace**: Leading/trailing whitespace trimmed
- **Content**: Cannot be only whitespace

### Description Validation
- **Optional**: Can be null
- **Length**: 0-2000 characters if provided
- **Whitespace**: Preserved (not trimmed)

### Completion Status
- **Type**: Boolean
- **Default**: False (new tasks are incomplete)
- **Toggle**: Can be changed between true/false

### User ID
- **Required**: Cannot be null
- **Source**: Extracted from JWT token only
- **Immutable**: Cannot be changed after task creation

### Timestamps
- **Auto-generated**: Set automatically on creation
- **Auto-updated**: `updated_at` refreshed on every modification
- **Timezone**: UTC for consistency

---

## Database Migration

### Initial Migration (Alembic)

```python
"""Create tasks table

Revision ID: 001_create_tasks
Revises:
Create Date: 2026-02-08
"""

from alembic import op
import sqlalchemy as sa
from sqlalchemy.dialects import postgresql

def upgrade():
    op.create_table(
        'tasks',
        sa.Column('id', sa.Integer(), nullable=False),
        sa.Column('user_id', sa.String(), nullable=False),
        sa.Column('title', sa.String(length=200), nullable=False),
        sa.Column('description', sa.String(length=2000), nullable=True),
        sa.Column('is_completed', sa.Boolean(), nullable=False, server_default='false'),
        sa.Column('created_at', sa.DateTime(), nullable=False, server_default=sa.text('now()')),
        sa.Column('updated_at', sa.DateTime(), nullable=False, server_default=sa.text('now()')),
        sa.PrimaryKeyConstraint('id')
    )

    # Create indexes
    op.create_index('idx_tasks_user_id', 'tasks', ['user_id'])
    op.create_index('idx_tasks_user_id_created_at', 'tasks', ['user_id', 'created_at'])

def downgrade():
    op.drop_index('idx_tasks_user_id_created_at', table_name='tasks')
    op.drop_index('idx_tasks_user_id', table_name='tasks')
    op.drop_table('tasks')
```

---

## Data Isolation Enforcement

### Query Patterns

**Always filter by user_id**:
```python
# List all tasks for user
tasks = await db.execute(
    select(Task).where(Task.user_id == current_user_id)
)

# Get single task with ownership check
task = await db.execute(
    select(Task).where(
        Task.id == task_id,
        Task.user_id == current_user_id
    )
)

# Update task with ownership check
await db.execute(
    update(Task)
    .where(Task.id == task_id, Task.user_id == current_user_id)
    .values(title="Updated title")
)

# Delete task with ownership check
await db.execute(
    delete(Task)
    .where(Task.id == task_id, Task.user_id == current_user_id)
)
```

**Never**:
- Query tasks without user_id filter
- Accept user_id from request body or URL parameters
- Allow cross-user task access

---

## Performance Considerations

### Query Optimization
- Use indexes on `user_id` for fast filtering
- Composite index on `(user_id, created_at)` for sorted lists
- Limit result sets for large task lists (pagination)

### Connection Pooling
- Pool size: 5-10 connections
- Reuse connections for multiple queries
- Pre-ping to verify connection health

### Async Operations
- Use async/await for all database operations
- Non-blocking I/O improves concurrency
- Critical for handling 100+ concurrent users

---

## Testing Considerations

### Test Data
```python
# Test user IDs
USER_A_ID = "test_user_a"
USER_B_ID = "test_user_b"

# Test tasks
task_a1 = Task(user_id=USER_A_ID, title="User A Task 1")
task_a2 = Task(user_id=USER_A_ID, title="User A Task 2")
task_b1 = Task(user_id=USER_B_ID, title="User B Task 1")
```

### Isolation Tests
- Verify User A cannot access User B's tasks
- Verify queries filter by user_id correctly
- Verify concurrent operations don't leak data

---

## Summary

The Task data model provides:
- ✅ User ownership tracking via `user_id`
- ✅ Complete CRUD support
- ✅ Data isolation at query level
- ✅ Automatic timestamp management
- ✅ Validation rules for data integrity
- ✅ Optimized indexes for performance
- ✅ Pydantic schemas for API contracts
