---
tags: [python, web-crawling, bfs, crawl-depth, politeness]
type: implementation
area: learning
---

# BFS Crawler with Depth Limiting in Python

See [[Concepts/Crawl_Depth]] for theory on depth, domain restriction, and crawl politeness.

This note contains the most complete implementation — it combines all crawling concepts.

---

## Per-Domain Rate Limiter

```python
import time
from urllib.parse import urlparse

class PoliteRateLimiter:
    """Tracks last-request time per domain and enforces a minimum delay."""

    def __init__(self, default_delay: float = 1.0):
        self.default_delay = default_delay
        self._last_request: dict[str, float] = {}

    def wait(self, url: str, crawl_delay: float | None = None):
        """Block until it is polite to fetch this URL."""
        host = urlparse(url).netloc
        delay = crawl_delay if crawl_delay is not None else self.default_delay
        elapsed = time.monotonic() - self._last_request.get(host, 0.0)
        if elapsed < delay:
            time.sleep(delay - elapsed)
        self._last_request[host] = time.monotonic()
```

---

## BFS Crawler with Depth + Domain + Robots + Rate Limiting

```python
import collections
import urllib.parse
import urllib.robotparser
import hashlib
import requests
from bs4 import BeautifulSoup
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

USER_AGENT = "MyCrawler/1.0 (+https://example.com/about-crawler)"

def make_session() -> requests.Session:
    retry = Retry(total=3, backoff_factor=1.0, status_forcelist=[429, 500, 502, 503, 504])
    adapter = HTTPAdapter(max_retries=retry)
    session = requests.Session()
    session.mount("https://", adapter)
    session.mount("http://", adapter)
    session.headers["User-Agent"] = USER_AGENT
    return session

def get_robot_parser(base_url: str, session: requests.Session):
    robots_url = urllib.parse.urljoin(base_url, "/robots.txt")
    rp = urllib.robotparser.RobotFileParser()
    try:
        resp = session.get(robots_url, timeout=10)
        if resp.status_code == 200:
            rp.parse(resp.text.splitlines())
    except requests.RequestException:
        pass
    return rp

def normalize_url(url: str, base: str = None) -> str:
    if base:
        url = urllib.parse.urljoin(base, url)
    p = urllib.parse.urlparse(url)
    scheme = p.scheme.lower()
    netloc = p.netloc.lower()
    query = urllib.parse.urlencode(sorted(urllib.parse.parse_qsl(p.query)))
    url_no_frag = urllib.parse.urlunparse((scheme, netloc, p.path, p.params, query, ""))
    return url_no_frag

def crawl(
    seed_url: str,
    max_depth: int = 2,
    max_pages: int = 100,
    default_delay: float = 1.0,
    same_domain_only: bool = True,
) -> dict[str, int]:
    """
    BFS crawler with depth limiting, domain restriction,
    robots.txt compliance, and per-domain rate limiting.

    Returns a dict of {url: depth} for all visited pages.
    """
    session = make_session()
    limiter = PoliteRateLimiter(default_delay=default_delay)

    seed = normalize_url(seed_url)
    seed_domain = urllib.parse.urlparse(seed).netloc
    base_url = urllib.parse.urlparse(seed).scheme + "://" + seed_domain

    # Fetch robots.txt once for the seed domain
    rp = get_robot_parser(base_url, session)
    crawl_delay = rp.crawl_delay(USER_AGENT.split("/")[0]) or \
                  rp.crawl_delay("*") or default_delay

    # Frontier: (url, depth) tuples
    frontier = collections.deque([(seed, 0)])
    seen = {seed}              # all enqueued URLs
    visited: dict[str, int] = {}   # url → depth
    content_hashes: set[str] = set()

    while frontier and len(visited) < max_pages:
        url, depth = frontier.popleft()

        # Robots check
        if not rp.can_fetch(USER_AGENT.split("/")[0], url):
            print(f"  [robots] Disallowed: {url}")
            continue

        # Polite delay
        limiter.wait(url, crawl_delay=crawl_delay)

        # Fetch
        try:
            response = session.get(url, timeout=(5, 30))
            response.raise_for_status()
        except requests.RequestException as e:
            print(f"  [error] {url}: {e}")
            continue

        # Content dedup
        content_fp = hashlib.sha256(response.content).hexdigest()
        if content_fp in content_hashes:
            print(f"  [dup-content] {url}")
            continue
        content_hashes.add(content_fp)

        visited[url] = depth
        print(f"  [depth={depth}] {url}")

        # Discover links (only if not at max depth)
        if depth < max_depth:
            soup = BeautifulSoup(response.text, "html.parser")
            for tag in soup.find_all("a", href=True):
                href = tag["href"].strip()
                if not href or href.startswith(("mailto:", "javascript:", "#")):
                    continue
                norm = normalize_url(href, base=url)
                if not norm.startswith(("http://", "https://")):
                    continue
                if same_domain_only:
                    if urllib.parse.urlparse(norm).netloc != seed_domain:
                        continue
                if norm not in seen:
                    seen.add(norm)
                    frontier.append((norm, depth + 1))

    return visited


# --- Usage ---
if __name__ == "__main__":
    results = crawl("https://example.com", max_depth=2, max_pages=50)
    for url, depth in sorted(results.items(), key=lambda x: x[1]):
        print(f"depth={depth}  {url}")
```

---

## Depth Reference

| `max_depth` | What gets crawled |
|---|---|
| 0 | Seed page only (fetched but links not followed) |
| 1 | Seed + all pages directly linked from seed |
| 2 | Seed + depth-1 pages + their linked pages |

---

## Gotchas

- **Track `seen` at enqueue time**, not fetch time — otherwise the same URL can be queued many times before being processed
- **Normalize URLs before the `seen` check** — `http://Example.COM/page` and `http://example.com/page` must match
- **Rate-limit per domain**, not globally — `limiter.wait(url)` uses `urlparse(url).netloc` as the key
- **Re-check `can_fetch` per URL**, not just per domain — different paths on the same domain can have different rules

---

## Related Notes

- [[Concepts/Crawl_Depth]] — theory: depth tracking, politeness, backoff formula
- [[WebCrawling]] — hub
- [[Python/HTTP_with_Requests_HTTPX]] — the session and retry logic
- [[Python/HTML_Parsing_BeautifulSoup]] — BeautifulSoup usage
- [[Python/Extracting_Links_Python]] — link extraction and normalization
- [[Python/URL_Frontier_Python]] — the `deque` and `seen` set patterns
- [[Python/Duplicate_Avoidance_Python]] — content hashing
- [[Python/Robots_txt_Python]] — `RobotFileParser` integration
