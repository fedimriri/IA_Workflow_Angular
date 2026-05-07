# IA Workflow Angular

IA Workflow Angular is a single-page Angular application for managing AI workflows, agents, and execution runs. It behaves like a lightweight workflow studio: users authenticate, inspect KPIs, design workflows, launch executions, and follow live progress in real time.

## Ownership

- Owner: fedimriri
- Copyright: © 2026 fedimriri
- License: MIT

## What this app does

- Secure login with JWT-based authentication stored in browser `localStorage`.
- Dashboard with KPI cards for total runs, success rate, duration metrics, top agents, and error breakdowns.
- Workflow studio for listing, filtering, creating, editing, validating, and deleting workflows.
- Workflow builder view that renders the workflow graph and its node connections.
- Workflow playground that launches a run and streams execution updates through server-sent events.
- Agents page for browsing active agents and their metadata.
- Runs page with execution history, step-by-step details, logs, inputs, outputs, and status breakdowns.
- Settings page for application configuration.

## Tech Stack

- Angular 21 with standalone components
- Angular Router with guarded routes
- Signals, computed state, and effects
- RxJS for HTTP and data loading flows
- TypeScript
- SCSS
- Vitest for unit testing

## Project Structure

- `src/app/app.ts` - root shell, navigation, and authenticated app bootstrap
- `src/app/app.routes.ts` - route definitions and lazy-loaded pages
- `src/app/services/` - API, auth, workflow, run, KPI, and data coordination services
- `src/app/models/` - domain models for users, agents, workflows, runs, KPI data, and pagination
- `src/app/pages/` - feature pages for dashboard, workflows, agents, runs, login, settings, and workflow views
- `src/app/components/` - shared UI building blocks

## Backend Contract

The frontend expects a backend API mounted under `/api` with endpoints such as:

- `POST /api/auth/login`
- `POST /api/auth/register`
- `GET /api/auth/me`
- `GET /api/agents`
- `GET /api/workflows`
- `POST /api/workflows`
- `POST /api/workflows/:id/validate`
- `GET /api/runs`
- `POST /api/runs`
- `GET /api/kpis`
- `GET /api/runs/:id/stream?token=...` for live execution updates

## Local Development

Install dependencies:

```bash
npm install
```

Run the development server:

```bash
npm start
```

The app is served at `http://localhost:4200/` and uses `proxy.conf.json` to forward `/api` requests during local development.

Build for production:

```bash
npm run build
```

Run unit tests:

```bash
npm test
```

## Docker

This repository includes a multi-stage `Dockerfile` and a `docker-compose.yml` for containerized runs.

Build the image:

```bash
docker build -t ia-workflow-angular:latest .
```

Run the container:

```bash
docker run --rm -p 4200:80 ia-workflow-angular:latest
```

Start with Docker Compose:

```bash
docker compose up --build -d
```

## Notes

- The app is route-protected: most pages require authentication and redirect to login when no token is present.
- Workflow and run data are loaded lazily and cached in `AppDataService` using Angular signals.
- Live run execution relies on EventSource streaming from the backend.
- If your backend is hosted separately in production, configure the frontend base URL or reverse proxy so `/api` resolves correctly.

## Useful Commands

```bash
npm start
npm run build
npm test
docker compose up --build -d
```

## Angular CLI

For more Angular CLI commands and generators, use:

```bash
npx ng --help
```

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for the full text.
