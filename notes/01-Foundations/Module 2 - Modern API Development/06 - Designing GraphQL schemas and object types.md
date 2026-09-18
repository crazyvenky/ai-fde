# Designing GraphQL Schemas and Object Types

## 1. What This Is & Why an FDE Needs It

GraphQL is an alternative API paradigm to REST: instead of many fixed-shape endpoints (`/orders`, `/orders/{id}/items`), a client sends a single query describing exactly which fields it wants, across potentially multiple related objects, and gets back exactly that shape — no more, no less. As an FDE, you'll encounter GraphQL less often than REST for new systems you build, but frequently when *integrating* with enterprise systems that already expose a GraphQL API (many modern SaaS platforms do), so being able to read and design against a schema fluently matters even if REST remains your default for new APIs.

## 2. Core Mechanics — Schema-First vs Code-First

- **Schema-first**: you write the GraphQL schema in its own definition language (SDL) first, then implement resolver functions that fulfill it.
- **Code-first** (what Strawberry and Graphene, the two libraries named in the curriculum, primarily do): you write Python classes/decorators, and the schema is generated *from* your code.

```python
import strawberry

@strawberry.type
class Address:
    city: str
    zip_code: str

@strawberry.type
class Customer:
    name: str
    address: Address

@strawberry.type
class Order:
    id: strawberry.ID
    total: float
    customer: Customer

@strawberry.type
class Query:
    @strawberry.field
    def order(self, id: strawberry.ID) -> Order:
        return fetch_order(id)

schema = strawberry.Schema(query=Query)
```

This generates a schema equivalent to:

```graphql
type Address { city: String!, zipCode: String! }
type Customer { name: String!, address: Address! }
type Order { id: ID!, total: Float!, customer: Customer! }
type Query { order(id: ID!): Order }
```

A client can now request exactly the fields it needs:

```graphql
query {
  order(id: "42") {
    total
    customer {
      name
      address { city }
    }
  }
}
```

— getting back `total`, `customer.name`, and `customer.address.city`, but *not* `zipCode` or `id`, because it didn't ask for them. This is GraphQL's headline feature: no over-fetching, no under-fetching.

## 3. Queries vs Mutations (Preview)

`Query` types are for reads (side-effect-free); `Mutation` types are for writes. This is covered in depth in *Executing queries vs mutations* — the schema design principle to internalize now is that **types are shared** between both: the same `Order` type returned by a query is also what a mutation like `createOrder(...)` returns after a write, keeping the client's data-shape expectations consistent regardless of how the data was obtained.

## 4. Comparisons — REST vs GraphQL

| Dimension | REST | GraphQL |
|---|---|---|
| Fetching related data | Multiple round-trips, or bespoke "expand" params | One request, nested query |
| Over/under-fetching | Common (fixed response shape) | Avoided by design (client specifies fields) |
| Caching | Simple — HTTP caching works per-URL out of the box | Harder — a single `POST /graphql` endpoint doesn't cache the same way |
| Versioning | New endpoint or version prefix | Add new fields/types without breaking old queries (deprecate old fields instead of versioning routes) |
| Learning curve for consumers | Lower — familiar HTTP semantics | Higher — need to learn the query language and schema |
| Best fit | Simple CRUD, public APIs, cache-friendly reads | Complex, deeply nested data with varied client needs (mobile vs web wanting different field subsets) |

Neither is strictly "better" — the honest FDE answer is: **REST for straightforward CRUD systems** (most of what this course builds — OmniGuard, AuditMesh), **GraphQL when a client genuinely has varied, nested data-fetching needs** across multiple consumer types (e.g., a mobile app wanting a lean payload and a web dashboard wanting a richer one, from the same underlying data).

## 5. Real-World Enterprise Scenario

A client's existing internal system already runs a GraphQL gateway (common at large enterprises with many backend microservices unified behind one graph). When integrating OmniGuard's data into their existing dashboard, you may need to expose an OmniGuard GraphQL type that this gateway can federate rather than force the client to bolt on a separate REST client just for your service. Recognizing this pattern — and knowing enough Strawberry/Graphene to expose a minimal, well-typed schema — is a real, recurring FDE integration scenario, not an academic exercise.

## 6. Common Pitfalls

- **Designing a GraphQL schema that mirrors your database tables 1:1** — defeats the purpose; the schema should model what *clients* need to ask for, which is often a different shape than storage.
- **Exposing a single giant `Query` type with dozens of unrelated top-level fields** and no clear organization — makes the schema hard to navigate; group related fields under sensible parent types.
- **Not thinking about nested-field cost upfront** — a deeply nested query can silently trigger many expensive underlying calls (this is exactly the N+1 problem, covered next), so schema design and resolver efficiency have to be considered together, not separately.
- **Treating GraphQL as a drop-in REST replacement everywhere** — adding GraphQL to a system that doesn't actually have varied client-fetching needs adds real complexity (a new query language, different caching story, different tooling) for no corresponding benefit.

## 7. Self-Check Questions

1. What specific problem does GraphQL's "client specifies exactly which fields it wants" model solve that REST's fixed-response-shape model doesn't?
2. Why might schema design and resolver efficiency need to be considered together rather than schema-first-then-optimize-later?
3. In what kind of client scenario would GraphQL's flexibility actually pay off compared to REST, versus a scenario where it would just add unnecessary complexity?

## 8. Connections

- **Writing efficient data resolvers** and **Handling N+1 query problems** (next two topics in this module): the direct performance consequences of how a schema like this actually gets *executed*, not just designed.
- **Executing queries vs mutations** (Module 2): the read/write split previewed above, covered in full detail next.
- **Building scalable CRUD endpoints** (Module 2): the REST counterpart this topic is explicitly being contrasted against — understanding both equips you to choose deliberately rather than defaulting to whichever one you learned first.
