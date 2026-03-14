---
tags: [web-crawling, index, moc]
type: moc
area: learning
---

# Web Crawling

Web crawling is the automated process of systematically browsing the web to collect documents. A crawler (or spider) starts from seed URLs, fetches pages, extracts links, and follows them — building a [[Notes/Areas/NLP/Concepts/Corpus|corpus]] for indexing, search, or analysis.

---

## Concepts (Theory)

| Topic | Note |
|---|---|
| HTTP Requests | [[Concepts/HTTP_Requests]] |
| HTML Parsing | [[Concepts/HTML_Parsing]] |
| Extracting Links | [[Concepts/Extracting_Links]] |
| URL Normalization | [[Concepts/URL_Normalization]] |
| URL Frontier | [[Concepts/URL_Frontier]] |
| Duplicate Avoidance | [[Concepts/Duplicate_Avoidance]] |
| robots.txt | [[Concepts/Robots_txt]] |
| Crawl Depth | [[Concepts/Crawl_Depth]] |

---

## Python Implementations

| Topic | Note |
|---|---|
| HTTP with `requests` / `httpx` | [[Python/HTTP_with_Requests_HTTPX]] |
| HTML Parsing with BeautifulSoup | [[Python/HTML_Parsing_BeautifulSoup]] |
| Extracting Links | [[Python/Extracting_Links_Python]] |
| URL Normalization | [[Python/URL_Normalization_Python]] |
| URL Frontier with `deque` | [[Python/URL_Frontier_Python]] |
| Duplicate Avoidance | [[Python/Duplicate_Avoidance_Python]] |
| robots.txt with `urllib.robotparser` | [[Python/Robots_txt_Python]] |
| BFS with Depth Limiting | [[Python/Crawl_Depth_Python]] |

---

## Python Tools Used

| Tool | Note |
|---|---|
| `requests` / `httpx` | HTTP clients for fetching pages |
| `BeautifulSoup` (`bs4`) | HTML parsing |
| `urllib.parse` | URL manipulation: `urljoin`, `urlparse`, `urlencode` |
| `urllib.robotparser` | Parsing and honoring `robots.txt` |
| `collections.deque` | Efficient FIFO frontier queue |
| `hashlib` | MD5/SHA-256 content fingerprinting |
| `heapq` | Priority queue for importance-driven crawling |
| `time` | Enforcing crawl delays |

---

## Applied In Projects

| Project | How crawling is used |
|---------|---------------------|
| [[Notes/Projects/python/mini-search-engine/00-MiniSearchEngine\|Mini Search Engine]] | Phase 3 — collect documents from the web for indexing and ranking |
