---
tags: [python, web-crawling, metadata, beautifulsoup]
type: implementation
area: learning
---

# Document Metadata Extraction in Python

See [[Concepts/Document_Metadata]] for theory on what metadata is and why it matters.

```bash
pip install beautifulsoup4 lxml
```

---

## Extracting All Key Fields

```python
from bs4 import BeautifulSoup
from dataclasses import dataclass
from datetime import datetime


@dataclass
class PageMetadata:
    url: str
    title: str
    description: str
    h1: str
    headings: list[str]   # all h2–h6 text
    crawled_at: str


def extract_metadata(html: str, url: str) -> PageMetadata:
    soup = BeautifulSoup(html, "html.parser")

    # Title
    title_tag = soup.find("title")
    title = title_tag.get_text(strip=True) if title_tag else ""

    # Meta description
    meta_desc = soup.find("meta", attrs={"name": "description"})
    description = meta_desc.get("content", "").strip() if meta_desc else ""

    # H1 — take the first one
    h1_tag = soup.find("h1")
    h1 = h1_tag.get_text(strip=True) if h1_tag else ""

    # All headings h2–h6
    headings = [
        tag.get_text(strip=True)
        for tag in soup.find_all(["h2", "h3", "h4", "h5", "h6"])
    ]

    return PageMetadata(
        url=url,
        title=title,
        description=description,
        h1=h1,
        headings=headings,
        crawled_at=datetime.utcnow().isoformat(),
    )
```

---

## Usage

```python
html = """
<html>
<head>
  <title>Python Tips for Beginners</title>
  <meta name="description" content="Learn Python faster with these practical tips.">
</head>
<body>
  <h1>Python Tips</h1>
  <h2>Variables</h2>
  <h2>Functions</h2>
  <p>Some content here...</p>
</body>
</html>
"""

meta = extract_metadata(html, "https://example.com/python-tips")
print(meta.title)        # 'Python Tips for Beginners'
print(meta.description)  # 'Learn Python faster with these practical tips.'
print(meta.h1)           # 'Python Tips'
print(meta.headings)     # ['Variables', 'Functions']
```

---

## Storing Metadata in SQLite

```python
import sqlite3

def store_page(meta: PageMetadata, conn: sqlite3.Connection) -> int:
    cursor = conn.execute(
        """
        INSERT OR IGNORE INTO pages (url, title, description, crawled_at)
        VALUES (?, ?, ?, ?)
        """,
        (meta.url, meta.title, meta.description, meta.crawled_at),
    )
    conn.commit()
    return cursor.lastrowid
```

---

## Gotchas

- `<title>` can be `None` — always guard with `if title_tag`
- `<meta name="description">` may be missing on many pages — default to `""`
- Some pages have multiple `<h1>` tags (bad practice but it happens) — `find()` takes the first
- `content` attribute on meta tags vs. `text` — always use `.get("content")`

---

## Related Notes

- [[Concepts/Document_Metadata]] — theory, field reference
- [[Python/HTML_Parsing_BeautifulSoup]] — BeautifulSoup mechanics
- [[Python/Text_Extraction_Python]] — getting body text
- [[WebCrawling]] — hub
- [[Notes/Resources/Python/Dataclasses]] — `@dataclass` reference
