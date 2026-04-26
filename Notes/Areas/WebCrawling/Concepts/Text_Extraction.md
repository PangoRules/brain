---
tags: [web-crawling, indexing, text-processing]
type: concept
area: learning
---

# Text Extraction

Text extraction is the process of pulling **visible, meaningful text** from a raw HTML page for indexing. The goal is to give the search engine clean text — not HTML tags, scripts, or styles.

---

## Why Not Just Use the Raw HTML?

Raw HTML contains noise that pollutes the index:
- `<script>` — JavaScript code, not content
- `<style>` — CSS rules, not content
- HTML tags themselves — `<div>`, `<p>`, `<span>` etc.
- Boilerplate — navbars, footers, cookie banners

Indexing raw HTML would cause irrelevant terms (like `function`, `display`, `flex`) to appear in search results.

---

## What to Extract

| Element | Extract? | Why |
|---|---|---|
| `<p>`, `<li>`, `<td>` | Yes | Main content |
| `<h1>`–`<h6>` | Yes (weighted) | High-signal terms |
| `<title>` | Yes | Most important signal |
| `<meta name="description">` | Yes | Summary signal |
| `<a>` text | Yes | Anchor text |
| `<script>` | No | Code, not content |
| `<style>` | No | CSS, not content |
| `<nav>`, `<footer>` | Optional | Often boilerplate |
| `<noscript>` | No | Usually empty fallback |

---

## The Extraction Strategy

1. Parse the HTML with BeautifulSoup
2. **Decompose** (remove) noise tags: `script`, `style`, `noscript`
3. Call `.get_text(separator=" ", strip=True)` on the remaining tree
4. Optionally normalize: lowercase, strip extra whitespace

---

## Headings as Weighted Signals

Headings (`<h1>`–`<h6>`) carry more meaning than body text. In a ranking system, terms that appear in headings score higher. Extract them separately to apply weights.

---

## Related Notes

- [[WebCrawling]] — hub
- [[Python/Text_Extraction_Python]] — implementation
- [[Python/HTML_Parsing_BeautifulSoup]] — BeautifulSoup mechanics (see `get_page_text()`)
- [[Concepts/Document_Metadata]] — extracting title, headings, URL
- [[Concepts/Indexing_Pipeline]] — where extraction fits in the pipeline
- [[NLP/Concepts/Inverted_Index]] — what the extracted text feeds into

## Open Questions

- How do you handle pages with most content loaded by JavaScript (SPAs)?
- When is it worth extracting `<alt>` text from images?
- How do you detect and skip boilerplate blocks (navbars, footers) automatically?
