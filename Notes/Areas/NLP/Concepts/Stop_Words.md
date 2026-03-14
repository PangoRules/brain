---
tags: [nlp, concepts, text-processing]
type: concept
area: learning
---

# Stop Words

**Stop words** are high-frequency words that carry minimal semantic value on their own — words like "the", "a", "is", "on", "and". Filtering them reduces noise and vocabulary size in many NLP tasks.

## Definition

Stop words are typically:
- Function words (articles, prepositions, conjunctions, pronouns)
- Words so common they appear in nearly every document
- Words that don't distinguish one document from another

There is no universal stop word list — what counts as a stop word depends on the task and domain.

## When to Remove vs. Preserve

| Remove stop words when... | Preserve stop words when... |
|---|---|
| Text classification | Machine translation (grammar matters) |
| Topic modeling / clustering | Text summarization |
| Information retrieval / search | Question-answering systems |
| Keyword extraction | Sentiment requiring "not", "never" |
| TF-IDF feature vectors | Sequence-to-sequence tasks |

> **Caution:** Words like "not" and "never" are technically stop words but invert sentiment. Always evaluate whether removal hurts your specific task.

## Domain-Specific Considerations

- Medical text: "patient", "disease" may be stop words if they appear in every document
- Legal text: "shall", "pursuant" are common but may be meaningful
- Code: "import", "return" appear frequently but carry semantic meaning
- Custom stop word lists are often needed for specialized corpora

## Comparison: NLTK vs. Other Sources

| Source | Size | Notes |
|---|---|---|
| NLTK (English) | 179 words | Good general-purpose baseline |
| spaCy | ~326 words | More comprehensive |
| Custom domain list | Varies | Best for specialized corpora |
| None | — | Required for sentiment, translation |

## Related Notes

- [[NLP]] — hub
- [[Python/Removing_Stop_Words_Python]] — NLTK implementation
- [[Tokenization]] — must tokenize before filtering stop words
- [[TF_IDF]] — TF-IDF naturally down-weights common words as an alternative
