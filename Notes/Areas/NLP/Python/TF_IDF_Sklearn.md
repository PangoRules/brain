---
tags: [python, nlp, text-processing, machine-learning, feature-extraction, scikit-learn]
type: implementation
area: learning
---

# TF-IDF with scikit-learn

See [[Concepts/TF_IDF]] for the formulas, worked numeric example, and when to use TF-IDF.

## TfidfVectorizer

```python
from sklearn.feature_extraction.text import TfidfVectorizer

documents = [
    'Geeks for geeks',
    'Geeks are smart',
    'Python is great'
]

tfidf = TfidfVectorizer()
result = tfidf.fit_transform(documents)

# IDF values per term
print("IDF values:")
for word, idf in zip(tfidf.get_feature_names_out(), tfidf.idf_):
    print(f"  {word:10} : {idf:.4f}")

# TF-IDF matrix (rows = documents, columns = terms)
print("\nTF-IDF matrix:")
print(result.toarray())
'''
Output (example):
IDF values:
  are        : 1.6931
  for        : 1.6931
  geeks      : 1.2877
  great      : 1.6931
  is         : 1.6931
  python     : 1.6931
  smart      : 1.6931

TF-IDF matrix:
[[0.     0.5844 0.7317 0.     0.     0.     0.    ]
 [0.6227 0.     0.4736 0.     0.     0.     0.6227]
 [0.     0.     0.     0.5    0.5    0.5    0.    ]]
'''
```

## Related Notes

- [[Concepts/TF_IDF]] — formulas, worked example, applications, limitations
- [[NLP]] — hub
- [[Tokenization_in_Python]]
- [[Removing_Stop_Words_Python]]
- [[Stemming_Words_NLTK]]
- [[Inverted_Index_Python]]

## Python Tools Used

- [[Notes/Resources/Python/Counter|Counter]] — counting term frequencies manually

## Open Questions

- How does `TfidfVectorizer`'s smoothing differ from the raw formula?
- How does TF-IDF compare to BM25 in search ranking?
- When should you use TF-IDF vs. word embeddings?
