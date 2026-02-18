# Crawl4AI Skill - LLM-Friendly Web Scraping

A comprehensive skill for building web scrapers using Crawl4AI, an open-source Python library optimized for LLMs and AI applications.

## What This Skill Does

This skill helps Claude build web scrapers and data extraction systems using Crawl4AI. It covers:

**Core Features:**
- Basic to advanced web scraping with async/await
- Multiple extraction strategies (CSS, XPath, Regex, LLM, Cosine Similarity)
- Dynamic content handling (JavaScript execution, wait conditions)
- Session management for multi-page workflows
- Authentication and hooks system
- Adaptive crawling (auto-stop when enough data gathered)
- Proxies, stealth modes, and anti-detection
- Error handling and production patterns

**Use Cases:**
- E-commerce product scraping
- News aggregation for RAG systems  
- Documentation crawling
- Social media monitoring
- Data pipeline building
- LLM training data collection

## Key Strengths

### 1. Multiple Extraction Strategies
- **No-LLM:** Fast, free, precise (CSS, XPath, Regex)
- **LLM-Based:** Intelligent, handles unstructured content
- **Cosine Similarity:** Semantic clustering and filtering
- **Table Extraction:** Specialized for tabular data

### 2. Async-First & Fast
- Built on Playwright for modern async crawling
- Parallel execution with `arun_many()`
- Automatic concurrency management
- Real-time performance

### 3. LLM-Friendly Output
- Clean Markdown perfect for RAG
- Structured JSON extraction
- Preserves semantic meaning
- Citations and references

### 4. Advanced Browser Control
- 8 lifecycle hooks for custom behavior
- Session management for multi-step workflows
- JavaScript execution
- Proxy and stealth support

### 5. Production-Ready
- Comprehensive error handling patterns
- Retry logic and rate limiting
- Caching strategies
- Resource cleanup

## Skill Coverage

### Installation & Setup
- Package installation and browser setup
- Optional dependencies (torch, transformers, cosine)
- Environment verification

### Basic Patterns
- Simple page crawling
- Multiple URL crawling
- Configuration (BrowserConfig, CrawlerRunConfig)
- Result object structure

### Extraction Strategies
- **CSS Selectors:** Structured data extraction with schemas
- **XPath:** Alternative selector strategy
- **Regex:** Pattern-based extraction (emails, phones, URLs)
- **LLM:** AI-powered extraction with Pydantic schemas
- **Cosine:** Semantic content clustering
- **Tables:** Specialized table extraction

### Dynamic Content
- JavaScript execution
- Wait conditions
- Lazy loading
- Infinite scroll
- Dynamic buttons

### Session Management
- Multi-page workflows
- Pagination handling
- State preservation
- Session cleanup

### Hooks & Auth
- 8 lifecycle hooks explained
- Authentication patterns (cookies, forms, tokens)
- Resource blocking for performance
- Custom behavior injection

### Advanced Features
- Adaptive crawling (auto-stop)
- Deep crawling with link following
- Screenshot capture
- Proxy configuration
- Stealth techniques
- Caching strategies

### Best Practices
- Strategy selection guide
- Performance optimization
- Error handling patterns
- Rate limiting
- Testing approaches

### Complete Examples
- E-commerce scraper
- News aggregator
- Documentation crawler
- Social media monitor
- Production-ready scraper class

## Evaluation Tests

12 test prompts covering:
1. Simple basic crawl
2. CSS selector extraction
3. LLM-based extraction
4. Dynamic content (Load More button)
5. Pagination with sessions
6. Authentication with hooks
7. Regex pattern extraction
8. Parallel crawling with error handling
9. Strategy comparison/explanation
10. Adaptive crawling
11. Stealth techniques
12. Production-ready class implementation

## What Makes This Skill Strong

1. **Comprehensive Coverage** - From hello world to production patterns
2. **Multiple Strategies** - Right tool for each job (CSS vs LLM vs Regex)
3. **Real Examples** - Working code for common use cases
4. **Best Practices** - Performance, errors, rate limiting, testing
5. **LLM-Focused** - Optimized for RAG, agents, and data pipelines
6. **Production-Ready** - Error handling, retries, resource management

## When to Use Each Strategy

| Strategy | When to Use | Pros | Cons |
|----------|-------------|------|------|
| **CSS/XPath** | Structured, consistent pages | Fast, free, precise | Requires page structure knowledge |
| **Regex** | Known patterns (emails, phones) | Very fast, simple | Limited to patterns |
| **LLM** | Unstructured, varying content | Intelligent, flexible | Slow, costs money |
| **Cosine** | Semantic similarity needed | Good for clustering | Requires transformer models |

## Next Steps

1. **Test the skill** - Run evals to measure performance
2. **Iterate** - Refine based on eval results
3. **Add examples** - More use case examples if needed
4. **Deploy** - Move to `/mnt/skills/user/crawl4ai/` when ready

## Usage

Place the skill where Claude can access it. Claude will use it when users mention:
- Web scraping
- Data extraction
- Crawl4AI specifically
- LLM data pipelines
- RAG content collection
- Website data gathering

The skill provides guidance from basic setup through production deployment.
