---
tags: [python, reference, file-io, pathlib, corpus]
type: reference
area: learning
---

# File I/O

Reading text files with Python's built-in `open()` and `pathlib.Path` — essential for loading corpora and processing large text collections in NLP.

---

## Key Approaches

| Approach | What it does | NLP use case |
|---|---|---|
| `Path.read_text(encoding)` | Read entire file as string | Small documents, config files |
| `open()` + `with` block | Safe file handle management | Standard single-file reading |
| Line-by-line `for line in f` | Stream one line at a time | Large files that don't fit in RAM |
| `Path.glob(pattern)` | Iterate matching files in directory | Batch-process a corpus directory |
| `errors='replace'` | Substitute undecodable bytes with `?` | Handling encoding errors in dirty corpora |

---

## Reading a Single File

```python
from pathlib import Path

path = Path("corpus/document.txt")

# Option A: one-liner (small files)
text = path.read_text(encoding="utf-8")

# Option B: explicit open (preferred when you need more control)
with open(path, encoding="utf-8") as f:
    text = f.read()

print(text[:80])
```

```
Output: (first 80 characters of the file)
```

Always specify `encoding="utf-8"` — omitting it uses the OS default, which varies across platforms.

---

## Line-by-Line for Large Files

```python
from pathlib import Path

path = Path("corpus/large_corpus.txt")
tokens_per_doc = []

with open(path, encoding="utf-8") as f:
    for line in f:
        line = line.strip()
        if line:
            tokens_per_doc.append(line.split())

print(f"Loaded {len(tokens_per_doc)} documents")
```

```
Output: Loaded N documents
```

Iterating `for line in f` streams one line at a time — constant memory regardless of file size.

---

## Batch Processing a Corpus

```python
from pathlib import Path

corpus_dir = Path("corpus/")

documents = {}
for path in sorted(corpus_dir.glob("*.txt")):
    text = path.read_text(encoding="utf-8", errors="replace")
    documents[path.stem] = text

print(f"Loaded {len(documents)} files")
print(list(documents.keys())[:5])
```

```
Output: Loaded N files
Output: ['doc001', 'doc002', 'doc003', 'doc004', 'doc005']
```

`path.stem` gives the filename without extension — useful as a document ID.

---

## Encoding Error Handling

```python
from pathlib import Path

path = Path("corpus/messy_file.txt")

# 'replace': substitute undecodable bytes with '?'
text_safe = path.read_text(encoding="utf-8", errors="replace")

# 'ignore': silently drop undecodable bytes
text_clean = path.read_text(encoding="utf-8", errors="ignore")

# 'strict' (default): raise UnicodeDecodeError on bad bytes
try:
    text_strict = path.read_text(encoding="utf-8")
except UnicodeDecodeError as e:
    print(f"Encoding error: {e}")
```

```
Output (if error): Encoding error: 'utf-8' codec can't decode byte ...
```

For NLP corpora: prefer `errors='replace'` to preserve document structure while flagging issues.

---

## Writing Processed Output

```python
from pathlib import Path

output_dir = Path("output/")
output_dir.mkdir(parents=True, exist_ok=True)

tokens = ["hello", "world", "nlp"]

out_path = output_dir / "tokens.txt"
out_path.write_text("\n".join(tokens), encoding="utf-8")

print(f"Wrote {out_path}")
```

```
Output: Wrote output/tokens.txt
```

---

## Related Notes

- [[Notes/Areas/NLP/Python/Inverted_Index_Python]] — builds an index from files loaded with these patterns
- [[Notes/Areas/NLP/Python/Normalizing_Textual_Data_in_Python]] — normalization pipeline applied to text loaded from files

---

## Open Questions

- When to use `Path.read_text()` vs `open()` — are there meaningful differences?
- Best approach for reading JSON-lines (`.jsonl`) corpora line by line?
- How does `mmap` help for very large read-only corpora?
