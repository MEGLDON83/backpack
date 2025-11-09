## Repo summary for AI agents

This is a monorepo (Yarn v4 + Turborepo) containing several packages and a web app. Key top-level artifacts:

- `package.json` — workspace root (engines: Node 20.10.0, Yarn 4). Uses `turbo` for orchestration.
- `turbo.json` — pipeline definition (build/test/start dependencies and important env vars).
- `tsconfig.json` — references showing the package-level TypeScript dependency graph (see `packages/*`).
- `packages/` — main code. notable packages: `app-extension` (Chrome extension), `background`, `secure-*`, `tamagui-core`, `wallet-standard`, `data-components`, `i18n`.

Keep guidance below short and actionable — reference these files when you need to understand dependency order, build outputs, or environment flags.

## Big picture and architecture notes

- Monorepo pattern: packages are built with `turbo`. `turbo` enforces ordering via package references and the `^build` relationship. When editing a package that other packages reference, expect builds/tests to run for dependents.
- Extension & background: `packages/app-extension` and `packages/background` are closely coupled. There are also `secure-background`, `secure-clients`, and `secure-ui` packages used to separate privileged background logic from UI and client code.
- UI system: Tamagui is used (see `packages/tamagui-core` and `TAMAGUI_*` env vars in `turbo.json`). Builds may require platform targets via `TAMAGUI_TARGET`.
- GraphQL/codegen: GraphQL generation outputs are declared in `turbo.json` (e.g. `packages/app-mobile/src/graphql/__generated__`). Use the repo `gql` script to trigger generation.
- i18n: `packages/i18n` contains localization tooling. There is a GitHub workflow `.github/workflows/i18n-update-translation-files-from-locize.yml` that syncs translations from Locize.

## Developer workflows (how to run things)

Root-level scripts you will use most (see `package.json`):

- Install

```powershell
yarn install
```

- Develop (runs turborepo `start` across packages)

```powershell
yarn start
```

Note: on a fresh clone run `yarn build` once before `yarn start`.

- Build all packages

```powershell
yarn build
```

- Run tests (turbo-run)

```powershell
yarn test
```

- Lint / format

```powershell
yarn lint
yarn format
```

Package-scoped starts/builds are available (examples):

```powershell
yarn start:ext    # starts extension-related packages only
yarn build:ext
yarn start:mobile
yarn build:mobile
```

Notes and gotchas:
- Commands often prefix `env-cmd` to inject environment variables. When running locally you can use `.env` files or pass env vars manually.
- The README recommends using `mkcert` for `app-extension` dev so browser ws/ssl errors don't interfere. The dev extension is in `packages/app-extension/dev` after running the dev build.
- Husky is installed in `postinstall` (git hooks). `lint-staged` runs Prettier + ESLint on staged files.

## Project-specific conventions and patterns

- Build orchestration: prefer `turbo` scripts (root `yarn start|build|test`) rather than running individual package builds unless debugging a single package.
- Workspaces: root `workspaces` include `packages/*`, `web`, and `examples`. Some scripts use `yarn workspaces foreach` for cross-package actions (e.g. `format`).
- env handling: many scripts rely on env vars listed in `turbo.json` (`SENTRY_*`, `BACKPACK_*`, `TAMAGUI_*`, `AIRTABLE_API_KEY`). Treat missing envs as an expected local setup issue rather than a code bug.
- Strong TypeScript project references: `tsconfig.json` references show the intended compile order. If TypeScript errors propagate between packages, ensure referenced packages are built first.

## Integration points & external dependencies

- Locize (i18n) — translations are fetched in CI via a workflow and scripts under `packages/i18n`.
- Airtable — there are scripts to sync localizations with Airtable (`scripts/airtable-to-localizations.ts`).
- Sentry — build/CI may use Sentry env variables for source maps and releases.
- GraphQL — codegen happens during the `gql` pipeline step; generated files live under package-specific `__generated__` directories.

## Where to look for examples

- Extension dev flow: `packages/app-extension/` (webpack configs, `dev` output dir). See README section “Install the development version of the extension.”
- Background/secure messaging: `packages/background`, `packages/secure-background`, `packages/secure-clients`.
- UI patterns: `packages/tamagui-core`, `packages/react-common`, and `web/components`.
- Scripts & infra helpers: top-level `scripts/` e.g. `monorepo_depcheck.ts`, `sync-expo-deps.js`.

## Quick checklist for code edits

1. Run `yarn build` for dependent packages, or run the root `yarn start` while developing.
2. Run `yarn lint` / `yarn format` before pushing; husky + lint-staged will also run on commit.
3. If changing GraphQL schemas, run `yarn gql` to regenerate.
4. If making i18n changes, be aware CI workflows will sync to Locize; modify `packages/i18n` scripts only after understanding them.

## If you need more context

Reference these files first: `package.json`, `turbo.json`, `tsconfig.json`, `packages/app-extension/README` (if present), and package-level `package.json` files for script details. If anything above is unclear or you want me to expand a specific section (e.g., build internals for `app-extension` or how `secure-*` packages interoperate), tell me which area and I'll iterate.

-- end
