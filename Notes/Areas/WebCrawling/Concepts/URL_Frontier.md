---
tags: [web-crawling, frontier, queue, bfs]
type: concept
area: learning
---

# URL Frontier

The URL frontier is the data structure that manages which URLs a crawler should visit next. It is the crawler's "to-do list" and its design determines three key properties: **coverage** (which pages get crawled), **freshness** (how quickly updates are recrawled), and **politeness** (how respectfully the server is treated).

---

## Key Vocabulary

| Term | Meaning |
|---|---|
| **Frontier** | The complete set of URLs waiting to be fetched |
| **Seed URLs** | The initial URLs the crawl starts from |
| **FIFO queue** | First-In-First-Out — the standard BFS structure |
| **Priority queue** | URLs assigned scores; highest-priority fetched first |
| **Politeness queue** | Per-host sub-queue enforcing a minimum delay between requests to the same domain |
| **Back queue** | The per-host queue responsible for crawl politeness |
| **Front queue** | The priority-sorted queue that feeds into back queues |
| **Crawl delay** | Minimum time between successive requests to one host |
| **Seen set** | URLs already added to the frontier — prevents duplicate enqueuing |

---

## BFS vs DFS — [[Notes/Resources/CS_Fundamentals/BFS_DFS|What are these?]]

| | BFS (`deque.popleft()`) | DFS (`deque.pop()`) |
|---|---|---|
| Order | Level by level outward | As deep as possible first |
| Politeness | Naturally spreads load across hosts | Can hammer one host repeatedly |
| Standard for crawling | **Yes** | No |

BFS is the industry standard because spreading across many domains is inherently polite.

---

## The Two-Queue Architecture (IR textbook model)

For production crawlers, the frontier splits into front queues (by priority) and back queues (by domain):

```
Discovered URLs
      ↓
[Prioritizer] → Front Queue 1 (high priority)
                Front Queue 2 (medium)
                Front Queue 3 (low)
                      ↓
              [Queue Router] → Back Queue: example.com
                               Back Queue: other.org
                               Back Queue: foo.net
                                     ↓
                               [Fetchers] (respect crawl-delay per host)
```

For a mini search engine, a simple per-domain politeness queue is sufficient.

---

## Two Sets to Maintain

| Set | What it tracks | When to update |
|---|---|---|
| **Seen set** | URLs added to the frontier | At enqueue time |
| **Visited set** | URLs actually fetched | At fetch time |

Add to `seen` the moment a URL is discovered — before it is fetched — to avoid queuing the same URL multiple times while it is waiting in the frontier.

---

## Memory Warning

A Python `deque` is in-memory only. At scale (millions of URLs), move the frontier to persistent storage: Redis sorted sets, SQLite, or a message queue. For a mini search engine, `deque` is fine.

---

## Related Notes

- [[WebCrawling]] — hub
- [[Python/URL_Frontier_Python]] — implementation with deque, heapq, and polite queue
- [[Concepts/URL_Normalization]] — normalize before enqueuing
- [[Concepts/Duplicate_Avoidance]] — the seen/visited sets that prevent re-crawling
- [[Concepts/Crawl_Depth]] — depth is tracked alongside each URL in the frontier
- [[Concepts/Robots_txt]] — crawl-delay from robots.txt feeds into the politeness queue

## Open Questions

- How does a distributed crawler coordinate a shared frontier across multiple machines?
- When is a priority queue worth the complexity over a simple FIFO?
- How do you implement "freshness" — recrawling updated pages?
