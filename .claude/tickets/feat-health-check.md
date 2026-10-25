# feat/health-check: phase 4, public health check and uptime monitor

Date: 2026-10-25. Status: committed (d38ea5d), not pushed.

## Why
/actuator/health answered 200 from the SPA fallback (Caddy proxied /api/* only), and /api/v1/auth/csrf does not touch
the database. A monitor on either would stay green with the database down.

## Changes
- frontend/Caddyfile: handle /actuator/health -> api:8080 (exact path; show-details=never, so only {"status":"UP"}).
- deploy.yml smoke test: /actuator/health instead of /api/v1/auth/csrf (curl -f fails on 503).
- docs/recovery.md: UptimeRobot keyword monitor on /actuator/health, keyword UP.

## Verified
caddy validate (caddy:2-alpine) OK. Not run end to end: the deploy smoke test checks it.

## Open
- Alexis: push, merge, open /actuator/health (expect {"status":"UP"}), create the UptimeRobot monitor, Bitwarden login.
- Next part: phone pass on real devices, then ASVS L1.
