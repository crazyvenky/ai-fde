# Unit Testing Fundamentals

## 1. What This Is & Why an FDE Needs It

Everything built in this course — guardrails, RAG pipelines, agent orchestration, secure Text-to-SQL — is exactly the kind of code where a subtle bug can silently produce a wrong answer, leak data across a permission boundary, or corrupt state, without ever raising an obvious error. Unit tests are how you get evidence that a specific piece of logic behaves correctly, on demand, before a client ever sees it — and, critically, evidence that stays valid every time you change the code afterward, catching regressions automatically instead of relying on manual re-checking.

For an FDE specifically, tests are also part of the *deliverable*, not just a personal safety net: UAT runbooks (Module 15) and client sign-off processes assume the system has been verified, and a test suite is concrete evidence of that verification.

## 2. Core Mechanics — Arrange, Act, Assert

Every good unit test follows the same three-part shape, regardless of language or framework:

```python
def test_order_total_includes_tax():
    # Arrange — set up the exact scenario being tested
    order = Order(subtotal=100.0, tax_rate=0.08)

    # Act — perform the one operation under test
    total = order.calculate_total()

    # Assert — check the result matches expectations
    assert total == 108.0
```

- **Arrange**: construct the specific input/state needed for this scenario, and *only* this scenario — no unrelated setup.
- **Act**: call the one function/method being tested. If a test needs multiple "acts," it's often actually testing multiple things and should be split into multiple tests.
- **Assert**: check the actual outcome against the expected one. A test with no assertion doesn't test anything, no matter how much code runs inside it.

## 3. Using pytest — A Minimal Test File

```python
# test_orders.py
import pytest
from myapp.orders import Order, InvalidOrderError

def test_order_total_includes_tax():
    order = Order(subtotal=100.0, tax_rate=0.08)
    assert order.calculate_total() == 108.0

def test_order_rejects_negative_subtotal():
    with pytest.raises(InvalidOrderError):
        Order(subtotal=-10.0, tax_rate=0.08)

@pytest.mark.parametrize("subtotal,tax_rate,expected", [
    (100.0, 0.0, 100.0),
    (100.0, 0.10, 110.0),
    (0.0, 0.08, 0.0),
])
def test_order_total_various_rates(subtotal, tax_rate, expected):
    order = Order(subtotal=subtotal, tax_rate=tax_rate)
    assert order.calculate_total() == expected
```

- pytest discovers any file matching `test_*.py` or `*_test.py`, and any function prefixed `test_` inside it — no manual test registration needed.
- `pytest.raises(...)` is how you assert that an exception *should* occur — testing failure paths is just as important as testing success paths, arguably more so for the security-sensitive systems this course builds.
- `@pytest.mark.parametrize` runs the same test logic against multiple input/output pairs without duplicating the test body — one of the highest-leverage pytest features for covering edge cases efficiently.

## 4. What Makes a Good Test Suite (Not Just "Some Tests")

- **Test behavior, not implementation** — a test should verify *what* the code does (given this input, produce that output), not *how* it does it internally. Tests that assert on private implementation details break every time you refactor, even when behavior is unchanged.
- **One logical assertion focus per test** — a test named `test_order_total_includes_tax` should fail for exactly one reason: tax calculation is wrong. A test that checks five unrelated things makes failures ambiguous to diagnose.
- **Fast and isolated** — unit tests should not hit a real database, real network, or real filesystem; that's what integration tests and fixtures/mocking (next topic) are for. A slow test suite gets skipped; a flaky one gets ignored — both defeat the entire purpose.
- **Deterministic** — the same test run twice should give the same result. Tests depending on current time, random values, or external service state without controlling for them will intermittently fail for reasons unrelated to actual bugs (a classic source of "flaky tests," which erode trust in the whole suite).

## 5. Comparisons / Testing Levels

| Level | Scope | Speed | Use in this course |
|---|---|---|---|
| Unit test | One function/class, in isolation | Fast (milliseconds) | Validating tax calculation, guardrail logic, Pydantic validators |
| Integration test | Multiple components together (e.g., API + real test DB) | Slower | Verifying an endpoint's full request/response cycle |
| End-to-end test | The whole system, as a user would experience it | Slowest | Verifying OmniGuard's full discovery-to-deployment flow works |

This topic focuses on the fastest, most foundational layer — but a mature test suite for a real project (like the capstones) needs all three levels, in roughly a pyramid: many unit tests, fewer integration tests, very few end-to-end tests.

## 6. Real-World Enterprise Scenario

Before OmniGuard's Text-to-SQL guardrail (the `QueryPlan` limit ceiling from *Pydantic data validation*) ever gets near a real MS SQL database, it needs a unit test suite proving: a plan requesting `limit=10_000_000` gets rejected or clamped, a plan referencing a table not on the allow-list gets rejected, and a well-formed plan passes through unchanged. These are exactly the kind of security-critical, easy-to-get-wrong-silently checks that must be verified by an automated test, not by a developer manually trying a few inputs once and assuming it's fine forever after.

## 7. Common Pitfalls

- **Writing tests only for the happy path** — the failure/edge cases (empty input, negative numbers, malformed data, permission denied) are usually where real bugs live, and where security-relevant guardrails need the most confidence.
- **Testing implementation details instead of behavior** — e.g., asserting a private helper method was called a specific number of times, rather than asserting the observable output was correct. This makes refactoring unnecessarily painful.
- **Letting tests depend on execution order** — a test that only passes if another test ran first (because it depends on shared mutable state) is broken; tests should be independently runnable in any order.
- **Not running the test suite before every commit/push** — a test suite that exists but isn't actually run regularly provides false confidence; it needs to be wired into CI (Module 4) to have real value.

## 8. Self-Check Questions

1. Why should a test verify observable behavior rather than internal implementation details, and what breaks if it doesn't?
2. Why is `pytest.raises` — testing that an exception *should* occur — just as important as testing successful outcomes, especially for this course's projects?
3. In the OmniGuard scenario, what three specific behaviors would you want unit tests to lock in before the Text-to-SQL guardrail ever runs against a real database?

## 9. Connections

- **Pydantic data validation** (Module 2): the guardrail example above is exactly the kind of validator logic unit tests need to cover thoroughly.
- **Fixtures and mocking** (next topic): the mechanism that lets unit tests stay fast and isolated by replacing real dependencies (DB, network) with controlled test doubles.
- **Testing rails against jailbreak libraries** (Module 13): the same "test the failure path deliberately, don't just hope it works" discipline, applied specifically to security guardrails later in the course.
