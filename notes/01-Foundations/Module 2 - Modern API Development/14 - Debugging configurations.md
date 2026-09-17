# Debugging Configurations

## 1. What This Is & Why an FDE Needs It

Print-statement debugging (`print(x)`, deleting it, adding another) works for trivial scripts but breaks down fast on async FastAPI endpoints, multi-step agent graphs, or anything with real concurrency — you need to *pause* execution and inspect live state. A properly configured debugger is the difference between guessing at a bug for an hour and finding it in five minutes by actually watching the code run, variable by variable.

## 2. Core Mechanics — Breakpoints and Launch Configs

A **breakpoint** pauses execution at a specific line, letting you inspect every variable in scope, step line-by-line, and even evaluate arbitrary expressions against the live program state.

VSCode's `launch.json` (the file that defines *how* to start your program under the debugger) for a FastAPI app:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "FastAPI: Debug",
      "type": "debugpy",
      "request": "launch",
      "module": "uvicorn",
      "args": ["myapp.main:app", "--reload", "--port", "8000"],
      "jinja": true,
      "justMyCode": true
    }
  ]
}
```

- **`module: "uvicorn"`**: tells the debugger to launch your app through uvicorn (the actual production entry point), not just run a script directly — so breakpoints hit inside real request-handling code, not a separate test harness.
- **`justMyCode: true`**: steps through *your* code only, skipping into library internals (FastAPI's, Pydantic's) unless you explicitly need to. Turn this off only when you suspect the bug is genuinely inside a library, not your code calling it.
- **`"--reload"`**: auto-restarts the server on code changes — convenient for iteration, but be aware a reload mid-debug-session drops your current breakpoint state.

## 3. Debugging Async Code

Debugging `async def` endpoints has a real wrinkle: a breakpoint pausing one coroutine doesn't pause the rest of the event loop by default in most setups — other requests may keep processing while you're stepping through one. This is *usually* what you want (you're debugging one request's logic, not freezing the whole server), but it means you can't assume no other code ran between two breakpoints in the same debugging session, unlike in fully synchronous code.

## 4. Remote/Container Debugging

Since this course deploys everything in Docker (Module 4), you'll frequently need to debug code running *inside* a container, not on your local machine directly:

```json
{
  "name": "Python: Remote Attach",
  "type": "debugpy",
  "request": "attach",
  "connect": { "host": "localhost", "port": 5678 },
  "pathMappings": [
    { "localRoot": "${workspaceFolder}", "remoteRoot": "/app" }
  ]
}
```

`pathMappings` is the detail people most often get wrong: the debugger needs to translate between your local file paths and the paths *inside* the container, or breakpoints set locally simply won't line up with the code actually executing remotely.

## 5. Comparisons / Tools

| Approach | When to use |
|---|---|
| Interactive breakpoint debugger (VSCode, PyCharm) | Any non-trivial bug where you need to inspect live state — the default first choice |
| `print()`/logging | Quick, low-friction checks; fine for very simple cases, but doesn't scale to complex control flow |
| `pdb`/`ipdb` (command-line debugger) | When no IDE/GUI is available (e.g., debugging directly inside a remote server or container shell) |
| Structured tracing (Module 14) | Understanding behavior *in production*, where you can't attach an interactive debugger at all |

## 6. Real-World Enterprise Scenario

Debugging why AuditMesh's LangGraph supervisor occasionally routes a compliance case to the wrong sub-agent requires setting a breakpoint inside the routing function and inspecting the actual `state` object at that exact moment — print statements scattered through a multi-node graph produce an overwhelming, hard-to-correlate wall of output, whereas a breakpoint lets you pause at the exact decision point and inspect the full state as one coherent snapshot, then step forward to watch exactly how it changes node-by-node.

## 7. Common Pitfalls

- **Debugging directly against a production or shared-staging environment** — pausing a shared server's request thread on a breakpoint blocks that request (and potentially others) for every other user of that environment; always debug against a local or personal environment.
- **Forgetting `pathMappings` when debugging inside a container** — breakpoints appear to be set but never trigger, because the debugger is comparing mismatched local/remote paths.
- **Leaving `justMyCode: false`/verbose settings on by default** — makes routine debugging sessions noisy by stepping into framework internals you didn't actually need to inspect.
- **Relying purely on `print()` for a genuinely complex async/concurrent bug** — timing-dependent bugs are exactly where print-statement debugging is least reliable, since the act of adding print statements can itself change timing behavior enough to hide or shift the bug.

## 8. Self-Check Questions

1. Why doesn't pausing one async coroutine at a breakpoint necessarily pause the entire event loop, and why does that matter when debugging concurrent request handling?
2. Why is `pathMappings` specifically necessary for container-based remote debugging, and what breaks if it's misconfigured?
3. In the AuditMesh scenario, why is a breakpoint inspecting the full state object more useful than scattering print statements through the graph's nodes?

## 9. Connections

- **Event loop architecture / Coroutines and tasks** (Module 1): understanding what actually happens when you pause an async coroutine mid-execution depends directly on this earlier foundation.
- **Debugging multi-step agent reasoning and tool inputs** (Module 14): the production-environment counterpart to this topic — when you *can't* attach a live debugger, distributed tracing is the tool that fills the same investigative role.
- **Containerizing FastAPI backends** (Module 4): remote/container debugging setup here directly depends on how that Dockerfile exposes ports and runs the app.
