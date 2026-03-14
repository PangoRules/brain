---
tags: [web-crawling, links, html]
type: concept
area: learning
---

# Extracting Links

Link extraction is the process of finding all URLs referenced from a page and resolving them to absolute form so they can be added to the crawler's frontier. It is the mechanism by which a crawler discovers new pages.

---

## Key Vocabulary

| Term | Meaning |
|---|---|
| **Absolute URL** | Full URL with scheme and host: `https://example.com/path` |
| **Relative URL** | Path only — must be resolved against a base: `/path` or `../path` |
| **Base URL** | The URL of the page currently being scraped |
| **`urljoin(base, url)`** | `urllib.parse` function that resolves relative → absolute |
| **Fragment** | `#anchor` — points to a section within the same page, never sent to server |
| **`mailto:`** | Email link — not a web URL, must be filtered out |
| **`javascript:`** | Inline JS in href — not a URL, must be filtered out |
| **`urldefrag`** | Strips the `#fragment` part from a URL |

---

## Link Types in the Wild

| href value | Type | Action |
|---|---|---|
| `https://other.com/page` | Absolute external | Resolve (no-op) → keep if cross-domain crawl |
| `/about` | Root-relative | Resolve against scheme+host |
| `../images/logo.png` | Relative | Resolve against current path |
| `#section-2` | Fragment | Resolves to same page — strip fragment, skip if same as current |
| `mailto:hi@example.com` | Email | Discard |
| `javascript:void(0)` | JS | Discard |
| `tel:+1234567890` | Phone | Discard |

---

## How `urljoin` Resolves URLs

```
base = "https://example.com/section/page.html"

/other          →  https://example.com/other
sibling.html    →  https://example.com/section/sibling.html
../up.html      →  https://example.com/up.html
https://other/  →  https://other/        (absolute always wins)
#anchor         →  https://example.com/section/page.html#anchor
mailto:x@y.com  →  mailto:x@y.com        (non-http scheme wins)
```

Always check the scheme of the result — `urljoin` returns something for every input.

---

## The `<base>` Tag Edge Case

Some pages declare a different base URL in `<head>`:
```html
<head><base href="https://cdn.example.com/"></head>
```
Check for it before resolving links, or all relative hrefs will resolve to the wrong domain.

---

## Domain Filtering

A crawler is usually restricted to one domain. After resolving to absolute, check:
```
urlparse(url).netloc == urlparse(base_url).netloc
```
For subdomain-inclusive matching: `netloc.endswith(".example.com") or netloc == "example.com"`.

---

## Related Notes

- [[WebCrawling]] — hub
- [[Python/Extracting_Links_Python]] — implementation
- [[Concepts/HTML_Parsing]] — how links are found in the HTML tree
- [[Concepts/URL_Normalization]] — normalize before adding to the frontier
- [[Concepts/URL_Frontier]] — where extracted links go

## Open Questions

- How do you extract links from `<link>`, `<script src>`, `<img src>` tags?
- What is anchor text and why does it matter for ranking?
- How does a crawler handle redirects that reveal new canonical URLs?
