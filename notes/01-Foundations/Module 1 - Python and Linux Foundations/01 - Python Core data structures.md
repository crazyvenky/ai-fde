# Python Core Data Structures

## What & Why

Python's four built-in collection types — `list`, `tuple`, `dict`, `set` — are the vocabulary every later data pipeline, API payload, and agent state object is built from. Picking the wrong one is the single most common source of accidental O(n²) code in a codebase.

## How It Works

- **`list`** — ordered, mutable, allows duplicates. Backed by a dynamic array.
- **`tuple`** — ordered, immutable, allows duplicates. Hashable if its contents are, so it can be a dict key.
- **`dict`** — key→value mapping, insertion-ordered since Python 3.7, backed by a hash table.
- **`set`** — unordered, unique elements, backed by a hash table (like `dict` without values).

```python
# Comprehensions are the idiomatic way to build these from an iterable
squares = [x * x for x in range(10)]          # list
lookup  = {x: x * x for x in range(10)}       # dict
uniques = {x % 3 for x in range(10)}          # set
```

## Comparisons / Tradeoffs

| Operation | list | dict | set |
|---|---|---|---|
| Index/key lookup | O(n) by value, O(1) by index | O(1) avg | O(1) avg (membership) |
| Insert at end | O(1) amortized | O(1) avg | O(1) avg |
| Membership test (`in`) | O(n) | O(1) avg | O(1) avg |
| Ordered? | Yes | Yes (insertion) | No |

## Pitfalls

- Using a `list` for membership checks in a hot loop instead of a `set` — silently turns O(n) into O(n²) as the list grows.
- Mutable default arguments (`def f(x=[])`) share the same list object across every call — covered in depth in the next topic.
- Confusing shallow copy (`list(x)`, `x[:]`, `dict(x)`) with deep copy — nested mutable objects are still shared after a shallow copy.

## Connections

Feeds directly into Pydantic model fields (Module 2, *Pydantic data validation*) and into how retrieved chunks/metadata are represented before being embedded (Module 6, *Vector Search and Core RAG*).
