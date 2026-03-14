---
tags: [python, reference, collections, data-structures, indexing]
type: reference
area: learning
---

# Dictionaries as Indexes

Python dicts and `collections.defaultdict` as lookup structures — the natural building block for inverted indexes, position indexes, and co-occurrence matrices in NLP.

---

## Key Patterns

| Pattern | What it does | NLP use case |
|---|---|---|
| `dict` | Key → value mapping | Term → document ID (simple index) |
| `defaultdict(list)` | Key → list (auto-initialized) | Term → [doc_id, doc_id, ...] |
| `defaultdict(set)` | Key → set (auto-initialized) | Term → {doc_id, ...} (no duplicates) |
| `defaultdict(int)` | Key → int (auto-initialized to 0) | Term → count |
| `defaultdict(lambda: defaultdict(int))` | Nested mapping | Co-occurrence matrix: word → {context_word: count} |

---

## Simple Inverted Index

```python
from collections import defaultdict

corpus = {
    0: ["the", "cat", "sat", "on", "the", "mat"],
    1: ["the", "dog", "sat", "on", "the", "log"],
    2: ["the", "cat", "ate", "the", "rat"],
}

index: dict[str, set[int]] = defaultdict(set)

for doc_id, tokens in corpus.items():
    for token in tokens:
        index[token].add(doc_id)

print(dict(index["cat"]))   # {0, 2}
print(dict(index["sat"]))   # {0, 1}
```

```
Output index["cat"]: {0, 2}
Output index["sat"]: {0, 1}
```

`defaultdict(set)` eliminates duplicate doc IDs automatically — no `if token not in index` guard needed.

---

## Word-Position Index (KWIC)

```python
from collections import defaultdict

# word → list of (doc_id, position) tuples
position_index: dict[str, list] = defaultdict(list)

corpus = {
    0: ["the", "cat", "sat", "on", "the", "mat"],
    1: ["the", "dog", "sat", "on", "the", "log"],
}

for doc_id, tokens in corpus.items():
    for pos, token in enumerate(tokens):
        position_index[token].append((doc_id, pos))

print(position_index["sat"])
# [(0, 2), (1, 2)]
```

```
Output: [(0, 2), (1, 2)]
```

Enables KWIC (keyword-in-context) lookup and proximity search.

---

## Co-occurrence Matrix

```python
from collections import defaultdict

# Nested defaultdict: target_word → {context_word: count}
cooc: dict = defaultdict(lambda: defaultdict(int))

tokens = ["the", "cat", "sat", "on", "the", "mat"]
window = 2

for i, word in enumerate(tokens):
    context = tokens[max(0, i - window): i] + tokens[i + 1: i + window + 1]
    for ctx_word in context:
        cooc[word][ctx_word] += 1

print(dict(cooc["cat"]))
# {'the': 2, 'sat': 1}
```

```
Output: {'the': 2, 'sat': 1}
```

---

## Boolean AND Query on Inverted Index

```python
from collections import defaultdict

# Reuse index from "Simple Inverted Index" above
def boolean_and(index, *terms):
    sets = [index[term] for term in terms]
    return set.intersection(*sets) if sets else set()

# Which docs contain both "cat" AND "sat"?
result = boolean_and(index, "cat", "sat")
# {0}
```

```
Output: {0}
```

---

## Related Notes

- [[Notes/Areas/NLP/Python/Inverted_Index_Python]] — full inverted index implementation
- [[Notes/Areas/NLP/Concepts/Inverted_Index]] — conceptual explanation of inverted indexes

---

## Open Questions

- When should you use `defaultdict` vs `dict.setdefault()` vs `collections.Counter`?
- How does a `defaultdict`-based inverted index compare to a sorted posting list (for ranked retrieval)?
- Memory: dict of sets vs sparse `scipy` matrix for large co-occurrence matrices?
