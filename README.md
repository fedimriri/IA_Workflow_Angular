# IA Workflow

IA Workflow is a full-stack platform for designing, validating, and running AI agent workflows. The repository is organized as a monorepo with a shared Node.js backend, an Angular frontend, and an additional React/Vite frontend implementation that uses the same API contract.

## Overview

The application lets users:

- authenticate with email and password
- browse and manage AI agents
- design workflows as node/edge graphs
- validate workflows before execution
- launch runs and monitor them in real time
- review KPI dashboards and execution history

## Architecture

| Layer | Location | Purpose |
| --- | --- | --- |
| Backend API | `backend_Node/backend` | Express + Prisma service exposing auth, agents, workflows, runs, and KPI endpoints |
| Primary frontend | `frontend_Angular` | Angular SPA with guarded routes, dashboard, workflow builder/editor, runs, agents, and settings |
| Additional frontend | `backend_Node/frontend` | React + Vite SPA with the same core workflow management experience |
| Database | PostgreSQL 16 | Persistent storage for users, agents, workflows, runs, and run steps |
| Realtime | Server-Sent Events | Streams execution progress for workflow runs |
| Email | MailHog + Nodemailer | Development email delivery and capture |
| API docs | Swagger UI | Interactive OpenAPI documentation under `/api/docs` |

## Technology Stack

### Backend
- Node.js 20
- Express
- TypeScript
- Prisma ORM
- PostgreSQL
- Zod for request validation
- JWT for authentication
- bcryptjs for password hashing
- Swagger UI / swagger-jsdoc
- Nodemailer
- Server-Sent Events

### Angular frontend
- Angular 21
- Standalone components
- Angular Router
- Signals and computed state
- RxJS
- TypeScript
- SCSS
- Angular CLI

### React frontend
- React 19
- Vite
- TypeScript
- React Router
- Zustand
- Axios
- React Flow / XYFlow
- Recharts
- Tailwind CSS
- shadcn-style UI primitives

## Repository Structure

```text
IA_Workflow/
├── backend_Node/
│   ├── .env.example
│   ├── docker-compose.yml
│   ├── backend/
│   │   ├── prisma/
│   │   │   └── schema.prisma
│   │   └── src/
│   │       ├── config/
│   │       ├── controllers/
│   │       ├── middleware/
│   │       ├── routes/
│   │       ├── services/
│   │       ├── utils/
│   │       └── __tests__/
│   └── frontend/
│       ├── src/
│       │   ├── components/
│       │   ├── pages/
│       │   ├── services/
│       │   ├── stores/
│       │   └── lib/
│       └── Dockerfile
└── frontend_Angular/
    ├── src/
    │   └── app/
    │       ├── components/
    │       ├── models/
    │       ├── pages/
    │       └── services/
    ├── proxy.conf.json
    └── Dockerfile
```

### Important backend modules

- `src/index.ts` initializes Express, security middleware, rate limits, Swagger, and API routes.
- `src/config/index.ts` loads environment variables from `backend_Node/.env`.
- `src/config/swagger.ts` defines the OpenAPI schema and shared response models.
- `src/controllers/auth.ts` handles register, login, and current-user profile lookup.
- `src/controllers/workflows.ts` validates workflow graphs, detects cycles, and manages workflow CRUD.
- `src/controllers/runs.ts` creates runs, fetches run details, and exposes the SSE stream.
- `src/services/executionEngine.ts` executes workflow steps and coordinates agent runs.
- `src/services/eventBus.ts` emits run lifecycle events for realtime updates.
- `src/services/emailService.ts` sends run-completion notifications.
- `src/middleware/auth.ts` enforces JWT authentication and role-based access control.
- `src/prisma/seed.ts` seeds demo users and sample data.

### Important Angular modules

- `src/app/app.ts` is the authenticated shell and navigation layout.
- `src/app/app.routes.ts` defines guarded routes for dashboard, workflows, agents, runs, login, and settings.
- `src/app/services/auth.service.ts` manages JWT login state in `localStorage`.
- `src/app/services/app-data.service.ts` preloads workflows, agents, and runs for the dashboard.
- `src/app/services/api.service.ts` wraps the `/api` contract and SSE run streaming.
- `src/app/services/workflow.service.ts`, `run.service.ts`, `agent.service.ts`, and `kpi.service.ts` provide feature-specific data access.
- `src/app/pages/` contains the dashboard, workflow builder/editor, workflow list, playground, agents, runs, login, and settings screens.
- `src/app/components/` contains shared UI cards and page heading components.

### Important React frontend modules

- `src/App.tsx` defines the protected routes and main application shell.
- `src/components/layout/MainLayout.tsx` renders the sidebar, navigation, and user menu.
- `src/components/layout/ProtectedRoute.tsx` guards authenticated routes.
- `src/services/api.ts` centralizes API calls and token handling.
- `src/stores/authStore.ts` manages auth state.
- `src/pages/` contains dashboard, agent library, workflow list/editor, login, and playground views.

## Installation

### Prerequisites

- Node.js 20 or newer
- npm 10 or newer
- PostgreSQL 16
- Docker and Docker Compose, if you prefer containerized setup

### Backend setup

```bash
cd backend_Node/backend
npm install
npm run prisma:generate
```

If you are using a local PostgreSQL instance, create the database and apply the schema:

```bash
npm run prisma:push
npm run seed
```

### Angular frontend setup

```bash
cd frontend_Angular
npm install
```

### React frontend setup

```bash
cd backend_Node/frontend
npm install
```

## Environment Variables

Create a `.env` file at `backend_Node/.env` using `backend_Node/.env.example` as the template.

| Variable | Description | Example |
| --- | --- | --- |
| `DATABASE_URL` | PostgreSQL connection string used by Prisma | `postgresql://postgres:postgres@localhost:5432/workflow_db` |
| `JWT_SECRET` | Secret used to sign JWT tokens | `your-secret-key-change-in-production` |
| `SMTP_HOST` | SMTP host for notification emails | `localhost` |
| `SMTP_PORT` | SMTP port for notification emails | `1025` |
| `SMTP_FROM` | From address used by outgoing emails | `noreply@workflow-app.local` |
| `CORS_ORIGIN` | Allowed frontend origin | `http://localhost:5173` |
| `PORT` | Backend HTTP port | `3001` |
| `NODE_ENV` | Runtime environment | `development` |

Notes:
- The backend loads environment variables from `backend_Node/.env`.
- The frontend clients use `/api` as their base path, so local proxying or reverse proxying is required when they do not share the same origin as the backend.

## Running Locally

### Option 1: Full stack with Docker Compose

This starts PostgreSQL, MailHog, the Node backend, and the React/Vite frontend from `backend_Node/docker-compose.yml`.

```bash
cd backend_Node
cp .env.example .env
docker compose up --build
```

Services:
- Frontend: http://localhost:3000
- Backend API: http://localhost:3001/api
- Swagger UI: http://localhost:3001/api/docs
- OpenAPI JSON: http://localhost:3001/api/docs.json
- MailHog: http://localhost:8025

### Option 2: Backend + Angular frontend

Run the backend and Angular app separately.

Backend:

```bash
cd backend_Node/backend
npm install
npm run prisma:generate
npm run prisma:push
npm run seed
npm run dev
```

Angular frontend:

```bash
cd frontend_Angular
npm install
npm start
```

Default URLs:
- Angular app: http://localhost:4200
- Backend API: http://localhost:3001/api

### Option 3: Backend + React/Vite frontend

React frontend:

```bash
cd backend_Node/frontend
npm install
npm run dev
```

Default URLs:
- React app: http://localhost:5173
- Backend API: http://localhost:3001/api

## Build Instructions

### Backend

```bash
cd backend_Node/backend
npm run build
npm run start
```

### Angular frontend

```bash
cd frontend_Angular
npm run build
```

### React frontend

```bash
cd backend_Node/frontend
npm run build
npm run preview
```

## Docker Usage

### Backend stack

The main backend stack is defined in `backend_Node/docker-compose.yml`.

```bash
cd backend_Node
docker compose up --build
```

This compose file includes:
- `postgres` for PostgreSQL 16
- `mailhog` for email capture
- `backend` for the Express API
- `frontend` for the React/Vite client

### Angular frontend container

The Angular client has its own Dockerfile and compose file.

```bash
cd frontend_Angular
docker build -t ia-workflow-angular .
docker run --rm -p 4200:80 ia-workflow-angular
```

Or with Compose:

```bash
cd frontend_Angular
docker compose up --build
```

## API Overview

All backend endpoints are mounted under `/api`.

| Method | Endpoint | Auth | Description |
| --- | --- | --- | --- |
| POST | `/api/auth/register` | No | Register a new user |
| POST | `/api/auth/login` | No | Login and receive a JWT |
| GET | `/api/auth/me` | Yes | Get the current user profile |
| GET | `/api/agents` | Yes | List agents with search/filter/sort/pagination |
| GET | `/api/agents/:id` | Yes | Get a single agent |
| POST | `/api/agents` | Yes, admin | Create an agent |
| PUT | `/api/agents/:id` | Yes, admin | Update an agent |
| DELETE | `/api/agents/:id` | Yes, admin | Delete an agent |
| GET | `/api/workflows` | Yes | List workflows for the current user |
| GET | `/api/workflows/:id` | Yes | Get workflow details |
| POST | `/api/workflows` | Yes | Create a workflow |
| PUT | `/api/workflows/:id` | Yes | Update a workflow |
| DELETE | `/api/workflows/:id` | Yes | Delete a workflow |
| POST | `/api/workflows/:id/validate` | Yes | Validate workflow graph and agent references |
| POST | `/api/runs` | Yes | Start a workflow run |
| GET | `/api/runs` | Yes | List runs with pagination |
| GET | `/api/runs/:id` | Yes | Get run details with steps |
| GET | `/api/runs/:id/stream` | Yes | Stream run progress through SSE |
| GET | `/api/kpis` | Yes | Get dashboard KPI data |
| GET | `/api/health` | No | Health check |

Swagger UI is available at `/api/docs` and the raw OpenAPI document is available at `/api/docs.json`.

## Authentication Flow

1. The user registers or logs in through `/api/auth/register` or `/api/auth/login`.
2. The backend returns a JWT and the user profile.
3. The frontend stores the token in browser storage.
4. All protected API requests send `Authorization: Bearer <token>`.
5. The Angular and React clients use route guards / protected routes to redirect unauthenticated users to `/login`.
6. Real-time run streaming uses EventSource, so the JWT is passed as a query parameter to `/api/runs/:id/stream?token=...`.

## Database Configuration

The backend uses Prisma with PostgreSQL.

### Core models

- `User` with `ADMIN` and `USER` roles
- `Agent` with schemas, tags, versioning, and activation state
- `Workflow` with nodes, edges, variables, version, and status
- `Run` with execution status, prompt, timing, and metrics
- `RunStep` with per-node status, logs, retries, and preview payloads

### Workflow validation rules

- workflow graphs cannot contain cycles
- every referenced agent must exist and be active
- edges must point to valid node IDs
- workflows with no nodes are rejected during validation

## Usage Examples

### Create a workflow run

```bash
curl -X POST http://localhost:3001/api/runs \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer <your-jwt>' \
  -d '{
    "workflowId": "<workflow-id>",
    "prompt": "Summarize the meeting notes"
  }'
```

### Stream run progress

```bash
curl -N "http://localhost:3001/api/runs/<run-id>/stream?token=<your-jwt>"
```

### Validate a workflow

```bash
curl -X POST http://localhost:3001/api/workflows/<workflow-id>/validate \
  -H 'Authorization: Bearer <your-jwt>'
```

## Troubleshooting

- **Database connection errors**: confirm `DATABASE_URL` is correct and PostgreSQL is running.
- **Prisma client errors**: rerun `npm run prisma:generate` from `backend_Node/backend`.
- **Migration/schema issues**: use `npm run prisma:push` for quick sync or `npm run prisma:migrate` for versioned migrations.
- **Port conflicts**: the common ports are `3000`, `3001`, `4200`, `5432`, and `8025`.
- **CORS failures**: make sure `CORS_ORIGIN` matches the frontend URL you are using.
- **SSE stream not updating**: verify the JWT token is valid and that the backend endpoint is reachable at `/api/runs/:id/stream`.
- **Email not visible**: open MailHog at http://localhost:8025 when using the Docker stack.
- **Docker startup race conditions**: the backend expects the database to be healthy; give PostgreSQL a few seconds on first boot.

## Testing

Backend:

```bash
cd backend_Node/backend
npm test
```

Angular frontend:

```bash
cd frontend_Angular
npm test
```

React frontend:

```bash
cd backend_Node/frontend
npm test
```

## CI/CD

No CI/CD workflow files were detected in the repository.

If you want to add delivery automation, a typical next step would be:
- GitHub Actions for build, test, and Docker image publishing
- Azure DevOps or GitLab CI for environment-specific deployment
- separate jobs for backend, Angular, and React frontend builds

## Contributing

1. Create a feature branch.
2. Make focused changes with clear commits.
3. Run the relevant tests and builds before opening a pull request.
4. Keep API contracts, frontend types, and Prisma schema changes in sync.
5. Update this README when you add a new service, route, or deployment target.

## License

The repository does not currently include a single root-level license file.
The Angular subproject includes an MIT `LICENSE` file in `frontend_Angular/`.

If you plan to publish the monorepo publicly, add a root `LICENSE` file and make the licensing scope explicit.
