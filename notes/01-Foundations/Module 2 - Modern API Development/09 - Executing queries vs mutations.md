# Executing Queries vs Mutations

## 1. What This Is & Why an FDE Needs It

GraphQL splits every operation into exactly one of two root types: `Query` (read, no side effects) and `Mutation` (write, has side effects). This mirrors the REST distinction between `GET` and `POST`/`PUT`/`DELETE` covered in *Building scalable CRUD endpoints*, but GraphQL makes the split an explicit, structural part of the schema rather than a convention layered onto HTTP methods.

## 2. Core Mechanics

```python
@strawberry.type
class Query:
    @strawberry.field
    def order(self, id: strawberry.ID) -> Order:
        return fetch_order(id)                    # read-only, no side effect

@strawberry.type
class Mutation:
    @strawberry.field
    def create_order(self, input: OrderInput) -> Order:
        return insert_order(input)                # side effect: writes to the DB

    @strawberry.field
    def cancel_order(self, id: strawberry.ID) -> Order:
        order = fetch_order(id)
        order.status = "cancelled"
        save_order(order)                          # side effect
        return order

schema = strawberry.Schema(query=Query, mutation=Mutation)
```

Client usage makes the distinction explicit at the call site:

```graphql
query {
  order(id: "42") { total }
}

mutation {
  createOrder(input: { customerId: "7", total: 99.5 }) {
    id
    total
  }
}
```

## 3. Execution Order Guarantee

This is a subtle but important spec-level detail: **queries execute their top-level fields in parallel** (since they have no side effects, order doesn't matter), but **mutations execute their top-level fields sequentially, in the order they appear in the request**. If a client sends a mutation document with two mutations back-to-back, GraphQL guarantees the first fully completes before the second starts — this matters when the second mutation logically depends on the effect of the first.

## 4. Designing Mutation Responses

A well-designed mutation returns **the resulting object**, not just a bare success flag — this lets the client immediately use the updated data without a follow-up query:

```graphql
type Mutation {
  createOrder(input: OrderInput!): CreateOrderPayload!
}

type CreateOrderPayload {
  order: Order          # the created object, so the client doesn't need a second round-trip
  errors: [UserError!]  # structured, field-level errors instead of a generic exception
}
```

This "payload object with both the result and structured errors" pattern is the GraphQL-community-standard way to handle partial failure — a mutation that *technically* succeeded at the transport level but failed a business rule (e.g., "insufficient inventory") returns a normal `200` response with `errors` populated, rather than throwing a GraphQL-level exception that's harder for clients to handle gracefully.

## 5. Comparisons / Design Notes

| Aspect | Query | Mutation |
|---|---|---|
| Side effects | None (by contract) | Yes |
| Execution order (top-level fields) | Parallel | Sequential |
| Idempotency | Naturally idempotent (reading twice is safe) | Not automatically idempotent — same idempotency concerns as REST `POST` |
| Typical response shape | The requested type directly | A payload object wrapping the result + errors |

## 6. Real-World Enterprise Scenario

If AuditMesh exposed a GraphQL mutation for "approve this compliance finding" (tying into the human-in-the-loop approval workflow from Module 10/16), the mutation's execution-order guarantee matters directly: a client submitting `approveFinding(id: "A")` followed by `approveFinding(id: "B")` in the same request can rely on A being fully processed — including any downstream side effects like triggering a Jira ticket update — before B begins, which is essential if B's approval logic needs to check A's already-updated state (e.g., a batch-approval rule that depends on sequential ordering).

## 7. Common Pitfalls

- **Putting a side effect inside a `Query` field** — e.g., a query resolver that "helpfully" logs an audit record or updates a "last viewed" timestamp as a side effect of a read. This breaks the fundamental read/write contract GraphQL relies on (and breaks the parallel-execution guarantee's safety, since queries are assumed side-effect-free and may be reordered or deduplicated by clients/caches).
- **Returning a bare boolean from a mutation** (`success: true`) instead of the resulting object — forces the client into an unnecessary follow-up query just to see the data they just changed.
- **Throwing a raw exception for expected business-rule failures** instead of returning structured `errors` in the payload — makes ordinary, expected failure modes (like "insufficient inventory") indistinguishable from genuine server errors on the client side.
- **Assuming mutations are idempotent by default** — they are not; retrying a `createOrder` mutation after a network timeout can create a duplicate order, exactly as with REST `POST` (see *Building scalable CRUD endpoints*, *Idempotency keys for safe tool execution*).

## 8. Self-Check Questions

1. Why does GraphQL guarantee sequential execution for top-level mutation fields but allow parallel execution for query fields?
2. Why does the "payload object with result + structured errors" pattern handle business-rule failures better than throwing an exception?
3. In the AuditMesh example, what would break if two chained `approveFinding` mutations were executed in parallel instead of sequentially?

## 9. Connections

- **Building scalable CRUD endpoints** (Module 2): the REST equivalent of this exact read/write, idempotency, and response-shape discussion.
- **Requesting manual state approval** (Module 10): the human-in-the-loop approval mutation scenario above previews this Module 10 concept directly.
- **Idempotency keys for safe tool execution** (Module 14): the direct fix for mutations' lack of built-in idempotency, same as for REST `POST`.
