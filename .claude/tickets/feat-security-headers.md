# feat/security-headers: phase 4, frontend security headers at Caddy

Date: 2026-10-22. Status: committed on feat/security-headers, not pushed.

## Commits
- fix(web): cache Angular chunks whose hash has lowercase letters, dashes or underscores
- feat(web): send security headers from Caddy

## What changed
- Caddyfile, static files only (the API sends its own, Spring adds HSTS): CSP, HSTS 1 year with
  includeSubDomains, nosniff, X-Frame-Options DENY, Referrer-Policy, Permissions-Policy, COOP. Server header removed.
- CSP: script-src 'self'; style-src needs 'unsafe-inline' (Angular component <style> tags at runtime, no
  per-request nonce on a static server). manifest-src and worker-src 'self' ready for the PWA.
- angular.json production: critical CSS inlining off (it adds an inline onload handler).
- Cache regex: Angular hashes use [A-Za-z0-9_-], the part A regex missed chunks.

## Verified
curl: headers on static, API headers unchanged, no inline script in index.html, cache rules.
Alexis: full app in Firefox, no CSP errors.

## Next
PWA on its own branch, then server items (VM, deploy, backups).
