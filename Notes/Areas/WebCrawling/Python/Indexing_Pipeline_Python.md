---
tags: [python, web-crawling, indexing, pipeline, sqlite]
type: implementation
area: learning
---

# Indexing Pipeline in Python

See [[Concepts/Indexing_Pipeline]] for the theory and stage breakdown.

This note assembles all Phase 4 components into a working pipeline.

---

## Imports and Setup

```python
import hashlib
import re
import sqlite3
from collections import Counter
from dataclasses import dataclass
from datetime import datetime

from bs4 import BeautifulSoup


NOISE_TAGS = ["script", "style", "noscript", "header", "footer", "nav"]
STOP_WORDS = {"the", "a", "an", "is", "it", "in", "on", "at", "to", "and", "or", "of"}
```

---

## Stage 1 — Parse + Extract Text

```python
def extract_text(html: str) -> str:
    soup = BeautifulSoup(html, "html.parser")
    for tag in soup(NOISE_TAGS):
        tag.decompose()
    return soup.get_text(separator=" ", strip=True)
```

---

## Stage 2 — Extract Metadata

```python
@dataclass
class PageMetadata:
    url: str
    title: str
    description: str
    crawled_at: str


def extract_metadata(html: str, url: str) -> PageMetadata:
    soup = BeautifulSoup(html, "html.parser")
    title_tag = soup.find("title")
    meta_desc = soup.find("meta", attrs={"name": "description"})
    return PageMetadata(
        url=url,
        title=title_tag.get_text(strip=True) if title_tag else "",
        description=meta_desc.get("content", "").strip() if meta_desc else "",
        crawled_at=datetime.utcnow().isoformat(),
    )
```

---

## Stage 3 — Content Deduplication

```python
def content_hash(text: str) -> str:
    return hashlib.sha256(text.encode("utf-8")).hexdigest()


def is_duplicate(text: str, conn: sqlite3.Connection) -> bool:
    h = content_hash(text)
    return conn.execute(
        "SELECT 1 FROM pages WHERE content_hash = ?", (h,)
    ).fetchone() is not None
```

---

## Stage 4 — Tokenize + Term Frequency

```python
def compute_term_frequencies(text: str) -> dict[str, int]:
    tokens = re.findall(r"[a-z]+", text.lower())
    tokens = [t for t in tokens if t not in STOP_WORDS and len(t) > 1]
    return dict(Counter(tokens))
```

---

## Stage 5 — Store to SQLite

```python
def store_page(meta: PageMetadata, text: str, conn: sqlite3.Connection) -> int:
    h = content_hash(text)
    cursor = conn.execute(
        """
        INSERT OR IGNORE INTO pages (url, title, description, content_hash, crawled_at)
        VALUES (?, ?, ?, ?, ?)
        """,
        (meta.url, meta.title, meta.description, h, meta.crawled_at),
    )
    conn.commit()
    if cursor.lastrowid == 0:
        return conn.execute("SELECT id FROM pages WHERE url = ?", (meta.url,)).fetchone()[0]
    return cursor.lastrowid


def store_postings(doc_id: int, term_freq: dict[str, int], conn: sqlite3.Connection):
    conn.executemany(
        "INSERT OR REPLACE INTO postings (term, doc_id, frequency) VALUES (?, ?, ?)",
        [(term, doc_id, freq) for term, freq in term_freq.items()],
    )
    conn.commit()
```

---

## Full Pipeline Entry Point

```python
def index_page(url: str, html: str, conn: sqlite3.Connection):
    text = extract_text(html)

    if is_duplicate(text, conn):
        print(f"[SKIP] Duplicate content: {url}")
        return

    meta = extract_metadata(html, url)
    doc_id = store_page(meta, text, conn)
    term_freq = compute_term_frequencies(text)
    store_postings(doc_id, term_freq, conn)
    print(f"[OK] Indexed {url} ({len(term_freq)} terms)")
```

---

## Basic Search

```python
def search(query: str, conn: sqlite3.Connection) -> list[dict]:
    terms = query.lower().split()
    placeholders = ",".join("?" * len(terms))
    rows = conn.execute(
        f"""
        SELECT p.url, p.title, SUM(po.frequency) AS score
        FROM postings po
        JOIN pages p ON p.id = po.doc_id
        WHERE po.term IN ({placeholders})
        GROUP BY po.doc_id
        HAVING COUNT(DISTINCT po.term) = ?
        ORDER BY score DESC
        """,
        (*terms, len(terms)),
    ).fetchall()
    return [{"url": r[0], "title": r[1], "score": r[2]} for r in rows]
```

---

## Related Notes

- [[Concepts/Indexing_Pipeline]] — stage-by-stage theory
- [[Python/Text_Extraction_Python]] — extract_text detail
- [[Python/Document_Metadata_Python]] — extract_metadata detail
- [[Python/Content_Deduplication_Python]] — dedup detail
- [[NLP/Python/SQLite_Postings_Python]] — storage detail
- [[WebCrawling]] — hub
