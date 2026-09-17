# Building Scalable CRUD Endpoints

## 1. What This Is & Why an FDE Needs It

CRUD (Create, Read, Update, Delete) endpoints are the unglamorous backbone of nearly every enterprise system — even the most sophisticated agentic AI system eventually needs a "list all past audit runs," "get this specific record," or "update this configuration" endpoint. Everything covered so far in this module (path/query params, Pydantic, dependency injection, CORS) comes together here into complete, production-shaped endpoints.

"Scalable" here doesn't (only) mean "handles more traffic" — it means the endpoint design doesn't fall apart as data grows, as concurrent clients increase, or as the underlying resource's shape evolves. A CRUD endpoint that works fine with 100 test records but times out or returns inconsistent results at 10 million records is not scalable, regardless of how clean the code looks.

## 2. Core Mechanics — REST Conventions

| Operation | HTTP Method | URL Pattern | Success Status |
|---|---|---|---|
| Create | `POST` | `/orders` | `201 Created` |
| Read (one) | `GET` | `/orders/{id}` | `200 OK` |
| Read (list) | `GET` | `/orders` | `200 OK` |
| Update (full replace) | `PUT` | `/orders/{id}` | `200 OK` |
| Update (partial) | `PATCH` | `/orders/{id}` | `200 OK` |
| Delete | `DELETE` | `/orders/{id}` | `204 No Content` |

```python
from fastapi import APIRouter, HTTPException, status

router = APIRouter(prefix="/orders")

@router.post("", status_code=status.HTTP_201_CREATED)
def create_order(order: OrderCreate) -> OrderRead:
    new_order = db_insert(order)
    return new_order

@router.get("/{order_id}")
def get_order(order_id: int) -> OrderRead:
    order = db_get(order_id)
    if order is None:
        raise HTTPException(status_code=404, detail="Order not found")
    return order

@router.get("")
def list_orders(limit: int = 20, offset: int = 0) -> list[OrderRead]:
    return db_list(limit=limit, offset=offset)

@router.delete("/{order_id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_order(order_id: int):
    if not db_delete(order_id):
        raise HTTPException(status_code=404, detail="Order not found")
```

Note the **separate read vs write models** (`OrderCreate` vs `OrderRead`) — this is a deliberate design choice, not boilerplate duplication: the create model shouldn't accept a client-supplied `id` or `created_at` (server-controlled fields), while the read model needs to include them.

## 3. Pagination — The Core Scalability Mechanism

A `list_orders` endpoint with no pagination is the single most common scalability bug in CRUD design: it works perfectly in every test and demo, then falls over the moment the table has real production volume.

```python
@router.get("")
def list_orders(
    limit: int = Query(20, ge=1, le=100),   # hard ceiling — never trust an unbounded client request
    offset: int = Query(0, ge=0),
) -> list[OrderRead]:
    return db_list(limit=limit, offset=offset)
```

**Offset pagination** (`limit`/`offset`) is simple but has a real weakness at scale: retrieving page 10,000 still requires the database to scan and discard the first 999,900 rows — this gets slower as `offset` grows. **Cursor-based pagination** (returning an opaque "next" token pointing at the last seen record) avoids this by letting the database jump directly to the right position using an index, and is the standard for genuinely large, high-traffic collections. Know both; use offset pagination for simplicity when scale doesn't demand more, and cursor pagination once it does.

## 4. Idempotency

An operation is **idempotent** if performing it multiple times has the same effect as performing it once. This matters enormously for CRUD reliability: network failures mean clients sometimes retry a request without knowing if the first attempt actually succeeded.

- `GET`, `PUT`, `DELETE` are *supposed* to be idempotent by HTTP convention — retrying a `DELETE` on an already-deleted resource should not error in a way that breaks a naive retry loop (returning 404 on the retry is generally fine and expected; the *state* of the world is the same either way).
- `POST` (create) is **not** idempotent by default — retrying it naively creates duplicate resources. This is exactly the problem **idempotency keys** (Module 14) solve: a client-supplied unique key lets the server recognize "this exact create request already happened" and return the original result instead of creating a duplicate.

## 5. Comparisons / Design Notes

| Design choice | Tradeoff |
|---|---|
| Offset pagination | Simple, supports "jump to page N," slower at very large offsets |
| Cursor pagination | Fast at any scale, no "jump to page N," slightly more complex client logic |
| Single model for read/write | Less code, but leaks server-controlled fields into what clients can submit |
| Separate `Create`/`Read`/`Update` models | More boilerplate, but a correct, explicit contract for what each operation actually accepts/returns |

## 6. Real-World Enterprise Scenario

AuditMesh (Module 16) exposes an endpoint listing all compliance audit runs for a dashboard. Early in the project, with a handful of test runs, `GET /audits?limit=100&offset=0` works instantly. Once the system has been live for months and accumulated hundreds of thousands of audit records, the same unbounded-`limit` request (if a client omits `limit` and the server has no hard ceiling) can attempt to serialize and return the entire table — this is exactly the kind of "worked in the demo, broke in production" incident that a `le=100` ceiling on `limit` (established back in *Path and query parameters*) exists to prevent categorically, not just mitigate after the fact.

## 7. Common Pitfalls

- **No upper bound on `limit`** — already covered, but worth repeating: this is the #1 CRUD scalability bug.
- **Using the same Pydantic model for create and read**, letting a client submit fields like `id` or `created_at` that should be server-controlled — always separate `Create`/`Read` (and often `Update`) schemas.
- **Returning `200 OK` for a delete instead of `204 No Content`** — minor, but breaks strict REST-client expectations and API-contract tooling that checks status codes.
- **Not indexing the columns used for filtering/sorting/pagination** — a `list_orders` endpoint that filters by `status` and sorts by `created_at` needs a database index on those columns, or "scalable" application code still produces full table scans underneath.
- **Treating `PUT` (full replace) and `PATCH` (partial update) as interchangeable** — a `PUT` that only receives 2 of 5 fields should either require all fields or explicitly null out the missing ones (full replace semantics); using `PUT` when you mean `PATCH` causes silent data loss on fields the client didn't include.

## 8. Self-Check Questions

1. Why does offset-based pagination get slower as the offset grows, and what alternative avoids this?
2. Why is a `POST` endpoint not naturally idempotent, and what mechanism (covered later in Module 14) fixes that?
3. In the AuditMesh scenario, why does the `limit` ceiling need to be enforced server-side even if the dashboard UI always sends a reasonable value?

## 9. Connections

- **Path and query parameters** (Module 2): pagination and filtering here are a direct, concrete application of that topic's query-parameter design.
- **Idempotency keys for safe tool execution** (Module 14): the full solution to the `POST`-is-not-idempotent problem raised here.
- **Vector Search and Core RAG — End-to-end basic retrieval** (Module 6): retrieval pipelines face the exact same "don't return unbounded results" scalability concern, just with `top_k` instead of `limit`.
