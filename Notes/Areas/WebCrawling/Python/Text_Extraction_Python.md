---
tags: [python, web-crawling, text-extraction, beautifulsoup]
type: implementation
area: learning
---

# Text Extraction in Python

See [[Concepts/Text_Extraction]] for theory on what to extract and why.
See [[Python/HTML_Parsing_BeautifulSoup]] for BeautifulSoup mechanics — it already covers `get_page_text()`.

This note focuses on **indexer-ready text extraction**: cleaning, weighting headings, and producing output fit for tokenization.

---

## Basic Visible Text Extraction

```python
from bs4 import BeautifulSoup

NOISE_TAGS = ["script", "style", "noscript", "header", "footer", "nav"]


def extract_text(html: str) -> str:
    soup = BeautifulSoup(html, "html.parser")
    for tag in soup(NOISE_TAGS):
        tag.decompose()
    return soup.get_text(separator=" ", strip=True)
```

---

## Weighted Extraction (Headings Boosted)

To give heading terms higher importance during indexing, repeat them — this increases their term frequency naturally.

```python
def extract_weighted_text(html: str) -> str:
    soup = BeautifulSoup(html, "html.parser")

    # Remove noise
    for tag in soup(NOISE_TAGS):
        tag.decompose()

    parts = []

    # Headings repeated for weight (h1 × 3, h2 × 2, h3 × 1 extra)
    for h1 in soup.find_all("h1"):
        text = h1.get_text(strip=True)
        parts.extend([text] * 3)   # weight × 3

    for h2 in soup.find_all("h2"):
        text = h2.get_text(strip=True)
        parts.extend([text] * 2)

    for h3 in soup.find_all("h3"):
        parts.append(h3.get_text(strip=True))

    # Body text
    parts.append(soup.get_text(separator=" ", strip=True))

    return " ".join(parts)
```

---

## Full Example

```python
html = """
<html>
<head><script>alert('noise')</script></head>
<body>
  <nav>Home | About</nav>
  <h1>Python Web Crawling</h1>
  <h2>Getting Started</h2>
  <p>Web crawling is the process of automatically fetching web pages.</p>
  <footer>Copyright 2026</footer>
</body>
</html>
"""

print(extract_text(html))
# 'Python Web Crawling Getting Started Web crawling is the process of automatically fetching web pages.'

print(extract_weighted_text(html))
# Headings appear multiple times, then body text
```

---

## What Comes Next

The extracted text feeds directly into the tokenization + term frequency stage:

```python
from collections import Counter
import re

def compute_term_frequencies(text: str) -> dict[str, int]:
    tokens = re.findall(r"[a-z]+", text.lower())
    return dict(Counter(tokens))
```

---

## Related Notes

- [[Concepts/Text_Extraction]] — theory, what to strip, heading weights
- [[Python/HTML_Parsing_BeautifulSoup]] — BeautifulSoup API reference
- [[Python/Document_Metadata_Python]] — extracting title/headings separately
- [[WebCrawling/Concepts/Indexing_Pipeline]] — full pipeline context
- [[NLP/Python/Tokenization_in_Python]] — next step after extraction
