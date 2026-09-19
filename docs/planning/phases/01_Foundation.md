# Phase 1: Foundation & Infrastructure Boilerplate

## Objective
Establish the core repository structure, configure the Next.js frontend and FastAPI backend, and set up the fundamental development environment. This phase ensures that any developer can clone the repository, run a single command, and have both services communicating successfully.

## 1. Directory Structure & Environment Variables
The repository will be a monorepo containing both the frontend and backend. 

### Environment Variables
**Frontend (`frontend/.env.local`)**:
```env
NEXT_PUBLIC_API_URL=http://localhost:8000
NEXT_PUBLIC_SUPABASE_URL=<supabase_project_url>
NEXT_PUBLIC_SUPABASE_ANON_KEY=<supabase_anon_key>
```

**Backend (`backend/.env`)**:
```env
SUPABASE_URL=<supabase_project_url>
SUPABASE_SERVICE_KEY=<supabase_service_key>
FRONTEND_URL=http://localhost:3000
LITELLM_MASTER_KEY=<optional_litellm_key>
```

## 2. Frontend Implementation (Next.js)
**Tech Stack**: Next.js 15 (App Router), TailwindCSS, TypeScript, TanStack Query.

### Tasks:
1. **Initialize Next.js**:
   - `bunx create-next-app@latest frontend --typescript --tailwind --eslint --app`
2. **TanStack Query Setup**:
   - `bun add @tanstack/react-query`
   - Create `src/app/providers.tsx` to instantiate `QueryClient`.
   - Wrap `{children}` in `src/app/layout.tsx` with `<Providers>`.
3. **API Client Setup (Axios/Fetch)**:
   - Create `src/lib/api.ts`.
   - Configure a base Axios instance pointing to `NEXT_PUBLIC_API_URL`.
   - Include interceptors for injecting Supabase Auth tokens.
4. **Standalone Output**:
   - Update `next.config.ts` to include `output: 'standalone'` for future Dockerization.

## 3. Backend Implementation (FastAPI)
**Tech Stack**: Python 3.12+, FastAPI, Uvicorn, Supabase Python Client.

### Tasks:
1. **Initialize Virtual Environment**:
   - `uv venv`
   - `source .venv/bin/activate`
2. **Install Dependencies**:
   - `uv pip install fastapi uvicorn pydantic supabase python-dotenv`
   - Freeze dependencies: `uv pip freeze > requirements.txt`
3. **Core Application Setup (`main.py`)**:
   - Initialize `FastAPI(title="Resumify API")`.
   - Configure CORS middleware to allow requests from `FRONTEND_URL` (localhost:3000 during dev).
4. **Health Check Endpoint**:
   - Create a `GET /health` endpoint that returns `{"status": "ok", "db_status": "connected"}`.
   - The `db_status` should verify the Supabase connection using the Supabase client.

## 4. Supabase Setup & Basic Auth Routing
1. **Supabase Client**:
   - Frontend: Install `@supabase/supabase-js`. Create `src/lib/supabase.ts` for the client instance.
   - Backend: Create `app/core/database.py` to instantiate the Supabase Python client.
2. **Auth Middleware (FastAPI)**:
   - Create `app/api/dependencies/auth.py`.
   - Implement a `get_current_user` dependency that reads the `Authorization: Bearer <token>` header, verifies it via Supabase Auth, and raises `401 Unauthorized` if invalid.

## 5. Acceptance Criteria
- [ ] Frontend runs on `:3000` and displays a basic layout.
- [ ] Backend runs on `:8000/docs` exposing the Swagger UI.
- [ ] Frontend can successfully fetch data from the `GET /health` FastAPI endpoint.
- [ ] Supabase clients are instantiated securely in both environments using `.env`.
- [ ] A protected route in FastAPI successfully rejects requests without a valid Supabase JWT.
