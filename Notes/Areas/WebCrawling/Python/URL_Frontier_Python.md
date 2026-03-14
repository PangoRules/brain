---
tags: [python, web-crawling, frontier, queue, deque]
type: implementation
area: learning
---

# URL Frontier in Python

See [[Concepts/URL_Frontier]] for theory on BFS vs DFS, politeness queues, and the front/back queue architecture.

---

## 1. Basic BFS Frontier — `collections.deque` ([[Notes/Resources/CS_Fundamentals/BFS_DFS|BFS explained]])

```python
from collections import deque

frontier: deque[str] = deque(seed_urls)   # FIFO: append → right, pop → left
seen: set[str] = set(seed_urls)           # prevent re-enqueuing

while frontier:
    url = frontier.popleft()              # BFS: take from the front

    # ... fetch and parse the page ...
    for link in discovered_links:
        if link not in seen:
            seen.add(link)                # mark at enqueue time, not fetch time
            frontier.append(link)
```

**Why `deque` not `list`:** `list.pop(0)` is [[Notes/Resources/CS_Fundamentals/Big_O_Notation|O(n)]]; `deque.popleft()` is O(1).

---

## 2. Priority Queue Frontier — `heapq`

For importance-driven crawling (e.g., PageRank-style scores). Lower number = higher priority.

```python
import heapq

class PriorityFrontier:
    def __init__(self):
        self._heap = []
        self._counter = 0   # tiebreaker — keeps heap stable when priorities are equal

    def push(self, url: str, priority: float = 0.0):
        heapq.heappush(self._heap, (priority, self._counter, url))
        self._counter += 1

    def pop(self) -> str:
        _, _, url = heapq.heappop(self._heap)
        return url

    def __len__(self) -> int:
        return len(self._heap)

# Usage
pf = PriorityFrontier()
pf.push("https://example.com/important", priority=1)
pf.push("https://example.com/low-priority", priority=10)
next_url = pf.pop()   # returns priority=1 URL first
```

---

## 3. Politeness Queue — Per-Domain Rate Limiting

```python
import time
from collections import deque, defaultdict
from urllib.parse import urlparse

class PoliteFrontier:
    """Per-domain queues with configurable crawl delay."""

    def __init__(self, delay: float = 1.0):
        self.delay = delay
        self._queues: dict[str, deque] = defaultdict(deque)
        self._next_fetch: dict[str, float] = {}   # host → next allowed fetch time

    def add(self, url: str):
        host = urlparse(url).netloc
        self._queues[host].append(url)

    def get_next(self) -> str | None:
        """Return the next URL ready to fetch, or None if all hosts are in cooldown."""
        now = time.monotonic()
        for host, queue in self._queues.items():
            if not queue:
                continue
            if now >= self._next_fetch.get(host, 0.0):
                url = queue.popleft()
                self._next_fetch[host] = now + self.delay
                return url
        return None   # all domains in cooldown

    def __len__(self) -> int:
        return sum(len(q) for q in self._queues.values())

# Usage
frontier = PoliteFrontier(delay=2.0)
frontier.add("https://example.com/page1")
frontier.add("https://example.com/page2")   # same host → will be delayed
frontier.add("https://other.org/home")       # different host → available immediately

while len(frontier) > 0:
    url = frontier.get_next()
    if url is None:
        time.sleep(0.05)   # brief sleep while all hosts are in cooldown
        continue
    print(f"Fetching: {url}")
    # ... fetch, then frontier.add() discovered links
```

---

## Combining Seen Set + Frontier

```python
from collections import deque

def crawl(seeds: list[str], max_pages: int = 50):
    frontier = deque(seeds)
    seen = set(seeds)       # tracks enqueued URLs (not just fetched ones)
    visited = {}            # url → depth or other metadata

    while frontier and len(visited) < max_pages:
        url = frontier.popleft()
        visited[url] = True

        # fetch ...
        new_links = []  # from HTML parsing

        for link in new_links:
            if link not in seen:     # check seen, not visited
                seen.add(link)
                frontier.append(link)
```

---

## Gotchas

- **Use `seen`, not `visited`**, to gate enqueuing. Adding to `seen` at fetch time means the same URL can be enqueued many times before it is ever processed.
- **`heapq` has no update-priority operation.** Workaround: push a new entry and use a separate `invalid` set to skip stale ones.
- **Rate-limit per hostname**, not globally. Multiple virtual hosts share IPs — always use the DNS name.
- **Move to persistent storage** (Redis, SQLite) for crawls beyond ~1M URLs — a deque will exhaust RAM.

---

## Related Notes

- [[Concepts/URL_Frontier]] — theory: BFS vs DFS, front/back queue architecture
- [[WebCrawling]] — hub
- [[Python/URL_Normalization_Python]] — normalize before adding to frontier
- [[Python/Duplicate_Avoidance_Python]] — the seen and visited sets
- [[Python/Crawl_Depth_Python]] — depth is stored as `(url, depth)` tuples in the frontier
