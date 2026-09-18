# Concurrency vs Parallelism

## What & Why

Picking threading vs multiprocessing vs asyncio for a task is a direct consequence of one question: is this workload waiting on something external (I/O-bound), or is it burning CPU cycles (CPU-bound)? Getting this wrong is a common source of code that "should be faster" but isn't.

## How It Works

- **Concurrency**: multiple tasks make *progress* over the same time period, but not necessarily at the exact same instant — achieved by interleaving (asyncio, threads under the GIL).
- **Parallelism**: multiple tasks run *simultaneously* on separate CPU cores — requires multiprocessing (separate processes, separate GILs) in standard CPython.
- **The GIL (Global Interpreter Lock)**: only one thread executes Python bytecode at a time, even on a multi-core machine. Threads *do* help for I/O-bound work (the GIL is released while waiting on I/O) but not for CPU-bound work.

## Comparisons / Tradeoffs

| Workload type | Best tool | Why |
|---|---|---|
| I/O-bound (network calls, DB queries, file reads) | `asyncio` | Lightweight, thousands of concurrent tasks, no GIL contention since it's single-threaded |
| I/O-bound, using a blocking (non-async) library | `threading` | GIL releases during I/O, so threads still help |
| CPU-bound (heavy computation, parsing, ML inference) | `multiprocessing` | Separate processes, separate GILs → true parallel execution |

## Pitfalls

- Using `threading` for CPU-bound work expecting a speedup — the GIL serializes it anyway, so you get overhead without the benefit.
- Using `multiprocessing` for I/O-bound work — the overhead of spinning up separate processes outweighs any benefit `asyncio` would have given for free.

## Connections

Directly informs the next topic, *ThreadPoolExecutor integration*, and explains why LangGraph's parallel node execution (Module 9) is concurrency, not necessarily CPU parallelism.
