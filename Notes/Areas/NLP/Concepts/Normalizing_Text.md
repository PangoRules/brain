---
tags: [nlp, concepts, text-processing]
type: concept
area: learning
---

# Text Normalization

**Text normalization** is the process of transforming raw text into a consistent, canonical form before processing. Since there is no universal procedure, normalization steps are chosen based on the task.

> Normalizing text before storing or processing allows for separation of concerns — downstream operations can assume the input is already in a consistent form.

## Standard Pipeline Steps

| Step | Purpose | Example |
|---|---|---|
| 1. Lowercasing | Make comparisons case-insensitive | "Python" → "python" |
| 2. Remove/replace numbers | Strip digits that carry no semantic meaning | "2008" → "" |
| 3. Remove punctuation | Strip non-word characters | "Hello!" → "Hello" |
| 4. Strip whitespace | Remove leading, trailing, and extra internal spaces | "  hi  " → "hi" |
| 5. Remove stop words | Drop high-frequency low-meaning words | "is a the" → "" |
| 6. Stem or lemmatize | Reduce words to root form | "running" → "run" |

Steps 5 and 6 are optional and task-dependent — always evaluate their impact before applying.

## When to Apply Each Step

| Step | Skip when... |
|---|---|
| Lowercasing | Case carries meaning (e.g., named entity recognition: "Apple" vs. "apple") |
| Remove numbers | Numbers are semantically meaningful (dates, prices, measurements) |
| Remove punctuation | Punctuation matters (e.g., sentiment: "great!" vs. "great") |
| Remove stop words | Grammar or negation matters (translation, QA, "not happy") |
| Stemming | You need real dictionary words (lemmatize instead) |

## Normalization vs. Related Concepts

| Concept | Scope | Goal |
|---|---|---|
| Normalization | Whole pipeline | Canonical, consistent text |
| Tokenization | Splitting step | Break into tokens |
| Stop word removal | Filtering step | Reduce noise |
| Stemming/Lemmatization | Word-level | Reduce inflections |

## Related Notes

- [[NLP]] — hub
- [[Python/Normalizing_Textual_Data_in_Python]] — Python implementation
- [[Tokenization]] — tokenization is part of the normalization pipeline
- [[Stop_Words]] — step 5 of normalization
- [[Stemming]] — step 6 of normalization
