---
tags: [nlp, concepts, information-retrieval]
type: concept
area: learning
---

# Inverted Index

An **inverted index** is a data structure that maps each unique term to the list of documents (or positions) where it appears. It is the core data structure behind search engines.

```
Word → [doc1, doc3, doc7, ...]
```

Called "inverted" because it reverses the natural document→words relationship into a words→documents mapping.

## How It Works

1. **Tokenize** each document into words
2. **Normalize** tokens (lowercase, strip punctuation, remove stop words)
3. **Map** each token → list of document IDs where it appears
4. **Query:** look up the token directly instead of scanning all documents

Without an inverted index, a search requires scanning every document (O(n·m)). With one, lookup is O(1) per term.

## Forward Index vs. Inverted Index

| Property | Forward Index | Inverted Index |
|---|---|---|
| Structure | document → list of words | word → list of documents |
| Built for | Displaying document content | Answering "which docs contain X?" |
| Query speed | Slow (must scan all docs) | Fast (direct lookup) |
| Storage | More natural, easier to build | Requires preprocessing |
| Used in | Document storage, retrieval of full docs | Search engines, full-text search |

## AND / OR Queries

- **AND query:** intersect the posting lists for each term → documents containing all terms
- **OR query:** union the posting lists → documents containing any term
- **Phrase query:** requires positional information (not just doc IDs)

## Real-World Use

| System | How it uses an inverted index |
|---|---|
| Google | Massive distributed inverted index over the web |
| PostgreSQL | GIN indexes for full-text search |
| Elasticsearch | Lucene-based inverted index |
| grep / ripgrep | On-the-fly linear scan (no pre-built index) |

## Open Questions

- How do production search engines handle index updates (insertions/deletions)?
- How does TF-IDF scoring plug into a search index for ranking?
- What is a positional inverted index and when is it needed?

## Related Notes

- [[NLP]] — hub
- [[Python/Inverted_Index_Python]] — Python implementation
- [[TF_IDF]] — TF-IDF scoring is used with inverted indexes for ranking
- [[Tokenization]] — tokenization is step 1 of building an index
- [[Stop_Words]] — stop words are typically removed before indexing
