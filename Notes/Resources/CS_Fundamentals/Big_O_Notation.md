---
tags: [cs-fundamentals, algorithms, complexity, performance]
type: concept
area: learning
---

# Big O Notation

Big O notation describes **how the time or memory an operation takes grows as the input gets larger**. It answers: "if I double the data, what happens to the speed?"

It's not about exact milliseconds — it's about the *shape* of the growth.

---

## The Common Classes (fastest → slowest)

| Notation | Name | What it means | Example |
|---|---|---|---|
| **O(1)** | Constant | Same speed regardless of size | `set` lookup, `dict` get |
| **O(log n)** | Logarithmic | Grows very slowly | Binary search |
| **O(n)** | Linear | Grows with the data | Loop through a list |
| **O(n log n)** | Linearithmic | Slightly worse than linear | Merge sort |
| **O(n²)** | Quadratic | Grows fast — nested loops | Bubble sort |
| **O(2ⁿ)** | Exponential | Blows up quickly | Brute-force combinations |

---

## O(1) — Constant Time

The operation takes the **same amount of time no matter how large the data is**.

```python
# These are all O(1) — instant regardless of size
visited = {"https://example.com", "https://other.com", ...}  # 10 million URLs

"https://example.com" in visited   # O(1) — hash lookup, always instant
visited.add("https://new.com")     # O(1) — hash insert, always instant
```

This is why Python `set` is used for the visited URL set in a crawler. If it were a `list`:

```python
"https://example.com" in my_list   # O(n) — checks every item one by one
```

With 1 million URLs, a `list` lookup checks up to 1,000,000 items. A `set` lookup checks **1** (roughly).

---

## O(n) — Linear Time

Time grows proportionally with the number of items.

```python
for url in all_urls:       # 1M urls = 1M iterations
    process(url)
```

Double the data → double the time. Still manageable.

---

## O(n²) — Quadratic Time

Nested loops. Gets painful fast.

```python
for url in all_urls:           # 1M iterations
    for other in all_urls:     # × 1M iterations = 1 trillion operations
        compare(url, other)
```

1,000 items → 1,000,000 operations. 1,000,000 items → 1,000,000,000,000 operations. Avoid.

---

## Intuition: Why It Matters for a Crawler

| Operation | Data structure | Complexity | 1M URLs |
|---|---|---|---|
| "Have I seen this URL?" | `set` | **O(1)** | Instant |
| "Have I seen this URL?" | `list` | O(n) | Checks 1M items |
| Add URL to frontier | `deque.append` | **O(1)** | Instant |
| Take next URL | `deque.popleft()` | **O(1)** | Instant |
| Take next URL | `list.pop(0)` | O(n) | Shifts 1M items |

This is why the crawler uses `set` for deduplication and `deque` for the frontier — both give O(1) for the operations that run on every single URL.

---

## Related Notes

- [[Notes/Resources/CS_Fundamentals/BFS_DFS|BFS and DFS]] — algorithms that run in O(n) over the graph
- [[Notes/Areas/WebCrawling/Concepts/Duplicate_Avoidance|Duplicate Avoidance]] — O(1) set lookup is what makes it cheap
- [[Notes/Areas/WebCrawling/Concepts/URL_Frontier|URL Frontier]] — deque vs list performance difference

## Open Questions

- What does O(log n) feel like in practice?
- What is space complexity vs time complexity?
- When does O(1) amortized differ from true O(1)?
