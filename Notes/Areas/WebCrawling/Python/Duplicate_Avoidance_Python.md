---
tags: [python, web-crawling, deduplication, hashlib]
type: implementation
area: learning
---

# Duplicate Avoidance in Python

See [[Concepts/Duplicate_Avoidance]] for theory on URL vs content deduplication, hashing algorithms, and Bloom filters.

---

## 1. URL Deduplication — `set`

```python
visited_urls: set[str] = set()

def should_crawl(normalized_url: str) -> bool:
    return normalized_url not in visited_urls

def mark_visited(normalized_url: str):
    visited_urls.add(normalized_url)
```

Always normalize before inserting. `"http://Example.COM/page"` and `"http://example.com/page"` are different strings but the same URL.

---

## 2. Content Deduplication — `hashlib`

Catches different URLs that serve identical page content (mirrors, printer-friendly versions, session-ID variants).

```python
import hashlib

content_hashes: set[str] = set()

def is_duplicate_content(body: bytes) -> bool:
    """Return True if this exact page content was seen before."""
    digest = hashlib.sha256(body).hexdigest()
    if digest in content_hashes:
        return True
    content_hashes.add(digest)
    return False
```

`body` must be **bytes** (`response.content`, not `response.text`).

---

## 3. URL Fingerprinting — Compact MD5 Key

Useful when you want a short, fixed-size identifier for database storage instead of storing full URL strings.

```python
import hashlib

def url_fingerprint(normalized_url: str) -> str:
    """32-char MD5 hex digest of a normalized URL."""
    return hashlib.md5(normalized_url.encode("utf-8")).hexdigest()

fp_seen: set[str] = set()

def should_crawl_by_fp(url: str) -> bool:
    fp = url_fingerprint(url)
    if fp in fp_seen:
        return False
    fp_seen.add(fp)
    return True
```

---

## 4. Putting It Together — Crawl Loop

```python
import hashlib, requests

visited_urls: set[str] = set()
content_hashes: set[str] = set()

def crawl(url: str):
    # Must normalize url before this call
    if url in visited_urls:
        return
    visited_urls.add(url)

    try:
        r = requests.get(url, timeout=10)
        r.raise_for_status()
    except Exception as e:
        print(f"Error: {e}")
        return

    # Content dedup
    digest = hashlib.sha256(r.content).hexdigest()
    if digest in content_hashes:
        print(f"Duplicate content at: {url}")
        return
    content_hashes.add(digest)

    print(f"New page: {url}")
    # ... parse links, add to frontier ...
```

---

## 5. Memory-Efficient Option — Bloom Filter

For very large crawls (millions of URLs), a plain `set` consumes too much RAM. The `pybloom-live` library offers probabilistic membership testing.

```bash
pip install pybloom-live
```

```python
from pybloom_live import ScalableBloomFilter

bloom = ScalableBloomFilter(
    mode=ScalableBloomFilter.SMALL_SET_GROWTH,
    error_rate=0.001,   # 0.1% false positive rate
)

def should_crawl_bloom(normalized_url: str) -> bool:
    if normalized_url in bloom:
        return False   # definitely seen before (or 0.1% false positive)
    bloom.add(normalized_url)
    return True
```

A Bloom filter has **no false negatives** — it will never tell you a URL is new when it has been seen. The false positive rate means it may occasionally skip a URL that was actually new.

---

## Gotchas

- **Always normalize before hashing or set-insertion** — the whole point of dedup is that `example.com/page` and `Example.COM/page` resolve to the same thing
- **`hashlib` takes bytes** — always `.encode("utf-8")` strings first, or pass `response.content` directly
- **MD5 is fine for dedup, not security** — fast and collision-resistant enough for fingerprinting; do not use for passwords
- **Keep `seen` and `visited` separate** — add to `seen` at enqueue time so the same URL is never queued twice before it is fetched

---

## Related Notes

- [[Concepts/Duplicate_Avoidance]] — theory: levels of dedup, hashing algorithms, Bloom filters
- [[WebCrawling]] — hub
- [[Python/URL_Normalization_Python]] — must normalize before dedup
- [[Python/URL_Frontier_Python]] — the `seen` set lives alongside the frontier
