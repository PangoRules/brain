---
tags: [python, web-crawling, robots-txt, urllib]
type: implementation
area: learning
---

# robots.txt in Python — `urllib.robotparser`

See [[Concepts/Robots_txt]] for theory on directives, wildcard rules, and the Robots Exclusion Protocol.

`urllib.robotparser` is part of the Python standard library — no install needed.

---

## Basic Usage

```python
import urllib.robotparser
from urllib.parse import urljoin

def build_robot_parser(base_url: str) -> urllib.robotparser.RobotFileParser:
    """Fetch and parse robots.txt for a domain."""
    rp = urllib.robotparser.RobotFileParser()
    rp.set_url(urljoin(base_url, "/robots.txt"))
    rp.read()   # performs an HTTP GET internally
    return rp

rp = build_robot_parser("https://example.com")

# Check if a URL is allowed
if rp.can_fetch("MyCrawler", "https://example.com/private/data"):
    print("Allowed")
else:
    print("Disallowed by robots.txt")

# Get Crawl-delay (returns None if not set)
delay = rp.crawl_delay("MyCrawler") or rp.crawl_delay("*") or 1.0
print(f"Delay: {delay}s")

# Get Request-rate (RequestRate namedtuple or None)
rate = rp.request_rate("*")
if rate:
    secs_per_request = rate.seconds / rate.requests
```

---

## Fetching robots.txt with Your Own Session

`rp.read()` uses `urllib.request` internally. If you have a custom session (with retries, User-Agent, etc.), fetch it yourself and parse the text:

```python
import requests
import urllib.robotparser

def get_robot_parser(base_url: str, session: requests.Session) -> urllib.robotparser.RobotFileParser:
    robots_url = base_url.rstrip("/") + "/robots.txt"
    rp = urllib.robotparser.RobotFileParser()
    try:
        resp = session.get(robots_url, timeout=10)
        if resp.status_code == 200:
            rp.parse(resp.text.splitlines())   # parse() takes an iterable of lines
        # 404 → allow all (don't call parse → rp allows everything by default)
        # 5xx → also allow all (be conservative)
    except requests.RequestException:
        pass   # network error → proceed with empty parser (allow all)
    return rp
```

---

## Checking Both Named Agent and Wildcard

```python
def can_fetch(rp: urllib.robotparser.RobotFileParser, url: str, agent: str) -> bool:
    """
    Check the named agent first, then fall back to '*'.
    urllib.robotparser.can_fetch() handles this automatically,
    but this makes the behavior explicit.
    """
    return rp.can_fetch(agent, url)
    # RobotFileParser matches named group if it exists,
    # otherwise falls back to the '*' group.
```

---

## Integrating with the Crawler Loop

```python
from urllib.parse import urlparse
import time

# Cache one parser per domain — don't fetch robots.txt on every request
robot_parsers: dict[str, urllib.robotparser.RobotFileParser] = {}

def get_rp_for_url(url: str, session: requests.Session):
    netloc = urlparse(url).netloc
    if netloc not in robot_parsers:
        base = urlparse(url).scheme + "://" + netloc
        robot_parsers[netloc] = get_robot_parser(base, session)
    return robot_parsers[netloc]

# In the crawl loop:
rp = get_rp_for_url(url, session)

if not rp.can_fetch("MyCrawler", url):
    print(f"robots.txt disallows: {url}")
    continue

delay = rp.crawl_delay("MyCrawler") or rp.crawl_delay("*") or 1.0
# ... apply delay, then fetch
```

---

## API Reference

| Method | Returns | Notes |
|---|---|---|
| `set_url(url)` | None | Set robots.txt URL before calling `read()` |
| `read()` | None | Fetches and parses internally |
| `parse(lines)` | None | Parse from an iterable of strings |
| `can_fetch(agent, url)` | bool | True = allowed |
| `crawl_delay(agent)` | float \| None | None if not set |
| `request_rate(agent)` | RequestRate \| None | `.requests` / `.seconds` |
| `mtime()` | float | Timestamp of last fetch — use to re-fetch after 24h |

---

## Gotchas

- **`read()` silently succeeds on 404** — treats missing file as "allow all" ✓
- **`read()` may raise or fail silently on 5xx** depending on Python version — wrap in try/except
- **`can_fetch()` does exact agent matching + fallback to `*`** — it will not match `Googlebot-Image` when checking `Googlebot`
- **Re-fetch periodically** — cache robot parsers per domain but re-fetch every 24h for long-running crawlers
- **Wildcard path patterns** (`/*.pdf$`) are supported from Python 3.x but edge-case behavior may differ from Googlebot's interpretation

---

## Related Notes

- [[Concepts/Robots_txt]] — theory: directives, specificity rule, missing file handling
- [[WebCrawling]] — hub
- [[Python/Crawl_Depth_Python]] — full crawler that integrates robots.txt check
- [[Python/HTTP_with_Requests_HTTPX]] — session used to fetch robots.txt
