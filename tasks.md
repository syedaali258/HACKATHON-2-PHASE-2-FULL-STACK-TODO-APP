# Tasks: Multi-User Todo Full-Stack Web Application

**Input**: Design documents from `/specs/002-multi-user-todo-app/`
**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, contracts/

**Tests**: No automated tests - manual validation only per research.md decision (tests not specified in feature specification)

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

- **Web app**: `backend/src/`, `frontend/src/`
- Paths shown below follow web application structure from plan.md

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and basic structure

- [x] T001 Create backend directory structure (backend/src/models/, backend/src/api/, backend/src/core/, backend/tests/)
- [x] T002 Create frontend directory structure (frontend/src/app/, frontend/src/components/, frontend/src/lib/, frontend/src/types/)
- [x] T003 [P] Initialize Python virtual environment and install backend dependencies (fastapi, uvicorn, sqlmodel, asyncpg, python-jose[cryptography], passlib[bcrypt], python-multipart) in backend/
- [x] T004 [P] Initialize Next.js project with TypeScript and install frontend dependencies (next, react, react-dom, better-auth, tailwindcss) in frontend/
- [x] T005 [P] Create backend/requirements.txt with all Python dependencies
- [x] T006 [P] Create backend/.env.example with template environment variables (BETTER_AUTH_SECRET, DATABASE_URL, BACKEND_PORT, BACKEND_HOST)
- [x] T007 [P] Create frontend/.env.local.example with template environment variables (NEXT_PUBLIC_API_URL, BETTER_AUTH_SECRET)
- [x] T008 [P] Configure Tailwind CSS in frontend/tailwind.config.js and frontend/postcss.config.js
- [x] T009 [P] Create frontend/tsconfig.json with TypeScript configuration for Next.js App Router

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [x] T010 Create environment configuration module in backend/src/core/config.py (load DATABASE_URL, BETTER_AUTH_SECRET, BACKEND_PORT, BACKEND_HOST from environment)
- [x] T011 Create Neon PostgreSQL async database connection in backend/src/core/database.py (async engine with asyncpg, connection pooling, SSL enabled)
- [x] T012 Create security utilities in backend/src/core/security.py (bcrypt password hashing with work factor 12, JWT token verification using python-jose with BETTER_AUTH_SECRET)
- [x] T013 [P] Create User SQLModel schema in backend/src/models/user.py (id UUID, email unique indexed, hashed_password, created_at, updated_at)
- [x] T014 [P] Create Task SQLModel schema in backend/src/models/task.py (id UUID, user_id FK to users.id, title max 200, description max 2000 nullable, is_complete default False, created_at, updated_at, indexes on user_id)
- [x] T015 Create JWT authentication dependency in backend/src/api/deps.py (extract Bearer token from Authorization header, verify signature, decode user_id, return HTTP 401 on failure)
- [x] T016 Create FastAPI application entry point in backend/src/main.py (initialize app, include routers, configure CORS for frontend origin, create database tables on startup)
- [x] T017 [P] Create Better Auth configuration in frontend/src/lib/auth.ts (enable JWT plugin, configure token expiration 1 hour, set BETTER_AUTH_SECRET)
- [x] T018 [P] Create API client wrapper in frontend/src/lib/api.ts (base URL from NEXT_PUBLIC_API_URL, automatically attach JWT token to Authorization header, handle 401 errors)
- [x] T019 [P] Create TypeScript type definitions in frontend/src/types/task.ts (User type, Task type, TaskStatus enum, API response types)
- [x] T020 [P] Create root layout in frontend/src/app/layout.tsx (Better Auth provider, global styles, metadata)

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - User Registration and Authentication (Priority: P1) 🎯 MVP

**Goal**: Users can create accounts and sign in to access their personal todo list with JWT-based authentication

**Independent Test**: Create new account, sign in, receive JWT token, verify token required for API calls

### Implementation for User Story 1

- [x] T021 [P] [US1] Implement POST /api/auth/signup endpoint in backend/src/api/auth.py (validate email format, check email uniqueness, hash password with bcrypt, create user in database, issue JWT token, return user + token, handle 409 conflict for duplicate email)
- [x] T022 [P] [US1] Implement POST /api/auth/signin endpoint in backend/src/api/auth.py (find user by email, verify password with bcrypt constant-time comparison, issue JWT token on success, return user + token, return 401 on invalid credentials without revealing email existence)
- [x] T023 [US1] Register authentication router in backend/src/main.py (include auth router with /api/auth prefix)
- [x] T024 [P] [US1] Create signup page in frontend/src/app/signup/page.tsx (email input with validation, password input min 8 chars, submit form to POST /api/auth/signup, store JWT token, redirect to /tasks on success, display error messages)
- [x] T025 [P] [US1] Create signin page in frontend/src/app/signin/page.tsx (email input, password input, submit form to POST /api/auth/signin, store JWT token, redirect to /tasks on success, display error messages)
- [x] T026 [P] [US1] Create landing page in frontend/src/app/page.tsx (welcome message, links to /signup and /signin, redirect authenticated users to /tasks)
- [x] T027 [US1] Create Next.js middleware in frontend/src/middleware.ts (check Better Auth session, redirect unauthenticated users from /tasks to /signin, redirect authenticated users from /signin or /signup to /tasks)

**Checkpoint**: At this point, User Story 1 should be fully functional and testable independently

---

## Phase 4: User Story 2 - Create and List Tasks (Priority: P2)

**Goal**: Users can create new tasks and view a list of all their tasks with strict data isolation

**Independent Test**: Sign in, create multiple tasks, verify all appear in task list sorted by creation date, verify other users cannot see these tasks

### Implementation for User Story 2

- [ ] T028 [P] [US2] Implement GET /api/tasks endpoint in backend/src/api/tasks.py (require JWT auth via deps.py, query tasks filtered by authenticated user_id, support ?status=all|complete|incomplete query parameter, order by created_at DESC, return task list)
- [ ] T029 [P] [US2] Implement POST /api/tasks endpoint in backend/src/api/tasks.py (require JWT auth, validate title not empty and max 200 chars, validate description max 2000 chars if provided, create task with authenticated user_id, set is_complete=False, return created task, return 422 on validation error)
- [ ] T030 [US2] Register tasks router in backend/src/main.py (include tasks router with /api/tasks prefix)
- [ ] T031 [US2] Create protected tasks page in frontend/src/app/tasks/page.tsx (fetch tasks from GET /api/tasks on mount, display TaskList component, display TaskForm component for creation, handle loading state, handle empty state, handle errors)
- [ ] T032 [P] [US2] Create TaskList component in frontend/src/components/TaskList.tsx (receive tasks array prop, render TaskItem for each task, display "No tasks yet" empty state, support filter dropdown for all/complete/incomplete)
- [ ] T033 [P] [US2] Create TaskForm component in frontend/src/components/TaskForm.tsx (title input required max 200 chars, description textarea optional max 2000 chars, submit to POST /api/tasks, clear form on success, display validation errors, emit event to refresh task list)
- [ ] T034 [P] [US2] Create TaskItem component in frontend/src/components/TaskItem.tsx (display task title and description, display created_at timestamp, display completion status, placeholder for edit/delete/complete actions to be added in later stories)

**Checkpoint**: At this point, User Stories 1 AND 2 should both work independently

---

## Phase 5: User Story 3 - Update Task Details (Priority: P3)

**Goal**: Users can edit title and description of their existing tasks with ownership verification

**Independent Test**: Create task, edit title and description, verify changes saved and displayed, verify cannot edit other users' tasks

### Implementation for User Story 3

- [ ] T035 [P] [US3] Implement GET /api/tasks/{task_id} endpoint in backend/src/api/tasks.py (require JWT auth, query task by id AND authenticated user_id, return task if found, return 404 if not found, return 403 if task belongs to another user)
- [ ] T036 [P] [US3] Implement PUT /api/tasks/{task_id} endpoint in backend/src/api/tasks.py (require JWT auth, validate title not empty and max 200 chars, validate description max 2000 chars if provided, update task WHERE id=task_id AND user_id=authenticated_user_id, update updated_at timestamp, return updated task, return 404 if not found, return 403 if wrong owner, return 422 on validation error)
- [ ] T037 [US3] Update TaskForm component in frontend/src/components/TaskForm.tsx (add edit mode prop, pre-populate fields with existing task data in edit mode, submit to PUT /api/tasks/{id} in edit mode, emit event to refresh task list on success)
- [ ] T038 [US3] Update TaskItem component in frontend/src/components/TaskItem.tsx (add Edit button, show TaskForm in edit mode when Edit clicked, handle inline editing or modal dialog, display validation errors)

**Checkpoint**: All user stories 1, 2, AND 3 should now be independently functional

---

## Phase 6: User Story 4 - Mark Tasks Complete/Incomplete (Priority: P4)

**Goal**: Users can toggle task completion status with visual distinction and filtering

**Independent Test**: Create tasks, mark as complete, verify visual distinction, toggle back to incomplete, use filter to show only complete/incomplete tasks

### Implementation for User Story 4

- [ ] T039 [US4] Implement PATCH /api/tasks/{task_id}/complete endpoint in backend/src/api/tasks.py (require JWT auth, toggle is_complete field using NOT is_complete WHERE id=task_id AND user_id=authenticated_user_id, update updated_at timestamp, return updated task, return 404 if not found, return 403 if wrong owner)
- [ ] T040 [US4] Update TaskItem component in frontend/src/components/TaskItem.tsx (add checkbox or Complete button, call PATCH /api/tasks/{id}/complete on click, apply visual distinction for completed tasks using Tailwind CSS - strikethrough text and muted color, emit event to refresh task list)
- [ ] T041 [US4] Update TaskList component in frontend/src/components/TaskList.tsx (implement filter dropdown with options: All, Complete, Incomplete, filter tasks client-side based on is_complete status, update displayed count)

**Checkpoint**: All user stories 1-4 should now be independently functional

---

## Phase 7: User Story 5 - Delete Tasks (Priority: P5)

**Goal**: Users can permanently delete tasks with ownership verification

**Independent Test**: Create tasks, delete them, verify removed from list, verify cannot delete other users' tasks, verify empty state message when no tasks

### Implementation for User Story 5

- [ ] T042 [US5] Implement DELETE /api/tasks/{task_id} endpoint in backend/src/api/tasks.py (require JWT auth, delete task WHERE id=task_id AND user_id=authenticated_user_id, return 204 No Content on success, return 404 if not found, return 403 if wrong owner)
- [ ] T043 [US5] Update TaskItem component in frontend/src/components/TaskItem.tsx (add Delete button, call DELETE /api/tasks/{id} on click, add confirmation dialog before deletion, emit event to refresh task list on success, handle errors)
- [ ] T044 [US5] Update tasks page in frontend/src/app/tasks/page.tsx (display empty state message when task list is empty - "No tasks yet. Create your first task to get started!")

**Checkpoint**: All user stories should now be independently functional

---

## Phase 8: Polish & Cross-Cutting Concerns

**Purpose**: Improvements that affect multiple user stories

- [ ] T045 [P] Add error handling and user-friendly error messages across all frontend pages (network errors, validation errors, authentication errors)
- [ ] T046 [P] Add loading states and spinners to all async operations in frontend (task list loading, form submissions, delete confirmations)
- [ ] T047 [P] Add success notifications for task operations in frontend (task created, updated, completed, deleted)
- [ ] T048 [P] Update backend/.env.example with detailed comments explaining each environment variable
- [ ] T049 [P] Update frontend/.env.local.example with detailed comments explaining each environment variable
- [ ] T050 [P] Add CORS configuration in backend/src/main.py to allow frontend origin from NEXT_PUBLIC_API_URL
- [ ] T051 Validate quickstart.md instructions by following setup steps and documenting any issues

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Stories (Phase 3-7)**: All depend on Foundational phase completion
  - User stories can then proceed in parallel (if staffed)
  - Or sequentially in priority order (P1 → P2 → P3 → P4 → P5)
- **Polish (Phase 8)**: Depends on all desired user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational (Phase 2) - No dependencies on other stories
- **User Story 2 (P2)**: Can start after Foundational (Phase 2) - Requires authentication from US1 to test but is independently implementable
- **User Story 3 (P3)**: Can start after Foundational (Phase 2) - Extends US2 functionality but is independently testable
- **User Story 4 (P4)**: Can start after Foundational (Phase 2) - Extends US2 functionality but is independently testable
- **User Story 5 (P5)**: Can start after Foundational (Phase 2) - Extends US2 functionality but is independently testable

### Within Each User Story

- Backend endpoints before frontend pages (frontend needs API to call)
- Models and utilities before endpoints (endpoints depend on models)
- Core components before pages (pages use components)
- Story complete before moving to next priority

### Parallel Opportunities

- All Setup tasks marked [P] can run in parallel
- All Foundational tasks marked [P] can run in parallel (within Phase 2)
- Once Foundational phase completes, all user stories can start in parallel (if team capacity allows)
- Backend and frontend tasks within a story marked [P] can run in parallel
- Different user stories can be worked on in parallel by different team members

---

## Parallel Example: User Story 1

```bash
# Launch backend auth endpoints together:
Task: "Implement POST /api/auth/signup endpoint in backend/src/api/auth.py"
Task: "Implement POST /api/auth/signin endpoint in backend/src/api/auth.py"

# Launch frontend auth pages together:
Task: "Create signup page in frontend/src/app/signup/page.tsx"
Task: "Create signin page in frontend/src/app/signin/page.tsx"
Task: "Create landing page in frontend/src/app/page.tsx"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup
2. Complete Phase 2: Foundational (CRITICAL - blocks all stories)
3. Complete Phase 3: User Story 1 (Authentication)
4. **STOP and VALIDATE**: Test User Story 1 independently
5. Deploy/demo if ready

### Incremental Delivery

1. Complete Setup + Foundational → Foundation ready
2. Add User Story 1 → Test independently → Deploy/Demo (MVP!)
3. Add User Story 2 → Test independently → Deploy/Demo
4. Add User Story 3 → Test independently → Deploy/Demo
5. Add User Story 4 → Test independently → Deploy/Demo
6. Add User Story 5 → Test independently → Deploy/Demo
7. Each story adds value without breaking previous stories

### Parallel Team Strategy

With multiple developers:

1. Team completes Setup + Foundational together
2. Once Foundational is done:
   - Developer A: User Story 1 (Authentication)
   - Developer B: User Story 2 (Create/List Tasks) - can work on backend/frontend structure
   - Developer C: User Story 3 (Update Tasks) - can work on backend/frontend structure
3. Stories complete and integrate independently

---

## Notes

- [P] tasks = different files, no dependencies
- [Story] label maps task to specific user story for traceability
- Each user story should be independently completable and testable
- No automated tests per research.md decision (manual validation against acceptance scenarios)
- Commit after each task or logical group
- Stop at any checkpoint to validate story independently
- Avoid: vague tasks, same file conflicts, cross-story dependencies that break independence
- All task queries MUST filter by authenticated user_id for data isolation
- All API endpoints MUST verify JWT tokens via deps.py dependency
- Frontend MUST use API client from lib/api.ts to ensure JWT tokens are attached
