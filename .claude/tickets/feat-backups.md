# feat/backups: phase 4, backups and recovery

Date: 2026-10-25. Status: committed (f604d4e..2d4236b), not pushed.

## Changes
- deploy/backup.sh (orma-backup): pg_dump -Fc + check, tar with .env.prod, compose.prod.yml, secrets/, .deployed-tag,
  age-encrypted to a public key, rclone to S3 remote daily/ (30 d) and monthly/ (first of month, 400 d),
  3 local copies in /opt/orma/backups, healthchecks.io pings (start, success, fail).
- deploy/restore.sh (orma-restore): latest | daily/.. | monthly/.. | local file. --check restores into a throwaway
  postgres:18 container and counts rows. Prod mode: confirm, stop api, safety dump, drop/create, pg_restore, start.
  New server: also restores config and says which SHA to deploy.
- deploy/backup.env.example: rclone remote via env vars (R2 default, OCI alternative), retention, ping URL.
- setup-server.sh: installs age, rclone, the three orma-* commands, systemd orma-backup.timer at 03:15
  (enabled only when /opt/orma/backup.env exists).
- docs/recovery.md: setup and runbook (bad deploy, data mistake, VM lost, Oracle account lost, leaked secret, lost key).
- Gmail to Resend in README, .env.prod.example, compose.prod.yml (comment + default host), backend property comments.

## Verified
bash -n on all scripts. Not run: Docker Desktop was off, no shellcheck or age locally.

## Open
- Alexis: age key, R2 bucket + token, healthchecks.io check, backup.env, run setup-server.sh, first backup, restore --check.
- GitHub variable PUBLIC_URL=https://orma-app.app (gh CLI not installed here, check by hand). Delete DuckDNS name.
- Next parts: UptimeRobot monitor, phone pass, ASVS L1 (with audit log, Dependency-Check, ZAP baseline).

## Portability (same branch)
- docs/new-machine.md: three places (GitHub orma, private repo orma-private, Bitwarden folder Orma), item list,
  must-keep vs regenerable, new computer steps. README links it.
- tools/private-files.sh save|restore: copies local-only files to ../orma-private (commit + push), restore also
  writes .git/info/exclude. Tested against a throwaway bare repo and a fresh clone.
- backup.env.example: RCLONE_CONFIG_ORMA_REGION=auto. recovery.md points to the Bitwarden item names.

## Server setup and test (2026-10-25)
- Bitwarden (EU) folder Orma: SSH key, .env.prod, server info, age key, backup.env (secure notes; web vault SSH key type
  cannot import a key). Private repo orma-private created.
- R2 bucket orma-backups (WEUR), lifecycle 400 d, bucket lock 28 d, bucket-scoped token. healthchecks.io check daily/2 h.
- setup-server.sh run, timer next 03:20 UTC. First backup 64K in daily/ and monthly/.
- rclone 1.60.1 (Ubuntu) + R2: PUT 200, then HEAD ?versionId= answers 501 (R2 does not implement it) -> retry noise.
  Fix: RCLONE_CONFIG_ORMA_NO_HEAD=true (integrity still checked by Content-MD5 on PUT). Added on the server and example.
- restore.sh --check on the laptop: OK, 8 tables (users 1, categories 13, exchange_rates 464).
- restore.sh: clear "File not found" for a missing local file. Server copy still the old one: copy again.
- Resend key was pasted in the chat: rotation recommended.
