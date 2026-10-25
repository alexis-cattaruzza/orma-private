# feat/prod-stack: phase 4 part A, production stack run locally

Date: 2026-10-22. Status: committed on feat/prod-stack, not pushed.

## Commits
- feat(api): send production email through Gmail SMTP
- build: add Docker images for the API and the web app
- build: add the production compose stack

## What changed
- application-prod.properties: Resend replaced by generic SMTP (MAIL_HOST, MAIL_PORT, MAIL_USERNAME,
  MAIL_PASSWORD), STARTTLS required (MAIL_STARTTLS_REQUIRED=false only for local Mailpit), 10 s timeouts.
- backend/Dockerfile: Maven build, eclipse-temurin:21-jre-alpine, non-root user, wget health check.
- frontend/Dockerfile + Caddyfile: Angular build served by caddy:2-alpine, /api/* proxied to api:8080,
  SPA fallback, hashed assets immutable, the rest no-cache. /actuator not exposed.
- compose.prod.yml: db on an internal network (no port), api on private + edge, web publishes 80/443,
  JWT keys as compose secrets from JWT_KEYS_DIR, log size caps.
- compose.local.yml: Mailpit (inbox 8026) for a local run. .env.prod.example. README section.

## Verified locally
Migrations 1 to 9, prod profile start, HTTPS on localhost, HTTP redirect, deep links, registration email
with https://localhost link. Alexis tested the app flows: OK.

## Notes for part B
- On Linux the PEM files must be readable by the non-root API user (chmod 644).
- MAIL_FROM must be the Gmail address. Frontend security headers at Caddy still open.
