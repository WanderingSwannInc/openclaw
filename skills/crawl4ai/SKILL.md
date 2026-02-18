---
name: crawl4ai
description: Build and run web crawlers with Crawl4AI for scraping pages, extracting structured data, and producing clean markdown/JSON for RAG pipelines. Use when users ask to crawl websites, handle JS-heavy pages, paginate, authenticate, or define extraction schemas (CSS/XPath/LLM/regex).
---

# Crawl4AI

Use this skill to implement Crawl4AI workflows quickly and safely.

## Workflow
1. Confirm target URLs, allowed scope, and required output format.
2. Choose extraction strategy:
   - CSS/XPath for stable structured pages
   - Regex for simple patterns
   - LLM extraction for unstructured pages
3. Start with a minimal async crawler, then add config for JS rendering, waits, pagination, and sessions.
4. Add error handling, retries, and rate limits.
5. Return structured output and a reproducible script.

## Defaults
- Prefer no-LLM extraction first for speed/cost.
- Use explicit selectors/schema when possible.
- Respect robots/ToS and avoid aggressive request rates.

## References
- Detailed patterns and examples: `references/crawl4ai-playbook.md`
