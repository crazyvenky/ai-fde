# Dependency Injection

## 1. What This Is & Why an FDE Needs It

Dependency Injection (DI) is a design pattern where a piece of code declares *what it needs* (a DB session, the current authenticated user, a config object) without knowing or caring *how that thing gets constructed*. The framework — FastAPI, in this course — is responsible for building that dependency and handing it to the function.

Why this matters specifically for enterprise AI systems: every endpoint in OmniGuard and AuditMesh needs the *same* handful of things repeated everywhere — an authenticated user, a scoped DB connection, a permission check, a request-scoped logger. Without DI, this logic gets copy-pasted into every endpoint (error-prone, hard to change centrally). With DI, it's written once and *declared* wherever it's needed — and, critically, it becomes trivially swappable in tests (see *Fixtures and mocking*).

## 2. Core Mechanics

```python
from fastapi import Depends, FastAPI, HTTPException, Header

app = FastAPI()

# A dependency is just a callable (function or class) FastAPI will call for you
def get_db_session():
    session = create_db_session()
    try:
        yield session          # "yield" dependencies run cleanup after the request
    finally:
        session.close()

def get_current_user(authorization: str = Header(...)):
    user = decode_and_validate_token(authorization)
    if user is None:
        raise HTTPException(status_code=401, detail="Invalid token")
    return user

@app.get("/orders")
def list_orders(
    db = Depends(get_db_session),
    user = Depends(get_current_user),
):
    return db.query(Order).filter(Order.user_id == user.id).all()
```

Three things are happening that make this powerful:

1. **FastAPI resolves the dependency graph automatically.** `list_orders` doesn't call `get_db_session()` or `get_current_user()` itself — it just declares them as parameters with `Depends(...)`, and FastAPI calls them, in the right order, before running the endpoint body.
2. **`yield`-style dependencies handle cleanup.** Code after `yield` runs *after* the response is sent (or if an exception occurs) — this is the correct place to close a DB session, release a lock, etc. It's the async-endpoint equivalent of a context manager.
3. **Dependencies can depend on other dependencies.** `get_current_user` could itself take `db = Depends(get_db_session)` to look up the user in the database — FastAPI resolves the whole chain.

## 3. Dependency Chaining in Practice

```python
def get_db_session():
    ...

def get_current_user(db = Depends(get_db_session), token: str = Header(...)):
    return db.query(User).filter(User.token == token).first()

def require_admin(user = Depends(get_current_user)):
    if not user.is_admin:
        raise HTTPException(status_code=403, detail="Admin access required")
    return user

@app.delete("/users/{user_id}")
def delete_user(user_id: int, admin = Depends(require_admin)):
    # by the time this line runs, we know: DB session was created,
    # user was authenticated, AND user was confirmed to be an admin
    ...
```

FastAPI caches dependency resolution *within a single request* by default — if both `get_current_user` and some other dependency both need `get_db_session`, FastAPI calls it once and reuses the result, not twice.

## 4. Overriding Dependencies for Testing

This is the single biggest practical payoff of DI, and it's why it's taught right before the testing topics in this module:

```python
from fastapi.testclient import TestClient

def override_get_db_session():
    yield fake_in_memory_db_session()

app.dependency_overrides[get_db_session] = override_get_db_session

client = TestClient(app)
response = client.get("/orders")   # uses the fake DB, no real database touched
```

Without DI, testing an endpoint that hits a real database means either standing up a real test database for every test run, or invasive monkey-patching. With DI, you swap the *entire dependency* for a test double in one line, and the endpoint code never changes.

## 5. Comparisons / Design Notes

| Pattern | Tradeoff |
|---|---|
| DI via `Depends()` | Explicit, testable, FastAPI-native — the standard approach |
| Global singletons (module-level DB connection) | Simple but untestable in isolation, and dangerous with concurrent requests sharing mutable state |
| Manual construction inside each endpoint | No abstraction cost, but massive duplication and no easy test override |

## 6. Real-World Enterprise Scenario

In AuditMesh (Module 16), every endpoint that touches Jira needs: an authenticated service identity, a scoped MCP client session, and an audit logger that records who requested what (for the compliance trail the whole project exists to produce). Structuring these as dependencies means:

- The MCP client session dependency can enforce **trust boundaries** (Module 10) in one place — every endpoint automatically gets a correctly-scoped session, no endpoint can accidentally get a broader one by forgetting a check.
- The audit logger dependency guarantees **every** request is logged consistently, because it's impossible to call an endpoint without FastAPI resolving that dependency first.

## 7. Common Pitfalls

- **Doing expensive work in a dependency that's called on every request** without caching — e.g., re-reading a config file from disk on every single call. Cache expensive, rarely-changing dependencies at the application level (`lru_cache`, or resolve once at startup) rather than per-request.
- **Forgetting `yield`-based cleanup entirely** — opening a DB session or file handle in a dependency with a plain `return` (not `yield`) means there's no framework-guaranteed place to close it.
- **Hiding business logic inside a dependency where it's hard to find** — dependencies are for cross-cutting concerns (auth, DB sessions, request-scoped context), not the core logic of what an endpoint actually does. If a "dependency" contains the real business logic, it should probably be a normal function called explicitly inside the endpoint.
- **Not overriding dependencies in tests and instead trying to mock deep internals** — fighting the framework instead of using `dependency_overrides` leads to brittle, implementation-detail-coupled tests.

## 8. Self-Check Questions

1. Why does FastAPI only call a shared dependency once per request even if multiple other dependencies need it?
2. What's the practical benefit of a `yield`-based dependency over a `return`-based one when the dependency opens a resource like a DB session?
3. In the AuditMesh scenario, why does putting the audit-logging dependency at the framework level (rather than as a line of code inside each endpoint) provide a stronger compliance guarantee?

## 9. Connections

- **CORS middleware** (next topic): another example of framework-level, cross-cutting request handling, though at the middleware layer rather than the dependency layer.
- **Fixtures and mocking** (Module 2): `dependency_overrides` is the direct mechanism that makes FastAPI endpoints cleanly testable — this topic is the prerequisite for that one.
- **Implementing Role-Based Access Control** (Module 12): the `require_admin`-style dependency chain shown above is exactly how RBAC checks get wired into real endpoints.
- **Host/Client/Server architectures** (Module 10): the MCP client session example above previews how DI-style scoping applies to agent tool access, not just HTTP endpoints.
