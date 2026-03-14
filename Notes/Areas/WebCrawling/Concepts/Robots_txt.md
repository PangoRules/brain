---
tags: [web-crawling, robots-txt, crawl-politeness]
type: concept
area: learning
---

# robots.txt

`robots.txt` is a plain-text file at the root of a website (`https://example.com/robots.txt`) that communicates access rules to automated crawlers. It implements the **Robots Exclusion Protocol (REP)** — a community convention that well-behaved bots voluntarily honor. It is not technically enforced; a malicious bot can ignore it entirely.

The file is fetched before any other page on a site.

---

## Key Vocabulary

| Term | Meaning |
|---|---|
| **REP** | Robots Exclusion Protocol — the informal standard |
| **User-agent** | Which crawler the rules apply to; `*` is a wildcard matching all bots |
| **Disallow** | Blocks a path and all subpaths from being crawled |
| **Allow** | Explicitly permits a path that a broader `Disallow` would block |
| **Crawl-delay** | Requests N seconds between requests — honored by most bots except Googlebot |
| **Sitemap** | Points to the XML sitemap — applies globally, not to a specific agent |
| **Specificity rule** | When multiple rules match a URL, the longest (most specific) path wins |

---

## Syntax

```text
# Comment
User-agent: *
Disallow: /private/
Allow: /private/public/
Crawl-delay: 5
Sitemap: https://example.com/sitemap.xml
```

A blank line separates rule groups. Each group starts with `User-agent`.

---

## Directive Details

**`Disallow`**
- `Disallow: /` — block the entire site
- `Disallow:` (empty) — allow everything
- Path matching is **case-sensitive**
- `Disallow: /folder` blocks `/folder` AND `/folderextra` (implicit `*` suffix)
- `Disallow: /folder/` blocks only `/folder/` and its children

**`Allow`** — carves exceptions out of broader `Disallow` rules:
```
Disallow: /products/
Allow: /products/featured/    # this subdirectory is still allowed
```

**Wildcards in paths**
- `*` matches zero or more characters: `Disallow: /*.pdf$`
- `$` anchors the end: `Disallow: /*.pdf$` blocks only `.pdf` URLs
- `Disallow: /search?*` blocks all query strings under `/search`

---

## Specificity Rule

When multiple rules match, the **longest matching path** wins — even if it is less restrictive:
```
Disallow: /admin/
Allow: /admin/public       # wins for /admin/public because it's more specific
```

---

## How Missing or Broken robots.txt Is Handled

| Server response | Behavior |
|---|---|
| 200 OK | Parse and honor |
| 404 Not Found | Treat as fully open — crawl anything |
| 5xx Server Error | Treat as temporarily inaccessible — retry later |

---

## Caching robots.txt

Fetch robots.txt **once per domain per crawl session**. Google recommends re-fetching every 24 hours for long-running crawlers. Never cache forever — sites update their rules.

---

## Related Notes

- [[WebCrawling]] — hub
- [[Python/Robots_txt_Python]] — implementation with `urllib.robotparser`
- [[Concepts/Crawl_Depth]] — crawl politeness overview
- [[Concepts/URL_Frontier]] — crawl-delay feeds into the politeness queue

## Open Questions

- What happens when `User-agent: Googlebot` and `User-agent: *` both exist — which applies to my bot?
- How do you handle robots.txt for sites that use authentication?
- What is the difference between `robots.txt` and the `X-Robots-Tag` HTTP header?
