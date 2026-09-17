# Handling N+1 Query Problems

## 1. What This Is & Why an FDE Needs It

The N+1 problem is the concrete performance failure mode the previous topic set up: **1 query to fetch a list of N parent objects, then N additional queries — one per parent — to fetch each parent's related data.** It's called out by name in this curriculum because it's one of the most common real-world performance bugs in *any* system with nested/relational data, GraphQL or not, and every FDE will eventually debug a "why is this endpoint suddenly slow" ticket that turns out to be exactly this.

## 2. Diagnosing It

```graphql
query {
  customers {         # Query 1: SELECT * FROM customers  (returns 100 rows)
    name
    orders { total }  # Query 2..101: SELECT * FROM orders WHERE customer_id = ?
  }                    #              — run once per customer, 100 separate queries
}
```

101 total database round-trips for what should conceptually be "get customers and their orders" — a request that, done correctly, needs at most 2 queries regardless of how many customers there are. The telltale sign in practice: enable query logging in development, run a nested query returning a list, and count identical-shaped queries repeating with only the ID parameter changing — that repetition *is* the N+1 pattern, visible directly in the logs.

## 3. The Fix — Batching (DataLoader Pattern)

Instead of resolving `orders` independently for each customer as soon as its resolver is called, **collect all the customer IDs first, then issue one batched query** for all of them at once, and distribute the results back to the right resolver calls.

```python
from strawberry.dataloader import DataLoader

async def batch_load_orders(customer_ids: list[int]) -> list[list[Order]]:
    # ONE query for all customer_ids, not one per customer
    all_orders = fetch_orders_where_customer_in(customer_ids)
    # group results back by customer_id, preserving input order
    grouped = {cid: [] for cid in customer_ids}
    for order in all_orders:
        grouped[order.customer_id].append(order)
    return [grouped[cid] for cid in customer_ids]

orders_loader = DataLoader(load_fn=batch_load_orders)

@strawberry.type
class Customer:
    id: strawberry.ID
    name: str

    @strawberry.field
    async def orders(self) -> list[Order]:
        return await orders_loader.load(self.id)   # batched, not called immediately
```

The `DataLoader` doesn't run `batch_load_orders` the instant `.load(self.id)` is called — it **collects all the `.load()` calls made during the current tick of the event loop, then fires one batched call** with all the collected IDs together. This is why it's an inherently async-friendly pattern: it relies on the event loop's scheduling (Module 1, *Event loop architecture*) to gather requests before dispatching.

## 4. Comparisons — Fix Strategies

| Fix | How it works | When to use |
|---|---|---|
| DataLoader / batching | Collects per-object requests, issues one batched query | Standard fix for GraphQL nested resolvers |
| Eager loading (`JOIN` or ORM `.prefetch_related()`) | Fetch parent + related data in one query upfront | Works well when you know ahead of time which related fields will be needed |
| Denormalization | Store the related count/summary directly on the parent record | Read-heavy, write-light data where a small staleness window is acceptable |
| Caching | Cache repeated lookups (e.g., the same customer requested across multiple queries) | Reduces *repeated* N+1 across requests, doesn't fix it within a single request by itself |

## 5. Real-World Enterprise Scenario

Continuing the OmniGuard audit-dashboard example from the previous topic: fetching 100 audit runs and their `findings` naively costs 101 queries. Adding a `DataLoader` that batches "get all findings for these audit run IDs" into a single `WHERE audit_run_id IN (...)` query collapses this to 2 queries total — the same conceptual fix regardless of whether the API layer is GraphQL or a REST endpoint doing the equivalent nested-fetch-and-serialize pattern. This exact fix (batch-fetch instead of per-item-fetch) also underlies why Module 6/8's retrieval pipelines batch embedding calls across chunks instead of embedding one chunk at a time.

## 6. Common Pitfalls

- **Fixing N+1 by caching alone** — caching helps if the *same* parent is requested repeatedly across different requests, but does nothing for the first request that still has to make N+1 calls; the real fix is batching within the request, not just caching across requests.
- **Batching correctly but losing the per-parent grouping** — a batched query returns a flat list of all related records; forgetting to re-group them by parent ID before returning results to each resolver silently corrupts the response (customer A gets some of customer B's orders).
- **Not noticing N+1 in development because the dataset is tiny** — 5 customers × 1 query each is invisible in a demo; the same code pattern becomes a serious bottleneck at 10,000 customers. Always check actual query counts/logs during testing, not just response correctness.
- **Over-batching everything "just in case,"** adding DataLoader complexity to fields that are never actually requested together at scale — apply this fix where profiling/logging shows it's needed, not reflexively everywhere.

## 7. Self-Check Questions

1. Why does a naive nested resolver produce exactly `N + 1` queries rather than some other number, for `N` parent objects?
2. Why does a `DataLoader`'s batching behavior depend on the async event loop's scheduling rather than running synchronously the instant `.load()` is called?
3. In the OmniGuard scenario, why does batching all `findings` lookups into one `WHERE audit_run_id IN (...)` query scale so much better than one query per audit run?

## 8. Connections

- **Writing efficient data resolvers** (previous topic): this is the direct performance consequence of that topic's per-object resolver execution model — the two should be understood as one continuous idea.
- **Event loop architecture** (Module 1): the DataLoader's request-collection behavior relies directly on how the asyncio event loop schedules and batches work within a single tick.
- **API integration for rerankers** (Module 6): reranking APIs are typically called in a single batched request across all retrieved chunks rather than one call per chunk — the exact same anti-N+1 discipline applied to a different pipeline.
