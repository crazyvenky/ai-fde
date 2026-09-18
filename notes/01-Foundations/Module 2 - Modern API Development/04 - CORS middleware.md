# CORS Middleware

## 1. What This Is & Why an FDE Needs It

CORS (Cross-Origin Resource Sharing) is a **browser-enforced** security mechanism — it has nothing to do with server-to-server calls, curl, Postman, or backend agents calling APIs. It only matters when a web page loaded from one origin (e.g., `https://client-dashboard.com`) tries to make a JavaScript `fetch`/`XHR` call to a different origin (e.g., `https://api.omniguard.internal`).

This is a topic every FDE hits in practice within the first week of integrating a demo UI with a backend: "it works in Postman but fails in the browser with a CORS error" is one of the most common early-integration support tickets, and understanding *why* Postman works and the browser doesn't (Postman isn't a browser and doesn't enforce CORS) resolves half the confusion immediately.

## 2. Core Mechanics

**Same-origin** means identical scheme + host + port. `https://app.com` and `https://api.app.com` are **different origins** (different host) even though they look related. `https://app.com` and `http://app.com` are also different origins (different scheme).

When a browser page on origin A tries to call an API on origin B:

1. For "simple" requests (GET, certain POST with simple content types), the browser sends the request directly but includes an `Origin` header. The server's response must include an `Access-Control-Allow-Origin` header matching (or `*`) or the browser blocks the *response* from reaching the calling JavaScript (the request still happened; the browser just hides the result).
2. For "non-simple" requests (custom headers like `Authorization`, methods like `PUT`/`DELETE`, JSON content type in some configurations), the browser first sends a **preflight** `OPTIONS` request asking "would you allow this?" — only if the server responds with the right `Access-Control-Allow-*` headers does the browser send the real request.

```
Browser (origin: https://dashboard.client.com)
   │
   ├── OPTIONS /api/orders   (preflight — "can I POST here with an Authorization header?")
   │       Access-Control-Request-Method: POST
   │       Access-Control-Request-Headers: authorization, content-type
   │
   ◄── 200 OK
   │       Access-Control-Allow-Origin: https://dashboard.client.com
   │       Access-Control-Allow-Methods: POST, GET
   │       Access-Control-Allow-Headers: authorization, content-type
   │
   ├── POST /api/orders   (the real request, now allowed to proceed)
```

## 3. Configuring It in FastAPI

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://dashboard.client.com"],   # exact origins, NOT "*" in production
    allow_credentials=True,                             # required if using cookies/auth headers
    allow_methods=["GET", "POST", "PUT", "DELETE"],
    allow_headers=["Authorization", "Content-Type"],
)
```

- `allow_origins`: an **exact allow-list** in production. Using `["*"]` means *any* website in the world can call your API from a user's browser — fine for a fully public, unauthenticated API, dangerous for anything behind auth.
- `allow_credentials=True`: required if the client sends cookies or `Authorization` headers cross-origin — but note that per the CORS spec, you **cannot** combine `allow_credentials=True` with `allow_origins=["*"]`; browsers reject that combination outright, forcing you to specify exact origins once credentials are involved. This is a helpful guardrail, not just an annoying restriction.

## 4. Comparisons / Design Notes

| Misconception | Reality |
|---|---|
| "CORS prevents my server from being called by unauthorized clients" | No — CORS is enforced by the *browser*, not the server. A server, script, or curl call bypasses it entirely. Real access control is auth (Module 12), not CORS. |
| "I'll just set `allow_origins=["*"]` everywhere to stop the errors" | Fine for a public read-only API; a real security gap for anything with cookies/auth in a browser context. |
| "CORS errors mean my API is broken" | Usually means the API is working correctly and *rejecting* an unapproved cross-origin browser request — the fix is almost always in the CORS config, not the endpoint logic. |

## 5. Real-World Enterprise Scenario

A client wants to embed the OmniGuard chat interface inside their existing internal portal (`https://portal.acmecorp.com`), while the OmniGuard API itself is hosted separately (`https://api.omniguard-acmecorp.com`). Without correct CORS configuration, the embedded chat widget's JavaScript can make the network request (it appears in the Network tab), but the browser silently withholds the response from the widget's code — producing a confusing "nothing happens when I click send" bug report rather than an obvious error, unless you know to check the browser console for the CORS rejection message.

The fix is exactly the allow-list pattern above, scoped precisely to `https://portal.acmecorp.com` — not a wildcard, since the OmniGuard API also handles authenticated, credentialed requests.

## 6. Common Pitfalls

- **Confusing CORS with authentication/authorization** — a wide-open CORS policy doesn't mean anyone can *use* your API meaningfully if it still requires a valid auth token; conversely, tight CORS doesn't substitute for real auth. They solve different problems and both are needed.
- **Debugging CORS by looking at server logs instead of the browser console** — the server usually returns something (even a legitimate 200 for the preflight); the actual rejection happens client-side, so the browser's console/network tab is the correct place to diagnose it, not the API's logs.
- **Setting `allow_origins=["*"]` with `allow_credentials=True`** — the browser spec forbids this combination, so it silently fails in ways that look like a config bug rather than the security restriction it actually is.
- **Forgetting the preflight `OPTIONS` request needs to succeed too** — a custom auth middleware that rejects unauthenticated requests can accidentally reject the preflight `OPTIONS` call itself (which carries no auth token), breaking every non-simple cross-origin request even though the "real" request handler is correctly configured.

## 7. Self-Check Questions

1. Why does a request that works fine in Postman fail with a CORS error only in the browser?
2. Why can't you combine `allow_origins=["*"]` with `allow_credentials=True`, and why is that restriction actually helpful?
3. If a client reports "clicking submit does nothing" on an embedded widget, what's the first place you'd check before touching any server code?

## 8. Connections

- **Building scalable CRUD endpoints** (next topic): CORS configuration sits at the middleware layer, wrapping every CRUD endpoint the API exposes — it's a one-time setup that then applies uniformly.
- **Authentication vs authorization** (Module 12): the natural follow-up question after "is CORS configured correctly" is always "is the request actually authenticated/authorized" — a completely separate concern this topic deliberately does not solve.
- **Webhooks and bot tokens for Slack/Teams** (Module 11): server-to-server integrations like Slack webhooks never trigger CORS at all, since CORS is browser-specific — a useful contrast that reinforces where this mechanism does and doesn't apply.
