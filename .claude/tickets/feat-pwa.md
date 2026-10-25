# feat/pwa: phase 4, installable PWA

Date: 2026-10-22. Status: committed on feat/pwa, not pushed.

## Commits
- feat(web): add app icons and web manifest
- feat(web): add service worker with update prompt

## What changed
- New dependency @angular/service-worker 22.2.3 (official, same version as core). Give the reason in the PR.
- Icons: white O ring with lime dot on primary blue, sources public/icons/icon.svg and icon-maskable.svg,
  rendered with an alpine container (rsvg-convert, imagemagick): 192, 512, maskable 512, apple-touch 180,
  favicon.ico (16/32/48).
- manifest.webmanifest: standalone, light theme colours. index.html: manifest, icons, apple title.
- ngsw-config.json: app shell prefetch, icons and fonts lazy, /api and /actuator never cached or navigated.
- core/pwa/provide-app-updates.ts: worker in production builds only, toast with reload on VERSION_READY,
  check on visibilitychange, reload on unrecoverable. Keys app.update.ready / app.update.reload.

## Verified
Format, lint, 64 tests, build. Local stack: manifest and worker served no-cache with right types.
Alexis: install in Chrome/Edge, offline shell, update toast and reload. Test text reverted, Caddy root
removed from the Windows user store.

## Notes
- npm audit: 2 pre-existing dev-only findings (vite under @angular/build), CI --omit=dev passes.
- Real phone install needs the public server: phone pass item.

## Next
Server items: Oracle VM, firewall, SSH, Docker; deploy from GitHub Actions; nightly backups with restore test.
Needs the Oracle account and the subdomain.
