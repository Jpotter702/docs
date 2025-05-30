Okay, I'm ready to generate the "Reasoning & Problem-Solving Framework" document for the `extraction` component, adhering strictly to the outline you provided and the principles for a "reasoning" document.

I will use the information from the previously shared code snippets and documentation to create illustrative examples, explain design rationale, and provide actionable advice.

Here is the `llm_reasoning_extraction.md` document:

```markdown
# Detailed Outline for crawl4ai - extraction Component

**Target Document Type:** reasoning
**Target Output Filename Suggestion:** `llm_reasoning_extraction.md`
**Library Version Context:** 0.6.0+ (based on provided code)
**Outline Generation Date:** 2024-05-24
---

# Mastering Data Extraction with Crawl4AI

## 1. Introduction: Why Structured Data Extraction Matters in Web Crawling
    * 1.1. The Value of Going Beyond Raw HTML: Turning Web Content into Actionable Data
        Web pages, in their raw HTML form, are designed for human consumption. While Crawl4AI excels at converting HTML to clean Markdown for LLMs, often the goal is to extract specific, structured pieces of information. This could be product prices, article headlines, author names, contact details, or any other data points that can be organized into a predictable format. Structured data is more readily usable for databases, APIs, analytics, training machine learning models, or feeding into other automated processes. Simply having the full HTML or Markdown isn't enough when you need to operate on discrete data fields.

    * 1.2. Common Challenges in Web Data Extraction (Dynamic content, varied structures, anti-scraping)
        Extracting data from the web isn't always straightforward. Common hurdles include:
        *   **Varied HTML Structures:** Websites change layouts, and even within a single site, different page types can have vastly different structures. A CSS selector that works today might break tomorrow.
        *   **Dynamic Content:** Much of the web's content is loaded via JavaScript after the initial HTML page. Extractors need to handle this, either by executing JS (as Crawl4AI's browser-based crawlers do) or by finding data in embedded JSON within `<script>` tags.
        *   **Anti-Scraping Measures:** Websites may employ techniques to deter or block automated scraping, requiring more sophisticated approaches.
        *   **Unstructured Data:** Sometimes, the data isn't neatly tagged. It might be buried in free-form text, requiring natural language understanding to identify and extract.
        *   **Scalability and Maintenance:** Writing and maintaining custom parsers for many sites can be a significant engineering effort.

    * 1.3. Crawl4AI's Approach: A Flexible, Strategy-Based Extraction Framework
        Crawl4AI tackles these challenges by offering a flexible and powerful extraction framework built around the concept of "strategies." This allows you to choose the best tool for the job, whether it's precise rule-based extraction or intelligent LLM-powered parsing.
        *   **`ExtractionStrategy` Interface:** This is the core. It defines a common contract for how extraction should happen. Crawl4AI provides several built-in strategies (CSS-based, XPath-based, Regex-based, LLM-based), and you can even implement your own for highly specialized needs. This promotes modularity â€“ you can swap out extraction logic without changing your core crawling code.
        *   **`ChunkingStrategy` Interface:** Specifically for LLM-based extraction, this interface helps prepare content by breaking it into manageable pieces that fit within an LLM's context window. This is crucial for both performance and accuracy when dealing with large documents.
        *   **Balancing Rule-Based and LLM-Powered Extraction:** Crawl4AI doesn't force you into one paradigm. You can use fast and efficient CSS selectors for well-structured sites and then leverage the power of LLMs for complex, unstructured data, or even combine them in hybrid approaches. This flexibility is key to building robust and adaptable web data extraction pipelines.

## 2. Core Concepts in Crawl4AI Extraction
    * 2.1. The `ExtractionStrategy` Interface: Your Key to Custom Extraction
        *   2.1.1. Purpose: Why an interface? Promoting modularity and extensibility.
            The `ExtractionStrategy` interface (defined in `crawl4ai/extraction_strategy.py`) is a fundamental design choice in Crawl4AI. It establishes a common contract for all extraction methods. The primary benefit is **modularity**: your main crawling logic doesn't need to know the specifics of *how* data is extracted. It simply invokes the strategy, and the strategy handles the details. This makes your code cleaner and more maintainable.
            Furthermore, it promotes **extensibility**: if the built-in strategies don't fit your exact needs (e.g., you're dealing with a proprietary data format or a very unique web structure), you can create your own class that implements the `ExtractionStrategy` interface and plug it directly into Crawl4AI.

        *   2.1.2. Key Methods to Understand (Conceptual): `extract()` and `run()`.
            While you typically won't call these directly if using built-in strategies (Crawl4AI handles it), understanding their roles is important if you plan to create custom strategies:
            *   `extract(url: str, html_content: str, *args, **kwargs) -> List[Dict[str, Any]]`: This is the core method that every concrete strategy must implement. It takes the URL and HTML content (or pre-processed content like Markdown, depending on the `input_format` of the strategy) and returns a list of dictionaries, where each dictionary represents an extracted item.
            *   `run(url: str, sections: List[str], *args, **kwargs) -> List[Dict[str, Any]]`: This method is often used for strategies that process content in chunks (like `LLMExtractionStrategy`). It takes a list of content `sections` and typically calls `extract()` for each section, then aggregates the results. For simpler strategies that operate on the whole content at once, `run` might just call `extract` with the joined sections.

        *   2.1.3. When Would You Implement Your Own `ExtractionStrategy`?
            You'd consider creating a custom `ExtractionStrategy` in scenarios like:
            *   **Highly Specialized Data Sources:** If you're extracting data from a non-standard format (e.g., custom XML, binary files, or a very idiosyncratic HTML structure not well-suited for CSS/XPath/Regex).
            *   **Integrating Proprietary Extraction Logic:** If your organization has existing, specialized parsing libraries or algorithms you want to use within the Crawl4AI framework.
            *   **Advanced Performance Optimizations:** For extremely high-volume scraping of a specific site, you might develop a hyper-optimized parser that bypasses more general tools.
            *   **Unique Pre-processing or Post-processing:** If your extraction requires complex data transformations or enrichments beyond what the built-in strategies offer.

    * 2.2. The `ChunkingStrategy` Interface: Preparing Content for LLMs
        *   2.2.1. Why Chunking is Crucial for LLM-Based Extraction
            Large Language Models (LLMs) have a "context window" â€“ a limit on the amount of text they can process at once (e.g., 4096, 8192, or even 128k+ tokens). If you feed an entire long webpage directly to an LLM for extraction:
            *   **Context Overflow:** The content might exceed the LLM's limit, leading to truncation and loss of information, or outright errors.
            *   **Reduced Accuracy:** Even if it fits, an LLM might struggle to find specific details in a very long, noisy document. Its attention can get diluted.
            *   **Higher Cost & Latency:** Processing more tokens means higher API costs (for paid models) and longer response times.
            Chunking addresses this by breaking down the input content into smaller, more focused segments, each of which can be processed by the LLM more effectively.

        *   2.2.2. How Chunking Strategies Work in Crawl4AI
            A `ChunkingStrategy` (defined in `crawl4ai/chunking_strategy.py`) is responsible for taking a single block of text (e.g., the Markdown content of a page) and dividing it into a list of smaller strings (chunks).
            *   The primary method is `chunk(document: str) -> List[str]`.
            *   The `LLMExtractionStrategy` then iterates over these chunks, sending each one (or a batch of them, depending on its internal logic) to the LLM for extraction. The results from each chunk are then typically aggregated.

        *   2.2.3. Overview of Built-in Chunking Strategies
            Crawl4AI provides a couple of ready-to-use chunking strategies:
            *   **`RegexChunking` (default for `LLMExtractionStrategy`):** This strategy (from `crawl4ai/chunking_strategy.py`) uses regular expressions to split text. By default, it might split by paragraphs or other common delimiters. It aims to create semantically meaningful chunks. This is often a good general-purpose choice.
                *   *When to use:* Good for text-heavy documents where paragraph or section breaks are meaningful.
            *   **`IdentityChunking`:** This strategy (from `crawl4ai/chunking_strategy.py`) doesn't actually do any chunking; it returns the input document as a single chunk.
                *   *When to use:*
                    *   When your input documents are already small enough to fit the LLM's context window.
                    *   When you have pre-processed your content into chunks *before* passing it to `LLMExtractionStrategy`.
                    *   When the LLM you're using has a very large context window and performs well on full documents for your specific task.

        *   2.2.4. When to Choose or Implement a Custom `ChunkingStrategy`.
            While the built-in chunkers are useful, you might need a custom `ChunkingStrategy` if:
            *   **Domain-Specific Document Structures:** Your content has unique structural elements that `RegexChunking` doesn't handle well (e.g., legal documents with numbered clauses, scripts with dialogue/scene breaks, log files).
            *   **Semantic Chunking Needs:** You require more sophisticated chunking based on semantic meaning rather than just regex patterns (though this can become complex and might involve NLP techniques within your custom chunker).
            *   **Fixed-Size Overlapping Chunks:** You want to implement a sliding window approach with precise control over chunk size and overlap, which might be beneficial for certain types_of information retrieval.
            *   **Table or List-Aware Chunking:** You need to ensure that tables or lists are not awkwardly split across chunks.

    * 2.3. Schema Definition: The Blueprint for Your Extracted Data
        *   2.3.1. Why a Well-Defined Schema is Essential
            A schema acts as a contract for your data. It defines:
            *   What pieces of information you expect to extract (the field names).
            *   The data type of each piece of information (e.g., string, integer, boolean, list, nested object).
            *   How to find each piece of information (e.g., CSS selector, XPath, or implied for LLM).
            Benefits include:
            *   **Consistency:** Ensures that extracted data always has the same structure, making it easier to process downstream.
            *   **Reliability:** Helps catch errors if a website's structure changes and a selector no longer works, or if an LLM fails to extract a required field.
            *   **Guidance:** For rule-based extractors, it provides the direct rules. For LLM-based extractors, it informs the LLM about the desired output structure, significantly improving the quality and predictability of results.
            *   **Validation:** Pydantic models, used with LLMs, offer automatic data validation.

        *   2.3.2. Defining Schemas for CSS/XPath/LXML Strategies (Dictionary-based)
            For strategies like `JsonCssExtractionStrategy`, `JsonXPathExtractionStrategy`, and `JsonLxmlExtractionStrategy`, the schema is a Python dictionary.
            *   **Structure:**
                ```python
                schema = {
                    "name": "MyExtractorName", # Optional: A name for your schema
                    "baseSelector": "div.product-item", # CSS selector for repeating items (e.g., products on a list page)
                    "fields": [
                        {
                            "name": "product_name",      # Name of the field in the output
                            "selector": "h2.product-title", # CSS/XPath selector relative to baseSelector (or page if no baseSelector)
                            "type": "text"             # "text", "attribute", "html", "nested", "list"
                        },
                        {
                            "name": "product_link",
                            "selector": "a.product-link",
                            "type": "attribute",
                            "attribute": "href"        # Name of the HTML attribute to extract (e.g., 'href' for links)
                        },
                        # ... more fields ...
                    ]
                }
                ```
            *   **Key Fields:**
                *   `baseSelector`: (Optional) If you're extracting a list of similar items (e.g., multiple products, articles), this selector targets the container element for each item. All field selectors will then be relative to this base element. If omitted, field selectors are relative to the whole document.
                *   `fields`: A list of dictionaries, each defining a field to extract.
                    *   `name`: The key for this field in the output JSON.
                    *   `selector`: The CSS selector or XPath expression to locate the data.
                    *   `type`:
                        *   `"text"`: Extracts the text content of the selected element.
                        *   `"attribute"`: Extracts the value of a specified HTML attribute (requires an additional `"attribute": "attr_name"` key).
                        *   `"html"`: Extracts the inner HTML of the selected element.
                        *   `"nested"`: Allows defining a sub-schema for extracting nested structured data (requires an additional `"fields": [...]` key, similar to the top-level fields).
                        *   `"list"`: Indicates that the selector is expected to return multiple elements, and the extraction logic (defined by sub-fields) should be applied to each. Often used with a nested `fields` definition.
            *   **Tips for Designing Dictionary-Based Schemas:**
                *   Be as specific as possible with your selectors to avoid ambiguity.
                *   Start with a simple schema and iteratively add more fields.
                *   Test your selectors in your browser's developer tools first.
                *   Use `baseSelector` for lists to keep field selectors concise and maintainable.
            *   **Example: Schema for extracting blog post titles and authors:**
                ```python
                blog_post_schema = {
                    "name": "BlogPostExtractor",
                    "baseSelector": "article.post",
                    "fields": [
                        {"name": "title", "selector": "h1.entry-title", "type": "text"},
                        {"name": "author", "selector": "span.author-name", "type": "text"},
                        {"name": "publication_date", "selector": "time.published-date", "type": "attribute", "attribute": "datetime"}
                    ]
                }
                ```

        *   2.3.3. Defining Schemas for `LLMExtractionStrategy` (Pydantic Models)
            When using `LLMExtractionStrategy` with `extraction_type="schema"` (the default), you provide a Pydantic model as the schema.
            *   **Advantages of Pydantic:**
                *   **Type Hints:** Clearly define the expected data type for each field.
                *   **Validation:** Pydantic automatically validates that the data extracted by the LLM conforms to your model's types and constraints. If not, it raises an error.
                *   **IDE Support:** Excellent autocompletion and type checking in modern IDEs.
                *   **Serialization:** Easy conversion to and from JSON.
            *   **How Pydantic Models Guide the LLM:** Crawl4AI internally converts your Pydantic model into a JSON schema representation, which is then included in the prompt to the LLM. This tells the LLM the exact structure and field names it should use in its JSON output.
            *   **Example: Pydantic model for product information:**
                ```python
                from pydantic import BaseModel, HttpUrl
                from typing import Optional, List

                class ProductInfo(BaseModel):
                    product_name: str
                    price: Optional[float]
                    description: str
                    image_urls: List[HttpUrl] = []
                    features: Optional[List[str]]
                ```
                When this model is used, the LLM will be instructed to return JSON objects that look like:
                ```json
                {
                  "product_name": "Awesome Laptop",
                  "price": 1299.99,
                  "description": "A very fast and light laptop.",
                  "image_urls": ["https://example.com/image1.jpg"],
                  "features": ["16GB RAM", "512GB SSD"]
                }
                ```

        *   2.3.4. Best Practices for Schema Design Across Strategy Types.
            *   **Be Specific with Field Names:** Use clear, descriptive names that reflect the data.
            *   **Start Simple:** Begin with a few key fields and expand as needed.
            *   **Handle Optional Data:** For fields that might not always be present, define them as optional in your Pydantic model (e.g., `Optional[str]`) or ensure your non-LLM logic handles missing elements gracefully (e.g., by providing default values or allowing `None`).
            *   **Consider Data Types:** Choose appropriate types (string, number, boolean, list, nested object) to ensure data integrity.
            *   **Test Iteratively:** Regularly test your schemas with real web content to catch issues early.

## 3. Non-LLM Based Extraction Strategies: Precision and Speed
    * 3.1. When to Choose Non-LLM Strategies
        Non-LLM (or rule-based) strategies are excellent choices when:
        *   **Website Structure is Consistent:** The target website has a stable and predictable HTML structure. Changes are infrequent.
        *   **Performance is Key:** These strategies are generally much faster and less resource-intensive than LLM-based approaches as they don't involve API calls to external services or loading large models.
        *   **Cost is a Major Factor:** Non-LLM strategies have no per-extraction operational cost beyond your own compute resources.
        *   **Data Points are Simple and Directly Targetable:** You need to extract clearly identifiable pieces of text, attributes, or simple lists.
        *   **You Have Expertise in CSS Selectors or XPath:** If you or your team are comfortable writing and maintaining these selectors.
        *   **No Semantic Interpretation Needed:** The data can be located purely by its position or tags in the HTML, without needing to understand the meaning of the surrounding text.

    * 3.2. Mastering `JsonCssExtractionStrategy`
        *   3.2.1. Understanding Its Strengths: Leveraging CSS Selectors
            `JsonCssExtractionStrategy` is often the first choice for non-LLM extraction due to the widespread familiarity with CSS selectors.
            *   **Strengths:**
                *   Relatively easy to learn and write.
                *   Well-supported by browsers' developer tools for testing.
                *   Efficient for most common extraction tasks.
            *   **Underlying Library:** Crawl4AI typically uses BeautifulSoup4 or LXML for parsing HTML and applying CSS selectors, providing robust and performant parsing.

        *   3.2.2. Workflow: Extracting Data with CSS
            *   **Step 1: Analyzing the Target HTML Structure:**
                *   Use your browser's developer tools (e.g., "Inspect Element") to examine the HTML of the page you want to scrape.
                *   Identify the HTML tags, classes, and IDs that uniquely contain the data you need.
                *   Example: If you want to extract an article's title, you might find it's always within an `<h1>` tag with class `article-title`.
            *   **Step 2: Crafting your Dictionary-Based Schema with CSS Selectors:**
                *   Define your schema as a Python dictionary, as described in section 2.3.2.
                *   Fill in the `selector` for each field with the appropriate CSS selector.
                ```python
                article_schema = {
                    "baseSelector": "article.post", # Target each article
                    "fields": [
                        {"name": "title", "selector": "h1.entry-title", "type": "text"},
                        {"name": "author_link", "selector": "a.author-url", "type": "attribute", "attribute": "href"}
                    ]
                }
                ```
            *   **Step 3: Configuring `CrawlerRunConfig` to use `JsonCssExtractionStrategy`:**
                ```python
                from crawl4ai import CrawlerRunConfig
                from crawl4ai.extraction_strategy import JsonCssExtractionStrategy

                extraction_strategy = JsonCssExtractionStrategy(schema=article_schema)
                run_config = CrawlerRunConfig(extraction_strategy=extraction_strategy)
                ```
            *   **Step 4: Interpreting the Results:**
                *   The `result.extracted_content` will be a JSON string containing a list of dictionaries, where each dictionary matches your schema.
                ```python
                import json
                # Assuming 'result' is the output from crawler.arun()
                if result.extracted_content:
                    data = json.loads(result.extracted_content)
                    for item in data:
                        print(f"Title: {item.get('title')}, Author Link: {item.get('author_link')}")
                ```

        *   3.2.3. Handling Nested Data Structures
            You can extract nested data by defining a field with `type: "nested"` and providing another `fields` list within it.
            *   **How to define:** The `selector` for the nested field targets the container of the nested data. The sub-fields' selectors are then relative to this nested container.
            *   **Example: Extracting comments and their authors:**
                ```python
                comment_schema = {
                    "baseSelector": "div.comment-thread",
                    "fields": [
                        {"name": "comment_id", "selector": "div.comment", "type": "attribute", "attribute": "data-comment-id"},
                        {
                            "name": "main_comment",
                            "selector": "div.comment-body", # Selector for the main comment container
                            "type": "nested",
                            "fields": [
                                {"name": "author", "selector": "span.comment-author", "type": "text"},
                                {"name": "text", "selector": "p.comment-text", "type": "text"}
                            ]
                        },
                        {
                            "name": "replies",
                            "selector": "div.reply", # Selector for each reply
                            "type": "list", # Indicates multiple replies
                            "fields": [ # Schema for each reply item
                                {"name": "reply_author", "selector": "span.reply-author", "type": "text"},
                                {"name": "reply_text", "selector": "p.reply-text", "type": "text"}
                            ]
                        }
                    ]
                }
                ```

        *   3.2.4. Extracting Lists of Items
            The `baseSelector` is key for extracting lists.
            *   **`baseSelector`:** Targets each individual item in the list (e.g., each `<li>` in a `<ul>`, each `div.product-card`).
            *   **Relative Field Selectors:** All selectors within the `fields` list are then evaluated *relative* to each element matched by `baseSelector`.
            *   **Example: Extracting a list of products from a category page:**
                ```python
                product_list_schema = {
                    "name": "ProductList",
                    "baseSelector": "div.product-listing div.product-item-container", # Each product card
                    "fields": [
                        {"name": "product_name", "selector": "h3.product-name a", "type": "text"},
                        {"name": "price", "selector": "span.price", "type": "text"},
                        {"name": "url", "selector": "h3.product-name a", "type": "attribute", "attribute": "href"}
                    ]
                }
                ```
                This would produce a list of product dictionaries.

        *   3.2.5. Code Example: Extracting News Headlines and Links from Hacker News (Illustrative)
            ```python
            import asyncio
            import json
            from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig
            from crawl4ai.extraction_strategy import JsonCssExtractionStrategy
            from crawl4ai.cache_manager import CacheMode

            async def extract_hn_news():
                hn_schema = {
                    "name": "HackerNewsFrontPage",
                    "baseSelector": "tr.athing", # Each story row in Hacker News
                    "fields": [
                        {
                            "name": "rank",
                            "selector": "span.rank",
                            "type": "text"
                        },
                        {
                            "name": "title",
                            "selector": "span.titleline > a", # Get the first 'a' tag within titleline
                            "type": "text"
                        },
                        {
                            "name": "url",
                            "selector": "span.titleline > a",
                            "type": "attribute",
                            "attribute": "href"
                        },
                        # Example for next row (subtext) data - shows using a more complex relative selector
                        {
                            "name": "points",
                            "selector": "xpath=./following-sibling::tr[1]/td[@class='subtext']/span[@class='score']",
                            "type": "text" # Note: Using XPath within CSS strategy for advanced relative selection
                                           # This is a conceptual example; pure CSS might be trickier for direct sibling access.
                                           # A more common CSS approach would be to have a slightly broader baseSelector
                                           # or separate extraction steps if nesting is too complex for pure CSS.
                        }
                    ]
                }

                extraction_strategy = JsonCssExtractionStrategy(schema=hn_schema)
                browser_config = BrowserConfig(headless=True)
                run_config = CrawlerRunConfig(
                    extraction_strategy=extraction_strategy,
                    cache_mode=CacheMode.BYPASS # For fresh data in this example
                )

                async with AsyncWebCrawler(config=browser_config) as crawler:
                    result = await crawler.arun(
                        url="https://news.ycombinator.com/",
                        config=run_config
                    )

                if result.success and result.extracted_content:
                    articles = json.loads(result.extracted_content)
                    print(f"Extracted {len(articles)} articles from Hacker News:")
                    for i, article in enumerate(articles[:5]): # Print first 5
                        print(f"  {i+1}. {article.get('title')} ({article.get('points', 'N/A points')}) - {article.get('url')}")
                else:
                    print(f"Failed to extract articles: {result.error_message}")

            if __name__ == "__main__":
                asyncio.run(extract_hn_news())
            ```
            *Self-correction during thought process: The original `points` selector was a bit too complex for a pure CSS example within `JsonCssExtractionStrategy`. While some libraries might allow mixing, it's better to illustrate clear CSS or mention that for such relative sibling traversals, XPath might be more direct, or the schema/baseSelector might need restructuring.*

        *   3.2.6. Best Practices for Writing Robust CSS Selectors.
            *   **Prefer IDs if Stable:** `#unique-id` is usually the most robust if available and unique.
            *   **Use Specific but Not Overly Specific Classes:** `.meaningful-class` is good. Avoid overly long chains like `div.container > div.row > div.col-md-8 > article.post > h1` if `h1.post-title` is unique enough.
            *   **Attribute Selectors:** `input[name="email"]` can be very precise.
            *   **Avoid Relying on Order (unless necessary):** `:nth-child()` can be brittle if the page structure changes slightly. Use it sparingly.
            *   **Test Thoroughly:** Use browser dev tools to validate your selectors on various pages of the target site.

        *   3.2.7. Troubleshooting: Common Issues and Solutions
            *   **Selector Returning `None` or Empty List:**
                *   *Cause:* Selector is incorrect, element doesn't exist, or content is loaded dynamically *after* initial HTML.
                *   *Solution:* Double-check selector in dev tools. For dynamic content, ensure Crawl4AI's browser is rendering JS (default) or use `wait_for` in `CrawlerRunConfig`.
            *   **Handling Dynamic Class Names:**
                *   *Cause:* Sites using CSS-in-JS or frameworks might generate dynamic class names (e.g., `_header_a83hf8`).
                *   *Solution:* Look for stable parent elements or use attribute selectors that target parts of class names (e.g., `div[class*="header_"]`), or rely on structural selectors (e.g., `article > h1`). This is where XPath or LLM strategies might be more robust.
            *   **Extracting Incorrect Data:**
                *   *Cause:* Selector is too broad and matches multiple elements.
                *   *Solution:* Make your selector more specific. Use direct child `>` or adjacent sibling `+` combinators if appropriate.

    * 3.3. Leveraging `JsonXPathExtractionStrategy`
        *   3.3.1. When XPath Shines: Complex Selections and Navigating the DOM
            XPath (XML Path Language) is a powerful query language for selecting nodes from an XML or HTML document. It excels where CSS selectors might fall short:
            *   **Complex Relationships:** Selecting elements based on their ancestors, siblings, or preceding/following elements (e.g., "find the `div` that follows an `h2` with text 'Price'").
            *   **Text Content Matching:** Selecting elements based on their text content (e.g., `//button[contains(text(), 'Add to Cart')]`).
            *   **Navigating Up the DOM:** Easily selecting parent or ancestor elements.
            *   **Using Functions:** XPath has built-in functions for string manipulation, counting, etc.

        *   3.3.2. Key Differences from CSS Strategy (Syntax, capabilities).
            *   **Syntax:** XPath uses a path-like syntax (e.g., `/html/body/div[1]/h1`) whereas CSS uses selectors like `div.my-class > h1`.
            *   **Capabilities:** XPath is generally more powerful for traversing the DOM in complex ways. CSS is often simpler for common class/ID/tag selections.
            *   **Performance:** For simple selections, CSS can sometimes be faster. For complex traversals, a well-written XPath might be more efficient than a convoluted CSS equivalent. Crawl4AI uses LXML for XPath, which is highly performant.

        *   3.3.3. Workflow: Similar to CSS, but with XPath expressions.
            The workflow is identical to `JsonCssExtractionStrategy`, except your schema's `selector` fields will contain XPath expressions.
            *   **Step 1: Analyzing HTML:** Use browser developer tools. Many browsers allow you to right-click an element and "Copy XPath."
            *   **Step 2: Crafting your Dictionary-Based Schema with XPath:**
                ```python
                xpath_schema = {
                    "baseSelector": "//article[@class='blog-entry']", # XPath for each article
                    "fields": [
                        {"name": "title", "selector": ".//h1[contains(@class, 'title')]/text()", "type": "text"},
                        {"name": "author_url", "selector": ".//a[contains(@class, 'author-profile')]/@href", "type": "attribute"}
                        # Note: type "attribute" for XPath will get the attribute value if selector ends with /@attr
                        # type "text" will get text content. If selector selects an element, text() can be appended.
                    ]
                }
                ```
                *Important for XPath `type` handling:*
                *   If your XPath selector directly targets an attribute (e.g., `//a/@href`), `type: "attribute"` is redundant but harmless; the attribute value is returned.
                *   If your XPath selector targets an element and you want its text, use `type: "text"` (or append `/text()` to your XPath).
                *   If your XPath targets an element and you want an attribute of *that* element, you'd use `type: "attribute"` and specify the `attribute` key, e.g., `{"selector": "//img", "type": "attribute", "attribute": "src"}`.

            *   **Step 3: Configuration in `CrawlerRunConfig`:**
                ```python
                from crawl4ai.extraction_strategy import JsonXPathExtractionStrategy
                extraction_strategy = JsonXPathExtractionStrategy(schema=xpath_schema)
                run_config = CrawlerRunConfig(extraction_strategy=extraction_strategy)
                ```

        *   3.3.4. Code Example: Extracting Data Using XPath Functions (e.g., `contains()`, `text()`)
            ```python
            import asyncio
            import json
            from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig
            from crawl4ai.extraction_strategy import JsonXPathExtractionStrategy
            from crawl4ai.cache_manager import CacheMode

            async def extract_with_xpath():
                # Example HTML content
                sample_html = """
                <html><body>
                    <div class="product">
                        <h2>Product A</h2>
                        <span class="price">Price: $19.99</span>
                        <a href="/product/a" class="details-link">View Details</a>
                    </div>
                    <div class="product">
                        <h2>Product B</h2>
                        <span class="price">Price: $29.99</span>
                        <a href="/product/b" class="details-link">More Info</a>
                    </div>
                </body></html>
                """

                product_schema_xpath = {
                    "name": "ProductXPathExtractor",
                    "baseSelector": "//div[@class='product']",
                    "fields": [
                        {"name": "name", "selector": ".//h2/text()", "type": "text"},
                        # Extracts text after "Price: "
                        {"name": "price_value", "selector": "substring-after(.//span[contains(@class,'price')]/text(), 'Price: $')", "type": "text"},
                        {"name": "details_url", "selector": ".//a[contains(@class,'details-link') or contains(text(),'More Info')]/@href", "type": "attribute"}
                    ]
                }
                extraction_strategy = JsonXPathExtractionStrategy(schema=product_schema_xpath)
                run_config = CrawlerRunConfig(extraction_strategy=extraction_strategy, cache_mode=CacheMode.BYPASS)

                async with AsyncWebCrawler(config=BrowserConfig(headless=True)) as crawler:
                    # Using raw HTML input for this example
                    result = await crawler.arun(url=f"raw://{sample_html}", config=run_config)

                if result.success and result.extracted_content:
                    products = json.loads(result.extracted_content)
                    print("Extracted Products using XPath:")
                    for product in products:
                        print(product)
                else:
                    print(f"XPath extraction failed: {result.error_message}")

            if __name__ == "__main__":
                asyncio.run(extract_with_xpath())
            ```

        *   3.3.5. Tips for Effective XPath Usage.
            *   **Start with `.` for relative paths:** Within a `baseSelector`, field selectors should usually start with `./` to be relative to the current base element.
            *   **Use `text()` to get text content:** `//div/text()` gets the direct text children. `//div//text()` gets all text within the div.
            *   **Select attributes with `/@attribute_name`:** `//img/@src`.
            *   **Leverage functions:** `contains()`, `starts-with()`, `substring-after()`, `normalize-space()` are very useful.
            *   **Be mindful of namespaces** if working with XML-heavy HTML or actual XML.

    * 3.4. Understanding `JsonLxmlExtractionStrategy`
        The `JsonLxmlExtractionStrategy` is essentially a specialized version of `JsonCssExtractionStrategy` that explicitly uses the LXML library for parsing and CSS selection.
        *   3.4.1. Potential Performance Gains: When to consider it.
            LXML is known for its speed. For very large HTML documents or high-throughput scraping scenarios where parsing speed is a bottleneck, `JsonLxmlExtractionStrategy` *might* offer better performance than the default BeautifulSoup-backed CSS selector engine (though BeautifulSoup itself can use LXML as a parser). The actual difference can vary.
        *   3.4.2. Usage and Configuration: Similarities and differences with `JsonCssExtractionStrategy`.
            Usage is identical to `JsonCssExtractionStrategy`. You provide the same dictionary-based schema with CSS selectors. Crawl4AI handles the backend difference.
            ```python
            from crawl4ai.extraction_strategy import JsonLxmlExtractionStrategy # Import this
            
            # Schema is the same as for JsonCssExtractionStrategy
            my_schema = { ... } 
            extraction_strategy = JsonLxmlExtractionStrategy(schema=my_schema)
            run_config = CrawlerRunConfig(extraction_strategy=extraction_strategy)
            ```
        *   3.4.3. When to benchmark against `JsonCssExtractionStrategy`.
            If you suspect CSS selection is a performance bottleneck in your Crawl4AI application, and you're processing a large volume of pages or very large pages, it's worth benchmarking `JsonLxmlExtractionStrategy` against the default `JsonCssExtractionStrategy` to see if it provides a noticeable speedup in your specific environment and use case.

    * 3.5. Precise Targeting with `RegexExtractionStrategy`
        *   3.5.1. The Power of Regular Expressions: When Are They the Right Tool?
            Regular expressions are ideal when:
            *   **Data is in Unstructured or Semi-Structured Text:** The information isn't neatly tagged with specific HTML elements or classes (e.g., extracting an email address from a paragraph of text).
            *   **Targeting Specific Patterns:** You need to find data that conforms to a known pattern, like email addresses, phone numbers, dates, URLs, postal codes, UUIDs, product SKUs, etc.
            *   **HTML Structure is Unreliable:** If the HTML tags around the data change frequently, but the data itself has a consistent textual pattern.
            *   **Fallback or Augmentation:** Can be used to extract data that CSS/XPath selectors miss, or to clean/validate data extracted by other means.

        *   3.5.2. Utilizing Built-in Patterns
            `RegexExtractionStrategy` (from `crawl4ai.extraction_strategy`) comes with a handy `BuiltInPatterns` IntFlag enum. This allows you to easily enable common extraction patterns without writing the regex yourself.
            *   **Overview:** Refer to `RegexExtractionStrategy._B` (or `RegexExtractionStrategy.BuiltInPatterns` if aliased publicly) for the available flags like `EMAIL`, `PHONE_US`, `URL`, `IPV4`, `UUID`, `DATE_ISO`, `CURRENCY`, etc. Each flag corresponds to a pre-defined, tested regex pattern.
            *   **How to use:** You pass the bitwise OR of the desired patterns to the `pattern` argument of the `RegexExtractionStrategy` constructor.
            *   **Code Example: Extracting all email addresses and US phone numbers from a webpage's text:**
                ```python
                import asyncio
                import json
                from crawl4ai import AsyncWebCrawler, CrawlerRunConfig
                from crawl4ai.extraction_strategy import RegexExtractionStrategy

                async def extract_contact_info():
                    # Combine built-in patterns
                    patterns_to_use = RegexExtractionStrategy.BuiltInPatterns.EMAIL | \
                                      RegexExtractionStrategy.BuiltInPatterns.PHONE_US
                    
                    extraction_strategy = RegexExtractionStrategy(pattern=patterns_to_use)
                    
                    # This strategy works best on plain text, so use 'markdown' or 'text' input_format
                    # if using with the standard crawler flow, or pass plain text directly.
                    run_config = CrawlerRunConfig(
                        extraction_strategy=extraction_strategy,
                        # input_format='text' # Alternative: let the strategy handle HTML to text
                    )

                    sample_text_content = """
                    Contact us at support@example.com or call (800) 555-1212.
                    Our sales team can be reached at sales@example.com.
                    For urgent matters, dial 1-800-555-1234.
                    Our website is https://example.com.
                    """

                    async with AsyncWebCrawler() as crawler:
                        # Here, we're directly using the 'extract' method for simplicity with raw text
                        # In a full crawl, you'd use crawler.arun() with the run_config
                        extracted_data = extraction_strategy.extract(
                            url="raw://text_content", # Dummy URL for raw content
                            html_content=sample_text_content # Provide text directly
                        )
                    
                    print("Extracted Contact Info:")
                    for item in extracted_data:
                        print(f"  Label: {item['label']}, Value: {item['value']}, Span: {item['span']}")

                if __name__ == "__main__":
                    asyncio.run(extract_contact_info())
                ```
                **Output structure for `RegexExtractionStrategy`:**
                Each extracted item is a dictionary:
                `{"url": "source_url", "label": "pattern_label", "value": "matched_string", "span": [start_index, end_index]}`

        *   3.5.3. Defining and Using Custom Regex Patterns
            If built-in patterns aren't sufficient, you can provide your own.
            *   **Passing a Dictionary:** Supply a dictionary where keys are labels (strings) for your patterns, and values are the regex pattern strings.
            *   **Tips for Writing Regex:**
                *   Use non-capturing groups `(?:...)` if you don't need to capture a part of the match.
                *   Be mindful of greediness (e.g., use `*?` or `+?` for non-greedy matches).
                *   Test your regex thoroughly with tools like regex101.com.
                *   Remember that regex patterns are raw strings in Python (e.g., `r"\b\d{5}\b"`).
            *   **Code Example: Extracting custom product SKUs (e.g., SKU-XXXX-YYYY):**
                ```python
                import asyncio
                import json
                from crawl4ai import AsyncWebCrawler, CrawlerRunConfig
                from crawl4ai.extraction_strategy import RegexExtractionStrategy

                async def extract_skus():
                    custom_patterns = {
                        "product_sku": r"SKU-\d{4}-[A-Z]{4}"
                    }
                    extraction_strategy = RegexExtractionStrategy(custom=custom_patterns)
                    run_config = CrawlerRunConfig(extraction_strategy=extraction_strategy)

                    sample_text = "Product Alpha SKU-1234-ABCD and Product Beta SKU-5678-EFGH."
                    
                    # Direct usage for simplicity
                    extracted_data = extraction_strategy.extract(url="raw://text", html_content=sample_text)

                    print("Extracted SKUs:")
                    for item in extracted_data:
                        print(item)
                
                if __name__ == "__main__":
                    asyncio.run(extract_skus())
                ```

        *   3.5.4. Leveraging `generate_pattern()` for Dynamic Regex Creation
            The static method `RegexExtractionStrategy.generate_pattern(examples: List[str], labels: List[str] = None, llm_config: LLMConfig = None, **kwargs) -> str` (or Dict[str, str] if labels are provided) is a powerful utility that uses an LLM to generate a regex pattern for you based on examples.
            *   **How it Works:** You provide a list of example strings that you want to match. Optionally, you can provide corresponding labels if you want to generate multiple patterns for different types of data. The method then queries an LLM (configurable via `llm_config`) to infer a regex pattern that would capture those examples.
            *   **Use Cases:**
                *   You have a clear set of examples of the data you want to extract but are not a regex expert.
                *   You need to quickly prototype an extraction for a new data type.
                *   The pattern is complex, and you want an AI-assisted starting point.
            *   **Code Example: Generating a regex pattern from a list of example IDs:**
                ```python
                import asyncio
                from crawl4ai.extraction_strategy import RegexExtractionStrategy
                from crawl4ai import LLMConfig # Assuming LLMConfig is correctly imported

                async def generate_and_use_regex():
                    example_ids = ["ID_123_XYZ", "ID_456_ABC", "ID_789_DEF"]
                    
                    # Configure LLM for pattern generation (replace with your actual config)
                    # For open-source, set api_token=None or your specific setup
                    llm_for_regex = LLMConfig(provider="openai/gpt-3.5-turbo", api_token="YOUR_OPENAI_API_KEY") 
                                            # Or: provider="ollama/llama3", api_token=None

                    try:
                        # Generate a single pattern
                        generated_pattern_str = await RegexExtractionStrategy.generate_pattern(
                            examples=example_ids,
                            llm_config=llm_for_regex,
                            # Optional: Add a query to guide the LLM
                            query="Generate a regex to capture these types of IDs."
                        )
                        print(f"Generated Regex for IDs: {generated_pattern_str}")

                        # You can then use this generated_pattern_str in RegexExtractionStrategy:
                        # custom_patterns = {"custom_id": generated_pattern_str}
                        # strategy = RegexExtractionStrategy(custom=custom_patterns)
                        # ... then use the strategy ...

                        # Example for generating multiple labeled patterns
                        example_data = {
                            "order_id": ["ORD-001", "ORD-002"],
                            "user_id": ["USR_A", "USR_B"]
                        }
                        generated_patterns_dict = await RegexExtractionStrategy.generate_pattern(
                            examples=list(example_data.values()), # Pass lists of examples
                            labels=list(example_data.keys()),    # Corresponding labels
                            llm_config=llm_for_regex
                        )
                        print(f"Generated Labeled Regex Patterns: {generated_patterns_dict}")
                        # strategy_multi = RegexExtractionStrategy(custom=generated_patterns_dict)

                    except Exception as e:
                        print(f"Error generating pattern: {e}")
                        print("Make sure your LLMConfig is correctly set up and the LLM is accessible.")


                if __name__ == "__main__":
                    asyncio.run(generate_and_use_regex())
                ```
            *   **Limitations and Considerations:**
                *   **LLM Dependency:** Requires a configured and accessible LLM.
                *   **Quality Varies:** The quality of the generated regex depends on the LLM's capabilities and the quality/quantity of your examples.
                *   **Review and Test:** Always review and test LLM-generated regex patterns thoroughly before deploying them in production. They might be overly broad or miss edge cases.
                *   **Cost/Latency:** Involves an LLM call, so it's not for runtime pattern generation in a tight loop. Best used for one-off generation or infrequent updates.

        *   3.5.5. Best Practices for `RegexExtractionStrategy`.
            *   **Target Plain Text:** Regex works best on clean text. If applying to HTML, consider extracting text content first or using the `input_format="text"` or `input_format="markdown"` options in `LLMExtractionStrategy` if combining.
            *   **Be Specific:** Craft regex to be as specific as possible to avoid false positives.
            *   **Use Non-Capturing Groups:** `(?:...)` can improve performance if you don't need to capture certain parts of the match.
            *   **Test with Diverse Examples:** Ensure your regex works for various valid inputs and doesn't match invalid ones.

        *   3.5.6. Debugging Regex: Ensuring Accuracy and Avoiding Over-matching.
            *   **Online Regex Testers:** Use tools like regex101.com or pythex.org to build and test your patterns interactively with sample text.
            *   **Break Down Complex Patterns:** If a regex is very complex, test its components separately.
            *   **Log Matched Values:** During development, print out the `value` extracted by your regex to verify it's capturing what you intend.
            *   **Consider Edge Cases:** Think about variations in formatting, optional components, or unusual inputs that your regex might encounter.

## 4. LLM-Based Extraction Strategies: Handling Complexity and Ambiguity
    * 4.1. When to Turn to LLMs for Data Extraction
        LLM-based extraction strategies shine when:
        *   **Unstructured or Inconsistently Structured Content:** The data isn't in neat HTML tables or consistently tagged elements. It might be embedded in paragraphs, reviews, or forum posts.
        *   **Need for Semantic Understanding:** You need to extract information based on its meaning, not just its position or HTML tags (e.g., "What is the main sentiment of this review?" or "Extract the key arguments from this article.").
        *   **Rapid Prototyping:** When defining precise CSS/XPath selectors is too time-consuming or the site structure is unknown/volatile, an LLM can often get you started quickly with a descriptive prompt.
        *   **Extracting Nuanced Information:** For tasks like summarization, topic extraction, or identifying relationships between entities.
        *   **Schema Flexibility:** When the desired output structure is complex or might evolve, LLMs (especially with Pydantic schema guidance) can adapt more easily than hand-crafted rules.
        *   **Handling Diverse Sources:** If you need to extract similar information from many different websites with varying layouts, a well-crafted LLM prompt can be more generalizable than site-specific selectors.

    * 4.2. Deep Dive into `LLMExtractionStrategy`
        *   4.2.1. Core Idea: Instructing an LLM to be Your Extractor.
            The `LLMExtractionStrategy` (from `crawl4ai.extraction_strategy`) leverages the power of Large Language Models. Instead of writing explicit rules (like CSS selectors), you provide:
            1.  **Content:** The text (HTML, Markdown, or plain text) to extract from.
            2.  **Instruction:** A natural language prompt telling the LLM *what* to extract and *how* to structure it.
            3.  **(Optional but Recommended) Schema:** A Pydantic model defining the desired output structure, which helps the LLM produce consistent and validated JSON.
            The LLM then processes the content based on your instructions and attempts to return the data in the requested format.

        *   4.2.2. Configuring the LLM: The `LLMConfig` Object
            The `LLMConfig` object (from `crawl4ai.types` or `crawl4ai.async_configs`) is crucial for telling Crawl4AI which LLM to use and how to interact with it.
            ```python
            from crawl4ai import LLMConfig

            # Example for OpenAI
            openai_config = LLMConfig(
                provider="openai/gpt-4o-mini", # Or "openai/gpt-3.5-turbo", etc.
                api_token="sk-YOUR_OPENAI_API_KEY", # Best practice: use os.environ.get("OPENAI_API_KEY")
                # Optional parameters:
                # temperature=0.7, 
                # max_tokens=1024 
            )

            # Example for a local Ollama model
            ollama_config = LLMConfig(
                provider="ollama/llama3", # Assumes Ollama is running and llama3 model is pulled
                api_token=None, # Not needed for local Ollama by default
                base_url="http://localhost:11434" # Default Ollama API endpoint
            )

            # Example for Groq
            groq_config = LLMConfig(
                provider="groq/llama3-8b-8192",
                api_token=os.environ.get("GROQ_API_KEY")
            )
            ```
            *   **`provider` (str):** Specifies the LLM provider and model (e.g., `"openai/gpt-4o-mini"`, `"ollama/llama3"`, `"groq/llama3-8b-8192"`). Crawl4AI uses LiteLLM under the hood, supporting a wide range of models.
            *   **`api_token` (Optional[str]):** Your API key for the chosen provider. For local models like Ollama, this is often not needed.
                *   **Best Practice:** Store API keys in environment variables (e.g., `os.environ.get("OPENAI_API_KEY")`) rather than hardcoding them.
            *   **`base_url` (Optional[str]):** For self-hosted LLMs or providers with custom API endpoints (like local Ollama), specify the base URL of the API.
            *   **LLM Behavior Parameters:**
                *   `temperature` (Optional[float]): Controls randomness. Lower values (e.g., 0.2) make output more deterministic/focused; higher values (e.g., 0.8) make it more creative. For extraction, lower temperatures are usually preferred.
                *   `max_tokens` (Optional[int]): Maximum number of tokens to generate in the completion.
                *   `top_p` (Optional[float]): Nucleus sampling. An alternative to temperature.
                *   `frequency_penalty` (Optional[float]), `presence_penalty` (Optional[float]): Penalize new tokens based on their existing frequency or presence in the text so far, influencing topic diversity.
            *   **Choosing Parameters for Extraction:** For structured data extraction, you generally want the LLM to be factual and stick to the provided schema. Good starting points:
                *   `temperature`: 0.0 to 0.3
                *   `max_tokens`: Sufficient to cover your expected output size.

        *   4.2.3. The Art of the `instruction`: Guiding the LLM
            The `instruction` string you provide to `LLMExtractionStrategy` is critical. It's your primary way of telling the LLM what you want.
            *   **Why Clarity is Paramount:** LLMs are powerful but work best with clear, specific, and unambiguous instructions. Vague instructions lead to inconsistent or incorrect results.
            *   **Elements of a Good Extraction Instruction:**
                1.  **State the Goal Clearly:** "Extract the following information about each product..."
                2.  **Define Output Format (if not using a rigid schema for `extraction_type="block"`):** "Provide the output as a list of bullet points." or "Return a JSON object with keys 'name' and 'price'." (Though for JSON, using a Pydantic schema is better).
                3.  **Provide Examples (Few-Shot Prompting):** Show the LLM exactly what you mean. This is one of the most effective ways to improve accuracy.
                    ```
                    Instruction: "Extract the name and price from the text. Example:
                    Text: 'The SuperWidget costs $19.99 and is amazing.'
                    Output: {'name': 'SuperWidget', 'price': 19.99}"
                    ```
                4.  **Specify Handling of Missing/Ambiguous Data:** "If a price is not found, use null for the price field." or "If multiple authors are listed, return them as a list of strings."
                5.  **Be Concise but Complete:** Avoid unnecessary jargon, but ensure all critical details are present.
            *   **Examples: Good vs. Improvable Instructions:**
                *   *Improvable:* "Get product data."
                *   *Good:* "Extract the product name, price (as a float, omitting currency symbols), and a brief 2-sentence summary for each product listed in the provided HTML. If a price is not available, set the price field to null. Return the data as a list of JSON objects, each adhering to the Pydantic schema provided."

        *   4.2.4. Defining Your Target Output: `schema` (Pydantic Models) vs. `extraction_type="block"`
            `LLMExtractionStrategy` supports two main modes for `extraction_type`:
            *   **Schema-based Extraction (`extraction_type="schema"`, default):**
                *   **How it works:** You provide a Pydantic model to the `schema` parameter. Crawl4AI converts this model to a JSON schema and includes it in the prompt, instructing the LLM to format its output accordingly.
                *   **Benefits:**
                    *   **Structured Output:** Ensures the LLM returns data in a predictable, usable JSON format.
                    *   **Type Safety:** Pydantic validates the LLM's output against your defined types.
                    *   **Clarity:** Makes the desired output structure explicit to the LLM.
                *   **Code Example: Using a Pydantic model to extract author, title, and publication date from an article.**
                    ```python
                    from pydantic import BaseModel, Field
                    from typing import Optional
                    from datetime import date

                    class ArticleMeta(BaseModel):
                        title: str = Field(..., description="The main title of the article")
                        author: Optional[str] = Field(None, description="The primary author of the article")
                        publication_date: Optional[date] = Field(None, description="The date the article was published, in YYYY-MM-DD format")

                    # In LLMExtractionStrategy:
                    # llm_strategy = LLMExtractionStrategy(
                    #     llm_config=my_llm_config,
                    #     schema=ArticleMeta.model_json_schema(), # Pass the JSON schema representation
                    #     instruction="Extract article metadata according to the provided JSON schema.",
                    #     extraction_type="schema" 
                    # )
                    ```
                    *Self-correction: The `schema` parameter expects the JSON schema dictionary, not the Pydantic model class itself directly. `ArticleMeta.model_json_schema()` provides this.*
                    *(Further correction based on `crawl4ai/extraction_strategy.py` `LLMExtractionStrategy`): The `schema` parameter actually *can* take a Pydantic `BaseModel` type or a dictionary. The internal logic handles converting the Pydantic model to a JSON schema if needed. So, `schema=ArticleMeta` would also work, or even `schema=ArticleMeta.model_json_schema()`.*
                    For clarity and directness with Pydantic:
                    ```python
                    # Corrected usage for LLMExtractionStrategy with Pydantic
                    llm_strategy = LLMExtractionStrategy(
                        llm_config=my_llm_config,
                        schema=ArticleMeta, # Pass the Pydantic model class directly
                        instruction="Extract article metadata according to the provided Pydantic model structure.",
                        extraction_type="schema"
                    )
                    ```

            *   **Block-based Extraction (`extraction_type="block"`):**
                *   **When to use:** Useful when you want the LLM to identify and extract larger, coherent blocks of text rather than specific, fine-grained fields. Examples:
                    *   The main textual content of an article, excluding ads and sidebars.
                    *   All user reviews for a product.
                    *   A specific section of a long document based on a topic.
                *   **How it differs:** Instead of a rigid schema, your `instruction` guides the LLM on what kind of blocks to look for. The output will typically be a list of strings, where each string is an extracted block.
                *   **Code Example: Extracting all paragraphs discussing "environmental impact" from an article.**
                    ```python
                    # llm_strategy = LLMExtractionStrategy(
                    #     llm_config=my_llm_config,
                    #     instruction="Extract all paragraphs from the text that discuss the environmental impact of the product. Each paragraph should be a separate item in the output list.",
                    #     extraction_type="block" 
                    # )
                    ```
                    The `extracted_content` would then be a JSON string representing a list of text blocks, e.g., `["Paragraph 1 about impact...", "Another paragraph..."]`.

        *   4.2.5. Managing LLM Context: `ChunkingStrategy` in Action
            The `LLMExtractionStrategy` has two key parameters for controlling how it uses the `ChunkingStrategy`:
            *   **`chunk_token_threshold` (int, default from `config.CHUNK_TOKEN_THRESHOLD`):** This is the target maximum size (in tokens, roughly) for each chunk sent to the LLM. The `ChunkingStrategy` will try to create chunks that don't exceed this.
            *   **`overlap_rate` (float, default from `config.OVERLAP_RATE`):** This determines how much overlap there should be between consecutive chunks. An overlap (e.g., 0.1 for 10%) can help ensure that information at the boundaries of chunks isn't missed.
            *   **Strategies for Choosing Values:**
                *   Consult your LLM's documentation for its maximum context window size. Set `chunk_token_threshold` safely below this (e.g., 70-80% of the max).
                *   A small `overlap_rate` (e.g., 0.05 to 0.2) is often beneficial. Too much overlap increases redundant processing and cost.
                *   If your chosen `ChunkingStrategy` (like `RegexChunking` by paragraphs) naturally creates chunks much smaller than the `chunk_token_threshold`, the threshold might not be hit often, but it still acts as an upper bound.
            *   **Interaction with `ChunkingStrategy` implementations:**
                *   **`RegexChunking` (default for `LLMExtractionStrategy`):** It will first split the input document by its regex patterns (e.g., newlines, paragraphs). Then, it will try to merge these smaller pieces into chunks that are close to, but not exceeding, `chunk_token_threshold`, incorporating the `overlap_rate`.
                *   **`IdentityChunking`:** This strategy ignores `chunk_token_threshold` and `overlap_rate` and passes the content as a single chunk. Use this if your content is already appropriately sized or if your LLM handles very large contexts well for your task.
            *   **Code Example: Setting up chunking for a long article to be summarized by an LLM.**
                ```python
                from crawl4ai.chunking_strategy import RegexChunking
                # Assuming my_llm_config is defined
                
                # A chunker that aims for ~1500 token chunks with 10% overlap
                custom_chunker = RegexChunking(
                    # RegexChunking specific params can be set here if needed, 
                    # but LLMExtractionStrategy's params often suffice.
                )

                llm_summarizer_strategy = LLMExtractionStrategy(
                    llm_config=my_llm_config,
                    instruction="Summarize the following text block in 3 key bullet points.",
                    extraction_type="block", # We want blocks of summaries
                    chunking_strategy=custom_chunker, # Explicitly set if not default
                    chunk_token_threshold=1500, 
                    overlap_rate=0.1
                )
                ```

        *   4.2.6. Workflow Walkthrough:
            *   **Step 1: Define Your Extraction Goal and Target Schema/Output:**
                *   What specific information do you need? (e.g., product names, prices, features).
                *   If using `extraction_type="schema"`, create a Pydantic model.
                *   If using `extraction_type="block"`, define what characterizes a "block" you want.
            *   **Step 2: Configure `LLMConfig` and `LLMExtractionStrategy`:**
                *   Choose your LLM provider and model in `LLMConfig`.
                *   Set API keys and any custom `base_url`.
                *   Craft a clear `instruction` for `LLMExtractionStrategy`.
                *   Provide the `schema` (Pydantic model) or set `extraction_type="block"`.
                *   Configure `chunk_token_threshold`, `overlap_rate`, and select a `chunking_strategy` if the default isn't suitable.
            *   **Step 3: Integrate with `CrawlerRunConfig`:**
                ```python
                run_config = CrawlerRunConfig(
                    extraction_strategy=llm_strategy_instance,
                    # ... other run_config settings ...
                )
                ```
            *   **Step 4: Run the Crawl and Parse `extracted_content`:**
                ```python
                # result = await crawler.arun(url="...", config=run_config)
                # if result.success and result.extracted_content:
                #     try:
                #         extracted_data = json.loads(result.extracted_content)
                #         # Process extracted_data (which will be a list of dicts if schema-based,
                #         # or list of strings if block-based)
                #     except json.JSONDecodeError:
                #         print("LLM did not return valid JSON.")
                ```
            *   **Step 5: Analyze `TokenUsage`:**
                After the extraction (especially during development), inspect the `TokenUsage` object from the `LLMExtractionStrategy` instance to understand costs.
                ```python
                # llm_strategy_instance.show_usage() # Prints a summary
                # total_prompt_tokens = llm_strategy_instance.total_usage.prompt_tokens
                ```

        *   4.2.7. Code Example: Extracting Key Highlights from News Articles
            ```python
            import asyncio
            import json
            import os
            from pydantic import BaseModel, Field
            from typing import List, Optional
            from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig, LLMConfig
            from crawl4ai.extraction_strategy import LLMExtractionStrategy
            from crawl4ai.chunking_strategy import RegexChunking
            from crawl4ai.cache_manager import CacheMode

            class ArticleHighlight(BaseModel):
                highlight: str = Field(..., description="A key highlight or main point from the article.")
                category: Optional[str] = Field(None, description="A potential category for this highlight (e.g., Technology, Politics, Sports)")

            class ArticleHighlights(BaseModel):
                article_title: Optional[str] = Field(None, description="The main title of the article, if identifiable.")
                highlights: List[ArticleHighlight] = Field(..., description="A list of 3-5 key highlights from the article.")

            async def extract_article_highlights():
                # Ensure OPENAI_API_KEY is set in your environment
                if not os.getenv("OPENAI_API_KEY"):
                    print("OPENAI_API_KEY environment variable not set. Skipping LLM example.")
                    return

                llm_config = LLMConfig(
                    provider="openai/gpt-3.5-turbo", # More cost-effective for this example
                    api_token=os.getenv("OPENAI_API_KEY"),
                    temperature=0.2
                )

                extraction_strategy = LLMExtractionStrategy(
                    llm_config=llm_config,
                    schema=ArticleHighlights, # Pass the Pydantic model class
                    instruction="From the provided news article content, identify the main title and extract 3 to 5 key highlights. For each highlight, also try to assign a general category.",
                    extraction_type="schema",
                    chunking_strategy=RegexChunking(), # Default, but explicit here
                    chunk_token_threshold=2000, # Adjust based on article length and model
                    overlap_rate=0.1,
                    input_format="markdown" # LLMs often work well with clean Markdown
                )

                browser_config = BrowserConfig(headless=True, user_agent_mode="random") # Use a real user agent
                run_config = CrawlerRunConfig(
                    extraction_strategy=extraction_strategy,
                    cache_mode=CacheMode.BYPASS, # Fresh crawl for demo
                    word_count_threshold=50 # Ensure we have some content
                )

                # A news article known for having decent text content
                news_url = "https://www.nbcnews.com/tech/tech-news" 

                async with AsyncWebCrawler(config=browser_config) as crawler:
                    print(f"Crawling {news_url} to extract highlights...")
                    result = await crawler.arun(url=news_url, config=run_config)

                if result.success and result.extracted_content:
                    try:
                        data = json.loads(result.extracted_content)
                        # Since we expect a single ArticleHighlights object from the whole page
                        if isinstance(data, list) and len(data) > 0: 
                             # LiteLLM might wrap single objects in a list if schema is complex, take first.
                            article_data = ArticleHighlights.model_validate(data[0])
                        elif isinstance(data, dict):
                            article_data = ArticleHighlights.model_validate(data)
                        else:
                            raise ValueError("Unexpected data format from LLM")

                        print(f"\nExtracted Highlights for: {article_data.article_title or 'Unknown Title'}")
                        for hl_obj in article_data.highlights:
                            print(f"  - [{hl_obj.category or 'General'}] {hl_obj.highlight}")
                        
                        extraction_strategy.show_usage() # Show token usage

                    except (json.JSONDecodeError, ValueError) as e:
                        print(f"Error parsing LLM output: {e}")
                        print("Raw LLM output:", result.extracted_content)
                elif result.success and not result.extracted_content:
                    print("LLM extraction returned no content. The page might have been too short or content unsuitable.")
                else:
                    print(f"Failed to crawl or extract: {result.error_message}")

            if __name__ == "__main__":
                asyncio.run(extract_article_highlights())
            ```

        *   4.2.8. Understanding and Optimizing Costs: The `TokenUsage` Model
            When using LLMs, especially commercial APIs, tracking token usage is vital for cost management. The `TokenUsage` model (from `crawl4ai.models`) stores this information.
            *   **Fields:**
                *   `prompt_tokens` (int): Number of tokens in the input prompt sent to the LLM.
                *   `completion_tokens` (int): Number of tokens in the output generated by the LLM.
                *   `total_tokens` (int): Sum of prompt and completion tokens.
                *   `prompt_tokens_details`, `completion_tokens_details` (Optional[dict]): Provider-specific detailed token counts if available.
            *   **How to Interpret:** After an `LLMExtractionStrategy` run, you can access `strategy_instance.total_usage` for aggregated counts across all chunks/calls, or `strategy_instance.usages` for a list of `TokenUsage` objects per call.
                ```python
                # After running the strategy
                llm_strategy.show_usage() 
                # print(f"Total prompt tokens: {llm_strategy.total_usage.prompt_tokens}")
                # print(f"Total completion tokens: {llm_strategy.total_usage.completion_tokens}")
                ```
            *   **Strategies for Reducing Token Consumption:**
                1.  **Precise Prompts/Instructions:** Shorter, more focused prompts consume fewer tokens.
                2.  **Efficient Chunking:** Optimize `chunk_token_threshold` and `overlap_rate`. Avoid overly small chunks (too many API calls) or excessive overlap.
                3.  **Pre-filtering Content:** If possible, use non-LLM methods (CSS, XPath) to isolate the most relevant sections of HTML *before* sending to the LLM. Pass this cleaner, shorter text.
                4.  **Choose Smaller/Cheaper Models:** For simpler extraction tasks, a less powerful (and cheaper) model might suffice (e.g., GPT-3.5-turbo instead of GPT-4, or a smaller Llama variant).
                5.  **Limit `max_tokens` in `LLMConfig`:** If you know your expected output is short, set a reasonable `max_tokens` to prevent the LLM from generating overly verbose responses.
                6.  **Ask for Concise Output:** Instruct the LLM to be brief or to only return the specified fields.

        *   4.2.9. Best Practices for `LLMExtractionStrategy`
            *   **Iterative Prompt Refinement:** Start with a simple prompt and schema. Test it. Refine the prompt based on the LLM's output until you get the desired results. This is often a trial-and-error process.
            *   **Few-Shot Examples:** Including 2-3 examples of desired input/output *within your instruction* can dramatically improve LLM performance and adherence to your schema.
            *   **Specificity is Key:** The more specific your instruction and schema (especially field descriptions in Pydantic models), the better the LLM will understand your intent.
            *   **Model Selection:** Different LLMs excel at different tasks. Some are better at following complex instructions, others at creative generation. Experiment if results aren't optimal. For pure extraction into a schema, models fine-tuned for function calling or JSON mode are often best.
            *   **Handle Failures Gracefully:** LLM outputs can sometimes be unpredictable. Implement try-except blocks for JSON parsing and Pydantic validation. Consider fallback logic if extraction fails.
            *   **Use `input_format` Wisely:**
                *   `input_format="markdown"` (default for `LLMExtractionStrategy` if `CrawlerRunConfig.markdown_generator` is set): Good for general text extraction, as Markdown is cleaner than raw HTML.
                *   `input_format="html"`: Useful if the LLM needs to see HTML tags (e.g., for extracting attributes or if table structure is critical and Markdown conversion loses it).
                *   `input_format="text"`: For when you only care about the raw textual content.
                *   `input_format="fit_html"`: Uses a preprocessed HTML more suitable for schema extraction, usually smaller.

        *   4.2.10. Troubleshooting LLM Extraction:
            *   **LLM Not Following Instructions / Incorrect Format:**
                *   *Cause:* Prompt too vague, ambiguous, or complex. LLM might not support forced JSON mode well (though LiteLLM tries to handle this).
                *   *Solution:* Simplify prompt. Add clear examples (few-shot). Use a Pydantic schema to strongly guide JSON output. Try a different model. Ensure `force_json_response=True` in `LLMExtractionStrategy` if your provider supports it robustly or if you are using a Pydantic schema.
            *   **Incorrect or Incomplete Data:**
                *   *Cause:* Instruction missing details, LLM misunderstanding, content chunking splitting relevant info.
                *   *Solution:* Refine instruction. Check `chunk_token_threshold` and `overlap_rate`. Ensure field descriptions in Pydantic schema are clear.
            *   **Handling Hallucinations or Fabricated Information:**
                *   *Cause:* LLMs can sometimes "invent" data if it's not present or if the prompt is leading.
                *   *Solution:* Instruct the LLM to use `null` or a specific placeholder (e.g., "N/A") for missing fields. Lower the `temperature`. Validate extracted data against known facts if possible.
            *   **Schema Validation Errors (Pydantic):**
                *   *Cause:* LLM output doesn't match the Pydantic model's types or constraints.
                *   *Solution:* Check the LLM's raw JSON output. Refine the prompt to better match the schema. Make Pydantic fields `Optional` if data might be missing.
            *   **API Errors / Rate Limits:**
                *   *Cause:* Invalid API key, insufficient credits, hitting provider rate limits.
                *   *Solution:* Check API key and account status. Implement backoff/retry logic (Crawl4AI does some of this internally). Reduce request frequency.

## 5. Choosing Your Extraction Weapon: A Decision Guide
    * 5.1. Factors to Consider:
        *   **Structure and Consistency of Target Data:**
            *   *Well-structured, consistent HTML?* => Favor Non-LLM (CSS, XPath).
            *   *Messy, inconsistent, or unstructured text?* => Favor LLM.
        *   **Complexity of Information to be Extracted:**
            *   *Simple fields, direct attributes?* => Non-LLM.
            *   *Nuanced relationships, summaries, sentiment, inferred data?* => LLM.
        *   **Development Time vs. Runtime Cost:**
            *   *Quick prototype needed, site structure complex/unknown?* => LLM can be faster to start.
            *   *High volume, long-term, cost-sensitive?* => Non-LLM, once set up, is cheaper to run.
        *   **Need for Semantic Understanding vs. Pattern Matching:**
            *   *Data identifiable by patterns (emails, dates, SKUs)?* => `RegexExtractionStrategy`.
            *   *Data requires understanding context or meaning?* => LLM.
        *   **Scalability and Performance Requirements:**
            *   *Need to scrape thousands of pages per minute?* => Non-LLM strategies are inherently faster. LLM API calls add latency.
            *   *Occasional or smaller-scale extraction?* => LLM latency might be acceptable.
        *   **Maintainability:**
            *   *Site changes frequently?* => LLM prompts *might* be more resilient than specific CSS/XPath selectors, but both can break. Regex is often robust if the underlying text pattern is stable.
        *   **Team Expertise:**
            *   *Strong in CSS/XPath/Regex?* => Leverage those skills with Non-LLM.
            *   *More comfortable with natural language prompts?* => LLM might be a good fit.

    * 5.2. Decision Table: Non-LLM vs. LLM Strategies
        | Feature                | Non-LLM (CSS, XPath, Regex)                       | LLM-Based (`LLMExtractionStrategy`)            |
        | ---------------------- | ------------------------------------------------- | ---------------------------------------------- |
        | **Best For**           | Well-structured, consistent HTML; pattern matching | Unstructured/complex data; semantic understanding |
        | **Development Speed**  | Slower if selectors are complex; faster for regex | Faster for initial prototype with good prompts    |
        | **Runtime Speed**      | Very Fast                                         | Slower (API latency, model inference)        |
        | **Runtime Cost**       | Negligible (CPU/Mem)                              | Can be significant (API calls, GPU if local)   |
        | **Accuracy**           | High if selectors are good; precise for regex   | Depends on prompt, model, content quality    |
        | **Resilience to Change**| Brittle to HTML changes (CSS/XPath)               | Potentially more resilient; prompt dependent   |
        | **Complexity Handled** | Lower for semantic, higher for pattern (regex)    | High for semantic and complex relationships    |
        | **Schema Enforcement** | Via schema definition                             | Strong via Pydantic schema; flexible otherwise |

    * 5.3. Hybrid Approaches: Combining the Best of Both Worlds
        Often, the most robust and efficient solution involves a hybrid approach:
        *   **Example 1: CSS/XPath Pre-filtering for LLM:**
            Use CSS or XPath selectors to isolate the main content block of an article (e.g., `<article class="main-story">`). Pass only this cleaned, focused HTML/Markdown to the `LLMExtractionStrategy`.
            *   *Why?* Reduces the amount of text the LLM needs to process, saving tokens (cost/latency) and potentially improving accuracy by removing noise.
            ```python
            # Conceptual - how you might structure the thought process
            # 1. Use AsyncWebCrawler with a CrawlerRunConfig that only does basic scraping (no LLM extraction yet)
            #    and uses a css_selector to get the main content.
            # 2. Get the result.cleaned_html (which is now just the main content).
            # 3. Pass this cleaned_html to a separate LLMExtractionStrategy call.
            # (Crawl4AI doesn't directly support "chaining" strategies in one run_config,
            # so this would involve multiple processing steps orchestrated by your code.)
            ```
        *   **Example 2: Regex for Simple Entities, LLM for Complex:**
            Use `RegexExtractionStrategy` to quickly and cheaply pull out all email addresses, phone numbers, and dates. Then, use `LLMExtractionStrategy` on the remaining text (or the full text) to extract more nuanced information like "the primary topic of discussion" or "the relationship between person A and company B."
        *   **How to Implement Hybrid:** Typically, you would run the crawl in stages or have a custom orchestrator.
            1.  First pass: Use a non-LLM strategy (e.g., `JsonCssExtractionStrategy` to get specific blocks, or just rely on `result.markdown`).
            2.  Second pass: Take the output from the first pass and feed it to an `LLMExtractionStrategy` (or another non-LLM strategy). You might do this by invoking the `extract` method of the second strategy directly with the content from the first.

## 6. The `NoExtractionStrategy`: When You Just Need the HTML/Markdown
    * 6.1. Purpose: Disabling structured data extraction.
        The `NoExtractionStrategy` (from `crawl4ai.extraction_strategy`) is a placeholder strategy that, as its name suggests, performs no actual data extraction. `result.extracted_content` will be `None` or an empty representation.
    * 6.2. Use Cases:
        *   **Archiving Raw Web Content:** If your goal is simply to fetch and store the raw HTML or the cleaned Markdown of pages.
        *   **Markdown Generation is Primary:** If you're primarily using Crawl4AI for its HTML-to-Markdown conversion capabilities and don't need structured data beyond that.
        *   **Feeding to External Pipelines:** If you have a separate, downstream system that will handle the data extraction and you just need Crawl4AI to fetch and pre-process the web pages.
        *   **Baseline/Testing:** Useful as a baseline when developing or debugging other parts of your crawling pipeline.
    * 6.3. How to Configure It.
        ```python
        from crawl4ai import CrawlerRunConfig
        from crawl4ai.extraction_strategy import NoExtractionStrategy

        run_config_no_extraction = CrawlerRunConfig(
            extraction_strategy=NoExtractionStrategy()
        )
        # When crawler.arun(url="...", config=run_config_no_extraction) is called,
        # result.extracted_content will likely be None.
        # You would primarily use result.html or result.markdown.
        ```

## 7. Integrating Extraction into Your Crawls
    * 7.1. The Role of `CrawlerRunConfig`
        The `CrawlerRunConfig` object is central to customizing how each individual crawl operation behaves. For extraction, its key parameters are:
        *   **`extraction_strategy: Optional[ExtractionStrategy]`:** You assign an instance of your chosen extraction strategy here (e.g., `JsonCssExtractionStrategy(...)`, `LLMExtractionStrategy(...)`). If `None`, no structured extraction specific to this strategy is performed, but default behaviors like Markdown generation might still occur.
        *   **`chunking_strategy: Optional[ChunkingStrategy]`:** Primarily used by `LLMExtractionStrategy`. If you want to use a non-default chunker (other than `RegexChunking`), you instantiate it and assign it here.
        *   **`input_format` (within `LLMExtractionStrategy`):** While not directly in `CrawlerRunConfig`, the `LLMExtractionStrategy` itself takes an `input_format` parameter (`"markdown"`, `"html"`, `"text"`, `"fit_html"`) that determines what version of the page content is fed to the LLM. `CrawlerRunConfig`'s `markdown_generator` influences the quality of the Markdown available.

    * 7.2. Data Flow: From Web Page to Extracted Data
        Here's a simplified conceptual data flow:
        ```
        [Web Page URL]
             |
             v
        AsyncWebCrawler.arun(config=CrawlerRunConfig)
             |
             v
        [Browser Engine (Playwright)] -- Fetches HTML, executes JS --> [Raw HTML]
             |
             v
        CrawlerRunConfig.scraping_strategy (e.g., WebScrapingStrategy)
             |--> Cleans HTML --> [Cleaned HTML]
             |--> (Optional) Generates Markdown via CrawlerRunConfig.markdown_generator --> [Markdown]
             |--> Extracts Links, Basic Media --> [Links, Media Objects]
             |
             v
        (If LLMExtractionStrategy with chunking)
        CrawlerRunConfig.chunking_strategy / LLMExtractionStrategy.chunking_strategy
             |--> Chunks the input_format content (e.g., Markdown) --> [List of Text Chunks]
             |
             v
        CrawlerRunConfig.extraction_strategy (e.g., LLMExtractionStrategy or JsonCssExtractionStrategy)
             |--> Processes HTML/Markdown/Chunks --> [Structured Data (JSON String)]
             |
             v
        [CrawlResult]
             - .html (raw)
             - .cleaned_html
             - .markdown (object with .raw_markdown, .fit_markdown etc.)
             - .extracted_content (JSON string from extraction_strategy)
             - .links
             - .media
        ```

    * 7.3. Complete Code Example: A Full Crawl with a Chosen Extraction Strategy
        ```python
        import asyncio
        import json
        from crawl4ai import (
            AsyncWebCrawler, BrowserConfig, CrawlerRunConfig, LLMConfig, CacheMode
        )
        from crawl4ai.extraction_strategy import JsonCssExtractionStrategy, LLMExtractionStrategy
        from crawl4ai.chunking_strategy import RegexChunking
        from pydantic import BaseModel, Field
        from typing import List, Optional

        # Define a Pydantic schema for LLM extraction
        class NewsSummary(BaseModel):
            title: str = Field(description="The main headline of the news article.")
            summary_points: List[str] = Field(description="A list of 2-3 key bullet points summarizing the article.")

        async def comprehensive_extraction_example():
            # --- Configuration ---
            browser_config = BrowserConfig(
                headless=True,
                user_agent_mode="random" # Use a realistic user agent
            )

            # Non-LLM: CSS-based extraction schema for basic info
            basic_info_schema = {
                "name": "PageLinks",
                "baseSelector": "a[href]", # Get all links
                "fields": [
                    {"name": "text", "selector": "self", "type": "text"}, # 'self' refers to the baseSelector element
                    {"name": "href", "selector": "self", "type": "attribute", "attribute": "href"}
                ]
            }
            css_extraction_strategy = JsonCssExtractionStrategy(schema=basic_info_schema)

            # LLM-based extraction for summarization (ensure API key is set for OpenAI)
            llm_config_openai = LLMConfig(provider="openai/gpt-3.5-turbo", api_token=os.getenv("OPENAI_API_KEY"))
            if not llm_config_openai.api_token: # Fallback to a local/free model if no key
                print("Warning: OPENAI_API_KEY not found. LLM summarization might be skipped or use a different model if configured.")
                # Optionally, configure a fallback like Ollama here if you have it running
                # llm_config_ollama = LLMConfig(provider="ollama/llama2", base_url="http://localhost:11434")
                # llm_summarization_strategy = LLMExtractionStrategy(...) using llm_config_ollama
                llm_summarization_strategy = None # Or a NoExtractionStrategy
            else:
                llm_summarization_strategy = LLMExtractionStrategy(
                    llm_config=llm_config_openai,
                    schema=NewsSummary, # Use Pydantic model
                    instruction="Analyze the provided news article content (likely in Markdown). Extract its main title and provide 2-3 key summary bullet points.",
                    extraction_type="schema",
                    chunking_strategy=RegexChunking(), # Default, good for articles
                    chunk_token_threshold=1500,
                    input_format="markdown"
                )
            
            # --- Create CrawlerRunConfig ---
            # We'll demonstrate two runs: one with CSS, one with LLM
            run_config_css = CrawlerRunConfig(
                extraction_strategy=css_extraction_strategy,
                cache_mode=CacheMode.BYPASS
            )
            run_config_llm = CrawlerRunConfig(
                extraction_strategy=llm_summarization_strategy,
                cache_mode=CacheMode.BYPASS
            )

            target_url = "https://www.bbc.com/news" # Example news site

            # --- Execute Crawl ---
            async with AsyncWebCrawler(config=browser_config) as crawler:
                print(f"--- Running CSS Extraction on {target_url} ---")
                result_css = await crawler.arun(url=target_url, config=run_config_css)
                if result_css.success and result_css.extracted_content:
                    links = json.loads(result_css.extracted_content)
                    print(f"Found {len(links)} links. First 3:")
                    for link_data in links[:3]:
                        print(f"  Text: {link_data.get('text', '')[:30]}..., Href: {link_data.get('href')}")
                else:
                    print(f"CSS Extraction failed or no content: {result_css.error_message}")

                if llm_summarization_strategy: # Only run if LLM is configured
                    print(f"\n--- Running LLM Summarization on {target_url} (using its Markdown) ---")
                    # The LLM strategy will use the Markdown from the previous crawl result if input_format is markdown
                    # or it would re-fetch if it was a different format or strategy.
                    # For simplicity here, we assume the crawler internally handles content reuse or re-fetch as needed
                    # based on the input_format.
                    # A more explicit way would be to pass result_css.markdown to llm_summarization_strategy.extract()
                    
                    result_llm = await crawler.arun(url=target_url, config=run_config_llm)
                    if result_llm.success and result_llm.extracted_content:
                        try:
                            summary_data_list = json.loads(result_llm.extracted_content)
                            # LLM might return a list if it finds multiple "articles" or if schema is treated as listable
                            if summary_data_list and isinstance(summary_data_list, list):
                                summary_data = NewsSummary.model_validate(summary_data_list[0]) # Take first for demo
                                print(f"Title: {summary_data.article_title}")
                                print("Summary Points:")
                                for point in summary_data.summary_points:
                                    print(f"  - {point}")
                            elif summary_data_list and isinstance(summary_data_list, dict): # Single object returned
                                summary_data = NewsSummary.model_validate(summary_data_list)
                                print(f"Title: {summary_data.article_title}")
                                print("Summary Points:")
                                for point in summary_data.summary_points:
                                    print(f"  - {point}")

                        except (json.JSONDecodeError, Exception) as e: # Broader exception for Pydantic validation
                            print(f"Error parsing LLM summary output: {e}")
                            print("Raw LLM output:", result_llm.extracted_content)
                        llm_summarization_strategy.show_usage()
                    else:
                        print(f"LLM Summarization failed or no content: {result_llm.error_message}")
                else:
                    print("\nLLM Summarization strategy not configured, skipping that part.")


        if __name__ == "__main__":
            asyncio.run(comprehensive_extraction_example())
        ```

## 8. Specialized Extraction: Working with PDF Content
    * 8.1. Understanding PDF Processing in Crawl4AI
        Crawl4AI provides dedicated strategies for handling PDF documents, as PDFs are a common format for reports, papers, and other important web content. The key components are:
        *   **`PDFCrawlerStrategy` (in `crawl4ai.processors.pdf.__init__.py`):**
            *   **Role:** This strategy is used as the `crawler_strategy` in `AsyncWebCrawler` when you intend to directly process a PDF URL. It doesn't crawl HTML pages to find PDFs; rather, it's designed to fetch a document *known* to be a PDF (or a URL that might serve a PDF). It primarily handles the downloading of the PDF content. The actual parsing is delegated to a "scraping" strategy.
            *   It sets the `Content-Type` in the response headers to `application/pdf` to signal to subsequent strategies that this is PDF content.
        *   **`PDFContentScrapingStrategy` (in `crawl4ai.processors.pdf.__init__.py`):**
            *   **Role:** This strategy is used as the `scraping_strategy` in `CrawlerRunConfig` when you're targeting PDFs. It takes the raw PDF bytes (fetched by `PDFCrawlerStrategy` or provided directly) and processes them.
            *   **Leveraging `NaivePDFProcessorStrategy`:** Internally, `PDFContentScrapingStrategy` uses `NaivePDFProcessorStrategy` (from `crawl4ai.processors.pdf.processor`) to do the heavy lifting of PDF parsing.
            *   **`NaivePDFProcessorStrategy` (from `crawl4ai.processors.pdf.processor`):** This is the workhorse. It uses the PyPDF2 library (and Pillow for images) to extract:
                *   **Text Content:** Page by page.
                *   **Images:** Can extract embedded images.
                *   **Metadata:** Document properties like title, author, creation date.
            *   **Key Outputs in `ScrapingResult`:** When `PDFContentScrapingStrategy` is used, the `ScrapingResult` object (which is then available as `result.cleaned_html` or `result.markdown` to the `ExtractionStrategy`, and also structured in `result.metadata` and `result.media`) will be populated as follows:
                *   `result.cleaned_html`: Contains an HTML representation of the PDF content, with each page typically wrapped in a `<div class="pdf-page">`. Images might be embedded as base64 or linked if saved locally.
                *   `result.markdown`: A Markdown representation of the PDF text content (via `DefaultMarkdownGenerator` applied to the HTML from `cleaned_html`).
                *   `result.metadata`: A dictionary containing metadata extracted from the PDF, mirroring the `PDFMetadata` model (title, author, pages, etc.).
                *   `result.media`: Will contain image information under `media["images"]` if image extraction is enabled.

    * 8.2. Configuring PDF Extraction
        Configuration options are primarily set on the `PDFContentScrapingStrategy` (which passes them to `NaivePDFProcessorStrategy`).
        *   **`extract_images` (bool, default: `False`):** Set to `True` to attempt to extract images from the PDF. This can increase processing time.
        *   **`save_images_locally` (bool, default: `False`):** If `extract_images` is `True`, setting this to `True` will save extracted images to disk.
        *   **`image_save_dir` (Optional[str], default: `None`):** Specifies the directory to save images if `save_images_locally` is `True`. If `None`, a temporary directory might be used by `NaivePDFProcessorStrategy` (or it might use a default configured path if the strategy has one). It's best to provide an explicit path.
        *   **`image_dpi` (int, default: `144` in `NaivePDFProcessorStrategy`):** Dots Per Inch for rendered images (if PDF pages are rendered as images, which is not the primary mode of `NaivePDFProcessorStrategy`'s image extraction; it usually extracts existing embedded images. This DPI might be more relevant for future strategies that render pages). For `NaivePDFProcessorStrategy`, this DPI is used if it falls back to rendering pages as images, for example if direct image extraction fails or for specific image types.
        *   **`batch_size` (int, default: `4` in `NaivePDFProcessorStrategy`):** Controls how many pages are processed in parallel by worker threads when using `process_batch`. This can speed up processing of multi-page PDFs.

        ```python
        from crawl4ai.processors.pdf import PDFContentScrapingStrategy

        pdf_scraping_config = PDFContentScrapingStrategy(
            extract_images=True,
            save_images_locally=True,
            image_save_dir="./pdf_extracted_images", # Ensure this directory exists
            # image_dpi=200 # Higher DPI for better quality, larger files
            batch_size=8    # Process more pages in parallel
        )
        ```

    * 8.3. Workflow: Extracting Content from PDFs
        1.  **Set `PDFCrawlerStrategy` in `AsyncWebCrawler`:** This tells the crawler to use the PDF-specific fetching logic.
            ```python
            from crawl4ai.processors.pdf import PDFCrawlerStrategy
            # crawler = AsyncWebCrawler(crawler_strategy=PDFCrawlerStrategy())
            ```
        2.  **Set `PDFContentScrapingStrategy` in `CrawlerRunConfig`:** This tells the scraping phase to use the PDF parser.
            ```python
            # run_config = CrawlerRunConfig(scraping_strategy=pdf_scraping_config)
            ```
        3.  **Run the Crawl:**
            ```python
            # result = await crawler.arun(url="https://example.com/mydoc.pdf", config=run_config)
            ```
        4.  **Accessing Extracted Data:**
            *   **Text:** `result.markdown.raw_markdown` (often the most useful for LLMs) or iterate through `result.cleaned_html` to get page-specific HTML.
            *   **Metadata:** `result.metadata` will be a dictionary (e.g., `result.metadata.get("title")`). This comes from `PDFProcessResult.metadata`.
            *   **Images:** `result.media["images"]` will be a list of image dictionaries if `extract_images=True`. Each image dict might contain `src` (path if saved locally, or base64 data URI), `alt`, `page` (page number where image was found).

    * 8.4. Code Example: Crawling a PDF and Extracting its Text and Metadata
        ```python
        import asyncio
        import os
        from pathlib import Path
        from crawl4ai import AsyncWebCrawler, BrowserConfig, CrawlerRunConfig, CacheMode
        from crawl4ai.processors.pdf import PDFCrawlerStrategy, PDFContentScrapingStrategy

        async def crawl_and_extract_pdf():
            # Example PDF URL (replace with a real, accessible PDF URL for testing)
            # For this example, let's assume a local PDF file to avoid network dependency.
            # Create a dummy PDF for testing if you don't have one handy
            # (Actual PDF creation is outside Crawl4AI scope, this is just for the example)
            
            # For a real URL:
            # pdf_url = "https://arxiv.org/pdf/1706.03762.pdf" # "Attention is All You Need" paper
            
            # For a local file:
            dummy_pdf_path = Path("dummy_test.pdf")
            if not dummy_pdf_path.exists():
                 try:
                    from reportlab.pdfgen import canvas
                    c = canvas.Canvas(str(dummy_pdf_path))
                    c.drawString(100, 750, "Hello World. This is page 1 of a dummy PDF.")
                    c.showPage()
                    c.drawString(100, 750, "This is page 2 with an important keyword: Crawl4AI.")
                    c.save()
                    print(f"Created dummy PDF: {dummy_pdf_path.resolve()}")
                 except ImportError:
                    print("reportlab not installed. Cannot create dummy PDF. Please provide a real PDF URL or local path.")
                    return

            pdf_url = f"file://{dummy_pdf_path.resolve()}"


            # Configure PDF processing
            pdf_image_output_dir = Path("./pdf_images_output")
            pdf_image_output_dir.mkdir(parents=True, exist_ok=True)

            pdf_scraping = PDFContentScrapingStrategy(
                extract_images=True, 
                save_images_locally=True, 
                image_save_dir=str(pdf_image_output_dir)
            )

            # Configure crawler run
            pdf_run_config = CrawlerRunConfig(
                scraping_strategy=pdf_scraping,
                cache_mode=CacheMode.BYPASS # Ensure fresh processing for demo
            )

            # Use PDFCrawlerStrategy for direct PDF handling
            # Note: BrowserConfig is less relevant here if directly fetching PDF, 
            # but AsyncWebCrawler still needs it.
            browser_cfg = BrowserConfig(headless=True) 
            
            async with AsyncWebCrawler(
                config=browser_cfg, 
                crawler_strategy=PDFCrawlerStrategy() # Crucial for PDF URLs
            ) as crawler:
                print(f"Processing PDF: {pdf_url}")
                result = await crawler.arun(url=pdf_url, config=pdf_run_config)

            if result.success:
                print("\n--- PDF Processing Successful ---")
                print(f"URL Processed: {result.url}")
                
                # Access metadata
                if result.metadata:
                    print("\nMetadata:")
                    print(f"  Title: {result.metadata.get('title', 'N/A')}")
                    print(f"  Author: {result.metadata.get('author', 'N/A')}")
                    print(f"  Pages: {result.metadata.get('pages', 'N/A')}")

                # Access text (via Markdown)
                if result.markdown:
                    print(f"\nMarkdown Content (first 300 chars):\n{result.markdown.raw_markdown[:300]}...")
                
                # Access images
                if result.media and result.media.get("images"):
                    print(f"\nExtracted {len(result.media['images'])} image(s):")
                    for img_info in result.media["images"]:
                        print(f"  - Src: {img_info.get('src', 'N/A')} (Page: {img_info.get('page', 'N/A')})")
                else:
                    print("\nNo images extracted or found.")
            else:
                print(f"\n--- PDF Processing Failed ---")
                print(f"Error: {result.error_message}")

            # Clean up dummy PDF
            if dummy_pdf_path.exists():
                # dummy_pdf_path.unlink() # Commented out to allow inspection
                print(f"Dummy PDF at {dummy_pdf_path.resolve()} can be manually deleted.")


        if __name__ == "__main__":
            asyncio.run(crawl_and_extract_pdf())
        ```

    * 8.5. When to Combine PDF Processing with Other Extraction Strategies
        The output of `PDFContentScrapingStrategy` (specifically `result.markdown.raw_markdown` or `result.cleaned_html`) can be fed into *another* `ExtractionStrategy` for more refined data extraction.
        *   **Using `LLMExtractionStrategy` on PDF Text:**
            *   *Why:* PDFs often contain unstructured text. An LLM can summarize, answer questions, or extract specific entities from the PDF's textual content.
            *   *How:*
                1.  Crawl the PDF using `PDFCrawlerStrategy` and `PDFContentScrapingStrategy`.
                2.  Take `result.markdown.raw_markdown`.
                3.  Instantiate an `LLMExtractionStrategy` with your desired schema and instruction.
                4.  Call `llm_strategy.extract(url=pdf_url, html_content=result.markdown.raw_markdown)` (using `html_content` as the parameter name, even though it's Markdown here, or ensure your LLM strategy is configured for `input_format="markdown"`).
        *   **Applying `RegexExtractionStrategy` to PDF Text:**
            *   *Why:* To find specific patterns (emails, phone numbers, case IDs, etc.) within the extracted text of the PDF.
            *   *How:* Similar to the LLM approach, use the text output from PDF processing as input to `RegexExtractionStrategy.extract()`.

## 9. Advanced Customization: Building Your Own Strategies
    * 9.1. Implementing a Custom `ExtractionStrategy`
        *   9.1.1. Why Create a Custom Strategy?
            *   **Unsupported Data Formats:** You're dealing with a data format Crawl4AI doesn't natively understand (e.g., custom binary formats, obscure XML dialects, non-standard text encodings that need special pre-processing).
            *   **Proprietary Internal APIs:** Your target data comes from an internal system with a unique API response structure that doesn't map well to JSON/CSS/XPath.
            *   **Highly Domain-Specific Logic:** The extraction rules are too complex or specific to your domain to be easily expressed with general-purpose selectors or even LLM prompts (e.g., extracting data from scientific diagrams based on their visual components, which might require CV models).
            *   **Performance-Critical Custom Parsing:** For extremely high-volume scraping of a single, known format, a hand-tuned parser might outperform general tools.

        *   9.1.2. Key Steps:
            1.  **Inherit from `ExtractionStrategy`:**
                ```python
                from crawl4ai.extraction_strategy import ExtractionStrategy, LLMConfig # Assuming LLMConfig is needed
                from typing import List, Dict, Any
                
                class MyCustomExtractionStrategy(ExtractionStrategy):
                    # ...
                ```
            2.  **Implement `__init__` (Optional but common):**
                To accept any configuration your strategy needs.
                ```python
                # def __init__(self, my_param: str, **kwargs):
                #     super().__init__(**kwargs) # Pass kwargs for base class (like input_format)
                #     self.my_param = my_param
                ```
            3.  **Implement the `extract` method:** This is the core logic.
                ```python
                # def extract(self, url: str, html_content: str, *q, **kwargs) -> List[Dict[str, Any]]:
                #     # Your custom parsing logic here
                #     # html_content will be whatever 'input_format' you specified (e.g., 'html', 'markdown')
                #     # or the raw content if not specified.
                #     processed_data = []
                #     # ... parse html_content ...
                #     # Example:
                #     # if "special_keyword" in html_content:
                #     #     processed_data.append({"url": url, "found_keyword": True, "snippet": html_content[:100]})
                #     return processed_data
                ```
            4.  **Implement `run` method (Optional):**
                The base `ExtractionStrategy.run` method simply takes a list of `sections` (chunks) and calls `self.extract` on their concatenation. You might override `run` if:
                *   You want to process chunks in parallel.
                *   Your strategy inherently works on chunks and needs to aggregate results differently.
                *   You need to manage state across chunk processing.
                ```python
                # async def run(self, url: str, sections: List[str], *q, **kwargs) -> List[Dict[str, Any]]:
                #     # Example: Process sections in parallel (conceptual, requires async/threading)
                #     all_results = []
                #     # In a real async scenario, you'd use asyncio.gather or similar
                #     for section in sections:
                #         # Note: self.extract is not async by default in base class. 
                #         # If your extract is I/O bound and async, you can await it.
                #         # Otherwise, use to_thread or a ThreadPoolExecutor for true parallelism.
                #         # For simplicity, this example is synchronous.
                #         all_results.extend(self.extract(url, section, **kwargs)) 
                #     return all_results
                ```
                *Note:* The base `ExtractionStrategy.run` is synchronous. If your custom `extract` method is I/O bound and you want true parallelism in `run`, you'll need to handle `asyncio` or threading appropriately. The `LLMExtractionStrategy` has a more complex `run` method for handling LLM calls.

        *   9.1.3. Simple Code Example: A Custom Strategy to Extract All `<meta>` Tags
            ```python
            import asyncio
            from bs4 import BeautifulSoup
            from crawl4ai import AsyncWebCrawler, CrawlerRunConfig
            from crawl4ai.extraction_strategy import ExtractionStrategy
            from typing import List, Dict, Any

            class MetaTagExtractor(ExtractionStrategy):
                def __init__(self, **kwargs):
                    # This strategy will work on HTML
                    super().__init__(input_format="html", **kwargs)

                def extract(self, url: str, html_content: str, *q, **kwargs) -> List[Dict[str, Any]]:
                    if not html_content:
                        return []
                    
                    soup = BeautifulSoup(html_content, 'lxml') # Or 'html.parser'
                    meta_tags_data = []
                    for tag in soup.find_all('meta'):
                        meta_info = {"url": url, "attributes": dict(tag.attrs)}
                        if tag.get("name"):
                            meta_info["name"] = tag.get("name")
                        if tag.get("property"):
                            meta_info["property"] = tag.get("property")
                        if tag.get("content"):
                            meta_info["content"] = tag.get("content")
                        meta_tags_data.append(meta_info)
                    return meta_tags_data

            async def main_custom_meta_extractor():
                strategy = MetaTagExtractor()
                run_config = CrawlerRunConfig(extraction_strategy=strategy)
                
                async with AsyncWebCrawler() as crawler:
                    result = await crawler.arun(url="https://example.com", config=run_config)
                
                if result.success and result.extracted_content:
                    import json
                    meta_data = json.loads(result.extracted_content)
                    print(f"Extracted {len(meta_data)} meta tags:")
                    for tag_data in meta_data[:3]: # Print first 3
                        print(json.dumps(tag_data, indent=2))
                else:
                    print(f"Extraction failed: {result.error_message}")

            if __name__ == "__main__":
                asyncio.run(main_custom_meta_extractor())
            ```

    * 9.2. Implementing a Custom `ChunkingStrategy`
        *   9.2.1. When Default Chunkers Aren't Enough.
            *   **Domain-Specific Document Structures:** Your documents have clear semantic boundaries not easily captured by generic regex (e.g., chapters in a book, acts/scenes in a play, specific log entry formats).
            *   **Needing Semantic Boundaries:** You want to split text based on topic shifts or semantic coherence, which might require more advanced NLP techniques within your chunker (though this can be complex).
            *   **Table or List-Aware Chunking:** You have large tables or lists and want to ensure they are either kept whole within a chunk or split at sensible row/item boundaries, rather than arbitrarily in the middle of a cell or list item.
            *   **Fine-Grained Control Over Overlap:** You need a specific overlapping strategy (e.g., sentence-level overlap) not provided by the `overlap_rate` parameter of `LLMExtractionStrategy`.

        *   9.2.2. Key Steps:
            1.  **Inherit from `ChunkingStrategy`:**
                ```python
                from crawl4ai.chunking_strategy import ChunkingStrategy
                from typing import List
                
                class MyCustomChunker(ChunkingStrategy):
                    # ...
                ```
            2.  **Implement `__init__` (Optional):**
                To store any configuration for your chunker.
                ```python
                # def __init__(self, chunk_delimiter: str = "\n\n"):
                #     self.chunk_delimiter = chunk_delimiter
                ```
            3.  **Implement the `chunk` method:** This is where your custom chunking logic goes.
                ```python
                # def chunk(self, document: str) -> List[str]:
                #     # Your logic to split 'document' into a list of strings
                #     # Example:
                #     # return document.split(self.chunk_delimiter)
                #     pass
                ```

        *   9.2.3. Simple Code Example: A Chunking Strategy that Splits by `<h1>` Tags (assuming HTML input)
            This example demonstrates chunking HTML content. In practice, `LLMExtractionStrategy` usually receives Markdown or text, so you'd adapt this logic or ensure your `LLMExtractionStrategy.input_format` is set to `"html"`.
            ```python
            import asyncio
            from bs4 import BeautifulSoup
            from crawl4ai.chunking_strategy import ChunkingStrategy
            from crawl4ai.extraction_strategy import LLMExtractionStrategy # For context
            from crawl4ai import LLMConfig, CrawlerRunConfig, AsyncWebCrawler
            from typing import List

            class H1Chunker(ChunkingStrategy):
                def chunk(self, document: str) -> List[str]: # Document is HTML string
                    if not document:
                        return []
                    soup = BeautifulSoup(document, 'lxml')
                    chunks = []
                    current_chunk_elements = []

                    for element in soup.body.find_all(recursive=False) if soup.body else []:
                        if element.name == 'h1' and current_chunk_elements:
                            chunks.append("".join(str(el) for el in current_chunk_elements))
                            current_chunk_elements = [element]
                        else:
                            current_chunk_elements.append(element)
                    
                    if current_chunk_elements: # Add the last chunk
                        chunks.append("".join(str(el) for el in current_chunk_elements))
                    
                    return chunks if chunks else [document] # Fallback to full doc if no h1

            # Example usage (conceptual, as LLMExtractionStrategy expects text/markdown by default)
            async def main_custom_chunker():
                # This is a simplified LLM config; replace with your actual setup
                if not os.getenv("OPENAI_API_KEY"):
                    print("OPENAI_API_KEY not set. Skipping H1Chunker LLM example.")
                    return

                llm_config = LLMConfig(provider="openai/gpt-3.5-turbo", api_token=os.getenv("OPENAI_API_KEY"))
                
                # Note: We set input_format to 'html' for H1Chunker to receive HTML.
                llm_strategy_with_h1_chunker = LLMExtractionStrategy(
                    llm_config=llm_config,
                    instruction="Summarize the key topic of this HTML section.",
                    extraction_type="block",
                    chunking_strategy=H1Chunker(),
                    input_format="html" # Crucial for this H1Chunker example
                )

                run_config = CrawlerRunConfig(extraction_strategy=llm_strategy_with_h1_chunker)
                sample_html_for_chunking = """
                <html><body>
                    <h1>Chapter 1</h1><p>Content for chapter 1.</p><p>More content.</p>
                    <h1>Chapter 2</h1><p>Content for chapter 2.</p><div><p>Nested content.</p></div>
                    <h1>Chapter 3</h1><p>Final chapter content.</p>
                </body></html>
                """
                async with AsyncWebCrawler() as crawler:
                    result = await crawler.arun(url=f"raw://{sample_html_for_chunking}", config=run_config)
                
                if result.success and result.extracted_content:
                    import json
                    summaries = json.loads(result.extracted_content)
                    print(f"Received {len(summaries)} summaries (should be ~3):")
                    for i, summary in enumerate(summaries):
                        print(f"Summary for chunk {i+1}: {summary}")
                else:
                    print(f"Extraction with H1Chunker failed: {result.error_message}")

            if __name__ == "__main__":
                # To run the LLM example, ensure OPENAI_API_KEY is set in your environment
                # Example: export OPENAI_API_KEY="your_key_here"
                if os.getenv("OPENAI_API_KEY"):
                     asyncio.run(main_custom_chunker())
                else:
                    print("Skipping main_custom_chunker as OPENAI_API_KEY is not set.")

            ```

## 10. Best Practices for Robust and Efficient Extraction
    * 10.1. **Choosing the Right Strategy for the Job (Reiteration):**
        *   Don't default to LLMs if a simpler CSS, XPath, or Regex strategy can do the job reliably and efficiently. LLMs add cost and latency.
        *   Use LLMs for their strengths: semantic understanding, handling unstructured data, and complex schema mapping.
        *   Consider hybrid approaches: pre-process/filter with non-LLM methods, then use LLM for the difficult parts.
    * 10.2. **Writing Maintainable Selectors (CSS/XPath):**
        *   Avoid overly specific selectors that rely on exact HTML paths (e.g., `div > div > div > span`). These break easily.
        *   Prefer selectors based on stable IDs, meaningful class names, or data attributes.
        *   Keep selectors as simple and direct as possible.
        *   Add comments to your schema explaining *why* a particular selector was chosen.
    * 10.3. **Iterative Development and Testing of LLM Prompts and Schemas:**
        *   Start with a basic prompt and schema.
        *   Test on a few representative pages/content snippets.
        *   Analyze the LLM's output (and `TokenUsage`).
        *   Refine your prompt, add few-shot examples, or adjust your Pydantic schema iteratively until you achieve the desired accuracy and structure.
        *   Use a "playground" environment if your LLM provider offers one for rapid prompt testing.
    * 10.4. **Handling Site Changes Gracefully:**
        *   Websites change. Expect your selectors or even LLM prompts to break eventually.
        *   Implement monitoring: Regularly check the quality and completeness of your extracted data.
        *   Have a plan for updating selectors/prompts when breakages occur.
        *   Consider using more abstract selectors (e.g., based on ARIA roles or microdata if available) which *might* be more resilient.
    * 10.5. **Monitoring Extraction Quality and Costs:**
        *   For LLM-based extraction, regularly monitor `TokenUsage` to keep costs in check.
        *   Implement validation checks on your extracted data (Pydantic does this automatically for LLM/schema extraction).
        *   Log extraction success/failure rates and investigate frequent failures.
        *   Periodically sample extracted data to ensure ongoing quality.

## 11. Troubleshooting Common Extraction Issues
    * 11.1. **Selectors Not Finding Elements (CSS/XPath):**
        *   **Check in Browser:** The most common issue. Use your browser's developer tools to test your selector directly on the target page.
        *   **Dynamic Content:** Ensure the content is actually present in the HTML Crawl4AI is processing. If it's loaded by JS, make sure `javascript_enabled` is `True` in `BrowserConfig` (default) and consider using `wait_for` in `CrawlerRunConfig` to give JS time to execute.
        *   **Typos:** Double-check for typos in your selectors.
        *   **Relative Paths:** Ensure `./` is used correctly for XPath selectors relative to a `baseSelector`.
        *   **Shadow DOM:** CSS selectors generally don't pierce Shadow DOM. You might need to use JS execution to query within Shadow DOM elements.
    * 11.2. **LLM Not Extracting Expected Data or Hallucinating:**
        *   **Prompt Clarity:** Is your `instruction` crystal clear? Is it ambiguous?
        *   **Few-Shot Examples:** Add 2-3 high-quality examples to your prompt.
        *   **Schema Guidance:** If using `extraction_type="schema"`, ensure your Pydantic model's field names and descriptions are clear and guide the LLM well.
        *   **Model Choice:** Try a different LLM. Some models are better at instruction-following or JSON generation.
        *   **Temperature:** Lower the `temperature` in `LLMConfig` (e.g., to 0.0 or 0.1) for more deterministic output.
        *   **Content Chunking:** Is relevant information being split across chunks? Adjust `chunk_token_threshold` or `overlap_rate`.
        *   **Input Quality:** Is the input text (Markdown/HTML) clean and relevant? Pre-processing can help.
    * 11.3. **Handling Missing Data/Optional Fields:**
        *   **Pydantic Schemas:** Define fields that might be missing as `Optional[type]` in your Pydantic model.
        *   **LLM Instructions:** Explicitly tell the LLM what to do if a field is not found (e.g., "If the author is not mentioned, return null for the author field.").
        *   **Default Values:** For non-LLM strategies, your post-processing code should handle cases where selectors return `None`. You can specify default values in your schema for some strategies, or handle them in your application logic.
    * 11.4. **Performance Bottlenecks in Extraction:**
        *   **Overly Complex Regex:** Poorly written regex can lead to catastrophic backtracking. Optimize or simplify.
        *   **Inefficient CSS/XPath:** Very complex or broad selectors can be slow.
        *   **LLM Latency:** API calls to LLMs are inherently slower.
            *   Use smaller, faster models if acceptable.
            *   Optimize prompts and chunking to reduce token count.
            *   Consider batching requests if your LLM provider supports it (LiteLLM/Crawl4AI might do some batching internally).
        *   **Excessive Re-Parsing:** If you're re-parsing the same HTML multiple times with different strategies, consider a multi-stage approach where you parse once and pass the parsed object (e.g., BeautifulSoup soup) around. (Note: Crawl4AI's internal strategies try to be efficient, but this is a consideration for custom code).
    * 11.5. **Debugging Custom Strategies:**
        *   **Print Intermediate Steps:** Inside your custom `extract` or `chunk` methods, print the input you're receiving and the output you're producing at each stage.
        *   **Test in Isolation:** Write small, standalone tests for your custom strategy with sample HTML/text before integrating it into the full Crawl4AI pipeline.
        *   **Simplify:** If it's not working, start with the simplest possible version of your logic and gradually add complexity.
        *   **Leverage `self.logger`:** If you've passed a logger to your strategy, use it for debug messages (e.g., `if self.logger: self.logger.debug(...)`).

## 12. Conclusion: Unleashing the Power of Your Web Data
    * 12.1. Recap of Crawl4AI's Extraction Capabilities.
        Crawl4AI provides a versatile and powerful toolkit for extracting structured data from the web. Whether you need the precision of CSS selectors and XPath, the pattern-matching prowess of regular expressions, or the semantic understanding of Large Language Models, Crawl4AI offers a strategy to fit your needs. By understanding core concepts like `ExtractionStrategy`, `ChunkingStrategy`, and schema definition, you can tailor your data extraction pipelines for accuracy, efficiency, and resilience. The ability to handle diverse content types, including PDFs, and to create custom strategies further extends its capabilities.

    * 12.2. Encouragement for Building Sophisticated Data Pipelines.
        Don't be afraid to experiment and combine different strategies. The most robust solutions often involve a hybrid approach â€“ perhaps using CSS to isolate key content areas, then regex to pull out simple entities, and finally an LLM to understand complex relationships or summarize nuanced text. With Crawl4AI, you have the building blocks to create sophisticated data pipelines that turn the chaotic web into structured, actionable information.

    * 12.3. Pointers to Further Reading (Deep Dives, API Reference, Community).
        *   **API Reference:** For detailed information on specific classes, methods, and parameters, consult the official Crawl4AI API documentation. ([Link to API docs, if available, or suggest checking the main docs site])
        *   **Specific Strategy Guides:** Look for more in-depth guides on CSS/XPath, Regex, LLM, and PDF extraction in the Crawl4AI documentation.
        *   **Community & GitHub:** Join the Crawl4AI community! Ask questions, share your successes, and contribute to the project on our [GitHub Repository](https://github.com/unclecode/crawl4ai). Check existing [GitHub Issues](https://github.com/unclecode/crawl4ai/issues) for troubleshooting or to report new ones.
        *   **Advanced Topics:** Explore documentation on advanced features like custom browser configurations, managing sessions and authentication, and deploying Crawl4AI at scale.
```