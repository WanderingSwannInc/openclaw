# Crawl4AI - LLM-Friendly Web Scraping

This skill guides the creation of web scrapers and crawlers using Crawl4AI, an open-source Python library optimized for extracting clean, structured data for Large Language Models, AI agents, and data pipelines.

## When to Use This Skill

Use this skill when the user wants to:
- Scrape websites and extract data (text, tables, media)
- Generate LLM-ready Markdown from web pages
- Build data extraction pipelines for RAG systems
- Handle dynamic JavaScript-heavy websites
- Extract structured JSON data using CSS/XPath selectors
- Use LLMs for intelligent content extraction
- Crawl multiple pages with session management
- Handle authentication and proxies
- Build adaptive crawlers that know when to stop

## Core Concepts

### What Makes Crawl4AI Different

**LLM-Optimized Output:**
- Generates clean Markdown perfect for RAG pipelines
- Strips noise and irrelevant content
- Preserves semantic structure
- Includes citations and references

**Multiple Extraction Strategies:**
- **No-LLM:** CSS/XPath selectors, regex patterns (fast, free, precise)
- **LLM-Based:** AI-powered extraction for complex/unstructured content
- **Cosine Similarity:** Semantic clustering for similar content blocks

**Async-First Design:**
- Built on Playwright for modern async crawling
- Parallel execution for multiple URLs
- Real-time performance

**Smart Features:**
- Adaptive crawling (stops when sufficient data gathered)
- Session management for multi-step workflows
- Custom hooks at every stage
- Proxy support and stealth modes

### Architecture Overview

```
AsyncWebCrawler
├── BrowserConfig         # Browser settings (headless, user agent, etc.)
├── CrawlerRunConfig      # Per-crawl settings (caching, extraction, etc.)
├── Extraction Strategies # How to extract data
│   ├── JsonCssExtractionStrategy
│   ├── JsonXPathExtractionStrategy
│   ├── RegexExtractionStrategy
│   ├── LLMExtractionStrategy
│   └── CosineStrategy
└── Hooks                 # Custom behavior at 8 lifecycle points
```

## Installation & Setup

### Basic Installation

```bash
# Install Crawl4AI
pip install crawl4ai --break-system-packages

# Setup browser (required)
crawl4ai-setup
```

**What `crawl4ai-setup` does:**
- Installs Playwright browsers (Chromium, Firefox, WebKit)
- Checks OS dependencies
- Verifies environment is ready

### Optional Dependencies

```bash
# For LLM extraction
pip install "crawl4ai[torch]" --break-system-packages

# For transformer-based features
pip install "crawl4ai[transformer]" --break-system-packages

# For cosine similarity
pip install "crawl4ai[cosine]" --break-system-packages

# Everything
pip install "crawl4ai[all]" --break-system-packages
```

### Verify Installation

```python
import asyncio
from crawl4ai import AsyncWebCrawler

async def test():
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun("https://example.com")
        print(result.markdown[:200])

asyncio.run(test())
```

## Basic Usage Patterns

### 1. Simple Page Crawl

```python
import asyncio
from crawl4ai import AsyncWebCrawler

async def simple_crawl():
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun("https://example.com")
        
        # Clean Markdown output
        print(result.markdown)
        
        # Raw HTML if needed
        print(result.html)
        
        # Extracted links
        print(result.links)
        
        # Media tags
        print(result.media)

asyncio.run(simple_crawl())
```

### 2. Crawl Multiple URLs

```python
async def crawl_multiple():
    urls = [
        "https://example.com/page1",
        "https://example.com/page2",
        "https://example.com/page3"
    ]
    
    async with AsyncWebCrawler() as crawler:
        results = await crawler.arun_many(urls)
        
        for result in results:
            if result.success:
                print(f"Success: {result.url}")
                print(result.markdown[:200])
            else:
                print(f"Failed: {result.url} - {result.error_message}")
```

### 3. Basic Configuration

```python
from crawl4ai import AsyncWebCrawler, BrowserConfig, CrawlerRunConfig

async def configured_crawl():
    # Browser settings
    browser_config = BrowserConfig(
        headless=True,
        verbose=True,
        user_agent="MyBot/1.0"
    )
    
    # Crawl settings
    run_config = CrawlerRunConfig(
        cache_mode="BYPASS",  # Don't use cache
        wait_for="body",      # Wait for body element
        delay_before_return_html=2.0  # Wait 2s before extraction
    )
    
    async with AsyncWebCrawler(config=browser_config) as crawler:
        result = await crawler.arun(
            url="https://example.com",
            config=run_config
        )
        print(result.markdown)
```

## Extraction Strategies

### No-LLM Strategies (Fast & Free)

#### CSS Selector Extraction

```python
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig
from crawl4ai.extraction_strategy import JsonCssExtractionStrategy

async def extract_with_css():
    # Define schema for extraction
    schema = {
        "name": "Product Listing",
        "baseSelector": "div.product-card",
        "fields": [
            {
                "name": "title",
                "selector": "h2.product-title",
                "type": "text"
            },
            {
                "name": "price",
                "selector": "span.price",
                "type": "text"
            },
            {
                "name": "image",
                "selector": "img",
                "type": "attribute",
                "attribute": "src"
            },
            {
                "name": "link",
                "selector": "a",
                "type": "attribute",
                "attribute": "href"
            }
        ]
    }
    
    strategy = JsonCssExtractionStrategy(schema)
    
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(
            url="https://example.com/products",
            extraction_strategy=strategy
        )
        
        # Parse extracted JSON
        import json
        products = json.loads(result.extracted_content)
        for product in products:
            print(f"{product['title']}: {product['price']}")
```

**Nested Fields Example:**

```python
schema = {
    "name": "Blog Post",
    "baseSelector": "article.post",
    "fields": [
        {
            "name": "title",
            "selector": "h1",
            "type": "text"
        },
        {
            "name": "author",
            "type": "nested",
            "selector": "div.author-info",
            "fields": [
                {"name": "name", "selector": "span.name", "type": "text"},
                {"name": "bio", "selector": "p.bio", "type": "text"}
            ]
        },
        {
            "name": "tags",
            "type": "list",
            "selector": "span.tag",
            "fields": [
                {"name": "tag", "selector": ".", "type": "text"}
            ]
        }
    ]
}
```

#### XPath Extraction

```python
from crawl4ai.extraction_strategy import JsonXPathExtractionStrategy

schema = {
    "name": "News Articles",
    "baseSelector": "//article[@class='news-item']",
    "fields": [
        {
            "name": "headline",
            "selector": ".//h2/text()",
            "type": "text"
        },
        {
            "name": "date",
            "selector": ".//time/@datetime",
            "type": "attribute"
        }
    ]
}

strategy = JsonXPathExtractionStrategy(schema)
```

#### Regex Extraction

```python
from crawl4ai.extraction_strategy import RegexExtractionStrategy

async def extract_patterns():
    # Extract emails, phones, URLs, dates
    strategy = RegexExtractionStrategy()
    
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(
            url="https://example.com/contact",
            extraction_strategy=strategy
        )
        
        data = json.loads(result.extracted_content)
        print("Emails:", data.get("emails", []))
        print("Phones:", data.get("phones", []))
        print("URLs:", data.get("urls", []))
```

### LLM-Based Strategies (Intelligent)

#### Basic LLM Extraction

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai import LLMConfig
import os

async def llm_extraction():
    # Configure LLM
    llm_config = LLMConfig(
        provider="openai/gpt-4o-mini",
        api_token=os.getenv("OPENAI_API_KEY")
    )
    
    # Define what to extract
    strategy = LLMExtractionStrategy(
        llm_config=llm_config,
        instruction="""
        Extract all product information from this page.
        Return a JSON array with objects containing:
        - name: product name
        - price: price as a number
        - description: brief description
        - availability: in stock or out of stock
        """,
        extraction_type="schema"
    )
    
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(
            url="https://example.com/products",
            extraction_strategy=strategy
        )
        
        products = json.loads(result.extracted_content)
        print(products)
```

#### LLM with Pydantic Schema

```python
from pydantic import BaseModel
from typing import List

class Product(BaseModel):
    name: str
    price: float
    description: str
    in_stock: bool

class ProductPage(BaseModel):
    products: List[Product]
    total_count: int

async def structured_llm_extraction():
    llm_config = LLMConfig(
        provider="openai/gpt-4o-mini",
        api_token=os.getenv("OPENAI_API_KEY")
    )
    
    strategy = LLMExtractionStrategy(
        llm_config=llm_config,
        schema=ProductPage.model_json_schema(),
        instruction="Extract all products from the page",
        extraction_type="schema"
    )
    
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(
            url="https://example.com/shop",
            extraction_strategy=strategy
        )
        
        data = ProductPage.model_validate_json(result.extracted_content)
        for product in data.products:
            print(f"{product.name}: ${product.price}")
```

#### LLM Chunking (for large pages)

```python
strategy = LLMExtractionStrategy(
    llm_config=llm_config,
    instruction="Extract article content",
    chunk_token_threshold=3000,  # Split into 3k token chunks
    overlap_rate=0.1,            # 10% overlap between chunks
    apply_chunking=True
)
```

### Cosine Similarity Strategy

```python
from crawl4ai.extraction_strategy import CosineStrategy

async def semantic_extraction():
    # Extract content based on semantic similarity
    strategy = CosineStrategy(
        semantic_filter="product reviews and ratings",
        word_count_threshold=20,   # Min words per cluster
        sim_threshold=0.4,         # Similarity threshold
        max_dist=0.3,              # Max distance for clustering
        top_k=10                   # Return top 10 matches
    )
    
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(
            url="https://example.com/reviews",
            extraction_strategy=strategy
        )
        
        reviews = json.loads(result.extracted_content)
        print(f"Found {len(reviews)} relevant review sections")
```

## Dynamic Content & JavaScript

### Execute JavaScript

```python
async def execute_js():
    # JavaScript to run after page load
    js_code = """
    // Scroll to bottom
    window.scrollTo(0, document.body.scrollHeight);
    
    // Click "Load More" button
    const btn = document.querySelector('.load-more');
    if (btn) btn.click();
    
    // Wait for content
    await new Promise(r => setTimeout(r, 2000));
    """
    
    run_config = CrawlerRunConfig(
        js_code=js_code,
        wait_for="div.content-loaded"  # Wait for this selector
    )
    
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(
            url="https://example.com/dynamic",
            config=run_config
        )
        print(result.markdown)
```

### Wait Conditions

```python
run_config = CrawlerRunConfig(
    # Wait for selector
    wait_for="div#data-loaded",
    
    # Or wait for JavaScript condition
    wait_for="() => document.querySelectorAll('.item').length > 10",
    
    # Delay before extraction
    delay_before_return_html=3.0
)
```

## Session Management

### Multi-Page Workflows

```python
async def session_crawl():
    async with AsyncWebCrawler() as crawler:
        session_id = "my_session"
        
        # First page
        result1 = await crawler.arun(
            url="https://example.com/page1",
            session_id=session_id
        )
        
        # Click next, reuse session
        js_next_page = "document.querySelector('.next').click();"
        result2 = await crawler.arun(
            url="https://example.com/page1",  # Same URL
            session_id=session_id,
            config=CrawlerRunConfig(js_code=js_next_page)
        )
        
        # Close session
        await crawler.kill_session(session_id)
```

### Pagination Example

```python
async def paginate():
    async with AsyncWebCrawler() as crawler:
        session_id = "pagination"
        all_items = []
        
        for page in range(5):  # Crawl 5 pages
            js = f"window.location.href = 'https://example.com/items?page={page+1}';"
            
            result = await crawler.arun(
                url=f"https://example.com/items?page={page+1}",
                session_id=session_id,
                config=CrawlerRunConfig(
                    js_code=js if page > 0 else None,
                    extraction_strategy=JsonCssExtractionStrategy(schema)
                )
            )
            
            items = json.loads(result.extracted_content)
            all_items.extend(items)
            print(f"Page {page+1}: {len(items)} items")
        
        await crawler.kill_session(session_id)
        print(f"Total: {len(all_items)} items")
```

## Hooks & Authentication

### Hook Lifecycle

Crawl4AI provides 8 hooks for custom behavior:

1. **on_browser_created** - Browser instance created
2. **on_page_context_created** - New page/context created (best for auth)
3. **before_goto** - Before navigating to URL
4. **after_goto** - After navigation completes
5. **on_user_agent_updated** - User agent changed
6. **on_execution_started** - Custom JS starting
7. **before_retrieve_html** - Before HTML snapshot
8. **before_return_html** - Before returning result

### Authentication Example

```python
from playwright.async_api import Page, BrowserContext

async def crawl_with_auth():
    async def on_page_context_created(page: Page, context: BrowserContext, **kwargs):
        """Login before crawling"""
        # Set cookies
        await context.add_cookies([
            {
                "name": "session_token",
                "value": "your-token-here",
                "domain": "example.com",
                "path": "/"
            }
        ])
        
        # Or set localStorage
        await page.goto("https://example.com")
        await page.evaluate("""
            localStorage.setItem('auth_token', 'your-token');
        """)
        
        return page
    
    browser_config = BrowserConfig(
        headless=True,
        extra_args=["--disable-blink-features=AutomationControlled"]
    )
    
    async with AsyncWebCrawler(config=browser_config) as crawler:
        # Attach hook
        crawler.crawler_strategy.set_hook(
            "on_page_context_created",
            on_page_context_created
        )
        
        result = await crawler.arun("https://example.com/protected")
        print(result.markdown)
```

### Form Login Example

```python
async def login_hook(page: Page, context: BrowserContext, **kwargs):
    """Fill login form"""
    await page.goto("https://example.com/login")
    await page.fill("#username", "myuser")
    await page.fill("#password", "mypass")
    await page.click("button[type=submit]")
    await page.wait_for_selector("#dashboard")  # Wait for redirect
    return page
```

### Block Resources Hook

```python
async def block_images(page: Page, context: BrowserContext, **kwargs):
    """Block images/videos to speed up crawling"""
    await context.route(
        "**/*.{png,jpg,jpeg,gif,svg,mp4,webm}",
        lambda route: route.abort()
    )
    return page
```

## Adaptive Crawling

```python
from crawl4ai import AdaptiveCrawler

async def adaptive_crawl():
    async with AsyncWebCrawler(verbose=True) as crawler:
        adaptive = AdaptiveCrawler(crawler)
        
        # Crawl until sufficient information gathered
        result = await adaptive.digest(
            start_url="https://docs.python.org/3/library/asyncio.html",
            query="async await context managers coroutines"
        )
        
        # View statistics
        adaptive.print_stats()
        
        # Get most relevant pages
        relevant = adaptive.get_relevant_content(top_k=5)
        for i, page in enumerate(relevant, 1):
            print(f"{i}. {page['url']}")
            print(f"   Score: {page['score']:.2%}")
            print(f"   Preview: {page['content'][:200]}")
        
        print(f"\nConfidence: {adaptive.confidence:.2%}")
        print(f"Pages crawled: {len(result.crawled_urls)}")
```

**What Adaptive Crawling Does:**
- Automatically determines when enough data is collected
- Follows only relevant links
- Stops when confidence threshold reached
- Provides confidence/coverage/consistency metrics

## Proxies & Stealth

### Using Proxies

```python
browser_config = BrowserConfig(
    headless=True,
    proxy="http://proxy.example.com:8080",
    proxy_config={
        "server": "http://proxy.example.com:8080",
        "username": "user",
        "password": "pass"
    }
)

async with AsyncWebCrawler(config=browser_config) as crawler:
    result = await crawler.arun("https://example.com")
```

### Stealth Mode

```python
browser_config = BrowserConfig(
    headless=True,
    user_agent="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36",
    extra_args=[
        "--disable-blink-features=AutomationControlled",
        "--disable-dev-shm-usage",
        "--no-sandbox"
    ]
)
```

### Rotating User Agents

```python
import random

user_agents = [
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) ...",
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) ...",
    "Mozilla/5.0 (X11; Linux x86_64) ..."
]

browser_config = BrowserConfig(
    user_agent=random.choice(user_agents)
)
```

## Advanced Patterns

### Deep Crawling

```python
from crawl4ai.async_configs import CrawlerRunConfig

async def deep_crawl():
    run_config = CrawlerRunConfig(
        # Follow links
        max_depth=2,  # Crawl 2 levels deep
        
        # Stay on same domain
        exclude_external_links=True,
        
        # Link filtering
        exclude_patterns=[
            r".*\.(jpg|png|gif|pdf)$",  # Skip media
            r".*/admin/.*"               # Skip admin pages
        ]
    )
    
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(
            url="https://example.com/docs",
            config=run_config
        )
        
        print(f"Crawled {len(result.links['internal'])} internal links")
```

### Screenshot Capture

```python
run_config = CrawlerRunConfig(
    screenshot=True,
    screenshot_wait_for="div.content"  # Wait before screenshot
)

async with AsyncWebCrawler() as crawler:
    result = await crawler.arun(
        url="https://example.com",
        config=run_config
    )
    
    # Save screenshot
    with open("screenshot.png", "wb") as f:
        f.write(result.screenshot)
```

### Table Extraction

```python
from crawl4ai.extraction_strategy import DefaultTableExtraction

async def extract_tables():
    strategy = DefaultTableExtraction(
        table_score_threshold=0.5  # Quality threshold
    )
    
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(
            url="https://example.com/data",
            extraction_strategy=strategy
        )
        
        tables = json.loads(result.extracted_content)
        for i, table in enumerate(tables):
            print(f"Table {i+1}:")
            print(table["markdown"])  # Markdown representation
```

### Caching

```python
from crawl4ai.async_configs import CacheMode

# Use cache (default)
run_config = CrawlerRunConfig(cache_mode=CacheMode.ENABLED)

# Bypass cache
run_config = CrawlerRunConfig(cache_mode=CacheMode.BYPASS)

# Write-only cache
run_config = CrawlerRunConfig(cache_mode=CacheMode.WRITE_ONLY)

# Read-only cache
run_config = CrawlerRunConfig(cache_mode=CacheMode.READ_ONLY)
```

## Error Handling & Best Practices

### Robust Error Handling

```python
async def safe_crawl(url: str):
    try:
        async with AsyncWebCrawler(verbose=True) as crawler:
            result = await crawler.arun(url)
            
            if not result.success:
                print(f"Crawl failed: {result.error_message}")
                return None
            
            return result.markdown
            
    except Exception as e:
        print(f"Exception occurred: {e}")
        return None
```

### Rate Limiting

```python
import asyncio

async def crawl_with_delays(urls: list):
    async with AsyncWebCrawler() as crawler:
        for url in urls:
            result = await crawler.arun(url)
            print(f"Crawled: {url}")
            
            # Wait between requests
            await asyncio.sleep(2)  # 2 second delay
```

### Retry Logic

```python
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=2, max=10)
)
async def crawl_with_retry(url: str):
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(url)
        if not result.success:
            raise Exception(f"Crawl failed: {result.error_message}")
        return result
```

### Resource Cleanup

```python
async def proper_cleanup():
    crawler = None
    try:
        crawler = AsyncWebCrawler()
        await crawler.__aenter__()
        
        result = await crawler.arun("https://example.com")
        return result
        
    finally:
        if crawler:
            await crawler.__aexit__(None, None, None)
```

## Common Use Cases

### 1. E-commerce Product Scraper

```python
async def scrape_products():
    schema = {
        "name": "Products",
        "baseSelector": "div.product",
        "fields": [
            {"name": "name", "selector": "h3.title", "type": "text"},
            {"name": "price", "selector": "span.price", "type": "text"},
            {"name": "rating", "selector": "div.rating", "type": "attribute", "attribute": "data-rating"},
            {"name": "image", "selector": "img", "type": "attribute", "attribute": "src"}
        ]
    }
    
    strategy = JsonCssExtractionStrategy(schema)
    
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(
            url="https://shop.example.com/products",
            extraction_strategy=strategy
        )
        
        products = json.loads(result.extracted_content)
        return products
```

### 2. News Article Aggregator

```python
async def scrape_news():
    llm_config = LLMConfig(
        provider="openai/gpt-4o-mini",
        api_token=os.getenv("OPENAI_API_KEY")
    )
    
    strategy = LLMExtractionStrategy(
        llm_config=llm_config,
        instruction="""
        Extract all news articles with:
        - headline
        - summary (1-2 sentences)
        - author
        - publish_date
        - category
        Return as JSON array.
        """
    )
    
    urls = [
        "https://news1.com",
        "https://news2.com",
        "https://news3.com"
    ]
    
    async with AsyncWebCrawler() as crawler:
        results = await crawler.arun_many(
            urls,
            config=CrawlerRunConfig(extraction_strategy=strategy)
        )
        
        all_articles = []
        for result in results:
            if result.success:
                articles = json.loads(result.extracted_content)
                all_articles.extend(articles)
        
        return all_articles
```

### 3. Documentation Crawler for RAG

```python
async def crawl_docs_for_rag():
    async with AsyncWebCrawler() as crawler:
        adaptive = AdaptiveCrawler(crawler)
        
        result = await adaptive.digest(
            start_url="https://docs.example.com",
            query="API authentication authorization methods"
        )
        
        # Get clean Markdown for each page
        documents = []
        for url in result.crawled_urls:
            page_result = await crawler.arun(url)
            documents.append({
                "url": url,
                "content": page_result.markdown,
                "metadata": {
                    "title": page_result.metadata.get("title", ""),
                    "description": page_result.metadata.get("description", "")
                }
            })
        
        return documents
```

### 4. Social Media Monitor

```python
async def monitor_social():
    session_id = "social_monitor"
    
    async def check_updates(page: Page, context: BrowserContext, **kwargs):
        # Login logic here
        await page.goto("https://social.example.com/login")
        await page.fill("#email", os.getenv("SOCIAL_EMAIL"))
        await page.fill("#password", os.getenv("SOCIAL_PASS"))
        await page.click("button[type=submit]")
        await page.wait_for_selector("#feed")
        return page
    
    async with AsyncWebCrawler() as crawler:
        crawler.crawler_strategy.set_hook(
            "on_page_context_created",
            check_updates
        )
        
        while True:
            result = await crawler.arun(
                url="https://social.example.com/feed",
                session_id=session_id
            )
            
            # Process new posts
            print(f"Checked at {datetime.now()}")
            
            # Wait 5 minutes
            await asyncio.sleep(300)
```

## Best Practices

### 1. Choose the Right Extraction Strategy

**Use CSS/XPath when:**
- Page structure is consistent
- Data is well-structured (tables, lists, cards)
- Speed is critical
- Cost is a concern

**Use LLM when:**
- Content is unstructured or varies
- Need semantic understanding
- Require summarization or transformation
- Complex reasoning needed

**Use Regex when:**
- Extracting specific patterns (emails, phones, dates)
- Simple, fast extraction
- Known data formats

**Use Cosine Similarity when:**
- Need semantic clustering
- Finding similar content blocks
- Topic-based filtering
- Structure is inconsistent

### 2. Optimize Performance

```python
# Parallel crawling
results = await crawler.arun_many(
    urls,
    config=CrawlerRunConfig(
        # Block unnecessary resources
        block_resources=["image", "media", "font"]
    )
)

# Use caching
run_config = CrawlerRunConfig(
    cache_mode=CacheMode.ENABLED
)

# Minimize wait times
run_config = CrawlerRunConfig(
    page_timeout=30000,  # 30 seconds max
    delay_before_return_html=1.0  # Minimal delay
)
```

### 3. Handle Dynamic Content

```python
# For infinite scroll
js_scroll = """
let lastHeight = document.body.scrollHeight;
while (true) {
    window.scrollTo(0, document.body.scrollHeight);
    await new Promise(r => setTimeout(r, 2000));
    let newHeight = document.body.scrollHeight;
    if (newHeight === lastHeight) break;
    lastHeight = newHeight;
}
"""

# For lazy loading
run_config = CrawlerRunConfig(
    js_code=js_scroll,
    wait_for="div.content-loaded",
    delay_before_return_html=3.0
)
```

### 4. Respect Websites

```python
# Add delays
await asyncio.sleep(2)

# Use appropriate user agent
browser_config = BrowserConfig(
    user_agent="YourBot/1.0 (+https://yoursite.com/bot)"
)

# Check robots.txt
# Implement rate limiting
# Honor website terms of service
```

### 5. Error Recovery

```python
async def resilient_crawl(urls: list):
    results = []
    failed = []
    
    async with AsyncWebCrawler() as crawler:
        for url in urls:
            try:
                result = await crawler.arun(url)
                if result.success:
                    results.append(result)
                else:
                    failed.append((url, result.error_message))
            except Exception as e:
                failed.append((url, str(e)))
                await asyncio.sleep(5)  # Back off on error
    
    # Retry failed
    if failed:
        print(f"Retrying {len(failed)} failed URLs...")
        # Implement retry logic
    
    return results, failed
```

## Debugging & Troubleshooting

### Verbose Logging

```python
browser_config = BrowserConfig(
    headless=False,  # See browser window
    verbose=True     # Detailed logs
)

async with AsyncWebCrawler(config=browser_config) as crawler:
    result = await crawler.arun("https://example.com")
```

### Common Issues

**Issue: Page not loading completely**
```python
# Solution: Increase wait time
run_config = CrawlerRunConfig(
    wait_for="div.content",
    delay_before_return_html=5.0,
    page_timeout=60000  # 60 seconds
)
```

**Issue: JavaScript not executing**
```python
# Solution: Verify JS syntax and timing
run_config = CrawlerRunConfig(
    js_code="console.log('test'); await new Promise(r => setTimeout(r, 2000));",
    wait_for="() => document.querySelector('.loaded') !== null"
)
```

**Issue: Authentication failing**
```python
# Solution: Use correct hook timing
async def auth_hook(page: Page, context: BrowserContext, **kwargs):
    # Always use on_page_context_created for auth
    await context.add_cookies([...])
    return page

crawler.crawler_strategy.set_hook("on_page_context_created", auth_hook)
```

**Issue: Getting blocked/detected**
```python
# Solution: Use stealth techniques
browser_config = BrowserConfig(
    headless=True,
    user_agent="Mozilla/5.0...",
    extra_args=[
        "--disable-blink-features=AutomationControlled",
        "--disable-dev-shm-usage"
    ]
)
```

## Testing Crawlers

```python
import pytest

@pytest.mark.asyncio
async def test_product_extraction():
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(
            url="https://example.com/products",
            extraction_strategy=product_strategy
        )
        
        assert result.success
        products = json.loads(result.extracted_content)
        assert len(products) > 0
        assert "name" in products[0]
        assert "price" in products[0]

@pytest.mark.asyncio
async def test_handles_errors():
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun("https://invalid-url-that-does-not-exist.com")
        assert not result.success
        assert result.error_message is not None
```

## Complete Example: Full Scraper

```python
import asyncio
import json
from datetime import datetime
from crawl4ai import AsyncWebCrawler, BrowserConfig, CrawlerRunConfig
from crawl4ai.extraction_strategy import JsonCssExtractionStrategy

class ProductScraper:
    def __init__(self):
        self.browser_config = BrowserConfig(
            headless=True,
            verbose=False,
            user_agent="ProductScraper/1.0"
        )
        
        self.schema = {
            "name": "Products",
            "baseSelector": "div.product-card",
            "fields": [
                {"name": "title", "selector": "h2", "type": "text"},
                {"name": "price", "selector": "span.price", "type": "text"},
                {"name": "rating", "selector": "div.rating", "type": "attribute", "attribute": "data-rating"},
                {"name": "url", "selector": "a", "type": "attribute", "attribute": "href"},
                {"name": "image", "selector": "img", "type": "attribute", "attribute": "src"}
            ]
        }
    
    async def scrape_page(self, url: str):
        """Scrape a single page"""
        strategy = JsonCssExtractionStrategy(self.schema)
        
        async with AsyncWebCrawler(config=self.browser_config) as crawler:
            result = await crawler.arun(
                url=url,
                extraction_strategy=strategy,
                config=CrawlerRunConfig(cache_mode="BYPASS")
            )
            
            if not result.success:
                print(f"Failed to scrape {url}: {result.error_message}")
                return []
            
            return json.loads(result.extracted_content)
    
    async def scrape_multiple_pages(self, base_url: str, num_pages: int):
        """Scrape multiple pages with pagination"""
        all_products = []
        
        for page_num in range(1, num_pages + 1):
            url = f"{base_url}?page={page_num}"
            print(f"Scraping page {page_num}...")
            
            products = await self.scrape_page(url)
            all_products.extend(products)
            
            # Rate limiting
            await asyncio.sleep(2)
        
        return all_products
    
    def save_to_json(self, products: list, filename: str):
        """Save products to JSON file"""
        data = {
            "timestamp": datetime.now().isoformat(),
            "total_products": len(products),
            "products": products
        }
        
        with open(filename, "w") as f:
            json.dump(data, f, indent=2)
        
        print(f"Saved {len(products)} products to {filename}")

async def main():
    scraper = ProductScraper()
    
    # Scrape 5 pages
    products = await scraper.scrape_multiple_pages(
        "https://example.com/products",
        num_pages=5
    )
    
    # Save results
    scraper.save_to_json(products, "products.json")
    
    print(f"Total products scraped: {len(products)}")

if __name__ == "__main__":
    asyncio.run(main())
```

## Conclusion

Crawl4AI is a powerful, flexible tool for web scraping and data extraction. Key takeaways:

1. **Start simple** - Basic crawls with `arun()` for single pages
2. **Choose the right strategy** - CSS/XPath for structured data, LLM for complex content
3. **Use sessions** - For multi-page workflows and authentication
4. **Leverage hooks** - Custom behavior at every stage
5. **Optimize** - Cache, parallel execution, resource blocking
6. **Handle errors** - Robust error handling and retry logic
7. **Respect websites** - Rate limiting, delays, proper user agents

Crawl4AI excels at:
- Generating LLM-ready Markdown
- Extracting structured data without LLMs
- Handling dynamic JavaScript content
- Adaptive crawling that knows when to stop
- Session management for complex workflows

Perfect for building RAG pipelines, data extraction systems, monitoring tools, and AI-powered scrapers.
