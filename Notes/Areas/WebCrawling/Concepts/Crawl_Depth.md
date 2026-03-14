---
tags: [web-crawling, bfs, crawl-depth, politeness]
type: concept
area: learning
---

# Crawl Depth

Crawl depth is the number of link hops from a seed URL to a given page. The seed itself is at depth 0; pages linked directly from it are at depth 1; pages linked from those are at depth 2; and so on.

Without a depth limit, a BFS crawler on a large site can run indefinitely — consuming unbounded memory and bandwidth, because each depth level multiplies URLs by the average page out-degree (often 10–100+).

---

## Key Vocabulary

| Term | Meaning |
|---|---|
| **Depth 0** | The seed URL itself |
| **Depth N** | A page reachable in exactly N hops from the seed |
| **Max depth** | The limit beyond which no new links are enqueued |
| **Out-degree** | The number of outgoing links on a page |
| **Domain restriction** | Only following links that stay on the same domain |
| **Crawl politeness** | Limiting request rate to avoid harming servers |

---

## Depth in BFS

Each item in the frontier queue is a `(url, depth)` tuple. When a page is fetched:
- If `depth < max_depth`: parse its links and enqueue at `depth + 1`
- If `depth == max_depth`: fetch and process it, but **do not enqueue its links**

This bounds the crawl to pages within N hops of the seed.

---

## Politeness

Politeness is the broader discipline of not harming servers. It encompasses:

| Practice | How |
|---|---|
| **Per-domain rate limiting** | Track last-request timestamp per host; sleep if too soon |
| **Respect Crawl-delay** | Read it from `robots.txt` and use it as the per-domain delay |
| **Exponential backoff** | On 429 or 5xx, double the wait time on each retry |
| **Honor Retry-After** | On 429, use the server's stated delay rather than your own |
| **Honest User-Agent** | Identify your bot by name so admins can contact you |

Rate limiting must be **per domain**, not global. A 1-second global delay is irrelevant if your crawler hits the same domain 100 times in a row.

---

## Exponential Backoff Formula

```
wait = base_delay * 2^attempt   (+ small random jitter to avoid thundering herd)
```

| Attempt | Wait (base=1s) |
|---|---|
| 0 | 1s |
| 1 | 2s |
| 2 | 4s |
| 3 | 8s |
| 4 | 16s |

Cap at a `max_delay` (e.g., 60s) so the crawler doesn't stall indefinitely.

---

## Domain Restriction

Combined with depth limiting, domain restriction keeps a crawl focused:
```python
urlparse(discovered_url).netloc == urlparse(seed_url).netloc
```
For subdomain-inclusive: `netloc.endswith(".example.com") or netloc == "example.com"`

---

## Related Notes

- [[WebCrawling]] — hub
- [[Python/Crawl_Depth_Python]] — complete BFS crawler implementation
- [[Concepts/URL_Frontier]] — depth is stored alongside each URL in the frontier
- [[Concepts/Robots_txt]] — source of Crawl-delay values
- [[Concepts/HTTP_Requests]] — backoff strategy for failed requests

## Open Questions

- How do you handle depth fairly when a URL is reachable at multiple depths?
- What is a good default max_depth for a mini search engine?
- How does freshness (recrawl scheduling) interact with depth?
