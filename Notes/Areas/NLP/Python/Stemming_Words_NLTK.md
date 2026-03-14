---
tags: [python, nlp, text-processing, nltk, stemming]
type: implementation
area: learning
---

# Stemming Words with NLTK in Python

See [[Concepts/Stemming]] for theory, stemmer comparison, and stemming vs. lemmatization tradeoffs.

## PorterStemmer — Most Common Stemmer

NLTK's `PorterStemmer` implements the Porter algorithm, the most widely used stemming algorithm for English.

```python
from nltk.stem import PorterStemmer
from nltk.tokenize import word_tokenize

ps = PorterStemmer()

words = ["program", "programs", "programmer", "programming", "programmers"]

for w in words:
    print(f"{w:15} -> {ps.stem(w)}")
'''
Output:
program         -> program
programs        -> program
programmer      -> program
programming     -> program
programmers     -> program
'''
```

## Stemming a Full Sentence

```python
from nltk.stem import PorterStemmer
from nltk.tokenize import word_tokenize

ps = PorterStemmer()

sentence = "Programmers program with programming languages"
tokens = word_tokenize(sentence)
stemmed = [ps.stem(token) for token in tokens]

print("Original:", tokens)
print("Stemmed: ", stemmed)
'''
Output:
Original: ['Programmers', 'program', 'with', 'programming', 'languages']
Stemmed:  ['programm', 'program', 'with', 'program', 'languag']
'''
```

## Other Stemmers in NLTK

```python
from nltk.stem import PorterStemmer, LancasterStemmer, SnowballStemmer

ls = LancasterStemmer()
ss = SnowballStemmer("english")

word = "generously"
print(f"Porter:    {PorterStemmer().stem(word)}")   # -> generous
print(f"Lancaster: {ls.stem(word)}")                # -> gen
print(f"Snowball:  {ss.stem(word)}")                # -> generous
```

## Related Notes

- [[Concepts/Stemming]] — theory, stemmer comparison table, stemming vs. lemmatization
- [[NLP]] — hub
- [[Tokenization_in_Python]]
- [[Removing_Stop_Words_Python]]
- [[TF_IDF_Sklearn]]

## Open Questions

- When should you use lemmatization instead of stemming?
- How does stemming interact with named entities (e.g., "Apple" vs. "apple")?
- Does stemming help or hurt neural NLP models like BERT?
