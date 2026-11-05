# Khoorooth Router: memory constraints and deployment profiles

## Reality check: whole VPS = 128 MiB, no swap, public multi-user API

**The full 9Router/Next.js app cannot responsibly be promised to run this workload in 128 MiB total host RAM.** The host OS, Docker daemon, Node runtime, Next.js, persistent database, and public API traffic all share that memory. Multiple requests and streaming/provider responses can increase RSS. A hard memory limit may cause OOM kills rather than graceful degradation.

The repo's normal Compose stack also runs Headroom separately; that is not included in the tiny profile.

## Changes in the Khoorooth branch

- Optional Next.js `preloadEntriesOnStart: false` using `KHOROOTH_LOW_MEMORY=1`, to reduce eager startup memory.
- Optional cap on h2c downgrade request buffering using `KHOROOTH_H2C_MAX_BODY_BYTES`; oversize request is rejected with HTTP 413.
- Experimental compose sets 8 MiB body limit, 40 MiB V8 old-space cap, and a 72 MiB container hard cap with swap disabled.
- The default build/runtime behavior remains unchanged unless the low-memory variables are set.

## Strict low-memory test

Do **not** build the Next.js project on the 128 MiB VPS. Build the branch on a machine with adequate RAM and make the resulting image available to the VPS:

```bash
docker compose -f docker-compose.low-memory.yml up -d
docker stats khoorooth-router
docker logs --tail=200 -f khoorooth-router
```

Observe idle RSS, startup, API request RSS, and OOM events. The profile may fail to start: 72 MiB is a test ceiling, not a proven working memory budget.

## Multi-user public deployment recommendation

For a public API with concurrent requests, use a lightweight Linux host with **at least 512 MiB RAM as a starting point**, preferably 1 GiB or more depending on model payloads, streaming, providers, and user count. This is not a guarantee; benchmark your real traffic.

If 128 MiB is an absolute requirement, split the architecture:
1. Keep the full dashboard/control plane on another machine.
2. Run only a purpose-built, minimal API gateway on the 128 MiB VPS (ideally a small Go/Rust service, not the Next.js dashboard).
3. Keep provider secrets and auth checks server-side; add explicit request/concurrency limits and backpressure.
4. Use an external database/Headroom service if needed.
5. Load-test and measure RSS before exposing it publicly.

Reducing Node heap does not limit total RSS (native allocations, buffers, TLS, and code memory remain). Docker memory constraints can trigger OOM kills; disabling swap makes failures more immediate rather than making the application fit.
