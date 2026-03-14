---
tags: [python, web-crawling, links, urllib]
type: implementation
area: learning
---

# Extracting Links in Python

See [[Concepts/Extracting_Links]] for theory on link types, `urljoin` behavior, and domain filtering.

---

## Core: Normalize and Filter a Single Link

```python
from urllib.parse import urljoin, urlparse, urldefrag

def normalize_link(base_url: str, href: str) -> str | None:
    """
    Resolve href against base_url.
    Returns an absolute URL string, or None if the link should be discarded.
    """
    href = href.strip()

    # Discard non-URL hrefs immediately
    if not href or href.startswith(("#", "mailto:", "javascript:", "data:", "tel:")):
        return None

    # Resolve relative → absolute
    absolute = urljoin(base_url, href)

    # Strip fragment (#section) — same page regardless of anchor
    absolute, _ = urldefrag(absolute)

    # Keep only http/https
    if urlparse(absolute).scheme not in ("http", "https"):
        return None

    return absolute
```

---

## Extract All Unique Links from a Page

```python
from bs4 import BeautifulSoup

def extract_links(html: str, base_url: str) -> list[str]:
    """Return deduplicated list of valid absolute URLs from page HTML."""
    soup = BeautifulSoup(html, "html.parser")
    seen = set()
    links = []
    for tag in soup.find_all("a", href=True):   # href=True skips <a> with no href
        url = normalize_link(base_url, tag["href"])
        if url and url not in seen:
            seen.add(url)
            links.append(url)
    return links
```

---

## Filter to Same Domain Only

```python
from urllib.parse import urlparse

def same_domain(url: str, base_url: str) -> bool:
    return urlparse(url).netloc == urlparse(base_url).netloc

# Internal links only
internal = [u for u in extract_links(html, base_url) if same_domain(u, base_url)]
```

---

## Handle the `<base>` Tag

Some pages declare a different base URL in `<head>`. Check for it:

```python
def get_effective_base(soup: BeautifulSoup, page_url: str) -> str:
    base_tag = soup.find("base", href=True)
    if base_tag:
        return urljoin(page_url, base_tag["href"])
    return page_url
```

---

## Full Example (fetch + extract)

```python
import requests
from bs4 import BeautifulSoup
from urllib.parse import urljoin, urlparse, urldefrag

def get_links(page_url: str) -> list[str]:
    response = requests.get(
        page_url,
        headers={"User-Agent": "MyCrawler/1.0"},
        timeout=(5, 30),
    )
    response.raise_for_status()
    soup = BeautifulSoup(response.text, "html.parser")

    base = get_effective_base(soup, page_url)
    return [u for u in extract_links(response.text, base) if same_domain(u, page_url)]
```

---

## Gotchas

- **`href=True`** in `find_all` skips `<a>` tags without `href` (e.g. `<a name="anchor">`)
- **`urljoin` always returns something** — even `mailto:` hrefs. Always check scheme of the result
- **Fragments** (`#`) resolve to the same page URL — strip with `urldefrag` to avoid duplicates
- **`<base>` tag** can redirect all relative links to a different domain — check for it
- **HTML entity decoding**: BeautifulSoup decodes `&amp;` → `&` in attributes automatically — don't double-decode

---

## Related Notes

- [[Concepts/Extracting_Links]] — theory: link types, urljoin behavior
- [[WebCrawling]] — hub
- [[Python/HTML_Parsing_BeautifulSoup]] — the BS4 patterns used here
- [[Python/URL_Normalization_Python]] — normalizing extracted links before enqueuing
- [[Python/URL_Frontier_Python]] — where the extracted links go
