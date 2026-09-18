# Writing Efficient Data Resolvers

## 1. What This Is & Why an FDE Needs It

A **resolver** is the function that actually fetches the data for one field in a GraphQL type. Every field in every type from the previous topic — `Order.total`, `Customer.address`, `Address.city` — has a resolver behind it, even if most of them are trivial (just returning an attribute). Understanding how and *when* resolvers execute is the difference between a GraphQL API that scales and one that silently issues hundreds of redundant database calls per request — which is precisely the setup for the next topic, the N+1 problem.

## 2. Core Mechanics — Resolver Execution Order

GraphQL resolves a query **field by field, top-down**, and each field's resolver only runs once GraphQL knows it needs that field's value.

```python
@strawberry.type
class Order:
    id: strawberry.ID

    @strawberry.field
    def customer(self) -> "Customer":
        print(f"Resolving customer for order {self.id}")
        return fetch_customer_for_order(self.id)   # a DB call

@strawberry.type
class Customer:
    name: str

    @strawberry.field
    def orders(self) -> list[Order]:
        print(f"Resolving orders for customer {self.name}")
        return fetch_orders_for_customer(self.name)  # another DB call
```

For a query like:

```graphql
query {
  order(id: "42") {
    id
    customer { name }
  }
}
```

Execution order is: resolve `order(id: "42")` → resolve `id` (trivial) → resolve `customer` (calls `fetch_customer_for_order`) → resolve `name` on that customer (trivial attribute access). **If the query hadn't asked for `customer`, `fetch_customer_for_order` would never run at all** — this lazy, field-driven execution is the direct mechanism behind GraphQL's "no over-fetching" property from the previous topic; it's not magic, it's just that unrequested fields' resolvers simply never get called.

## 3. The Performance Trap Hiding in This Design

The same laziness that avoids over-fetching a *single* object becomes a serious problem across a **list** of objects — which is the entire subject of the next topic. The core insight to carry forward: a resolver on a nested field runs *once per parent object* in the result set, by default, with no batching unless you explicitly add it.

```graphql
query {
  customers {          # returns, say, 100 customers
    name
    orders { total }   # this resolver runs 100 times — once per customer
  }
}
```

## 4. Writing Resolvers Well

A few concrete practices that separate an efficient resolver from a naive one:

- **Push filtering/pagination down to the resolver's arguments**, not into application code after fetching everything: `orders(status: "shipped", limit: 10)` as a resolver argument lets the underlying query do the filtering, rather than fetching all orders and filtering in Python.
- **Keep resolvers thin** — a resolver should call into a service/repository layer (the same DB-access code your REST endpoints would use), not contain raw SQL or business logic inline. This keeps the same data-access logic reusable and testable regardless of whether it's called from GraphQL or REST.
- **Be deliberate about what triggers a database call vs what's already in memory** — a resolver on a field that's just an attribute of an already-fetched object (`Order.id`) should be a trivial attribute lookup, not an accidental extra query.

## 5. Comparisons / Design Notes

| Resolver style | Behavior |
|---|---|
| Naive (independent query per field, per object) | Simple to write, but causes N+1 at scale |
| Batched (DataLoader-style, covered next topic) | More setup, but collapses per-object queries into one batched query |
| Denormalized/precomputed field | Fastest at read time, but requires keeping the precomputed value in sync on writes |

## 6. Real-World Enterprise Scenario

If OmniGuard (or a client integration built on top of it) exposed a GraphQL layer over its audit records, a naive `AuditRun.findings` resolver that issues one query per audit run would perform fine in a demo with 5 audit runs and become a serious bottleneck the moment a compliance dashboard requests `findings` across a page of 100 audit runs — 100 separate round-trips to the database instead of 1 or 2. This exact scenario is why the next topic (N+1) is taught immediately after this one, not as a separate, unrelated concern.

## 7. Common Pitfalls

- **Assuming a resolver is called once per query** — it's called once per *object instance* it's a field on, which for list fields means once per item in the list.
- **Putting expensive computation directly in a resolver with no caching**, when the same expensive value is likely to be requested repeatedly across a single request (or across requests) — consider request-scoped caching or precomputation.
- **Ignoring resolver arguments in favor of always fetching everything and filtering afterward in Python** — pushes unnecessary data transfer and computation onto the API layer instead of letting the data layer (which is usually far more efficient at filtering) do the work.
- **Writing resolver logic directly against the database (raw SQL/ORM calls) instead of a shared service layer** — duplicates logic that a REST endpoint elsewhere in the same codebase already implements correctly.

## 8. Self-Check Questions

1. Why does a resolver for a field that's never requested by the client simply never execute, and why is this the underlying mechanism for GraphQL's "no over-fetching" property?
2. Given a query returning 100 customers, each with a nested `orders` field, how many times does the `orders` resolver run by default, and why does that number matter for performance?
3. Why should a resolver call into a shared service/repository layer instead of containing raw database queries directly?

## 9. Connections

- **Handling N+1 query problems** (next topic): the direct, unavoidable consequence of per-object resolver execution described here — read that topic immediately after this one, they're two halves of the same idea.
- **Building scalable CRUD endpoints** (Module 2): the "push filtering to the query, not application code" principle here is identical to the pagination/filtering discipline taught for REST endpoints.
- **End-to-end basic retrieval** (Module 6): the same "avoid N redundant calls" discipline reappears when batching embedding lookups or reranking calls across many retrieved chunks.
