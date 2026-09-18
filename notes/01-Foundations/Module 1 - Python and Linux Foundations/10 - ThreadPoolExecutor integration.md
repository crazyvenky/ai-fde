# ThreadPoolExecutor Integration

## What & Why

Sometimes you're stuck calling a blocking, non-async library (a sync DB driver, a CPU-light but blocking SDK call) from inside async code. `ThreadPoolExecutor` lets you run that blocking call on a background thread so it doesn't freeze the entire event loop for every other concurrent request.

## How It Works

```python
import asyncio
from concurrent.futures import ThreadPoolExecutor

def blocking_call(x: int) -> int:
    # some synchronous, blocking library call
    return x * x

async def main():
    loop = asyncio.get_running_loop()
    with ThreadPoolExecutor(max_workers=4) as pool:
        result = await loop.run_in_executor(pool, blocking_call, 5)
    print(result)
```

- `loop.run_in_executor(pool, fn, *args)` schedules `fn` to run on a worker thread and returns an `await`-able Future — the event loop stays free to handle other tasks while it waits.
- If `pool` is omitted, asyncio uses a default shared thread pool.

## Comparisons / Tradeoffs

| Approach | When to use |
|---|---|
| Native async client (`httpx`, async DB driver) | Always prefer this first — no thread overhead |
| `run_in_executor` + ThreadPoolExecutor | When only a blocking library is available and it's I/O-bound |
| `multiprocessing` | When the blocking call is actually CPU-bound, not I/O-bound |

## Pitfalls

- Reaching for this as a default instead of checking if an async-native alternative exists — threads have real overhead (context switching, GIL contention) that a native async client avoids.
- Using it for CPU-bound work — the GIL still serializes actual computation across threads; you need `multiprocessing` for genuine parallel CPU work.

## Connections

Relevant wherever a FastAPI endpoint (Module 2) needs to call a blocking library, and ties directly back to the tradeoffs laid out in *Concurrency vs parallelism*.
