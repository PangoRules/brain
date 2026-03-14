---
tags: [nlp, concepts, text-processing, machine-learning, feature-extraction]
type: concept
area: learning
---

# TF-IDF (Term Frequency–Inverse Document Frequency)

**TF-IDF** is a statistical technique that measures how important a word is to a specific document within a collection ([[Corpus]]). It rewards words that are frequent in a document but rare across the corpus — automatically down-weighting common filler words.

## The Formulas

### Term Frequency (TF)

How often a term appears in a document, normalized by document length:

```
TF(term, doc) = (count of term in doc) / (total terms in doc)
```

### Inverse Document Frequency (IDF)

Penalizes terms that appear in many documents (common words get low scores):

```
IDF(term) = log(total documents / documents containing the term)
```

### TF-IDF Score

```
TF-IDF(term, doc) = TF(term, doc) × IDF(term)
```

> A high TF-IDF score means the word is important to that specific document but rare across the whole corpus.

## Worked Numeric Example

Given 3 documents and the word "cat":
- Doc1: "The cat sat on the mat" (6 words, "cat" appears 1 time)
- Doc2: "The dog sat on the log" (no "cat")
- Doc3: "The cat and the dog" (no second "cat", 1 appearance)

```
TF(cat, Doc1)  = 1/6 ≈ 0.167
IDF(cat)       = log(3/2) ≈ 0.176
TF-IDF         = 0.167 × 0.176 ≈ 0.029
```

Words like "the" appear in all 3 documents → IDF = log(3/3) = 0 → TF-IDF = 0.

## Applications

| Use Case | Why TF-IDF Helps |
|---|---|
| Search engines | Rank documents by query relevance |
| Text classification | Convert text to feature vectors for ML |
| Keyword extraction | High TF-IDF words are the "key" terms |
| Spam detection | Rare trigger words get high scores |
| Document clustering | Compare documents as numeric vectors |
| Recommendation systems | Find similar content by term overlap |

## Limitations

- Ignores word order and context (bag-of-words assumption)
- Can't capture synonyms ("car" ≠ "automobile")
- Doesn't handle new vocabulary at inference time (fixed vocabulary)
- Replaced by word embeddings (Word2Vec, GloVe) and transformers (BERT) for semantic tasks

## Open Questions

- How does TF-IDF compare to BM25 in search ranking?
- When should you use TF-IDF vs. word embeddings?
- How does `TfidfVectorizer`'s smoothing differ from the raw formula?

## Related Notes

- [[NLP]] — hub
- [[Corpus]] — the collection of documents TF-IDF is computed across
- [[Python/TF_IDF_Sklearn]] — scikit-learn implementation
- [[Inverted_Index]] — TF-IDF scoring plugs into search indexes for ranking
- [[Stop_Words]] — stop words naturally get low TF-IDF scores (but removal still common)
- [[Tokenization]] — text must be tokenized before computing TF-IDF
