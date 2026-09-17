# Path and Query Parameters

## 1. What This Is & Why an FDE Needs It

Every AI system built in this course — OmniGuard, AuditMesh, or any client-facing tool — is ultimately exposed to the world through HTTP endpoints. Before anything about RAG, agents, or guardrails matters, the API surface itself has to be designed correctly: what identifies *which* resource a client wants (path parameters), and what modifies or filters *how* that resource is returned (query parameters).

Getting this distinction wrong is one of the most common junior-engineer mistakes, and it matters more for an FDE than most: you are frequently designing the *contract* a client's existing systems (Slack bots, internal dashboards, other microservices) will integrate against. A poorly designed contract becomes technical debt that's expensive to change once a client depends on it, because breaking a path structure breaks every caller, whereas a badly designed query parameter can usually be extended non-destructively.

## 2. The Core Distinction

- **Path parameters** identify a *specific resource* — they are part of the URL's hierarchical structure and are (almost always) required. Think of them as "which thing."
- **Query parameters** modify the *request against a resource* — filtering, sorting, pagination, optional flags. They come after `?` and are (almost always) optional. Think of them as "how do you want it."

```
GET /users/42/orders?status=shipped&limit=10&sort=-created_at
        └──┬──┘        └────────────────┬────────────────┘
      path param              query parameters
     (which order set)      (filter/sort/paginate)
```

If removing a parameter from the URL still leaves a *meaningful, resolvable* endpoint, it should be a query parameter. If removing it makes the URL point to nothing (or something entirely different), it should be a path parameter.

## 3. Implementing Both in FastAPI

```python
from fastapi import FastAPI, Query, Path
from typing import Optional

app = FastAPI()

@app.get("/users/{user_id}/orders")
def get_orders(
    user_id: int = Path(..., description="The unique ID of the user", ge=1),
    status: Optional[str] = Query(None, description="Filter by order status"),
    limit: int = Query(10, ge=1, le=100, description="Max results to return"),
    offset: int = Query(0, ge=0, description="Pagination offset"),
):
    # user_id is guaranteed to be present and >= 1 (FastAPI enforces this
    # before your function body even runs)
    # status, limit, offset are optional with sane defaults and bounds
    ...
```

Key things this example demonstrates that a real production API needs:

- **Type coercion is automatic**: `user_id` arrives as a string in the raw URL but FastAPI converts it to `int` and rejects the request with a 422 if it can't.
- **Validation constraints inline**: `ge=1` (greater-or-equal), `le=100` (less-or-equal) prevent a client from requesting `limit=999999999` and accidentally taking down your database with an unbounded query — this is a real production incident pattern, not a theoretical concern.
- **Self-documenting**: FastAPI auto-generates OpenAPI/Swagger docs from these type hints and `description=` fields — this becomes the contract you hand to a client's integration team without writing separate documentation.

## 4. Comparisons / Design Guidance

| Scenario | Use Path Param | Use Query Param |
|---|---|---|
| Identifying a specific record (`/orders/{id}`) | ✅ | |
| Filtering a collection (`?status=active`) | | ✅ |
| Pagination (`?limit=10&offset=20`) | | ✅ |
| Sorting (`?sort=-created_at`) | | ✅ |
| Nested resource ownership (`/users/{id}/orders`) | ✅ | |
| Optional feature flags (`?include_deleted=true`) | | ✅ |

A useful litmus test: **path parameters should never be optional.** If you find yourself wanting an "optional path parameter," that's a signal the parameter actually belongs in the query string, or that you need two separate routes (`/orders` and `/orders/{id}`).

## 5. Real-World Enterprise Scenario

Imagine you're building the OmniGuard API and a client's internal dashboard needs to fetch compliance audit records for a specific department, with optional date-range filtering. The correct design:

```
GET /departments/{department_id}/audits?start_date=2026-01-01&end_date=2026-03-31&severity=high
```

`department_id` is a path parameter — there is no meaningful "list all audits with no department" endpoint at this route; if you need that, it's a *different* endpoint (`/audits`) with `department_id` as an optional *query* filter instead. This is a classic FDE decision point: the same underlying data can be exposed two different ways depending on what the client's actual access pattern is, and you should ask the client which pattern they need rather than guessing.

## 6. Common Pitfalls

- **Using a query parameter to identify a resource** (`/orders?id=42` instead of `/orders/42`) — breaks REST conventions, makes caching harder (many CDNs/proxies cache based on path), and confuses anyone reading the API who expects idempotent GETs on stable resource URLs.
- **Forgetting upper bounds on numeric query params** (`limit`, `page_size`) — a client (or a bug in a client) requesting `limit=10000000` can exhaust database memory or cause a multi-minute query that blocks other requests.
- **Silently ignoring unknown query parameters** — if a client typos `?statuss=active` instead of `?status=active`, and your API just ignores unrecognized params, they'll get an unfiltered result set and assume the filter worked. Prefer strict validation (FastAPI does this by default with typed models; be careful with `**kwargs`-style catch-alls).
- **Encoding assumptions**: query parameter values must be URL-encoded (spaces become `%20` or `+`, special characters get escaped) — don't manually concatenate raw strings into a query string; always use your HTTP client's/framework's built-in encoding.

## 7. Self-Check Questions

1. Why should path parameters generally not be optional, while query parameters generally should be?
2. What real production risk does omitting `le=` bounds on a `limit` query parameter create?
3. Given `/reports/{report_id}/export?format=csv`, why is `format` a query parameter and not part of the path?

## 8. Connections

- **Pydantic data validation** (next topic): query and path parameters use the same type-hint-driven validation system as full request bodies — this topic is the simplest case of that broader mechanism.
- **Building scalable CRUD endpoints** (Module 2): pagination (`limit`/`offset`) query parameters are a core building block of every list endpoint.
- **URL-based vs Header-based API versioning** (Module 2): an alternative design decision about what belongs in the URL structure vs elsewhere, using the same "does this identify the resource or modify the request" reasoning.
