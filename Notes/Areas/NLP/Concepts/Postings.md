---
tags: [nlp, information-retrieval, indexing]
type: concept
area: learning
---

# Postings

A **posting** is a single entry in an inverted index. It records that a specific term appeared in a specific document, along with supporting data like frequency or position.

A **posting list** is the full list of postings for one term — the answer to "which documents contain this word?"

---

## Anatomy of a Posting

| Field | Description | Example |
|---|---|---|
| `term` | The indexed word | `"python"` |
| `doc_id` | Which document | `42` |
| `frequency` | How many times in that doc | `7` |
| `positions` | Where exactly (optional) | `[12, 45, 103]` |

Minimal posting: `(term, doc_id)` — just "this term appears here."
Standard posting: `(term, doc_id, frequency)` — enough for TF-IDF ranking.
Positional posting: `(term, doc_id, frequency, positions)` — needed for phrase queries.

---

## Posting List vs. Inverted Index

| | Posting | Posting List | Inverted Index |
|---|---|---|---|
| Scope | One entry | All entries for one term | All posting lists |
| Example | `("python", 42, 7)` | All docs containing "python" | Entire index |

The inverted index is just all posting lists organized together.

---

## How Posting Lists Enable Search

**Single term query:** look up the posting list → get all doc_ids
**AND query:** intersect two posting lists → docs containing both terms
**Ranked query:** use `frequency` to compute TF-IDF scores, sort by score

---

## Storage: In-Memory vs. SQLite

| Approach | When to use |
|---|---|
| Python dict `{term: [doc_id, ...]}` | Small corpus, learning |
| SQLite `postings` table | Persistent, queryable, production-ready |

For the mini search engine, SQLite is the right choice — it survives restarts and supports SQL queries for ranked retrieval.

---

## Related Notes

- [[NLP]] — hub
- [[Concepts/Inverted_Index]] — the structure postings build
- [[Concepts/TF_IDF]] — how frequency is used for ranking
- [[Python/SQLite_Postings_Python]] — storing postings in SQLite
- [[WebCrawling/Concepts/Indexing_Pipeline]] — where postings are generated
