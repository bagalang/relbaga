# relbaga

**Resilience primitives** for Baga services (Track **C6**).

This repository is the package. The compiler and `std` stay in the baga
language monorepo. Check this tree out as `app-product/relbaga` there
(git submodule) so path deps and `-I app-product` keep working.

| | |
|--|--|
| **sandak** | `relbaga` **0.2.0** |
| **Gaps** | [gaps.md](gaps.md) |

## Checkout

Inside a baga language clone:

```bash
git submodule update --init app-product/relbaga
# or, first time from a fresh baga tree without the submodule recorded:
git clone git@github.com:bagalang/relbaga.git app-product/relbaga
```

`sandak.toml` keeps `std = { path = "../../std" }`.
`tests/rel_test.baga` stays in baga.

| Tool | API |
|------|-----|
| Exponential backoff | `rel_backoff_ms(attempt, base, max, mult)` |
| Retry loop | `rel_retry<T>(op, arg, max_attempts, base, max, mult)` — `op` returns 0 on success |
| Jittered backoff | `rel_backoff_jitter_ms` / `rel_retry_jitter` — deterministic `[d/2, d]` |
| Circuit breaker | `brk_new` / `brk_start` / `brk_success` / `brk_failure` (or tick/allow/on_start) |
| Bulkhead | `bh_new(cap)` / `bh_acquire` / `bh_try_acquire` / `bh_release` (chan tokens) |

```baga
rel_retry(flaky, 0, 5, 10, 200, 2)?

let mut b = brk_new(3, 1000)
let g = brk_start(b, now)
if g.ok == 1 {
    b = g.brk
    // … call …
    b = brk_success(b, now)   // or brk_failure
}

let bh = bh_new(4)?
bh_acquire(bh)?
// … limited work …
bh_release(bh)?
```

`brk_start` ticks, checks allow, and reserves the half-open probe.
Do **not** call `brk_on_start` then `brk_allow` — that denies the probe.

Honest: jitter is opt-in and deterministic (seeded); breaker is
single-threaded state (caller serializes); bulkhead is process-local
(extra `bh_release` adds a token — do not over-release).

## License

[MIT](LICENSE) — Copyright (c) 2026 Dim Gigov.
