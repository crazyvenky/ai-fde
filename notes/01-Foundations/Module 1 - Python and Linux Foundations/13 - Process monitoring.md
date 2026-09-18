# Process Monitoring

## What & Why

When a deployed service misbehaves — pegged CPU, stuck request, memory growth — the first response is always on the command line, not in application logs. Knowing how to inspect and safely stop processes is a baseline production skill.

## How It Works

```bash
ps aux | grep uvicorn        # list processes matching "uvicorn"
top                          # live view: CPU/memory usage per process
htop                         # nicer interactive version of top (if installed)

kill -15 <pid>                # SIGTERM: ask the process to shut down gracefully
kill -9  <pid>                # SIGKILL: force-terminate immediately, no cleanup
```

- **`ps aux`**: snapshot of all running processes — user, PID, CPU%, memory%, command.
- **`top`/`htop`**: live, continuously refreshing view — sort by CPU or memory to spot the runaway process.
- **Signals**: `SIGTERM` (15) is a polite request a well-behaved process can catch and clean up after (closing DB connections, flushing logs) before exiting; `SIGKILL` (9) terminates immediately with no chance to clean up — a last resort.
- **Log locations**: application logs typically under `/var/log/` or wherever the app/container is configured to write; always check these *before* killing a suspicious process, since the process might explain itself.

## Pitfalls

- Reaching for `kill -9` by default — skips graceful shutdown, which can leave open DB connections, half-written files, or corrupted state.
- Killing a process without first checking *why* it's using resources — sometimes it's legitimately processing a large batch, not actually stuck.

## Connections

Directly relevant once services are containerized and orchestrated in Module 4 — ECS/Fargate healthchecks and auto-scaling are essentially automated versions of this same monitor-and-react loop.
