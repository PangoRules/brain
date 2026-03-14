---
tags: [python, nlp, text-processing]
type: implementation
area: learning
---

# Tokenization in Python

See [[Concepts/Tokenization]] for theory, token types, and tradeoffs.

## Split Method

`split()` is the most basic way to tokenize in Python. It splits a string into a list based on a specified delimiter (whitespace by default).

> **When to use:** Quick tokenization where punctuation handling is not important.

```python
text = "Geek and Knowledge are best friends"
tokens = text.split()
print(tokens)
'''
Output:
['Geek', 'and', 'Knowledge', 'are', 'best', 'friends']
'''
```

## NLTK's word_tokenize()

[NLTK (Natural Language Toolkit)](https://www.nltk.org/) provides `word_tokenize()`, which tokenizes text into words **and** punctuation marks as separate tokens. It handles contractions and edge cases that `split()` misses.

> **When to use:** When punctuation matters or you need robust, language-aware tokenization.

```python
import nltk
nltk.download('punkt')
from nltk.tokenize import word_tokenize

text = "Don't stop learning NLP, it's worth it!"
tokens = word_tokenize(text)
print(tokens)
'''
Output:
['Do', "n't", 'stop', 'learning', 'NLP', ',', 'it', "'s", 'worth', 'it', '!']
'''
```

## NLTK's sent_tokenize()

`sent_tokenize()` splits text into a list of **sentences** instead of words. Useful when you need to process documents sentence by sentence.

```python
from nltk.tokenize import sent_tokenize

text = "Hello there. How are you? I am learning NLP."
sentences = sent_tokenize(text)
print(sentences)
'''
Output:
['Hello there.', 'How are you?', 'I am learning NLP.']
'''
```

## Related Notes

- [[Concepts/Tokenization]] — theory and tradeoffs
- [[NLP]] — hub
- [[Removing_Stop_Words_Python]]
- [[Stemming_Words_NLTK]]

## Python Tools Used

- [[Notes/Resources/Python/String_Methods|String Methods]] — `str.split()` for basic tokenization
- [[Notes/Resources/Python/Regex|Regex]] — `re.split()` for pattern-based splitting

## Open Questions

- How does punctuation affect downstream tasks like sentiment analysis?
- What happens with emojis or non-ASCII characters?
- How do subword tokenizers (BPE, WordPiece) differ from word-level tokenization?
- How do large-scale systems tokenize efficiently at scale?
