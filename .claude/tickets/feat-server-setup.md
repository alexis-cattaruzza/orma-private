# feat/server-setup: phase 4, server and deploy from GitHub Actions

Date: 2026-10-24. Status: committed, not pushed.

## Commits
- build: add the server setup script
- ci: deploy to the server on merge to main

## Server
Oracle eu-zurich-1, Pay As You Go (budget alerts: forecast 1, actual 0.01), VM.Standard.A1.Flex 1 OCPU / 6 GB
(free up to 4 / 24), Ubuntu 24.04 aarch64, 144.24.237.117, orma-prod.duckdns.org. First VM was an E2.1.Micro
(x86) by mistake, replaced. Security list: 22, 80, 443 tcp, 443 udp.
deploy/setup-server.sh run: updates, unattended upgrades (reboot 04:30), iptables 80/443 before Oracle's REJECT
(saved before Docker install), key-only SSH, fail2ban, Docker, /opt/orma, /usr/local/bin/orma-deploy.
On the server: /opt/orma/.env.prod (600), secrets/jwt-*.pem (644, dir 700), deploy key in ubuntu's
authorized_keys with restrict,command="/usr/local/bin/orma-deploy". Host key SHA256:UeuQAUMAVgPlmHLseCO+5RJwcAgFYOX+SuEn0LXtZRc.

## Deploy
deploy.yml on push to main (not docs): calls backend.yml and frontend.yml (now workflow_call, no longer on push),
builds arm64 images (build stages on $BUILDPLATFORM, QEMU for the runtime layer), pushes
ghcr.io/alexis-cattaruzza/orma-{api,web}:<sha>, sends "<actor> <GITHUB_TOKEN>" + compose.prod.yml over SSH,
orma-deploy pulls, up --wait, rollback to .deployed-tag on failure, prunes images older than 7 days.
Smoke test on vars.PUBLIC_URL. Manual rollback: workflow_dispatch with a sha.
Repo secrets: DEPLOY_HOST, DEPLOY_SSH_KEY, DEPLOY_KNOWN_HOSTS; variable PUBLIC_URL.

## Verified locally
actionlint clean, compose config, both images build for linux/arm64.

## Open
- Gmail: new account blocked from 2-Step Verification, retry. Then set MAIL_PASSWORD in /opt/orma/.env.prod
  and redeploy (workflow_dispatch with the current sha, or any merge).
- Backups to Oracle Object Storage, phone pass, ASVS L1.

## After merge (2026-10-24, config only, no code)
- PR #25 merged, first deploy OK on orma-prod.duckdns.org.
- Domain orma-app.app bought at Cloudflare Registrar. DNS: A @ 144.24.237.117 (DNS only), CAA letsencrypt.org,
  Resend records via auto-configure, DMARC p=none added by hand.
- Resend (EU) verified. Server .env.prod: SITE_ADDRESS, APP_BASE_URL, MAIL_* to smtp.resend.com:587, user resend.
- Restarted with IMAGE_TAG=$(cat .deployed-tag). Let's Encrypt cert OK. Sign-up email received, Alexis signed in.
- Server alias `orma` in ~/.bashrc for compose commands and logs.
- Todo: GitHub variable PUBLIC_URL=https://orma-app.app (asked), docs still say Gmail, delete DuckDNS name.
