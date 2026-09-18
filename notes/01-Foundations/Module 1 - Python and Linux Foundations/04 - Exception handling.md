# Exception Handling

## What & Why

Every network call, tool execution, and LLM API request in this course can fail. Exception handling is how failures are contained, reported, and — where appropriate — retried, instead of crashing the whole request or agent run.

## How It Works

```python
try:
    result = call_external_api()
except TimeoutError as e:
    log.warning("timed out: %s", e)
    raise                                  # re-raise after logging
except ValueError as e:
    raise RuntimeError("bad response shape") from e   # exception chaining
else:
    process(result)                        # only runs if no exception
finally:
    cleanup()                              # always runs
```

- **Order matters**: `try` → (`except` clauses, most specific first) → `else` (only if no exception) → `finally` (always, even on an unhandled exception).
- **Custom exceptions**: define your own (`class ToolExecutionError(Exception): ...`) so callers can catch *your* failure modes specifically instead of catching bare `Exception`.
- **Exception chaining** (`raise X from Y`): preserves the original traceback context so debugging doesn't lose the root cause.

## Common Built-in Exceptions

| Exception | When it's raised |
|---|---|
| `ValueError` | Right type, invalid value |
| `TypeError` | Wrong type entirely |
| `KeyError` | Missing dict key |
| `TimeoutError` | Operation exceeded time limit |
| `ConnectionError` | Network-level failure |

## Pitfalls

- Catching bare `except:` (or `except Exception:` too broadly) — swallows real bugs (e.g., `KeyboardInterrupt`, programming errors) along with the expected failure.
- Retrying *every* exception type identically — a `ValueError` (bad input) retried the same way as a `TimeoutError` (transient) just wastes time and can mask a real bug.
- Not logging before re-raising — the exception propagates but the context of *where* it happened is lost.

## Connections

This is the direct foundation for Module 14's *Exponential backoff strategies* and Module 5's *Handling and retrying output parsing errors gracefully* — both are exception-handling patterns applied to specific failure types (transient network errors vs malformed LLM output).
