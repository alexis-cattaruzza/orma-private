# Local working notes (not in git)

Loaded with CLAUDE.md. Rules stay in CLAUDE.md; this file is the map and the way of working.

## Status

`STATUS.md` holds what is done and what is left for v1. Read it when starting or planning a part,
and update it when a part is merged. It is shared with Claude in the claude.ai app.

## Map

- `backend/`: Spring Boot API, Maven. Details in `backend/CLAUDE.md`, loaded when you touch it.
- `frontend/`: Angular app. Details in `frontend/CLAUDE.md`, loaded when you touch it.
- `docs/adr/`: decisions 0001 to 0007.
- `compose.prod.yml` (+ `compose.local.yml` for a local run), `.env.prod.example`, `deploy/setup-server.sh`.
- Server: Oracle eu-zurich-1, Ubuntu 24.04 ARM, 144.24.237.117, `orma-app.app` (Cloudflare, DNS only; DuckDNS `orma-prod.duckdns.org` before), user `ubuntu`,
  key `~/.ssh/orma_oci`, app folder `/opt/orma`.
- `docker-compose.yml`: PostgreSQL 18 on 127.0.0.1:5432, Mailpit (SMTP 1025, inbox http://localhost:8025).
- `.run/`: IntelliJ run configurations: Backend (dev), Frontend (dev), Database (PostgreSQL).
- `.github/workflows/`: backend.yml, frontend.yml, secrets.yml (gitleaks). `.github/dependabot.yml`.
- Local only, listed in `.git/info/exclude`, never stage: `scripts/` (test data), `CLAUDE.local.md`,
  `STATUS.md`, `backend/CLAUDE.md`, `frontend/CLAUDE.md`, `.claude/`.

## Backend and frontend contract

- One git repository: one branch covers both sides of a feature.
- REST paths come from the controllers (`/api/v1/...`). The dev server proxies `/api` and `/actuator`
  to http://localhost:8080 (`frontend/proxy.conf.json`).
- After any controller or DTO change: restart the backend with the dev profile, run
  `npm run api:generate` in `frontend/` (reads http://localhost:8080/v3/api-docs), and commit the
  generated client with the change. Never edit `frontend/src/app/core/api/generated/` by hand.
- Generated function names come from `@Operation(operationId = ...)`, else the controller method
  name. They must be unique across the whole API.
- Errors: the backend throws `ApiException` (code + message key); the frontend reads them with
  `toApiProblem` (core/api/api-problem.ts) and `injectProblemMessage` (core/i18n/problem-message.ts).
- Full-stack order: migration, entity, DTO, service, controller, unit test; then `api:generate`;
  then the frontend data service, store, components, and keys in `en.json` and `fr.json`.

## Way of working

- One part per branch, from a fresh main. Alexis tests, then asks for the commit; he pushes and merges.
- Commits authored by Alexis Cattaruzza <alexis.cattaruzza@gmail.com>, Conventional Commits, split
  by aspect, no Co-Authored-By or other AI trailer.
- After each part: short test steps, then wait for his OK.

## Token rules

- Locate before reading (grep, glob or the IntelliJ index), then read line ranges only.
- Never read build output, dependencies, logs or `.env` (blocked in `.claude/settings.json`).
- In the generated client, read only the one function file needed (`fn/<tag>/<operation>.ts`).
