# Pydantic Data Validation

## 1. What This Is & Why an FDE Needs It

Pydantic is the validation layer underneath almost everything in this course: FastAPI request/response bodies (this module), structured LLM output parsing (Module 5), and the schemas that describe tool arguments to an agent (Module 5, 9, 10). If you understand Pydantic deeply here, three later modules become dramatically easier — this is one of the highest-leverage topics in the entire curriculum.

The core problem it solves: **untrusted input arrives as untyped, unstructured data** (raw JSON from a client, raw text from an LLM) and your code needs typed, validated, guaranteed-correct Python objects before it does anything with that data. Pydantic is the boundary between "data from the outside world" and "data my business logic can trust."

## 2. Core Mechanics

```python
from pydantic import BaseModel, Field, field_validator
from typing import Optional
from datetime import datetime

class Order(BaseModel):
    order_id: int
    customer_email: str
    total_amount: float = Field(..., gt=0, description="Must be positive")
    status: str = "pending"                       # default value
    notes: Optional[str] = None                    # explicitly optional
    created_at: datetime

    @field_validator("customer_email")
    @classmethod
    def validate_email_format(cls, v: str) -> str:
        if "@" not in v:
            raise ValueError("customer_email must contain '@'")
        return v.lower()                            # normalize while validating
```

What happens when you instantiate this from raw dict-like JSON:

```python
raw = {"order_id": "42", "customer_email": "USER@Example.com",
       "total_amount": "99.5", "created_at": "2026-01-15T10:00:00"}

order = Order(**raw)
# order.order_id == 42        (str "42" coerced to int)
# order.total_amount == 99.5  (str "99.5" coerced to float)
# order.customer_email == "user@example.com"  (validator normalized it)
# order.status == "pending"   (default applied, field wasn't in raw input)
```

If `total_amount` were `-5`, or `customer_email` had no `@`, Pydantic raises a `ValidationError` *before* your endpoint logic ever runs — this is the single biggest reason to validate at the boundary rather than deep inside business logic: fail fast, fail with a precise, structured error message, and never let invalid data reach code that assumes it's clean.

## 3. Validation Error Shape

```python
from pydantic import ValidationError

try:
    Order(order_id="not-a-number", customer_email="bad-email",
          total_amount=-5, created_at="2026-01-15T10:00:00")
except ValidationError as e:
    print(e.errors())
    # [
    #   {'type': 'int_parsing', 'loc': ('order_id',), 'msg': 'Input should be a valid integer', ...},
    #   {'type': 'value_error', 'loc': ('customer_email',), 'msg': "Value error, customer_email must contain '@'", ...},
    #   {'type': 'greater_than', 'loc': ('total_amount',), 'msg': 'Input should be greater than 0', ...},
    # ]
```

Note that Pydantic collects **all** validation errors in one pass, not just the first one — this matters for API usability: a client fixing one field at a time based on single-error feedback is a miserable integration experience. Always surface the full `errors()` list back to API consumers (with appropriate sanitization for sensitive fields).

## 4. Nested Models

Real enterprise payloads are rarely flat. Pydantic models nest naturally:

```python
class Address(BaseModel):
    street: str
    city: str
    zip_code: str

class Customer(BaseModel):
    name: str
    address: Address              # nested model — validated recursively

class Order(BaseModel):
    order_id: int
    customer: Customer            # Pydantic validates Customer, which validates Address
    items: list[str]
```

A malformed nested field produces a `loc` path showing exactly where: `('customer', 'address', 'zip_code')` — critical for debugging deeply nested enterprise API payloads (think: a Text-to-SQL result schema, or a multi-step tool-call argument structure).

## 5. Comparisons / Design Notes

| Approach | When to use |
|---|---|
| `BaseModel` with explicit fields | Standard case — known, stable schema |
| `field_validator` (custom validators) | Business-rule validation beyond type/range (format checks, cross-field consistency) |
| `model_validator` | Validation that depends on *multiple* fields together (e.g., "end_date must be after start_date") |
| Plain `dataclass` (no Pydantic) | Internal-only data with no external/untrusted input — skip validation overhead when there's nothing to validate against |

## 6. Real-World Enterprise Scenario

In OmniGuard (Module 15), the Text-to-SQL component generates a structured query plan from a natural-language request. Before that plan ever touches a real database connection, it's parsed into a Pydantic model:

```python
class QueryPlan(BaseModel):
    table: str
    columns: list[str]
    filters: dict[str, str]
    limit: int = Field(le=1000)   # hard ceiling regardless of what the LLM generates
```

If the LLM hallucinates a `limit` of 10 million rows, or a `table` name that doesn't match an allow-list, Pydantic's validation (plus custom validators checking against a known schema/allow-list) is the first line of defense — this is a **guardrail**, not just a data-quality nicety. This is precisely why Module 5's "Validating LLM responses natively against type hints" and Module 13's guardrails topics both point back to this mechanism.

## 7. Common Pitfalls

- **Validating too late**: parsing raw dicts by hand deep inside business logic instead of at the API/tool boundary — defeats the entire purpose; validate as early as possible.
- **Swallowing `ValidationError` and returning a generic 500**: throws away exactly the structured, actionable error information Pydantic worked hard to produce. Catch it explicitly and return a 422 with the `.errors()` payload.
- **Using `Optional[X] = None` when you actually mean "required but nullable"** — `Optional` in Pydantic just widens the type to allow `None`; it does **not** by itself make the field optional to omit. A field is only "not required" if it has a default value (`None` or otherwise).
- **Over-validating internal, already-trusted data** — re-validating data that already passed validation earlier in the same request (e.g., re-parsing a model you just built) adds overhead with no safety benefit.
- **Forgetting that validators run in field-declaration order** — a validator on `field_b` that depends on `field_a` having already been validated/coerced can break if the fields are declared in the wrong order (use a `model_validator` for genuine cross-field logic instead).

## 8. Self-Check Questions

1. Why does Pydantic collect *all* validation errors in one pass instead of stopping at the first one, and why does that matter for API design?
2. What's the difference between `Optional[str]` and `Optional[str] = None` in terms of whether the field can be omitted entirely?
3. In the OmniGuard Text-to-SQL example, why is a hard `limit` ceiling enforced via Pydantic considered a *guardrail* rather than just a data-quality check?

## 9. Connections

- **Object-oriented programming basics** (Module 1): `BaseModel` is built on standard Python class mechanics — this topic is the direct, practical payoff of that foundation.
- **Defining complex nested Pydantic models** (Module 5): the exact same nested-model mechanics shown here, applied specifically to structuring LLM output.
- **Validating LLM responses natively against type hints** (Module 5) and **Parsing and validating tool arguments** (Module 5): both are this exact validation pattern applied to AI-generated data instead of client-submitted data.
- **Constructing Hybrid RAG alongside secure Text-to-SQL** (Module 15): the `QueryPlan` guardrail example above is a direct preview of OmniGuard's actual architecture.
