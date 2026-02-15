# Build in Five — Skeleton Application Scaffold

## Purpose
This prompt generates a standardised full-stack application skeleton that matches the approved tech stack used across the Advisory Services application portfolio (IRP Adoption Tracker, Clara). Use this as the starting point for every Build in Five demo prototype and Application Factory app.

## How to use
1. Open Cursor with GSD installed
2. Run `/gsd/new-project`
3. When prompted for the project description, paste **Section 2 (The Prompt)** below
4. GSD will generate the scaffold as a structured plan — execute with `/gsd/execute`
5. Customise the generated app for your specific demo scenario

---

## 1. Tech Stack Reference

| Layer | Technology | Version | Notes |
|-------|-----------|---------|-------|
| **Backend** | FastAPI | 0.115.x | Async Python web framework |
| **Backend server** | Uvicorn | Latest | ASGI server |
| **ORM** | SQLAlchemy | 2.x | Async support via `asyncpg` |
| **Migrations** | Alembic | Latest | Auto-generate from models |
| **Database** | PostgreSQL | 15+ | Amazon RDS in production |
| **Frontend** | React | 18.x | Functional components + hooks |
| **Frontend build** | Vite | Latest | Fast dev server + build |
| **CSS** | Tailwind CSS | 3.x | Utility-first styling |
| **Auth** | Microsoft Entra ID | — | OAuth2/OIDC, JWT validation |
| **AI (optional)** | OpenAI API | Latest | For NLP features if needed |
| **Containerisation** | Docker | — | Multi-stage builds |
| **Reverse proxy** | Nginx | Alpine | Frontend serving in prod |
| **CI/CD** | GitHub Actions | — | Build → ECR → ECS |
| **Infrastructure** | Terraform | 1.5+ | Application Factory modules |
| **Cloud** | AWS ECS Fargate | — | awsvpc network mode |

---

## 2. The Prompt

Paste the following into Cursor / GSD:

```
You are scaffolding a new full-stack web application for the Moody's Advisory Services team. This application MUST use the exact tech stack defined below. Do not substitute any technologies — consistency across our application portfolio is critical for deployment to our shared AWS ECS environment.

## APPLICATION DETAILS
- Name: [APP_NAME]
- Description: [ONE_LINE_DESCRIPTION]
- Purpose: [BRIEF_PURPOSE]

## MANDATORY TECH STACK

### Backend (Python)
- **Framework**: FastAPI 0.115.x
- **Server**: Uvicorn (ASGI)
- **ORM**: SQLAlchemy 2.x with async support (asyncpg driver)
- **Migrations**: Alembic (auto-generate from SQLAlchemy models)
- **Database**: PostgreSQL 15+ (Amazon RDS in production)
- **Auth**: Microsoft Entra ID (Azure AD) via OAuth2/OIDC
  - JWT validation using `python-jose` or `authlib`
  - RBAC with configurable roles (Admin, Editor, Viewer as minimum)
  - Roles extracted from JWT `roles` claim (Entra ID App Roles)
  - All endpoints protected by default; public endpoints explicitly marked
- **API documentation**: Auto-generated via FastAPI (Swagger UI at /docs)
- **Environment config**: Pydantic Settings (BaseSettings) loading from environment variables
- **Logging**: Python `logging` module, structured JSON format for CloudWatch compatibility
- **Testing**: pytest + httpx (async test client)

### Frontend (JavaScript/TypeScript)
- **Framework**: React 18.x (functional components only, no class components)
- **Build tool**: Vite
- **CSS**: Tailwind CSS 3.x (utility classes only, no custom CSS files unless absolutely necessary)
- **Routing**: React Router v6
- **State management**: React hooks (useState, useReducer, useContext). No Redux unless explicitly justified.
- **HTTP client**: Axios with interceptors for auth token injection
- **Auth integration**: MSAL React (@azure/msal-react) for Entra ID SSO
  - Silent token acquisition with fallback to redirect
  - Token attached to all API requests via Axios interceptor
  - Protected routes wrapper component
- **Testing**: Vitest + React Testing Library

### Infrastructure
- **Containerisation**: Docker with multi-stage builds
  - Backend: python:3.11-slim base → install deps → copy app → run uvicorn
  - Frontend: node:20-alpine build stage → nginx:alpine serve stage
- **Docker Compose**: For local development (backend + frontend + postgres)
- **Reverse proxy**: Nginx (serves frontend static files + proxies /api/* to backend)

## PROJECT STRUCTURE

Generate the following directory structure:

```
[APP_NAME]/
├── backend/
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py                  # FastAPI app creation, CORS, middleware
│   │   ├── config.py                # Pydantic BaseSettings
│   │   ├── database.py              # Async SQLAlchemy engine + session
│   │   ├── models/
│   │   │   ├── __init__.py
│   │   │   └── base.py              # Base model with id, created_at, updated_at
│   │   ├── schemas/
│   │   │   └── __init__.py          # Pydantic request/response models
│   │   ├── api/
│   │   │   ├── __init__.py
│   │   │   ├── router.py            # Main API router aggregating all routes
│   │   │   ├── deps.py              # Dependencies: get_db, get_current_user, require_role
│   │   │   └── v1/
│   │   │       ├── __init__.py
│   │   │       └── health.py        # Health check endpoint
│   │   ├── auth/
│   │   │   ├── __init__.py
│   │   │   ├── entra.py             # Entra ID JWT validation + JWKS fetching
│   │   │   └── rbac.py              # Role-based access control decorators
│   │   └── services/
│   │       └── __init__.py          # Business logic layer
│   ├── alembic/
│   │   ├── env.py
│   │   └── versions/                # Migration scripts
│   ├── tests/
│   │   ├── conftest.py              # Test fixtures, async client setup
│   │   └── test_health.py
│   ├── alembic.ini
│   ├── requirements.txt
│   ├── Dockerfile
│   └── .env.example
├── frontend/
│   ├── src/
│   │   ├── main.jsx                 # React entry point
│   │   ├── App.jsx                  # App shell with router + auth provider
│   │   ├── auth/
│   │   │   ├── msalConfig.js        # MSAL configuration for Entra ID
│   │   │   ├── AuthProvider.jsx     # MSAL provider wrapper
│   │   │   └── ProtectedRoute.jsx   # Route guard checking auth state
│   │   ├── api/
│   │   │   └── client.js            # Axios instance with auth interceptor
│   │   ├── components/
│   │   │   ├── layout/
│   │   │   │   ├── AppLayout.jsx    # Main layout: nav + sidebar + content
│   │   │   │   ├── Navbar.jsx       # Top nav with user info + logout
│   │   │   │   └── Sidebar.jsx      # Side navigation
│   │   │   └── common/
│   │   │       ├── LoadingSpinner.jsx
│   │   │       └── ErrorBoundary.jsx
│   │   ├── pages/
│   │   │   ├── Dashboard.jsx        # Landing page placeholder
│   │   │   └── NotFound.jsx         # 404 page
│   │   └── hooks/
│   │       └── useApi.js            # Custom hook for API calls with loading/error state
│   ├── public/
│   │   └── favicon.ico
│   ├── index.html
│   ├── vite.config.js
│   ├── tailwind.config.js
│   ├── postcss.config.js
│   ├── package.json
│   ├── Dockerfile
│   └── nginx.conf                   # Nginx config for serving + API proxy
├── docker-compose.yml               # Local dev: backend + frontend + postgres
├── docker-compose.prod.yml          # Production-like: built images
├── .github/
│   └── workflows/
│       └── deploy.yml               # GitHub Actions: build → ECR → ECS
├── terraform/
│   ├── main.tf                      # Application Factory module reference
│   ├── variables.tf                 # App-specific variables
│   ├── terraform.tfvars.example     # Example variable values
│   └── outputs.tf                   # ALB DNS, ECS service name, etc.
├── .gitignore
├── README.md
└── factory-manifest.json            # Application Factory deployment manifest
```

## IMPLEMENTATION REQUIREMENTS

### backend/app/main.py
- Create FastAPI app with title, description, version
- Add CORS middleware (configurable origins from settings)
- Include API router
- Add startup/shutdown events for database connection
- Health check at GET /api/v1/health returning {"status": "healthy", "version": "x.x.x"}

### backend/app/config.py
- Pydantic BaseSettings loading from environment variables
- Required: DATABASE_URL, ENTRA_TENANT_ID, ENTRA_CLIENT_ID, ENTRA_AUDIENCE
- Optional: OPENAI_API_KEY, CORS_ORIGINS, LOG_LEVEL
- All secrets loaded from environment (AWS Secrets Manager in production)

### backend/app/database.py
- Async SQLAlchemy engine using asyncpg
- Async session factory
- get_db dependency yielding async sessions

### backend/app/auth/entra.py
- Fetch JWKS from Entra ID well-known endpoint
- Validate JWT: signature, audience, issuer, expiry
- Extract user info: oid, name, email, roles
- Cache JWKS keys with TTL (avoid fetching per request)

### backend/app/auth/rbac.py
- `require_role(*roles)` dependency that checks JWT roles claim
- `get_current_user` dependency returning user info from token
- Return 401 if no token, 403 if insufficient role
- Roles are strings matching Entra ID App Role values

### backend/app/api/deps.py
- `get_db` — async database session dependency
- `get_current_user` — extracts and validates user from Authorization header
- `require_role` — parameterised dependency for role checking

### frontend/src/auth/msalConfig.js
- MSAL configuration with Entra ID tenant and client IDs
- Redirect URI from environment variable
- Scopes: openid, profile, email, plus API scope

### frontend/src/api/client.js
- Axios instance with baseURL from environment
- Request interceptor: acquire token silently, attach as Bearer header
- Response interceptor: handle 401 (redirect to login), 403 (show forbidden)

### frontend/src/auth/ProtectedRoute.jsx
- Check MSAL authentication state
- Redirect to login if unauthenticated
- Optional role check prop for role-gated routes

### docker-compose.yml
- **backend**: build from ./backend, expose 8000, depends on postgres, env_file
- **frontend**: build from ./frontend, expose 3000, depends on backend
- **postgres**: postgres:15-alpine, volume for data persistence, health check

### Dockerfiles
- **Backend Dockerfile**: Multi-stage. python:3.11-slim. Copy requirements.txt first (layer caching). Install deps. Copy app. CMD: uvicorn app.main:app --host 0.0.0.0 --port 8000
- **Frontend Dockerfile**: Multi-stage. Stage 1: node:20-alpine, npm install, npm run build. Stage 2: nginx:alpine, copy build output to /usr/share/nginx/html, copy nginx.conf

### factory-manifest.json
```json
{
  "app_name": "[APP_NAME]",
  "owner": "[YOUR_NAME]",
  "team": "advisory-services",
  "services": {
    "frontend": {
      "dockerfile": "frontend/Dockerfile",
      "port": 80,
      "health_check_path": "/",
      "cpu": 256,
      "memory": 512
    },
    "backend": {
      "dockerfile": "backend/Dockerfile",
      "port": 8000,
      "health_check_path": "/api/v1/health",
      "cpu": 512,
      "memory": 1024
    }
  },
  "database": {
    "engine": "postgres",
    "version": "15"
  },
  "environment": {
    "required": [
      "DATABASE_URL",
      "ENTRA_TENANT_ID",
      "ENTRA_CLIENT_ID",
      "ENTRA_AUDIENCE"
    ],
    "optional": [
      "OPENAI_API_KEY",
      "CORS_ORIGINS",
      "LOG_LEVEL"
    ]
  }
}
```

### .github/workflows/deploy.yml
- Trigger on push to main
- Steps: checkout → configure AWS credentials (OIDC) → login to ECR → build + tag images (Git SHA) → push to ECR → update ECS task definition → deploy ECS service
- Environment variables from GitHub secrets

### README.md
- App name, description, purpose
- Tech stack summary (link to this document)
- Local development setup: `docker-compose up`
- Environment variables reference
- Deployment: "Deployed via Application Factory — see factory-manifest.json"

## CRITICAL CONSTRAINTS
- Do NOT use any technology not listed above
- Do NOT use class-based React components
- Do NOT use Redux or any external state management library
- Do NOT use CSS modules or styled-components — Tailwind only
- Do NOT use SQLite — PostgreSQL only, even in development (via Docker)
- Do NOT hardcode any secrets, URLs, or configuration — all from environment variables
- Do NOT use :latest Docker tags — always pin versions
- All API endpoints MUST be behind authentication by default
- All database access MUST go through SQLAlchemy ORM (no raw SQL unless explicitly justified)
- All frontend routes MUST be behind the ProtectedRoute wrapper by default

## OUTPUT
Generate all files listed in the project structure above with working, production-quality code. Every file should be complete and functional — no placeholder comments like "TODO" or "implement later". The application should start successfully with `docker-compose up` and display a working dashboard page behind Entra ID authentication.
```

---

## 3. Customisation Points

After generating the scaffold, customise these areas for your specific demo scenario:

| What to change | Where | Example |
|---------------|-------|---------|
| App name and description | `factory-manifest.json`, `README.md`, `backend/app/main.py` | "exposure-ingestion-demo" |
| Database models | `backend/app/models/` | Add Exposure, RiskScore, etc. |
| API endpoints | `backend/app/api/v1/` | Add domain-specific routes |
| Frontend pages | `frontend/src/pages/` | Add scenario-specific views |
| RBAC roles | `backend/app/auth/rbac.py` | Add domain-specific roles if needed |
| OpenAI integration | `backend/app/services/` | Add if NLP features needed |
| Terraform variables | `terraform/variables.tf` | App-specific infra config |

---

## 4. Quick Verification

After scaffold generation, verify:

```bash
# Start locally
docker-compose up --build

# Check backend health
curl http://localhost:8000/api/v1/health
# Expected: {"status": "healthy", "version": "0.1.0"}

# Check frontend loads
open http://localhost:3000
# Expected: Entra ID login redirect (or landing page if auth is mocked for local dev)

# Check API docs
open http://localhost:8000/docs
# Expected: Swagger UI with all endpoints listed
```
