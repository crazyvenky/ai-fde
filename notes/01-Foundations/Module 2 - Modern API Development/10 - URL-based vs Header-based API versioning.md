# URL-Based vs Header-Based API Versioning

## 1. What This Is & Why an FDE Needs It

Every API that survives long enough eventually needs to change in a breaking way — a field renamed, a response shape restructured, a field removed. Versioning is how you ship that breaking change without instantly breaking every existing client. For an FDE, this is not academic: client integrations (Slack bots, internal dashboards, other services calling OmniGuard/AuditMesh) are exactly the kind of external consumer you cannot force to update on your schedule, so a deliberate versioning strategy — decided *before* the first breaking change is needed — is part of a responsible initial API design.

## 2. URL-Based Versioning

```
GET /v1/orders/42
GET /v2/orders/42
```

The version is baked directly into the path. Simple, highly visible, and trivially routable — a load balancer or API gateway can route `/v1/*` and `/v2/*` to entirely different backend deployments without any special header-parsing logic.

```python
from fastapi import APIRouter

v1_router = APIRouter(prefix="/v1")
v2_router = APIRouter(prefix="/v2")

@v1_router.get("/orders/{order_id}")
def get_order_v1(order_id: int) -> OrderReadV1:
    ...

@v2_router.get("/orders/{order_id}")
def get_order_v2(order_id: int) -> OrderReadV2:   # new response shape
    ...
```

## 3. Header-Based Versioning

```
GET /orders/42
Accept: application/vnd.myapi.v2+json
```

or a custom header:

```
GET /orders/42
API-Version: 2
```

The URL stays stable — clients always call the *same* resource path — and version negotiation happens through headers instead. This keeps URLs "clean" (a resource has exactly one canonical address regardless of version) and is closer to strict REST/HATEOAS philosophy, at the cost of being less visible: you can't tell which version a request is targeting just by glancing at a URL or browser address bar, and it requires every client, proxy, and piece of infrastructure in the request path to correctly read and preserve the header.

## 4. Comparisons / Design Notes

| Dimension | URL-based | Header-based |
|---|---|---|
| Visibility / discoverability | High — visible in the URL, logs, browser | Low — requires inspecting headers |
| Caching | Simple — different URLs cache independently | Harder — same URL needs cache keys that account for the header (`Vary` header) |
| Routing at infrastructure layer (LB, gateway) | Trivial — path-based routing rules | Requires header-aware routing, more infra complexity |
| REST purity | Arguably "wrong" — a resource's identity shouldn't include its version | Arguably "correct" — same resource, different representation |
| Client tooling ergonomics | Very easy — just change the URL | Slightly more setup — must configure headers on every request |

In practice, **URL-based versioning is far more common** in real enterprise APIs, precisely because of the infrastructure-routing and cache-simplicity advantages — most engineering teams find the "REST purity" argument for header-based versioning less valuable in practice than the operational simplicity URL-based versioning provides.

## 5. Migration Strategy for Breaking Changes

Regardless of which mechanism you pick, the actual migration discipline matters more than the mechanism itself:

1. **Add the new version alongside the old one** — never remove `v1` the same day `v2` ships.
2. **Document exactly what changed** and communicate a deprecation timeline to known consumers.
3. **Instrument usage** — log which version each request uses so you know, with data (not guesses), when it's safe to actually retire the old version.
4. **Prefer additive, non-breaking changes when possible** to avoid needing a new version at all — adding a new optional field to a response is not a breaking change; removing or renaming an existing field is.

## 6. Real-World Enterprise Scenario

If OmniGuard's Text-to-SQL response schema needs to change (say, adding structured confidence scores per query plan, restructuring the shape of `filters`), and a client's Slack bot integration already parses the current response shape, shipping that change as a breaking `v1 → v2` transition (URL-based, for the routing simplicity reasons above) with both versions live for an agreed migration window is the correct FDE move — silently changing `v1`'s response shape in place would break the client's bot at a time you don't control or even know about until they file an incident.

## 7. Common Pitfalls

- **Introducing a breaking change without bumping the version at all** — "it's just a small tweak" is exactly how silent breaking changes happen; any change that removes/renames/restructures an existing field is breaking, no matter how small it feels.
- **Bumping the version for every single change**, including fully backward-compatible additions — unnecessarily fragments client integrations and support burden; reserve version bumps for genuinely breaking changes.
- **Never deprecating old versions** — keeping every version alive forever multiplies maintenance burden indefinitely; a clear deprecation policy (with a real removal date, communicated in advance) is part of a healthy versioning strategy, not an afterthought.
- **Choosing header-based versioning without accounting for caching infrastructure (CDNs, proxies)** that may not vary its cache key on the version header, silently serving one client's cached v1 response to another client requesting v2.

## 8. Self-Check Questions

1. Why is URL-based versioning generally simpler for load balancer / API gateway routing than header-based versioning?
2. What makes a schema change "breaking" versus "safe to ship without a version bump"?
3. In the OmniGuard scenario, why does silently changing the `v1` response shape in place (instead of shipping a `v2`) create risk the client can't see coming?

## 9. Connections

- **Path and query parameters** (Module 2): the same "what belongs in the URL structure" reasoning applies here — a version prefix is a structural part of the URL, similar in spirit to a path parameter identifying a resource.
- **Constructing Hybrid RAG alongside secure Text-to-SQL** (Module 15): the OmniGuard Text-to-SQL scenario above is a direct preview of a real versioning decision that project's API will eventually face.
- **Managing GitHub secrets / Continuous deployment to AWS** (Module 4): running two API versions simultaneously has direct deployment implications — both versions need to be deployed, monitored, and eventually retired through the same CI/CD pipeline.
