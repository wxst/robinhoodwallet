# AGENTS.md

## Repository purpose

This repository is a downstream product fork of `1250237215/robinhoodwallet`. The upstream project is a self-hosted Node.js smart-wallet and social-activity radar for Robinhood Chain, Base, BSC, Solana, X, Telegram, Fomo, and related data sources.

The downstream goal is to add a focused Wind-style real-time information monitor without making routine upstream upgrades unnecessarily difficult.

## Downstream and upstream boundaries

- `main` is the downstream integration branch. Never replace it with upstream history and never force-push it.
- The canonical upstream is `1250237215/robinhoodwallet:main`.
- `.github/workflows/upstream-sync.yml` creates a reviewed merge pull request when upstream changes.
- Keep downstream product code in `src/wind/`, `public/wind/`, `test/wind/`, and `docs/wind/` whenever practical.
- Treat existing files outside those directories as upstream-owned. Modify them only through small, explicit integration points when isolation is impossible.
- Do not rename or delete upstream services merely to hide them from the Wind UI. Prefer separate entry points, routes, adapters, and feature configuration.
- Keep MIT license notices and upstream attribution intact.

## Runtime and architecture

- Node.js 22.13.0 or newer.
- ECMAScript modules throughout the Node.js code.
- Production services are bundled with esbuild.
- SQLite databases are runtime state and must never be committed.
- The social subsystem lives under `src/social/`; chain services remain isolated under their existing directories.
- The current web application under `public/` is large vanilla JavaScript. New Wind UI code must be split into focused modules rather than added to the existing monolithic `public/app.js`.

## Development workflow

1. Create a dedicated branch from current `main`.
2. For behavior changes, write a focused failing test first and verify the expected failure.
3. Implement the smallest coherent change and keep downstream files isolated.
4. Run the narrow tests while iterating.
5. Before marking a pull request ready, run:

```bash
npm ci
npm test
npm run build:all
git status --short
```

6. Open changes as a pull request. GitHub Copilot code review is requested automatically when the pull request is ready.
7. Address or explicitly resolve every actionable review thread before merging.
8. Prefer squash merge for downstream feature pull requests. Upstream synchronization pull requests preserve their generated merge commit.

## Engineering constraints

- Preserve the initial low-latency event path; market, risk, holder, translation, and media enrichment must remain asynchronous or bounded.
- Normalize external events behind adapters. Do not make the UI depend directly on provider-specific payloads.
- Stable IDs, deterministic deduplication, canonical timestamps, restart recovery, and bounded retention are required for real-time feeds.
- Validate and sanitize all external text, media URLs, source URLs, contract addresses, chain identifiers, and webhook payloads.
- Use parameterized SQL and atomic transactions for multi-step state changes.
- Explicitly bound timeouts, retries, concurrency, cache size, polling intervals, and response sizes.
- Never log or commit cookies, Telegram sessions, private keys, API keys, Bark endpoints, bridge tokens, production hosts, or raw authorization headers.
- A `pull_request_target` workflow must not check out or execute pull-request head code.

## Pull request expectations

A pull request description must explain the user-visible behavior, affected integration points, upstream-owned files touched, validation performed, and any deployment or migration step. Avoid mixing unrelated refactors with a feature or fix.