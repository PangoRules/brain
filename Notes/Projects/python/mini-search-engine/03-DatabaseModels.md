---
title: Mini Search Engine — Database Models
status: active
tags: [project]
---

> **Navigation:** [[00-MiniSearchEngine|← 00 Overview]] → [[01-Features]] → [[02-RecommendedStructure]] → 03 DB Models → [[04-Roadmap]]

### Core tables

**documents**
- `id`
- `url`
- `normalized_url`
- `title`
- `content_text`
- `status_code`
- `content_hash`
- `fetched_at`
- `last_crawled_at`

**links**
- `from_document_id`
- `to_url`
- `to_normalized_url`
- maybe later `to_document_id`

**terms**
- `id`
- `term`

**postings**
- `term_id`
- `document_id`
- `tf`
- maybe positions

**crawl_queue**
- `url`
- `normalized_url`
- `depth`
- `status` (`pending`, `processing`, `done`, `failed`)
- `discovered_at`
- `next_attempt_at`

**page_rank**
- `document_id`
- `score`

That already gives you a real search engine core.