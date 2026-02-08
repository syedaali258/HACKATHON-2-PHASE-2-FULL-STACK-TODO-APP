# Implementation Plan: Frontend Application (Next.js App Router)

**Branch**: `004-nextjs-frontend` | **Date**: 2026-02-08 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/004-nextjs-frontend/spec.md`

**Note**: This template is filled in by the `/sp.plan` command. See `.specify/templates/commands/plan.md` for the execution workflow.

## Summary

Build a modern, responsive, auth-aware web interface using Next.js 16+ App Router that consumes the JWT-protected FastAPI backend. The frontend provides user authentication (signup/signin), protected task dashboard, and complete CRUD operations for task management with strict user data isolation enforced at both UI and API client levels.

## Technical Context

**Language/Version**: TypeScript 5.x with Next.js 16+ (App Router)
**Primary Dependencies**: Next.js 16+, React 19+, Better Auth (client), Tailwind CSS 3.x, axios or native fetch for API client
**Storage**: N/A (frontend consumes backend API; JWT tokens stored in httpOnly cookies or localStorage)
**Testing**: NEEDS CLARIFICATION (not specified in requirements - manual testing assumed)
**Target Platform**: Modern web browsers (Chrome, Firefox, Safari, Edge - latest 2 versions)
**Project Type**: Web (frontend application)
**Performance Goals**: Task creation visible within 3 seconds, user feedback within 1 second, signup completion under 2 minutes, signin completion under 30 seconds
**Constraints**: Mobile-first responsive design (minimum 320px width), all features accessible on mobile/tablet/desktop, client-side form validation, no direct database access
**Scale/Scope**: 8 user stories (3 P1, 3 P2, 2 P3), 20 functional requirements, auth + CRUD operations for task management

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

### I. Spec-Driven Development ✅ PASS
- Feature specification exists at `specs/004-nextjs-frontend/spec.md`
- All 8 user stories documented with priorities (P1-P3)
- 20 functional requirements defined
- Following workflow: Spec → Plan → Tasks → Implementation
- No manual coding permitted

### II. Security-First Architecture ✅ PASS
- JWT authentication required for all protected routes (FR-003, FR-004)
- Tokens stored securely (httpOnly cookies or localStorage with proper handling)
- User identity derived from JWT token only
- Unauthenticated users redirected to signin (FR-004)
- UI displays only authenticated user's tasks (FR-006)
- Logout functionality clears authentication (FR-013)

### III. Technology Stack Compliance ✅ PASS
- Next.js 16+ with App Router (as specified)
- Better Auth for authentication (as specified)
- TypeScript for type safety
- Tailwind CSS for styling
- No framework substitutions

### IV. API Contract Enforcement ✅ PASS
- Frontend consumes backend exclusively via authenticated API calls
- No direct database access from frontend
- JWT token attached to all API requests (FR-003)
- Proper HTTP status code handling (FR-015, FR-018)
- API contracts will be documented in contracts/ directory

### V. Data Isolation & Multi-Tenancy ✅ PASS
- UI displays only tasks belonging to authenticated user (FR-006)
- User isolation enforced at API client level (JWT token per user)
- No cross-user data access possible from frontend
- Backend enforces data isolation at database level

### VI. Deterministic & Reproducible Builds ✅ PASS
- Following Agentic Dev Stack workflow strictly
- PHR will be created for this planning phase
- All changes traceable to spec requirements
- No manual file edits outside Claude Code workflow

**Overall Status**: ✅ ALL GATES PASSED - Proceeding to Phase 0 Research

## Project Structure

### Documentation (this feature)

```text
specs/004-nextjs-frontend/
├── plan.md              # This file (/sp.plan command output)
├── research.md          # Phase 0 output (/sp.plan command)
├── data-model.md        # Phase 1 output (/sp.plan command)
├── quickstart.md        # Phase 1 output (/sp.plan command)
├── contracts/           # Phase 1 output (/sp.plan command)
│   └── frontend-routes.md  # Route definitions and API client contracts
└── tasks.md             # Phase 2 output (/sp.tasks command - NOT created by /sp.plan)
```

### Source Code (repository root)

```text
frontend/
├── app/
│   ├── (auth)/              # Auth route group (public)
│   │   ├── signin/
│   │   │   └── page.tsx     # Sign in page
│   │   └── signup/
│   │       └── page.tsx     # Sign up page
│   ├── (protected)/         # Protected route group (requires auth)
│   │   └── dashboard/
│   │       └── page.tsx     # Task dashboard
│   ├── layout.tsx           # Root layout
│   ├── page.tsx             # Landing/redirect page
│   └── globals.css          # Global styles
├── components/
│   ├── auth/
│   │   ├── SignInForm.tsx   # Sign in form component
│   │   └── SignUpForm.tsx   # Sign up form component
│   ├── tasks/
│   │   ├── TaskList.tsx     # Task list display
│   │   ├── TaskItem.tsx     # Individual task item
│   │   ├── CreateTaskForm.tsx  # Task creation form
│   │   ├── EditTaskModal.tsx   # Task edit modal
│   │   └── DeleteConfirmDialog.tsx  # Delete confirmation
│   └── ui/
│       ├── Button.tsx       # Reusable button component
│       ├── Input.tsx        # Reusable input component
│       └── LoadingSpinner.tsx  # Loading indicator
├── lib/
│   ├── api/
│   │   ├── client.ts        # API client with JWT attachment
│   │   └── tasks.ts         # Task API methods
│   ├── auth/
│   │   ├── better-auth.ts   # Better Auth configuration
│   │   └── session.ts       # Session management utilities
│   └── utils/
│       └── validation.ts    # Form validation helpers
├── middleware.ts            # Route protection middleware
├── types/
│   └── task.ts              # TypeScript types for Task entity
├── .env.example             # Environment variables template
├── .env.local               # Local environment variables (gitignored)
├── next.config.js           # Next.js configuration
├── tailwind.config.js       # Tailwind CSS configuration
├── tsconfig.json            # TypeScript configuration
└── package.json             # Dependencies
```

**Structure Decision**: Web application structure with `frontend/` directory. Using Next.js 16+ App Router with route groups for auth (public) and protected routes. Components organized by feature (auth, tasks, ui). API client layer in `lib/api/` with automatic JWT attachment. Better Auth integration in `lib/auth/`. Middleware for route protection at the root level.

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

No violations - all constitution principles satisfied.

---

## Architectural Decisions

### Decision 1: JWT Token Storage Strategy

**Context**: Need to securely store JWT tokens on the frontend for authenticated API requests.

**Options Considered**:
1. **httpOnly cookies** (chosen)
2. localStorage
3. sessionStorage
4. Memory only

**Decision**: Use httpOnly cookies as primary storage mechanism

**Rationale**:
- **Security**: httpOnly cookies cannot be accessed by JavaScript, providing XSS protection
- **Automatic transmission**: Cookies sent automatically with every request to same domain
- **Better Auth support**: Built-in support for httpOnly cookie mode
- **Production-ready**: Secure flag ensures HTTPS-only transmission in production
- **CSRF protection**: SameSite=lax attribute prevents cross-site request forgery

**Trade-offs**:
- Requires backend and frontend on same domain (or proper CORS configuration)
- Slightly more complex setup than localStorage
- Cannot be accessed by JavaScript (but this is a security feature, not a limitation)

**Alternatives Rejected**:
- localStorage: Vulnerable to XSS attacks, easier to implement but less secure
- sessionStorage: Lost on tab close, poor UX for persistent sessions
- Memory only: Lost on page refresh, terrible UX

---

### Decision 2: Route Protection Mechanism

**Context**: Need to prevent unauthenticated users from accessing protected pages.

**Options Considered**:
1. **Next.js middleware** (chosen)
2. Component-level checks with HOCs
3. Server Component checks only
4. Client-side checks only

**Decision**: Use Next.js middleware.ts for centralized route protection

**Rationale**:
- **Edge runtime**: Runs before page rendering, fast redirects
- **Centralized logic**: Single source of truth for authentication checks (DRY)
- **Pattern matching**: Supports route groups and wildcards
- **No flash of content**: Redirects before any component renders
- **Works with both**: Server Components and Client Components

**Trade-offs**:
- Middleware runs on every request (but very fast on edge)
- Cannot access React context or hooks
- Limited to cookie-based authentication (but that's our choice anyway)

**Alternatives Rejected**:
- Component-level checks: Duplicated logic, flash of wrong content before redirect
- Server Component checks only: Doesn't prevent client-side navigation
- Client-side checks only: Not secure, can be bypassed

---

### Decision 3: API Client Implementation

**Context**: Need to make authenticated API calls to the FastAPI backend.

**Options Considered**:
1. **Axios with interceptors** (chosen)
2. Native fetch with manual token attachment
3. SWR/React Query with custom fetcher
4. GraphQL client (Apollo/urql)

**Decision**: Use axios with request/response interceptors

**Rationale**:
- **Automatic JWT attachment**: Request interceptor adds token to all requests
- **Centralized error handling**: Response interceptor handles 401 errors globally
- **Type safety**: Works well with TypeScript
- **Familiar API**: Similar to fetch but with more features
- **Interceptors**: Powerful middleware pattern for cross-cutting concerns

**Trade-offs**:
- Adds ~13KB to bundle size (but worth it for features)
- Another dependency to maintain
- Slightly different API than native fetch

**Alternatives Rejected**:
- Native fetch: More boilerplate, no interceptors, manual token attachment everywhere
- SWR/React Query: Overkill for simple CRUD, adds complexity, caching not needed
- GraphQL: Not aligned with REST API requirement, backend is REST

---

### Decision 4: Form Validation Strategy

**Context**: Need to validate user input before sending to API.

**Options Considered**:
1. **Zod + react-hook-form** (chosen)
2. Manual validation with useState
3. Yup + Formik
4. HTML5 validation only

**Decision**: Use Zod for schema validation with react-hook-form for form state

**Rationale**:
- **Type safety**: Zod is TypeScript-first, provides type inference
- **Reusable schemas**: Define once, use everywhere (client and server)
- **Integration**: @hookform/resolvers provides seamless integration
- **DRY principle**: Types inferred from schemas, no duplication
- **Validation rules match backend**: Ensures consistency

**Trade-offs**:
- Two dependencies instead of one
- Learning curve for Zod syntax
- Slightly larger bundle than manual validation

**Alternatives Rejected**:
- Manual validation: Error-prone, not type-safe, lots of boilerplate
- Yup: Less TypeScript-friendly than Zod, older library
- HTML5 validation only: Not sufficient for complex rules, poor UX

---

### Decision 5: State Management Approach

**Context**: Need to manage task list state and UI state.

**Options Considered**:
1. **React hooks (useState/useEffect)** (chosen)
2. Redux Toolkit
3. Zustand
4. React Query for server state

**Decision**: Use React hooks for component-level state management

**Rationale**:
- **Simplicity**: Simple CRUD operations don't need complex state management
- **Built-in**: No additional dependencies
- **Sufficient**: Component-level state adequate for this scope
- **Optimistic updates**: Easy to implement with useState
- **No global state needed**: Each page manages its own state

**Trade-offs**:
- State not shared across components (but we don't need that)
- No built-in caching (but we don't need that either)
- Manual optimistic updates (but gives us full control)

**Alternatives Rejected**:
- Redux: Massive overkill for simple CRUD, adds complexity and boilerplate
- Zustand: Unnecessary for component-level state, no global state needed
- React Query: Adds complexity, caching not required for this scope

---

### Decision 6: Styling Approach

**Context**: Need to style components with responsive design.

**Options Considered**:
1. **Tailwind CSS utility classes** (chosen)
2. CSS Modules
3. Styled Components (CSS-in-JS)
4. Plain CSS

**Decision**: Use Tailwind CSS with mobile-first utility classes

**Rationale**:
- **Mobile-first**: Aligns with requirement (minimum 320px width)
- **Utility classes**: Reduces CSS bundle size via tree-shaking
- **Design system**: Consistent spacing, colors, typography out of the box
- **No runtime overhead**: Unlike CSS-in-JS solutions
- **Developer experience**: Fast iteration, no context switching

**Trade-offs**:
- Verbose class names in JSX
- Learning curve for utility-first approach
- Requires Tailwind configuration

**Alternatives Rejected**:
- CSS Modules: More boilerplate, harder to maintain consistency
- Styled Components: Runtime overhead, not aligned with Next.js best practices
- Plain CSS: No design system, harder to maintain, larger bundle

---

### Decision 7: Component Architecture

**Context**: Need to organize React components for maintainability.

**Options Considered**:
1. **Feature-based organization** (chosen)
2. Flat components directory
3. Atomic design (atoms/molecules/organisms)
4. Page-based organization

**Decision**: Organize components by feature (auth, tasks, ui)

**Rationale**:
- **Colocation**: Related components grouped together
- **Scalability**: Easy to add new features without reorganizing
- **Clear boundaries**: Each feature is self-contained
- **Reusability**: UI components separated for cross-feature use

**Structure**:
```
components/
├── auth/        # Authentication-specific components
├── tasks/       # Task management components
└── ui/          # Reusable UI components (buttons, inputs, etc.)
```

**Trade-offs**:
- Some components might fit in multiple categories
- Need to decide where shared components go

**Alternatives Rejected**:
- Flat directory: Becomes unwieldy as project grows
- Atomic design: Too complex for this scope, over-engineering
- Page-based: Doesn't promote reusability across pages

---

### Decision 8: Error Handling Strategy

**Context**: Need to handle errors gracefully across the application.

**Options Considered**:
1. **Next.js error boundaries (error.tsx) + API client interceptor** (chosen)
2. Try-catch in every component
3. Global error boundary only
4. Toast notifications only

**Decision**: Use Next.js error.tsx files with API client error interceptor

**Rationale**:
- **Built-in support**: Next.js App Router provides error.tsx pattern
- **Granular recovery**: Each route can have its own error handling
- **Automatic**: Catches unhandled errors during rendering
- **Centralized API errors**: Interceptor handles 401 globally
- **User-friendly**: Provides recovery options (retry button)

**Trade-offs**:
- Need to create error.tsx for each route that needs custom handling
- Error boundaries don't catch errors in event handlers (need try-catch there)

**Alternatives Rejected**:
- Try-catch everywhere: Duplicated logic, easy to miss
- Global error boundary only: Less granular, harder to recover
- Toast notifications only: No fallback UI for critical errors

---

## Post-Design Constitution Check

*Re-evaluation after Phase 1 design artifacts completed*

### I. Spec-Driven Development ✅ PASS
- All design decisions traced to spec requirements
- research.md resolves all technical unknowns
- data-model.md defines types matching backend contracts
- contracts/ documents all routes and API interactions
- quickstart.md provides reproducible setup instructions

### II. Security-First Architecture ✅ PASS
- httpOnly cookies chosen for XSS protection (Decision 1)
- Middleware enforces authentication before page load (Decision 2)
- API client automatically attaches JWT to all requests (Decision 3)
- 401 errors trigger automatic sign out and redirect
- No sensitive data stored in frontend state

### III. Technology Stack Compliance ✅ PASS
- Next.js 16+ App Router confirmed in all design artifacts
- Better Auth integration documented in research.md
- TypeScript types defined in data-model.md
- Tailwind CSS chosen for styling (Decision 6)
- All dependencies align with approved stack

### IV. API Contract Enforcement ✅ PASS
- API client contract documented in contracts/api-client.md
- All endpoints match backend API exactly
- Type definitions match backend Pydantic schemas
- Error handling covers all HTTP status codes
- No direct database access from frontend

### V. Data Isolation & Multi-Tenancy ✅ PASS
- JWT token identifies user on every request
- Backend enforces data isolation at query level
- Frontend trusts backend for data filtering
- No cross-user data access possible
- User ID never sent from client (only from JWT)

### VI. Deterministic & Reproducible Builds ✅ PASS
- All design artifacts created via /sp.plan workflow
- Architectural decisions documented with rationale
- quickstart.md provides step-by-step setup
- Environment variables templated in .env.example
- No manual configuration required

**Overall Status**: ✅ ALL GATES PASSED - Ready for /sp.tasks phase

---

## Summary

**Planning Complete**: All Phase 0 and Phase 1 artifacts generated successfully.

**Artifacts Created**:
1. ✅ `plan.md` - This file (implementation plan with architectural decisions)
2. ✅ `research.md` - Technical research resolving all unknowns (9 research areas)
3. ✅ `data-model.md` - TypeScript types and validation schemas
4. ✅ `contracts/frontend-routes.md` - Route definitions and navigation flows
5. ✅ `contracts/api-client.md` - API client interface and error handling
6. ✅ `quickstart.md` - Setup and development instructions

**Key Decisions**:
- httpOnly cookies for secure JWT storage
- Next.js middleware for centralized route protection
- Axios with interceptors for API client
- Zod + react-hook-form for type-safe validation
- React hooks for simple state management
- Tailwind CSS for mobile-first responsive design
- Feature-based component organization
- Next.js error boundaries for error handling

**Technology Stack Confirmed**:
- Next.js 16+ (App Router)
- React 19+
- TypeScript 5.x
- Better Auth (client)
- Tailwind CSS 3.x
- Axios 1.6+
- Zod 3.22+
- react-hook-form 7.49+

**Next Steps**:
1. Run `/sp.tasks` to generate actionable task list
2. Tasks will be organized by user story (US1-US8)
3. Implementation will follow priority order (P1 → P2 → P3)
4. Each user story independently testable

**Dependencies**:
- Backend API must be running at http://localhost:8000
- Better Auth must be configured and operational
- Neon PostgreSQL database must be accessible
- CORS must allow requests from http://localhost:3000

**No Blockers**: All technical unknowns resolved, ready to proceed to task generation.
