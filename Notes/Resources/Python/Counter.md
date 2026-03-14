---
tags: [python, reference, collections, frequency-analysis]
type: reference
area: learning
---

# `collections.Counter`

A dict subclass for counting hashable objects — the go-to tool for word frequency, n-gram counts, and corpus comparison in NLP.

---

## Key Operations

| Operation | What it does | NLP use case |
|---|---|---|
| `Counter(iterable)` | Count occurrences of each element | Word frequency from token list |
| `.most_common(n)` | Return top-n `(element, count)` pairs | Top words / vocabulary inspection |
| `c1 + c2` | Sum counts (union) | Merge frequency tables from multiple docs |
| `c1 - c2` | Subtract counts (drops negatives) | Remove common words from a distribution |
| `c1 & c2` | Intersection (min of counts) | Words shared by two corpora |
| `c1 \| c2` | Union (max of counts) | Combined vocabulary |
| `.elements()` | Repeat each element by its count | Reconstruct weighted token list |

---

## Word Frequency

```python
from collections import Counter

tokens = ["the", "cat", "sat", "on", "the", "mat", "the", "cat"]

freq = Counter(tokens)
# Counter({'the': 3, 'cat': 2, 'sat': 1, 'on': 1, 'mat': 1})

print(freq.most_common(3))
# [('the', 3), ('cat', 2), ('sat', 1)]
```

```
Output: [('the', 3), ('cat', 2), ('sat', 1)]
```

---

## N-gram Counts

```python
from collections import Counter

tokens = ["the", "cat", "sat", "on", "the", "mat"]

bigrams = list(zip(tokens, tokens[1:]))
bigram_freq = Counter(bigrams)

print(bigram_freq.most_common(3))
# [(('the', 'cat'), 1), (('cat', 'sat'), 1), ...]
```

```
Output: [(('the', 'cat'), 1), (('cat', 'sat'), 1), (('sat', 'on'), 1)]
```

---

## Corpus Comparison (Intersection / Subtraction)

```python
from collections import Counter

sports = Counter(["the", "ball", "goal", "the", "team", "ball"])
tech   = Counter(["the", "data", "model", "the", "code", "data"])

# Words in both corpora (shared vocabulary)
shared = sports & tech
# Counter({'the': 2})

# Words in sports but not (or less) in tech
sports_specific = sports - tech
# Counter({'ball': 2, 'goal': 1, 'team': 1})
```

```
Output shared:          Counter({'the': 2})
Output sports_specific: Counter({'ball': 2, 'goal': 1, 'team': 1})
```

---

## Filtering Low-frequency Tokens

```python
from collections import Counter

tokens = ["nlp", "is", "fun", "nlp", "is", "the", "best", "nlp"]

freq = Counter(tokens)

# Keep only tokens appearing >= 2 times
# Subtract a counter of all-ones to drop freq-1 items, then add positive filter
filtered = +Counter({w: c for w, c in freq.items() if c >= 2})
# Counter({'nlp': 3, 'is': 2})
```

```
Output: Counter({'nlp': 3, 'is': 2})
```

Tip: the `+` prefix drops zero/negative counts, making it a clean idiom after subtraction.

---

## Related Notes

- [[Notes/Areas/NLP/Python/TF_IDF_Sklearn]] — TF-IDF builds on raw frequency counts
- [[Notes/Areas/NLP/Python/Removing_Stop_Words_Python]] — stop word removal can use Counter subtraction

---

## Open Questions

- How does `Counter` compare to `numpy` or `pandas` value_counts for large corpora?
- Can `Counter` be used to build a unigram language model (probabilities)?
- Memory trade-offs: `Counter` vs sparse matrix representations?
