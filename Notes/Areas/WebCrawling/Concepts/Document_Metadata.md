---
tags: [web-crawling, indexing, metadata]
type: concept
area: learning
---

# Document Metadata

Document metadata is structured information **about** a page, separate from its body text. It tells the search engine what a page is, not just what it contains.

---

## Why Metadata Matters

Metadata fields are high-signal inputs for both indexing and ranking:
- The `<title>` is the strongest relevance signal on a page
- Headings structure the page and reveal key topics
- The URL itself carries semantic meaning (e.g., `/blog/python-tips`)
- Meta description is what search engines often display in results

---

## Key Fields to Extract

| Field | Source | Signal Strength | Notes |
|---|---|---|---|
| **Title** | `<title>` tag | Very high | Usually the best single descriptor |
| **URL** | Passed in from crawler | High | Slug often reveals topic |
| **H1** | `<h1>` | High | Should match the title conceptually |
| **H2–H6** | `<h2>` through `<h6>` | Medium | Section structure |
| **Meta description** | `<meta name="description">` | Medium | Author-written summary |
| **Lang** | `<html lang="...">` | Low | Useful for multilingual filtering |

---

## Metadata vs. Content

| | Metadata | Content |
|---|---|---|
| What | About the page | The page itself |
| Example | Title, URL, headings | Body paragraphs |
| Structured? | Yes — specific fields | No — raw text |
| Indexed? | Both | Yes |
| Stored? | Yes — in the pages table | Yes — in the index |

---

## Storage Pattern

Metadata typically goes in a `pages` table in the database:

```
pages(id, url, title, description, crawled_at, content_hash)
```

While the raw text feeds into the postings/index tables.

---

## Related Notes

- [[WebCrawling]] — hub
- [[Python/Document_Metadata_Python]] — implementation
- [[Concepts/Text_Extraction]] — extracting body text (separate from metadata)
- [[Concepts/Indexing_Pipeline]] — where metadata extraction fits
- [[NLP/Concepts/Postings]] — where extracted data ends up
