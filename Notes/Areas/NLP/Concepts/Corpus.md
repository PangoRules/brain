---
tags: [nlp, concepts, text-processing, information-retrieval]
type: concept
area: learning
---

# Corpus

A **corpus** (plural: *corpora*) is a structured collection of text documents used as the data set for NLP or information retrieval tasks. It is the raw material that every downstream operation — tokenization, indexing, TF-IDF scoring, search — works on.

In a search engine context, the corpus is the set of all pages the crawler has collected and the indexer has processed.

---

## Key Vocabulary

| Term | Meaning |
|---|---|
| **Corpus** | The full collection of documents |
| **Document** | A single unit in the corpus — a webpage, article, book chapter, etc. |
| **Token** | A single word or unit extracted from a document |
| **Vocabulary** | The set of all unique tokens across the entire corpus |
| **Bag of words** | Representing a document as a multiset of tokens, ignoring order |
| **Document frequency** | How many documents in the corpus contain a given term |
| **Corpus size** | Total number of documents — affects IDF scores (more docs = more signal) |

---

## How Size and Composition Affect NLP

| Factor | Effect |
|---|---|
| Small corpus | IDF scores are noisy — rare words may appear rare just by chance |
| Large corpus | More reliable IDF; common words are clearly identified |
| Domain-specific corpus | Specialized terms get high TF-IDF because they're rare elsewhere |
| General corpus | Better coverage but noisier for specialized tasks |

This is why a search engine corpus built by crawling general web pages behaves differently from one built from medical journals.

---

## Corpus in This Project

In the [[Notes/Projects/python/mini-search-engine/00-MiniSearchEngine|Mini Search Engine]]:

- **Phase 2** — the corpus is local text files (documents you already have)
- **Phase 3** — the corpus is built by the [[Notes/Areas/WebCrawling/WebCrawling|crawler]] fetching pages from the web
- **Phase 4** — the [[Notes/Areas/NLP/Concepts/Inverted_Index|inverted index]] is built over the corpus
- **Phase 5** — [[Notes/Areas/NLP/Concepts/TF_IDF|TF-IDF]] scores are computed across the corpus to rank results

---

## Related Notes

- [[NLP]] — hub
- [[Notes/Areas/NLP/Concepts/TF_IDF|TF-IDF]] — uses the corpus to compute IDF (how rare a term is across all documents)
- [[Notes/Areas/NLP/Concepts/Inverted_Index|Inverted Index]] — maps terms to their locations across the corpus
- [[Notes/Areas/NLP/Concepts/Tokenization|Tokenization]] — first step in processing each document in the corpus
- [[Notes/Areas/WebCrawling/WebCrawling|Web Crawling]] — how the corpus is built from the web

## Open Questions

- How do you decide what documents to include in a corpus?
- How does corpus size affect the quality of TF-IDF scores?
- What is a balanced corpus vs a biased corpus?
