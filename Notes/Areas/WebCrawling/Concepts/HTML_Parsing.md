---
tags: [web-crawling, html, parsing]
type: concept
area: learning
---

# HTML Parsing

HTML parsing is the process of reading a raw HTML string and converting it into a navigable tree of objects. A crawler parses HTML to extract links (for the frontier) and page content (for the indexer). BeautifulSoup is the standard Python library for this.

---

## Key Vocabulary

| Term | Meaning |
|---|---|
| **Parser** | The backend that reads raw HTML and builds the tree (`html.parser`, `lxml`, `html5lib`) |
| **Tag** | A `bs4.element.Tag` — represents one HTML element like `<div>`, `<a>`, `<p>` |
| **NavigableString** | The text content inside a tag |
| **`.string`** | The single text string of a tag — `None` if it has child tags |
| **`.get_text()`** | All text inside a tag concatenated — safer than `.string` |
| **`.attrs`** | Dict of the tag's HTML attributes: `{"href": "/page", "class": ["product"]}` |
| **CSS selector** | A pattern like `div.product > a.title` passed to `.select()` |
| **DOM** | Document Object Model — the tree structure of an HTML document |

---

## The Three Parsers

| Parser | Speed | Leniency | Notes |
|---|---|---|---|
| `html.parser` | Medium | Medium | Built into Python — no install needed |
| `lxml` | Fast | High | `pip install lxml` — best for production |
| `html5lib` | Slow | Very high | Parses like a browser — use for broken HTML |

**Rule of thumb:** Start with `html.parser`. Switch to `lxml` once you need speed.

---

## Finding Elements

Three approaches, from simple to flexible:

**1. `find()` / `find_all()`** — search by tag name and attributes
```python
soup.find("a")                          # first <a> tag
soup.find_all("a", href=True)           # all <a> tags that have href
soup.find("div", class_="product")     # by CSS class
soup.find(id="title")                   # by id
```

**2. `.select()` / `.select_one()`** — CSS selector syntax
```python
soup.select("div.product > a")          # direct child
soup.select('a[href^="http"]')          # href starting with "http"
```

**3. Tree navigation** — `.parent`, `.children`, `.next_sibling`

---

## class is Always a List

BeautifulSoup returns `class` as a list, not a string:
```python
tag["class"]           # ['product', 'featured']  ← list
"featured" in tag.get("class", [])     # correct membership check
```

---

## `.string` vs `.get_text()`

- `.string` returns `None` if a tag has multiple child elements — unreliable
- `.get_text(strip=True)` always works, concatenates all nested text

---

## Related Notes

- [[WebCrawling]] — hub
- [[Python/HTML_Parsing_BeautifulSoup]] — implementation
- [[Concepts/Extracting_Links]] — the main use of HTML parsing in a crawler

## Open Questions

- How do you parse pages that render content with JavaScript (dynamic pages)?
- When does `html5lib` outperform `lxml`?
- What is the XPath alternative to CSS selectors?
