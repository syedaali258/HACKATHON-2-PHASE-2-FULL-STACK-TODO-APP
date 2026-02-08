# Research: Multi-User Todo Full-Stack Web Application

**Feature**: 002-multi-user-todo-app
**Date**: 2026-02-08
**Purpose**: Resolve technical unknowns and document architectural decisions

## Research Questions

### 1. Testing Strategy

**Question**: What testing approach should be used for this project?

**Context**: The feature specification does not explicitly require tests. The tasks template indicates tests are optional unless explicitly requested. However, the constitution emphasizes reproducibility and quality.

**Decision**: Manual testing and validation only (no automated test suite)

**Rationale**:
- The feature specification does not include testing requirements in the functional requirements (FR-001 through FR-020)
- Success criteria (SC-001 through SC-010) focus on functional outcomes and user experience, not test coverage
- The constitution's spec-driven principle states "All functionality MUST be derived strictly from written specifications" - since tests are not specified, they should not be implemented
- The tasks template explicitly states: "Tests are OPTIONAL - only include them if explicitly requested in the feature specification"
- Manual validation against acceptance scenarios in the spec is sufficient for hackathon evaluation
- Focus should be on demonstrating the agentic development workflow, not test automation

**Alternatives Considered**:
1. **Full test suite (unit + integration + E2E)**: Rejected because tests are not specified in requirements and would violate spec-driven principle by adding undocumented functionality
2. **Minimal unit tests only**: Rejected for same reason - not specified in requirements
3. **Contract tests only**: Rejected - while API contracts will be documented, automated contract tests are not required by spec

**Implementation Impact**:
- No test files or test frameworks needed
- Validation will be manual against acceptance scenarios
- Focus development effort on core functionality specified in requirements

---

## Technology Decisions

### 2. Better Auth Configuration

**Question**: How should Better Auth be configured for JWT token issuance in Next.js?

**Decision**: Use Better Auth with JWT plugin, configured for stateless authentication

**Rationale**:
- Constitution mandates Better Auth with JWT-based authentication
- Better Auth provides built-in JWT plugin for token issuance
- Stateless JWT tokens align with serverless architecture (Neon PostgreSQL)
- Shared secret (BETTER_AUTH_SECRET) enables backend verification without database lookups

**Configuration Approach**:
- Install `better-auth` package in Next.js frontend
- Enable JWT plugin in Better Auth configuration
- Configure token expiration (1 hour access token recommended)
- Store tokens in httpOnly cookies for security
- Share BETTER_AUTH_SECRET between frontend and backend via environment variables

**References**:
- Better Auth documentation: JWT plugin configuration
- Constitution: Authentication Requirements section

---

### 3. FastAPI JWT Verification

**Question**: How should FastAPI verify JWT tokens issued by Better Auth?

**Decision**: Use python-jose library with shared secret verification

**Rationale**:
- python-jose is the standard JWT library for Python/FastAPI
- Supports HS256 algorithm (HMAC with SHA-256) for symmetric key verification
- Shared secret approach (BETTER_AUTH_SECRET) enables stateless verification
- No database lookup required for token validation (performance benefit)

**Implementation Approach**:
- Install `python-jose[cryptography]` package
- Create JWT verification dependency in `backend/src/api/deps.py`
- Extract token from Authorization header (Bearer scheme)
- Verify signature using BETTER_AUTH_SECRET
- Decode token to extract user_id claim
- Inject authenticated user_id into route handlers

**Error Handling**:
- Invalid signature → HTTP 401 Unauthorized
- Expired token → HTTP 401 Unauthorized
- Missing token → HTTP 401 Unauthorized
- Malformed token → HTTP 401 Unauthorized

---

### 4. Neon PostgreSQL Connection Management

**Question**: How should the application connect to Neon Serverless PostgreSQL?

**Decision**: Use asyncpg driver with SQLModel, connection pooling enabled

**Rationale**:
- Neon requires PostgreSQL-compatible drivers
- asyncpg is the recommended async driver for Python
- SQLModel supports asyncpg through SQLAlchemy async engine
- Connection pooling reduces latency for serverless cold starts
- Async/await patterns align with FastAPI's async capabilities

**Configuration Approach**:
- Install `asyncpg` and `sqlmodel` packages
- Create async engine with Neon connection string (DATABASE_URL)
- Configure connection pool (min: 1, max: 10 connections)
- Use async session management for database operations
- Enable SSL mode for secure connections

**Connection String Format**:
```
postgresql+asyncpg://user:password@host/database?ssl=require
```

---

### 5. Password Hashing

**Question**: Which password hashing algorithm should be used?

**Decision**: Use passlib with bcrypt algorithm

**Rationale**:
- Constitution assumption: "Password hashing uses industry-standard algorithms (e.g., bcrypt, argon2)"
- bcrypt is widely adopted, well-tested, and secure
- passlib provides a high-level interface for bcrypt
- Configurable work factor for future-proofing against hardware improvements

**Implementation Approach**:
- Install `passlib[bcrypt]` package
- Create password hashing utilities in `backend/src/core/security.py`
- Use bcrypt with work factor of 12 (industry standard)
- Hash passwords before storing in database
- Verify passwords during signin using constant-time comparison

---

### 6. API Endpoint Design

**Question**: What REST API endpoints are needed to support the 5 user stories?

**Decision**: 8 RESTful endpoints following standard HTTP semantics

**Endpoints**:

**Authentication (User Story 1 - P1)**:
- `POST /api/auth/signup` - Create new user account
- `POST /api/auth/signin` - Authenticate user and issue JWT token

**Task Management (User Stories 2-5 - P2-P5)**:
- `GET /api/tasks` - List all tasks for authenticated user (with optional filter: ?status=complete|incomplete|all)
- `POST /api/tasks` - Create new task for authenticated user
- `GET /api/tasks/{task_id}` - Get single task by ID (ownership verified)
- `PUT /api/tasks/{task_id}` - Update task title/description (ownership verified)
- `PATCH /api/tasks/{task_id}/complete` - Toggle task completion status (ownership verified)
- `DELETE /api/tasks/{task_id}` - Delete task (ownership verified)

**Rationale**:
- Follows REST conventions (GET for read, POST for create, PUT for full update, PATCH for partial update, DELETE for remove)
- Separate PATCH endpoint for completion toggle (common operation, deserves dedicated endpoint)
- Query parameter for filtering (GET /api/tasks?status=complete) supports User Story 4 requirement
- All task endpoints require authentication (JWT token in Authorization header)
- All task endpoints enforce ownership at query level (user_id filter)

---

### 7. Frontend Routing Strategy

**Question**: How should Next.js App Router be structured for authentication flows?

**Decision**: Use middleware-based route protection with Better Auth session checking

**Route Structure**:
- `/` - Public landing page
- `/signup` - Public signup page
- `/signin` - Public signin page
- `/tasks` - Protected task list page (requires authentication)

**Protection Approach**:
- Create Next.js middleware to check Better Auth session
- Redirect unauthenticated users from `/tasks` to `/signin`
- Redirect authenticated users from `/signin` or `/signup` to `/tasks`
- Use Better Auth's session management hooks in components

**Rationale**:
- Middleware-based protection is the recommended approach for Next.js App Router
- Centralized authentication logic (DRY principle)
- Better user experience (automatic redirects)
- Aligns with constitution's security-first architecture

---

## Summary

All technical unknowns resolved. Key decisions:
1. **Testing**: Manual validation only (no automated tests per spec-driven principle)
2. **Authentication**: Better Auth with JWT plugin, python-jose for backend verification
3. **Database**: asyncpg + SQLModel with connection pooling for Neon PostgreSQL
4. **Security**: bcrypt password hashing, stateless JWT verification
5. **API Design**: 8 RESTful endpoints with standard HTTP semantics
6. **Frontend**: Middleware-based route protection with Better Auth

Ready to proceed to Phase 1: Design & Contracts.
