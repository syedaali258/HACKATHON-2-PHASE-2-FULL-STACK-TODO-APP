# Quickstart Guide: Multi-User Todo Full-Stack Web Application

**Feature**: 002-multi-user-todo-app
**Date**: 2026-02-08
**Purpose**: Step-by-step guide for setting up and running the application

## Prerequisites

- Python 3.11+ installed
- Node.js 18+ and npm installed
- Neon PostgreSQL database account (free tier available at neon.tech)
- Git installed

## Environment Setup

### 1. Clone Repository

```bash
git clone <repository-url>
cd HACKATHON-PHASE-2
git checkout 002-multi-user-todo-app
```

### 2. Create Environment Variables

Create a `.env` file in the project root:

```bash
# Shared JWT Secret (generate a secure random string)
BETTER_AUTH_SECRET=your-super-secret-jwt-key-min-32-chars

# Neon PostgreSQL Connection
DATABASE_URL=postgresql+asyncpg://user:password@host/database?ssl=require

# Backend Configuration
BACKEND_PORT=8000
BACKEND_HOST=0.0.0.0

# Frontend Configuration
NEXT_PUBLIC_API_URL=http://localhost:8000
```

**Generate JWT Secret**:
```bash
# Using Python
python -c "import secrets; print(secrets.token_urlsafe(32))"

# Using OpenSSL
openssl rand -base64 32
```

**Get Neon Database URL**:
1. Sign up at https://neon.tech
2. Create a new project
3. Copy the connection string from the dashboard
4. Replace `postgresql://` with `postgresql+asyncpg://` for async support
5. Ensure `?ssl=require` is appended

### 3. Backend Setup

```bash
cd backend

# Create virtual environment
python -m venv venv

# Activate virtual environment
# On Windows:
venv\Scripts\activate
# On macOS/Linux:
source venv/bin/activate

# Install dependencies
pip install fastapi uvicorn sqlmodel asyncpg python-jose[cryptography] passlib[bcrypt] python-multipart

# Create requirements.txt
pip freeze > requirements.txt
```

### 4. Frontend Setup

```bash
cd frontend

# Install dependencies
npm install next@latest react@latest react-dom@latest
npm install better-auth
npm install -D typescript @types/node @types/react @types/react-dom
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p

# Create TypeScript config
npx tsc --init
```

## Database Initialization

### 1. Create Database Tables

The application will automatically create tables on first run using SQLModel's `create_all()` method.

Alternatively, run this Python script to initialize the database:

```python
# backend/init_db.py
import asyncio
from sqlmodel import SQLModel
from src.core.database import engine

async def init_db():
    async with engine.begin() as conn:
        await conn.run_sync(SQLModel.metadata.create_all)
    print("Database tables created successfully!")

if __name__ == "__main__":
    asyncio.run(init_db())
```

Run:
```bash
cd backend
python init_db.py
```

### 2. Verify Tables

Connect to your Neon database and verify tables exist:
- `users` table with columns: id, email, hashed_password, created_at, updated_at
- `tasks` table with columns: id, user_id, title, description, is_complete, created_at, updated_at

## Running the Application

### 1. Start Backend Server

```bash
cd backend
source venv/bin/activate  # or venv\Scripts\activate on Windows

# Run with uvicorn
uvicorn src.main:app --reload --host 0.0.0.0 --port 8000
```

Backend will be available at: http://localhost:8000

**Verify Backend**:
- Visit http://localhost:8000/docs for interactive API documentation (Swagger UI)
- Visit http://localhost:8000/redoc for alternative API documentation

### 2. Start Frontend Server

Open a new terminal:

```bash
cd frontend

# Run development server
npm run dev
```

Frontend will be available at: http://localhost:3000

## Testing the Application

### 1. User Registration Flow

1. Navigate to http://localhost:3000
2. Click "Sign Up" or navigate to http://localhost:3000/signup
3. Enter email and password (minimum 8 characters)
4. Submit form
5. You should be automatically signed in and redirected to /tasks

**Expected Behavior**:
- Account created in database
- JWT token issued and stored
- Redirected to task list page

### 2. User Sign In Flow

1. Navigate to http://localhost:3000/signin
2. Enter registered email and password
3. Submit form
4. You should be signed in and redirected to /tasks

**Expected Behavior**:
- Credentials verified against database
- JWT token issued and stored
- Redirected to task list page

### 3. Task Management Flow

**Create Task**:
1. On /tasks page, find the "Create Task" form
2. Enter task title (required)
3. Optionally enter description
4. Submit form
5. Task should appear in the list immediately

**View Tasks**:
1. All your tasks are displayed on /tasks page
2. Tasks are sorted by creation date (newest first)
3. Use filter dropdown to show: All, Complete, or Incomplete tasks

**Update Task**:
1. Click "Edit" button on a task
2. Modify title or description
3. Submit changes
4. Task should update immediately in the list

**Mark Complete/Incomplete**:
1. Click checkbox or "Complete" button on a task
2. Task status should toggle
3. Visual distinction should appear (e.g., strikethrough for completed tasks)

**Delete Task**:
1. Click "Delete" button on a task
2. Confirm deletion (if confirmation dialog implemented)
3. Task should be removed from the list immediately

### 4. Data Isolation Testing

**Critical Security Test**:

1. Create Account A (user1@example.com)
2. Sign in as Account A
3. Create several tasks
4. Sign out
5. Create Account B (user2@example.com)
6. Sign in as Account B
7. Verify you see NO tasks from Account A
8. Create tasks as Account B
9. Sign out and sign back in as Account A
10. Verify you still see only Account A's tasks

**Expected Behavior**:
- Each user sees only their own tasks
- No cross-user data leakage
- Task counts are independent per user

### 5. Authentication Testing

**Test Unauthorized Access**:

1. Sign out (clear JWT token)
2. Try to access http://localhost:3000/tasks directly
3. You should be redirected to /signin

**Test Token Expiration** (if implemented):

1. Sign in
2. Wait for token to expire (1 hour if using recommended settings)
3. Try to perform any task operation
4. You should be prompted to sign in again

## API Testing with cURL

### Signup

```bash
curl -X POST http://localhost:8000/api/auth/signup \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"SecurePass123"}'
```

### Signin

```bash
curl -X POST http://localhost:8000/api/auth/signin \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"SecurePass123"}'
```

Save the token from the response.

### Create Task

```bash
curl -X POST http://localhost:8000/api/tasks \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -d '{"title":"Test Task","description":"This is a test"}'
```

### List Tasks

```bash
curl -X GET http://localhost:8000/api/tasks \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

### Update Task

```bash
curl -X PUT http://localhost:8000/api/tasks/TASK_ID \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -d '{"title":"Updated Task","description":"Updated description"}'
```

### Toggle Complete

```bash
curl -X PATCH http://localhost:8000/api/tasks/TASK_ID/complete \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

### Delete Task

```bash
curl -X DELETE http://localhost:8000/api/tasks/TASK_ID \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

## Troubleshooting

### Backend Issues

**Database Connection Error**:
- Verify DATABASE_URL is correct
- Check Neon dashboard for connection string
- Ensure `?ssl=require` is appended
- Verify network connectivity to Neon

**JWT Verification Error**:
- Ensure BETTER_AUTH_SECRET is the same in frontend and backend
- Verify token is being sent in Authorization header
- Check token hasn't expired

**Import Errors**:
- Verify all dependencies are installed: `pip install -r requirements.txt`
- Ensure virtual environment is activated

### Frontend Issues

**API Connection Error**:
- Verify backend is running on port 8000
- Check NEXT_PUBLIC_API_URL in .env.local
- Verify CORS is enabled in FastAPI (if needed)

**Authentication Not Working**:
- Check browser console for errors
- Verify Better Auth is configured correctly
- Ensure JWT token is being stored (check browser cookies/localStorage)

**Build Errors**:
- Delete node_modules and package-lock.json
- Run `npm install` again
- Verify Node.js version is 18+

### Common Errors

**401 Unauthorized**:
- Token is missing, invalid, or expired
- Sign in again to get a new token

**403 Forbidden**:
- Attempting to access another user's task
- Verify you're signed in as the correct user

**404 Not Found**:
- Task ID doesn't exist
- Verify the task ID in the URL

**422 Validation Error**:
- Request body doesn't match expected schema
- Check API documentation for required fields

## Development Workflow

### Making Changes

1. Make code changes in backend or frontend
2. Backend: Server auto-reloads with `--reload` flag
3. Frontend: Next.js auto-reloads on file changes
4. Test changes manually or with cURL
5. Verify data isolation is maintained

### Adding New Features

1. Update spec.md with new requirements
2. Run /sp.plan to update plan.md
3. Run /sp.tasks to generate implementation tasks
4. Follow Agentic Dev Stack workflow

## Production Deployment

### Backend Deployment

1. Set environment variables in production environment
2. Use production-grade ASGI server (e.g., Gunicorn with Uvicorn workers)
3. Enable HTTPS (required for JWT security)
4. Configure CORS for frontend domain
5. Set up monitoring and logging

### Frontend Deployment

1. Build production bundle: `npm run build`
2. Deploy to Vercel, Netlify, or similar platform
3. Set environment variables (NEXT_PUBLIC_API_URL, BETTER_AUTH_SECRET)
4. Ensure HTTPS is enabled
5. Configure domain and DNS

### Security Checklist

- [ ] HTTPS enabled on both frontend and backend
- [ ] BETTER_AUTH_SECRET is strong and secret (32+ characters)
- [ ] Database credentials are secure and not exposed
- [ ] CORS is configured to allow only trusted domains
- [ ] JWT token expiration is set appropriately
- [ ] Password hashing is using bcrypt with sufficient work factor
- [ ] All API endpoints verify JWT tokens
- [ ] All database queries filter by user_id

## Success Criteria Validation

Verify the application meets all success criteria from spec.md:

- [ ] SC-001: Account creation + first task creation completes in under 3 minutes
- [ ] SC-002: Data isolation verified - no cross-user access possible
- [ ] SC-003: All HTTP status codes are correct (401, 403, 404, 422, 500)
- [ ] SC-004: All 5 todo operations work (create, read, update, complete, delete)
- [ ] SC-005: JWT tokens issued on signin and verified on every request
- [ ] SC-006: Task list updates immediately without page refresh
- [ ] SC-007: System handles 100 concurrent users (load testing required)
- [ ] SC-008: 95% of operations complete in under 2 seconds (performance testing required)
- [ ] SC-009: Zero security vulnerabilities (security audit required)
- [ ] SC-010: Spec-driven workflow demonstrated with clear traceability

## Next Steps

After verifying the quickstart guide works:

1. Run `/sp.tasks` to generate implementation tasks
2. Follow tasks.md to implement features in priority order (P1 → P2 → P3 → P4 → P5)
3. Validate each user story independently before proceeding to the next
4. Create PHRs for all implementation work
5. Run `/sp.git.commit_pr` to commit and create pull request
