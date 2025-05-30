```markdown
# Detailed Outline for crawl4ai - vibe Component

**Target Document Type:** reasoning
**Target Output Filename Suggestion:** `llm_reasoning_vibe.md`
**Library Version Context:** 0.6.3
**Outline Generation Date:** 2025-05-24
---

# Vibe Coding with Crawl4AI: A Guide for AI Coding Assistants

## 1. Introduction: What is "Vibe Coding" with Crawl4AI?

*   **1.1. Explaining the Core Idea:**
    "Vibe Coding" with Crawl4AI is about empowering users, even those with limited direct coding experience, to build powerful web data applications. Instead of meticulously writing every line of Python, you focus on clearly describing your data extraction or web interaction goals to an AI coding assistant. This guide teaches you how to provide that AI assistant with the *right context* about Crawl4AI, so it can generate effective and efficient code for you. The goal is to translate your "vibe" â€“ your high-level intent â€“ into working Crawl4AI solutions quickly.

*   **1.2. Who is this Guide For?**
    This guide is designed for:
    *   **Users new to web scraping or `crawl4ai`:** If you prefer to articulate your needs in natural language and have an AI assistant handle the code generation, this guide is for you.
    *   **Data analysts, researchers, and product managers:** Anyone who needs web data but doesn't want to get bogged down in the intricacies of web scraping libraries.
    *   **Developers looking for rapid prototyping:** Even experienced developers can use "vibe coding" to quickly generate boilerplate or test ideas with `crawl4ai` before refining the code.
    *   **AI Coding Assistant Users:** This guide helps you understand what information to feed your AI to get the best `crawl4ai` code.

*   **1.3. How this Guide Helps You (and Your AI Assistant):**
    By understanding the concepts in this guide, you (and by extension, your AI assistant) will:
    *   Grasp the high-level capabilities of `crawl4ai` that are most relevant for prompting an AI.
    *   Learn the key terminology and building blocks of `crawl4ai` to include in your prompts for precise code generation.
    *   Discover common "vibe recipes" â€“ typical data extraction tasks and how to prompt an AI to solve them using `crawl4ai`.
    *   Pick up effective prompting patterns to maximize the quality of AI-generated `crawl4ai` code.

## 2. High-Level Capabilities of Crawl4AI (What to Tell Your AI Assistant Crawl4AI Can Do)

When you're "vibe coding" with your AI assistant, you don't need to explain every nuance of `crawl4ai`. Instead, focus on what it *can do* for you. Here's a high-level overview of capabilities you can confidently tell your AI assistant about:

*   **2.1. Fetching Any Webpage:**
    *   **How to tell your AI:** "Crawl4AI can fetch the content of any webpage, whether it's a simple static page or a complex JavaScript-heavy application."
    *   **Why it's important:** This establishes the fundamental capability â€“ getting the raw HTML from a target URL.

*   **2.2. Converting Web Content into Clean Markdown:**
    *   **How to tell your AI:** "Crawl4AI is great at turning messy web pages into clean, readable Markdown. This is perfect if I need to summarize an article, feed content into another LLM for Q&A, or just get the main text."
    *   **Why it's important:** Markdown is often the desired end-format for LLM-based tasks, and `crawl4ai` simplifies this conversion.

*   **2.3. Extracting Specific Pieces of Information (Structured Data/JSON):**
    *   **How to tell your AI:** "If I need specific data from a page, like all the product names and prices from an e-commerce site, or all the headlines from a news page, Crawl4AI can extract that and give it to me as structured JSON."
    *   **Why it's important:** This highlights `crawl4ai`'s ability to go beyond simple text extraction and pull out specific, targeted information.

*   **2.4. Crawling Multiple Pages at Once:**
    *   **How to tell your AI:** "If I have a list of URLs, Crawl4AI can process them all efficiently, often in parallel, instead of me having to do them one by one."
    *   **Why it's important:** This points to the library's capability for batch processing, saving significant time for larger tasks.

*   **2.5. Taking Screenshots or Making PDFs of Pages:**
    *   **How to tell your AI:** "Crawl4AI can take a screenshot of a webpage or save it as a PDF document. This is useful for archiving, visual checks, or sharing."
    *   **Why it's important:** Visual capture and PDF generation are common requirements that `crawl4ai` handles easily.

*   **2.6. Handling Simple Page Interactions:**
    *   **How to tell your AI:** "For pages that need a bit of interaction, like clicking a 'load more' button to reveal more content or scrolling down, Crawl4AI can execute JavaScript to do that."
    *   **Why it'S important:** This shows that `crawl4ai` isn't limited to static content and can handle basic dynamic page elements to access more data.

## 3. Key Building Blocks: Communicating Crawl4AI Concepts to Your AI Assistant

To get the best results from your AI assistant, you'll want to mention the key components of `crawl4ai` it should use. Think of these as the essential vocabulary for your "vibe" prompts.

*   **3.1. `AsyncWebCrawler`**: The Main Tool
    *   **3.1.1. Explaining its Role:**
        *   **What to tell your AI:** "The main tool we'll use in `crawl4ai` is called `AsyncWebCrawler`. We always need to create an instance of this first. It's best to use it with an `async with` statement to make sure resources are managed correctly."
        *   **Why it's important for the AI:** This directs the AI to the central class for all crawling operations and hints at the asynchronous nature and proper resource management.
    *   **3.1.2. Basic Usage Pattern:**
        *   **Vibe Prompt:** "Let's start a `crawl4ai` script. We'll need an `AsyncWebCrawler`."
        *   **Expected AI action:** The AI should generate the basic setup:
            ```python
            import asyncio
            from crawl4ai import AsyncWebCrawler, BrowserConfig, CrawlerRunConfig # Common imports

            async def main():
                # browser_config = BrowserConfig(...) # Optional, if needed
                async with AsyncWebCrawler(# config=browser_config
                                          ) as crawler:
                    # ... your crawl logic will go here ...
                    pass

            if __name__ == "__main__":
                asyncio.run(main())
            ```

*   **3.2. `arun(url, config)`**: Crawling a Single Page
    *   **3.2.1. Explaining its Purpose:**
        *   **What to tell your AI:** "To crawl a single page, we'll use the `arun` method of the `AsyncWebCrawler` instance. It takes the URL and an optional configuration object."
        *   **Why it's important for the AI:** This points to the primary method for individual page crawls.
    *   **3.2.2. Guiding the AI on `url` Specification:**
        *   **What to tell your AI:** "The `url` for `arun` can be a regular web address (like 'https://example.com'), a local file path (if it starts with `file://`), or even raw HTML content (if it starts with `raw:`)."
        *   **Vibe Prompts & AI Interpretation:**
            *   "Crawl this URL: https://my-target-site.com" -> AI uses `url="https://my-target-site.com"`
            *   "Process this local HTML file: /path/to/my/page.html" -> AI uses `url="file:///path/to/my/page.html"`
            *   "I have some raw HTML I want to parse: `<html><body><h1>Test</h1></body></html>`" -> AI uses `url="raw:<html><body><h1>Test</h1></body></html>"`
    *   **3.2.3. Emphasizing `config` (`CrawlerRunConfig`):**
        *   **What to tell your AI:** "The most important part for customizing a crawl is the `config` parameter of `arun`. This is where we'll use `CrawlerRunConfig` to tell `crawl4ai` *how* to crawl, what to extract, and what to do with the page."
        *   **Vibe Prompt:** "...and when you call `arun`, pass in a `CrawlerRunConfig` object with these settings: [describe settings]."

*   **3.3. `arun_many(urls, config)`**: Crawling Multiple Pages
    *   **3.3.1. Explaining its Purpose:**
        *   **What to tell your AI:** "If I have a list of URLs, we should use `arun_many` instead of calling `arun` in a loop. It's more efficient for batch crawling."
        *   **Why it's important for the AI:** Guides the AI to use the optimized method for multiple URLs, which handles concurrency internally.
    *   **3.3.2. Vibe Prompt:**
        *   "Crawl all these pages: https://example.com/page1, https://example.com/page2, https://example.com/page3."
        *   **Expected AI action:**
            ```python
            # ... inside async def main() and async with AsyncWebCrawler ...
            list_of_urls = [
                "https://example.com/page1",
                "https://example.com/page2",
                "https://example.com/page3"
            ]
            # run_config = CrawlerRunConfig(...) # Define config for all URLs
            results = await crawler.arun_many(urls=list_of_urls, config=run_config)
            for result in results:
                # process each result
                pass
            ```

*   **3.4. `CrawlerRunConfig`**: Tailoring the Crawl (Simplified for Vibe Coding)
    *   **3.4.1. Getting Markdown:**
        *   **Basic Request Vibe:** "Get the Markdown for this page [URL]."
            *   **AI's understanding:** The AI should know that `crawl4ai` produces Markdown by default, so a very simple `CrawlerRunConfig()` or even `None` might suffice if no other settings are needed.
        *   **For Cleaner Markdown Vibe:** "I need very clean Markdown from [URL], suitable for feeding into another LLM. Remove boilerplate like headers, footers, and navigation."
            *   **AI's understanding:** "Ah, the user wants cleaner output. I should suggest or use `DefaultMarkdownGenerator` combined with `PruningContentFilter`. The `PruningContentFilter` heuristically tries to remove common non-content sections."
            *   **Code Snippet AI might generate:**
                ```python
                from crawl4ai import CrawlerRunConfig, DefaultMarkdownGenerator
                from crawl4ai.content_filter_strategy import PruningContentFilter

                markdown_generator = DefaultMarkdownGenerator(
                    content_filter=PruningContentFilter()
                )
                run_config = CrawlerRunConfig(
                    markdown_generator=markdown_generator
                )
                # result = await crawler.arun(url="...", config=run_config)
                # print(result.markdown.fit_markdown) # Access the filtered markdown
                ```
            *   **Reasoning to convey (implicitly or explicitly):** `PruningContentFilter` is a good first step for general boilerplate removal. `DefaultMarkdownGenerator` then converts the filtered HTML to Markdown. Accessing `result.markdown.fit_markdown` gives this cleaner version.
    *   **3.4.2. Extracting Specific Data (JSON):**
        *   **3.4.2.1. Using `JsonCssExtractionStrategy` for Structured Pages:**
            *   **When to suggest it (Vibe):** "The page I want to crawl ([URL]) has a list of items, like products on an e-commerce category page, and each item has a similar layout. I want to extract these repeating items into a JSON list."
            *   **Information to provide the AI (Vibe):** "For each item, I want to get the 'product_name', which is usually in an `<h2>` tag, and the 'price', which seems to be in a `<span>` tag with a class like 'price-tag' or 'current-price'."
            *   **AI's Role & Reasoning:** The AI should recognize this pattern and suggest `JsonCssExtractionStrategy`. It understands that the user is describing a schema. The AI's job is to translate "name from h2" into `{"name": "product_name", "selector": "h2", "type": "text"}` within the `fields` list of a schema dictionary, and the overall repeating item selector into `baseSelector`. The AI should also know to set `extraction_type="schema"` on `LLMExtractionStrategy` if it were using that for schema generation, but here it's direct CSS.
            *   **Code Snippet AI might generate:**
                ```python
                from crawl4ai import CrawlerRunConfig
                from crawl4ai.extraction_strategy import JsonCssExtractionStrategy

                # AI would help construct this schema based on user's description
                schema = {
                    "name": "ProductList",
                    "baseSelector": "div.product-item", # Example selector for each product block
                    "fields": [
                        {"name": "product_name", "selector": "h2.product-title", "type": "text"},
                        {"name": "price", "selector": "span.price-tag", "type": "text"}
                    ]
                }
                extraction_strategy = JsonCssExtractionStrategy(schema=schema)
                run_config = CrawlerRunConfig(extraction_strategy=extraction_strategy)
                # result = await crawler.arun(url="...", config=run_config)
                # if result.success and result.extracted_content:
                #     products = json.loads(result.extracted_content)
                #     for product in products:
                #         print(f"Name: {product.get('product_name')}, Price: {product.get('price')}")
                ```
        *   **3.4.2.2. Using `LLMExtractionStrategy` for Complex/Unclear Structures:**
            *   **When to suggest it (Vibe):** "The page ([URL]) has the information I want, but it's not in a clear, repeating list, or it's mixed in with a lot of text. I need the AI to understand the content to pull out specific details." Or, "I want to extract information that requires some interpretation, like summarizing a paragraph."
            *   **Information to provide the AI (Vibe):**
                *   "Use `LLMExtractionStrategy` for this."
                *   "The LLM I want to use is [LLM provider/model, e.g., 'openai/gpt-4o-mini'] and my API key is [YOUR_API_KEY_OR_ENV_VAR_NAME] (or tell it to look for an env var)."
                *   **Option A (Describing fields):** "I need a JSON object with the following fields: 'author_name', 'article_publish_date', and a 'short_summary' (about 2 sentences)."
                *   **Option B (Example JSON):** "The JSON output should look something like this: `{\"author\": \"Jane Doe\", \"published_on\": \"2024-05-23\", \"summary\": \"This article discusses...\"}`."
                *   **Option C (Pydantic Model - more advanced but best for AI):** "Here's a Pydantic model that defines the structure I want: [Pydantic Class Code Snippet]. Use this for the schema."
            *   **AI's Role & Reasoning:** The AI needs to construct an `LLMConfig` and an `LLMExtractionStrategy`. If the user provides field descriptions or an example JSON, the AI can generate a simple schema dictionary. If a Pydantic model is provided, the AI should use `MyPydanticModel.model_json_schema()` to create the schema for `LLMExtractionStrategy`. This strategy is powerful because it leverages the LLM's understanding.
            *   **Code Snippet AI might generate (with Pydantic example):**
                ```python
                from crawl4ai import CrawlerRunConfig, LLMConfig
                from crawl4ai.extraction_strategy import LLMExtractionStrategy
                from pydantic import BaseModel, Field # Assuming user might provide this

                # User might provide this, or AI generates it from description
                class ArticleInfo(BaseModel):
                    author_name: str = Field(description="The main author of the article")
                    publication_date: str = Field(description="The date the article was published, e.g., YYYY-MM-DD")
                    short_summary: str = Field(description="A concise 2-3 sentence summary of the article")

                llm_config = LLMConfig(
                    provider="openai/gpt-4o-mini", # Or user's choice
                    api_token="env:OPENAI_API_KEY" # Or direct key if user insists and understands risk
                )
                extraction_strategy = LLMExtractionStrategy(
                    llm_config=llm_config,
                    schema=ArticleInfo.model_json_schema(),
                    # instruction="Extract author, publication date, and a summary." # Could also be used
                    extraction_type="schema" # Important for Pydantic/JSON schema
                )
                run_config = CrawlerRunConfig(extraction_strategy=extraction_strategy)
                # result = await crawler.arun(url="...", config=run_config)
                # if result.success and result.extracted_content:
                #     article_data = json.loads(result.extracted_content) # Or ArticleInfo.model_validate_json(result.extracted_content)
                #     print(article_data)
                ```
    *   **3.4.3. Interacting with Pages (Dynamic Content):**
        *   **How to tell your AI (Vibe):** "This page ([URL]) loads more content when you scroll down, or when you click a 'Show More' button. `crawl4ai` needs to perform this interaction."
        *   **For clicking (Vibe):** "To get all the data, we need to click the button with text 'Load All Comments'."
            *   **AI's understanding:** This requires `js_code` to find and click the button. The AI should be guided that finding elements by text might involve more complex JS like `Array.from(document.querySelectorAll('button')).find(btn => btn.textContent.includes('Load All Comments')).click();`.
        *   **For scrolling (Vibe):** "Scroll to the bottom of the page to make sure everything loads."
            *   **AI's understanding:** `js_code` like `window.scrollTo(0, document.body.scrollHeight);`
        *   **Ensuring actions complete (Vibe):** "After clicking 'Load More', wait for the new items to appear. They usually show up in a `div` with class `comment-list` and we expect more than 10 comments."
            *   **AI's understanding:** Use `wait_for`. This can be a CSS selector (e.g., `wait_for="css:.comment-list .comment-item:nth-child(11)"`) or a JS condition (e.g., `wait_for="js:() => document.querySelectorAll('.comment-item').length > 10"`).
        *   **For multi-step interactions on the same page (Vibe):** "I need to first click 'Agree to Cookies', then click 'Show Details', then extract the text. Make sure these happen on the same page view."
            *   **AI's understanding:** "This requires a `session_id` to persist the page state across multiple `arun` calls. Each `arun` call will perform one step of the interaction."
        *   **If only JS interaction is needed (Vibe):** "After the first page load, the next actions (like clicking 'Next Page') only update part of the page with JavaScript, they don't reload everything."
            *   **AI's understanding:** "For these subsequent `arun` calls within the same session, set `js_only=True` in `CrawlerRunConfig` to prevent unnecessary full page navigations, making it faster."
    *   **3.4.4. Taking Screenshots or PDFs:**
        *   **Simple request (Vibe):** "Get me a screenshot of [URL]."
            *   **AI's understanding:** Set `screenshot=True` in `CrawlerRunConfig`.
        *   **Or (Vibe):** "I need a PDF version of [URL]."
            *   **AI's understanding:** Set `pdf=True` in `CrawlerRunConfig`.
    *   **3.4.5. Caching for Speed and Freshness:**
        *   **For faster testing/development (Vibe):** "I'm testing my script for [URL]. Can you make it use the cache so it's faster after the first run?"
            *   **AI's understanding:** Use `cache_mode=CacheMode.ENABLED`.
            *   **Reasoning:** This saves time during development by re-using previously fetched content.
        *   **To get the latest data (Vibe):** "I need the absolute latest version of [URL], don't use any cached data."
            *   **AI's understanding:** Use `cache_mode=CacheMode.BYPASS`.
            *   **Reasoning:** Ensures `crawl4ai` fetches a fresh copy from the web, ignoring any local cache. This is crucial for production or when data changes frequently.

*   **3.5. `BrowserConfig`**: Browser-Level Settings (Simplified for Vibe Coding)
    *   **3.5.1. When it's usually not needed:**
        *   **What to tell your AI (or what it should infer):** "For most simple crawls where you just need to fetch a page and get its content, you probably don't need to worry about `BrowserConfig`. `crawl4ai` uses sensible defaults."
    *   **3.5.2. Headless Mode:**
        *   **Default behavior (Vibe):** "By default, `crawl4ai` runs 'headless', which means you won't see a browser window pop up. This is usually what we want for automated scripts."
        *   **To see the browser (Vibe):** "I'm having trouble with [URL], and I want to see what the browser is doing. Can you make it run in non-headless mode?"
            *   **AI's understanding:** The user wants to visually debug. Set `headless=False` in `BrowserConfig`.
    *   **3.5.3. Using a Proxy:**
        *   **How to ask (Vibe):** "I need to make `crawl4ai` use a proxy server for this crawl."
        *   **Information to provide (Vibe):** "The proxy server address is `http://myproxy.server.com:8080`. If it needs a username and password, they are `myuser` and `mypass`. Make sure to use the `proxy_config` setting in `BrowserConfig`."
            *   **AI's understanding:** The AI should construct a `ProxyConfig` object (or dictionary that `BrowserConfig` can handle) and pass it to `BrowserConfig`.
    *   **3.5.4. Changing User Agent:**
        *   **How to ask (Vibe):** "The website [URL] might be blocking default user agents. Can we make `crawl4ai` look like it's Firefox on a Mac?"
        *   **Information to provide (Vibe):** "You can set a custom `user_agent` string in `BrowserConfig`. For example, 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:109.0) Gecko/20100101 Firefox/115.0'."
            *   **AI's understanding:** The AI should pass the provided string to the `user_agent` parameter of `BrowserConfig`.

*   **3.6. `LLMConfig`**: Configuring Language Models (Simplified for Vibe Coding)
    *   **3.6.1. When it's needed:**
        *   **What to tell your AI:** "If we're using `LLMExtractionStrategy` to extract structured data or `LLMContentFilter` to clean up content, we need to tell `crawl4ai` which language model to use. This is done with an `LLMConfig` object."
    *   **3.6.2. Information to provide the AI (Vibe):**
        *   **Model choice:** "For this task, let's use the `provider` called 'openai/gpt-4o-mini'." (Other examples: 'ollama/llama3', 'anthropic/claude-3-opus-20240229').
        *   **API Key:** "My `api_token` for this provider is [YOUR_API_KEY_PLACEHOLDER]. (Best practice is to tell the AI to get it from an environment variable, e.g., 'env:OPENAI_API_KEY')."
            *   **AI's understanding:** The AI will create an `LLMConfig(provider="...", api_token="...")` and pass it to the relevant strategy.
            *   **Code Snippet AI might generate:**
                ```python
                from crawl4ai import LLMConfig
                # For OpenAI
                llm_conf = LLMConfig(provider="openai/gpt-4o-mini", api_token="env:OPENAI_API_KEY")
                # For Ollama (locally running Llama3)
                # llm_conf = LLMConfig(provider="ollama/llama3") # api_token often not needed for local Ollama
                ```

*   **3.7. The `CrawlResult`**: Understanding What You Get Back
    *   **3.7.1. Checking for Success:**
        *   **What to tell your AI (Crucial Vibe):** "When `crawl4ai` finishes an `arun` or `arun_many` call, the most important first step is to check if it was successful. Tell the AI to always generate code that checks `result.success`. This will be `True` or `False`."
        *   **If `False` (Vibe):** "If `result.success` is `False`, the AI should print or log `result.error_message` to tell us what went wrong."
    *   **3.7.2. Accessing Markdown Content:**
        *   **Raw Markdown (Vibe):** "The main text content of the page, converted to Markdown, is usually in `result.markdown.raw_markdown`."
        *   **Filtered Markdown (Vibe):** "If we used a content filter (like `PruningContentFilter`), the cleaner, more focused Markdown will be in `result.markdown.fit_markdown`."
    *   **3.7.3. Accessing Extracted Structured Data (JSON):**
        *   **Where to find it (Vibe):** "If we asked `crawl4ai` to extract specific structured data (using `JsonCssExtractionStrategy` or `LLMExtractionStrategy`), that data will be in `result.extracted_content`."
        *   **How to use it (Vibe):** "The `result.extracted_content` is a JSON string. To use it in Python, tell the AI to parse it using `json.loads()`."
    *   **3.7.4. Accessing Links:**
        *   **What it contains (Vibe):** "`result.links` is a dictionary. It usually has two keys: 'internal' and 'external'. Each of these is a list of links found on the page, with details like the link text and URL."
    *   **3.7.5. Accessing Media Information:**
        *   **What it contains (Vibe):** "`result.media` is another dictionary. It can contain lists of 'images', 'videos', or 'audios' found on the page, along with their URLs and sometimes alt text or descriptions."
    *   **3.7.6. Accessing Screenshots and PDFs:**
        *   **Screenshots (Vibe):** "If we set `screenshot=True` in the config, the screenshot image data (which is a base64 encoded string) will be in `result.screenshot`."
        *   **PDFs (Vibe):** "If we set `pdf=True`, the PDF data (as raw bytes) will be in `result.pdf`."

## 4. Common "Vibe" Recipes & Prompting Patterns for AI Assistants

Here are some common scenarios and how you can prompt your AI assistant to use `crawl4ai` to solve them.

*   **4.1. Recipe 1: Get Clean Markdown from a Page**
    *   **4.1.1. User Vibe Prompt to AI:**
        *   *Example 1 (Basic):* "Hey AI, can you use `crawl4ai` to get me the clean Markdown content of `https://blog.example.com/my-latest-post`?"
        *   *Example 2 (Requesting more cleaning):* "AI, I need *very* clean Markdown from `https://some.news.site/article123`. It's for an LLM, so please try to remove ads, navigation, and footers."
    *   **4.1.2. Key `crawl4ai` elements the AI should use (and why):**
        *   **`AsyncWebCrawler`:** The starting point for any crawl.
        *   **`arun`:** Since it's a single URL.
        *   **`CrawlerRunConfig`:**
            *   For Example 1, the default `DefaultMarkdownGenerator` might be sufficient.
            *   For Example 2 ("very clean"), the AI should be guided to use `DefaultMarkdownGenerator(content_filter=PruningContentFilter())`.
            *   **Reasoning:** `PruningContentFilter` is designed to heuristically remove common boilerplate like headers, footers, and navigation bars before Markdown conversion, leading to cleaner text suitable for LLMs.
    *   **4.1.3. Expected Output from AI-generated code:**
        *   A Python script that initializes `AsyncWebCrawler`, calls `arun` with the appropriate URL and config.
        *   The script should then access and print (or save) `result.markdown.raw_markdown` (for basic) or `result.markdown.fit_markdown` (if `PruningContentFilter` was used).
        *   **Code Example (for "very clean"):**
            ```python
            import asyncio
            from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, DefaultMarkdownGenerator
            from crawl4ai.content_filter_strategy import PruningContentFilter

            async def get_clean_markdown(url_to_crawl):
                markdown_generator = DefaultMarkdownGenerator(
                    content_filter=PruningContentFilter()
                )
                run_config = CrawlerRunConfig(
                    markdown_generator=markdown_generator,
                    cache_mode="BYPASS" # Ensure fresh crawl for demo
                )
                async with AsyncWebCrawler() as crawler:
                    result = await crawler.arun(url=url_to_crawl, config=run_config)
                    if result.success:
                        print(f"--- Fit Markdown for {url_to_crawl} ---")
                        print(result.markdown.fit_markdown)
                        # You might also want to see raw_markdown to compare
                        # print(f"--- Raw Markdown for {url_to_crawl} ---")
                        # print(result.markdown.raw_markdown)
                    else:
                        print(f"Failed to crawl {url_to_crawl}: {result.error_message}")

            # asyncio.run(get_clean_markdown("https://en.wikipedia.org/wiki/Python_(programming_language)"))
            ```

*   **4.2. Recipe 2: Extract All Product Names and Prices from an E-commerce Category Page**
    *   **4.2.1. User Vibe Prompt to AI:**
        *   *Example:* "AI, I need to use `crawl4ai` to get all product names and their prices from `https://www.example-store.com/laptops`. On that page, product names look like they are in `<h3>` tags with a class `product-title`, and prices are in `<span>` elements with the class `final-price`."
    *   **4.2.2. Key `crawl4ai` elements AI should use (and why):**
        *   **`AsyncWebCrawler`**, **`arun`**.
        *   **`CrawlerRunConfig`** with **`JsonCssExtractionStrategy`**.
            *   **Reasoning:** The user described a page with repeating structured items. `JsonCssExtractionStrategy` is ideal for this as it uses CSS selectors to pinpoint the data. The AI's task is to translate the user's description of element locations into a valid schema for the strategy.
            *   The AI needs to understand that `baseSelector` in the schema should target the container for each product, and `fields` will target individual pieces of data within that container.
    *   **4.2.3. Expected Output from AI-generated code:**
        *   A Python script that defines the schema dictionary.
        *   Initializes `JsonCssExtractionStrategy` with this schema.
        *   Passes the strategy to `CrawlerRunConfig`.
        *   After `arun`, it parses `result.extracted_content` using `json.loads()` and likely iterates through the list of extracted product dictionaries.
        *   **Code Example:**
            ```python
            import asyncio
            import json
            from crawl4ai import AsyncWebCrawler, CrawlerRunConfig
            from crawl4ai.extraction_strategy import JsonCssExtractionStrategy

            async def extract_products(url_to_crawl):
                # AI helps create this schema based on user's description
                product_schema = {
                    "name": "LaptopList",
                    "baseSelector": "div.product-listing-item", # Hypothetical selector for each product's container
                    "fields": [
                        {"name": "product_name", "selector": "h3.product-title", "type": "text"},
                        {"name": "price", "selector": "span.final-price", "type": "text"}
                    ]
                }
                extraction_strategy = JsonCssExtractionStrategy(schema=product_schema)
                run_config = CrawlerRunConfig(
                    extraction_strategy=extraction_strategy,
                    cache_mode="BYPASS"
                )
                async with AsyncWebCrawler() as crawler:
                    result = await crawler.arun(url=url_to_crawl, config=run_config)
                    if result.success and result.extracted_content:
                        products = json.loads(result.extracted_content)
                        print(f"Found {len(products)} products:")
                        for i, product in enumerate(products[:3]): # Print first 3
                            print(f"  Product {i+1}: Name='{product.get('product_name')}', Price='{product.get('price')}'")
                    else:
                        print(f"Failed to extract products from {url_to_crawl}: {result.error_message}")

            # asyncio.run(extract_products("https://www.example-store.com/laptops")) # Replace with a real URL for testing
            ```

*   **4.3. Recipe 3: Extract Key Information from an Article using an LLM**
    *   **4.3.1. User Vibe Prompt to AI:**
        *   *Example:* "AI, I want `crawl4ai` to read this article: `https://example.com/news/ai-breakthrough`. Use `openai/gpt-4o-mini` to extract the author's name, the publication date, and a short (2-3 sentence) summary. The output should be JSON. My OpenAI API key is in the `OPENAI_API_KEY` environment variable."
    *   **4.3.2. Key `crawl4ai` elements AI should use (and why):**
        *   **`AsyncWebCrawler`**, **`arun`**.
        *   **`CrawlerRunConfig`** with **`LLMExtractionStrategy`**.
        *   **`LLMConfig`**: To specify the `provider` ("openai/gpt-4o-mini") and `api_token` ("env:OPENAI_API_KEY").
            *   **Reasoning:** The task requires understanding and summarization, making `LLMExtractionStrategy` suitable. The AI needs to construct a schema (either a simple dictionary or a Pydantic model `model_json_schema()`) that tells the LLM what fields to populate. The instruction to the LLM will be implicitly derived from the schema field descriptions or can be explicitly provided.
    *   **4.3.3. Expected Output from AI-generated code:**
        *   Python script that defines a Pydantic model (or a dictionary schema).
        *   Initializes `LLMConfig` and `LLMExtractionStrategy`.
        *   Parses `result.extracted_content`.
        *   **Code Example (using Pydantic):**
            ```python
            import asyncio
            import json
            import os
            from pydantic import BaseModel, Field
            from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, LLMConfig
            from crawl4ai.extraction_strategy import LLMExtractionStrategy

            class ArticleDetails(BaseModel):
                author_name: str = Field(..., description="The main author of the article.")
                publication_date: str = Field(..., description="The date the article was published (e.g., YYYY-MM-DD).")
                summary: str = Field(..., description="A concise 2-3 sentence summary of the article.")

            async def extract_article_info_llm(url_to_crawl):
                if not os.getenv("OPENAI_API_KEY"): # Or your specific key variable
                    print("API key environment variable not set. Skipping LLM extraction.")
                    return

                llm_config = LLMConfig(
                    provider="openai/gpt-4o-mini", # Use a cost-effective model for demos
                    api_token="env:OPENAI_API_KEY"
                )
                extraction_strategy = LLMExtractionStrategy(
                    llm_config=llm_config,
                    schema=ArticleDetails.model_json_schema(),
                    extraction_type="schema" # Crucial for Pydantic/JSON schema
                )
                run_config = CrawlerRunConfig(
                    extraction_strategy=extraction_strategy,
                    cache_mode="BYPASS"
                )
                async with AsyncWebCrawler() as crawler:
                    result = await crawler.arun(url=url_to_crawl, config=run_config)
                    if result.success and result.extracted_content:
                        try:
                            article_data = ArticleDetails.model_validate_json(result.extracted_content)
                            print(f"Extracted Article Info for {url_to_crawl}:")
                            print(json.dumps(article_data.model_dump(), indent=2))
                        except Exception as e:
                            print(f"Error parsing LLM output: {e}")
                            print(f"Raw LLM output: {result.extracted_content}")
                    else:
                        print(f"Failed to extract article info from {url_to_crawl}: {result.error_message}")

            # asyncio.run(extract_article_info_llm("https://www.example.com/news/ai-breakthrough")) # Replace with real article
            ```

*   **4.4. Recipe 4: Crawl the first 3 pages of a blog (clicking "Next Page")**
    *   **4.4.1. User Vibe Prompt to AI:**
        *   *Example:* "AI, can you use `crawl4ai` to get the Markdown from the first 3 pages of `https://myblog.example.com/archive`? To get to the next page, I think you need to click a link that says 'Older Posts'."
    *   **4.4.2. Key `crawl4ai` elements AI should use (and why):**
        *   **`AsyncWebCrawler`**.
        *   **Multiple `arun` calls** in a loop (3 iterations).
        *   **`CrawlerRunConfig`** with:
            *   `session_id="blog_session"`: **Crucial** for maintaining the browser state (cookies, current page) across the multiple clicks.
            *   `js_code`: JavaScript to find and click the "Older Posts" link. The AI might need to generate robust JS like:
                `Array.from(document.querySelectorAll('a')).find(a => a.textContent.trim() === 'Older Posts')?.click();`
            *   `wait_for`: After clicking, wait for a condition that indicates the next page has loaded (e.g., a specific element on the new page, or a change in an existing element). This can be tricky and might require some iteration. A simple `wait_for` for a few seconds could also be a starting point, like `wait_for=3000` (milliseconds).
            *   `js_only=True`: For the second and third `arun` calls, after the initial page load. This tells `crawl4ai` to only execute the JS and not perform a full new navigation to the original URL.
    *   **4.4.3. Expected Output from AI-generated code:**
        *   A Python script with a loop that calls `arun` three times.
        *   The script should collect and potentially print or save the Markdown from each page.
        *   **Code Example:**
            ```python
            import asyncio
            from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, CacheMode

            async def crawl_blog_pages(start_url, num_pages=3):
                session_id = "my_blog_crawl_session"
                all_markdowns = []

                # JavaScript to find and click "Older Posts" (example)
                js_click_older_posts = """
                (() => {
                    const links = Array.from(document.querySelectorAll('a'));
                    const olderPostsLink = links.find(a => a.textContent.trim().toLowerCase() === 'older posts');
                    if (olderPostsLink) {
                        olderPostsLink.click();
                        return true; // Indicate click was attempted
                    }
                    return false; // Indicate link not found
                })();
                """

                async with AsyncWebCrawler() as crawler:
                    current_url = start_url
                    for i in range(num_pages):
                        print(f"Crawling page {i+1}...")
                        run_config_dict = {
                            "session_id": session_id,
                            "cache_mode": CacheMode.BYPASS,
                            "wait_for": 2000 # Wait 2s for content to potentially load after click
                        }
                        if i > 0: # For subsequent pages, click and don't re-navigate
                            run_config_dict["js_code"] = js_click_older_posts
                            run_config_dict["js_only"] = True
                        
                        run_config = CrawlerRunConfig(**run_config_dict)
                        
                        result = await crawler.arun(url=current_url, config=run_config) # URL is mainly for context in js_only
                        
                        if result.success:
                            print(f"  Page {i+1} ({result.url}) - Markdown length: {len(result.markdown.raw_markdown)}")
                            all_markdowns.append({"url": result.url, "markdown": result.markdown.raw_markdown})
                            if i < num_pages - 1 and i > 0 and not run_config_dict.get("js_code_executed_successfully", True): # Hypothetical flag
                                print(f"  'Older Posts' link might not have been found or clicked on page {i+1}. Stopping.")
                                break
                        else:
                            print(f"  Failed to crawl page {i+1}: {result.error_message}")
                            break
                    
                    # Important: Clean up the session
                    await crawler.crawler_strategy.kill_session(session_id) 
                
                print(f"\nCollected markdown for {len(all_markdowns)} pages.")
                # For demo, print first 100 chars of each
                # for i, md_data in enumerate(all_markdowns):
                #     print(f"\n--- Page {i+1} URL: {md_data['url']} ---")
                #     print(md_data['markdown'][:100] + "...")

            # asyncio.run(crawl_blog_pages("YOUR_BLOG_START_URL_HERE"))
            ```

*   **4.5. Recipe 5: Get Screenshots of a List of URLs**
    *   **4.5.1. User Vibe Prompt to AI:**
        *   *Example:* "AI, use `crawl4ai` to take a screenshot of each of these pages: `https://example.com`, `https://crawl4ai.com`, `https://github.com`. Save them as `example_com.png`, `crawl4ai_com.png`, and `github_com.png`."
    *   **4.5.2. Key `crawl4ai` elements AI should use (and why):**
        *   **`AsyncWebCrawler`**.
        *   **`arun_many`**: Efficient for processing a list of URLs.
        *   **`CrawlerRunConfig`** with `screenshot=True`.
            *   **Reasoning:** `arun_many` will process each URL with the same config. The AI needs to add logic to iterate through the results and save each `result.screenshot` (which is base64 data) to a uniquely named file.
    *   **4.5.3. Expected Output from AI-generated code:**
        *   Python script.
        *   PNG files saved to the current directory or a specified output directory.
        *   **Code Example:**
            ```python
            import asyncio
            import base64
            import os
            from urllib.parse import urlparse
            from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, CacheMode

            async def take_screenshots(urls_to_screenshot):
                run_config = CrawlerRunConfig(
                    screenshot=True,
                    cache_mode=CacheMode.BYPASS # Get fresh screenshots
                )
                output_dir = "screenshots_output"
                os.makedirs(output_dir, exist_ok=True)

                async with AsyncWebCrawler() as crawler:
                    results = await crawler.arun_many(urls=urls_to_screenshot, config=run_config)
                    
                    for result in results:
                        if result.success and result.screenshot:
                            # Create a filename from the URL
                            parsed_url = urlparse(result.url)
                            filename = "".join(c if c.isalnum() else '_' for c in parsed_url.netloc + parsed_url.path)
                            if not filename or filename == "_": # Handle root path or empty paths
                                filename = "homepage"
                            filepath = os.path.join(output_dir, f"{filename}.png")
                            
                            try:
                                screenshot_data = base64.b64decode(result.screenshot)
                                with open(filepath, "wb") as f:
                                    f.write(screenshot_data)
                                print(f"Screenshot saved to {filepath}")
                            except Exception as e:
                                print(f"Error saving screenshot for {result.url}: {e}")
                        elif not result.success:
                            print(f"Failed to crawl {result.url}: {result.error_message}")
                        elif not result.screenshot:
                            print(f"Crawled {result.url} but no screenshot data was returned.")
            
            # urls = ["https://example.com", "https://crawl4ai.com", "https://github.com"]
            # asyncio.run(take_screenshots(urls))
            ```

## 5. Tips for Effective Prompting Your AI Assistant for Crawl4AI Tasks

To get the best code from your AI assistant when working with `crawl4ai`, consider these prompting tips:

*   **5.1. Be Clear About Your Goal:**
    *   Start with a high-level objective. Instead of just "Crawl a page," say "I need to extract all article titles from the homepage of this news site," or "Get the main content of this blog post as clean Markdown," or "Take full-page screenshots of these product pages." This helps the AI choose the right strategies and configurations.

*   **5.2. Always Provide the URL(s):**
    *   This seems obvious, but be precise. If it's a list, provide the list.
    *   Remember to use the `file:///` prefix for local files (e.g., `file:///Users/me/Documents/mypage.html`) and `raw:` for inline HTML (e.g., `raw:<html><body>...</body></html>`). The AI might not always infer this correctly without a hint.

*   **5.3. Describe Data for Extraction (Especially for `JsonCssExtractionStrategy` or `LLMExtractionStrategy`):**
    *   **What you want:** List the specific pieces of information you need (e.g., "product name," "price," "author," "publication_date," "article summary").
    *   **Where to find it (for CSS/XPath):** If you have an idea of the HTML structure, share it. "Product names seem to be in `<h2>` tags with class `item-title`." "The price is always in a `<span>` element right after a `<strong>` tag that says 'Price:'." This helps the AI generate accurate CSS selectors or XPath expressions for `JsonCssExtractionStrategy`.
    *   **Desired structure (for LLM):** For `LLMExtractionStrategy`, tell the AI the desired JSON structure. "I want a list of objects, where each object has a 'title' and a 'link'." Or even better, "Can you define a Pydantic model for me that has 'title' as a string and 'link' as a string, and then use that for extraction?"

*   **5.4. Specify LLM Details for LLM Extraction or Filtering:**
    *   **Model/Provider:** "Use `openai/gpt-4o-mini` for this extraction." or "I want to use my local Ollama model, `ollama/llama3`."
    *   **API Key:** Clearly state where the API key should come from. "My API key is in the environment variable `OPENAI_API_KEY`." (This is safer than putting the key directly in the prompt). If you must provide it directly, be aware of the security implications.

*   **5.5. Mention Page Dynamics and Interactions:**
    *   "This page loads more items when you scroll down."
    *   "You need to click the 'View All Reviews' button to see all the reviews."
    *   "The data I want only appears after selecting 'Category X' from a dropdown."
    *   This signals to the AI that `js_code`, `wait_for`, and possibly `session_id` will be necessary. You might need to guide it on *how* to identify the elements to interact with (e.g., "The 'Load More' button has the ID `load-more-btn`").

*   **5.6. Iterative Refinement is Key:**
    *   Your first prompt might not yield perfect code. That's okay!
    *   Treat it as a conversation. If the AI-generated code misses something or makes a mistake:
        *   "That was close, but it missed extracting the product ratings. Ratings seem to be in a `div` with class `star-rating` inside each product item."
        *   "The script timed out. Can we increase the `page_timeout` in `CrawlerRunConfig` to 90 seconds?"
        *   "It didn't click the 'Next' button correctly. The button actually has the text '>>' instead of 'Next Page'."
    *   Provide the error messages or incorrect output back to the AI for context.

## 6. What to Expect as Output (From AI-Generated Code)

When you use "Vibe Coding" with an AI assistant for `crawl4ai`, you should generally expect the following:

*   **6.1. Python Code:**
    *   The primary output will be a Python script that uses the `crawl4ai` library.
    *   It should include necessary imports like `asyncio`, `AsyncWebCrawler`, `CrawlerRunConfig`, etc.
    *   It will typically define an `async def main():` function and run it with `asyncio.run(main())`.

*   **6.2. Accessing the `CrawlResult`:**
    *   The core of the script will involve one or more calls to `crawler.arun(...)` or `crawler.arun_many(...)`.
    *   These calls return `CrawlResult` objects (or a list of them for `arun_many`).
    *   The AI-generated code should then show you how to access the specific data you asked for from these `CrawlResult` objects. For example:
        *   `print(result.markdown.raw_markdown)` or `print(result.markdown.fit_markdown)`
        *   `data = json.loads(result.extracted_content)`
        *   `screenshot_data = base64.b64decode(result.screenshot)`
        *   `if not result.success: print(result.error_message)`

*   **6.3. Files Saved to Disk (if requested):**
    *   If your vibe prompt included saving data (e.g., "save the screenshots as PNG files," "write the extracted JSON to `output.json`"), the AI-generated code should include the Python logic to perform these file operations.
    *   **Example for saving a screenshot:**
        ```python
        import base64
        # ... inside your async function, after getting 'result' ...
        if result.success and result.screenshot:
            with open("myscreenshot.png", "wb") as f:
                f.write(base64.b64decode(result.screenshot))
            print("Screenshot saved to myscreenshot.png")
        ```

## 7. Conclusion: Vibe Your Way to Web Data!

*   **7.1. Recap of "Vibe Coding" Benefits with `crawl4ai`:**
    "Vibe Coding" empowers you to leverage the full capabilities of `crawl4ai` without needing to memorize every API detail. By understanding the high-level concepts and key building blocks outlined in this guide, you can effectively communicate your data extraction and web interaction needs to an AI coding assistant. This leads to faster prototyping, easier access to web data for non-programmers, and a more intuitive way to build data-driven applications.

*   **7.2. Encouragement to experiment with different prompts and `crawl4ai` features:**
    The key to successful "Vibe Coding" is experimentation. Try different ways of describing your goals to your AI assistant. If the first attempt doesn't yield the perfect `crawl4ai` code, refine your prompt with more specific details or hints. Don't be afraid to mention `crawl4ai` specific terms like `CrawlerRunConfig`, `js_code`, or `LLMExtractionStrategy` â€“ this guide has equipped you with the essential vocabulary. The more context you provide, the better the AI can assist you.

*   **7.3. Pointers to more detailed `crawl4ai` documentation for users who want to learn direct coding or advanced configurations:**
    While "Vibe Coding" is a great way to get started and be productive quickly, you might eventually want to dive deeper into `crawl4ai`'s capabilities or fine-tune the generated code yourself. For that, refer to:
    *   **The Official Crawl4AI API Reference:** (Assuming this exists or will exist - replace with actual link if available, e.g., `https://docs.crawl4ai.com/api/`) For detailed information on all classes, methods, and parameters.
    *   **Specific "Reasoning & Problem-Solving" Guides:** Check the `crawl4ai` documentation for other guides that delve into specific components like advanced `CrawlerRunConfig` options, deep crawling strategies, or custom extraction techniques.

Happy Vibe Coding, and may your web data adventures be fruitful!
```