---
id: 001
title: Create Phase II Todo App Constitution
stage: constitution
date: 2026-02-08
surface: agent
model: claude-sonnet-4-5-20250929
feature: none
branch: 001-task-management-app
user: syedaali258
command: /sp.constitution
labels: ["constitution", "governance", "principles", "security", "spec-driven", "multi-user", "authentication"]
links:
  spec: null
  ticket: null
  adr: null
  pr: null
files:
  - .specify/memory/constitution.md
tests:
  - none
---

## Prompt

/sp.constitution

Project: Phase II – Multi-User Todo Full-Stack Web Application (AI-Spec Driven)

Core principles:
- Spec-driven development (no manual coding, all output generated via prompts)
- Security-first architecture (authentication, authorization, data isolation)
- Deterministic and reproducible builds using Claude Code + Spec-Kit Plus
- Clear separation of concerns (Auth, Backend, Frontend)
- Production-ready engineering standards

Key standards:
- All functionality must be derived strictly from written specs
- No undocumented behavior or implicit logic allowed
- Authentication must be enforced on every API endpoint
- User data isolation is mandatory (no cross-user access)
- JWT-based auth verification must be stateless and deterministic
- REST APIs must follow standard HTTP semantics and status codes
- Frontend must consume backend only via authenticated API calls

Technology constraints:
- Frontend: Next.js 16+ (App Router only)
- Backend: Python FastAPI
- ORM: SQLModel
- Database: Neon Serverless PostgreSQL
- Authentication: Better Auth (JWT-based)
- Spec tooling: Claude Code + Spec-Kit Plus only

Security constraints:
- All API routes require a valid JWT token
- JWT must be verified using shared secret (BETTER_AUTH_SECRET)
- Tokens must expire and be validated on every request
- User identity must be derived only from JWT, not client input
- Task ownership must be enforced at query level
- Unauthorized requests must return 401 consistently

Data & API constraints:
- Persistent storage required for all tasks
- RESTful endpoints must exactly match defined API contract
- CRUD operations must only affect authenticated user's data
- No hardcoded user IDs or mock authentication
- Database schema must support multi-user isolation

Development workflow constraints:
- Follow Agentic Dev Stack strictly

## Response snapshot

Created comprehensive project constitution (v1.0.0) for Phase II Multi-User Todo Full-Stack Web Application with 6 core non-negotiable principles:

1. **Spec-Driven Development**: All functionality derived from written specs, no manual coding
2. **Security-First Architecture**: JWT authentication enforced on all endpoints, mandatory user data isolation
3. **Technology Stack Compliance**: Next.js 16+ (App Router), FastAPI, SQLModel, Neon PostgreSQL, Better Auth
4. **API Contract Enforcement**: RESTful endpoints with standard HTTP semantics, documented contracts
5. **Data Isolation & Multi-Tenancy**: Query-level user data isolation, no cross-user access
6. **Deterministic & Reproducible Builds**: Agentic Dev Stack workflow with PHR/ADR documentation

Added sections:
- **Technology Constraints**: Detailed requirements for frontend (Next.js/TypeScript/Tailwind), backend (FastAPI/SQLModel), database (Neon), and authentication (Better Auth JWT)
- **Development Workflow**: Mandatory 5-phase workflow (Specify → Plan → Tasks → Implement → Commit/PR) with PHR and ADR guidelines
- **Governance**: Amendment process, versioning policy (semantic versioning), compliance review, and enforcement mechanisms

Template alignment verified:
- spec-template.md: User stories with priorities support spec-driven approach
- plan-template.md: Constitution check section present
- tasks-template.md: Organized by user stories for incremental delivery

## Outcome

- ✅ Impact: Established foundational governance for entire project, defining mandatory principles for security, spec-driven development, and technology compliance
- 🧪 Tests: N/A (governance document)
- 📁 Files: Created .specify/memory/constitution.md (v1.0.0)
- 🔁 Next prompts: Ready to proceed with /sp.specify to define first feature specification following constitution principles
- 🧠 Reflection: Constitution provides clear guardrails for multi-user security (JWT auth, data isolation), enforces spec-driven workflow for hackathon evaluation, and mandates technology stack compliance

## Evaluation notes (flywheel)

- Failure modes observed: None - initial constitution creation
- Graders run and results (PASS/FAIL): N/A (no automated graders for constitution)
- Prompt variant (if applicable): N/A (initial creation)
- Next experiment (smallest change to try): Apply constitution principles in first feature spec to validate clarity and completeness of governance rules
