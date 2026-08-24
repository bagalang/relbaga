# relbaga — gaps

## R1 — ~~no automatic jitter~~ — closed

`rel_backoff_jitter_ms` / `rel_retry_jitter` — deterministic jitter in
`[d - d/2, d]`, mixed from `seed + attempt`. Optional; the plain
`rel_backoff_ms` path stays exact.

## R2 — breaker not shared across threads without mutex

Maps/`Breaker` struct must be owned by one loop or guarded by `mutex`.

## R3 — ~~retry only for `fn(i64)->i64`~~ — closed (M21)

`rel_retry<T>` / `rel_retry_jitter<T>` take `fn(T) -> i64`. The op still
returns 0 on success (no effect polymorphism on the failure itself).

## R4 — extra `bh_release` donates a token

**Symptom.** `bh_new(2)` + three releases → three successful acquires.
The channel is the semaphore; there is no `held` counter (that would
need a mutex — same as R2).

**Workaround.** Pair every acquire with one release.

**Severity.** Medium if a `?` path skips release.

**Verdict.** Documented. Mutex-guarded held count is P1 with R2.

## R5 — `brk_on_start` then `brk_allow` denies the probe

**Symptom.** `on_start` sets `half_inflight` first; `allow` then sees a
busy half-open and returns 0. OPEN-expired `allow` without `tick` also
stays 0 (dead `if now >= open_until` branch).

**Workaround.** `brk_start` (tick + allow + reserve). Or tick, allow,
then on_start — the old test order.

**Severity.** High for the half-open probe.

**Verdict.** Closed 2026-08-24 (`brk_start` + dead branch removed).
