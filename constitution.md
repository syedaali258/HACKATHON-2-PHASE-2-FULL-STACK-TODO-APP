<!--
Sync Impact Report:
- Version: 1.0.0 (Initial constitution)
- New constitution created for Phase II Multi-User Todo Full-Stack Web Application
- Principles defined: 6 core principles covering spec-driven development, security, technology compliance, API contracts, data isolation, and reproducibility
- Sections added: Technology Constraints, Development Workflow, Governance
- Templates status:
  ✅ spec-template.md - Aligned (user stories with priorities support spec-driven approach)
  ✅ plan-template.md - Aligned (constitution check section present)
  ✅ tasks-template.md - Aligned (organized by user stories for incremental delivery)
- Follow-up: None required
-->

# Phase II Multi-User Todo Full-Stack Web Application Constitution

## Core Principles

### I. Spec-Driven Development (NON-NEGOTIABLE)

All functionality MUST be derived strictly from written specifications. No manual coding is permitted; all implementation output must be generated via prompts through Claude Code and Spec-Kit Plus workflow.

**Rules**:
- Every feature begins with a specification document (`spec.md`)
- Implementation follows the workflow: Spec → Plan → Tasks → Implementation
- No undocumented behavior or implicit logic is allowed
- All changes must reference the originating spec
- Code that cannot be traced to a spec requirement is prohibited

**Rationale**: Ensures reproducibility, traceability, and alignment with project evaluation criteria (quality of spec-driven process, prompt effectiveness, iteration count).

### II. Security-First Architecture (NON-NEGOTIABLE)

Security is a foundational requirement, not an afterthought. Authentication and authorization MUST be enforced at every layer, with mandatory user data isolation.

**Rules**:
- Authentication MUST be enforced on every API endpoint (no exceptions)
- All API routes require a valid JWT token
- JWT MUST be verified using shared secret (`BETTER_AUTH_SECRET`)
- Tokens MUST expire and be validated on every request
- User identity MUST be derived only from JWT, never from client input
- Task ownership MUST be enforced at the database query level
- Unauthorized requests MUST return HTTP 401 consistently
- No cross-user data access is permitted under any circumstances

**Rationale**: Multi-user applications require strict security boundaries to prevent data leakage and unauthorized access. JWT-based stateless authentication ensures scalability in serverless environments.

### III. Technology Stack Compliance (NON-NEGOTIABLE)

The project MUST use the specified technology stack without substitution. Deviations require explicit documentation and justification.

**Approved Stack**:
- **Frontend**: Next.js 16+ (App Router only, no Pages Router)
- **Backend**: Python FastAPI
- **ORM**: SQLModel
- **Database**: Neon Serverless PostgreSQL
- **Authentication**: Better Auth (JWT-based)
- **Spec Tooling**: Claude Code + Spec-Kit Plus exclusively

**Rules**:
- No alternative frameworks or libraries for core stack components
- All dependencies must be compatible with the approved stack
- Technology choices must be documented in `plan.md`

**Rationale**: Stack consistency ensures predictable behavior, reduces integration issues, and aligns with hackathon evaluation criteria.

### IV. API Contract Enforcement (NON-NEGOTIABLE)

RESTful API endpoints MUST follow standard HTTP semantics and exactly match the defined API contract. Backend and frontend communicate exclusively via authenticated API calls.

**Rules**:
- REST APIs MUST follow standard HTTP semantics and status codes
- API contracts MUST be documented in `specs/<feature>/contracts/`
- RESTful endpoints MUST exactly match the defined API contract
- Frontend MUST consume backend only via authenticated API calls
- No direct database access from frontend
- CRUD operations MUST only affect the authenticated user's data
- Error responses MUST use appropriate HTTP status codes (401, 403, 404, 422, 500)

**Rationale**: Clear API contracts prevent integration issues, enable parallel frontend/backend development, and ensure consistent error handling.

### V. Data Isolation & Multi-Tenancy (NON-NEGOTIABLE)

Every user's data MUST be isolated at the database level. No user can access, modify, or view another user's data.

**Rules**:
- Persistent storage required for all tasks
- Database schema MUST support multi-user isolation (user_id foreign keys)
- All queries MUST filter by authenticated user's ID
- No hardcoded user IDs or mock authentication in production code
- Data models MUST include user ownership fields
- Database migrations MUST preserve data isolation constraints

**Rationale**: Multi-user applications require strict data boundaries. Query-level enforcement prevents accidental data leakage through application bugs.

### VI. Deterministic & Reproducible Builds (NON-NEGOTIABLE)

All development artifacts MUST be reproducible through the Agentic Dev Stack workflow. Manual interventions break reproducibility.

**Rules**:
- Follow Agentic Dev Stack strictly: Spec → Plan → Tasks → Implementation
- All prompts and responses MUST be recorded in Prompt History Records (PHRs)
- Architectural decisions MUST be documented in ADRs when significant
- No manual file edits outside the Claude Code workflow
- All changes MUST be traceable to a specific prompt or command
- Build and deployment processes MUST be scripted and version-controlled

**Rationale**: Reproducibility is a core evaluation criterion for the hackathon. Deterministic builds enable debugging, auditing, and iterative improvement of the prompt-driven process.

## Technology Constraints

### Frontend Requirements
- Next.js 16+ with App Router architecture (no Pages Router)
- Server Components as default, Client Components only when necessary
- TypeScript for type safety
- Tailwind CSS for styling
- Better Auth client integration for authentication state

### Backend Requirements
- Python 3.11+ with FastAPI framework
- SQLModel for ORM and data validation
- Pydantic v2 for request/response models
- JWT verification middleware on all protected routes
- Async/await patterns for database operations
- Neon serverless PostgreSQL connection pooling

### Database Requirements
- Neon Serverless PostgreSQL (no local PostgreSQL)
- SQLModel schema definitions
- Alembic for migrations (if needed)
- User ownership columns on all user-scoped tables
- Indexes on user_id columns for query performance

### Authentication Requirements
- Better Auth configured for JWT token issuance
- Tokens include user ID and expiration claims
- Backend verifies JWT signature using `BETTER_AUTH_SECRET`
- Token expiration enforced (recommended: 1 hour access, 7 days refresh)
- Secure token storage on frontend (httpOnly cookies preferred)

## Development Workflow

### Agentic Dev Stack Workflow (MANDATORY)

1. **Specification Phase** (`/sp.specify`):
   - Write feature specification in `specs/<feature>/spec.md`
   - Define user stories with priorities (P1, P2, P3)
   - Document acceptance criteria and edge cases
   - Identify functional requirements

2. **Planning Phase** (`/sp.plan`):
   - Generate implementation plan in `specs/<feature>/plan.md`
   - Document technical decisions and architecture
   - Create data models in `specs/<feature>/data-model.md`
   - Define API contracts in `specs/<feature>/contracts/`
   - Run constitution check and document any violations

3. **Task Generation Phase** (`/sp.tasks`):
   - Generate actionable tasks in `specs/<feature>/tasks.md`
   - Organize tasks by user story for independent implementation
   - Define dependencies and parallel execution opportunities
   - Include test tasks if explicitly requested

4. **Implementation Phase** (`/sp.implement`):
   - Execute tasks in dependency order
   - Implement one user story at a time (P1 → P2 → P3)
   - Validate each story independently before proceeding
   - Create PHRs for all significant prompts

5. **Commit & PR Phase** (`/sp.git.commit_pr`):
   - Commit completed work with descriptive messages
   - Create pull requests with summary and test plan
   - Reference related specs, tasks, and ADRs

### Prompt History Records (PHRs)

**MANDATORY**: Create a PHR after every user prompt that involves:
- Implementation work (code changes, new features)
- Planning or architecture discussions
- Debugging sessions
- Spec, plan, or task creation

**PHR Routing** (all under `history/prompts/`):
- Constitution-related → `history/prompts/constitution/`
- Feature-specific → `history/prompts/<feature-name>/`
- General → `history/prompts/general/`

### Architectural Decision Records (ADRs)

**When to Create**: Suggest ADR creation when a decision meets ALL criteria:
- **Impact**: Long-term consequences (framework, data model, API, security, platform)
- **Alternatives**: Multiple viable options were considered
- **Scope**: Cross-cutting and influences system design

**Process**: Suggest with "📋 Architectural decision detected: [brief]. Document? Run `/sp.adr <title>`" and wait for user consent. Never auto-create ADRs.

## Governance

### Amendment Process

1. **Proposal**: Document proposed changes with rationale
2. **Impact Analysis**: Identify affected templates, specs, and code
3. **Version Bump**: Follow semantic versioning (MAJOR.MINOR.PATCH)
4. **Propagation**: Update all dependent templates and documentation
5. **Approval**: Requires explicit user consent
6. **Migration**: Update existing specs/plans to comply if needed

### Versioning Policy

- **MAJOR**: Backward-incompatible governance or principle removals/redefinitions
- **MINOR**: New principle/section added or materially expanded guidance
- **PATCH**: Clarifications, wording, typo fixes, non-semantic refinements

### Compliance Review

- All PRs MUST verify compliance with constitution principles
- Constitution violations MUST be documented in `plan.md` Complexity Tracking section with justification
- Unjustified complexity or violations MUST be rejected
- Constitution supersedes all other practices and conventions

### Enforcement

- Automated checks in CI/CD pipeline (where applicable)
- Manual review during PR approval
- Regular audits of specs, plans, and implementation against constitution
- Non-compliance blocks merge until resolved or justified

**Version**: 1.0.0 | **Ratified**: 2026-02-08 | **Last Amended**: 2026-02-08
