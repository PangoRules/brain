---
title: Mini Search Engine
status: active
start: 2026-02-26
tags: [project]
---

> **Navigation:** 00 Overview → [[01-Features]] → [[02-RecommendedStructure]] → [[03-DatabaseModels]] → [[04-Roadmap]]

A mini search engine will let me understand:

- crawling
- parsing
- indexing
- ranking
- query processing
- backend API design
- pagination
- rate limiting
- systems thinking

- **Crawler**
    Starts from seed URLs, fetches pages, respects `robots.txt`, normalizes URLs, extracts links, and stores raw page data. The Robots Exclusion Protocol is defined in RFC 9309, and it explicitly says robots rules are not access authorization, but are rules crawlers are requested to honor. → See [[Notes/Areas/WebCrawling/WebCrawling|Web Crawling concepts]]
    
- **Indexer**  
    Takes crawled pages, strips HTML, extracts title/body/headings, tokenizes text, applies normalization, and builds an inverted index.
    
- **Ranker**  
    Scores results using something simple but defendable:
    - TF-IDF or BM25 for text relevance
    - plus a tiny PageRank-style link score  
        Lucene documents BM25 as its optimized implementation of the Okapi BM25 model, so BM25 is a totally legitimate “basic but real” ranking choice.
        
- **Search API**  
    Accepts query params like:
    - `q`
    - `page`
    - `page_size`
    - maybe `site`
    - maybe `sort`
        
- **Web UI**  
    Search box + results page + pagination.
    
- **Operational concerns**
    - rate limiting
    - crawl politeness
    - deduplication
    - retry rules
    - logging
    - error handling

---

## NLP Concepts Applied

| Component | Concept | Implementation |
|-----------|---------|---------------|
| Indexer tokenizes text | [[Notes/Areas/NLP/Concepts/Tokenization\|Tokenization]] | [[Notes/Areas/NLP/Python/Tokenization_in_Python\|Tokenization in Python]] |
| Indexer filters noise | [[Notes/Areas/NLP/Concepts/Stop_Words\|Stop Words]] | [[Notes/Areas/NLP/Python/Removing_Stop_Words_Python\|Removing Stop Words]] |
| Indexer normalizes terms | [[Notes/Areas/NLP/Concepts/Stemming\|Stemming]] | [[Notes/Areas/NLP/Python/Stemming_Words_NLTK\|Stemming with NLTK]] |
| Ranker scores relevance | [[Notes/Areas/NLP/Concepts/TF_IDF\|TF-IDF]] | [[Notes/Areas/NLP/Python/TF_IDF_Sklearn\|TF-IDF with scikit-learn]] |
| Indexer builds lookup | [[Notes/Areas/NLP/Concepts/Inverted_Index\|Inverted Index]] | [[Notes/Areas/NLP/Python/Inverted_Index_Python\|Inverted Index in Python]] |
| Indexer cleans text | [[Notes/Areas/NLP/Concepts/Normalizing_Text\|Text Normalization]] | [[Notes/Areas/NLP/Python/Normalizing_Textual_Data_in_Python\|Normalizing Text in Python]] |