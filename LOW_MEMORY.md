# Khoorooth Router — 128 MiB VPS experiment

This is an experimental profile, not a guaranteed minimum requirement. Next.js/Node can exceed 128 MiB RSS due to native memory, TLS, buffers, runtime overhead, and request concurrency. Node heap limits do not cap total process RSS.

## Start
```bash
cp .env.example .env
# Configure provider keys and settings in .env
docker compose -f docker-compose.low-memory.yml up -d
docker stats khoorooth-router
docker logs -f khoorooth-router
```

Build the image on a larger machine; do not run Next.js build on the 128 MiB VPS. This profile intentionally omits the bundled Headroom container; configure HEADROOM_URL to an external instance if needed.

If it exits with code 137 or restarts under load, the host cannot sustain the workload. Consider 256–512 MiB minimum for light single-user use, and more for concurrent requests. A strict no-swap requirement means OOM cannot be avoided by swap.
