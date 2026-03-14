---
tags: [python, nlp, text-processing, information-retrieval]
type: implementation
area: learning
---

# Inverted Index in Python

See [[Concepts/Inverted_Index]] for theory, forward vs. inverted index, and real-world use cases.

## Building an Inverted Index Step by Step

Given a text file with multiple lines, build a dictionary where each unique word maps to the line numbers it appears on.

### Step 1 — Read the File and Count Lines

```python
file = open('file.txt', encoding='utf8')
read = file.read()
file.seek(0)  # reset pointer to re-read line by line

line_count = read.count('\n') + 1
lines = [file.readline() for _ in range(line_count)]
```

> `file.seek(0)` resets the file pointer after reading the whole file — without it, `readline()` would return empty strings.

### Step 2 — Strip Punctuation and Lowercase

```python
import string

cleaned = read.lower()
for char in string.punctuation:
    cleaned = cleaned.replace(char, ' ')
```

### Step 3 — Tokenize and Remove Stop Words

```python
import nltk
from nltk.tokenize import word_tokenize
from nltk.corpus import stopwords

nltk.download('stopwords')
nltk.download('punkt')

tokens = word_tokenize(cleaned)
stop_words = set(stopwords.words('english'))
meaningful_tokens = [w for w in tokens if w not in stop_words]
```

### Step 4 — Build the Index

```python
index = {}

for line_num, line in enumerate(lines, start=1):
    line_lower = line.lower()
    for token in meaningful_tokens:
        if token in line_lower:
            if token not in index:
                index[token] = []
            if line_num not in index[token]:  # avoid duplicate line numbers
                index[token].append(line_num)

print(index)
```

## Complete Example

```python
import nltk
import string
from nltk.tokenize import word_tokenize
from nltk.corpus import stopwords

nltk.download('stopwords')
nltk.download('punkt')

# --- Sample file content (simulate reading from a file) ---
content = """first word example
second text hello world
third line appears here"""

lines = content.split('\n')

# Clean
cleaned = content.lower()
for char in string.punctuation:
    cleaned = cleaned.replace(char, ' ')

# Tokenize + remove stopwords
tokens = word_tokenize(cleaned)
stop_words = set(stopwords.words('english'))
tokens = [w for w in tokens if w not in stop_words]

# Build index
index = {}
for line_num, line in enumerate(lines, start=1):
    line_lower = line.lower()
    for token in tokens:
        if token in line_lower:
            if token not in index:
                index[token] = []
            if line_num not in index[token]:
                index[token].append(line_num)

print(index)
'''
Output:
{
  'first': [1], 'word': [1], 'example': [1],
  'second': [2], 'text': [2], 'hello': [2], 'world': [2],
  'third': [3], 'line': [3], 'appears': [3]
}
'''
```

## Querying the Index

```python
def search(index, query):
    query_tokens = query.lower().split()
    results = None
    for token in query_tokens:
        if token in index:
            if results is None:
                results = set(index[token])
            else:
                results &= set(index[token])  # AND search
    return sorted(results) if results else []

print(search(index, "hello world"))  # → [2]
print(search(index, "first"))        # → [1]
```

## Related Notes

- [[Concepts/Inverted_Index]] — theory, forward vs. inverted index, AND/OR queries
- [[NLP]] — hub
- [[Tokenization_in_Python]]
- [[Removing_Stop_Words_Python]]
- [[TF_IDF_Sklearn]]

## Python Tools Used

- [[Notes/Resources/Python/Dicts_as_Indexes|Dicts as Indexes]] — core data structure for the index
- [[Notes/Resources/Python/File_IO|File I/O]] — reading input files for indexing

## Open Questions

- How do production search engines handle index updates (insertions/deletions)?
- How does TF-IDF scoring plug into a search index for ranking?
- What is a positional inverted index and when is it needed?
