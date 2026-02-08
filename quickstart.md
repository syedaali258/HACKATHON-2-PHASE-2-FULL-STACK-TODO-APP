# Quickstart Guide: Frontend Application (Next.js App Router)

**Feature**: 004-nextjs-frontend
**Date**: 2026-02-08
**Purpose**: Setup and development instructions for the Next.js frontend application

## Prerequisites

- Node.js 20.x or higher
- npm or yarn package manager
- Backend API running at `http://localhost:8000` (see backend README)
- Better Auth configured and operational
- Modern web browser (Chrome, Firefox, Safari, or Edge)

---

## Initial Setup

### 1. Install Dependencies

```bash
cd frontend
npm install
```

**Dependencies installed**:
- next@^16.0.0
- react@^19.0.0
- react-dom@^19.0.0
- better-auth@latest
- axios@^1.6.0
- zod@^3.22.0
- react-hook-form@^7.49.0
- @hookform/resolvers@^3.3.0
- tailwindcss@^3.4.0
- typescript@^5.3.0

### 2. Configure Environment Variables

Create `.env.local` file in the `frontend/` directory:

```bash
# Copy from example
cp .env.example .env.local
```

Edit `.env.local` with your configuration:

```bash
# Backend API URL
NEXT_PUBLIC_API_URL=http://localhost:8000

# Better Auth URL (usually same as frontend URL)
NEXT_PUBLIC_BETTER_AUTH_URL=http://localhost:3000

# Environment
NODE_ENV=development
```

**Important**: Never commit `.env.local` to version control (already in .gitignore)

### 3. Verify Backend Connection

Ensure the backend API is running and accessible:

```bash
# Test backend health endpoint
curl http://localhost:8000/health
```

Expected response:
```json
{"status": "healthy"}
```

---

## Development

### Start Development Server

```bash
npm run dev
```

The application will be available at: **http://localhost:3000**

**What happens on startup**:
1. Next.js compiles the application
2. Development server starts on port 3000
3. Hot module replacement (HMR) enabled
4. TypeScript type checking in watch mode

### Development Workflow

1. **Make changes** to files in `app/`, `components/`, or `lib/`
2. **Save the file** - changes appear automatically (HMR)
3. **Check browser** - see updates without manual refresh
4. **Check terminal** - see TypeScript errors and warnings

### Available Scripts

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server (after build)
npm start

# Run TypeScript type checking
npm run type-check

# Run ESLint
npm run lint

# Format code with Prettier (if configured)
npm run format
```

---

## Testing the Application

### Manual Testing Checklist

#### Authentication Flow
```bash
# 1. Visit http://localhost:3000
# Expected: Redirect to /signin

# 2. Click "Sign up" link
# Expected: Navigate to /signup

# 3. Fill in signup form:
#    - Email: test@example.com
#    - Password: password123
#    - Name: Test User (optional)
# Expected: Account created, auto sign in, redirect to /dashboard

# 4. Click "Logout"
# Expected: Sign out, redirect to /signin

# 5. Sign in with credentials
# Expected: Redirect to /dashboard
```

#### Task Management Flow
```bash
# 1. On dashboard, click "Create Task"
# Expected: Show create task form

# 2. Fill in task details:
#    - Title: "Complete project"
#    - Description: "Finish the hackathon" (optional)
# Expected: Task created, appears in list immediately

# 3. Click completion checkbox
# Expected: Task marked as complete, visual update

# 4. Click "Edit" on task
# Expected: Show edit modal with current values

# 5. Update task title
# Expected: Changes saved, modal closes, list updates

# 6. Click "Delete" on task
# Expected: Confirmation dialog appears

# 7. Confirm deletion
# Expected: Task removed from list
```

#### Responsive Design Testing
```bash
# 1. Open browser DevTools (F12)
# 2. Toggle device toolbar (Ctrl+Shift+M)
# 3. Test different screen sizes:
#    - Mobile: 320px, 375px, 414px
#    - Tablet: 768px, 1024px
#    - Desktop: 1280px, 1920px
# Expected: Layout adapts, all features accessible
```

#### Error Handling Testing
```bash
# 1. Stop backend API server
# 2. Try to create a task
# Expected: Error message "Failed to create task"

# 3. Restart backend API
# 4. Click retry
# Expected: Task created successfully

# 5. Sign in with wrong password
# Expected: Error message "Invalid credentials"

# 6. Try to create task with empty title
# Expected: Validation error "Title is required"
```

---

## API Integration Testing

### Test API Client with curl

```bash
# 1. Sign up a user (via Better Auth)
curl -X POST http://localhost:3000/api/auth/signup \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"password123"}'

# 2. Sign in to get JWT token
curl -X POST http://localhost:3000/api/auth/signin \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"password123"}' \
  -c cookies.txt

# 3. Create a task (with JWT from cookies)
curl -X POST http://localhost:8000/api/tasks \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -d '{"title":"Test task","description":"Testing API"}'

# 4. List all tasks
curl -X GET http://localhost:8000/api/tasks \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"

# 5. Update a task
curl -X PUT http://localhost:8000/api/tasks/1 \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -d '{"is_completed":true}'

# 6. Delete a task
curl -X DELETE http://localhost:8000/api/tasks/1 \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

---

## Project Structure Overview

```
frontend/
├── app/                      # Next.js App Router
│   ├── (auth)/              # Public auth pages
│   │   ├── signin/page.tsx
│   │   └── signup/page.tsx
│   ├── (protected)/         # Protected pages
│   │   └── dashboard/page.tsx
│   ├── layout.tsx           # Root layout
│   ├── page.tsx             # Landing page
│   └── globals.css          # Global styles
├── components/              # React components
│   ├── auth/               # Auth components
│   ├── tasks/              # Task components
│   └── ui/                 # Reusable UI components
├── lib/                     # Utilities and helpers
│   ├── api/                # API client
│   ├── auth/               # Better Auth config
│   └── utils/              # Validation, helpers
├── types/                   # TypeScript types
├── middleware.ts            # Route protection
├── .env.local              # Environment variables (gitignored)
├── .env.example            # Environment template
├── next.config.js          # Next.js config
├── tailwind.config.js      # Tailwind config
├── tsconfig.json           # TypeScript config
└── package.json            # Dependencies
```

---

## Common Issues and Solutions

### Issue: "Cannot connect to backend API"

**Symptoms**: API calls fail with network errors

**Solutions**:
1. Verify backend is running: `curl http://localhost:8000/health`
2. Check `NEXT_PUBLIC_API_URL` in `.env.local`
3. Verify CORS is configured in backend (allow `http://localhost:3000`)
4. Check browser console for CORS errors

### Issue: "Authentication not working"

**Symptoms**: Always redirected to signin, even after login

**Solutions**:
1. Check Better Auth configuration in `lib/auth/better-auth.ts`
2. Verify `NEXT_PUBLIC_BETTER_AUTH_URL` in `.env.local`
3. Check browser cookies (should see `better-auth.session_token`)
4. Clear browser cookies and try again
5. Check backend JWT verification (shared secret must match)

### Issue: "TypeScript errors"

**Symptoms**: Red squiggly lines in editor, build fails

**Solutions**:
1. Run `npm run type-check` to see all errors
2. Ensure all types are imported correctly
3. Check `tsconfig.json` is properly configured
4. Restart TypeScript server in VS Code (Cmd+Shift+P → "Restart TS Server")

### Issue: "Styles not applying"

**Symptoms**: Tailwind classes not working

**Solutions**:
1. Verify `tailwind.config.js` content paths include all component files
2. Check `globals.css` imports Tailwind directives
3. Restart dev server (`npm run dev`)
4. Clear `.next` cache: `rm -rf .next && npm run dev`

### Issue: "Hot reload not working"

**Symptoms**: Changes don't appear without manual refresh

**Solutions**:
1. Check file is saved
2. Restart dev server
3. Check for syntax errors in terminal
4. Try hard refresh in browser (Ctrl+Shift+R)

---

## Development Best Practices

### Code Organization

1. **Components**: One component per file, named exports
2. **Types**: Define in `types/` directory, import where needed
3. **API calls**: Centralize in `lib/api/`, don't call axios directly in components
4. **Validation**: Define Zod schemas in `lib/utils/validation.ts`
5. **Styles**: Use Tailwind utility classes, avoid custom CSS

### TypeScript

1. **Enable strict mode**: Already configured in `tsconfig.json`
2. **Avoid `any` type**: Use proper types or `unknown`
3. **Use type inference**: Let TypeScript infer types from Zod schemas
4. **Export types**: Make types reusable across components

### Performance

1. **Use Server Components**: Default in App Router, faster initial load
2. **Client Components only when needed**: For interactivity (forms, buttons)
3. **Optimize images**: Use Next.js `<Image>` component
4. **Lazy load modals**: Use dynamic imports for heavy components

### Security

1. **Never expose secrets**: Use environment variables, never commit `.env.local`
2. **Validate user input**: Use Zod schemas before API calls
3. **Handle errors gracefully**: Don't expose sensitive error details to users
4. **Use httpOnly cookies**: For JWT token storage (XSS protection)

---

## Deployment Preparation

### Build for Production

```bash
# Create optimized production build
npm run build

# Test production build locally
npm start
```

### Environment Variables for Production

Update `.env.production` (or deployment platform settings):

```bash
NEXT_PUBLIC_API_URL=https://api.yourdomain.com
NEXT_PUBLIC_BETTER_AUTH_URL=https://yourdomain.com
NODE_ENV=production
```

### Pre-deployment Checklist

- [ ] All environment variables configured for production
- [ ] Backend API accessible from production frontend URL
- [ ] CORS configured to allow production frontend origin
- [ ] HTTPS enabled (required for secure cookies)
- [ ] Error tracking configured (e.g., Sentry)
- [ ] Analytics configured (if needed)
- [ ] All manual tests passing
- [ ] TypeScript build succeeds (`npm run build`)
- [ ] No console errors in production build

---

## Additional Resources

### Documentation

- [Next.js 16 Documentation](https://nextjs.org/docs)
- [React 19 Documentation](https://react.dev)
- [Better Auth Documentation](https://better-auth.com/docs)
- [Tailwind CSS Documentation](https://tailwindcss.com/docs)
- [Zod Documentation](https://zod.dev)

### Related Files

- Backend API: `backend/README.md`
- Feature Specification: `specs/004-nextjs-frontend/spec.md`
- Implementation Plan: `specs/004-nextjs-frontend/plan.md`
- Data Model: `specs/004-nextjs-frontend/data-model.md`
- API Contracts: `specs/004-nextjs-frontend/contracts/`

---

## Support

For issues or questions:
1. Check this quickstart guide
2. Review the feature specification
3. Check backend API documentation
4. Review Better Auth documentation
5. Check browser console for errors
6. Check terminal for build errors

---

**Last Updated**: 2026-02-08
