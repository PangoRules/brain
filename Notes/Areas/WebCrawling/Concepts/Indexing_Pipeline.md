---
tags: [web-crawling, indexing, pipeline, architecture]
type: concept
area: learning
---

# Indexing Pipeline

The indexing pipeline is the sequence of transformations that takes a **raw crawled page** and turns it into **searchable index entries (postings)**. It sits between the crawler and the search query engine.

---

## Pipeline Stages

```
Raw HTML
   ↓
[1] Parse HTML  (BeautifulSoup)
   ↓
[2] Extract metadata  (title, URL, headings, description)
   ↓
[3] Extract visible text  (strip scripts/styles, get body text)
   ↓
[4] Content deduplication  (hash check → skip or continue)
   ↓
[5] Tokenize + normalize  (lowercase, punctuation, stop words)
   ↓
[6] Compute term frequencies  (count occurrences per term)
   ↓
[7] Store postings  (term, doc_id, frequency → SQLite)
   ↓
Searchable Index
```

---

## Pipeline Pattern in Python

Each stage is a function. They chain together — output of one is input of the next.

```python
def index_page(url: str, html: str, db_conn):
    metadata  = extract_metadata(html, url)
    text      = extract_text(html)
    if is_duplicate(text, db_conn):
        return
    tokens    = tokenize_and_normalize(text)
    term_freq = compute_term_frequencies(tokens)
    doc_id    = store_page(metadata, db_conn)
    store_postings(doc_id, term_freq, db_conn)
```

This is the **pipeline pattern**: each function does one thing, passes its result forward.

---

## Why This Separation Matters

| Concern | Stage |
|---|---|
| Raw HTML → structured data | Parse |
| Structured data → clean text | Extract |
| Avoid wasted work | Dedup |
| Text → index terms | Tokenize |
| Terms → searchable store | Store |

Each stage is independently testable and replaceable.

---

## What Goes into the Database

After the pipeline runs, the database has two main tables:

```
pages(id, url, title, description, content_hash, crawled_at)
postings(term, doc_id, frequency)
```

A search query looks up `term` in `postings`, gets a list of `doc_id`s, then retrieves page metadata from `pages`.

---

## Related Notes

- [[WebCrawling]] — hub
- [[Concepts/Text_Extraction]] — stage 3
- [[Concepts/Document_Metadata]] — stage 2
- [[Concepts/Content_Deduplication]] — stage 4
- [[NLP/Concepts/Inverted_Index]] — the data structure being built
- [[NLP/Concepts/Postings]] — what's stored in stage 7
- [[NLP/Concepts/Tokenization]] — stage 5
- [[NLP/Concepts/TF_IDF]] — term frequency theory
- [[Python/Indexing_Pipeline_Python]] — full implementation

## Open Questions

- How do you handle pipeline failures mid-way (partial indexing)?
- When would you parallelize pipeline stages?
- How do you re-index a page that has been updated?
