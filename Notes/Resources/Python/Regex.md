---
tags: [python, reference, regex, text-processing]
type: reference
area: learning
---

# Regular Expressions (`re`)

Python's `re` module for pattern-based text matching, extraction, and substitution — essential for robust NLP preprocessing.

---

## Key Functions

| Function | What it does | NLP use case |
|---|---|---|
| `re.compile(pattern, flags)` | Compile pattern for reuse | Build reusable tokenizer/cleaner |
| `re.findall(pattern, text)` | Return all non-overlapping matches as list | Extract emails, URLs, hashtags |
| `re.sub(pattern, repl, text)` | Replace matches with replacement string | Remove punctuation, normalize whitespace |
| `re.split(pattern, text)` | Split on pattern | Sentence/token splitting |
| `re.finditer(pattern, text)` | Return match objects (with position) | KWIC / span extraction |

## Common Flags

| Flag | Shorthand | Effect |
|---|---|---|
| `re.IGNORECASE` | `re.I` | Case-insensitive matching |
| `re.MULTILINE` | `re.M` | `^`/`$` match line boundaries |
| `re.DOTALL` | `re.S` | `.` matches newlines |

---

## Removing Punctuation and Numbers

```python
import re

text = "Hello, world! This is NLP (2024) — amazing."

# Remove anything that isn't a letter or whitespace
clean = re.sub(r"[^a-zA-Z\s]", "", text)
# 'Hello world This is NLP   amazing'

# Collapse multiple spaces
clean = re.sub(r"\s+", " ", clean).strip()
# 'Hello world This is NLP amazing'
```

```
Output: 'Hello world This is NLP amazing'
```

---

## Extracting Patterns (Emails, URLs)

```python
import re

text = "Contact us at info@example.com or visit https://nlp.org for details."

emails = re.findall(r"[\w.+-]+@[\w-]+\.[a-z]{2,}", text, re.I)
# ['info@example.com']

urls = re.findall(r"https?://\S+", text)
# ['https://nlp.org']
```

```
Output emails: ['info@example.com']
Output urls:   ['https://nlp.org']
```

---

## Tokenization with `re.split()`

```python
import re

text = "Hello, world! This is NLP."

# Split on whitespace or punctuation
tokens = re.split(r"[\s\W]+", text)
tokens = [t for t in tokens if t]  # remove empty strings
# ['Hello', 'world', 'This', 'is', 'NLP']
```

```
Output: ['Hello', 'world', 'This', 'is', 'NLP']
```

---

## Named Groups

```python
import re

# Extract year and title from citation
pattern = re.compile(r"\((?P<year>\d{4})\)\s+(?P<title>.+)")
text = "(2024) Attention Is All You Need"

m = pattern.search(text)
if m:
    print(m.group("year"))   # '2024'
    print(m.group("title"))  # 'Attention Is All You Need'
```

```
Output year:  '2024'
Output title: 'Attention Is All You Need'
```

---

## Compiled Pattern for Reuse

```python
import re

# Compile once, reuse across corpus
punct_re = re.compile(r"[^a-zA-Z\s]")
space_re = re.compile(r"\s+")

def normalize(text: str) -> str:
    text = punct_re.sub("", text)
    text = space_re.sub(" ", text)
    return text.strip().lower()

corpus = ["Hello, World!", "NLP is (really) fun.", "2024: the year of LLMs."]
cleaned = [normalize(doc) for doc in corpus]
# ['hello world', 'nlp is really fun', 'the year of llms']
```

```
Output: ['hello world', 'nlp is really fun', 'the year of llms']
```

---

## Related Notes

- [[Notes/Areas/NLP/Python/Normalizing_Textual_Data_in_Python]] — normalization pipeline using `re.sub()`
- [[Notes/Areas/NLP/Python/Tokenization_in_Python]] — tokenization strategies including regex-based splits

---

## Open Questions

- When to use `re.split()` vs `nltk.word_tokenize()` for tokenization?
- How do lookaheads/lookbehinds help with punctuation that should split tokens but be discarded?
- Performance: compiled `re` vs `str.replace()` for simple single-character substitutions?
