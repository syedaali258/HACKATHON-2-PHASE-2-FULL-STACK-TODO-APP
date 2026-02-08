# Tasks: Frontend Application (Next.js App Router)

**Input**: Design documents from `/specs/004-nextjs-frontend/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

All paths are relative to repository root: `frontend/...`

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and basic structure

- [ ] T001 Create frontend directory structure per plan.md (frontend/app/, frontend/components/, frontend/lib/, frontend/types/, frontend/app/(auth)/, frontend/app/(protected)/, frontend/components/auth/, frontend/components/tasks/, frontend/components/ui/, frontend/lib/api/, frontend/lib/auth/, frontend/lib/utils/)
- [ ] T002 Initialize Next.js 16+ project with TypeScript in frontend/ directory using create-next-app
- [ ] T003 Create package.json with dependencies: next@^16.0.0, react@^19.0.0, react-dom@^19.0.0, better-auth@latest, axios@^1.6.0, zod@^3.22.0, react-hook-form@^7.49.0, @hookform/resolvers@^3.3.0, tailwindcss@^3.4.0, typescript@^5.3.0
- [ ] T004 [P] Create .env.example with NEXT_PUBLIC_API_URL, NEXT_PUBLIC_BETTER_AUTH_URL placeholders
- [ ] T005 [P] Create frontend/README.md with setup instructions from quickstart.md
- [ ] T006 [P] Configure tailwind.config.js with mobile-first breakpoints and content paths
- [ ] T007 [P] Configure tsconfig.json with strict mode and path aliases (@/*)
- [ ] T008 [P] Configure next.config.js with environment variables and build settings
- [ ] T009 [P] Create frontend/app/globals.css with Tailwind directives and base styles

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [x] T010 Create frontend/types/task.ts with Task, TaskCreateInput, TaskUpdateInput TypeScript interfaces per data-model.md
- [x] T011 [P] Create frontend/types/auth.ts with UserSession, SignUpInput, SignInInput TypeScript interfaces per data-model.md
- [x] T012 [P] Create frontend/types/api.ts with ApiResponse, ApiError, LoadingState, ErrorState TypeScript interfaces per data-model.md
- [x] T013 Create frontend/lib/utils/validation.ts with Zod schemas (taskCreateSchema, taskUpdateSchema, signUpSchema, signInSchema) per data-model.md
- [x] T014 Create frontend/lib/auth/better-auth.ts with Better Auth client configuration (httpOnly cookies, secure settings)
- [x] T015 Create frontend/lib/api/client.ts with axios instance, request interceptor for JWT attachment, response interceptor for 401 handling
- [x] T016 Create frontend/lib/api/tasks.ts with taskApi methods (getAll, getById, create, update, delete) using apiClient
- [x] T017 [P] Create frontend/lib/api/errors.ts with handleApiError utility function for centralized error handling
- [x] T018 Create frontend/middleware.ts with route protection logic (check JWT cookie, redirect unauthenticated users, redirect authenticated users away from auth pages)
- [x] T019 Create frontend/app/layout.tsx with root layout, metadata, and global styles import
- [x] T020 Create frontend/app/page.tsx with landing page that redirects based on auth status (authenticated → /dashboard, unauthenticated → /signin)

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - User Authentication (Priority: P1) 🎯 MVP

**Goal**: Users can sign up, sign in, and access protected pages with JWT authentication

**Independent Test**: Visit signup page, create account, verify auto-signin and redirect to dashboard. Sign out, sign in with credentials, verify redirect to dashboard. Try accessing /dashboard without auth, verify redirect to signin.

### Implementation for User Story 1

- [x] T021 [US1] Create frontend/components/ui/Button.tsx with reusable button component (primary, secondary, loading states)
- [x] T022 [US1] Create frontend/components/ui/Input.tsx with reusable input component (text, email, password types, error states)
- [x] T023 [US1] Create frontend/components/ui/LoadingSpinner.tsx with loading indicator component
- [x] T024 [US1] Create frontend/components/auth/SignUpForm.tsx with signup form using react-hook-form and signUpSchema validation
- [x] T025 [US1] Create frontend/app/(auth)/signup/page.tsx with signup page that renders SignUpForm and handles Better Auth signup
- [x] T026 [US1] Create frontend/components/auth/SignInForm.tsx with signin form using react-hook-form and signInSchema validation
- [x] T027 [US1] Create frontend/app/(auth)/signin/page.tsx with signin page that renders SignInForm and handles Better Auth signin
- [x] T028 [US1] Add error handling in SignUpForm for validation errors (422) and duplicate email errors
- [x] T029 [US1] Add error handling in SignInForm for invalid credentials (401) and network errors
- [ ] T030 [US1] Test middleware redirects: unauthenticated users to /signin, authenticated users away from /signin and /signup

**Checkpoint**: At this point, User Story 1 should be fully functional - users can sign up, sign in, and middleware protects routes

---

## Phase 4: User Story 2 - View Task Dashboard (Priority: P1) 🎯 MVP

**Goal**: Authenticated users can view their personal task dashboard with all their tasks

**Independent Test**: Sign in as a user, navigate to /dashboard, verify tasks are displayed. Create tasks via backend API, refresh dashboard, verify they appear. Sign in as different user, verify only that user's tasks are shown.

### Implementation for User Story 2

- [x] T031 [US2] Create frontend/components/tasks/TaskItem.tsx with task display component (title, description, completion status, timestamps)
- [x] T032 [US2] Create frontend/components/tasks/TaskList.tsx with task list component that maps over tasks array and renders TaskItem components
- [x] T033 [US2] Create frontend/app/(protected)/dashboard/page.tsx with dashboard page that fetches tasks on mount using taskApi.getAll()
- [x] T034 [US2] Add loading state to dashboard page with LoadingSpinner component during API fetch
- [x] T035 [US2] Add empty state to dashboard page when user has no tasks (message + "Create your first task" prompt)
- [x] T036 [US2] Add error state to dashboard page with error message and retry button when API fetch fails
- [x] T037 [US2] Create frontend/app/(protected)/dashboard/loading.tsx with loading UI for dashboard
- [x] T038 [US2] Create frontend/app/(protected)/dashboard/error.tsx with error UI and reset button for dashboard
- [x] T039 [US2] Add responsive grid layout to TaskList component (1 column mobile, 2 columns tablet, 3 columns desktop)

**Checkpoint**: At this point, User Stories 1 AND 2 should both work independently - users can authenticate and view their task dashboard

---

## Phase 5: User Story 3 - Create New Tasks (Priority: P1) 🎯 MVP

**Goal**: Authenticated users can create new tasks that appear in their dashboard immediately

**Independent Test**: Sign in, click "Create Task", enter title and description, submit form, verify task appears in dashboard immediately. Test validation by submitting empty title, verify error message.

### Implementation for User Story 3

- [x] T040 [US3] Create frontend/components/tasks/CreateTaskForm.tsx with task creation form using react-hook-form and taskCreateSchema validation
- [x] T041 [US3] Add CreateTaskForm to dashboard page (inline or modal, as per design preference)
- [x] T042 [US3] Implement form submission in CreateTaskForm that calls taskApi.create() with form data
- [x] T043 [US3] Add optimistic UI update in dashboard page: add new task to local state immediately after form submission
- [x] T044 [US3] Add loading state to CreateTaskForm submit button during API request
- [x] T045 [US3] Add error handling in CreateTaskForm for validation errors (422) and API errors (500)
- [x] T046 [US3] Add success feedback after task creation (toast notification or inline message)
- [x] T047 [US3] Reset form fields after successful task creation
- [x] T048 [US3] Prevent duplicate submissions by disabling submit button during API request

**Checkpoint**: At this point, User Stories 1, 2, AND 3 should all work independently - users can authenticate, view dashboard, and create tasks (MVP complete!)

---

## Phase 6: User Story 4 - Update and Edit Tasks (Priority: P2)

**Goal**: Authenticated users can edit existing tasks (title, description) with changes persisted

**Independent Test**: Create a task, click edit action, modify title and description, save changes, verify updates appear in dashboard. Test validation by clearing title, verify error message. Cancel edit, verify original values retained.

### Implementation for User Story 4

- [x] T049 [US4] Create frontend/components/tasks/EditTaskModal.tsx with edit form using react-hook-form and taskUpdateSchema validation
- [x] T050 [US4] Add edit button to TaskItem component that opens EditTaskModal with current task data
- [x] T051 [US4] Implement form submission in EditTaskModal that calls taskApi.update() with task ID and updated fields
- [x] T052 [US4] Add optimistic UI update in dashboard page: update task in local state immediately after form submission
- [x] T053 [US4] Add loading state to EditTaskModal save button during API request
- [x] T054 [US4] Add error handling in EditTaskModal for validation errors (422) and API errors (404, 500)
- [x] T055 [US4] Add cancel button to EditTaskModal that closes modal and discards changes
- [x] T056 [US4] Add error rollback: revert optimistic update if API call fails
- [x] T057 [US4] Close modal automatically after successful update

**Checkpoint**: At this point, User Stories 1-4 should all work independently - full CRUD except delete and completion toggle

---

## Phase 7: User Story 5 - Toggle Task Completion (Priority: P2)

**Goal**: Authenticated users can quickly toggle task completion status with a single click

**Independent Test**: Create a task, click completion checkbox, verify task marked as complete immediately. Click again, verify task marked as incomplete. Refresh page, verify completion state persists.

### Implementation for User Story 5

- [x] T058 [US5] Add completion checkbox to TaskItem component (styled with Tailwind, shows checked/unchecked state)
- [x] T059 [US5] Implement checkbox onChange handler that calls taskApi.update() with is_completed toggle
- [x] T060 [US5] Add optimistic UI update: toggle is_completed in local state immediately on checkbox click
- [x] T061 [US5] Add loading state to checkbox during API request (disabled, spinner icon)
- [x] T062 [US5] Add error handling: revert optimistic update if API call fails, show error message
- [x] T063 [US5] Add visual distinction for completed tasks (strikethrough title, muted colors)
- [x] T064 [US5] Test completion toggle persists after page refresh

**Checkpoint**: At this point, User Stories 1-5 should all work independently - full CRUD except delete

---

## Phase 8: User Story 6 - Delete Tasks (Priority: P2)

**Goal**: Authenticated users can permanently delete tasks with confirmation to prevent accidents

**Independent Test**: Create a task, click delete button, verify confirmation dialog appears. Confirm deletion, verify task removed from dashboard. Create another task, click delete, cancel confirmation, verify task remains.

### Implementation for User Story 6

- [x] T065 [US6] Create frontend/components/tasks/DeleteConfirmDialog.tsx with confirmation dialog component (modal with confirm/cancel buttons)
- [x] T066 [US6] Add delete button to TaskItem component that opens DeleteConfirmDialog
- [x] T067 [US6] Implement confirm handler in DeleteConfirmDialog that calls taskApi.delete() with task ID
- [x] T068 [US6] Add optimistic UI update in dashboard page: remove task from local state immediately after confirmation
- [x] T069 [US6] Add loading state to DeleteConfirmDialog during API request (disabled buttons, spinner)
- [x] T070 [US6] Add error handling: restore task to local state if API call fails, show error message
- [x] T071 [US6] Close dialog automatically after successful deletion
- [x] T072 [US6] Add cancel button that closes dialog without deleting task

**Checkpoint**: All user stories 1-6 should now be independently functional - complete CRUD operations with authentication

---

## Phase 9: User Story 7 - User Logout (Priority: P3)

**Goal**: Authenticated users can sign out to end their session and protect their account

**Independent Test**: Sign in, click logout button, verify redirect to signin page. Try accessing /dashboard, verify redirect to signin. Click browser back button, verify cannot access protected pages.

### Implementation for User Story 7

- [x] T073 [US7] Add logout button to dashboard page header or navigation
- [x] T074 [US7] Implement logout handler that calls authClient.signOut() from Better Auth
- [x] T075 [US7] Add redirect to /signin after successful logout
- [x] T076 [US7] Verify middleware prevents access to protected pages after logout
- [x] T077 [US7] Test browser back button cannot access protected pages after logout

**Checkpoint**: All user stories 1-7 should now be independently functional - complete auth + CRUD operations

---

## Phase 10: User Story 8 - Responsive Design (Priority: P3)

**Goal**: Application adapts to different screen sizes (mobile, tablet, desktop) while maintaining functionality

**Independent Test**: Open application in browser DevTools, test at 320px, 768px, 1024px, 1920px widths. Verify all features accessible, layout adapts, no horizontal scrolling, touch targets appropriately sized.

### Implementation for User Story 8

- [x] T078 [US8] Add responsive breakpoints to TaskList component (grid-cols-1 sm:grid-cols-2 lg:grid-cols-3)
- [x] T079 [US8] Add responsive padding and spacing to dashboard page (p-4 sm:p-6 lg:p-8)
- [x] T080 [US8] Add responsive font sizes to headings and text (text-base sm:text-lg lg:text-xl)
- [x] T081 [US8] Add responsive button sizes for touch targets on mobile (min-h-12 on mobile)
- [x] T082 [US8] Add responsive modal widths (full-width on mobile, max-w-md on desktop)
- [x] T083 [US8] Add responsive form layouts (stacked on mobile, side-by-side on desktop)
- [x] T084 [US8] Test all features on mobile viewport (320px width minimum)
- [x] T085 [US8] Test all features on tablet viewport (768px width)
- [x] T086 [US8] Test all features on desktop viewport (1024px+ width)

**Checkpoint**: All user stories should now be independently functional with responsive design across all devices

---

## Phase 11: Polish & Cross-Cutting Concerns

**Purpose**: Improvements that affect multiple user stories and production readiness

- [ ] T087 [P] Add loading indicators to all async operations (form submissions, API calls)
- [ ] T088 [P] Add error messages for all failure scenarios (network errors, validation errors, API errors)
- [ ] T089 [P] Add success feedback for all user actions (task created, updated, deleted, completed)
- [ ] T090 [P] Implement form validation error display for all forms (field-specific error messages)
- [ ] T091 [P] Add keyboard accessibility (Enter to submit forms, Escape to close modals)
- [ ] T092 [P] Add ARIA labels to all interactive elements (buttons, inputs, checkboxes)
- [ ] T093 [P] Add focus management for modals (trap focus, restore focus on close)
- [ ] T094 [P] Test token expiration handling (401 errors trigger signout and redirect)
- [ ] T095 [P] Test network failure handling (show retry options, preserve form data)
- [ ] T096 [P] Test rapid successive actions (prevent duplicate submissions, debounce if needed)
- [ ] T097 [P] Add meta tags for SEO (title, description, viewport)
- [ ] T098 [P] Optimize images and assets for production build
- [ ] T099 Create frontend/.env.local from .env.example with actual API URLs
- [ ] T100 Run npm run build to verify production build succeeds
- [ ] T101 Test all user stories end-to-end with production build
- [ ] T102 Update frontend/README.md with final deployment instructions

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Story 1 (Phase 3)**: Depends on Foundational (Phase 2) - No dependencies on other stories
- **User Story 2 (Phase 4)**: Depends on Foundational (Phase 2) - No dependencies on other stories (can run parallel with US1)
- **User Story 3 (Phase 5)**: Depends on Foundational (Phase 2) and US2 (needs dashboard to display created tasks)
- **User Story 4 (Phase 6)**: Depends on Foundational (Phase 2) and US2 (needs tasks to edit)
- **User Story 5 (Phase 7)**: Depends on Foundational (Phase 2) and US2 (needs tasks to toggle)
- **User Story 6 (Phase 8)**: Depends on Foundational (Phase 2) and US2 (needs tasks to delete)
- **User Story 7 (Phase 9)**: Depends on Foundational (Phase 2) and US1 (needs auth to logout)
- **User Story 8 (Phase 10)**: Depends on all UI components being implemented (US1-US7)
- **Polish (Phase 11)**: Depends on all desired user stories being complete

### User Story Dependencies

**Independent Stories** (can start after Foundational):
- **User Story 1 (P1)**: Authentication - No dependencies on other stories
- **User Story 2 (P1)**: View Dashboard - No dependencies on other stories

**Dependent Stories** (require other stories first):
- **User Story 3 (P1)**: Create Tasks - Depends on US2 (needs dashboard to display)
- **User Story 4 (P2)**: Edit Tasks - Depends on US2 (needs tasks to edit)
- **User Story 5 (P2)**: Toggle Completion - Depends on US2 (needs tasks to toggle)
- **User Story 6 (P2)**: Delete Tasks - Depends on US2 (needs tasks to delete)
- **User Story 7 (P3)**: Logout - Depends on US1 (needs auth to logout)
- **User Story 8 (P3)**: Responsive Design - Depends on US1-US7 (needs all UI components)

### Parallel Opportunities

**Setup Phase (Phase 1)**:
- T004, T005, T006, T007, T008, T009 can run in parallel (different files)

**Foundational Phase (Phase 2)**:
- T011, T012, T017 can run in parallel with T010, T013, T014, T015, T016
- T019, T020 can run in parallel with T010-T018

**User Stories (Phase 3-10)**:
- After Foundational phase completes:
  - US1 (Auth) and US2 (Dashboard) can be worked on in parallel
  - US3-US6 must wait for US2 to complete
  - US7 must wait for US1 to complete
  - US8 must wait for all UI components (US1-US7)

**Polish Phase (Phase 11)**:
- T087, T088, T089, T090, T091, T092, T093, T094, T095, T096, T097, T098 can all run in parallel

---

## Implementation Strategy

### MVP First (User Stories 1, 2, 3 Only)

**Minimum Viable Product** - Users can authenticate, view dashboard, and create tasks:

1. Complete Phase 1: Setup (T001-T009)
2. Complete Phase 2: Foundational (T010-T020) - CRITICAL
3. Complete Phase 3: User Story 1 (T021-T030) - Authentication
4. Complete Phase 4: User Story 2 (T031-T039) - View Dashboard
5. Complete Phase 5: User Story 3 (T040-T048) - Create Tasks
6. **STOP and VALIDATE**: Test US1, US2, US3 independently
7. Deploy/demo MVP

**MVP Delivers**: Users can sign up, sign in, view their personal task dashboard, and create new tasks with complete data isolation.

### Incremental Delivery

**Recommended approach** - Add one story at a time:

1. Complete Setup + Foundational → Foundation ready
2. Add User Story 1 → Test independently → Deploy/Demo
3. Add User Story 2 → Test independently → Deploy/Demo
4. Add User Story 3 → Test independently → Deploy/Demo (MVP!)
5. Add User Story 4 → Test independently → Deploy/Demo
6. Add User Story 5 → Test independently → Deploy/Demo
7. Add User Story 6 → Test independently → Deploy/Demo
8. Add User Story 7 → Test independently → Deploy/Demo
9. Add User Story 8 → Test independently → Deploy/Demo
10. Add Polish → Final production-ready version

Each story adds value without breaking previous stories.

### Parallel Team Strategy

With multiple developers:

1. Team completes Setup + Foundational together (T001-T020)
2. Once Foundational is done (after T020):
   - Developer A: User Story 1 (T021-T030) - Authentication
   - Developer B: User Story 2 (T031-T039) - Dashboard
3. After US1 and US2 complete:
   - Developer C: User Story 3 (T040-T048) - Create Tasks
   - Developer D: User Story 4 (T049-T057) - Edit Tasks
   - Developer E: User Story 5 (T058-T064) - Toggle Completion
   - Developer F: User Story 6 (T065-T072) - Delete Tasks
4. After US1 completes:
   - Developer G: User Story 7 (T073-T077) - Logout
5. After all UI components complete:
   - Developer H: User Story 8 (T078-T086) - Responsive Design
6. Team completes Polish together (T087-T102)

---

## Task Summary

**Total Tasks**: 102 tasks

**Tasks per Phase**:
- Phase 1 (Setup): 9 tasks
- Phase 2 (Foundational): 11 tasks (BLOCKING)
- Phase 3 (US1 - Authentication): 10 tasks
- Phase 4 (US2 - View Dashboard): 9 tasks
- Phase 5 (US3 - Create Tasks): 9 tasks
- Phase 6 (US4 - Edit Tasks): 9 tasks
- Phase 7 (US5 - Toggle Completion): 7 tasks
- Phase 8 (US6 - Delete Tasks): 8 tasks
- Phase 9 (US7 - Logout): 5 tasks
- Phase 10 (US8 - Responsive Design): 9 tasks
- Phase 11 (Polish): 16 tasks

**Parallel Opportunities**: 25 tasks marked [P] can run in parallel within their phases

**Independent Test Criteria**:
- US1: Visit signup, create account, verify auto-signin and redirect. Sign out, sign in, verify redirect.
- US2: Sign in, navigate to dashboard, verify tasks displayed. Test with multiple users, verify data isolation.
- US3: Sign in, create task, verify appears in dashboard immediately. Test validation errors.
- US4: Create task, edit it, verify changes saved. Test validation, cancel, error handling.
- US5: Create task, toggle completion, verify state changes and persists after refresh.
- US6: Create task, delete it, verify removed from dashboard. Test confirmation and cancel.
- US7: Sign in, logout, verify redirect and cannot access protected pages.
- US8: Test application at 320px, 768px, 1024px, 1920px widths, verify all features accessible.

**MVP Scope**: User Stories 1, 2, 3 (39 tasks after Setup + Foundational)

---

## Notes

- [P] tasks = different files, no dependencies within phase
- [Story] label maps task to specific user story for traceability
- Each user story is independently completable and testable
- No automated tests included (manual testing only per spec)
- All endpoints enforce JWT authentication via middleware
- All API calls use axios client with automatic JWT attachment
- Commit after each task or logical group
- Stop at any checkpoint to validate story independently
- Follow quickstart.md for manual testing procedures
