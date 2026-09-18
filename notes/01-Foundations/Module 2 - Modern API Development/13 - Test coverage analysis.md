# Test Coverage Analysis

## 1. What This Is & Why an FDE Needs It

Test coverage measures **which lines/branches of your code actually executed while running the test suite**. It's a useful diagnostic tool for finding *untested* code — but it is not, by itself, a measure of test *quality*, and treating it as one is one of the most common and costly misunderstandings in professional software engineering. An FDE needs to know how to read a coverage report correctly, and just as importantly, how to explain to a client or teammate why "100% coverage" is not the same claim as "this code is correct."

## 2. Generating and Reading a Coverage Report

```bash
pip install pytest-cov
pytest --cov=myapp --cov-report=term-missing
```

```
Name                     Stmts   Miss  Cover   Missing
------------------------------------------------------
myapp/orders.py             45      3    93%   67-69
myapp/guardrails.py         30     12    60%   15-22, 40-43
------------------------------------------------------
TOTAL                        75     15    80%
```

- **Stmts**: total executable statements in the file.
- **Miss**: statements that never ran during the test suite.
- **Cover**: percentage executed at least once.
- **Missing**: the exact line numbers never hit — this is the actionable part of the report. `myapp/guardrails.py` at 60% with lines `15-22, 40-43` uncovered tells you *exactly* where to look — likely an error-handling branch or an edge case nobody wrote a test for yet.

## 3. What Coverage Percentage Actually Tells You (and Doesn't)

**What it tells you**: a line with 0% coverage has *definitely* never been exercised by any test — if there's a bug on that line, no test would have caught it. This is genuinely useful for finding blind spots.

**What it does NOT tell you**:
- Whether the assertions in the tests that *did* execute that line are actually meaningful. A test that calls a function and asserts nothing about the result gives that function's lines 100% "coverage" while verifying literally nothing.
- Whether edge cases *within* a covered line are handled — a single `if x > 0:` line can be "covered" by a test using `x=5`, while `x=0` and `x=-1` (different logical branches) are never actually tested, depending on whether you're measuring line coverage or branch coverage (see below).
- Whether the code is *correct* — coverage measures execution, not correctness. Code with a bug can have 100% coverage if the tests happen not to assert on the buggy behavior.

## 4. Line Coverage vs Branch Coverage

```python
def classify(x):
    if x > 0:
        return "positive"     # line A
    else:
        return "non-positive" # line B
```

A test calling `classify(5)` gives **line coverage** credit to line A but not line B — depending on your tool's granularity, a naive line-coverage tool might report high coverage for this function even though the `else` branch was never exercised. **Branch coverage** specifically tracks whether *both* the true and false paths of every conditional were taken, which is a stricter and more meaningful signal than line coverage alone for logic-heavy, guardrail-style code.

## 5. Comparisons / Design Notes

| Coverage % | What it's reasonable to conclude |
|---|---|
| 0-40% | Large blind spots; high risk of untested bugs |
| 60-80% | Reasonable baseline for most application code; check *which* lines are missing, not just the percentage |
| 90-100% | Good — but always spot-check that critical paths (guardrails, security checks, retry logic) are specifically among the covered lines, not just incidentally covered by unrelated tests |
| Chasing 100% everywhere, uniformly | Often wasted effort — trivial getters/setters and framework boilerplate rarely need dedicated tests; the *risk-weighted* lines (parsing, validation, security, financial calculations) matter far more than the percentage itself |

## 6. Real-World Enterprise Scenario

For OmniGuard's guardrail module (Presidio/NeMo integration, Module 13), a coverage report showing 60% with the *specific* missing lines being the PII-redaction fallback path and the jailbreak-detection rejection branch is a serious, actionable finding — those are exactly the security-critical failure paths that must be verified, not incidentally-uncovered boilerplate. Reporting "we're at 60% coverage" without looking at *which* 40% is missing would miss that this specific gap is a security risk, not a minor housekeeping item.

## 7. Common Pitfalls

- **Treating a coverage percentage target as a proxy for quality** — chasing "90% coverage" as a goal in itself, rather than using coverage as a tool to find specific untested risk areas, produces tests written to hit lines rather than to verify behavior (e.g., calling a function with no assertions just to make the coverage tool happy).
- **Ignoring the `Missing` line numbers and only looking at the aggregate percentage** — the aggregate number is far less useful than knowing exactly which risk-relevant lines are uncovered.
- **Assuming high coverage means low bug risk** — as shown above, coverage and correctness are different axes entirely; a codebase can have both high coverage and a serious, untested-for-the-wrong-reason bug.
- **Not distinguishing risk-weighted code from boilerplate** — spending equal testing effort on a trivial `__repr__` method and a security guardrail's core logic misallocates limited testing time.

## 8. Self-Check Questions

1. Why can a function have 100% line coverage while still containing an untested, buggy edge case?
2. What's the practical difference between line coverage and branch coverage, and why does that difference matter more for guardrail-style conditional logic?
3. In the OmniGuard scenario, why does knowing *which* lines are uncovered matter more than the aggregate 60% figure by itself?

## 9. Connections

- **Unit testing fundamentals** (Module 2): coverage analysis is a diagnostic *for* the test suite built in that topic — it finds gaps, it doesn't replace writing meaningful tests.
- **Testing rails against jailbreak libraries** (Module 13): the guardrail-coverage example above is a direct preview of exactly the kind of risk-weighted coverage analysis that topic requires.
- **Tracking evaluation scores across deployment runs** (Module 14): both topics share the theme of "a single aggregate metric can hide exactly the detail that matters" — coverage % and an average eval score are both susceptible to this same blind spot.
