# Quickstart Guide: Backend API & Database

**Feature**: Backend API & Database (Task Management Core)
**Date**: 2026-02-08
**Phase**: Phase 1 - Design & Contracts

## Overview

This guide helps you set up and run the FastAPI backend for the task management system. The backend provides secure, JWT-authenticated REST API endpoints with persistent storage in Neon Serverless PostgreSQL.

---

## Prerequisites

- **Python**: 3.11 or higher
- **Neon PostgreSQL**: Account and database created at [neon.tech](https://neon.tech)
- **Better Auth**: Authentication system configured and issuing JWT tokens
- **Git**: For version control
- **Code Editor**: VS Code, PyCharm, or similar

---

## Initial Setup

### 1. Clone Repository

```bash
cd HACKATHON-PHASE-2
```

### 2. Create Virtual Environment

**Windows (PowerShell)**:
```powershell
cd backend
python -m venv venv
.\venv\Scripts\Activate.ps1
```

**macOS/Linux**:
```bash
cd backend
python3 -m venv venv
source venv/bin/activate
```

### 3. Install Dependencies

Create `backend/requirements.txt`:
```txt
fastapi==0.109.0
uvicorn[standard]==0.27.0
sqlmodel==0.0.14
asyncpg==0.29.0
pydantic==2.5.3
python-jose[cryptography]==3.3.0
python-dotenv==1.0.0
alembic==1.13.1
pytest==7.4.4
pytest-asyncio==0.23.3
httpx==0.26.0
```

Install:
```bash
pip install -r requirements.txt
```

### 4. Configure Environment Variables

Create `backend/.env`:
```env
# Database
DATABASE_URL=postgresql+asyncpg://user:password@ep-xxx.neon.tech/dbname?sslmode=require

# JWT Authentication
BETTER_AUTH_SECRET=your-secret-key-here
JWT_ALGORITHM=HS256

# Application
ENVIRONMENT=development
DEBUG=true
```

**Important**:
- Replace `DATABASE_URL` with your Neon PostgreSQL connection string
- Replace `BETTER_AUTH_SECRET` with the same secret used by Better Auth
- Never commit `.env` to version control

Create `backend/.env.example` (for documentation):
```env
DATABASE_URL=postgresql+asyncpg://user:password@host/dbname
BETTER_AUTH_SECRET=your-secret-key
JWT_ALGORITHM=HS256
ENVIRONMENT=development
DEBUG=true
```

---

## Project Structure

```
backend/
├── app/
│   ├── __init__.py
│   ├── main.py              # FastAPI application entry point
│   ├── config.py            # Configuration and environment variables
│   ├── database.py          # Database connection and session management
│   ├── models/
│   │   ├── __init__.py
│   │   └── task.py          # SQLModel Task entity
│   ├── schemas/
│   │   ├── __init__.py
│   │   └── task.py          # Pydantic request/response schemas
│   ├── api/
│   │   ├── __init__.py
│   │   ├── deps.py          # Dependency injection (JWT verification)
│   │   └── routes/
│   │       ├── __init__.py
│   │       └── tasks.py     # Task CRUD endpoints
│   └── core/
│       ├── __init__.py
│       └── security.py      # JWT verification utilities
├── tests/
│   ├── __init__.py
│   ├── conftest.py          # Pytest fixtures
│   ├── test_auth.py         # Authentication tests
│   ├── test_tasks.py        # Task CRUD tests
│   └── test_isolation.py    # Data isolation tests
├── alembic/                 # Database migrations
├── requirements.txt         # Python dependencies
├── .env                     # Environment variables (not in git)
├── .env.example             # Environment template
└── README.md                # Backend documentation
```

---

## Database Setup

### 1. Get Neon Connection String

1. Log in to [Neon Console](https://console.neon.tech)
2. Select your project
3. Copy the connection string from the dashboard
4. Replace `postgresql://` with `postgresql+asyncpg://` for async support
5. Add to `.env` as `DATABASE_URL`

### 2. Initialize Database

The database tables will be created automatically when you run the application for the first time (using SQLModel's `create_all`).

**For production**, use Alembic migrations:

```bash
# Initialize Alembic (first time only)
alembic init alembic

# Create migration
alembic revision --autogenerate -m "Create tasks table"

# Apply migration
alembic upgrade head
```

---

## Running the Application

### Development Server

```bash
cd backend
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

**Options**:
- `--reload`: Auto-reload on code changes
- `--host 0.0.0.0`: Accept connections from any IP
- `--port 8000`: Run on port 8000

**Output**:
```
INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
INFO:     Started reloader process [12345] using StatReload
INFO:     Started server process [12346]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
```

### Verify Server is Running

Open browser to: http://localhost:8000/docs

You should see the FastAPI interactive API documentation (Swagger UI).

---

## Testing the API

### 1. Get a JWT Token

**Option A**: Use Better Auth to authenticate and get a token

**Option B**: For testing, create a test token:

```python
from jose import jwt
from datetime import datetime, timedelta

secret = "your-secret-key"
payload = {
    "sub": "test_user_123",  # user_id
    "exp": datetime.utcnow() + timedelta(hours=1)
}
token = jwt.encode(payload, secret, algorithm="HS256")
print(token)
```

### 2. Test Endpoints with curl

**Create a task**:
```bash
curl -X POST http://localhost:8000/api/tasks \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Buy groceries",
    "description": "Milk, eggs, bread",
    "is_completed": false
  }'
```

**List all tasks**:
```bash
curl -X GET http://localhost:8000/api/tasks \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

**Get a single task**:
```bash
curl -X GET http://localhost:8000/api/tasks/1 \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

**Update a task**:
```bash
curl -X PUT http://localhost:8000/api/tasks/1 \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Buy groceries and cook dinner",
    "is_completed": true
  }'
```

**Delete a task**:
```bash
curl -X DELETE http://localhost:8000/api/tasks/1 \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

### 3. Test with Swagger UI

1. Open http://localhost:8000/docs
2. Click "Authorize" button
3. Enter: `Bearer YOUR_JWT_TOKEN`
4. Click "Authorize"
5. Try any endpoint by clicking "Try it out"

---

## Running Tests

### Run All Tests

```bash
cd backend
pytest
```

### Run Specific Test Files

```bash
# Authentication tests
pytest tests/test_auth.py

# Task CRUD tests
pytest tests/test_tasks.py

# Data isolation tests
pytest tests/test_isolation.py
```

### Run with Coverage

```bash
pytest --cov=app --cov-report=html
```

View coverage report: `open htmlcov/index.html`

### Test Output Example

```
============================= test session starts ==============================
collected 15 items

tests/test_auth.py ....                                                  [ 26%]
tests/test_tasks.py .......                                              [ 73%]
tests/test_isolation.py ....                                             [100%]

============================== 15 passed in 2.34s ===============================
```

---

## Common Issues & Solutions

### Issue: Database Connection Failed

**Error**: `asyncpg.exceptions.InvalidPasswordError`

**Solution**:
- Verify `DATABASE_URL` in `.env` is correct
- Check Neon dashboard for correct credentials
- Ensure connection string uses `postgresql+asyncpg://`

### Issue: JWT Verification Failed

**Error**: `401 Unauthorized - Invalid or expired token`

**Solution**:
- Verify `BETTER_AUTH_SECRET` matches the auth service
- Check token hasn't expired
- Ensure token is in format: `Bearer <token>`

### Issue: Import Errors

**Error**: `ModuleNotFoundError: No module named 'fastapi'`

**Solution**:
- Activate virtual environment: `source venv/bin/activate`
- Install dependencies: `pip install -r requirements.txt`

### Issue: Port Already in Use

**Error**: `OSError: [Errno 48] Address already in use`

**Solution**:
- Kill process using port 8000: `lsof -ti:8000 | xargs kill -9`
- Or use different port: `uvicorn app.main:app --port 8001`

---

## Development Workflow

### 1. Make Code Changes

Edit files in `backend/app/`

### 2. Server Auto-Reloads

With `--reload` flag, server automatically restarts on file changes.

### 3. Test Changes

```bash
# Run tests
pytest

# Or test manually with curl/Swagger UI
```

### 4. Commit Changes

```bash
git add .
git commit -m "Add task update endpoint"
```

---

## API Documentation

### Interactive Docs (Swagger UI)
- URL: http://localhost:8000/docs
- Features: Try endpoints, see schemas, authentication

### Alternative Docs (ReDoc)
- URL: http://localhost:8000/redoc
- Features: Clean layout, better for reading

### OpenAPI Schema
- URL: http://localhost:8000/openapi.json
- Features: Machine-readable API specification

---

## Environment Variables Reference

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `DATABASE_URL` | Yes | - | Neon PostgreSQL connection string |
| `BETTER_AUTH_SECRET` | Yes | - | JWT verification secret |
| `JWT_ALGORITHM` | No | HS256 | JWT signing algorithm |
| `ENVIRONMENT` | No | development | Environment name |
| `DEBUG` | No | false | Enable debug mode |

---

## Next Steps

1. **Implement Authentication**: Integrate with Better Auth for user signup/signin
2. **Add Frontend**: Build Next.js frontend to consume this API
3. **Deploy**: Deploy to production (Vercel, Railway, etc.)
4. **Monitor**: Add logging and monitoring
5. **Scale**: Optimize for production load

---

## Additional Resources

- **FastAPI Documentation**: https://fastapi.tiangolo.com/
- **SQLModel Documentation**: https://sqlmodel.tiangolo.com/
- **Neon Documentation**: https://neon.tech/docs/
- **Pydantic Documentation**: https://docs.pydantic.dev/
- **JWT Best Practices**: https://tools.ietf.org/html/rfc8725

---

## Support

For issues or questions:
1. Check the [API documentation](http://localhost:8000/docs)
2. Review the [specification](./spec.md)
3. Check the [implementation plan](./plan.md)
4. Review test files for usage examples

---

## Quick Reference

**Start server**: `uvicorn app.main:app --reload`
**Run tests**: `pytest`
**View docs**: http://localhost:8000/docs
**API base URL**: http://localhost:8000/api
