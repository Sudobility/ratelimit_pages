# RateLimit Pages

> **Git policy — never auto-commit or auto-push.** Leave work in the working tree.
> Run `git commit`, `git push`, or `gh pr create` **only when the user explicitly asks
> in that turn**. Approval for an earlier change does not carry forward. Stay on the
> current branch (currently `main`); do not create feature branches.

Self-contained rate limit page components for React apps: a usage dashboard, a
usage-history chart, and a combined tabbed dashboard.

**Package**: `@sudobility/ratelimit_pages` — published to GitHub Packages (not npmjs).

## Tech Stack

- **Language**: TypeScript (strict). **Runtime/PM**: Bun (do not use npm/yarn/pnpm to install).
- **Build**: Vite library mode, **ESM-only** (`formats: ['es']`) → `dist/index.js` + `dist/index.d.ts`.
- **Test**: Vitest (jsdom, Testing Library).
- **Lint/format**: ESLint (flat config) + Prettier.

## Commands

```bash
bun run build        # tsc (emits .d.ts) && vite build && node scripts/fix-esm-imports.mjs
bun run dev          # vite build --watch
bun run type-check   # tsc --noEmit  (hyphenated — see gotcha)
bun run lint         # eslint .
bun run lint:fix     # eslint . --fix
bun run format       # prettier --write
bun run test:unit    # vitest run (excludes *.integration.test)
bun run test:watch   # vitest watch
```

Pre-commit (no `verify` script): `bun run type-check && bun run lint && bun run test:unit && bun run build`

## Project Structure

```
src/
├── index.ts                      # Public exports (3 components, types, cn)
├── types.ts                      # All prop + label interfaces
├── lib/cn.ts                     # Tailwind class merge (clsx + tailwind-merge)
├── pages/
│   ├── RateLimitsPage.tsx        # Usage bars + tier comparison
│   ├── RateLimitHistoryPage.tsx  # Usage-history chart w/ period tabs
│   └── RateLimitsDashboard.tsx   # Tabs combining the two pages
└── __tests__/                    # setup.ts + *.test.tsx / *.test.ts
scripts/fix-esm-imports.mjs       # post-build ESM specifier rewrite
```

There is no `pages/index.ts`; `src/index.ts` imports each page module directly.

## Architecture

- **Data**: pages call `useRateLimits(networkClient, baseUrl, testMode)` from
  `@sudobility/ratelimit_client`, then `refreshConfig(token, entitySlug ?? '')`.
- **UI primitives**: come from `@sudobility/components` and its **`@sudobility/components/ratelimit`
  subpath** (`UsageDashboard`, `TierComparisonTable`, `UsageHistoryChart`). Design tokens
  from `@sudobility/design` (`colors`).
- Pages wrap themselves in `<Section>` internally — consumers must **not** wrap them again.
- `RateLimitsDashboard` renders `RateLimitsPage` / `RateLimitHistoryPage` per active tab.

## Conventions

- **Labels are required and exhaustive.** Component `labels` props are
  `Required<...Labels>` — no built-in default strings; callers supply every field.
  `RateLimitsDashboardLabels` nests `limitsPage` + `historyPage` label objects.
- `token` is `FirebaseIdToken` = `string | null`; effects no-op when falsy.
- Styling via Tailwind classes merged with `cn`.

## Gotchas

- **Typecheck script is `type-check` (hyphenated).** `bun run typecheck` does nothing.
- **Test script is `test:unit`, not `test`.** There is no `bun run test`.
- **ESM-only build, 3-step.** Build is `tsc && vite build && node scripts/fix-esm-imports.mjs`;
  the final step rewrites extensionless relative imports so Node's ESM loader resolves them.
  There is no UMD/CJS output.
- **No separate `ratelimit-components` package.** Rate-limit UI lives under the
  `@sudobility/components/ratelimit` subpath of `@sudobility/components`.
- **`@sudobility/ratelimit_client` is required but declared in `peerDependenciesMeta`**
  (`optional: false`), not in `peerDependencies`. Check both sections when debugging deps.
- **GitHub Packages registry.** `.npmrc` sets `@sudobility:registry=https://npm.pkg.github.com`;
  installs need a token with `read:packages`. Publishing is done by CI, not `npm publish` locally.

## Workspace Context

Part of the **ShapeShyft** multi-project workspace (see `../CLAUDE.md`). Consumed by
downstream apps (e.g. `shapeshyft_app`) that render these dashboard pages. After changes:
run the pre-commit checks; CI publishes on `main`/`develop`; consumers bump + rebuild.
