# Event Loop Architecture

## What & Why

Every "async" thing later in the course — FastAPI endpoints, concurrent agent nodes, streaming LLM responses — runs on top of a single mechanism: Python's asyncio event loop. Understanding it demystifies why `async`/`await` looks the way it does.

## How It Works

Python's event loop is a **single-threaded scheduler**. It doesn't run code in parallel — it rapidly switches between tasks whenever one hits an `await` on something that isn't ready yet (I/O, a timer, another coroutine).

```
┌─────────────────────────────────────────┐
│               Event Loop                 │
│                                           │
│  Task A ──await─┐                        │
│                  ▼                        │
│           [ waiting on I/O ]              │
│                  │                        │
│  Task B ─────────┴─runs while A waits─►   │
│                                           │
│  Task A ◄── I/O ready, resumes ──         │
└─────────────────────────────────────────┘
```

- **Blocking call**: occupies the thread until it finishes (e.g., `time.sleep()`, a synchronous `requests.get()`) — freezes the *entire* event loop, including unrelated tasks.
- **Non-blocking call**: yields control back to the loop while waiting (e.g., `await asyncio.sleep()`, `await httpx.AsyncClient().get()`) — lets other tasks run in the meantime.

## Pitfalls

- Calling a blocking library (e.g., the synchronous `requests` package) inside an `async def` function — it blocks the whole loop, defeating the purpose of async entirely. Use an async-native client (`httpx`, `aiohttp`) or offload to a thread pool (see *ThreadPoolExecutor integration*).
- Assuming async gives you parallelism — on a single core, it only helps with I/O-bound waiting, not CPU-bound work (see *Concurrency vs parallelism*).

## Connections

This is the mechanism underneath FastAPI's async endpoints (Module 2) and LangGraph's concurrent node execution (Module 9, *Running agent tasks concurrently using async execution*).
