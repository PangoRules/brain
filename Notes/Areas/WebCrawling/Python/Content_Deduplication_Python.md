---
tags: [python, web-crawling, deduplication, hashing]
type: implementation
area: learning
---

# Content Deduplication in Python

See [[Concepts/Content_Deduplication]] for theory on why content dedup is different from URL dedup.

---

## Hashing Extracted Text

```python
import hashlib


def content_hash(text: str) -> str:
    """SHA-256 hash of the extracted text. Used for deduplication."""
    return hashlib.sha256(text.encode("utf-8")).hexdigest()
```

Always hash the **extracted, normalized text** — not raw HTML. This ensures minor HTML changes (whitespace, attributes) don't break dedup.

---

## Checking Against the Database

```python
import sqlite3


def is_duplicate(text: str, conn: sqlite3.Connection) -> bool:
    h = content_hash(text)
    row = conn.execute(
        "SELECT 1 FROM pages WHERE content_hash = ?", (h,)
    ).fetchone()
    return row is not None


def store_content_hash(page_id: int, text: str, conn: sqlite3.Connection):
    h = content_hash(text)
    conn.execute(
        "UPDATE pages SET content_hash = ? WHERE id = ?", (h, page_id)
    )
    conn.commit()
```

---

## SQLite Schema

```sql
CREATE TABLE IF NOT EXISTS pages (
    id           INTEGER PRIMARY KEY AUTOINCREMENT,
    url          TEXT UNIQUE NOT NULL,
    title        TEXT,
    description  TEXT,
    content_hash TEXT,          -- SHA-256 of extracted text
    crawled_at   TEXT
);

CREATE INDEX IF NOT EXISTS idx_content_hash ON pages(content_hash);
```

The index on `content_hash` makes dedup checks fast even with many pages.

---

## In the Pipeline

```python
def index_page(url: str, html: str, conn: sqlite3.Connection):
    text = extract_text(html)

    if is_duplicate(text, conn):
        print(f"Skipping duplicate: {url}")
        return

    meta = extract_metadata(html, url)
    doc_id = store_page(meta, conn)
    store_content_hash(doc_id, text, conn)

    term_freq = compute_term_frequencies(text)
    store_postings(doc_id, term_freq, conn)
```

---

## Gotchas

- Hash **after** normalization — if two pages differ only in whitespace, normalize first so they produce the same hash
- `content_hash` column should allow `NULL` initially (set after extraction)
- For a mini search engine, SHA-256 is overkill — MD5 or even a shorter hash works fine

---

## Related Notes

- [[Concepts/Content_Deduplication]] — theory, exact vs. near-duplicate
- [[Concepts/Duplicate_Avoidance]] — URL-level dedup (different stage)
- [[Python/Text_Extraction_Python]] — the text being hashed
- [[Python/SQLite_Postings_Python]] — storing the indexed data
- [[WebCrawling/Concepts/Indexing_Pipeline]] — where this fits
