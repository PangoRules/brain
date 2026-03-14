---
tags: [python, nlp, text-processing, nltk]
type: implementation
area: learning
---

# Removing Stop Words with NLTK in Python

See [[Concepts/Stop_Words]] for theory, when to remove vs. preserve, and domain-specific considerations.

## Removing Stop Words with NLTK

```python
import nltk
from nltk.corpus import stopwords
from nltk.tokenize import word_tokenize

nltk.download('stopwords')
nltk.download('punkt')

text = "This is a sample sentence showing how stopword removal works."

stop_words = set(stopwords.words('english'))
tokens = word_tokenize(text.lower())
filtered = [word for word in tokens if word not in stop_words]

print("Original tokens:", tokens)
print("Filtered tokens:", filtered)
'''
Output:
Original tokens: ['this', 'is', 'a', 'sample', 'sentence', 'showing', 'how', 'stopword', 'removal', 'works', '.']
Filtered tokens: ['sample', 'sentence', 'showing', 'stopword', 'removal', 'works', '.']
'''
```

## Inspecting NLTK's Stop Words

NLTK ships with 179 English stop words by default.

```python
from nltk.corpus import stopwords

sw = stopwords.words('english')
print(f"Total stop words: {len(sw)}")
print(sw[:20])
'''
Output:
Total stop words: 179
['i', 'me', 'my', 'myself', 'we', 'our', 'ours', 'ourselves', 'you', "you're", ...]
'''
```

## Adding Custom Stop Words

You can extend the default list with domain-specific terms.

```python
from nltk.corpus import stopwords
from nltk.tokenize import word_tokenize

custom_stops = {"geeksforgeeks", "python", "example"}
stop_words = set(stopwords.words('english')).union(custom_stops)

text = "Python is a great language, geeksforgeeks provides great examples."
tokens = word_tokenize(text.lower())
filtered = [w for w in tokens if w not in stop_words]

print(filtered)
'''
Output:
['great', 'language', ',', 'provides', 'great', 'examples', '.']
'''
```

## Related Notes

- [[Concepts/Stop_Words]] — theory and when to remove vs. preserve
- [[NLP]] — hub
- [[Tokenization_in_Python]]
- [[Stemming_Words_NLTK]]

## Open Questions

- How do deep learning models (BERT, GPT) handle stop words differently?
- When does removing "not" break sentiment analysis?
- Are there language-specific stop word lists for non-English text in NLTK?
