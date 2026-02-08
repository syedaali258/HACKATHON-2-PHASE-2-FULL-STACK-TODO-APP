# Tasks: Backend API & Database (Task Management Core)

**Input**: Design documents from `/specs/003-backend-task-api/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

All paths are relative to repository root: `backend/app/...`

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and basic structure

- [x] T001 Create backend directory structure per plan.md (backend/app/, backend/tests/, backend/app/models/, backend/app/schemas/, backend/app/api/, backend/app/api/routes/, backend/app/core/)
- [x] T002 Create requirements.txt with dependencies: fastapi==0.109.0, uvicorn[standard]==0.27.0, sqlmodel==0.0.14, asyncpg==0.29.0, pydantic==2.5.3, python-jose[cryptography]==3.3.0, python-dotenv==1.0.0, pytest==7.4.4, pytest-asyncio==0.23.3, httpx==0.26.0
- [x] T003 [P] Create .env.example with DATABASE_URL, BETTER_AUTH_SECRET, JWT_ALGORITHM, ENVIRONMENT, DEBUG placeholders
- [x] T004 [P] Create backend/README.md with setup instructions from quickstart.md
- [x] T005 [P] Create all __init__.py files in backend/app/, backend/app/models/, backend/app/schemas/, backend/app/api/, backend/app/api/routes/, backend/app/core/

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [x] T006 Create backend/app/config.py with Settings class using pydantic BaseSettings for DATABASE_URL, BETTER_AUTH_SECRET, JWT_ALGORITHM environment variables
- [x] T007 Create backend/app/database.py with async SQLAlchemy engine, connection pooling (pool_size=5, max_overflow=10, pool_recycle=3600, pool_pre_ping=True), and get_db dependency function
- [x] T008 Create backend/app/models/task.py with Task SQLModel entity including id, user_id, title, description, is_completed, created_at, updated_at fields per data-model.md
- [x] T009 [P] Create backend/app/schemas/task.py with TaskCreate, TaskUpdate, TaskResponse Pydantic schemas per data-model.md
- [x] T010 [P] Create backend/app/core/security.py with JWT verification utilities using python-jose (decode token, verify signature, extract user_id from 'sub' claim)
- [x] T011 Create backend/app/api/deps.py with get_current_user dependency function using HTTPBearer and Depends(security.verify_jwt) to extract user_id from Authorization header
- [x] T012 Create backend/app/main.py with FastAPI app initialization, CORS middleware, database table creation on startup (SQLModel.metadata.create_all), and health check endpoint
- [x] T013 Create backend/app/api/routes/tasks.py with APIRouter initialization and placeholder for task endpoints

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - View Personal Task List (Priority: P1) 🎯 MVP

**Goal**: Users can retrieve all their tasks in a list, filtered by their user_id from JWT token

**Independent Test**: Authenticate as a user, create several tasks via POST endpoint, call GET /api/tasks and verify only that user's tasks are returned. Test with multiple users to confirm data isolation.

### Implementation for User Story 1

- [x] T014 [US1] Implement GET /api/tasks endpoint in backend/app/api/routes/tasks.py with user_id from Depends(get_current_user), query tasks filtered by user_id, return list of TaskResponse
- [x] T015 [US1] Add query-level filtering in GET /api/tasks to ensure WHERE user_id = current_user_id in database query
- [x] T016 [US1] Add error handling for database connection failures in GET /api/tasks endpoint (return 500 with appropriate message)
- [x] T017 [US1] Register tasks router in backend/app/main.py with app.include_router(tasks.router, prefix="/api", tags=["tasks"])

**Checkpoint**: At this point, User Story 1 should be fully functional - users can view their task list with complete data isolation

---

## Phase 4: User Story 2 - Create New Tasks (Priority: P1) 🎯 MVP

**Goal**: Users can create new tasks that are automatically associated with their user_id from JWT token

**Independent Test**: Authenticate as a user, call POST /api/tasks with task data, verify task is created with correct user_id, then call GET /api/tasks to confirm it appears in the list.

### Implementation for User Story 2

- [x] T018 [US2] Implement POST /api/tasks endpoint in backend/app/api/routes/tasks.py with user_id from Depends(get_current_user), TaskCreate request body, create Task with user_id auto-assigned, return 201 with TaskResponse
- [x] T019 [US2] Add validation in POST /api/tasks to ensure title is not empty/whitespace using Pydantic validator from TaskCreate schema
- [x] T020 [US2] Add database session commit and refresh in POST /api/tasks to persist task and return created task with generated id
- [x] T021 [US2] Add error handling for validation failures in POST /api/tasks (return 422 with Pydantic validation errors)
- [x] T022 [US2] Add error handling for database errors in POST /api/tasks (return 500 with appropriate message)

**Checkpoint**: At this point, User Stories 1 AND 2 should both work independently - users can create tasks and view their list

---

## Phase 5: User Story 3 - View Individual Task Details (Priority: P2)

**Goal**: Users can retrieve complete details of a specific task by ID, with ownership verification

**Independent Test**: Create a task via POST, note its ID, call GET /api/tasks/{id} and verify all details are returned. Test with another user's token to confirm 404 is returned (not 403, to prevent enumeration).

### Implementation for User Story 3

- [x] T023 [US3] Implement GET /api/tasks/{id} endpoint in backend/app/api/routes/tasks.py with task_id path parameter, user_id from Depends(get_current_user), query task with WHERE id = task_id AND user_id = current_user_id
- [x] T024 [US3] Add ownership verification in GET /api/tasks/{id} to return 404 if task not found OR user doesn't own it (prevents information leakage)
- [x] T025 [US3] Add error handling for invalid task_id format in GET /api/tasks/{id} (return 422 for non-integer IDs)
- [x] T026 [US3] Return TaskResponse with all task fields including timestamps in GET /api/tasks/{id}

**Checkpoint**: At this point, User Stories 1, 2, AND 3 should all work independently - users can create, list, and view individual tasks

---

## Phase 6: User Story 4 - Update Existing Tasks (Priority: P2)

**Goal**: Users can update their tasks (title, description, completion status) with changes persisted to database

**Independent Test**: Create a task, call PUT /api/tasks/{id} with updated fields, verify changes are saved. Toggle is_completed between true/false and verify persistence. Test with another user's token to confirm 404 is returned.

### Implementation for User Story 4

- [x] T027 [US4] Implement PUT /api/tasks/{id} endpoint in backend/app/api/routes/tasks.py with task_id path parameter, user_id from Depends(get_current_user), TaskUpdate request body, query task with ownership check
- [x] T028 [US4] Add ownership verification in PUT /api/tasks/{id} to return 404 if task not found OR user doesn't own it
- [x] T029 [US4] Implement partial update logic in PUT /api/tasks/{id} to only update fields provided in TaskUpdate (skip None values)
- [x] T030 [US4] Update updated_at timestamp automatically in PUT /api/tasks/{id} when task is modified
- [x] T031 [US4] Add validation in PUT /api/tasks/{id} to ensure title is not empty/whitespace if provided
- [x] T032 [US4] Add database session commit and refresh in PUT /api/tasks/{id} to persist changes and return updated TaskResponse
- [x] T033 [US4] Add error handling for validation failures and database errors in PUT /api/tasks/{id}

**Checkpoint**: At this point, User Stories 1-4 should all work independently - full CRUD except delete

---

## Phase 7: User Story 5 - Delete Tasks (Priority: P3)

**Goal**: Users can permanently delete their tasks from the database

**Independent Test**: Create a task, call DELETE /api/tasks/{id}, verify 204 response. Call GET /api/tasks to confirm task no longer appears. Call GET /api/tasks/{id} to confirm 404 is returned. Test with another user's token to confirm 404 is returned.

### Implementation for User Story 5

- [x] T034 [US5] Implement DELETE /api/tasks/{id} endpoint in backend/app/api/routes/tasks.py with task_id path parameter, user_id from Depends(get_current_user), query task with ownership check
- [x] T035 [US5] Add ownership verification in DELETE /api/tasks/{id} to return 404 if task not found OR user doesn't own it
- [x] T036 [US5] Execute database delete operation in DELETE /api/tasks/{id} with session.delete(task) and session.commit()
- [x] T037 [US5] Return 204 No Content status in DELETE /api/tasks/{id} on successful deletion
- [x] T038 [US5] Add error handling for database errors in DELETE /api/tasks/{id}

**Checkpoint**: All user stories should now be independently functional - complete CRUD operations with data isolation

---

## Phase 8: Polish & Cross-Cutting Concerns

**Purpose**: Improvements that affect multiple user stories and production readiness

- [x] T039 [P] Add comprehensive error handling for expired JWT tokens across all endpoints (return 401 with "Token expired" message)
- [x] T040 [P] Add comprehensive error handling for malformed JWT tokens across all endpoints (return 401 with "Invalid token" message)
- [x] T041 [P] Add logging for all authentication failures in backend/app/api/deps.py using Python logging module
- [x] T042 [P] Add logging for all database operations (create, update, delete) in backend/app/api/routes/tasks.py
- [x] T043 [P] Validate field length constraints in all endpoints (title max 200 chars, description max 2000 chars)
- [x] T044 [P] Add database connection health check in backend/app/main.py startup event
- [x] T045 Create backend/.env file from .env.example with actual Neon PostgreSQL connection string and BETTER_AUTH_SECRET
- [x] T046 Test all endpoints with curl commands from quickstart.md to verify API contract compliance
- [x] T047 Verify all endpoints return correct HTTP status codes per contracts/task-api.yaml (200, 201, 204, 401, 404, 422, 500)
- [x] T048 Run manual data isolation test with two different JWT tokens to confirm no cross-user data access
- [x] T049 Update backend/README.md with final API documentation and deployment instructions

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Story 1 (Phase 3)**: Depends on Foundational (Phase 2) - No dependencies on other stories
- **User Story 2 (Phase 4)**: Depends on Foundational (Phase 2) - No dependencies on other stories (can run parallel with US1)
- **User Story 3 (Phase 5)**: Depends on Foundational (Phase 2) - No dependencies on other stories (can run parallel with US1/US2)
- **User Story 4 (Phase 6)**: Depends on Foundational (Phase 2) - No dependencies on other stories (can run parallel with US1/US2/US3)
- **User Story 5 (Phase 7)**: Depends on Foundational (Phase 2) - No dependencies on other stories (can run parallel with US1/US2/US3/US4)
- **Polish (Phase 8)**: Depends on all desired user stories being complete

### User Story Dependencies

All user stories are **INDEPENDENT** after Foundational phase:

- **User Story 1 (P1)**: Can start after Foundational - Implements GET /api/tasks (list)
- **User Story 2 (P1)**: Can start after Foundational - Implements POST /api/tasks (create)
- **User Story 3 (P2)**: Can start after Foundational - Implements GET /api/tasks/{id} (get single)
- **User Story 4 (P2)**: Can start after Foundational - Implements PUT /api/tasks/{id} (update)
- **User Story 5 (P3)**: Can start after Foundational - Implements DELETE /api/tasks/{id} (delete)

**Note**: While US1 and US2 are both P1 priority, they can be implemented in parallel since they touch different endpoints. US3-US5 can also be implemented in parallel with US1/US2 if team capacity allows.

### Within Each User Story

- All tasks within a user story should be completed sequentially in the order listed
- Tasks marked [P] within a phase can run in parallel
- Each user story phase should be completed and tested before moving to the next priority

### Parallel Opportunities

**Setup Phase (Phase 1)**:
- T003, T004, T005 can run in parallel (different files)

**Foundational Phase (Phase 2)**:
- T009 (schemas) and T010 (security) can run in parallel with T006-T008
- T009, T010 can run in parallel with each other

**User Stories (Phase 3-7)**:
- After Foundational phase completes, ALL user stories can be worked on in parallel by different developers
- Each story is completely independent and touches different endpoints

**Polish Phase (Phase 8)**:
- T039, T040, T041, T042, T043, T044 can all run in parallel (different concerns)

---

## Parallel Example: After Foundational Phase

```bash
# All user stories can start in parallel:
Developer A: Phase 3 (User Story 1 - View List)
Developer B: Phase 4 (User Story 2 - Create)
Developer C: Phase 5 (User Story 3 - View Single)
Developer D: Phase 6 (User Story 4 - Update)
Developer E: Phase 7 (User Story 5 - Delete)

# Or with single developer, implement in priority order:
1. Complete US1 (P1) → Test independently
2. Complete US2 (P1) → Test independently
3. Complete US3 (P2) → Test independently
4. Complete US4 (P2) → Test independently
5. Complete US5 (P3) → Test independently
```

---

## Implementation Strategy

### MVP First (User Stories 1 & 2 Only)

**Minimum Viable Product** - Users can create and view tasks:

1. Complete Phase 1: Setup (T001-T005)
2. Complete Phase 2: Foundational (T006-T013) - CRITICAL
3. Complete Phase 3: User Story 1 (T014-T017) - View task list
4. Complete Phase 4: User Story 2 (T018-T022) - Create tasks
5. **STOP and VALIDATE**: Test US1 and US2 independently
6. Deploy/demo MVP

**MVP Delivers**: Users can create tasks and see their personal task list with complete data isolation.

### Incremental Delivery

**Recommended approach** - Add one story at a time:

1. Complete Setup + Foundational → Foundation ready
2. Add User Story 1 → Test independently → Deploy/Demo
3. Add User Story 2 → Test independently → Deploy/Demo (MVP!)
4. Add User Story 3 → Test independently → Deploy/Demo
5. Add User Story 4 → Test independently → Deploy/Demo
6. Add User Story 5 → Test independently → Deploy/Demo
7. Add Polish → Final production-ready version

Each story adds value without breaking previous stories.

### Parallel Team Strategy

With multiple developers:

1. Team completes Setup + Foundational together (T001-T013)
2. Once Foundational is done (after T013):
   - Developer A: User Story 1 (T014-T017)
   - Developer B: User Story 2 (T018-T022)
   - Developer C: User Story 3 (T023-T026)
   - Developer D: User Story 4 (T027-T033)
   - Developer E: User Story 5 (T034-T038)
3. Stories complete and integrate independently
4. Team completes Polish together (T039-T049)

---

## Task Summary

**Total Tasks**: 49 tasks

**Tasks per Phase**:
- Phase 1 (Setup): 5 tasks
- Phase 2 (Foundational): 8 tasks (BLOCKING)
- Phase 3 (US1 - View List): 4 tasks
- Phase 4 (US2 - Create): 5 tasks
- Phase 5 (US3 - View Single): 4 tasks
- Phase 6 (US4 - Update): 7 tasks
- Phase 7 (US5 - Delete): 5 tasks
- Phase 8 (Polish): 11 tasks

**Parallel Opportunities**: 15 tasks marked [P] can run in parallel within their phases

**Independent Test Criteria**:
- US1: Authenticate, create tasks, verify list shows only user's tasks
- US2: Authenticate, create task, verify it appears in list with correct user_id
- US3: Create task, retrieve by ID, verify all details returned, test ownership
- US4: Create task, update fields, verify changes persist, test completion toggle
- US5: Create task, delete it, verify it's gone from list and returns 404

**MVP Scope**: User Stories 1 & 2 (13 tasks after Setup + Foundational)

---

## Notes

- [P] tasks = different files, no dependencies within phase
- [Story] label maps task to specific user story for traceability
- Each user story is independently completable and testable
- No tests included (not requested in specification)
- All endpoints enforce JWT authentication and data isolation
- Commit after each task or logical group
- Stop at any checkpoint to validate story independently
- Follow quickstart.md for manual testing procedures
