# Orma frontend

Angular 22 (standalone, signals, OnPush), TypeScript 6 strict, Tailwind 4, spartan/ui helm
components, Transloco (en, fr), ng-icons (lucide). Calls the API of ../backend under `/api/v1`.

## Structure (src/app)

- `core/api`: `generated/` (ng-openapi-gen, never edited by hand) and `api-problem.ts`
  (`toApiProblem`, `applyFieldErrors`).
- `core/auth`: `Session` (access token in memory, refresh cookie, `useNewAccessToken`,
  `updateUser`, `signOut`, `endEverywhere`), interceptor, guards, `account-rules.ts`,
  `user-preferences.ts`.
- `core/i18n`: Transloco setup, `LanguageService`, `problem-message.ts`, translated page titles.
- `core/money`: amounts as strings with BigInt maths (`parseAmount`, `sumAmounts`,
  `convertAmount`, `formatMoney`, `amountValidator`), exchange rates, money pipe.
- `core/period`: financial months (`financial-month.ts`) and the `Periods` service (pay cycle of
  the signed-in user). Features get month ranges from `Periods`, never from calendar maths.
- `core/theme`: theme service (sets `data-theme` on `<html>`).
- `ui/helm`: spartan helm components, imported as `@orma/helm/<name>` (tsconfig paths).
- `ui/tokens`: `prism.css` (`[data-style='prism']` and `[data-theme]` tokens), `helm-bridge.css`.
- `ui/charts`: hand-built SVG `DonutChart`, `ProgressRing`. `ui/category`: `CategoryChip` and visuals.
- `ui/gallery`: component gallery at `/ui`, development builds only (`dev-tools.routes.development.ts`).
- `layout/shell`: side rail from md, floating bottom bar on phones. `layout/auth-shell`: signed-out pages.
- `features/<feature>`: accounts, auth, categories, dashboard, profile, transactions.
  Each has `<feature>.routes.ts` (lazy, `loadChildren` in `app.routes.ts`), `pages`, `components`,
  `models`, and `data` (except auth).

## Rules

- `pages` are routed and smart; `components` only take inputs and emit outputs.
- Only `data/` calls HTTP, through `Api.invoke(fn, params)` with functions from the generated client.
- Stores are `@Injectable()` classes with signals, provided in the route's `providers`, not in root.
- A feature never imports another feature; shared code goes to `core` or `ui`.
- No hard-coded text: keys in `src/i18n/en.json` and `fr.json`. `translations.spec.ts` checks both
  have the same keys and the same `{{ parameters }}`.
- Amounts are strings, never `number`: use `core/money`.
- Colours, sizes and radii from the tokens only; `dark:` follows `data-theme`.
- Reactive forms: a form with `(ngSubmit)` needs `[formGroup]`, or the browser submits and reloads.
- Transaction filters and the month live in the URL query params.

## Tests

- Vitest through `@angular/build:unit-test`, specs next to the file (`*.spec.ts`).

## Commands (run in frontend/)

- Dev server: `npm start` (port 4200, proxies `/api` and `/actuator` to http://localhost:8080)
- Build: `npm run build`. Lint: `npm run lint`. Tests: `npm run test:ci`.
- Format: `npm run format`, check: `npm run format:check`.
- Client: `npm run api:generate` (backend running with the dev profile).
- CI runs `npm ci`, format:check, lint, test:ci, build, `npm audit --omit=dev --audit-level=high`.
