---
tags: [web-crawling, networking, http]
type: concept
area: learning
---

# HTTP Requests

HTTP (HyperText Transfer Protocol) is the foundation of data communication on the web. A crawler uses HTTP GET to fetch pages: the client sends a request with a method, URL, and headers; the server responds with a status code, response headers, and a body.

---

## Key Vocabulary

| Term | Meaning |
|---|---|
| **GET** | HTTP method for retrieving a resource — the only method a crawler needs |
| **Status code** | 3-digit integer indicating the outcome of the request |
| **Request headers** | Metadata sent with the request (User-Agent, Accept, etc.) |
| **Response headers** | Metadata returned by the server (Content-Type, Retry-After, etc.) |
| **User-Agent** | Header identifying the client — tells servers who is making the request |
| **Timeout** | Max time to wait for a connection or response before giving up |
| **Retry** | Re-issuing a failed request, usually after a delay |
| **Backoff** | Progressively longer wait between retries |
| **Session** | A persistent connection object that reuses TCP connections and cookies |
| **Redirect** | 3xx response telling the client the resource has moved to a new URL |

---

## Status Codes for Crawlers

| Code | Name | What to do |
|---|---|---|
| **200** | OK | Success — parse the body |
| **301 / 302** | Redirect | Follow automatically (libraries do this by default) |
| **404** | Not Found | Skip — do not retry, mark as dead |
| **429** | Too Many Requests | Back off — honor the `Retry-After` header |
| **503** | Service Unavailable | Retry with exponential backoff |
| **5xx** | Server Error | Retry with backoff; stop after N attempts |

---

## Why Timeouts Are Non-Negotiable

Both `requests` and `httpx` wait **forever** by default. A single hung connection will stall your entire crawler. Always set:
- **Connect timeout** — how long to wait to establish the TCP connection
- **Read timeout** — how long to wait for the server to send data

A good default: `timeout=(5, 30)` — 5s to connect, 30s to read.

---

## Crawl Politeness via Headers

Always set a descriptive `User-Agent` that identifies your bot:
```
User-Agent: MyCrawler/1.0 (+https://mysite.com/about-crawler)
```
Many servers block the default `python-requests/2.x` and `python-httpx/0.x` strings. Impersonating a browser User-Agent is dishonest and bad practice.

---

## Related Notes

- [[WebCrawling]] — hub
- [[Python/HTTP_with_Requests_HTTPX]] — implementation
- [[Concepts/URL_Frontier]] — how fetched pages feed back into the queue
- [[Concepts/Crawl_Depth]] — politeness and rate limiting

## Open Questions

- When should you use `httpx` async vs synchronous `requests`?
- How do you handle gzip-compressed responses correctly?
- What is HTTP/2 and does it matter for crawling?
- How does connection pooling affect crawl speed?
