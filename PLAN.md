# relbaga — plan

Date: 2026-08-05 · Status: **P0 done** (C6) · **0.2.0** `brk_start` (2026-08-24)

## P0 ✅

Retry/backoff, circuit breaker, bulkhead, tests.

## P1

Jitter ✅ (`rel_backoff_jitter_ms` / `rel_retry_jitter`).
`brk_start` ✅ (half-open probe without the on_start/allow footgun).
Shared breaker via mutex helper, metrics hooks, bulkhead over-release
guard — still open.
