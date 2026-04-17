# @regboy/retailctrl-contracts

Shared API contracts (Zod schemas + inferred TypeScript types) for the
RetailCtrl services. Backend and frontend both depend on this package so
the shape of every HTTP exchange has exactly one source of truth.

## Layout

```
src/
└── priceCheck/
    ├── schemas.ts   Zod schemas (exchange shapes, constraints, warnings)
    ├── index.ts     Inferred type exports
    └── __tests__/   Contract tests — parse a real fixture through each schema
fixtures/
└── compare-response.sample.json   Captured /price-check/compare response
```

## Using locally (phase A — `file:` link)

From the consumer repo (backend or frontend):

```bash
pnpm add file:../retailctrl-contracts
```

Rebuild the contracts package after every change so consumers pick it up:

```bash
cd retailctrl-contracts && pnpm build
```

## Publishing to GitHub Packages (phase B)

First-time setup:

1. Create a GitHub repo (public or private) at `regboy/retailctrl-contracts`.
   Push the current branch.

2. The workflow at `.github/workflows/publish.yml` triggers on tag push.
   Tagging `v0.1.0` and pushing publishes to GitHub Packages automatically:

   ```bash
   git tag v0.1.0
   git push origin v0.1.0
   ```

3. The workflow uses `GITHUB_TOKEN` (automatic). Nothing else to configure.

### Consumer setup after first publish

Swap the `file:` dep for the registry version. Both consumers need an
`.npmrc` pointing GitHub Packages at the `@regboy` scope:

```
# .npmrc (in each consumer repo — gitignore'd)
@regboy:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken=${GITHUB_TOKEN}
```

Set `GITHUB_TOKEN` to a Personal Access Token (classic) with
`read:packages` scope. `pnpm install` then resolves `@regboy/retailctrl-contracts`
from GitHub Packages.

Update consumer `package.json`:

```diff
- "@regboy/retailctrl-contracts": "file:../retailctrl-contracts",
+ "@regboy/retailctrl-contracts": "^0.1.0",
```

### Releasing a new version

```bash
# Bump version, run tests, build, commit, tag, push
pnpm version patch  # or minor / major
git push --follow-tags
```

The GitHub Action picks up the tag and publishes.

## Versioning rule

Semver. Breaking changes to any exported schema bump **major**. Consumers
pin to `^major.minor.0` and upgrade on their own schedule — drift is
explicit, never silent.

## Scripts

```bash
pnpm build        # tsc → dist/
pnpm typecheck    # tsc --noEmit
pnpm test         # vitest run
pnpm test:watch   # vitest
```
