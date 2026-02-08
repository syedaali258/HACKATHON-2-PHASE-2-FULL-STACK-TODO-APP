# Research: Frontend Application (Next.js App Router)

**Feature**: 004-nextjs-frontend
**Date**: 2026-02-08
**Purpose**: Resolve technical unknowns and document best practices for Next.js 16+ App Router frontend with Better Auth integration

## Research Areas

### 1. Better Auth Integration with Next.js App Router

**Decision**: Use Better Auth client library with JWT token management

**Rationale**:
- Better Auth provides built-in JWT token handling for Next.js
- Supports both Server Components and Client Components
- Handles token refresh automatically
- Provides React hooks for authentication state
- Compatible with Next.js 16+ App Router architecture

**Implementation Pattern**:
```typescript
// lib/auth/better-auth.ts
import { createAuthClient } from "better-auth/client"

export const authClient = createAuthClient({
  baseURL: process.env.NEXT_PUBLIC_API_URL,
  // Better Auth will handle JWT storage and refresh
})

// Usage in components
import { useSession } from "better-auth/react"

export function ProtectedComponent() {
  const { data: session, status } = useSession()

  if (status === "loading") return <LoadingSpinner />
  if (status === "unauthenticated") redirect("/signin")

  return <div>Protected content for {session.user.email}</div>
}
```

**Alternatives Considered**:
- NextAuth.js: More complex setup, heavier bundle size
- Custom JWT implementation: Reinventing the wheel, more error-prone
- Auth0/Clerk: Third-party services, not aligned with Better Auth requirement

---

### 2. JWT Token Storage Strategy

**Decision**: Use httpOnly cookies for JWT token storage (primary), with localStorage as fallback for development

**Rationale**:
- httpOnly cookies are immune to XSS attacks (JavaScript cannot access them)
- Cookies are automatically sent with every request to the same domain
- Better Auth supports httpOnly cookie mode out of the box
- More secure than localStorage for production environments
- Prevents token theft via malicious scripts

**Implementation Pattern**:
```typescript
// Better Auth configuration
export const authClient = createAuthClient({
  baseURL: process.env.NEXT_PUBLIC_API_URL,
  cookieOptions: {
    httpOnly: true,
    secure: process.env.NODE_ENV === "production", // HTTPS only in production
    sameSite: "lax", // CSRF protection
    maxAge: 60 * 60 * 24 * 7, // 7 days
  },
})
```

**Security Considerations**:
- httpOnly cookies cannot be accessed by JavaScript (XSS protection)
- Secure flag ensures cookies only sent over HTTPS in production
- SameSite=lax provides CSRF protection
- Token expiration enforced server-side

**Alternatives Considered**:
- localStorage: Vulnerable to XSS attacks, easier to implement but less secure
- sessionStorage: Lost on tab close, not suitable for persistent sessions
- Memory only: Lost on page refresh, poor UX

---

### 3. Route Protection with Next.js Middleware

**Decision**: Use Next.js middleware.ts for route-level authentication checks

**Rationale**:
- Middleware runs before page rendering (edge runtime)
- Can redirect unauthenticated users before loading protected pages
- Centralized authentication logic (DRY principle)
- Works with both Server Components and Client Components
- Supports pattern matching for route groups

**Implementation Pattern**:
```typescript
// middleware.ts
import { NextResponse } from "next/server"
import type { NextRequest } from "next/server"

export function middleware(request: NextRequest) {
  const token = request.cookies.get("better-auth.session_token")
  const isAuthPage = request.nextUrl.pathname.startsWith("/signin") ||
                     request.nextUrl.pathname.startsWith("/signup")
  const isProtectedPage = request.nextUrl.pathname.startsWith("/dashboard")

  // Redirect authenticated users away from auth pages
  if (token && isAuthPage) {
    return NextResponse.redirect(new URL("/dashboard", request.url))
  }

  // Redirect unauthenticated users to signin
  if (!token && isProtectedPage) {
    return NextResponse.redirect(new URL("/signin", request.url))
  }

  return NextResponse.next()
}

export const config = {
  matcher: ["/((?!api|_next/static|_next/image|favicon.ico).*)"],
}
```

**Alternatives Considered**:
- Component-level checks: Duplicated logic, flash of wrong content
- Server Component checks only: Doesn't prevent client-side navigation
- Higher-order components: More boilerplate, less performant

---

### 4. API Client with Automatic JWT Attachment

**Decision**: Create a custom axios instance with request interceptor for JWT attachment

**Rationale**:
- Centralized API configuration (base URL, headers, timeouts)
- Automatic JWT token attachment to all requests
- Request/response interceptors for error handling
- Type-safe API methods with TypeScript
- Easy to mock for testing

**Implementation Pattern**:
```typescript
// lib/api/client.ts
import axios from "axios"
import { authClient } from "@/lib/auth/better-auth"

const apiClient = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL,
  timeout: 10000,
  headers: {
    "Content-Type": "application/json",
  },
})

// Request interceptor: attach JWT token
apiClient.interceptors.request.use(
  async (config) => {
    const session = await authClient.getSession()
    if (session?.accessToken) {
      config.headers.Authorization = `Bearer ${session.accessToken}`
    }
    return config
  },
  (error) => Promise.reject(error)
)

// Response interceptor: handle errors
apiClient.interceptors.response.use(
  (response) => response,
  async (error) => {
    if (error.response?.status === 401) {
      // Token expired, redirect to signin
      await authClient.signOut()
      window.location.href = "/signin"
    }
    return Promise.reject(error)
  }
)

export default apiClient
```

**Alternatives Considered**:
- Native fetch with manual token attachment: More boilerplate, no interceptors
- SWR/React Query with custom fetcher: Adds complexity, overkill for simple CRUD
- GraphQL client: Not aligned with REST API requirement

---

### 5. Form Validation Strategy

**Decision**: Use Zod for schema validation with react-hook-form for form state management

**Rationale**:
- Zod provides TypeScript-first schema validation
- Integrates seamlessly with react-hook-form via @hookform/resolvers
- Reusable validation schemas across components
- Client-side validation before API calls (reduces server load)
- Type inference from schemas (DRY principle)

**Implementation Pattern**:
```typescript
// lib/utils/validation.ts
import { z } from "zod"

export const taskCreateSchema = z.object({
  title: z.string()
    .min(1, "Title is required")
    .max(200, "Title must be 200 characters or less")
    .trim(),
  description: z.string()
    .max(2000, "Description must be 2000 characters or less")
    .optional(),
  is_completed: z.boolean().default(false),
})

export type TaskCreateInput = z.infer<typeof taskCreateSchema>

// Usage in component
import { useForm } from "react-hook-form"
import { zodResolver } from "@hookform/resolvers/zod"

const { register, handleSubmit, formState: { errors } } = useForm<TaskCreateInput>({
  resolver: zodResolver(taskCreateSchema),
})
```

**Alternatives Considered**:
- Manual validation: Error-prone, not type-safe, lots of boilerplate
- Yup: Less TypeScript-friendly than Zod
- Formik: Heavier bundle, more opinionated

---

### 6. Responsive Design with Tailwind CSS

**Decision**: Use Tailwind CSS utility classes with mobile-first breakpoints

**Rationale**:
- Mobile-first approach aligns with requirement (minimum 320px width)
- Utility classes reduce CSS bundle size (tree-shaking)
- Consistent design system with predefined spacing/colors
- No CSS-in-JS runtime overhead
- Easy to customize via tailwind.config.js

**Implementation Pattern**:
```typescript
// Mobile-first responsive component
export function TaskList({ tasks }: { tasks: Task[] }) {
  return (
    <div className="
      grid
      grid-cols-1           /* Mobile: 1 column */
      sm:grid-cols-2        /* Tablet: 2 columns (640px+) */
      lg:grid-cols-3        /* Desktop: 3 columns (1024px+) */
      gap-4
      p-4
    ">
      {tasks.map(task => (
        <TaskItem key={task.id} task={task} />
      ))}
    </div>
  )
}
```

**Breakpoints**:
- Mobile: < 640px (default, no prefix)
- Tablet: 640px+ (sm:)
- Desktop: 1024px+ (lg:)
- Large Desktop: 1280px+ (xl:)

**Alternatives Considered**:
- CSS Modules: More boilerplate, harder to maintain
- Styled Components: Runtime overhead, not aligned with Next.js best practices
- Plain CSS: No design system, harder to maintain consistency

---

### 7. State Management for Task CRUD

**Decision**: Use React hooks (useState, useEffect) with optimistic UI updates

**Rationale**:
- Simple CRUD operations don't require complex state management
- React hooks sufficient for component-level state
- Optimistic updates improve perceived performance
- Server state managed by API calls (no global state needed)
- Reduces bundle size (no Redux/Zustand)

**Implementation Pattern**:
```typescript
// Optimistic update pattern
const [tasks, setTasks] = useState<Task[]>([])
const [isLoading, setIsLoading] = useState(false)

const toggleTaskCompletion = async (taskId: number) => {
  // Optimistic update
  setTasks(prev => prev.map(task =>
    task.id === taskId
      ? { ...task, is_completed: !task.is_completed }
      : task
  ))

  try {
    await apiClient.put(`/api/tasks/${taskId}`, {
      is_completed: !tasks.find(t => t.id === taskId)?.is_completed
    })
  } catch (error) {
    // Revert on error
    setTasks(prev => prev.map(task =>
      task.id === taskId
        ? { ...task, is_completed: !task.is_completed }
        : task
    ))
    toast.error("Failed to update task")
  }
}
```

**Alternatives Considered**:
- Redux: Overkill for simple CRUD, adds complexity
- Zustand: Unnecessary for component-level state
- React Query: Adds complexity, caching not required for this scope

---

### 8. Error Handling and Loading States

**Decision**: Use React Suspense boundaries with error.tsx and loading.tsx files

**Rationale**:
- Next.js App Router provides built-in error and loading UI support
- Suspense boundaries prevent entire page crashes
- loading.tsx shows during async operations (automatic)
- error.tsx catches errors and provides recovery options
- Consistent error handling across all routes

**Implementation Pattern**:
```typescript
// app/(protected)/dashboard/loading.tsx
export default function Loading() {
  return (
    <div className="flex items-center justify-center min-h-screen">
      <LoadingSpinner />
    </div>
  )
}

// app/(protected)/dashboard/error.tsx
"use client"

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  return (
    <div className="flex flex-col items-center justify-center min-h-screen">
      <h2 className="text-2xl font-bold mb-4">Something went wrong!</h2>
      <p className="text-gray-600 mb-4">{error.message}</p>
      <button onClick={reset} className="btn-primary">
        Try again
      </button>
    </div>
  )
}
```

**Alternatives Considered**:
- Try-catch in every component: Duplicated logic, easy to miss
- Global error boundary only: Less granular error recovery
- Toast notifications only: No fallback UI for critical errors

---

### 9. Testing Strategy (NEEDS CLARIFICATION Resolution)

**Decision**: Manual testing only (no automated tests in initial implementation)

**Rationale**:
- Specification does not explicitly require automated tests
- Manual testing sufficient for hackathon evaluation
- Focus on feature completeness over test coverage
- Automated tests can be added in future iterations if needed

**Manual Testing Checklist**:
- [ ] User can sign up with email and password
- [ ] User can sign in with correct credentials
- [ ] Unauthenticated users redirected to signin
- [ ] Authenticated users can view task dashboard
- [ ] Users can create new tasks
- [ ] Users can edit existing tasks
- [ ] Users can toggle task completion
- [ ] Users can delete tasks with confirmation
- [ ] Users can log out
- [ ] UI responsive on mobile (320px+), tablet, and desktop
- [ ] Form validation shows appropriate error messages
- [ ] Loading indicators shown during async operations
- [ ] Error messages displayed when operations fail

**Future Considerations**:
- Jest + React Testing Library for unit tests
- Playwright or Cypress for E2E tests
- Visual regression testing with Percy or Chromatic

---

## Summary of Key Decisions

| Area | Decision | Rationale |
|------|----------|-----------|
| Authentication | Better Auth with JWT | Built-in Next.js support, automatic token refresh |
| Token Storage | httpOnly cookies | XSS protection, automatic request attachment |
| Route Protection | Next.js middleware | Centralized, runs before rendering |
| API Client | Axios with interceptors | Automatic JWT attachment, error handling |
| Form Validation | Zod + react-hook-form | Type-safe, reusable schemas |
| Styling | Tailwind CSS | Mobile-first, utility classes, small bundle |
| State Management | React hooks | Simple CRUD, no global state needed |
| Error Handling | Suspense + error.tsx | Built-in Next.js support, granular recovery |
| Testing | Manual testing | Not required in spec, focus on features |

---

## Dependencies

**Production Dependencies**:
- next@^16.0.0
- react@^19.0.0
- react-dom@^19.0.0
- better-auth@latest
- axios@^1.6.0
- zod@^3.22.0
- react-hook-form@^7.49.0
- @hookform/resolvers@^3.3.0
- tailwindcss@^3.4.0

**Development Dependencies**:
- typescript@^5.3.0
- @types/node@^20.0.0
- @types/react@^19.0.0
- @types/react-dom@^19.0.0
- eslint@^8.56.0
- eslint-config-next@^16.0.0

---

## Open Questions

None - all technical unknowns have been resolved through research.
