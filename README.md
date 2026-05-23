# CodeCollab — Collaborative Coding Platform

CodeCollab is a full‑stack, real‑time collaborative coding platform that lets teams, learners, and interviewers co-edit code, chat, run code snippets, and get AI assistance in a single web application.

---

## Overview

- Real‑time collaborative code editing with role‑based access (Admin / Editor / Viewer).
- Project management with invite codes, collaborator roles, and versioned code history.
- Session‑based live collaboration (cursor positions, active users, in‑session chat).
- Server‑side local code execution for multiple languages (requires secure sandboxing in production).
- AI chat endpoint backed by Google Gemini (server) with client‑side fallback answers.

## Main Purpose & Target Users

This project is intended for developers, interviewers, educators, and teams who need a collaborative coding environment for pair programming, live interviews, teaching, or collaborative problem solving.

## Core Features

- Projects: create, update, delete, invite collaborators via invite codes.
- Roles: owner/admin/editor/viewer with enforced ownership checks.
- Sessions: create/join/leave sessions, track active users and cursors, session chat.
- Real‑time: socket rooms for projects and user notification channels.
- Auth: JWT authentication (cookie + Bearer) with password reset support.
- AI Chat: `/api/v1/ai/chat` endpoint that proxies to Gemini and falls back to prewritten answers.
- Execution: `/api/v1/execution/execute` endpoint that runs code on the server (local runtimes).

## Tech Stack

- Frontend: React, Vite, Tailwind CSS, Ace editor (`react-ace`), axios.
- Backend: Node.js, Express, Socket.io, Mongoose (MongoDB), JWT (`jsonwebtoken`), `bcryptjs`.
- Database: MongoDB (use a local instance or MongoDB Atlas).
- Dev: nodemon (backend), Vite (frontend), npm.

## Repository Layout

```
. (root)
├─ backend/           # Express API, controllers, models, websockets
│  ├─ controllers/
│  ├─ models/
│  ├─ routes/
│  ├─ utils/
│  ├─ middleware/
│  └─ server.js       # backend entrypoint
├─ frontend/          # React + Vite client
│  ├─ src/
│  ├─ public/
│  └─ vite.config.js
└─ README.md
```

## Quick Start (Local)

Prerequisites: Node.js (LTS), npm, MongoDB (local or Atlas).

1) Backend

```bash
cd backend
npm install
# create a backend/.env file (see Required Environment Variables below)
npm run dev   # starts server with nodemon
```

2) Frontend

```bash
cd frontend
npm install
npm run dev
```

Open the frontend at `http://localhost:5173` and the backend API at `http://localhost:5000` (default).

## Live Demo

The application is deployed on Render. You can access the live backend API at:

- `https://collaborative-coding-platform-backend-h8dn.onrender.com` (API base: `.../api/v1`)

If you deployed a frontend, point `VITE_API_URL` to the API base above.

## Required Environment Variables

Create `backend/.env` with at least the following keys:

- `MONGO_URI` — MongoDB connection string
- `JWT_SECRET` — JWT signing secret
- `JWT_EXPIRE` — token lifetime (e.g., `30d`)
- `JWT_COOKIE_EXPIRE` — cookie expiry in days (number)
- `GEMINI_API_KEY` — (optional) API key for Gemini AI
- `NODE_ENV` — `development` or `production`
- `PORT` — port for backend (default 5000)
- `LOCAL_EXEC_TIMEOUT_MS` — (optional) runtime execution timeout in ms
- `LOCAL_EXEC_MAX_BUFFER` — (optional) max buffer size for child process output

Frontend optional env (create `frontend/.env`):

- `VITE_API_URL` — backend base URL, e.g. `http://localhost:5000/api/v1`

## Running & Building

- Backend dev: `cd backend && npm run dev`
- Backend production: `cd backend && npm start`
- Frontend dev: `cd frontend && npm run dev`
- Frontend build: `cd frontend && npm run build`

## API Summary

- `POST /api/v1/auth/register` — register
- `POST /api/v1/auth/login` — login
- `GET /api/v1/auth/me` — current user (protected)
- `GET /api/v1/auth/logout` — logout
- `POST /api/v1/projects` — create project (protected)
- `GET /api/v1/projects` — list projects (protected)
- `POST /api/v1/projects/join` — join via invite code
- `POST /api/v1/sessions` — create session or join existing (protected)
- `POST /api/v1/sessions/:id/join` — join session
- `POST /api/v1/sessions/:id/leave` — leave session
- `POST /api/v1/sessions/:id/chat` — post chat message
- `POST /api/v1/execution/execute` — execute code (protected)
- `POST /api/v1/ai/chat` — AI chat

Refer to `backend/routes/` and `backend/controllers/` for inputs and response shapes.

## WebSocket Events (high level)

- Clients authenticate sockets using a JWT token passed in `handshake.auth.token` or the `Authorization` header.
- Client emits `join_project` with a project id to join a project's room; server performs authorization checks.
- Server emits `your_role`, `current_collaborators`, `collaborator_role_changed`, `role_updated`, `collaborator_removed`, and `project_update` to keep clients synchronized.

## Security & Production Recommendations

- The execution endpoint runs arbitrary code via `child_process.spawn`. Do NOT enable this in production without strong sandboxing (use containerized workers, seccomp, or specialized sandboxing like Firecracker).
- Remove debug logging of sensitive data (e.g., password reset tokens are printed in development code). Use secure email delivery instead.
- Use HTTPS and set the cookie `secure` flag in production (already conditionally set in `tokenManager`).
- Add rate limiting (`express-rate-limit`) to auth, AI, and execution endpoints.
- Use a secrets manager for `GEMINI_API_KEY` and `JWT_SECRET`.

## Performance & Maintainability Suggestions

- Add MongoDB indexes on fields used in queries: `inviteCode`, `owner`, `collaborators.user`, and session lookups by `project`.
- Centralize ownership and role checks into a utility to avoid duplicated logic.
- Replace console logging with structured logging (winston or pino) and add monitoring (Sentry / Prometheus).
- Add unit and integration tests and a CI pipeline to run linting and tests automatically.

## Notable Files

- `backend/server.js` — application bootstrap, DB connect, route mounting, and Socket.io initialization
- `backend/models/Project.js`, `backend/models/User.js`, `backend/models/Session.js` — Mongoose models
- `backend/utils/socketHandlers.js` — real‑time collaboration logic
- `backend/controllers/*` — REST API business logic
- `frontend/src/utils/geminiService.js` — client AI integration and fallback answers
- `frontend/src/utils/codeExecutionService.js` — client wrapper for `execution/execute`

## Recommended VS Code Extensions

- ESLint, Prettier, Tailwind CSS IntelliSense, MongoDB for VS Code, GitLens, npm Intellisense.

## Next Steps I Can Help With

- Generate `backend/.env.example` and `frontend/.env.example`.
- Add `Dockerfile` and `docker-compose.yml` for local development.
- Add a GitHub Actions workflow to run ESLint and tests.

---

If you want me to add `backend/.env.example` or scaffold Docker/CI files now, tell me which to create first.
