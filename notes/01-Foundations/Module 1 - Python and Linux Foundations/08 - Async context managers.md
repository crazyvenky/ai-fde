# Async Context Managers

## What & Why

Regular `with` blocks (file handles, locks) assume setup/teardown is instant. Async context managers extend the same guaranteed-cleanup pattern to resources whose setup/teardown itself involves waiting on I/O — like opening a pooled database connection or an HTTP session.

## How It Works

```python
class AsyncDBConnection:
    async def __aenter__(self):
        self.conn = await open_connection()   # async setup
        return self.conn

    async def __aexit__(self, exc_type, exc, tb):
        await self.conn.close()               # async teardown, always runs

async def main():
    async with AsyncDBConnection() as conn:
        await conn.execute("SELECT 1")
    # connection is guaranteed closed here, even if execute() raised
```

- **`__aenter__` / `__aexit__`**: the async equivalents of `__enter__`/`__exit__`, both `await`-able.
- Most async libraries (async DB drivers, `httpx.AsyncClient`, MCP client sessions) already implement this — you consume it with `async with`, you rarely need to write your own.

## Pitfalls

- Using a plain `with` on an object that only supports `async with` (or vice versa) — raises a `TypeError` at runtime, not at write-time.
- Opening an async resource manually (`conn = await open_connection()`) and forgetting to close it on an exception path — this is exactly what `async with` exists to prevent.

## Connections

Used for async database drivers in Module 11 (*Establishing secure connections using pyodbc, oracledb, and SQLAlchemy*) and for MCP client/server session handling in Module 10 (*Host/Client/Server architectures*).
