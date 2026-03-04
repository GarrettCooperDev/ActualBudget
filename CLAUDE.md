# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Actual Budget is a local-first personal finance app built with TypeScript/React. It's a Yarn 4 monorepo with SQLite (embedded, no external DB needed) and CRDT-based multi-device sync.

## Essential Commands

**All yarn commands must be run from the repository root, never from child workspaces.**

```bash
# Development
yarn start                  # Web dev server (port 3001)
yarn start:server-dev       # Web + sync server (port 5006)

# Quality checks (run before committing)
yarn typecheck              # TypeScript checking
yarn lint:fix               # Oxlint + Oxfmt auto-fix
yarn test                   # All tests via lage (parallel, cached)
yarn test:debug             # Tests without cache

# Package-specific tests
yarn workspace loot-core run test
yarn workspace @actual-app/api run test

# E2E / Visual regression
yarn e2e                    # Playwright E2E
yarn vrt                    # Visual regression tests
yarn vrt:docker             # VRT in Docker (consistent env)

# Specific E2E test
yarn workspace @actual-app/web run playwright test accounts.test.ts --browser=chromium
```

## AI Agent Requirements

- **ALL commit messages MUST be prefixed with `[AI]`** — no exceptions
- **ALL PR titles MUST be prefixed with `[AI]`** — no exceptions
- **Never fill in the PR template** — leave it for humans
- Add the **"AI generated"** label to PRs

## Monorepo Packages

| Package                | Alias                     | Purpose                                                               |
| ---------------------- | ------------------------- | --------------------------------------------------------------------- |
| `loot-core`            | —                         | Core business logic (platform-agnostic), client/server/shared modules |
| `desktop-client`       | `@actual-app/web`         | React UI for web and Electron                                         |
| `sync-server`          | `@actual-app/sync-server` | Express.js sync server (JS→TS migration in progress)                  |
| `api`                  | `@actual-app/api`         | Public Node.js API for integrations                                   |
| `component-library`    | `@actual-app/components`  | Shared React UI components, theme system, icons                       |
| `crdt`                 | `@actual-app/crdt`        | CRDT sync protocol with protobuf serialization                        |
| `eslint-plugin-actual` | —                         | Custom lint rules (i18n, typography, logging)                         |
| `docs`                 | —                         | Docusaurus documentation site                                         |

## Architecture

**Client-Server with CRDT sync**: The client (desktop-client) communicates with core logic (loot-core) which handles business rules, database ops, and sync. Data is stored locally in SQLite and optionally synced via the sync server using CRDTs.

**Platform abstraction in loot-core**: Platform-specific code uses conditional exports resolved at build time:

- `.electron.ts` — Electron-specific
- `.web.ts` / `.browser.ts` — Browser-specific
- Base files — Shared code
- Never directly reference platform-specific imports; use the export system

**State management**: Redux Toolkit for UI state, TanStack React Query for server state.

**Key source directories**:

- `packages/loot-core/src/client/` — Client-side core logic
- `packages/loot-core/src/server/` — Server-side core logic
- `packages/loot-core/src/shared/` — Shared utilities
- `packages/loot-core/src/types/` — Type definitions
- `packages/desktop-client/src/components/` — React components
- `packages/desktop-client/src/hooks/` — Custom React hooks
- `packages/desktop-client/e2e/` — E2E tests with page models in `e2e/page-models/`
- `packages/component-library/src/icons/` — Auto-generated icons (don't edit)

## Code Conventions

**TypeScript**: Use `type` over `interface`. Avoid `enum` (use objects/maps). Avoid `any`/`unknown`. Use `satisfies` over `as`. Use inline type imports: `import { type MyType } from '...'`.

**React**: Functional components only, no `React.FC`. Use named exports. Use custom hooks from `src/hooks` (not react-router directly). Use `useDispatch`/`useSelector`/`useStore` from `src/redux` (not react-redux). Use `<Link>` not `<a>`.

**i18n**: All user-facing strings must be translated. Use `Trans` component over `t()` function. Custom lint rule `actual/no-untranslated-strings` enforces this.

**Financial numbers**: Wrap with `FinancialText` or apply `styles.tnum`.

**Logging**: Use logger instead of console in loot-core (enforced by `actual/prefer-logger-over-console`).

**Imports**: Never import `@actual-app/web/*` in loot-core. Never import colors directly (use theme). Import uuid as `import { v4 as uuidv4 } from 'uuid'`.

## Quality Rules

- No `@ts-strict-ignore` comments (project uses typescript-strict-plugin)
- No `eslint-disable` or `oxlint-disable` comments
- No new settings for minor UI tweaks — prefer hardcoded values or theme tokens
- Minimize test mocking — prefer real implementations
- Tests use Vitest with globals (`describe`, `it`, `expect` available without import)

## Tooling Notes

- **Lage** orchestrates tasks across packages (config: `lage.config.js`). Cache in `.lage/` — delete if tests behave unexpectedly.
- **Husky** pre-commit hook runs lint-staged (oxfmt + oxlint). Run `yarn prepare` after fresh install.
- **Node.js >=22** and **Yarn ^4.9.1** required. Native modules (`better-sqlite3`, `bcrypt`) need build tools.
