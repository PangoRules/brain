---
tags: [python, nlp, sqlite, indexing, information-retrieval]
type: implementation
area: learning
---

# Storing Postings in SQLite

See [[Concepts/Postings]] for theory on what postings are and what they contain.

SQLite is the right storage layer for a mini search engine — persistent, queryable, zero setup.

---

## Schema

```sql
CREATE TABLE IF NOT EXISTS pages (
    id           INTEGER PRIMARY KEY AUTOINCREMENT,
    url          TEXT UNIQUE NOT NULL,
    title        TEXT,
    description  TEXT,
    content_hash TEXT,
    crawled_at   TEXT
);

CREATE TABLE IF NOT EXISTS postings (
    term     TEXT NOT NULL,
    doc_id   INTEGER NOT NULL REFERENCES pages(id),
    frequency INTEGER NOT NULL,
    PRIMARY KEY (term, doc_id)
);

CREATE INDEX IF NOT EXISTS idx_postings_term ON postings(term);
```

The `term` index is critical — every search query does a `WHERE term = ?` lookup.

---

## Setting Up the Database

```python
import sqlite3


def create_db(path: str = "search.db") -> sqlite3.Connection:
    conn = sqlite3.connect(path)
    conn.execute("PRAGMA journal_mode=WAL")   # better concurrent write performance
    conn.executescript("""
        CREATE TABLE IF NOT EXISTS pages (
            id           INTEGER PRIMARY KEY AUTOINCREMENT,
            url          TEXT UNIQUE NOT NULL,
            title        TEXT,
            description  TEXT,
            content_hash TEXT,
            crawled_at   TEXT
        );

        CREATE TABLE IF NOT EXISTS postings (
            term      TEXT NOT NULL,
            doc_id    INTEGER NOT NULL REFERENCES pages(id),
            frequency INTEGER NOT NULL,
            PRIMARY KEY (term, doc_id)
        );

        CREATE INDEX IF NOT EXISTS idx_postings_term ON postings(term);
    """)
    conn.commit()
    return conn
```

---

## Storing a Page

```python
def store_page(url: str, title: str, description: str,
               content_hash: str, crawled_at: str,
               conn: sqlite3.Connection) -> int:
    cursor = conn.execute(
        """
        INSERT OR IGNORE INTO pages (url, title, description, content_hash, crawled_at)
        VALUES (?, ?, ?, ?, ?)
        """,
        (url, title, description, content_hash, crawled_at),
    )
    conn.commit()
    # If the row already existed, fetch its id
    if cursor.lastrowid == 0:
        row = conn.execute("SELECT id FROM pages WHERE url = ?", (url,)).fetchone()
        return row[0]
    return cursor.lastrowid
```

---

## Storing Postings

```python
def store_postings(doc_id: int, term_freq: dict[str, int],
                   conn: sqlite3.Connection):
    conn.executemany(
        """
        INSERT OR REPLACE INTO postings (term, doc_id, frequency)
        VALUES (?, ?, ?)
        """,
        [(term, doc_id, freq) for term, freq in term_freq.items()],
    )
    conn.commit()
```

`executemany` batches the inserts — much faster than looping with individual `execute` calls.

---

## Computing Term Frequencies

```python
from collections import Counter
import re

STOP_WORDS = {"the", "a", "an", "is", "it", "in", "on", "at", "to", "and", "or"}


def compute_term_frequencies(text: str) -> dict[str, int]:
    tokens = re.findall(r"[a-z]+", text.lower())
    tokens = [t for t in tokens if t not in STOP_WORDS and len(t) > 1]
    return dict(Counter(tokens))
```

> For a more complete stop word list, use `nltk.corpus.stopwords` — see [[Removing_Stop_Words_Python]].

---

## Querying the Index

```python
def search(query: str, conn: sqlite3.Connection) -> list[dict]:
    terms = query.lower().split()

    # AND search: docs containing ALL query terms
    placeholders = ",".join("?" * len(terms))
    rows = conn.execute(
        f"""
        SELECT p.url, p.title, SUM(po.frequency) as score
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

## Full Usage Example

```python
conn = create_db("search.db")

# After crawling and extracting a page:
doc_id = store_page(
    url="https://example.com/python",
    title="Python Tips",
    description="Learn Python faster.",
    content_hash="abc123...",
    crawled_at="2026-03-13T00:00:00",
    conn=conn,
)

term_freq = compute_term_frequencies("Python is great for scripting and automation")
store_postings(doc_id, term_freq, conn)

results = search("python automation", conn)
for r in results:
    print(r["score"], r["title"], r["url"])
```

---

## Related Notes

- [[Concepts/Postings]] — theory, posting list anatomy
- [[Concepts/Inverted_Index]] — the structure being built
- [[NLP]] — hub
- [[WebCrawling/Python/Content_Deduplication_Python]] — dedup before storing
- [[WebCrawling/Concepts/Indexing_Pipeline]] — full pipeline
- [[Removing_Stop_Words_Python]] — for better stop word handling
- [[TF_IDF_Sklearn]] — TF-IDF scoring for ranking
