


# Implementation Plan: Backend API & Database (Task Management Core)

**Branch**: `003-backend-task-api` | **Date**: 2026-02-08 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/003-backend-task-api/spec.md`

**Note**: This template is filled in by the `/sp.plan` command. See `.specify/templates/commands/plan.md` for the execution workflow.

## Summary

Implement a secure, multi-user REST API for task management with persistent storage in Neon Serverless PostgreSQL. The backend enforces JWT-based authentication on all endpoints, extracts user identity exclusively from tokens, and ensures complete data isolation at the database query level. The API supports full CRUD operations (Create, Read, Update, Delete) for tasks, with each task automatically associated with its owner and accessible only by that user.

## Technical Context

**Language/Version**: Python 3.11+
**Primary Dependencies**: FastAPI, SQLModel, Pydantic v2, python-jose[cryptography] (JWT), psycopg2-binary (PostgreSQL driver)
**Storage**: Neon Serverless PostgreSQL with connection pooling
**Testing**: pytest with pytest-asyncio for async tests
**Target Platform**: Web server (Linux/cloud environment, serverless-compatible)
**Project Type**: Web application (backend API)
**Performance Goals**: Task creation <2s, list retrieval (1000 tasks) <3s, completion toggle <1s
**Constraints**: Support 100 concurrent users, JWT authentication required on all endpoints, 99.9% data consistency
**Scale/Scope**: Multi-user system with user-scoped data isolation, 5 REST endpoints, 1 data model (Task)

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

### Principle I: Spec-Driven Development ✅
- Feature derived from written specification (spec.md)
- Following workflow: Spec → Plan → Tasks → Implementation
- All requirements traceable to spec.md functional requirements (FR-001 through FR-015)

### Principle II: Security-First Architecture ✅
- JWT authentication enforced on all 5 API endpoints
- User identity extracted exclusively from JWT token (FR-003)
- No user_id accepted from request parameters or body
- Data isolation enforced at database query level (FR-004)
- Unauthorized requests return HTTP 401/403 consistently (FR-013)
- Token verification using shared secret (BETTER_AUTH_SECRET)

### Principle III: Technology Stack Compliance ✅
- Backend: Python FastAPI (as specified)
- ORM: SQLModel (as specified)
- Database: Neon Serverless PostgreSQL (as specified)
- Authentication: JWT verification (Better Auth integration)
- No deviations from approved stack

### Principle IV: API Contract Enforcement ✅
- REST API with standard HTTP semantics
- Endpoints follow RESTful conventions (GET, POST, PUT, DELETE)
- Appropriate HTTP status codes (200, 201, 400, 401, 403, 404, 500)
- API contracts will be documented in contracts/ directory
- Frontend will consume backend exclusively via authenticated API calls

### Principle V: Data Isolation & Multi-Tenancy ✅
- Task model includes user_id foreign key for ownership
- All queries filter by authenticated user's ID
- No cross-user data access permitted
- Database schema supports multi-user isolation
- Persistent storage for all tasks (FR-001)

### Principle VI: Deterministic & Reproducible Builds ✅
- Following Agentic Dev Stack workflow
- PHRs will be created for all significant prompts
- ADRs will be suggested for architectural decisions
- All changes traceable to spec requirements

**Gate Status**: ✅ PASSED - No constitution violations detected

## Project Structure

### Documentation (this feature)

```text
specs/003-backend-task-api/
├── spec.md              # Feature specification (completed)
├── plan.md              # This file (/sp.plan command output)
├── research.md          # Phase 0 output (technology best practices)
├── data-model.md        # Phase 1 output (Task entity schema)
├── quickstart.md        # Phase 1 output (setup and run instructions)
├── contracts/           # Phase 1 output (OpenAPI/REST contracts)
│   └── task-api.yaml    # REST API contract for task endpoints
├── checklists/          # Quality validation checklists
│   └── requirements.md  # Spec quality checklist (completed)
└── tasks.md             # Phase 2 output (/sp.tasks command - NOT created by /sp.plan)
```

### Source Code (repository root)

```text
backend/
├── app/
│   ├── __init__.py
│   ├── main.py              # FastAPI application entry point
│   ├── config.py            # Configuration and environment variables
│   ├── database.py          # Database connection and session management
│   ├── models/
│   │   ├── __init__.py
│   │   └── task.py          # SQLModel Task entity
│   ├── schemas/
│   │   ├── __init__.py
│   │   └── task.py          # Pydantic request/response schemas
│   ├── api/
│   │   ├── __init__.py
│   │   ├── deps.py          # Dependency injection (JWT verification)
│   │   └── routes/
│   │       ├── __init__.py
│   │       └── tasks.py     # Task CRUD endpoints
│   └── core/
│       ├── __init__.py
│       └── security.py      # JWT verification utilities
├── tests/
│   ├── __init__.py
│   ├── conftest.py          # Pytest fixtures
│   ├── test_auth.py         # Authentication tests
│   ├── test_tasks.py        # Task CRUD tests
│   └── test_isolation.py    # Data isolation tests
├── requirements.txt         # Python dependencies
├── .env.example             # Environment variable template
└── README.md                # Backend setup instructions
```

**Structure Decision**: Web application structure with backend focus. The backend/ directory contains the FastAPI application organized by layers: models (database entities), schemas (API contracts), api/routes (endpoints), and core (shared utilities). This structure supports clear separation of concerns and aligns with FastAPI best practices.

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

No violations detected. All constitution principles are satisfied by the planned implementation.

---

## Phase 0: Research & Technology Decisions

### Research Tasks

1. **FastAPI JWT Authentication Patterns**
   - Research: Best practices for JWT verification in FastAPI
   - Research: Dependency injection patterns for user context
   - Research: Error handling for authentication failures

2. **SQLModel with Neon PostgreSQL**
   - Research: Connection pooling strategies for Neon serverless
   - Research: Async database operations with SQLModel
   - Research: Migration strategies (Alembic integration)

3. **Data Isolation Patterns**
   - Research: Query-level filtering patterns in SQLModel
   - Research: Preventing N+1 queries with user-scoped data
   - Research: Index optimization for user_id columns

4. **API Error Handling**
   - Research: Consistent error response formats in FastAPI
   - Research: HTTP status code conventions for REST APIs
   - Research: Validation error handling with Pydantic v2

### Research Output Location

All research findings will be consolidated in `specs/003-backend-task-api/research.md` with the following structure:
- Decision: [what was chosen]
- Rationale: [why chosen]
- Alternatives considered: [what else evaluated]
- Implementation guidance: [how to apply]

---

## Phase 1: Design & Contracts

### Data Model Design

**Output**: `specs/003-backend-task-api/data-model.md`

**Task Entity** (from spec.md Key Entities):
- `id`: Unique identifier (UUID or auto-increment integer)
- `user_id`: Foreign key to user (from authentication system)
- `title`: String, required, max 200 characters
- `description`: String, optional, max 2000 characters
- `is_completed`: Boolean, default False
- `created_at`: Timestamp, auto-generated
- `updated_at`: Timestamp, auto-updated

**Relationships**:
- Task belongs to User (many-to-one)
- User has many Tasks (one-to-many)

**Constraints**:
- `user_id` NOT NULL
- Index on `user_id` for query performance
- Unique constraint on `id`

### API Contract Design

**Output**: `specs/003-backend-task-api/contracts/task-api.yaml`

**Endpoints** (from spec.md API scope and functional requirements):

1. **GET /api/tasks** - List all tasks for authenticated user
   - Auth: Required (JWT)
   - Response: 200 (array of tasks), 401 (unauthorized)
   - Query params: None (filtered by JWT user_id)

2. **POST /api/tasks** - Create new task
   - Auth: Required (JWT)
   - Request body: `{ title: string, description?: string, is_completed?: boolean }`
   - Response: 201 (created task), 400 (validation error), 401 (unauthorized)
   - Auto-assigns user_id from JWT

3. **GET /api/tasks/{id}** - Get single task by ID
   - Auth: Required (JWT)
   - Response: 200 (task), 401 (unauthorized), 403 (not owner), 404 (not found)
   - Ownership verification required

4. **PUT /api/tasks/{id}** - Update task
   - Auth: Required (JWT)
   - Request body: `{ title?: string, description?: string, is_completed?: boolean }`
   - Response: 200 (updated task), 400 (validation error), 401 (unauthorized), 403 (not owner), 404 (not found)
   - Ownership verification required

5. **DELETE /api/tasks/{id}** - Delete task
   - Auth: Required (JWT)
   - Response: 204 (no content), 401 (unauthorized), 403 (not owner), 404 (not found)
   - Ownership verification required

**Note**: The original spec mentioned `/api/{user_id}/tasks` pattern, but per constitution Principle II (Security-First), user_id MUST be derived from JWT, not URL parameters. The API will use `/api/tasks` and filter by the authenticated user's ID extracted from the token.

### Quickstart Guide

**Output**: `specs/003-backend-task-api/quickstart.md`

Will include:
- Environment setup (Python 3.11+, virtual environment)
- Dependency installation (requirements.txt)
- Environment variables (.env configuration)
- Database connection setup (Neon PostgreSQL)
- Running the development server
- Testing the API (curl examples with JWT)
- Running tests (pytest)

---

## Phase 2: Task Generation

**Note**: Phase 2 (task generation) is handled by the `/sp.tasks` command, NOT by `/sp.plan`.

The `/sp.tasks` command will generate `specs/003-backend-task-api/tasks.md` with actionable implementation tasks organized by user story priority (P1 → P2 → P3).

---

## Architectural Decisions

### Decision 1: JWT Verification Strategy

**Context**: Need to verify JWT tokens on every request and extract user identity.

**Decision**: Use FastAPI dependency injection with a `get_current_user` dependency that:
1. Extracts JWT from Authorization header (Bearer token)
2. Verifies signature using BETTER_AUTH_SECRET
3. Validates expiration
4. Extracts user_id from token claims
5. Returns user_id to route handlers

**Rationale**:
- Dependency injection is idiomatic FastAPI pattern
- Centralizes authentication logic (DRY principle)
- Automatically handles 401 responses for invalid tokens
- Makes user_id available to all protected routes

**Alternatives Considered**:
- Middleware: More complex, harder to test individual routes
- Manual verification in each route: Violates DRY, error-prone

### Decision 2: Database Connection Pooling

**Context**: Neon Serverless PostgreSQL requires efficient connection management.

**Decision**: Use SQLModel with async SQLAlchemy engine and connection pooling configured for serverless:
- Pool size: 5-10 connections
- Max overflow: 10
- Pool recycle: 3600 seconds (1 hour)
- Pool pre-ping: True (verify connections before use)

**Rationale**:
- Neon serverless benefits from connection reuse
- Pre-ping prevents stale connection errors
- Pool recycle handles Neon's connection lifecycle
- Async operations improve concurrency

**Alternatives Considered**:
- No pooling: Poor performance, connection overhead
- Large pool: Wastes resources in serverless environment

### Decision 3: User ID Source of Truth

**Context**: Original spec mentioned `/api/{user_id}/tasks` pattern, but constitution requires JWT-only user identity.

**Decision**: Remove user_id from URL paths. Use `/api/tasks` and filter by JWT-extracted user_id.

**Rationale**:
- Constitution Principle II: "User identity MUST be derived only from JWT, never from client input"
- Prevents user impersonation attacks
- Simplifies API contract (no user_id parameter validation)
- Aligns with security-first architecture

**Alternatives Considered**:
- URL parameter with verification: Redundant, confusing, potential security risk if verification fails

### Decision 4: Error Response Format

**Context**: Need consistent error responses across all endpoints.

**Decision**: Use standardized error response format:
```json
{
  "detail": "Human-readable error message",
  "error_code": "MACHINE_READABLE_CODE",
  "status_code": 401
}
```

**Rationale**:
- FastAPI's HTTPException provides this structure by default
- Consistent format aids frontend error handling
- Machine-readable codes enable error categorization
- Aligns with REST API best practices

**Alternatives Considered**:
- Custom error classes: Over-engineering for this scope
- Plain string errors: Harder for frontend to parse

---

## Implementation Sequence

**Note**: Detailed tasks will be generated by `/sp.tasks` command.

**High-Level Sequence** (by user story priority):

1. **Foundation** (Prerequisites for P1):
   - Setup FastAPI project structure
   - Configure database connection to Neon
   - Implement JWT verification dependency
   - Create Task SQLModel entity

2. **P1 User Stories** (View & Create):
   - Implement GET /api/tasks (list tasks)
   - Implement POST /api/tasks (create task)
   - Test data isolation with multiple users

3. **P2 User Stories** (View Details & Update):
   - Implement GET /api/tasks/{id} (get single task)
   - Implement PUT /api/tasks/{id} (update task)
   - Test ownership verification

4. **P3 User Stories** (Delete):
   - Implement DELETE /api/tasks/{id} (delete task)
   - Test complete CRUD lifecycle

5. **Hardening**:
   - Add comprehensive error handling
   - Validate all edge cases from spec
   - Performance testing (concurrent users)
   - Security audit (data isolation verification)

---

## Testing Strategy

### Test Categories

1. **Authentication Tests** (`tests/test_auth.py`):
   - Valid JWT accepted
   - Invalid JWT rejected (401)
   - Expired JWT rejected (401)
   - Missing JWT rejected (401)
   - Malformed JWT rejected (401)

2. **CRUD Operation Tests** (`tests/test_tasks.py`):
   - Create task with valid data (201)
   - Create task with missing title (400)
   - List tasks returns only user's tasks (200)
   - Get task by ID returns correct task (200)
   - Update task modifies fields (200)
   - Toggle completion status (200)
   - Delete task removes it (204)

3. **Data Isolation Tests** (`tests/test_isolation.py`):
   - User A cannot access User B's tasks (403)
   - User A cannot update User B's tasks (403)
   - User A cannot delete User B's tasks (403)
   - Task list filtered by user_id
   - Concurrent operations don't leak data

4. **Edge Case Tests**:
   - Non-existent task ID (404)
   - Extremely long title/description (400)
   - Concurrent updates to same task
   - Database connection failure handling

### Test Data

- Use pytest fixtures for test users and JWT tokens
- Create isolated test database or use transactions with rollback
- Mock JWT verification for unit tests
- Use real JWT for integration tests

---

## Deployment Considerations

### Environment Variables

Required environment variables (documented in .env.example):
- `DATABASE_URL`: Neon PostgreSQL connection string
- `BETTER_AUTH_SECRET`: JWT verification secret (shared with auth service)
- `JWT_ALGORITHM`: JWT signing algorithm (default: HS256)
- `ENVIRONMENT`: dev/staging/production

### Database Migrations

- Use Alembic for schema migrations if needed
- Initial migration creates tasks table with indexes
- Migrations must preserve data isolation constraints

### Monitoring & Observability

- Log all authentication failures (security monitoring)
- Log database connection errors
- Track API response times (performance monitoring)
- Monitor data isolation violations (should be zero)

---

## Success Validation

### Acceptance Criteria Mapping

Each success criterion from spec.md will be validated:

- **SC-001** (Task creation <2s): Performance test with timer
- **SC-002** (List 1000 tasks <3s): Load test with large dataset
- **SC-003** (100% ownership enforcement): Data isolation test suite
- **SC-004** (99.9% data consistency): Integration tests with database verification
- **SC-005** (100% auth rejection): Authentication test suite
- **SC-006** (Toggle <1s): Performance test for update operations
- **SC-007** (100 concurrent users): Load test with concurrent requests
- **SC-008** (Consistent responses): Contract validation tests

### Definition of Done

- [ ] All 5 REST endpoints implemented and tested
- [ ] JWT authentication enforced on all endpoints
- [ ] Data isolation verified with multi-user tests
- [ ] All functional requirements (FR-001 to FR-015) satisfied
- [ ] All success criteria (SC-001 to SC-008) validated
- [ ] API contracts documented in OpenAPI format
- [ ] Quickstart guide enables new developers to run the API
- [ ] No constitution violations
- [ ] All tests passing (>90% coverage)

---

## Next Steps

1. **Execute Phase 0**: Generate research.md with technology best practices
2. **Execute Phase 1**: Generate data-model.md, contracts/, and quickstart.md
3. **Update Agent Context**: Run agent context update script
4. **Re-validate Constitution**: Confirm no violations after design phase
5. **Proceed to /sp.tasks**: Generate actionable implementation tasks

**Command to continue**: `/sp.tasks` (after this plan is approved)
