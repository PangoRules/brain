---
tags: [python, web-crawling, html, beautifulsoup, bs4]
type: implementation
area: learning
---

# HTML Parsing with BeautifulSoup

See [[Concepts/HTML_Parsing]] for theory, parser comparison, and vocabulary.

```bash
pip install beautifulsoup4 lxml
```

---

## Parsing HTML

```python
from bs4 import BeautifulSoup

html = """
<html><body>
  <h1 id="title">Hello</h1>
  <div class="product featured">
    <a href="/item/1" class="name">Widget A</a>
    <span class="price">$9.99</span>
  </div>
</body></html>
"""

# Always specify a parser — omitting it causes a warning
soup = BeautifulSoup(html, "html.parser")   # built-in, no install
# soup = BeautifulSoup(html, "lxml")        # faster, pip install lxml
```

---

## `find()` and `find_all()`

```python
# find() — first match or None
soup.find("h1")                            # first <h1>
soup.find("div", class_="product")        # by class
soup.find(id="title")                      # by id
soup.find("a", href=True)                  # must have href attribute

# find_all() — list of all matches (empty list if none)
soup.find_all("a")                         # all <a> tags
soup.find_all("a", href=True)              # all <a> with href
soup.find_all(["h1", "h2", "h3"])          # multiple tag names
soup.find_all("div", limit=5)             # first 5 only
```

---

## CSS Selectors with `select()` / `select_one()`

```python
soup.select_one("h1#title")               # id selector
soup.select("div.product")                # class selector
soup.select("div.product > a")            # direct child
soup.select("div.product a")              # any descendant
soup.select('a[href^="http"]')            # href starts with "http"
soup.select('a[href$=".pdf"]')            # href ends with ".pdf"
soup.select('a[href*="example"]')         # href contains "example"
```

---

## Extracting Text and Attributes

```python
tag = soup.find("a", class_="name")

# Text
tag.string          # 'Widget A' — only if tag has ONE text node, else None
tag.get_text()      # 'Widget A' — always works, concatenates all nested text
tag.get_text(strip=True)                     # strip whitespace
tag.get_text(separator=" ", strip=True)      # join with separator

# Attributes
tag["href"]                 # '/item/1' — raises KeyError if missing
tag.get("href")             # '/item/1' — returns None if missing (safer)
tag.get("href", "")         # with a default value
tag.attrs                   # {'href': '/item/1', 'class': ['name']}

# class is always a LIST
tag["class"]                # ['name']
"featured" in tag.get("class", [])   # correct membership check
```

---

## Navigating the Tree

```python
tag.parent                      # the containing tag
list(tag.children)              # direct children (generator → list)
list(tag.next_siblings)         # tags after it at the same level
tag.find_next("span")           # next <span> anywhere in the document
```

---

## Practical: Extract All Text from a Page

```python
def get_page_text(html: str) -> str:
    soup = BeautifulSoup(html, "html.parser")
    # Remove script and style elements
    for tag in soup(["script", "style", "noscript"]):
        tag.decompose()
    return soup.get_text(separator=" ", strip=True)
```

---

## Gotchas

- **`find()` returns `None`** on no match — calling `.text` on `None` raises `AttributeError`. Always guard: `if tag: ...`
- **`class_` not `class`** in `find()` — `class` is a Python reserved word
- **`class` is a list** — `tag["class"]` returns `["product", "featured"]`
- **`lxml` is much faster** than `html.parser` for large pages — install it for production
- **`html5lib`** is most lenient (handles broken HTML like a browser) but slow

---

## Related Notes

- [[Concepts/HTML_Parsing]] — theory, parser comparison
- [[WebCrawling]] — hub
- [[Python/Extracting_Links_Python]] — the main use of BS4 in a crawler
- [[Python/Crawl_Depth_Python]] — full crawler that uses BeautifulSoup
