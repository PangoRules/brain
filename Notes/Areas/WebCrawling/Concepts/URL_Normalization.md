---
tags: [web-crawling, url, normalization]
type: concept
area: learning
---

# URL Normalization

URL normalization (also called canonicalization) transforms a URL into a consistent, standard form so that two URLs pointing to the same resource always produce the same string. Without it, `HTTP://Example.COM/foo`, `http://example.com/foo`, and `http://example.com/foo#section` are treated as three different pages.

It is the foundation of deduplication — every URL must be normalized before being checked against the visited set or added to the frontier.

---

## Key Vocabulary

| Term | Meaning |
|---|---|
| **Canonical URL** | The single normalized form chosen to represent a resource |
| **Scheme** | The protocol part: `http`, `https` |
| **Netloc / Authority** | `host:port` combined |
| **Path** | The `/path/to/page` part |
| **Query string** | `?key=value` parameters |
| **Fragment** | `#anchor` — never sent to the server; always safe to drop |
| **Dot segments** | `.` (current dir) and `..` (parent dir) in paths — must be resolved |

---

## Normalization Steps

| Step | Before | After |
|---|---|---|
| Lowercase scheme | `HTTP://example.com/` | `http://example.com/` |
| Lowercase host | `http://EXAMPLE.COM/` | `http://example.com/` |
| Remove default port | `http://example.com:80/` | `http://example.com/` |
| Resolve dot segments | `http://example.com/a/b/../c` | `http://example.com/a/c` |
| Remove fragment | `http://example.com/page#top` | `http://example.com/page` |
| Sort query params | `?b=2&a=1` | `?a=1&b=2` |
| Trailing slash | Pick a policy (strip or keep) and apply consistently |

---

## Why Each Step Matters

- **Lowercase**: servers are case-insensitive for scheme/host but the string comparison is not
- **Default port removal**: `:80` and `:443` are implicit — without removal they look like different URLs
- **Fragment stripping**: fragments are client-only anchors; the server returns the same page regardless
- **Query param sorting**: `?a=1&b=2` and `?b=2&a=1` fetch the same page but are different strings
- **Dot segments**: `/a/b/../c` and `/a/c` are the same path but different strings

---

## `urllib.parse` Toolkit

| Function | Purpose |
|---|---|
| `urlparse(url)` | Split into `(scheme, netloc, path, params, query, fragment)` |
| `urlunparse(parts)` | Reassemble 6-tuple back into a URL string |
| `urljoin(base, url)` | Resolve relative URL against a base |
| `parse_qsl(query)` | Parse query string into `[(key, value), ...]` — use this for sorting |
| `urlencode(pairs)` | Encode a list of pairs back into a query string |
| `urldefrag(url)` | Strip the fragment: returns `(url_without_fragment, fragment)` |

Use `parse_qsl` (not `parse_qs`) when sorting — it preserves duplicate keys like `?tag=a&tag=b`.

---

## Related Notes

- [[WebCrawling]] — hub
- [[Python/URL_Normalization_Python]] — implementation
- [[Concepts/Duplicate_Avoidance]] — normalization must happen before hashing
- [[Concepts/Extracting_Links]] — normalize after resolving relative URLs
- [[Concepts/URL_Frontier]] — normalize before enqueuing

## Open Questions

- When should you NOT sort query params (e.g., session-sensitive ordering)?
- How do you handle internationalized domain names (IDN)?
- Should trailing slashes be kept or stripped? Does it matter for which sites?
