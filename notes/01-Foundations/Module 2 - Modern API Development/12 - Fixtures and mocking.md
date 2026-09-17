# Fixtures and Mocking

## 1. What This Is & Why an FDE Needs It

Unit tests need to be fast and isolated (previous topic), but real code depends on real things: databases, LLM provider APIs, Slack/Jira APIs, the current time. Fixtures and mocking are the two complementary tools that let a test exercise real logic without actually touching those real, slow, non-deterministic, or costly external dependencies — which matters enormously in this course, since a naive test suite for an LLM-calling agent would otherwise burn real API costs and take real network round-trips on every test run.

## 2. Fixtures — Reusable Test Setup

A **fixture** is a pytest mechanism for providing a piece of test setup (an object, a value, a resource) to any test that asks for it, with automatic cleanup afterward.

```python
import pytest

@pytest.fixture
def sample_order():
    return Order(subtotal=100.0, tax_rate=0.08)

@pytest.fixture
def db_session():
    session = create_test_db_session()
    yield session              # test runs here
    session.rollback()         # cleanup — runs after the test, even if it failed
    session.close()

def test_order_total(sample_order):          # fixture injected by name
    assert sample_order.calculate_total() == 108.0

def test_insert_order(db_session, sample_order):  # multiple fixtures, composed freely
    db_session.add(sample_order)
    db_session.commit()
    assert db_session.query(Order).count() == 1
```

Fixtures compose naturally — a test can request several, and fixtures can request other fixtures — and pytest resolves the whole dependency graph automatically, conceptually identical to FastAPI's `Depends()` mechanism from earlier in this module, just for test setup instead of request handling.

## 3. Fixture Scope

```python
@pytest.fixture(scope="function")   # default: fresh instance per test (safest)
def db_session(): ...

@pytest.fixture(scope="module")     # shared across all tests in one file
def expensive_setup(): ...

@pytest.fixture(scope="session")    # shared across the entire test run
def app_config(): ...
```

Wider scopes (`module`, `session`) reduce setup overhead for genuinely expensive, safely-shareable resources (e.g., a config object that's never mutated), but **the default (`function`) is the safe choice** — sharing mutable state (like a DB session) across tests at a wider scope risks one test's leftover state silently affecting another test's result, exactly the "tests depending on execution order" pitfall flagged in the previous topic.

## 4. Mocking — Replacing Real Dependencies

Where a fixture provides test *setup*, **mocking** replaces a specific function/object's *behavior* with a controlled stand-in, so a test can simulate scenarios (a slow API, a failing API, a specific returned value) without the real dependency existing at all.

```python
from unittest.mock import Mock, patch

def test_agent_retries_on_timeout():
    mock_llm_client = Mock()
    mock_llm_client.complete.side_effect = [TimeoutError(), "success response"]

    result = call_llm_with_retry(mock_llm_client, prompt="hello")

    assert result == "success response"
    assert mock_llm_client.complete.call_count == 2   # verify it actually retried

@patch("myapp.agent.external_api_call")   # patches the reference used inside myapp.agent
def test_tool_execution(mock_api_call):
    mock_api_call.return_value = {"status": "ok"}
    result = run_tool("check_status")
    assert result["status"] == "ok"
    mock_api_call.assert_called_once_with(expected_args)
```

- **`Mock`**: a flexible stand-in object — calling any method on it just records the call and returns another `Mock` (or whatever you configure via `.return_value`/`.side_effect`).
- **`monkeypatch`** (pytest's built-in alternative to `unittest.mock.patch`): temporarily replaces an attribute/function for the duration of a test, then automatically restores the original afterward — often preferred within pytest-native test suites for its automatic cleanup and simpler syntax for common cases (env vars, module attributes).
- **`side_effect`**: lets a mock return *different* values (or raise exceptions) on successive calls — exactly what's needed to simulate "the first call times out, the retry succeeds."

## 5. Comparisons — Mock vs Monkeypatch vs Real Test Double

| Tool | Best for |
|---|---|
| `unittest.mock.Mock`/`MagicMock` | Flexible, general-purpose stand-ins with call verification |
| `unittest.mock.patch` | Replacing a specific function/class reference for a test's duration |
| pytest `monkeypatch` | Simpler cases, especially env vars and straightforward attribute swaps, with automatic teardown |
| A real in-memory fake (e.g., an in-memory SQLite DB instead of mocking the ORM) | When you want to test real logic (queries, constraints) without a full external dependency |

## 6. Real-World Enterprise Scenario

Testing OmniGuard's retry-on-parse-failure logic (Module 5) without mocking would mean actually calling a real LLM provider, crafting a prompt that reliably produces malformed JSON, and hoping it fails in exactly the way you want to test — slow, costly, and non-deterministic. Mocking the LLM client to return a malformed response on the first call and a valid one on the second lets you write a fast, deterministic, free test that proves the retry logic works correctly, every single time it runs, regardless of what a real LLM provider happens to do on any given day.

## 7. Common Pitfalls

- **Over-mocking** — replacing so much of the system under test with mocks that the test ends up just verifying the mocks were called correctly, rather than verifying any real logic. If a test would pass even with the actual business logic completely broken, it's testing the mocks, not the code.
- **Mocking at the wrong boundary** — patching a function deep inside a third-party library instead of your own code's interface to that library couples the test to the library's internal implementation, breaking on library upgrades unrelated to your actual logic.
- **Forgetting `side_effect` vs `return_value`** — `return_value` gives the same result every call; `side_effect` (a list or exception) is needed to simulate a sequence of different behaviors across calls, like the retry scenario above.
- **Sharing mutable fixture state at too wide a scope** — a `session`-scoped fixture holding mutable state that one test modifies can silently corrupt the assumptions of every subsequent test that shares it.

## 8. Self-Check Questions

1. Why is the default fixture scope (`function`) the "safe" choice, and what specifically can go wrong with a wider scope holding mutable state?
2. Why does simulating "first call fails, retry succeeds" require `side_effect` rather than `return_value`?
3. In the OmniGuard scenario, why is mocking the LLM client actually a *better* test than calling a real LLM provider and hoping for a malformed response?

## 9. Connections

- **Dependency injection** (Module 2): `dependency_overrides` in FastAPI is the framework-level version of exactly this same "swap in a controlled test double" pattern.
- **Handling and retrying output parsing errors gracefully** (Module 5): the retry-logic scenario mocked above is a direct preview of a real Module 5 topic.
- **Automating LLM-as-a-judge scoring pipelines** (Module 14): production LLM evaluation faces the same "don't want to call a real, costly, non-deterministic API for every check" tension, solved with different tools at a different layer (offline eval datasets rather than mocks).
