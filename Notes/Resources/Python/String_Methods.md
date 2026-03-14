---
tags: [python, reference, strings, text-processing]
type: reference
area: learning
---

# String Methods

Built-in Python string methods for cleaning, normalizing, and splitting text — the foundation of any NLP preprocessing pipeline.

---

## Key Methods

| Method | What it does | NLP use case |
|---|---|---|
| `str.split(sep)` | Split string on separator (default: whitespace) | Basic whitespace tokenization |
| `str.strip()` | Remove leading/trailing whitespace | Clean raw corpus lines |
| `str.lower()` | Convert to lowercase | Case normalization |
| `str.replace(old, new)` | Replace substring | Remove punctuation, expand contractions |
| `str.join(iterable)` | Join tokens with separator | Reconstruct text from token list |
| `str.startswith(prefix)` | Test prefix | Filter tokens (e.g. URLs starting with `http`) |
| `str.endswith(suffix)` | Test suffix | Filter by file extension or punctuation |

---

## Basic Tokenization

```python
text = "The quick brown fox jumps over the lazy dog."

tokens = text.split()
# ['The', 'quick', 'brown', 'fox', 'jumps', 'over', 'the', 'lazy', 'dog.']
```

```
Output: ['The', 'quick', 'brown', 'fox', 'jumps', 'over', 'the', 'lazy', 'dog.']
```

Note: punctuation stays attached — use `re.split()` or `str.replace()` for cleaner splits.

---

## Case Normalization

```python
tokens = ["The", "quick", "Brown", "FOX"]

normalized = [t.lower() for t in tokens]
# ['the', 'quick', 'brown', 'fox']
```

```
Output: ['the', 'quick', 'brown', 'fox']
```

---

## Whitespace Normalization

```python
lines = ["  hello world  \n", "\tnatural language\t", "  processing  "]

cleaned = [line.strip() for line in lines]
# ['hello world', 'natural language', 'processing']
```

```
Output: ['hello world', 'natural language', 'processing']
```

---

## Chained Pipeline

```python
text = "  Hello, World! This is NLP.  "

tokens = (
    text
    .strip()
    .lower()
    .replace(",", "")
    .replace("!", "")
    .replace(".", "")
    .split()
)
# ['hello', 'world', 'this', 'is', 'nlp']
```

```
Output: ['hello', 'world', 'this', 'is', 'nlp']
```

For more complex punctuation removal, prefer `re.sub()` — see [[Regex]].

---

## Joining Tokens Back

```python
tokens = ["hello", "world", "this", "is", "nlp"]

text = " ".join(tokens)
# 'hello world this is nlp'

# Join with newlines for output
output = "\n".join(tokens)
```

```
Output: 'hello world this is nlp'
```

---

## Related Notes

- [[Notes/Areas/NLP/Python/Normalizing_Textual_Data_in_Python]] — full normalization pipeline using these methods
- [[Notes/Areas/NLP/Python/Tokenization_in_Python]] — tokenization strategies using `split()` and regex

---

## Open Questions

- When does `str.split()` fall short compared to `nltk.word_tokenize()`?
- How does `str.maketrans()` / `str.translate()` compare to `str.replace()` for bulk punctuation removal?
