---
tags: [python, nlp, text-processing]
type: implementation
area: learning
---

# Normalizing Textual Data in Python

See [[Concepts/Normalizing_Text]] for the full pipeline, when to apply each step, and tradeoffs.

## 1. Case Conversion (Lowercase)

```python
string = "Python 3.0, released in 2008, was a major revision of the language."

lower_string = string.lower()
print(lower_string)
'''
Output:
python 3.0, released in 2008, was a major revision of the language.
'''
```

## 2. Removing Numbers

Use `re.sub()` with `\d+` to strip numeric characters when they carry no semantic meaning.

```python
import re

lower_string = "python 3.0, released in 2008, was a major revision."

no_number_string = re.sub(r'\d+', '', lower_string)
print(no_number_string)
'''
Output:
python ., released in , was a major revision.
'''
```

## 3. Removing Punctuation

Use `[^\w\s]` to remove everything that isn't a word character or whitespace.

```python
import re

no_number_string = "python  released in  was a major revision of the language"

no_punc_string = re.sub(r'[^\w\s]', '', no_number_string)
print(no_punc_string)
'''
Output:
python  released in  was a major revision of the language
'''
```

## 4. Removing White Spaces

`strip()` removes leading and trailing whitespace. For internal extra spaces use `re.sub(r'\s+', ' ', ...)`.

```python
text = "   too   many   spaces   "

clean = text.strip()
print(repr(clean))
'''
Output:
'too   many   spaces'
'''
```

## 5. Removing Stop Words

```python
import re
import nltk
from nltk.corpus import stopwords

nltk.download('stopwords')

string = "       Python 3.0, released in 2008, was a major revision of the language that is not completely backward compatible."

# Pipeline
lower_string     = string.lower()
no_number_string = re.sub(r'\d+', '', lower_string)
no_punc_string   = re.sub(r'[^\w\s]', '', no_number_string)
no_wspace_string = no_punc_string.strip()

stop_words = set(stopwords.words('english'))
lst_string = no_wspace_string.split()
filtered   = [w for w in lst_string if w not in stop_words]

result = ' '.join(filtered)
print(result)
'''
Output:
python released major revision language completely backward compatible
'''
```

![[Assets/Pasted image 20260307210935.png]]

## Related Notes

- [[Concepts/Normalizing_Text]] — pipeline steps, when to skip each, tradeoffs
- [[NLP]] — hub
- [[Tokenization_in_Python]]
- [[Removing_Stop_Words_Python]]
- [[Stemming_Words_NLTK]]

## Python Tools Used

- [[Notes/Resources/Python/String_Methods|String Methods]] — `.lower()`, `.strip()` for case and whitespace
- [[Notes/Resources/Python/Regex|Regex]] — `re.sub()` for removing numbers and punctuation

## Open Questions

- When should you keep numbers instead of removing them?
- How does normalization differ for languages other than English?
- What's the difference between normalization and lemmatization?
