# Data Model: Multi-User Todo Full-Stack Web Application

**Feature**: 002-multi-user-todo-app
**Date**: 2026-02-08
**Purpose**: Define database schema and entity relationships

## Entities

### User

**Purpose**: Represents a registered user account with authentication credentials.

**Attributes**:
- `id` (UUID, Primary Key): Unique identifier for the user
- `email` (String, Unique, Not Null): User's email address (used for authentication)
- `hashed_password` (String, Not Null): Bcrypt-hashed password
- `created_at` (DateTime, Not Null): Timestamp when account was created
- `updated_at` (DateTime, Not Null): Timestamp when account was last updated

**Validation Rules**:
- Email must be valid format (RFC 5322)
- Email must be unique across all users
- Password must be hashed before storage (never store plaintext)
- Minimum password length: 8 characters (enforced before hashing)

**Relationships**:
- One user has many tasks (one-to-many)

**Indexes**:
- Primary key index on `id` (automatic)
- Unique index on `email` (for fast lookup during signin)

**Security Considerations**:
- Password field is write-only (never returned in API responses)
- Email is case-insensitive for uniqueness checks
- User ID is derived from JWT token, never from client input

---

### Task

**Purpose**: Represents a todo item owned by a specific user.

**Attributes**:
- `id` (UUID, Primary Key): Unique identifier for the task
- `user_id` (UUID, Foreign Key, Not Null): Reference to owning user
- `title` (String, Not Null): Task title (max 200 characters)
- `description` (String, Nullable): Optional task description (max 2000 characters)
- `is_complete` (Boolean, Not Null, Default: False): Completion status
- `created_at` (DateTime, Not Null): Timestamp when task was created
- `updated_at` (DateTime, Not Null): Timestamp when task was last updated

**Validation Rules**:
- Title must not be empty (minimum 1 character)
- Title maximum length: 200 characters
- Description maximum length: 2000 characters (if provided)
- `is_complete` defaults to False for new tasks
- `user_id` must reference an existing user

**Relationships**:
- Many tasks belong to one user (many-to-one)
- Foreign key constraint on `user_id` references `User.id`
- Cascade delete: When user is deleted, all their tasks are deleted

**Indexes**:
- Primary key index on `id` (automatic)
- Index on `user_id` (critical for query performance - all queries filter by user)
- Composite index on `(user_id, created_at DESC)` (for sorted task list queries)
- Composite index on `(user_id, is_complete)` (for filtered queries by completion status)

**Security Considerations**:
- All queries MUST filter by authenticated user's ID
- Task ownership verified before any update or delete operation
- No task can be transferred between users (user_id is immutable after creation)

---

## Entity Relationship Diagram

```
┌─────────────────────┐
│       User          │
├─────────────────────┤
│ id (PK)             │
│ email (UNIQUE)      │
│ hashed_password     │
│ created_at          │
│ updated_at          │
└─────────────────────┘
          │
          │ 1
          │
          │ owns
          │
          │ *
          ▼
┌─────────────────────┐
│       Task          │
├─────────────────────┤
│ id (PK)             │
│ user_id (FK)        │
│ title               │
│ description         │
│ is_complete         │
│ created_at          │
│ updated_at          │
└─────────────────────┘
```

**Relationship**: One User → Many Tasks (1:N)

---

## State Transitions

### Task Completion Status

```
┌─────────────┐
│  Incomplete │ (is_complete = False)
│  (default)  │
└─────────────┘
      │
      │ PATCH /api/tasks/{id}/complete
      ▼
┌─────────────┐
│  Complete   │ (is_complete = True)
└─────────────┘
      │
      │ PATCH /api/tasks/{id}/complete (toggle)
      ▼
┌─────────────┐
│  Incomplete │ (is_complete = False)
└─────────────┘
```

**Rules**:
- Tasks are created in incomplete state by default
- Completion status can be toggled between True and False
- No other states exist (binary: complete or incomplete)

---

## Database Schema (SQLModel)

### User Model

```python
from sqlmodel import SQLModel, Field
from datetime import datetime
from uuid import UUID, uuid4

class User(SQLModel, table=True):
    __tablename__ = "users"

    id: UUID = Field(default_factory=uuid4, primary_key=True)
    email: str = Field(unique=True, index=True, nullable=False)
    hashed_password: str = Field(nullable=False)
    created_at: datetime = Field(default_factory=datetime.utcnow, nullable=False)
    updated_at: datetime = Field(default_factory=datetime.utcnow, nullable=False)
```

### Task Model

```python
from sqlmodel import SQLModel, Field
from datetime import datetime
from uuid import UUID, uuid4

class Task(SQLModel, table=True):
    __tablename__ = "tasks"

    id: UUID = Field(default_factory=uuid4, primary_key=True)
    user_id: UUID = Field(foreign_key="users.id", nullable=False, index=True)
    title: str = Field(max_length=200, nullable=False)
    description: str | None = Field(default=None, max_length=2000)
    is_complete: bool = Field(default=False, nullable=False)
    created_at: datetime = Field(default_factory=datetime.utcnow, nullable=False)
    updated_at: datetime = Field(default_factory=datetime.utcnow, nullable=False)
```

---

## Query Patterns

### User Queries

**Signup (Create User)**:
```sql
INSERT INTO users (id, email, hashed_password, created_at, updated_at)
VALUES (?, ?, ?, ?, ?)
```

**Signin (Find User by Email)**:
```sql
SELECT id, email, hashed_password, created_at, updated_at
FROM users
WHERE email = ?
```

### Task Queries (All filtered by user_id)

**List All Tasks for User**:
```sql
SELECT id, user_id, title, description, is_complete, created_at, updated_at
FROM tasks
WHERE user_id = ?
ORDER BY created_at DESC
```

**List Tasks by Completion Status**:
```sql
SELECT id, user_id, title, description, is_complete, created_at, updated_at
FROM tasks
WHERE user_id = ? AND is_complete = ?
ORDER BY created_at DESC
```

**Create Task**:
```sql
INSERT INTO tasks (id, user_id, title, description, is_complete, created_at, updated_at)
VALUES (?, ?, ?, ?, ?, ?, ?)
```

**Get Single Task**:
```sql
SELECT id, user_id, title, description, is_complete, created_at, updated_at
FROM tasks
WHERE id = ? AND user_id = ?
```

**Update Task**:
```sql
UPDATE tasks
SET title = ?, description = ?, updated_at = ?
WHERE id = ? AND user_id = ?
```

**Toggle Task Completion**:
```sql
UPDATE tasks
SET is_complete = NOT is_complete, updated_at = ?
WHERE id = ? AND user_id = ?
```

**Delete Task**:
```sql
DELETE FROM tasks
WHERE id = ? AND user_id = ?
```

**Critical**: All task queries include `user_id` filter to enforce data isolation.

---

## Migration Strategy

**Initial Schema Creation**:
1. Create `users` table with indexes
2. Create `tasks` table with foreign key constraint and indexes
3. No seed data required (users register via signup endpoint)

**Schema Evolution**:
- Use Alembic for future migrations if schema changes are needed
- Preserve data isolation constraints in all migrations
- Test migrations against Neon PostgreSQL before applying to production

---

## Performance Considerations

**Indexes**:
- `users.email` index: Fast signin lookups
- `tasks.user_id` index: Fast task list queries
- `tasks.(user_id, created_at)` composite index: Optimized sorted list queries
- `tasks.(user_id, is_complete)` composite index: Optimized filtered queries

**Query Optimization**:
- All task queries use indexed `user_id` column
- Limit result sets if task count grows large (pagination not required for MVP)
- Use connection pooling to reduce latency

**Scalability**:
- UUID primary keys enable distributed ID generation
- No sequential IDs that could leak user/task counts
- Foreign key constraints maintain referential integrity
