---
tags: [nlp, concepts, text-processing]
type: concept
area: learning
---

# Tokenization

**Tokenization** is the process of breaking text into smaller units called **tokens**. It is the first step in most NLP pipelines — text must be tokenized before any further processing can occur.

## Token Types

| Type | Description | Example |
|---|---|---|
| Word tokens | Split by whitespace/punctuation | "Hello world" → ["Hello", "world"] |
| Sentence tokens | Split text into sentences | Paragraph → list of sentences |
| Subword tokens | Split words into meaningful subunits | "unhappiness" → ["un", "happiness"] |
| Character tokens | Each character is a token | "Hi" → ["H", "i"] |
| BPE tokens | Byte Pair Encoding — learned subword splits | Used by GPT, BERT |

## When to Use Each Approach

| Approach | Best For |
|---|---|
| Word tokenization | Simple classification, bag-of-words models |
| Sentence tokenization | Summarization, translation, sentence-level tasks |
| Subword/BPE | Neural models (GPT, BERT) — handles rare/unknown words |
| Character-level | Languages without whitespace (Chinese, Japanese) |

## Key Tradeoffs

- **Simple split (`str.split()`):** Fast, but mishandles punctuation and contractions
- **Rule-based tokenizers (NLTK):** Better edge-case handling, still language-dependent
- **Learned tokenizers (BPE/WordPiece):** Handle any vocabulary, required for transformer models
- Tokenization is language-specific — English rules break on Chinese, Arabic, etc.

## Related Notes

- [[NLP]] — hub
- [[Python/Tokenization_in_Python]] — NLTK implementation
- [[Stop_Words]]
- [[Stemming]]
