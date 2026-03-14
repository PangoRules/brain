---
tags: [python, web-crawling, http, requests, httpx]
type: implementation
area: learning
---

# HTTP with `requests` and `httpx`

See [[Concepts/HTTP_Requests]] for theory, status codes, and header conventions.

---

## `requests` — Session with Auto-Retry

```python
import requests
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

def make_session(user_agent: str = "MyCrawler/1.0 (+https://mysite.com/bot)") -> requests.Session:
    """Session with exponential backoff on 429 and 5xx responses."""
    retry_strategy = Retry(
        total=5,                               # max retry attempts
        backoff_factor=1,                      # waits: 0, 1, 2, 4, 8s
        status_forcelist=[429, 500, 502, 503, 504],  # transient errors worth retrying
        allowed_methods=["GET", "HEAD"],
        respect_retry_after_header=True,       # honor Retry-After on 429
    )
    adapter = HTTPAdapter(max_retries=retry_strategy)
    session = requests.Session()
    session.mount("https://", adapter)
    session.mount("http://", adapter)
    session.headers.update({
        "User-Agent": user_agent,
        "Accept": "text/html,application/xhtml+xml",
    })
    return session

# Usage
session = make_session()
response = session.get("https://example.com", timeout=(5, 30))
response.raise_for_status()   # raises HTTPError on 4xx/5xx
print(response.text)          # HTML body
print(response.status_code)
print(response.url)           # final URL after any redirects
```

---

## `requests` — Basic GET

```python
import requests

# Always set timeout — default is None (waits forever)
response = requests.get(
    "https://example.com",
    headers={"User-Agent": "MyCrawler/1.0"},
    timeout=(5, 30),           # (connect_timeout, read_timeout) in seconds
)

response.raise_for_status()   # raises on 4xx/5xx
html = response.text           # decoded string
raw  = response.content        # raw bytes (use for hashing)
```

---

## `httpx` — Client (drop-in replacement for requests)

`httpx` has no built-in retry — use manual backoff or `tenacity`.

```python
import httpx
import time

timeout = httpx.Timeout(connect=5.0, read=30.0, write=10.0, pool=5.0)

with httpx.Client(
    headers={"User-Agent": "MyCrawler/1.0"},
    timeout=timeout,
    follow_redirects=True,
) as client:
    response = client.get("https://example.com")
    response.raise_for_status()
    print(response.text)
```

---

## `httpx` — Manual Retry with Backoff

```python
import httpx, time

def fetch(url: str, max_retries: int = 5) -> httpx.Response:
    for attempt in range(max_retries):
        try:
            r = httpx.get(url, timeout=httpx.Timeout(10.0))
            if r.status_code == 429:
                wait = int(r.headers.get("Retry-After", 2 ** attempt))
                time.sleep(wait)
                continue
            r.raise_for_status()
            return r
        except (httpx.TimeoutException, httpx.NetworkError):
            if attempt == max_retries - 1:
                raise
            time.sleep(2 ** attempt)   # 1, 2, 4, 8, 16s
    raise RuntimeError(f"Failed after {max_retries} retries: {url}")
```

---

## `httpx` — Async (fetching many URLs concurrently)

```python
import httpx, asyncio

async def fetch_all(urls: list[str]) -> list[str]:
    timeout = httpx.Timeout(10.0)
    async with httpx.AsyncClient(
        headers={"User-Agent": "MyCrawler/1.0"},
        timeout=timeout,
        follow_redirects=True,
    ) as client:
        responses = await asyncio.gather(*[client.get(u) for u in urls])
        return [r.text for r in responses]

results = asyncio.run(fetch_all(["https://example.com", "https://example.org"]))
```

---

## Gotchas

- `requests` does **not** raise on 4xx/5xx — always call `.raise_for_status()` or check `.status_code`
- `httpx` has **no built-in retry** — the `urllib3.Retry` adapter only works with `requests`
- `respect_retry_after_header=True` is **not the default** in `urllib3.Retry` — add it explicitly
- `backoff_factor=1` gives delays: 0, 1, 2, 4, 8s (`factor * 2^(attempt-1)`)
- Default `User-Agent` strings (`python-requests/...`) are blocked by many servers — always set your own

---

## Related Notes

- [[Concepts/HTTP_Requests]] — theory, status codes, header conventions
- [[WebCrawling]] — hub
- [[Python/Crawl_Depth_Python]] — full crawler that uses a session
- [[Python/Robots_txt_Python]] — fetching robots.txt with the same session
