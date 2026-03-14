---
tags: [web-crawling, deduplication, indexing]
type: concept
area: learning
---

# Content Deduplication

Content deduplication is the process of detecting and skipping pages whose **content** has already been indexed — even if their URLs are different.

This is different from [[Concepts/Duplicate_Avoidance]], which prevents re-visiting the same URL. Content dedup handles cases where multiple URLs serve identical or near-identical content.

---

## Why It Happens

- `example.com/page` and `example.com/page?utm_source=twitter` → same content, different URLs
- `http://` vs `https://` variants
- `www.` vs. non-www versions
- Paginated pages with duplicate boilerplate
- Syndicated/mirrored content across domains

---

## Exact Deduplication (Hash-Based)

The simplest approach: hash the extracted text of each page and check if that hash already exists in the database.

```
content_hash = SHA256(extracted_text)
```

If the hash exists → skip indexing. If not → index and store the hash.

**Pros:** Fast, exact, zero false positives.
**Cons:** Any change in the text (even one character) produces a different hash. Doesn't catch near-duplicates.

---

## Near-Duplicate Detection (Advanced)

For pages that are mostly the same but not identical (e.g., different timestamps, banners):

| Technique | How it works |
|---|---|
| **Shingling** | Break text into overlapping N-grams; compare sets |
| **MinHash** | Approximates Jaccard similarity between shingle sets |
| **SimHash** | Produces a fingerprint; close fingerprints = similar content |

> For a mini search engine, exact hashing (SHA256) is sufficient.

---

## Where It Fits in the Pipeline

```
Crawl page → Extract text → Hash text → Check DB → Skip or Index
```

The hash should be computed **after** text extraction and normalization, not on raw HTML (otherwise minor HTML differences break the dedup).

---

## Related Notes

- [[WebCrawling]] — hub
- [[Concepts/Duplicate_Avoidance]] — URL-level dedup (different problem)
- [[Concepts/Indexing_Pipeline]] — where dedup sits
- [[Python/Content_Deduplication_Python]] — implementation
- [[NLP/Concepts/Postings]] — what gets stored after dedup passes

## Open Questions

- When would you choose SimHash over exact hashing?
- How do you handle content that changes slightly every visit (timestamps, ads)?
