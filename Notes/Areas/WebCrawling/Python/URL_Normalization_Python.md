---
tags: [python, web-crawling, url, urllib]
type: implementation
area: learning
---

# URL Normalization in Python

See [[Concepts/URL_Normalization]] for theory on why each step matters.

---

## Complete `normalize_url()` Function

```python
import posixpath
from urllib.parse import (
    urlparse, urlunparse, urljoin,
    parse_qsl, urlencode,
)

def normalize_url(url: str, base: str = None) -> str:
    """
    Normalize a URL for consistent deduplication.
    Steps: resolve relative → lowercase scheme+host → remove default port
           → resolve dot segments → sort query params → strip fragment
    """
    # 1. Resolve relative URLs against a base
    if base:
        url = urljoin(base, url)

    parsed = urlparse(url)

    # 2. Lowercase scheme and host
    scheme = parsed.scheme.lower()
    netloc = parsed.netloc.lower()

    # 3. Remove default ports
    if ":" in netloc:
        host, port = netloc.rsplit(":", 1)
        if (scheme == "http" and port == "80") or \
           (scheme == "https" and port == "443"):
            netloc = host

    # 4. Resolve dot segments in path (/a/b/../c → /a/c)
    path = posixpath.normpath(parsed.path) if parsed.path else ""
    # normpath strips trailing slash — restore it if the original had one
    if parsed.path.endswith("/") and not path.endswith("/"):
        path += "/"

    # 5. Sort query parameters
    query_pairs = parse_qsl(parsed.query, keep_blank_values=False)
    query_pairs.sort()
    query = urlencode(query_pairs)

    # 6. Drop fragment entirely
    return urlunparse((scheme, netloc, path, parsed.params, query, ""))
```

---

## Examples

```python
normalize_url("HTTP://Example.COM/foo/../bar?z=3&a=1#section")
# → "http://example.com/bar?a=1&z=3"

normalize_url("http://example.com:80/page")
# → "http://example.com/page"

normalize_url("https://example.com:443/secure")
# → "https://example.com/secure"

# Resolving a relative URL found while crawling
normalize_url("/about", base="https://example.com/blog/post")
# → "https://example.com/about"

normalize_url("../images/logo.png", base="https://example.com/blog/post/")
# → "https://example.com/blog/images/logo.png"
```

---

## Quick Version (no dot-segment resolution)

For most crawlers, a simpler version is sufficient:

```python
from urllib.parse import urlparse, urlunparse, parse_qsl, urlencode

def normalize_simple(url: str) -> str:
    p = urlparse(url)
    scheme = p.scheme.lower()
    netloc = p.netloc.lower()
    query = urlencode(sorted(parse_qsl(p.query)))
    return urlunparse((scheme, netloc, p.path, p.params, query, ""))
```

---

## Using `parse_qsl` vs `parse_qs`

```python
from urllib.parse import parse_qsl, parse_qs

# parse_qs — dict, loses duplicate keys
parse_qs("tag=a&tag=b&z=1")    # {'tag': ['a', 'b'], 'z': ['1']}

# parse_qsl — list of tuples, preserves duplicates — use this for sorting
parse_qsl("tag=a&tag=b&z=1")   # [('tag', 'a'), ('tag', 'b'), ('z', '1')]
```

Always use `parse_qsl` when the goal is normalization — it correctly handles duplicate keys like `?tag=a&tag=b`.

---

## Gotchas

- **`posixpath.normpath` removes trailing slashes** — restore them if your policy keeps them
- **`urljoin` trailing slash is tricky**: `urljoin("http://example.com/a/b", "page.html")` → `/a/page.html` (treats `b` as file); `urljoin("http://example.com/a/b/", "page.html")` → `/a/b/page.html` (treats `b/` as directory)
- **Normalize BEFORE set insertion or hashing** — `http://Example.COM/page` and `http://example.com/page` are the same URL

---

## Related Notes

- [[Concepts/URL_Normalization]] — theory: steps, why each matters
- [[WebCrawling]] — hub
- [[Python/Extracting_Links_Python]] — normalize after resolving relative URLs
- [[Python/Duplicate_Avoidance_Python]] — normalize before hashing or set insertion
