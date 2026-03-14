---
tags: [web-crawling, deduplication, hashing]
type: concept
area: learning
---

# Duplicate Avoidance

Duplicate avoidance prevents a crawler from wasting resources fetching the same content twice. It operates at two levels: **URL deduplication** (same URL) and **content deduplication** (same page, different URLs).

---

## Key Vocabulary

| Term | Meaning |
|---|---|
| **Visited set** | A Python `set` of normalized URLs already fetched |
| **Seen set** | URLs already added to the frontier (superset of visited) |
| **URL fingerprint** | A hash of a normalized URL used as a compact key |
| **Content hash** | A hash of the page body to detect duplicate content |
| **MD5** | 128-bit hash — fast, fine for deduplication (not cryptographic security) |
| **SHA-1** | 160-bit hash — historically used in crawl deduplication |
| **SHA-256** | 256-bit hash — current standard |
| **SimHash / MinHash** | Locality-sensitive hashing for *near-duplicate* detection |
| **Bloom filter** | Probabilistic set — no false negatives, tunable false positive rate, memory-efficient |

---

## Two Levels of Deduplication

### Level 1 — URL Deduplication

Cheapest: [[Notes/Resources/CS_Fundamentals/Big_O_Notation|O(1)]] set lookup. Catch the same URL visited via different paths.

**Critical:** always normalize before hashing or inserting into the set. `http://Example.COM/page` and `http://example.com/page` are different strings but the same resource.

### Level 2 — Content Deduplication

Catches pages that have different URLs but serve identical content — common with:
- Trailing slash variants (`/page` vs `/page/`)
- Session IDs in query strings
- Printer-friendly mirrors (`/print/article`)
- 301 redirect targets

Hash the raw response body bytes. If the hash is in `content_hashes`, skip processing.

---

## Hashing Considerations

| Algorithm | Size | Speed | Use for |
|---|---|---|---|
| MD5 | 32 hex chars | Fastest | URL/content dedup (not security) |
| SHA-1 | 40 hex chars | Fast | Crawl dedup (legacy standard) |
| SHA-256 | 64 hex chars | Medium | When collision resistance matters |

All `hashlib` functions take **bytes** — always `.encode('utf-8')` strings first.

---

## Near-Duplicate Detection

Exact hashing misses pages that are *mostly* the same — like a page with a changing timestamp or rotating ad banner. For near-duplicate detection, use **SimHash** (`pip install simhash`). Out of scope for a mini search engine but good to know exists.

---

## Memory at Scale

A Python `set` grows linearly. 100 million URL strings → 8–16 GB RAM. For large crawls:
- **Bloom filter** (`pybloom-live`) — probabilistic, tunable error rate, much smaller
- **Redis SET** — persistent, shareable across processes
- **SQLite** — simple persistent dedup for medium-scale crawls

---

## Related Notes

- [[WebCrawling]] — hub
- [[Python/Duplicate_Avoidance_Python]] — implementation
- [[Concepts/URL_Normalization]] — must normalize before deduplicating
- [[Concepts/URL_Frontier]] — seen set vs visited set distinction

## Open Questions

- What false positive rate is acceptable for a Bloom filter in a crawler?
- How does SimHash work for near-duplicate detection?
- How do you persist the visited set across crawler restarts?
