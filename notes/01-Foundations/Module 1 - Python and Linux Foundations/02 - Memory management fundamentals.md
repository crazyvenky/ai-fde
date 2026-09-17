# Memory Management Fundamentals

## What & Why

Python manages memory automatically, but "automatic" doesn't mean "invisible." Understanding reference counting and garbage collection explains why some objects free instantly, why others don't, and why a long-running FastAPI or agent process can quietly grow its memory footprint over days.

## How It Works

- **Reference counting**: every object tracks how many references point to it. When the count hits zero, CPython frees it immediately.
- **Generational garbage collector**: catches reference *cycles* (e.g., object A refers to B, B refers back to A) that reference counting alone can't clean up. Objects are grouped into 3 generations; younger generations are scanned more often.
- **`is` vs `==`**: `is` checks identity (same object in memory); `==` checks value equality (calls `__eq__`). Small integers and interned strings can make `is` appear to "work" for equality by coincidence — never rely on that.

```python
import sys
a = [1, 2, 3]
b = a
print(sys.getrefcount(a))  # reference count, includes the temp arg itself
```

## Pitfalls

- **Mutable default arguments**: `def add_item(item, bucket=[])` — the list `bucket` is created *once* at function definition time and reused across every call that doesn't pass its own. Fix: `def add_item(item, bucket=None): bucket = bucket or []`.
- **Growing caches / module-level lists** in a long-running server process — nothing is technically "leaked" (still referenced), but memory never comes back down. This is the most common real-world "leak" in Python services.
- **Unclosed resources** (file handles, DB connections, HTTP sessions) holding references alive longer than intended — always prefer context managers (`with`).

## Connections

Directly relevant to Module 14 (*AI Observability and Gateway Management*) — memory growth in a long-running agent/gateway process is exactly the kind of thing production monitoring needs to catch before it causes an outage.
