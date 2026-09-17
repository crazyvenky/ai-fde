# Coroutines and Tasks

## What & Why

`async def` and `await` are the syntax; coroutines and Tasks are what actually get scheduled by the event loop underneath them. Knowing the difference between "a coroutine object" and "a running Task" avoids a very common beginner bug: awaiting things in the wrong order and running them sequentially by accident.

## How It Works

```python
import asyncio

async def fetch(url: str) -> str:
    await asyncio.sleep(1)     # simulates I/O wait
    return f"result from {url}"

async def main():
    # Sequential — takes ~3 seconds total
    a = await fetch("a")
    b = await fetch("b")

    # Concurrent — takes ~1 second total
    results = await asyncio.gather(fetch("a"), fetch("b"), fetch("c"))
```

- **Coroutine**: calling `fetch("a")` doesn't run it — it creates a coroutine *object*, a paused function waiting to be driven.
- **Task**: `asyncio.create_task(fetch("a"))` schedules that coroutine to actually run on the event loop, concurrently with other tasks.
- **Future**: a lower-level "promise" of a result; Tasks are built on top of Futures. You rarely create Futures directly in application code.
- **`asyncio.gather(...)`**: runs multiple coroutines/Tasks concurrently and waits for all of them, returning results in the same order given.

## Pitfalls

- Writing `await fetch("a"); await fetch("b")` when you meant concurrency — this runs strictly sequentially, one full await at a time.
- Creating a Task with `asyncio.create_task()` but never awaiting or storing it — it can be garbage-collected mid-execution, silently dropping the work.

## Connections

Directly the mechanism behind LangGraph's *Running agent tasks concurrently using async execution* (Module 9) — parallel graph branches are Tasks under the hood.
