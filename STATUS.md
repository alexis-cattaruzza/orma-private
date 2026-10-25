# Orma: development status

Updated 2026-10-25. Shared by Claude Code (IntelliJ) and Claude in the claude.ai app: read it before
starting a part, update it when a part is merged. Not in git. Full plan: the "Orma: Project Plan" doc.

## Where we are

- v1 features are all built and merged (phases 1 to 3). Last merged: PR #26 (backups and recovery). Prod live on https://orma-app.app (Resend email working), deploys on merge to main.
- Next: finish phase 4: uptime monitor, phone pass, ASVS L1 (deploy and backups done).
- Open PR to leave alone: Dependabot #5 (TypeScript 7), until Angular supports it.

## Done

| Part | PR |
| --- | --- |
| Phase 1 foundations: repo, CI, auth backend, design system, PostgreSQL 18, sign-in screens | #7 to #10 |
| Phase 2 money: French and English, accounts, categories, ECB rates, transactions and transfers | #11 to #15 |
| Profile settings: details, password, language, theme, main currency, budget | #16 |
| Dashboard with SVG charts | #17 |
| Monthly details: URL filters, totals, CSV export | #18 |
| Privacy: data export, sign out everywhere, delete account | #19 |
| Pay cycle: financial month from payday, weekend rules | #20 |
| Phone safe areas (notch, home indicator) | #21 |
| Phase 4 part A: production config (Gmail SMTP), Docker images, production compose, run locally | #22 |
| Frontend security headers at Caddy (CSP, HSTS), cache fix | #23 |
| PWA: icons, manifest, service worker, update toast | #24 |
| Server setup script, deploy from GitHub Actions (GHCR, health-gated, rollback) | #25 |

## Left for v1: phase 4 (everywhere)

Hosting: Oracle Cloud Always Free ARM VM, own domain (about 14 USD/year), Let's Encrypt, Resend SMTP.

- [x] Production config: prod profile values, JWT key pair on the server, SMTP, `APP_BASE_URL`
- [x] Docker images: backend jar; frontend static files behind Caddy, which also proxies `/api`
      (same origin, `environment.ts` keeps `apiRootUrl: ''`)
- [x] Production compose: PostgreSQL on the private network only, Caddy with HTTPS
- [x] Server: VM, firewall 80 and 443 only, SSH keys, Docker
- [x] Deploy from GitHub Actions on merge to main
- [x] Nightly encrypted database backups off the VM, restore tested (feat/backups: age + rclone to
      Cloudflare R2, healthchecks.io, restore check OK 2026-10-25; runbook docs/recovery.md)
- [x] Frontend security headers at Caddy (CSP, HSTS); the API already sends CSP, frame deny,
      referrer and permissions policy
- [x] PWA: manifest, icons, service worker, installable on Android, iPhone and desktop
- [ ] Phone pass on real devices
- [ ] ASVS Level 1 checklist before going live

Security baseline items still open: audit log of security events, OWASP Dependency-Check (needs an
NVD API key), ZAP baseline scan in CI. Already in place: gitleaks, Dependabot, npm audit in CI,
404 tests for other users' data (accounts, categories).

Next: merge feat/health-check (Caddy proxies /actuator/health, smoke test uses it), then the
UptimeRobot keyword monitor on it. Then phone pass, ASVS L1. Check GitHub variable PUBLIC_URL is
https://orma-app.app. Delete the DuckDNS name.

Server: Oracle eu-zurich-1 (Pay As You Go, budget alerts at 1), A1 VM 144.24.237.117, domain
`orma-app.app` (Cloudflare Registrar, DNS only), email through Resend SMTP (Gmail dropped: new account blocked).

## After v1 (phase 5 and later)

- [x] Pay cycle (#20)
- [ ] Statement import: CSV and camt.053 first, then PDF per bank (UBS, BNP, Revolut), preview before saving
- [ ] Bank sync: provider to choose (GoCardless closed to new sign-ups; Enable Banking restricted
      mode for own accounts; UBS outside PSD2)
- Later: recurring transactions, category budgets, receipt AI, partner sharing, offline mode,
  budget notifications, selectable styles, TOTP and passkeys, device list

## Notes

- CLAUDE.md lists Playwright for critical flows; it is not installed yet.
- Pay cycle ignores bank holidays; a payday after the 15th opens the next month.
- Test data for admin: `scripts/dev/sample-transactions.sql` (local only).
