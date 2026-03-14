---
tags: [python, reference, dataclasses, data-modeling]
type: reference
area: learning
---

# Dataclasses

Python's `@dataclass` decorator for boilerplate-free data containers — useful for modeling tokens, documents, and NLP pipeline structures cleanly.

---

## Key Features

| Feature | What it does | NLP use case |
|---|---|---|
| `@dataclass` | Auto-generates `__init__`, `__repr__`, `__eq__` | Token / Document model |
| `field(default_factory=...)` | Safe mutable defaults (list, dict) | Document.tokens: list[str] = field(default_factory=list) |
| `__post_init__` | Run code after `__init__` | Compute derived fields (token_count, vocab) |
| `frozen=True` | Make instance immutable (adds `__hash__`) | Hashable Token — usable in sets/dict keys |
| `slots=True` (3.10+) | Use `__slots__` for lower memory | Memory-efficient Token for large corpora |

---

## Token Model

```python
from dataclasses import dataclass

@dataclass
class Token:
    text: str
    pos: int          # position in document
    lemma: str = ""   # optional, set by lemmatizer

t = Token(text="running", pos=3, lemma="run")
print(t)
# Token(text='running', pos=3, lemma='run')
```

```
Output: Token(text='running', pos=3, lemma='run')
```

---

## Document Model with `__post_init__`

```python
from dataclasses import dataclass, field

@dataclass
class Document:
    doc_id: str
    text: str
    tokens: list[str] = field(default_factory=list)
    token_count: int = field(init=False)  # derived, not passed by caller

    def __post_init__(self):
        if not self.tokens:
            self.tokens = self.text.split()
        self.token_count = len(self.tokens)

doc = Document(doc_id="doc001", text="The cat sat on the mat")
print(doc.token_count)   # 6
print(doc.tokens[:3])    # ['The', 'cat', 'sat']
```

```
Output token_count: 6
Output tokens[:3]:  ['The', 'cat', 'sat']
```

`field(init=False)` means `token_count` is excluded from `__init__` — it's set in `__post_init__`.

---

## Frozen / Hashable Token

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Token:
    text: str
    pos_tag: str

# Can now be used in sets and as dict keys
tokens = {Token("cat", "NN"), Token("sat", "VBD"), Token("cat", "NN")}
print(tokens)
# {Token(text='cat', pos_tag='NN'), Token(text='sat', pos_tag='VBD')}
```

```
Output: {Token(text='cat', pos_tag='NN'), Token(text='sat', pos_tag='VBD')}
```

`frozen=True` prevents mutation and generates `__hash__` — necessary for set/dict membership.

---

## Memory-Efficient with `slots=True` (Python 3.10+)

```python
from dataclasses import dataclass

@dataclass(slots=True)
class Token:
    text: str
    pos: int
    lemma: str = ""

# Uses __slots__ internally — lower per-instance memory overhead
# Useful when building a list of millions of tokens
tokens = [Token(text=w, pos=i) for i, w in enumerate("the cat sat on the mat".split())]
print(tokens[0])
# Token(text='the', pos=0, lemma='')
```

```
Output: Token(text='the', pos=0, lemma='')
```

Benchmark before using `slots=True` — the gains are most significant for millions of instances.

---

## Related Notes

- [[Notes/Areas/NLP/Python/Inverted_Index_Python]] — Document model can be used as the index input
- [[Notes/Areas/NLP/Python/TF_IDF_Sklearn]] — Document dataclass can wrap TF-IDF vectors alongside raw text

---

## Open Questions

- When to use `@dataclass` vs `typing.TypedDict` vs `pydantic.BaseModel`?
- Does `frozen=True` + `slots=True` together give additive benefits?
- How do dataclasses interact with `pickle` for caching processed corpora?
