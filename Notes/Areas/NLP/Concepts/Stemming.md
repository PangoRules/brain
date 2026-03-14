---
tags: [nlp, concepts, text-processing]
type: concept
area: learning
---

# Stemming

**Stemming** is the process of reducing a word to its base or root form by stripping suffixes. The goal is to map inflected forms of a word to a common stem so they can be treated as equivalent.

Example: "running", "runner", "runs" → "run"

> The result is **not required to be a real dictionary word** — only a consistent root. For a linguistically accurate approach, see [[Stemming#Stemming vs. Lemmatization|Lemmatization]].

## Key Problems

- **Overstemming:** Two different words incorrectly collapse to the same stem
  - "university" and "universe" → "univers" (false equivalence)
- **Understemming:** Related words fail to reduce to the same stem
  - "data" and "datum" remain different (missed equivalence)

## Stemmer Comparison

| Stemmer | Aggressiveness | Accuracy | Languages | Notes |
|---|---|---|---|---|
| Porter | Moderate | Good | English only | Most popular, well-studied |
| Lancaster | High | Lower | English only | Most aggressive, fastest, over-stems frequently |
| Snowball | Moderate | Better than Porter | 15+ languages | Improved Porter, preferred for multilingual work |
| Lemmatizer | N/A (not a stemmer) | Highest | Language-dependent | Uses vocabulary; always produces real words |

## Stemming vs. Lemmatization

| Property | Stemming | Lemmatization |
|---|---|---|
| Output | May not be a real word | Always a real dictionary word |
| Speed | Fast (rule-based suffix stripping) | Slower (requires vocabulary lookup) |
| Accuracy | Lower | Higher |
| Context-aware | No | Can be (with POS tagging) |
| Use case | Search, IR, simple classification | Tasks requiring real word forms |

## When to Use Stemming

- Search engines — match "running" queries to "run" documents
- Text classification and clustering where exact word form doesn't matter
- Information retrieval systems with large vocabularies
- When speed matters more than linguistic accuracy

## Related Notes

- [[NLP]] — hub
- [[Python/Stemming_Words_NLTK]] — NLTK implementation
- [[Tokenization]] — tokenize before stemming
- [[Stop_Words]] — typically remove stop words before stemming
