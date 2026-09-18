# Object-Oriented Programming Basics

## What & Why

Nearly every framework used later in the course — FastAPI, Pydantic, LangGraph — hands you a base class and expects you to extend or compose it correctly. Getting comfortable with classes, inheritance, and composition now avoids cargo-culting framework code later without understanding it.

## How It Works

```python
from dataclasses import dataclass

class Shape:                       # base class
    def area(self) -> float:
        raise NotImplementedError

class Circle(Shape):                # inheritance — "is-a"
    def __init__(self, radius: float):
        self.radius = radius
    def area(self) -> float:
        return 3.14159 * self.radius ** 2

@dataclass
class Point:                        # dataclass: auto __init__/__repr__/__eq__
    x: float
    y: float
```

- **Inheritance** ("is-a"): a `Circle` *is a* `Shape`. Use it for genuine type hierarchies.
- **Composition** ("has-a"): a `Car` *has an* `Engine`. Prefer this when you just want to reuse behavior — it avoids fragile, deep inheritance chains.
- **Key dunder methods**: `__init__` (constructor), `__repr__` (debug string), `__eq__` (equality), `__hash__` (usable as dict key/set member).
- **`dataclass`**: generates boilerplate (`__init__`, `__repr__`, `__eq__`) for simple data-holding classes — the ancestor of the validated models used in Module 2 and Module 5.

## Pitfalls

- Reaching for inheritance when composition would be simpler and more flexible — deep hierarchies get brittle fast.
- Forgetting to call `super().__init__()` in a subclass constructor, silently skipping base-class setup.
- Defining `__eq__` without `__hash__` — makes instances unhashable by default, breaking their use in sets/dict keys.

## Connections

This is the direct prerequisite for Pydantic's `BaseModel` (Module 2, *Pydantic data validation*) and the nested schema modeling used to validate LLM output (Module 5, *Defining complex nested Pydantic models*).
