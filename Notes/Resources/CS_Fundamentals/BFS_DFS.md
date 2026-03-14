---
tags: [cs-fundamentals, algorithms, graph, search]
type: concept
area: learning
---

# BFS and DFS — Graph Traversal

BFS (Breadth-First Search) and DFS (Depth-First Search) are the two fundamental strategies for visiting every node in a graph or tree. In web crawling, pages are nodes and links are edges — choosing BFS vs DFS changes how the crawler explores the web.

---

## The Core Idea

Imagine you're exploring a building you've never been in:

- **BFS** — you check every room on floor 1 before going to floor 2. You spread outward level by level.
- **DFS** — you pick one hallway and follow it as far as it goes before backtracking. You go deep before going wide.

---

## BFS — Breadth-First Search

Uses a **queue** (FIFO — first in, first out). Process a node, then add all its neighbors to the back of the queue.

```
Start: A
Graph: A → [B, C], B → [D, E], C → [F]

Queue:  [A]
Visit A → add B, C
Queue:  [B, C]
Visit B → add D, E
Queue:  [C, D, E]
Visit C → add F
Queue:  [D, E, F]
Visit D, E, F...

Visit order: A → B → C → D → E → F
```

**In Python:** `collections.deque` + `popleft()`

```python
from collections import deque

def bfs(graph, start):
    visited = set()
    queue = deque([start])
    visited.add(start)

    while queue:
        node = queue.popleft()      # take from the FRONT
        print(node)

        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)  # add to the BACK
```

---

## DFS — Depth-First Search

Uses a **stack** (LIFO — last in, first out). Process a node, then immediately go deeper into one neighbor before handling others.

```
Start: A
Graph: A → [B, C], B → [D, E], C → [F]

Stack:  [A]
Visit A → add C, B (reversed so B is on top)
Stack:  [C, B]
Visit B → add E, D
Stack:  [C, E, D]
Visit D
Stack:  [C, E]
Visit E
Stack:  [C]
Visit C → add F
...

Visit order: A → B → D → E → C → F
```

**In Python:** `collections.deque` + `pop()` (or use recursion)

```python
from collections import deque

def dfs(graph, start):
    visited = set()
    stack = deque([start])

    while stack:
        node = stack.pop()          # take from the END (LIFO)
        if node in visited:
            continue
        visited.add(node)
        print(node)

        for neighbor in graph[node]:
            if neighbor not in visited:
                stack.append(neighbor)
```

---

## BFS vs DFS — Side by Side

| | BFS | DFS |
|---|---|---|
| Data structure | Queue (`deque.popleft()`) | Stack (`deque.pop()` or recursion) |
| Order | Level by level (wide) | Branch by branch (deep) |
| Finds shortest path? | **Yes** | No |
| Memory usage | Higher (stores whole frontier) | Lower (stores one path at a time) |
| Risk | Large memory on wide graphs | Stack overflow on deep graphs (recursion) |
| Web crawling | **Standard choice** | Bad — hammers one domain repeatedly |

---

## Why BFS for Web Crawling

BFS naturally spreads requests across many different domains:
- Depth 0: seed page (1 domain)
- Depth 1: pages linked from seed (many domains)
- Depth 2: pages linked from those (even more domains)

DFS would follow one link chain deep into a single site before ever visiting another domain — impolite and inefficient.

---

## Related Notes

- [[Notes/Areas/WebCrawling/Concepts/URL_Frontier|URL Frontier]] — where BFS/DFS is applied in a crawler
- [[Notes/Areas/WebCrawling/Concepts/Crawl_Depth|Crawl Depth]] — depth limit is a BFS concept
- [[Notes/Areas/WebCrawling/Python/URL_Frontier_Python|URL Frontier in Python]] — `deque.popleft()` = BFS, `deque.pop()` = DFS

## Open Questions

- When would DFS ever be preferable in crawling?
- What is bidirectional BFS and when is it used?
- How does BFS relate to finding the shortest path (Dijkstra's algorithm)?
