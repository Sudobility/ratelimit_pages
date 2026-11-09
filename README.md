# @sudobility/ratelimit_pages

Self-contained rate limit page components for React applications: a usage dashboard,
a usage-history chart, and a combined tabbed dashboard.

**Package**: `@sudobility/ratelimit_pages` — published to GitHub Packages
(`https://npm.pkg.github.com`, `@sudobility` scope). ESM-only library.

## Installation

This package and its `@sudobility/*` dependencies live on GitHub Packages, so the
consuming project needs an `.npmrc` pointing the scope at that registry (plus a token
with `read:packages` for installs):

```
@sudobility:registry=https://npm.pkg.github.com
```

```bash
bun add @sudobility/ratelimit_pages
```

### Peer Dependencies

Required in the consuming app:

- `react` >= 18 (18 or 19), `react-dom` >= 18
- `@sudobility/types` — common types (`NetworkClient`, `RateLimitPeriodType`)
- `@sudobility/design` — design tokens (`colors`)
- `@sudobility/components` — base UI components, incl. the `@sudobility/components/ratelimit` subpath (`UsageDashboard`, `TierComparisonTable`, `UsageHistoryChart`)
- `@sudobility/ratelimit_client` — data layer (`useRateLimits` hook); required, declared via `peerDependenciesMeta`

## Usage

```tsx
import { RateLimitsDashboard } from '@sudobility/ratelimit_pages';

<RateLimitsDashboard
  networkClient={networkClient}
  baseUrl="https://api.example.com"
  token={authToken}          // Firebase ID token, string | null
  entitySlug="my-org"        // optional; falls back to the user's personal entity
  labels={labels}            // required — see note below
  onUpgradeClick={() => navigate('/subscribe')}
  initialTab="limits"
/>
```

> **Labels are required and fully specified.** Each component takes a `labels` object
> with every string provided (`Required<...Labels>`). There are no built-in English
> defaults — supply all label fields (the dashboard nests `limitsPage` and
> `historyPage` label objects). See `src/types.ts` for the exact shape.

## API

### Page Components

| Component | Description |
|-----------|-------------|
| `RateLimitsPage` | Usage dashboard: usage bars (hourly/daily/monthly) + tier comparison table, auto-fetch, optional refresh interval, loading/error states with retry |
| `RateLimitHistoryPage` | Historical usage chart with hourly/daily/monthly period tabs |
| `RateLimitsDashboard` | Combines the two pages behind `limits` / `history` tabs |

### Key props (shared)

All three components share `networkClient`, `baseUrl`, `token: FirebaseIdToken`
(`string | null`), optional `entitySlug`, `className`, `testMode`, and a required
`labels` object. Component-specific props:

- `RateLimitsPage`: `autoFetch` (default `true`), `refreshInterval` (ms, `0` = off), `onUpgradeClick`, `upgradeButtonLabel`
- `RateLimitHistoryPage`: `autoFetch`, `initialPeriodType`, `chartHeight`, `barColor`, `limitLineColor`, `showLimitLine`
- `RateLimitsDashboard`: `initialTab` (`'limits' | 'history'`, default `'limits'`), `chartHeight` (default `350`), `onUpgradeClick`, `upgradeButtonLabel`

See `src/types.ts` for the full prop and label interfaces.

### Type Exports

`FirebaseIdToken`, `RateLimitsPageLabels`, `RateLimitsPageProps`, `HistoryPeriodType`,
`RateLimitHistoryPageLabels`, `RateLimitHistoryPageProps`, `RateLimitsDashboardTab`,
`RateLimitsDashboardLabels`, `RateLimitsDashboardProps`

### Utilities

`cn` — Tailwind class-name merge utility (`clsx` + `tailwind-merge`).

## Development

```bash
bun install          # install deps (GitHub Packages registry — see Installation)
bun run build        # tsc (emit .d.ts) + vite build (ESM) + fix-esm-imports post-step
bun run dev          # vite build --watch
bun run type-check   # tsc --noEmit  (note: hyphenated name)
bun run lint         # eslint .
bun run lint:fix     # eslint . --fix
bun run format       # prettier --write
bun run test:unit    # vitest run (excludes *.integration.test)
bun run test:watch   # vitest in watch mode
```

Pre-commit (no `verify` script — run manually):

```bash
bun run type-check && bun run lint && bun run test:unit && bun run build
```

## Build notes

- **ESM-only** library (Vite library mode, `formats: ['es']`) → `dist/index.js` + `dist/index.d.ts`.
- `scripts/fix-esm-imports.mjs` runs after `vite build` to rewrite extensionless relative
  specifiers in the emitted `.js`/`.d.ts` to fully-qualified paths so Node's native ESM
  loader can resolve them.

## Publishing

Publishing is handled by CI (`.github/workflows/ci-cd.yml`, `publish-target: github`) on
pushes to `main`/`develop`. `dist/` is built from source during that workflow.

## License

BUSL-1.1
