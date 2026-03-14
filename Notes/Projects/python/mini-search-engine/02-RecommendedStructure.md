---
title: Mini Search Engine — Recommended Structure
status: active
tags: [project]
---

> **Navigation:** [[00-MiniSearchEngine|← 00 Overview]] → [[01-Features]] → 02 Structure → [[03-DatabaseModels]] → [[04-Roadmap]]

```text
mini-search-engine/
├─ README.md
├─ docker-compose.yml
├─ pyproject.toml
├─ .env.example
├─ app/
│  ├─ main.py
│  ├─ config.py
│  ├─ api/
│  │  ├─ routes_search.py
│  │  ├─ routes_admin.py
│  │  └─ dependencies.py
│  ├─ crawler/
│  │  ├─ fetcher.py
│  │  ├─ robots.py
│  │  ├─ parser.py
│  │  ├─ normalizer.py
│  │  ├─ frontier.py
│  │  └─ scheduler.py
│  ├─ indexer/
│  │  ├─ tokenizer.py
│  │  ├─ text_extractor.py
│  │  ├─ analyzer.py
│  │  ├─ inverted_index.py
│  │  └─ linker.py
│  ├─ ranking/
│  │  ├─ tfidf.py
│  │  ├─ bm25.py
│  │  ├─ pagerank.py
│  │  └─ scorer.py
│  ├─ search/
│  │  ├─ query_parser.py
│  │  ├─ snippets.py
│  │  └─ service.py
│  ├─ db/
│  │  ├─ models.py
│  │  ├─ session.py
│  │  └─ migrations/
│  ├─ core/
│  │  ├─ logging.py
│  │  ├─ rate_limit.py
│  │  └─ utils.py
│  └─ templates/
│     ├─ search.html
│     └─ results.html
├─ scripts/
│  ├─ seed_crawl.py
│  ├─ run_index.py
│  └─ recalc_pagerank.py
├─ tests/
│  ├─ test_normalizer.py
│  ├─ test_tokenizer.py
│  ├─ test_indexer.py
│  ├─ test_ranker.py
│  └─ test_search_api.py
└─ data/
   └─ search.db
```

Why this structure works:
- `crawler/` teaches distributed-systems style thinking
- `indexer/` isolates IR logic
- `ranking/` makes your scoring explainable
- `search/` keeps query-time logic separate from crawl-time logic
- `scripts/` gives you clean entry points
- `tests/` makes it professional