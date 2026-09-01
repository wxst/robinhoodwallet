# GitHub Copilot code review instructions

When performing a pull request review, respond in Simplified Chinese.

## Review priorities

Prioritize correctness, security, data integrity, regressions, production operability, and upstream compatibility over style. Leave a comment only when it identifies an actionable issue introduced by the pull request. Avoid compliments, broad summaries, and low-value formatting comments.

Label findings by impact:

- `[P0]` security compromise, secret exposure, irreversible data loss, or remote code execution.
- `[P1]` production outage, incorrect trading or wallet signal, broken upgrade path, or material data corruption.
- `[P2]` user-visible malfunction, reliability regression, bad error handling, or missing focused test.
- `[P3]` maintainability issue that is likely to cause a concrete future defect.

Every finding should state the failure scenario, why it matters, and the smallest practical fix. Anchor comments to the narrowest relevant line range.

## Repository context

This is a downstream fork of `1250237215/robinhoodwallet`. It is a Node.js 22.13+ ESM application with four chain services, a shared social activity subsystem, browser and Telegram bridges, SQLite persistence, a large vanilla-JavaScript web UI, and esbuild production bundles.

Treat the existing upstream implementation as upstream-owned. New downstream product work should normally live in `src/wind/`, `public/wind/`, `test/wind/`, and `docs/wind/`. Flag broad edits to upstream-owned files when a small integration adapter, wrapper, route registration, or configuration hook would preserve easier future upstream merges.

## Required review checks

- Preserve isolation between Robinhood, Base, BSC, and Solana data, settings, streams, alerts, and databases.
- Preserve the fast monitoring path: enrichment must not delay delivery of the initial live event.
- Check stable event IDs, timestamp normalization, ordering, deduplication, retries, timeouts, backoff, rate-limit handling, and restart recovery.
- Treat every X, Telegram, Fomo, Pump, RPC, webhook, browser-bridge, and market-data payload as untrusted input.
- Check HTML escaping, URL validation, SSRF boundaries, SQL parameterization, command construction, origin checks, and accidental secret logging.
- Reject committed credentials, cookies, sessions, device tokens, production domains, populated environment files, live databases, logs, generated bundles, and downloaded media.
- Check SQLite migrations and shared-file access for backward compatibility, atomicity, locking, and partial-upgrade behavior.
- Require focused regression tests for behavior changes. Expected validation is `npm ci`, `npm test`, and `npm run build:all`; workflow-only changes must be checked for valid YAML and least-privilege permissions.
- A workflow using `pull_request_target` must never check out or execute code, scripts, actions, or configuration from the pull request head branch.
- Do not suggest unrelated refactors or stylistic rewrites.

## Upstream synchronization

Review upstream-sync pull requests especially carefully for overwritten downstream behavior, renamed routes, schema conflicts, duplicated services, dependency changes, and edits to downstream-owned directories. Never recommend force-pushing `main` or replacing downstream history with upstream history.