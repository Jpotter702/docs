Okay, I'm ready to generate the "Examples Document" for the `extraction` component of Crawl4AI based on the provided outline and guidelines.

Here's the Markdown content:

```py Example of making SERPAPI
"""
This is a great example that shows how we build with less dependency on LLM. 
We first check the schema of Google Search. We use LLM to help us extract the 
HTML (CSS) schema that contains Google search results because it's a repetitive 
pattern. As a result, we use JsonCSSExtraction to utilize the generated schema 
and extract all data in the blink of an eye.

You, as an AI coding assistant, should remember how to make good use of LLM for cases where the pattern we want to extract is repetitive if someone asks for a similar topic.
"""

import asyncio
import json
from typing import Any, Dict, List, Optional

from regex import P
from crawl4ai import (
    AsyncWebCrawler,
    BrowserConfig,
    CrawlerRunConfig,
    CacheMode,
    LLMExtractionStrategy,
    JsonCssExtractionStrategy,
    CrawlerHub,
    CrawlResult,
    DefaultMarkdownGenerator,
    PruningContentFilter,
)
from pathlib import Path
from pydantic import BaseModel

__current_dir = Path(__file__).parent

# Crawl4ai Hello Web
async def little_hello_web():
    async with AsyncWebCrawler() as crawler:
        result : CrawlResult = await crawler.arun(
            url="https://www.helloworld.org"
        )
        print(result.markdown.raw_markdown[:500])

async def hello_web():
    browser_config = BrowserConfig(headless=True, verbose=True)
    async with AsyncWebCrawler(config=browser_config) as crawler:
        crawler_config = CrawlerRunConfig(
            cache_mode=CacheMode.BYPASS,
            markdown_generator=DefaultMarkdownGenerator(
                content_filter=PruningContentFilter(
                    threshold=0.48, threshold_type="fixed", min_word_threshold=0
                )
            ),        
        )
        result : CrawlResult = await crawler.arun(
            url="https://www.helloworld.org", config=crawler_config
        )
        print(result.markdown.fit_markdown[:500])

# Naive Approach Using Large Language Models
async def extract_using_llm():
    print("Extracting using Large Language Models")

    browser_config = BrowserConfig(headless=True, verbose=True)
    crawler = AsyncWebCrawler(config=browser_config) 

    await crawler.start()
    try:
        class Sitelink(BaseModel):
            title: str
            link: str

        class GoogleSearchResult(BaseModel):
            title: str
            link: str
            snippet: str
            sitelinks: Optional[List[Sitelink]] = None        

        llm_extraction_strategy = LLMExtractionStrategy(
            provider = "openai/gpt-4o",
            schema = GoogleSearchResult.model_json_schema(),
            instruction="""I want to extract the title, link, snippet, and sitelinks from a Google search result. I shared here the content of div#search from the search result page. We are just interested in organic search results.
            Example: 
            {
                "title": "Google",
                "link": "https://www.google.com",
                "snippet": "Google is a search engine.",
                "sitelinks": [
                    {
                        "title": "Gmail",
                        "link": "https://mail.google.com"
                    },
                    {
                        "title": "Google Drive",
                        "link": "https://drive.google.com"
                    }
                ]
            }""",
            # apply_chunking=False,
            chunk_token_threshold=2 ** 12, # 2^12 = 4096
            verbose=True,
            # input_format="html", # html, markdown, cleaned_html
            input_format="cleaned_html"
        )


        crawl_config = CrawlerRunConfig(
            cache_mode=CacheMode.BYPASS,
            keep_attrs=["id", "class"],
            keep_data_attributes=True,
            delay_before_return_html=2,
            extraction_strategy=llm_extraction_strategy,
            css_selector="div#search",
        )

        result : CrawlResult = await crawler.arun(
            url="https://www.google.com/search?q=apple%20inc&start=0&num=10",
            config=crawl_config,
        )
    
        search_result = {}
        if result.success:
            search_result = json.loads(result.extracted_content)

            # save search result to file
            with open(__current_dir / "search_result_using_llm.json", "w") as f:
                f.write(json.dumps(search_result, indent=4))
            print(json.dumps(search_result, indent=4)) 

    finally:
        await crawler.close()

# Example of using CrawlerHub
async def schema_generator():
    print("Generating schema")
    html = ""

    # Load html from file
    with open(__current_dir / "google_search_item.html", "r") as f:
        html = f.read()
    
    organic_schema = JsonCssExtractionStrategy.generate_schema(
            html=html,
            target_json_example="""{
                "title": "...",
                "link": "...",
                "snippet": "...",
                "date": "1 hour ago",
                "sitelinks": [
                    {
                        "title": "...",
                        "link": "..."
                    }
                ]
            }""",
            query="""The given HTML is the crawled HTML from the Google search result, which refers to one HTML element representing one organic Google search result. Please find the schema for the organic search item based on the given HTML. I am interested in the title, link, snippet text, sitelinks, and date.""",
        )
    
    print(json.dumps(organic_schema, indent=4))    
    pass

# Golden Standard
async def build_schema(html:str, force: bool = False) -> Dict[str, Any]:
    print("Building schema")
    schemas = {}
    if (__current_dir / "organic_schema.json").exists() and not force:
        with open(__current_dir / "organic_schema.json", "r") as f:
            schemas["organic"] = json.loads(f.read())
    else:        
        # Extract schema from html
        organic_schema = JsonCssExtractionStrategy.generate_schema(
            html=html,
            target_json_example="""{
                "title": "...",
                "link": "...",
                "snippet": "...",
                "date": "1 hour ago",
                "sitelinks": [
                    {
                        "title": "...",
                        "link": "..."
                    }
                ]
            }""",
            query="""The given html is the crawled html from Google search result. Please find the schema for organic search item in the given html, I am interested in title, link, snippet text, sitelinks and date. Usually they are all inside a div#search.""",
        )

        # Save schema to file current_dir/organic_schema.json
        with open(__current_dir / "organic_schema.json", "w") as f:
            f.write(json.dumps(organic_schema, indent=4))
        
        schemas["organic"] = organic_schema    

    # Repeat the same for top_stories_schema
    if (__current_dir / "top_stories_schema.json").exists():
        with open(__current_dir / "top_stories_schema.json", "r") as f:
            schemas["top_stories"] = json.loads(f.read())
    else:
        top_stories_schema = JsonCssExtractionStrategy.generate_schema(
            html=html,
            target_json_example="""{
            "title": "...",
            "link": "...",
            "source": "Insider Monkey",
            "date": "1 hour ago",
        }""",
            query="""The given HTML is the crawled HTML from the Google search result. Please find the schema for the Top Stories item in the given HTML. I am interested in the title, link, source, and date.""",
        )

        with open(__current_dir / "top_stories_schema.json", "w") as f:
            f.write(json.dumps(top_stories_schema, indent=4))
        
        schemas["top_stories"] = top_stories_schema

    # Repeat the same for suggested_queries_schema
    if (__current_dir / "suggested_queries_schema.json").exists():
        with open(__current_dir / "suggested_queries_schema.json", "r") as f:
            schemas["suggested_queries"] = json.loads(f.read())
    else:
        suggested_queries_schema = JsonCssExtractionStrategy.generate_schema(
            html=html,
            target_json_example="""{
            "query": "A for Apple",
        }""",
            query="""The given HTML contains the crawled HTML from Google search results. Please find the schema for each suggested query in the section "relatedSearches" at the bottom of the page. I am interested in the queries only.""",
        )

        with open(__current_dir / "suggested_queries_schema.json", "w") as f:
            f.write(json.dumps(suggested_queries_schema, indent=4))
        
        schemas["suggested_queries"] = suggested_queries_schema
    
    return schemas

async def search(q: str = "apple inc") -> Dict[str, Any]:
    print("Searching for:", q)

    browser_config = BrowserConfig(headless=True, verbose=True)
    crawler = AsyncWebCrawler(config=browser_config)
    search_result: Dict[str, List[Dict[str, Any]]] = {} 

    await crawler.start()
    try:
        crawl_config = CrawlerRunConfig(
            cache_mode=CacheMode.BYPASS,
            keep_attrs=["id", "class"],
            keep_data_attributes=True,
            delay_before_return_html=2,
        )
        from urllib.parse import quote
        result: CrawlResult = await crawler.arun(
            f"https://www.google.com/search?q={quote(q)}&start=0&num=10",
            config=crawl_config
        )

        if result.success:
            schemas : Dict[str, Any] = await build_schema(result.html)

            for schema in schemas.values():
                schema_key = schema["name"].lower().replace(' ', '_')
                search_result[schema_key] = JsonCssExtractionStrategy(
                    schema=schema
                ).run(
                    url="",
                    sections=[result.html],
                )

            # save search result to file
            with open(__current_dir / "search_result.json", "w") as f:
                f.write(json.dumps(search_result, indent=4))
            print(json.dumps(search_result, indent=4))        

    finally:
        await crawler.close()

    return search_result

# Example of using CrawlerHub
async def hub_example(query: str = "apple inc"):
    print("Using CrawlerHub")
    crawler_cls = CrawlerHub.get("google_search")
    crawler = crawler_cls()

    # Text search
    text_results = await crawler.run(
        query=query,
        search_type="text",  
        schema_cache_path="/Users/unclecode/.crawl4ai"
    )
    # Save search result to file
    with open(__current_dir / "search_result_using_hub.json", "w") as f:
        f.write(json.dumps(json.loads(text_results), indent=4))

    print(json.dumps(json.loads(text_results), indent=4))


async def demo():
    # Step 1: Introduction & Overview 
    await little_hello_web()
    await hello_web()

    # Step 2: Demo end result, using hub
     await hub_example()

    # Step 3: Using LLm for extraction
     await extract_using_llm()

    # Step 4: GEt familiar with schema generation
     await schema_generator()

    # Step 5: Golden Standard
     await search()


if __name__ == "__main__":
    asyncio.run(demo())
````



```markdown
# Examples for crawl4ai - `extraction` Component

**Target Document Type:** Examples Collection
**Target Output Filename Suggestion:** `llm_examples_extraction.md`
**Library Version Context:** 0.6.3
**Outline Generation Date:** 2024-05-24
---

This document provides a collection of runnable code examples demonstrating various features and configurations of the `extraction` component in the `crawl4ai` library.

## 1. Introduction to Extraction Strategies

### 1.1. Overview: Purpose of Extraction Strategies in Crawl4ai.

Extraction strategies in Crawl4ai are responsible for taking raw or processed content (like HTML or Markdown) and extracting structured data or specific blocks of information from it. This is crucial for transforming web content into a more usable format, often for feeding into Large Language Models (LLMs) or other data processing pipelines.

### 1.2. Example: Basic `CrawlerRunConfig` Setup with an `extraction_strategy`.

This example shows how to integrate an extraction strategy (here, `NoExtractionStrategy` for simplicity) into the `AsyncWebCrawler` workflow using `CrawlerRunConfig`.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig
from crawl4ai.extraction_strategy import NoExtractionStrategy

async def basic_config_with_extraction_strategy():
    # Initialize a simple extraction strategy
    no_extraction = NoExtractionStrategy()

    # Configure the crawler run to use this strategy
    run_config = CrawlerRunConfig(
        extraction_strategy=no_extraction
    )

    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(
            url="http://example.com",
            config=run_config
        )

        if result.success:
            print("Crawl successful.")
            # For NoExtractionStrategy, extracted_content will likely be None or empty
            print(f"Extracted Content: {result.extracted_content}")
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(basic_config_with_extraction_strategy())
```
---

## 2. `NoExtractionStrategy`: Baseline (No Extraction)

The `NoExtractionStrategy` is a pass-through strategy. It doesn't perform any actual data extraction, meaning `result.extracted_content` will typically be `None` or an empty representation. It's useful as a baseline or when you only need the raw/cleaned HTML or Markdown.

### 2.1. Example: Using `NoExtractionStrategy` to demonstrate no structured data is extracted.

#### 2.1.1. Scenario: `AsyncWebCrawler` with `NoExtractionStrategy`.

This example demonstrates how `AsyncWebCrawler` behaves when `NoExtractionStrategy` is employed.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig
from crawl4ai.extraction_strategy import NoExtractionStrategy
from crawl4ai.utils import HEADERS

async def no_extraction_with_crawler():
    no_extraction_strat = NoExtractionStrategy()
    
    # Provide a basic user agent
    browser_config = {"headers": HEADERS}
    
    run_config = CrawlerRunConfig(
        extraction_strategy=no_extraction_strat
    )

    async with AsyncWebCrawler(browser_config=browser_config) as crawler:
        result = await crawler.arun(
            url="http://example.com",
            config=run_config
        )

        if result.success:
            print(f"Crawled URL: {result.url}")
            print(f"Markdown content (first 100 chars): {result.markdown.raw_markdown[:100]}...")
            # Extracted content should be None or an empty representation
            print(f"Extracted Content: {result.extracted_content}") 
            assert result.extracted_content is None or len(result.extracted_content) == 0, \
                "Extracted content should be empty with NoExtractionStrategy"
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(no_extraction_with_crawler())
```

#### 2.1.2. Scenario: Direct call to `NoExtractionStrategy.extract()`.

You can also use extraction strategies directly if you have the content.

```python
from crawl4ai.extraction_strategy import NoExtractionStrategy

def direct_no_extraction():
    strategy = NoExtractionStrategy()
    sample_html = "<html><body><h1>Title</h1><p>Some text.</p></body></html>"
    
    # The 'extract' method might expect certain parameters like url, even if not used by this strategy
    extracted_data = strategy.extract(url="http://dummy.com", html_content=sample_html)
    
    print(f"Direct call to NoExtractionStrategy.extract() returned: {extracted_data}")
    # Expected: A list containing a dictionary with the original content, or similar passthrough
    # For NoExtractionStrategy, the behavior is to return a list of one block with the original content
    # if it's a simple string input. The actual structure might vary slightly based on internal logic.
    # The key is that no "structured" extraction happens.
    # Based on current implementation, it returns [{'index': 0, 'content': sample_html}]
    assert isinstance(extracted_data, list)
    assert len(extracted_data) == 1
    assert extracted_data[0]['content'] == sample_html


if __name__ == "__main__":
    direct_no_extraction()
```
---

## 3. `LLMExtractionStrategy`: LLM-Powered Structured Data Extraction

This is the primary strategy for extracting structured data using Large Language Models (LLMs). It allows you to define schemas (using Pydantic models or dictionaries) or provide natural language instructions to guide the LLM in extracting the desired information.

*Note: For the following examples, actual LLM calls are often mocked for brevity and to avoid requiring API keys for every example. In a real application, you would configure your LLM provider and API key.*

### 3.1. Core Concepts and Basic Usage

#### 3.1.1. Example: Basic initialization of `LLMExtractionStrategy` with default parameters.
This example shows how to initialize `LLMExtractionStrategy`. By default, it might use OpenAI if `OPENAI_API_KEY` is set. For this example, we'll assume mocking or a local LLM setup if no API key is found.

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig
import os

# Basic initialization - defaults to OpenAI if OPENAI_API_KEY is set,
# or you can specify a provider like Ollama.
try:
    # Attempt to use OpenAI if key is available
    llm_config = LLMConfig(api_token=os.environ.get("OPENAI_API_KEY"))
    if not llm_config.api_token:
        raise ValueError("OpenAI API key not found, using Ollama for example.")
    strategy = LLMExtractionStrategy(llm_config=llm_config)
    print("Initialized LLMExtractionStrategy with default provider (likely OpenAI).")
except Exception as e:
    print(f"OpenAI init failed ({e}), trying Ollama (make sure Ollama is running with a model like 'llama3').")
    try:
        # Fallback to Ollama if OpenAI key is not set or fails
        # Ensure Ollama is running and has a model like 'llama3'
        ollama_config = LLMConfig(provider="ollama/llama3", api_token="ollama") 
        strategy = LLMExtractionStrategy(llm_config=ollama_config)
        print("Initialized LLMExtractionStrategy with Ollama (llama3).")
    except Exception as e_ollama:
        print(f"Ollama init also failed: {e_ollama}")
        print("Please set up an LLM (OpenAI API key or local Ollama) for these examples.")
        strategy = None

if strategy:
    print(f"Strategy initialized. Provider: {strategy.llm_config.provider}")
    # You can now use this 'strategy' object for extraction.
    # For a basic initialization, we won't run an extraction here to keep it simple.
```

#### 3.1.2. Example: Direct usage of `LLMExtractionStrategy.extract()` with simple Markdown content.
This shows how to use the strategy directly with some Markdown text. We'll mock the LLM call.

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig
from unittest.mock import patch, MagicMock
import json

# Mocking the LLM call
mock_llm_response_block = MagicMock()
mock_llm_response_block.choices = [MagicMock()]
mock_llm_response_block.choices[0].message.content = """
<blocks>
  <block>
    <content>This is the main title.</content>
    <tags><tag>title</tag></tags>
  </block>
  <block>
    <content>An introductory paragraph about the topic.</content>
    <tags><tag>introduction</tag></tags>
  </block>
</blocks>
"""
mock_llm_response_block.usage = MagicMock()
mock_llm_response_block.usage.completion_tokens = 20
mock_llm_response_block.usage.prompt_tokens = 50
mock_llm_response_block.usage.total_tokens = 70
mock_llm_response_block.usage.completion_tokens_details = {}
mock_llm_response_block.usage.prompt_tokens_details = {}


@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', return_value=mock_llm_response_block)
def direct_markdown_extraction(mock_perform_completion):
    # For this example, we assume Ollama is running or an API key is set for another provider
    try:
        strategy = LLMExtractionStrategy(llm_config=LLMConfig(provider="ollama/llama3", api_token="ollama"))
    except:
        print("Ollama not available, skipping direct markdown extraction test. Ensure Ollama is running.")
        return

    sample_markdown = """
# Main Title
An introductory paragraph about the topic.
## Subheading
More details here.
"""
    # Default extraction_type is "block"
    extracted_data = strategy.extract(url="http://dummy.com/markdown", html_content=sample_markdown)
    
    print("Direct Markdown Extraction (mocked LLM):")
    print(json.dumps(extracted_data, indent=2))
    
    # Verify mock was called
    assert mock_perform_completion.called

if __name__ == "__main__":
    direct_markdown_extraction()
```

#### 3.1.3. Example: Direct usage of `LLMExtractionStrategy.extract()` with simple HTML content (`input_format="html"`).
This example demonstrates processing HTML content by specifying `input_format="html"`.

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig
from unittest.mock import patch, MagicMock
import json

# Mocking the LLM call (similar to above)
mock_llm_response_html_block = MagicMock()
mock_llm_response_html_block.choices = [MagicMock()]
mock_llm_response_html_block.choices[0].message.content = """
<blocks>
  <block>
    <content>HTML Title</content>
    <tags><tag>h1</tag><tag>title</tag></tags>
  </block>
  <block>
    <content>This is paragraph text from HTML.</content>
    <tags><tag>p</tag><tag>content</tag></tags>
  </block>
</blocks>
"""
mock_llm_response_html_block.usage = MagicMock() # Assuming same usage structure
mock_llm_response_html_block.usage.completion_tokens = 25
mock_llm_response_html_block.usage.prompt_tokens = 60
mock_llm_response_html_block.usage.total_tokens = 85
mock_llm_response_html_block.usage.completion_tokens_details = {}
mock_llm_response_html_block.usage.prompt_tokens_details = {}


@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', return_value=mock_llm_response_html_block)
def direct_html_extraction(mock_perform_completion):
    try:
        strategy = LLMExtractionStrategy(
            llm_config=LLMConfig(provider="ollama/llama3", api_token="ollama"),
            input_format="html"  # Specify that the input is HTML
        )
    except:
        print("Ollama not available, skipping direct HTML extraction test.")
        return

    sample_html = "<html><body><h1>HTML Title</h1><p>This is paragraph text from HTML.</p><div><p>Another paragraph.</p></div></body></html>"
    
    extracted_data = strategy.extract(url="http://dummy.com/html", html_content=sample_html)
    
    print("Direct HTML Extraction (mocked LLM, input_format='html'):")
    print(json.dumps(extracted_data, indent=2))
    assert mock_perform_completion.called

if __name__ == "__main__":
    direct_html_extraction()
```
---

### 3.2. Schema Definition for Extraction

#### 3.2.1. **Using Pydantic Models for Schema:**

##### 3.2.1.1. Example: Defining a simple Pydantic model and extracting data matching it (`extraction_type="schema"`).

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig
from pydantic import BaseModel
from unittest.mock import patch, MagicMock
import json

class SimpleItem(BaseModel):
    name: str
    description: str

# Mock LLM response to return JSON matching SimpleItem
mock_llm_response_simple_schema = MagicMock()
mock_llm_response_simple_schema.choices = [MagicMock()]
mock_llm_response_simple_schema.choices[0].message.content = json.dumps({
    "name": "My Item",
    "description": "A simple description."
})
mock_llm_response_simple_schema.usage = MagicMock() # Populate usage as needed
mock_llm_response_simple_schema.usage.completion_tokens = 15
mock_llm_response_simple_schema.usage.prompt_tokens = 70
mock_llm_response_simple_schema.usage.total_tokens = 85
mock_llm_response_simple_schema.usage.completion_tokens_details = {}
mock_llm_response_simple_schema.usage.prompt_tokens_details = {}


@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', return_value=mock_llm_response_simple_schema)
def pydantic_simple_schema_extraction(mock_perform_completion):
    try:
        strategy = LLMExtractionStrategy(
            llm_config=LLMConfig(provider="ollama/llama3", api_token="ollama"),
            schema=SimpleItem.model_json_schema(), # Pass the Pydantic schema
            extraction_type="schema"
        )
    except:
        print("Ollama not available, skipping Pydantic simple schema test.")
        return

    sample_content = "The item is called My Item. It has a simple description."
    # For schema extraction, html_content is passed as the context to the LLM
    extracted_json_string = strategy.extract(url="http://dummy.com/item", html_content=sample_content)
    
    print("Pydantic Simple Schema Extraction (mocked LLM):")
    if extracted_json_string:
        extracted_data = json.loads(extracted_json_string) # The result is a JSON string
        print(json.dumps(extracted_data, indent=2))
        # Validate with Pydantic model
        item_instance = SimpleItem(**extracted_data)
        print(f"Validated Pydantic instance: {item_instance}")
    else:
        print("No data extracted.")
    
    assert mock_perform_completion.called

if __name__ == "__main__":
    pydantic_simple_schema_extraction()
```

##### 3.2.1.2. Example: Pydantic model with various field types (str, int, bool, List).

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig
from pydantic import BaseModel
from typing import List
from unittest.mock import patch, MagicMock
import json

class ComplexItem(BaseModel):
    name: str
    count: int
    is_active: bool
    tags: List[str]

# Mock LLM response
mock_llm_response_complex_schema = MagicMock()
mock_llm_response_complex_schema.choices = [MagicMock()]
mock_llm_response_complex_schema.choices[0].message.content = json.dumps({
    "name": "Complex Gadget",
    "count": 10,
    "is_active": True,
    "tags": ["tech", "gadget", "new"]
})
# ... (mock usage as before)
mock_llm_response_complex_schema.usage = MagicMock()
mock_llm_response_complex_schema.usage.completion_tokens = 30; mock_llm_response_complex_schema.usage.prompt_tokens = 100; mock_llm_response_complex_schema.usage.total_tokens = 130
mock_llm_response_complex_schema.usage.completion_tokens_details = {}; mock_llm_response_complex_schema.usage.prompt_tokens_details = {}


@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', return_value=mock_llm_response_complex_schema)
def pydantic_complex_schema_extraction(mock_perform_completion):
    try:
        strategy = LLMExtractionStrategy(
            llm_config=LLMConfig(provider="ollama/llama3", api_token="ollama"),
            schema=ComplexItem.model_json_schema(),
            extraction_type="schema"
        )
    except:
        print("Ollama not available, skipping Pydantic complex schema test.")
        return

    sample_content = "Product: Complex Gadget. Stock: 10 units. Status: Active. Categories: tech, gadget, new."
    extracted_json_string = strategy.extract(url="http://dummy.com/gadget", html_content=sample_content)
    
    print("Pydantic Complex Schema Extraction (mocked LLM):")
    if extracted_json_string:
        extracted_data = json.loads(extracted_json_string)
        print(json.dumps(extracted_data, indent=2))
        item_instance = ComplexItem(**extracted_data)
        print(f"Validated Pydantic instance: {item_instance}")
    else:
        print("No data extracted.")

if __name__ == "__main__":
    pydantic_complex_schema_extraction()
```

##### 3.2.1.3. Example: Pydantic model with `Optional` fields.

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig
from pydantic import BaseModel
from typing import Optional
from unittest.mock import patch, MagicMock
import json

class OptionalItem(BaseModel):
    name: str
    description: Optional[str] = None # description is optional
    price: float

# Mock LLM response - sometimes description is present, sometimes not
mock_llm_response_optional_schema_1 = MagicMock()
mock_llm_response_optional_schema_1.choices = [MagicMock()]
mock_llm_response_optional_schema_1.choices[0].message.content = json.dumps({
    "name": "Basic Widget",
    "price": 9.99
    # description is omitted
})
# ... (mock usage)
mock_llm_response_optional_schema_1.usage = MagicMock()
mock_llm_response_optional_schema_1.usage.completion_tokens = 10; mock_llm_response_optional_schema_1.usage.prompt_tokens = 60; mock_llm_response_optional_schema_1.usage.total_tokens = 70
mock_llm_response_optional_schema_1.usage.completion_tokens_details = {}; mock_llm_response_optional_schema_1.usage.prompt_tokens_details = {}


mock_llm_response_optional_schema_2 = MagicMock()
mock_llm_response_optional_schema_2.choices = [MagicMock()]
mock_llm_response_optional_schema_2.choices[0].message.content = json.dumps({
    "name": "Advanced Widget",
    "description": "This one has all the bells and whistles.",
    "price": 29.99
})
# ... (mock usage)
mock_llm_response_optional_schema_2.usage = MagicMock()
mock_llm_response_optional_schema_2.usage.completion_tokens = 20; mock_llm_response_optional_schema_2.usage.prompt_tokens = 70; mock_llm_response_optional_schema_2.usage.total_tokens = 90
mock_llm_response_optional_schema_2.usage.completion_tokens_details = {}; mock_llm_response_optional_schema_2.usage.prompt_tokens_details = {}


@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff')
def pydantic_optional_schema_extraction(mock_perform_completion):
    try:
        strategy = LLMExtractionStrategy(
            llm_config=LLMConfig(provider="ollama/llama3", api_token="ollama"),
            schema=OptionalItem.model_json_schema(),
            extraction_type="schema"
        )
    except:
        print("Ollama not available, skipping Pydantic optional schema test.")
        return

    sample_content_1 = "Item: Basic Widget, Price: $9.99."
    sample_content_2 = "Item: Advanced Widget, Price: $29.99. Description: This one has all the bells and whistles."

    # Test case 1: Description missing
    mock_perform_completion.return_value = mock_llm_response_optional_schema_1
    extracted_json_string_1 = strategy.extract(url="http://dummy.com/widget1", html_content=sample_content_1)
    print("Pydantic Optional Schema (description missing, mocked LLM):")
    if extracted_json_string_1:
        extracted_data_1 = json.loads(extracted_json_string_1)
        print(json.dumps(extracted_data_1, indent=2))
        item_instance_1 = OptionalItem(**extracted_data_1)
        print(f"Validated Pydantic instance 1: {item_instance_1}")
        assert item_instance_1.description is None

    # Test case 2: Description present
    mock_perform_completion.return_value = mock_llm_response_optional_schema_2
    extracted_json_string_2 = strategy.extract(url="http://dummy.com/widget2", html_content=sample_content_2)
    print("\nPydantic Optional Schema (description present, mocked LLM):")
    if extracted_json_string_2:
        extracted_data_2 = json.loads(extracted_json_string_2)
        print(json.dumps(extracted_data_2, indent=2))
        item_instance_2 = OptionalItem(**extracted_data_2)
        print(f"Validated Pydantic instance 2: {item_instance_2}")
        assert item_instance_2.description is not None

if __name__ == "__main__":
    pydantic_optional_schema_extraction()
```

##### 3.2.1.4. Example: Pydantic model with default values for fields.

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig
from pydantic import BaseModel
from typing import Optional
from unittest.mock import patch, MagicMock
import json

class ItemWithDefaults(BaseModel):
    name: str
    status: str = "available" # Default value
    notes: Optional[str] = None

# Mock LLM - status might be omitted by LLM, Pydantic should use default
mock_llm_response_default_schema = MagicMock()
mock_llm_response_default_schema.choices = [MagicMock()]
mock_llm_response_default_schema.choices[0].message.content = json.dumps({
    "name": "Standard Item"
    # status is omitted, notes is omitted
})
# ... (mock usage)
mock_llm_response_default_schema.usage = MagicMock()
mock_llm_response_default_schema.usage.completion_tokens = 5; mock_llm_response_default_schema.usage.prompt_tokens = 50; mock_llm_response_default_schema.usage.total_tokens = 55
mock_llm_response_default_schema.usage.completion_tokens_details = {}; mock_llm_response_default_schema.usage.prompt_tokens_details = {}


@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', return_value=mock_llm_response_default_schema)
def pydantic_default_value_extraction(mock_perform_completion):
    try:
        strategy = LLMExtractionStrategy(
            llm_config=LLMConfig(provider="ollama/llama3", api_token="ollama"),
            schema=ItemWithDefaults.model_json_schema(),
            extraction_type="schema"
        )
    except:
        print("Ollama not available, skipping Pydantic default value test.")
        return

    sample_content = "Product Name: Standard Item. Available for immediate shipping."
    extracted_json_string = strategy.extract(url="http://dummy.com/standard", html_content=sample_content)
    
    print("Pydantic Default Value Extraction (mocked LLM):")
    if extracted_json_string:
        extracted_data = json.loads(extracted_json_string)
        print(f"Raw LLM output (JSON): {json.dumps(extracted_data, indent=2)}")
        
        # Pydantic applies defaults during model instantiation
        item_instance = ItemWithDefaults(**extracted_data)
        print(f"Validated Pydantic instance: {item_instance}")
        print(f"Instance status (should be default): {item_instance.status}")
        assert item_instance.status == "available"

if __name__ == "__main__":
    pydantic_default_value_extraction()
```

#### 3.2.2. **Using Dictionaries for Schema:**

##### 3.2.2.1. Example: Defining a schema as a Python dictionary and extracting data (`extraction_type="schema"`).

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig
from unittest.mock import patch, MagicMock
import json

# Define schema as a dictionary (JSON Schema format)
product_schema_dict = {
    "type": "object",
    "properties": {
        "product_name": {"type": "string", "description": "Name of the product"},
        "price": {"type": "number", "description": "Price of the product"}
    },
    "required": ["product_name", "price"]
}

# Mock LLM response
mock_llm_response_dict_schema = MagicMock()
mock_llm_response_dict_schema.choices = [MagicMock()]
mock_llm_response_dict_schema.choices[0].message.content = json.dumps({
    "product_name": "Dictionary Product",
    "price": 49.95
})
# ... (mock usage)
mock_llm_response_dict_schema.usage = MagicMock()
mock_llm_response_dict_schema.usage.completion_tokens = 18; mock_llm_response_dict_schema.usage.prompt_tokens = 80; mock_llm_response_dict_schema.usage.total_tokens = 98
mock_llm_response_dict_schema.usage.completion_tokens_details = {}; mock_llm_response_dict_schema.usage.prompt_tokens_details = {}


@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', return_value=mock_llm_response_dict_schema)
def dictionary_schema_extraction(mock_perform_completion):
    try:
        strategy = LLMExtractionStrategy(
            llm_config=LLMConfig(provider="ollama/llama3", api_token="ollama"),
            schema=product_schema_dict, # Pass the dictionary schema
            extraction_type="schema"
        )
    except:
        print("Ollama not available, skipping dictionary schema test.")
        return

    sample_content = "Check out the Dictionary Product, only $49.95!"
    extracted_json_string = strategy.extract(url="http://dummy.com/dictprod", html_content=sample_content)
    
    print("Dictionary Schema Extraction (mocked LLM):")
    if extracted_json_string:
        extracted_data = json.loads(extracted_json_string)
        print(json.dumps(extracted_data, indent=2))
        assert "product_name" in extracted_data
        assert "price" in extracted_data
    else:
        print("No data extracted.")

if __name__ == "__main__":
    dictionary_schema_extraction()
```

#### 3.2.3. **Nested Schemas:**

##### 3.2.3.1. Example: Using a Pydantic model with nested Pydantic models as fields.

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig
from pydantic import BaseModel
from typing import List
from unittest.mock import patch, MagicMock
import json

class Author(BaseModel):
    name: str
    email: Optional[str] = None

class Article(BaseModel):
    title: str
    author_details: Author # Nested Pydantic model
    tags: List[str]

# Mock LLM response
mock_llm_response_nested_schema = MagicMock()
mock_llm_response_nested_schema.choices = [MagicMock()]
mock_llm_response_nested_schema.choices[0].message.content = json.dumps({
    "title": "The Future of AI",
    "author_details": {"name": "Dr. AI Expert", "email": "ai@example.com"},
    "tags": ["AI", "ML", "Future"]
})
# ... (mock usage)
mock_llm_response_nested_schema.usage = MagicMock()
mock_llm_response_nested_schema.usage.completion_tokens = 40; mock_llm_response_nested_schema.usage.prompt_tokens = 120; mock_llm_response_nested_schema.usage.total_tokens = 160
mock_llm_response_nested_schema.usage.completion_tokens_details = {}; mock_llm_response_nested_schema.usage.prompt_tokens_details = {}


@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', return_value=mock_llm_response_nested_schema)
def pydantic_nested_schema_extraction(mock_perform_completion):
    try:
        strategy = LLMExtractionStrategy(
            llm_config=LLMConfig(provider="ollama/llama3", api_token="ollama"),
            schema=Article.model_json_schema(),
            extraction_type="schema"
        )
    except:
        print("Ollama not available, skipping Pydantic nested schema test.")
        return

    sample_content = "Article: The Future of AI by Dr. AI Expert (ai@example.com). Tags: AI, ML, Future."
    extracted_json_string = strategy.extract(url="http://dummy.com/article", html_content=sample_content)
    
    print("Pydantic Nested Schema Extraction (mocked LLM):")
    if extracted_json_string:
        extracted_data = json.loads(extracted_json_string)
        print(json.dumps(extracted_data, indent=2))
        article_instance = Article(**extracted_data)
        print(f"Validated Pydantic instance: {article_instance}")
        assert article_instance.author_details.name == "Dr. AI Expert"
    else:
        print("No data extracted.")

if __name__ == "__main__":
    pydantic_nested_schema_extraction()
```

##### 3.2.3.2. Example: Extracting a list of Pydantic model instances (e.g., list of products).

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig
from pydantic import BaseModel, Field
from typing import List
from unittest.mock import patch, MagicMock
import json

class Product(BaseModel):
    name: str
    price: float

class ProductList(BaseModel):
    products: List[Product] = Field(description="A list of products found on the page")


# Mock LLM response
mock_llm_response_list_schema = MagicMock()
mock_llm_response_list_schema.choices = [MagicMock()]
mock_llm_response_list_schema.choices[0].message.content = json.dumps({
    "products": [
        {"name": "Laptop Pro", "price": 1200.00},
        {"name": "Wireless Mouse", "price": 25.00},
        {"name": "Keyboard", "price": 75.00}
    ]
})
# ... (mock usage)
mock_llm_response_list_schema.usage = MagicMock()
mock_llm_response_list_schema.usage.completion_tokens = 50; mock_llm_response_list_schema.usage.prompt_tokens = 150; mock_llm_response_list_schema.usage.total_tokens = 200
mock_llm_response_list_schema.usage.completion_tokens_details = {}; mock_llm_response_list_schema.usage.prompt_tokens_details = {}


@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', return_value=mock_llm_response_list_schema)
def pydantic_list_extraction(mock_perform_completion):
    try:
        strategy = LLMExtractionStrategy(
            llm_config=LLMConfig(provider="ollama/llama3", api_token="ollama"),
            schema=ProductList.model_json_schema(),
            extraction_type="schema"
        )
    except:
        print("Ollama not available, skipping Pydantic list extraction test.")
        return

    sample_content = """
    Available products:
    1. Laptop Pro - $1200.00
    2. Wireless Mouse - $25.00
    3. Mechanical Keyboard - $75.00
    """
    extracted_json_string = strategy.extract(url="http://dummy.com/products", html_content=sample_content)
    
    print("Pydantic List Extraction (mocked LLM):")
    if extracted_json_string:
        extracted_data = json.loads(extracted_json_string)
        print(json.dumps(extracted_data, indent=2))
        product_list_instance = ProductList(**extracted_data)
        print(f"Validated Pydantic instance: {product_list_instance}")
        assert len(product_list_instance.products) == 3
        assert product_list_instance.products[0].name == "Laptop Pro"
    else:
        print("No data extracted.")

if __name__ == "__main__":
    pydantic_list_extraction()
```

#### 3.2.4. **Dynamic Schema Generation (Advanced):**

##### 3.2.4.1. Example: Programmatically generating a Pydantic model for the schema at runtime.

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig
from pydantic import create_model, BaseModel
from typing import List, Type
from unittest.mock import patch, MagicMock
import json

# Mock LLM response
mock_llm_response_dynamic_schema = MagicMock()
mock_llm_response_dynamic_schema.choices = [MagicMock()]
mock_llm_response_dynamic_schema.choices[0].message.content = json.dumps({
    "user_id": "user123",
    "username": "john_doe",
    "is_premium_member": True
})
# ... (mock usage)
mock_llm_response_dynamic_schema.usage = MagicMock()
mock_llm_response_dynamic_schema.usage.completion_tokens = 20; mock_llm_response_dynamic_schema.usage.prompt_tokens = 90; mock_llm_response_dynamic_schema.usage.total_tokens = 110
mock_llm_response_dynamic_schema.usage.completion_tokens_details = {}; mock_llm_response_dynamic_schema.usage.prompt_tokens_details = {}


@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', return_value=mock_llm_response_dynamic_schema)
def dynamic_schema_generation_extraction(mock_perform_completion):
    # Define fields dynamically
    fields_to_extract = {
        "user_id": (str, ...),  # '...' means required
        "username": (str, ...),
        "is_premium_member": (bool, False) # Optional with default
    }
    
    # Create Pydantic model dynamically
    DynamicUserModel: Type[BaseModel] = create_model(
        'DynamicUserModel',
        **fields_to_extract
    )

    try:
        strategy = LLMExtractionStrategy(
            llm_config=LLMConfig(provider="ollama/llama3", api_token="ollama"),
            schema=DynamicUserModel.model_json_schema(),
            extraction_type="schema"
        )
    except:
        print("Ollama not available, skipping dynamic schema test.")
        return

    sample_content = "User ID: user123, Username: john_doe, Premium: Yes"
    extracted_json_string = strategy.extract(url="http://dummy.com/userprofile", html_content=sample_content)
    
    print("Dynamic Schema Extraction (mocked LLM):")
    if extracted_json_string:
        extracted_data = json.loads(extracted_json_string)
        print(json.dumps(extracted_data, indent=2))
        user_instance = DynamicUserModel(**extracted_data)
        print(f"Validated Dynamic Pydantic instance: {user_instance}")
        assert user_instance.username == "john_doe"
    else:
        print("No data extracted.")

if __name__ == "__main__":
    dynamic_schema_generation_extraction()
```
---

### 3.3. Instruction Customization

#### 3.3.1. Example: Using `extraction_type="block"` with a custom `instruction` to guide block extraction (e.g., "Extract main paragraphs about AI").

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig
from unittest.mock import patch, MagicMock
import json

# Mock LLM response
mock_llm_response_block_instr = MagicMock()
mock_llm_response_block_instr.choices = [MagicMock()]
mock_llm_response_block_instr.choices[0].message.content = """
<blocks>
  <block>
    <content>Artificial intelligence is rapidly evolving.</content>
    <tags><tag>AI_paragraph</tag></tags>
  </block>
  <block>
    <content>It impacts various industries.</content>
    <tags><tag>AI_paragraph</tag></tags>
  </block>
</blocks>
"""
# ... (mock usage)
mock_llm_response_block_instr.usage = MagicMock()
mock_llm_response_block_instr.usage.completion_tokens = 30; mock_llm_response_block_instr.usage.prompt_tokens = 100; mock_llm_response_block_instr.usage.total_tokens = 130
mock_llm_response_block_instr.usage.completion_tokens_details = {}; mock_llm_response_block_instr.usage.prompt_tokens_details = {}


@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', return_value=mock_llm_response_block_instr)
def block_extraction_with_instruction(mock_perform_completion):
    try:
        strategy = LLMExtractionStrategy(
            llm_config=LLMConfig(provider="ollama/llama3", api_token="ollama"),
            extraction_type="block",
            instruction="Extract only the main paragraphs discussing Artificial Intelligence. Ignore other sections."
        )
    except:
        print("Ollama not available, skipping block extraction with instruction test.")
        return

    sample_content = """
# The Future of Computing
This is an intro.
## Artificial Intelligence
Artificial intelligence is rapidly evolving. It impacts various industries.
## unrelated section
This is not about AI.
"""
    extracted_data = strategy.extract(url="http://dummy.com/ai_article", html_content=sample_content)
    
    print("Block Extraction with Custom Instruction (mocked LLM):")
    print(json.dumps(extracted_data, indent=2))
    if extracted_data:
        assert len(extracted_data) == 2
        assert "Artificial intelligence" in extracted_data[0]["content"]

if __name__ == "__main__":
    block_extraction_with_instruction()
```

#### 3.3.2. Example: Using `extraction_type="schema"` with a Pydantic schema and a guiding `instruction` (e.g., "Extract product details according to the schema. Focus on electronics.").

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig
from pydantic import BaseModel
from unittest.mock import patch, MagicMock
import json

class ProductSchema(BaseModel):
    productName: str
    category: str
    price: Optional[float] = None

# Mock LLM response
mock_llm_response_schema_instr = MagicMock()
mock_llm_response_schema_instr.choices = [MagicMock()]
mock_llm_response_schema_instr.choices[0].message.content = json.dumps({
    "productName": "Smart TV",
    "category": "Electronics",
    "price": 499.99
})
# ... (mock usage)
mock_llm_response_schema_instr.usage = MagicMock()
mock_llm_response_schema_instr.usage.completion_tokens = 25; mock_llm_response_schema_instr.usage.prompt_tokens = 110; mock_llm_response_schema_instr.usage.total_tokens = 135
mock_llm_response_schema_instr.usage.completion_tokens_details = {}; mock_llm_response_schema_instr.usage.prompt_tokens_details = {}


@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', return_value=mock_llm_response_schema_instr)
def schema_extraction_with_instruction(mock_perform_completion):
    try:
        strategy = LLMExtractionStrategy(
            llm_config=LLMConfig(provider="ollama/llama3", api_token="ollama"),
            schema=ProductSchema.model_json_schema(),
            extraction_type="schema",
            instruction="Extract product details. Prioritize items in the 'Electronics' category if multiple products are mentioned."
        )
    except:
        print("Ollama not available, skipping schema extraction with instruction test.")
        return
        
    sample_content = "We sell books, clothes, and a Smart TV for $499.99. The Smart TV is an electronic device."
    extracted_json_string = strategy.extract(url="http://dummy.com/store", html_content=sample_content)
    
    print("Schema Extraction with Instruction (mocked LLM):")
    if extracted_json_string:
        extracted_data = json.loads(extracted_json_string)
        print(json.dumps(extracted_data, indent=2))
        product_instance = ProductSchema(**extracted_data)
        assert product_instance.category == "Electronics"
    else:
        print("No data extracted.")

if __name__ == "__main__":
    schema_extraction_with_instruction()
```

#### 3.3.3. Example: Using `extraction_type="schema_from_instruction"` where the LLM infers the schema from a detailed `instruction` (e.g., "Extract the title, author, and publication date of the article.").
This powerful feature lets the LLM decide the schema based on your textual request.

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig
from unittest.mock import patch, MagicMock
import json

# Mock LLM response - LLM invents the schema and fills it
mock_llm_response_infer_schema = MagicMock()
mock_llm_response_infer_schema.choices = [MagicMock()]
mock_llm_response_infer_schema.choices[0].message.content = json.dumps({
    "title": "Adventures in AI",
    "author": "Jane Coder",
    "publication_date": "2024-05-15"
})
# ... (mock usage)
mock_llm_response_infer_schema.usage = MagicMock()
mock_llm_response_infer_schema.usage.completion_tokens = 30; mock_llm_response_infer_schema.usage.prompt_tokens = 90; mock_llm_response_infer_schema.usage.total_tokens = 120
mock_llm_response_infer_schema.usage.completion_tokens_details = {}; mock_llm_response_infer_schema.usage.prompt_tokens_details = {}


@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', return_value=mock_llm_response_infer_schema)
def schema_from_instruction_extraction(mock_perform_completion):
    try:
        strategy = LLMExtractionStrategy(
            llm_config=LLMConfig(provider="ollama/llama3", api_token="ollama"),
            extraction_type="schema_from_instruction",
            instruction="Please extract the title, author, and publication date (YYYY-MM-DD) of this news article."
        )
    except:
        print("Ollama not available, skipping schema_from_instruction test.")
        return

    sample_content = """
    # Adventures in AI
    By Jane Coder, Published on May 15, 2024.
    This article explores the latest trends...
    """
    extracted_json_string = strategy.extract(url="http://dummy.com/news_article", html_content=sample_content)
    
    print("Schema from Instruction Extraction (mocked LLM):")
    if extracted_json_string:
        extracted_data = json.loads(extracted_json_string)
        print(json.dumps(extracted_data, indent=2))
        assert "title" in extracted_data and "author" in extracted_data and "publication_date" in extracted_data
    else:
        print("No data extracted.")

if __name__ == "__main__":
    schema_from_instruction_extraction()
```

#### 3.3.4. Example: Comparing outputs with and without a specific `instruction` when using `extraction_type="schema"`.

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig
from pydantic import BaseModel
from unittest.mock import patch, MagicMock
import json

class Event(BaseModel):
    eventName: str
    location: str
    date: str

# Mock LLM responses
mock_llm_no_instr = MagicMock()
mock_llm_no_instr.choices = [MagicMock()]
mock_llm_no_instr.choices[0].message.content = json.dumps({"eventName": "Tech Meetup", "location": "Online", "date": "2024-06-01"})
mock_llm_no_instr.usage = MagicMock(); mock_llm_no_instr.usage.completion_tokens = 20; mock_llm_no_instr.usage.prompt_tokens = 80; mock_llm_no_instr.usage.total_tokens = 100
mock_llm_no_instr.usage.completion_tokens_details = {}; mock_llm_no_instr.usage.prompt_tokens_details = {}


mock_llm_with_instr = MagicMock()
mock_llm_with_instr.choices = [MagicMock()]
mock_llm_with_instr.choices[0].message.content = json.dumps({"eventName": "AI Conference", "location": "San Francisco", "date": "2024-07-20"})
mock_llm_with_instr.usage = MagicMock(); mock_llm_with_instr.usage.completion_tokens = 22; mock_llm_with_instr.usage.prompt_tokens = 95; mock_llm_with_instr.usage.total_tokens = 117
mock_llm_with_instr.usage.completion_tokens_details = {}; mock_llm_with_instr.usage.prompt_tokens_details = {}


@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff')
def schema_extraction_instruction_comparison(mock_perform_completion):
    try:
        llm_conf = LLMConfig(provider="ollama/llama3", api_token="ollama")
    except:
        print("Ollama not available, skipping schema instruction comparison test.")
        return

    sample_content = """
    Upcoming Events:
    - Tech Meetup, Online, June 1st, 2024
    - AI Conference, San Francisco, July 20th, 2024
    - Local Bake Sale, Town Hall, June 5th, 2024
    """

    # Case 1: No specific instruction
    mock_perform_completion.return_value = mock_llm_no_instr
    strategy_no_instr = LLMExtractionStrategy(
        llm_config=llm_conf,
        schema=Event.model_json_schema(),
        extraction_type="schema"
    )
    result_no_instr_json = strategy_no_instr.extract(url="http://dummy.com/events", html_content=sample_content)
    result_no_instr = json.loads(result_no_instr_json) if result_no_instr_json else {}
    print(f"Without specific instruction: {result_no_instr}")

    # Case 2: With instruction to focus
    mock_perform_completion.return_value = mock_llm_with_instr
    strategy_with_instr = LLMExtractionStrategy(
        llm_config=llm_conf,
        schema=Event.model_json_schema(),
        extraction_type="schema",
        instruction="Focus on extracting details for the 'AI Conference'."
    )
    result_with_instr_json = strategy_with_instr.extract(url="http://dummy.com/events", html_content=sample_content)
    result_with_instr = json.loads(result_with_instr_json) if result_with_instr_json else {}
    print(f"With instruction to focus on AI Conference: {result_with_instr}")

    assert result_no_instr.get("eventName") == "Tech Meetup" # Mock returns first one
    assert result_with_instr.get("eventName") == "AI Conference" # Mock returns AI conf due to instruction

if __name__ == "__main__":
    schema_extraction_instruction_comparison()
```
---

### 3.4. Controlling `extraction_type`

#### 3.4.1. Example: Demonstrating `extraction_type="block"` - output structure (list of blocks with content and tags).

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig
from unittest.mock import patch, MagicMock
import json

# Mock LLM response for block extraction
mock_llm_block_output = MagicMock()
mock_llm_block_output.choices = [MagicMock()]
mock_llm_block_output.choices[0].message.content = """
<blocks>
  <block>
    <content>First important point.</content>
    <tags><tag>key_takeaway</tag></tags>
  </block>
  <block>
    <content>Second supporting detail.</content>
    <tags><tag>detail</tag><tag>supporting_info</tag></tags>
  </block>
</blocks>
"""
mock_llm_block_output.usage = MagicMock(); mock_llm_block_output.usage.completion_tokens=30; mock_llm_block_output.usage.prompt_tokens=70; mock_llm_block_output.usage.total_tokens=100
mock_llm_block_output.usage.completion_tokens_details = {}; mock_llm_block_output.usage.prompt_tokens_details = {}


@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', return_value=mock_llm_block_output)
def demonstrate_block_extraction_type(mock_perform_completion):
    try:
        strategy = LLMExtractionStrategy(
            llm_config=LLMConfig(provider="ollama/llama3", api_token="ollama"),
            extraction_type="block" # Explicitly set to block
        )
    except:
        print("Ollama not available, skipping block extraction type demo.")
        return

    sample_content = "Some text with a First important point and then a Second supporting detail."
    extracted_data = strategy.extract(url="http://dummy.com/blocks", html_content=sample_content)
    
    print("Block Extraction Output Structure (mocked LLM):")
    print(json.dumps(extracted_data, indent=2))
    
    # Expected output is a list of dictionaries (blocks)
    assert isinstance(extracted_data, list)
    if extracted_data:
        assert "content" in extracted_data[0]
        assert "tags" in extracted_data[0]
        assert isinstance(extracted_data[0]["tags"], list)

if __name__ == "__main__":
    demonstrate_block_extraction_type()
```

#### 3.4.2. Example: Demonstrating `extraction_type="schema"` - output structure (JSON string matching the schema).

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig
from pydantic import BaseModel
from unittest.mock import patch, MagicMock
import json

class MyData(BaseModel):
    field1: str
    field2: int

# Mock LLM response
mock_llm_schema_output = MagicMock()
mock_llm_schema_output.choices = [MagicMock()]
mock_llm_schema_output.choices[0].message.content = json.dumps({"field1": "value1", "field2": 123})
mock_llm_schema_output.usage = MagicMock(); mock_llm_schema_output.usage.completion_tokens=15; mock_llm_schema_output.usage.prompt_tokens=60; mock_llm_schema_output.usage.total_tokens=75
mock_llm_schema_output.usage.completion_tokens_details = {}; mock_llm_schema_output.usage.prompt_tokens_details = {}


@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', return_value=mock_llm_schema_output)
def demonstrate_schema_extraction_type(mock_perform_completion):
    try:
        strategy = LLMExtractionStrategy(
            llm_config=LLMConfig(provider="ollama/llama3", api_token="ollama"),
            schema=MyData.model_json_schema(),
            extraction_type="schema" # Explicitly set to schema
        )
    except:
        print("Ollama not available, skipping schema extraction type demo.")
        return

    sample_content = "Field one is value1 and field two is 123."
    extracted_json_string = strategy.extract(url="http://dummy.com/schema_data", html_content=sample_content)
    
    print("Schema Extraction Output Structure (mocked LLM):")
    print(f"Raw JSON string from LLM: {extracted_json_string}")
    
    # Expected output is a JSON string that can be parsed into the schema
    if extracted_json_string:
        data = json.loads(extracted_json_string)
        print(f"Parsed data: {data}")
        instance = MyData(**data) # Validate with Pydantic
        assert instance.field1 == "value1"

if __name__ == "__main__":
    demonstrate_schema_extraction_type()
```

#### 3.4.3. Example: Demonstrating `extraction_type="schema_from_instruction"` - output structure (JSON string based on inferred schema).

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig
from unittest.mock import patch, MagicMock
import json

# Mock LLM response - LLM infers schema and provides data
mock_llm_infer_schema_output = MagicMock()
mock_llm_infer_schema_output.choices = [MagicMock()]
mock_llm_infer_schema_output.choices[0].message.content = json.dumps({
    "book_title": "The LLM Handbook",
    "pages": 300
})
mock_llm_infer_schema_output.usage = MagicMock(); mock_llm_infer_schema_output.usage.completion_tokens=20; mock_llm_infer_schema_output.usage.prompt_tokens=70; mock_llm_infer_schema_output.usage.total_tokens=90
mock_llm_infer_schema_output.usage.completion_tokens_details = {}; mock_llm_infer_schema_output.usage.prompt_tokens_details = {}


@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', return_value=mock_llm_infer_schema_output)
def demonstrate_schema_from_instruction_type(mock_perform_completion):
    try:
        strategy = LLMExtractionStrategy(
            llm_config=LLMConfig(provider="ollama/llama3", api_token="ollama"),
            extraction_type="schema_from_instruction",
            instruction="Extract the book title and number of pages."
        )
    except:
        print("Ollama not available, skipping schema_from_instruction type demo.")
        return

    sample_content = "The book 'The LLM Handbook' contains 300 pages of valuable insights."
    extracted_json_string = strategy.extract(url="http://dummy.com/book_info", html_content=sample_content)
    
    print("Schema from Instruction Output Structure (mocked LLM):")
    print(f"Raw JSON string from LLM: {extracted_json_string}")
    
    if extracted_json_string:
        data = json.loads(extracted_json_string)
        print(f"Parsed data: {data}")
        assert "book_title" in data and "pages" in data

if __name__ == "__main__":
    demonstrate_schema_from_instruction_type()
```
---

### 3.5. LLM Configuration (`llm_config`)

#### 3.5.1. Example: Using the default LLM provider and model.
This assumes `OPENAI_API_KEY` is set in the environment, as OpenAI is often a default. If not, it will error or fallback if a global default is set elsewhere (less common for `LLMExtractionStrategy` without explicit config).

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig # For explicit configuration if needed
import os

# Note: For this example to run successfully without explicit llm_config, 
# the environment variable OPENAI_API_KEY should be set, or another
# global default LLM provider must be configured for LiteLLM.
# If neither is true, LLMExtractionStrategy() might raise an error.

try:
    # This will try to use the default provider (often OpenAI if key is set)
    # or another globally configured default for LiteLLM.
    strategy_default = LLMExtractionStrategy() 
    print(f"Successfully initialized LLMExtractionStrategy with default provider: {strategy_default.llm_config.provider}")
    # To make this example runnable without a real API call:
    print("Note: Actual extraction would require a configured LLM and API key.")
except Exception as e:
    print(f"Failed to initialize with default LLM provider: {e}")
    print("This example requires a default LLM (e.g., OPENAI_API_KEY set) or LiteLLM global config.")
    print("Alternatively, provide an explicit LLMConfig to LLMExtractionStrategy.")

# To show an explicit (but still default-targeting) configuration:
# if os.getenv("OPENAI_API_KEY"):
#     llm_config_openai = LLMConfig(provider="openai/gpt-3.5-turbo", api_token=os.getenv("OPENAI_API_KEY"))
#     strategy_explicit_openai = LLMExtractionStrategy(llm_config=llm_config_openai)
#     print(f"Initialized explicitly with OpenAI: {strategy_explicit_openai.llm_config.provider}")
# else:
#     print("OPENAI_API_KEY not set, cannot show explicit OpenAI example.")
```

#### 3.5.2. Example: Configuring `LLMExtractionStrategy` with a specific OpenAI model via `LLMConfig` (e.g., `openai/gpt-4o-mini`).

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig
import os

# Ensure you have your OPENAI_API_KEY set in your environment variables
openai_api_key = os.getenv("OPENAI_API_KEY")

if not openai_api_key:
    print("OPENAI_API_KEY not found in environment. Skipping OpenAI example.")
    print("To run this, set your OPENAI_API_KEY.")
else:
    llm_config_openai = LLMConfig(
        provider="openai/gpt-4o-mini", # Specify the OpenAI model
        api_token=openai_api_key
    )
    strategy_openai = LLMExtractionStrategy(llm_config=llm_config_openai)
    print(f"Initialized LLMExtractionStrategy with OpenAI provider: {strategy_openai.llm_config.provider}")
    # To test, you would call strategy_openai.extract(...)
    # For this example, we'll just show initialization.
    print("Strategy ready to use OpenAI gpt-4o-mini.")
```

#### 3.5.3. Example: Configuring `LLMExtractionStrategy` with a specific Ollama model via `LLMConfig` (e.g., `ollama/llama3`).
This requires Ollama to be running locally and the specified model (e.g., `llama3`) to be pulled.

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig

# Assumes Ollama is running locally and 'llama3' model is available
# 'api_token' for Ollama is typically 'ollama' or can be omitted if not needed by your setup.
try:
    llm_config_ollama = LLMConfig(
        provider="ollama/llama3", 
        api_token="ollama", # Often 'ollama' or can be None if not required by local setup
        base_url="http://localhost:11434" # Default Ollama API URL
    )
    strategy_ollama = LLMExtractionStrategy(llm_config=llm_config_ollama)
    print(f"Initialized LLMExtractionStrategy with Ollama provider: {strategy_ollama.llm_config.provider}")
    print("Strategy ready to use Ollama with llama3.")
    print("Note: For actual extraction, ensure Ollama server is running and has the 'llama3' model pulled.")
except Exception as e:
    print(f"Failed to initialize Ollama strategy: {e}")
    print("Ensure Ollama is running (e.g., `ollama serve`) and you have pulled the model (e.g., `ollama pull llama3`).")

# Example of a test call (would require Ollama to be active)
# from unittest.mock import patch, MagicMock
# import json
# mock_response = MagicMock()
# mock_response.choices = [MagicMock()]
# mock_response.choices[0].message.content = json.dumps({"info": "extracted by ollama"})
# mock_response.usage = MagicMock(completion_tokens=5, prompt_tokens=10, total_tokens=15)
# mock_response.usage.completion_tokens_details = {}; mock_response.usage.prompt_tokens_details = {}
# @patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', return_value=mock_response)
# def _test_ollama_call(mock_call, strategy):
#     if strategy:
#         result = strategy.extract("url", "content", extraction_type="schema_from_instruction", instruction="get info")
#         print(f"Ollama mock call result: {result}")
# _test_ollama_call(None, strategy_ollama if 'strategy_ollama' in locals() else None)
```

#### 3.5.4. Example: Configuring `LLMExtractionStrategy` with a specific Gemini model via `LLMConfig`.

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig
import os

gemini_api_key = os.getenv("GEMINI_API_KEY")

if not gemini_api_key:
    print("GEMINI_API_KEY not found in environment. Skipping Gemini example.")
    print("To run this, set your GEMINI_API_KEY.")
else:
    llm_config_gemini = LLMConfig(
        provider="gemini/gemini-1.5-pro-latest", # Or another Gemini model
        api_token=gemini_api_key
    )
    strategy_gemini = LLMExtractionStrategy(llm_config=llm_config_gemini)
    print(f"Initialized LLMExtractionStrategy with Gemini provider: {strategy_gemini.llm_config.provider}")
    print("Strategy ready to use Gemini.")
```

#### 3.5.5. Example: Configuring `LLMExtractionStrategy` with a specific Anthropic (Claude) model via `LLMConfig`.

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig
import os

anthropic_api_key = os.getenv("ANTHROPIC_API_KEY")

if not anthropic_api_key:
    print("ANTHROPIC_API_KEY not found in environment. Skipping Anthropic Claude example.")
    print("To run this, set your ANTHROPIC_API_KEY.")
else:
    llm_config_claude = LLMConfig(
        provider="anthropic/claude-3-opus-20240229", # Or another Claude model
        api_token=anthropic_api_key
    )
    strategy_claude = LLMExtractionStrategy(llm_config=llm_config_claude)
    print(f"Initialized LLMExtractionStrategy with Anthropic provider: {strategy_claude.llm_config.provider}")
    print("Strategy ready to use Claude.")

```

#### 3.5.6. Example: Overriding LLM parameters like `temperature` and `max_tokens` using `LLMConfig`.

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig
import os

openai_api_key = os.getenv("OPENAI_API_KEY")

if not openai_api_key:
    print("OPENAI_API_KEY not found. This example shows parameter override with OpenAI.")
else:
    llm_config_custom_params = LLMConfig(
        provider="openai/gpt-3.5-turbo", # Using a common model for this example
        api_token=openai_api_key,
        temperature=0.2,       # Lower temperature for more deterministic output
        max_tokens=150         # Limit the maximum number of tokens in the response
        # You can add other provider-specific parameters here in extra_args if needed
        # extra_args={"top_p": 0.9} 
    )
    
    strategy_custom_params = LLMExtractionStrategy(
        llm_config=llm_config_custom_params,
        instruction="Extract the main point." # A simple instruction
    )
    
    print(f"Initialized LLMExtractionStrategy with custom LLM parameters for provider: {strategy_custom_params.llm_config.provider}")
    print(f"  Temperature: {strategy_custom_params.llm_config.temperature}")
    print(f"  Max Tokens: {strategy_custom_params.llm_config.max_tokens}")
    # print(f"  Extra Args: {strategy_custom_params.extra_args}") # Note: extra_args on LLMExtractionStrategy, not LLMConfig for this
    
    # A mock test to show parameters would be passed to perform_completion_with_backoff
    # from unittest.mock import patch, MagicMock
    # mock_response = MagicMock() # ... (setup mock_response) ...
    # @patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', return_value=mock_response)
    # def _test_params(mock_call):
    #     strategy_custom_params.extract("url", "content")
    #     called_args = mock_call.call_args
    #     assert called_args is not None
    #     assert called_args.kwargs.get('temperature') == 0.2
    #     assert called_args.kwargs.get('max_tokens') == 150
    #     print("LLM call would have used temperature=0.2 and max_tokens=150.")
    # _test_params()
```

#### 3.5.7. Example: Using `LLMConfig` to specify a custom `base_url` for a self-hosted LLM.
This is useful for local LLMs like Ollama (already shown), vLLM, or other self-hosted OpenAI-compatible endpoints.

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig

# Example for a generic OpenAI-compatible API running locally
# The provider name might need to be "openai/custom-model-name" or just "custom/model-name"
# depending on how LiteLLM handles it. For Ollama, it's typically "ollama/model-name".
custom_llm_config = LLMConfig(
    provider="custom/my-local-model", # Or "openai/my-local-model"
    api_token="no_key_needed_for_local", # Or your actual local key
    base_url="http://localhost:8000/v1" # Adjust to your local LLM API endpoint
)

try:
    strategy_custom_endpoint = LLMExtractionStrategy(llm_config=custom_llm_config)
    print(f"Initialized LLMExtractionStrategy with custom endpoint:")
    print(f"  Provider: {strategy_custom_endpoint.llm_config.provider}")
    print(f"  Base URL: {strategy_custom_endpoint.llm_config.base_url}")
    print("Note: This example assumes an OpenAI-compatible API is running at the specified base_url.")
except Exception as e:
    print(f"Failed to initialize custom endpoint strategy: {e}")

# Example for Ollama (already covered more specifically, but fits here too)
ollama_local_config = LLMConfig(
    provider="ollama/mistral", # Assuming mistral model is pulled
    base_url="http://localhost:11434", # Default Ollama
    api_token="ollama" # Usually 'ollama' or None
)
try:
    strategy_ollama_local = LLMExtractionStrategy(llm_config=ollama_local_config)
    print(f"\nInitialized LLMExtractionStrategy with local Ollama endpoint:")
    print(f"  Provider: {strategy_ollama_local.llm_config.provider}")
    print(f"  Base URL: {strategy_ollama_local.llm_config.base_url}")
except Exception as e:
    print(f"Failed to initialize local Ollama strategy via custom base_url example: {e}")
```
---

### 3.6. Chunking Configuration (`apply_chunking` and related parameters)

#### 3.6.1. Example: Default chunking behavior (`apply_chunking=True`).
When content is long, `LLMExtractionStrategy` automatically chunks it.

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig
from unittest.mock import patch, MagicMock
import json

# Mock LLM to be called multiple times if chunking happens
mock_responses = []
for i in range(3): # Simulate 3 chunks
    mock_resp = MagicMock()
    mock_resp.choices = [MagicMock()]
    mock_resp.choices[0].message.content = json.dumps({"chunk_data": f"Data from chunk {i+1}"})
    mock_resp.usage = MagicMock(completion_tokens=10, prompt_tokens=50, total_tokens=60)
    mock_resp.usage.completion_tokens_details = {}; mock_resp.usage.prompt_tokens_details = {}
    mock_responses.append(mock_resp)

@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', side_effect=mock_responses)
def default_chunking_behavior(mock_perform_completion):
    try:
        # Use a small chunk_token_threshold to force chunking for the example
        strategy = LLMExtractionStrategy(
            llm_config=LLMConfig(provider="ollama/llama3", api_token="ollama"),
            extraction_type="schema_from_instruction",
            instruction="Extract data from this chunk.",
            chunk_token_threshold=10, # Very small to ensure chunking
            word_token_rate=0.75, # Approx tokens per word
            apply_chunking=True # Default
        )
    except:
        print("Ollama not available, skipping default chunking test.")
        return

    # Create content long enough to be chunked based on threshold and word_token_rate
    # (10 tokens / 0.75 tokens/word) approx 13 words for threshold.
    # Let's use 50 words.
    long_content = " ".join(["word"] * 50) 
    
    extracted_data_json = strategy.extract(url="http://dummy.com/long_content", html_content=long_content)
    
    print("Default Chunking Behavior (mocked LLM):")
    # The strategy internally merges results from chunks if schema-based.
    # For "schema_from_instruction", it might return a list of JSON strings or a merged JSON.
    # Current LLMExtractionStrategy for schema type returns a single JSON string, implying merging.
    # If it's block extraction, it would be a list of blocks.
    # Let's assume for schema extraction, it returns a list of dicts before final JSON dump in the example
    
    # The mock setup implies multiple calls. LLMExtractionStrategy aggregates results.
    # If the LLM returns a list for each chunk, results would be concatenated.
    # If it returns a dict, they'd be in a list.
    # The current mock returns a dict per chunk, so we expect a list of dicts if not merged by strategy.
    # However, the `extract` method's return is a single JSON string if schema-based.
    # This means the strategy handles merging internally or expects LLM to handle it.
    # For this test, we'll check if the LLM was called multiple times.
    
    print(f"LLM called {mock_perform_completion.call_count} times.")
    assert mock_perform_completion.call_count > 1, "Chunking should have occurred, LLM expected to be called multiple times."
    print(f"Final extracted JSON string: {extracted_data_json}")
    # Final result depends on how LLM merges/formats if it gets multiple chunk results.
    # Assuming the mock's structure (list of chunk_data) would be presented as a list by the LLM.
    if extracted_data_json:
        final_data = json.loads(extracted_data_json)
        print(json.dumps(final_data, indent=2))
        # Depending on LLM's aggregation logic, this might be a list or a single dict.
        # For this example, if our mock returns individual dicts, and the LLM is asked to provide a final JSON,
        # it might wrap them in a list. Let's assume it does.
        # For this particular mock structure and schema_from_instruction, the LLM would typically be
        # instructed to return a list of items if the content implies multiple items.
        # Here, the mock returns individual dicts, and the strategy just returns the last one.
        # To properly test chunk aggregation, a more sophisticated mock or real LLM is needed.
        # For now, verifying multiple calls is the main goal.

if __name__ == "__main__":
    default_chunking_behavior()
```

#### 3.6.2. Example: Disabling chunking (`apply_chunking=False`) for short content.

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig
from unittest.mock import patch, MagicMock
import json

mock_llm_no_chunking = MagicMock()
mock_llm_no_chunking.choices = [MagicMock()]
mock_llm_no_chunking.choices[0].message.content = json.dumps({"summary": "This is short content."})
mock_llm_no_chunking.usage = MagicMock(completion_tokens=5, prompt_tokens=20, total_tokens=25)
mock_llm_no_chunking.usage.completion_tokens_details = {}; mock_llm_no_chunking.usage.prompt_tokens_details = {}

@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', return_value=mock_llm_no_chunking)
def disable_chunking_behavior(mock_perform_completion):
    try:
        strategy = LLMExtractionStrategy(
            llm_config=LLMConfig(provider="ollama/llama3", api_token="ollama"),
            apply_chunking=False, # Explicitly disable chunking
            extraction_type="schema_from_instruction",
            instruction="Summarize this."
        )
    except:
        print("Ollama not available, skipping disable chunking test.")
        return

    # Content is short enough that chunking wouldn't happen anyway, but this forces it.
    short_content = "This is a piece of short content." 
    extracted_data_json = strategy.extract(url="http://dummy.com/short", html_content=short_content)
    
    print("Disabled Chunking Behavior (mocked LLM):")
    print(f"LLM called {mock_perform_completion.call_count} time(s).")
    assert mock_perform_completion.call_count == 1, "Chunking was disabled, LLM should be called once."
    if extracted_data_json:
        print(json.dumps(json.loads(extracted_data_json), indent=2))


if __name__ == "__main__":
    disable_chunking_behavior()
```

#### 3.6.3. Example: Customizing `chunk_token_threshold` for smaller/larger chunks.

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig
from unittest.mock import patch, MagicMock
import json

# Mock setup
mock_llm_response_chunk_size = MagicMock()
mock_llm_response_chunk_size.choices = [MagicMock()]
mock_llm_response_chunk_size.choices[0].message.content = json.dumps({"info": "some data"})
mock_llm_response_chunk_size.usage = MagicMock(completion_tokens=5, prompt_tokens=10, total_tokens=15)
mock_llm_response_chunk_size.usage.completion_tokens_details = {}; mock_llm_response_chunk_size.usage.prompt_tokens_details = {}


@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', return_value=mock_llm_response_chunk_size)
def customize_chunk_token_threshold(mock_perform_completion):
    long_text = " ".join(["word_for_testing_chunk_size"] * 100) # Approx 100 words
    
    # Scenario 1: Small threshold, more chunks
    try:
        strategy_small_chunks = LLMExtractionStrategy(
            llm_config=LLMConfig(provider="ollama/llama3", api_token="ollama"),
            chunk_token_threshold=20, # Small threshold, e.g., ~26 words
            word_token_rate=0.75,
            extraction_type="schema_from_instruction", instruction="get info"
        )
    except:
        print("Ollama not available, cannot run small chunk test.")
        strategy_small_chunks = None

    if strategy_small_chunks:
        strategy_small_chunks.extract(url="http://dummy.com/small_chunks", html_content=long_text)
        print(f"Small chunk_token_threshold (20): LLM called {mock_perform_completion.call_count} times.")
        small_chunk_calls = mock_perform_completion.call_count
        mock_perform_completion.reset_mock() # Reset for next call
    else:
        small_chunk_calls = 0

    # Scenario 2: Larger threshold, fewer chunks
    try:
        strategy_large_chunks = LLMExtractionStrategy(
            llm_config=LLMConfig(provider="ollama/llama3", api_token="ollama"),
            chunk_token_threshold=80, # Larger threshold, e.g., ~106 words
            word_token_rate=0.75,
            extraction_type="schema_from_instruction", instruction="get info"
        )
    except:
        print("Ollama not available, cannot run large chunk test.")
        strategy_large_chunks = None

    if strategy_large_chunks:
        strategy_large_chunks.extract(url="http://dummy.com/large_chunks", html_content=long_text)
        print(f"Large chunk_token_threshold (80): LLM called {mock_perform_completion.call_count} times.")
        large_chunk_calls = mock_perform_completion.call_count
    else:
        large_chunk_calls = 0
        
    if strategy_small_chunks and strategy_large_chunks:
        assert small_chunk_calls > large_chunk_calls, "Smaller threshold should result in more LLM calls."

if __name__ == "__main__":
    customize_chunk_token_threshold()
```

#### 3.6.4. Example: Customizing `overlap_rate` between chunks.

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig
from unittest.mock import patch, MagicMock
import json

# Mock setup - we'll primarily observe the number of calls or potentially the prompt content
mock_llm_overlap = MagicMock()
mock_llm_overlap.choices = [MagicMock()]
mock_llm_overlap.choices[0].message.content = json.dumps({"data_point": "value"})
mock_llm_overlap.usage = MagicMock(completion_tokens=5, prompt_tokens=10, total_tokens=15)
mock_llm_overlap.usage.completion_tokens_details = {}; mock_llm_overlap.usage.prompt_tokens_details = {}


@patch('crawl4ai.extraction_strategy.LLMExtractionStrategy._create_llm_query_tasks') # Patching internal method to inspect chunks
def customize_overlap_rate(mock_create_tasks):
    # The _create_llm_query_tasks method is a good place to see the generated chunks.
    # It returns a list of partials. We can inspect the 'content' arg of the partials.
    
    long_text = "This is a moderately long text to demonstrate the effect of chunk overlap. " * 5
    # word_token_rate (default 0.75) means ~1 token per 1.33 words.
    # chunk_token_threshold (default 2048) is large.
    # Let's use a smaller threshold for demonstration.
    # Threshold 30 tokens => ~40 words.
    # Overlap 0.1 => 3 token overlap => ~4 words.
    # Overlap 0.5 => 15 token overlap => ~20 words.

    # Scenario 1: Small overlap
    try:
        strategy_small_overlap = LLMExtractionStrategy(
            llm_config=LLMConfig(provider="ollama/llama3", api_token="ollama"),
            chunk_token_threshold=30,
            overlap_rate=0.1, # 10% overlap
            extraction_type="block" # Block type is easier to see chunk content
        )
    except:
        print("Ollama not available. Skipping overlap rate test.")
        return

    strategy_small_overlap.extract(url="http://dummy.com/overlap1", html_content=long_text)
    chunks_small_overlap = [task.args[1] for task in mock_create_tasks.call_args[0][0]] # (tasks_list, url)
    print(f"Small overlap_rate (0.1) generated {len(chunks_small_overlap)} chunks.")
    if len(chunks_small_overlap) > 1:
        print(f"  Chunk 1 (end): ...{chunks_small_overlap[0][-30:]}")
        print(f"  Chunk 2 (start): {chunks_small_overlap[1][:30]}...")
    mock_create_tasks.reset_mock()

    # Scenario 2: Larger overlap
    strategy_large_overlap = LLMExtractionStrategy(
        llm_config=LLMConfig(provider="ollama/llama3", api_token="ollama"),
        chunk_token_threshold=30,
        overlap_rate=0.5, # 50% overlap
        extraction_type="block"
    )
    strategy_large_overlap.extract(url="http://dummy.com/overlap2", html_content=long_text)
    chunks_large_overlap = [task.args[1] for task in mock_create_tasks.call_args[0][0]]
    print(f"Large overlap_rate (0.5) generated {len(chunks_large_overlap)} chunks.")
    if len(chunks_large_overlap) > 1:
        print(f"  Chunk 1 (end): ...{chunks_large_overlap[0][-30:]}")
        print(f"  Chunk 2 (start): {chunks_large_overlap[1][:30]}...")
    
    # With more overlap, for the same content and threshold, you might get more chunks,
    # or similar number of chunks but with more redundant content.
    # This example primarily shows the parameter being used.
    # Actual number of chunks can be complex to predict without exact tokenization.

if __name__ == "__main__":
    customize_overlap_rate()
```

#### 3.6.5. Example: Demonstrating the effect of different `word_token_rate` values.
The `word_token_rate` helps estimate token count from word count for chunking.

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig
from unittest.mock import patch, MagicMock

# We'll patch the internal chunking function to see how many chunks are made.
# Or, more simply, observe the number of LLM calls if apply_chunking=True.

mock_llm_wtr = MagicMock() # Generic mock for counting calls
mock_llm_wtr.choices = [MagicMock(message=MagicMock(content="{}"))]
mock_llm_wtr.usage = MagicMock(completion_tokens=1, prompt_tokens=1, total_tokens=2)
mock_llm_wtr.usage.completion_tokens_details = {}; mock_llm_wtr.usage.prompt_tokens_details = {}

@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', return_value=mock_llm_wtr)
def customize_word_token_rate(mock_perform_completion):
    # Approx 50 words.
    # word_token_rate helps estimate token length for chunking.
    # chunk_token_threshold default is large (2048), so let's use a smaller one.
    test_content = "This is a test sentence. It has ten words precisely. " * 5 

    try:
        # Scenario 1: Lower word_token_rate (means more words per token, so fewer tokens for same text)
        # Should result in fewer chunks if text length is near threshold.
        strategy_low_wtr = LLMExtractionStrategy(
            llm_config=LLMConfig(provider="ollama/llama3", api_token="ollama"),
            chunk_token_threshold=30, # e.g. needs 30 tokens
            word_token_rate=0.5, # Estimates 0.5 tokens per word (i.e., 2 words/token)
                                 # So 50 words -> ~25 tokens. Should be 1 chunk.
            extraction_type="block"
        )
        strategy_low_wtr.extract("url", test_content)
        calls_low_wtr = mock_perform_completion.call_count
        print(f"word_token_rate=0.5 (estimates fewer tokens): LLM calls = {calls_low_wtr}")
        mock_perform_completion.reset_mock()

        # Scenario 2: Higher word_token_rate (means fewer words per token, so more tokens for same text)
        # Should result in more chunks if text length is near threshold.
        strategy_high_wtr = LLMExtractionStrategy(
            llm_config=LLMConfig(provider="ollama/llama3", api_token="ollama"),
            chunk_token_threshold=30,
            word_token_rate=1.0, # Estimates 1 token per word
                                 # So 50 words -> ~50 tokens. Should be >1 chunk.
            extraction_type="block"
        )
        strategy_high_wtr.extract("url", test_content)
        calls_high_wtr = mock_perform_completion.call_count
        print(f"word_token_rate=1.0 (estimates more tokens): LLM calls = {calls_high_wtr}")

        assert calls_high_wtr >= calls_low_wtr, \
            "Higher word_token_rate should lead to more or equal chunks for the same content and token threshold."

    except Exception as e:
        print(f"Ollama not available or other error, skipping word_token_rate test: {e}")


if __name__ == "__main__":
    customize_word_token_rate()
```

#### 3.6.6. Example: Processing very large content that requires multiple chunks.
This is similar to 3.6.1, emphasizing that the system handles large inputs by breaking them down.

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig
from unittest.mock import patch, MagicMock
import json

# Simulate multiple LLM calls for multiple chunks
mock_responses_large_content = []
for i in range(5): # Expecting around 5 chunks for this example
    mock_resp = MagicMock()
    mock_resp.choices = [MagicMock()]
    # Simulate LLM returning structured data for each chunk
    mock_resp.choices[0].message.content = json.dumps({"document_part": f"Content from part {i+1} of the large document."})
    mock_resp.usage = MagicMock(completion_tokens=15, prompt_tokens=100, total_tokens=115) # Example usage
    mock_resp.usage.completion_tokens_details = {}; mock_resp.usage.prompt_tokens_details = {}
    mock_responses_large_content.append(mock_resp)


@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', side_effect=mock_responses_large_content)
def process_very_large_content(mock_perform_completion):
    # Content designed to be split into several chunks
    # Assuming chunk_token_threshold=50 and word_token_rate=0.75 (~66 words/chunk)
    # This content has 300 words. 300 / (50/0.75) = 300 / 66.6 = ~4.5 chunks
    very_large_content = ("This is a segment of a very large document that needs to be processed. " * 60) # 300 words

    try:
        strategy = LLMExtractionStrategy(
            llm_config=LLMConfig(provider="ollama/llama3", api_token="ollama"),
            extraction_type="schema_from_instruction",
            instruction="Extract key information from this segment of the document.",
            chunk_token_threshold=50, # Smaller threshold to ensure multiple chunks
            word_token_rate=0.75,      # Estimate tokens per word
            apply_chunking=True
        )
    except:
        print("Ollama not available, skipping large content chunking test.")
        return

    print("Processing very large content (mocked LLM calls per chunk)...")
    # The extract method will handle the chunking and aggregation if extraction_type is schema-based.
    # The final extracted_data_json should ideally be a merged/structured result.
    # For this mock, the LLM is assumed to return a list of objects if instruction implies it.
    # Here, we'll get the last chunk's result if schema_from_instruction unless LLM aggregates.
    # To truly show aggregation, a more complex mocking or real LLM is needed.
    # Focus here is on the multiple calls due to chunking.
    
    extracted_data_json_string = strategy.extract(url="http://dummy.com/very_large_doc", html_content=very_large_content)
    
    print(f"LLM was called {mock_perform_completion.call_count} times due to chunking.")
    assert mock_perform_completion.call_count > 1, "Expected multiple LLM calls for large content."

    print("Final Extracted Data (structure depends on LLM's handling of chunked results):")
    if extracted_data_json_string:
        print(json.dumps(json.loads(extracted_data_json_string), indent=2))
    else:
        print("No data extracted or an error occurred.")
        
if __name__ == "__main__":
    process_very_large_content()
```
---

### 3.7. Input Format Selection (`input_format`)

#### 3.7.1. Example: Extracting from Markdown content (`input_format="markdown"`, default).
This is the default behavior if `input_format` is not specified.

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig
from unittest.mock import patch, MagicMock
import json

mock_llm_md_input = MagicMock() # Setup mock as before
mock_llm_md_input.choices = [MagicMock(message=MagicMock(content=json.dumps({"title": "Markdown Test"})))]
mock_llm_md_input.usage = MagicMock(completion_tokens=5, prompt_tokens=30, total_tokens=35)
mock_llm_md_input.usage.completion_tokens_details = {}; mock_llm_md_input.usage.prompt_tokens_details = {}


@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', return_value=mock_llm_md_input)
def extract_from_markdown(mock_perform_completion):
    try:
        strategy = LLMExtractionStrategy(
            llm_config=LLMConfig(provider="ollama/llama3", api_token="ollama"),
            input_format="markdown", # Explicitly set, though it's the default
            extraction_type="schema_from_instruction",
            instruction="Extract the main title."
        )
    except:
        print("Ollama not available, skipping markdown input test.")
        return

    sample_markdown = "# Markdown Test\nThis is some **bold** text."
    extracted_json = strategy.extract(url="http://dummy.com/md_page", html_content=sample_markdown)
    
    print("Extraction from Markdown (mocked LLM):")
    if extracted_json:
        print(json.dumps(json.loads(extracted_json), indent=2))

if __name__ == "__main__":
    extract_from_markdown()
```

#### 3.7.2. Example: Extracting directly from raw HTML (`input_format="html"`).

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig
from unittest.mock import patch, MagicMock
import json

mock_llm_html_input = MagicMock() # Setup mock
mock_llm_html_input.choices = [MagicMock(message=MagicMock(content=json.dumps({"page_heading": "HTML Document"})))]
mock_llm_html_input.usage = MagicMock(completion_tokens=6, prompt_tokens=40, total_tokens=46)
mock_llm_html_input.usage.completion_tokens_details = {}; mock_llm_html_input.usage.prompt_tokens_details = {}


@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', return_value=mock_llm_html_input)
def extract_from_raw_html(mock_perform_completion):
    try:
        strategy = LLMExtractionStrategy(
            llm_config=LLMConfig(provider="ollama/llama3", api_token="ollama"),
            input_format="html",
            extraction_type="schema_from_instruction",
            instruction="Extract the main heading (h1)."
        )
    except:
        print("Ollama not available, skipping raw HTML input test.")
        return

    sample_html = "<html><head><title>Test</title></head><body><h1>HTML Document</h1><p>Content</p></body></html>"
    extracted_json = strategy.extract(url="http://dummy.com/html_page", html_content=sample_html)
    
    print("Extraction from Raw HTML (mocked LLM):")
    if extracted_json:
        print(json.dumps(json.loads(extracted_json), indent=2))

if __name__ == "__main__":
    extract_from_raw_html()
```

#### 3.7.3. Example: Extracting from filtered HTML (`input_format="fit_html"`) after `MarkdownGenerator` with a `ContentFilterStrategy` has run.
This example shows a two-step process: first filtering HTML using `MarkdownGenerator` and a `ContentFilterStrategy`, then feeding its `fit_html` output to `LLMExtractionStrategy`.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, DefaultMarkdownGenerator, LLMConfig
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.content_filter_strategy import PruningContentFilter # Example filter
from unittest.mock import patch, MagicMock
import json

# Mock for the LLMExtractionStrategy part
mock_llm_fit_html = MagicMock()
mock_llm_fit_html.choices = [MagicMock(message=MagicMock(content=json.dumps({"main_content_summary": "Summary of pruned content."})))]
mock_llm_fit_html.usage = MagicMock(completion_tokens=10, prompt_tokens=50, total_tokens=60)
mock_llm_fit_html.usage.completion_tokens_details = {}; mock_llm_fit_html.usage.prompt_tokens_details = {}


@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', return_value=mock_llm_fit_html)
async def extract_from_fit_html(mock_perform_completion):
    # Step 1: Setup MarkdownGenerator with a content filter to produce fit_html
    # For this example, we'll use PruningContentFilter.
    # In a real scenario, you might need an LLM for more advanced filters.
    # We'll use a simple mock HTML for this part.
    
    sample_raw_html = """
    <html><body>
        <header>Site Navigation</header>
        <nav>Links...</nav>
        <main>
            <h1>Main Article Title</h1>
            <p>This is the core content we want to keep.</p>
            <p>Another paragraph of important stuff.</p>
        </main>
        <aside>Related links</aside>
        <footer>Copyright info</footer>
    </body></html>
    """

    # Simulate getting fit_html (normally from crawler.arun() and result.markdown.fit_html)
    # Here we manually instantiate and run the filter's logic conceptually
    # Note: MarkdownGenerator itself creates fit_html when a filter is active
    # For simplicity, let's assume PruningContentFilter directly gives us usable HTML for LLM
    
    # A more accurate simulation would involve creating a MarkdownGenerator
    # and getting its `fit_html`. PruningContentFilter directly manipulates soup.
    from bs4 import BeautifulSoup
    soup = BeautifulSoup(sample_raw_html, "lxml")
    pruning_filter = PruningContentFilter() 
    # PruningContentFilter.filter_content modifies soup in-place and returns string list
    # We will simulate its effect by just taking the main content for this test
    main_content_element = soup.find("main")
    fit_html_content = str(main_content_element) if main_content_element else "<p>Filtered content.</p>"
    
    print(f"--- Simulated Fit HTML (for LLM input) ---\n{fit_html_content}\n--------------------------------------")

    # Step 2: Use LLMExtractionStrategy with input_format="fit_html" (or just "html" if it's valid HTML)
    try:
        strategy = LLMExtractionStrategy(
            llm_config=LLMConfig(provider="ollama/llama3", api_token="ollama"),
            input_format="html", # fit_html is still HTML, or use "fit_html" if specific handling is added
            extraction_type="schema_from_instruction",
            instruction="Summarize the main content provided."
        )
    except:
        print("Ollama not available, skipping fit_html extraction test.")
        return

    extracted_json = strategy.extract(url="http://dummy.com/filtered_page", html_content=fit_html_content)
    
    print("\nExtraction from Fit HTML (mocked LLM):")
    if extracted_json:
        print(json.dumps(json.loads(extracted_json), indent=2))
    
    assert mock_perform_completion.called

if __name__ == "__main__":
    asyncio.run(extract_from_fit_html())
```

#### 3.7.4. Example: Extracting from plain text content (`input_format="text"`).

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig
from unittest.mock import patch, MagicMock
import json

mock_llm_text_input = MagicMock() # Setup mock
mock_llm_text_input.choices = [MagicMock(message=MagicMock(content=json.dumps({"sentiment": "positive"})))]
mock_llm_text_input.usage = MagicMock(completion_tokens=3, prompt_tokens=25, total_tokens=28)
mock_llm_text_input.usage.completion_tokens_details = {}; mock_llm_text_input.usage.prompt_tokens_details = {}


@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', return_value=mock_llm_text_input)
def extract_from_plain_text(mock_perform_completion):
    try:
        strategy = LLMExtractionStrategy(
            llm_config=LLMConfig(provider="ollama/llama3", api_token="ollama"),
            input_format="text",
            extraction_type="schema_from_instruction",
            instruction="Determine the sentiment of this text."
        )
    except:
        print("Ollama not available, skipping plain text input test.")
        return

    sample_text = "Crawl4ai is an amazing library for web scraping and data extraction!"
    extracted_json = strategy.extract(url="http://dummy.com/text_page", html_content=sample_text) 
    # html_content parameter is used for any text-based input, despite its name
    
    print("Extraction from Plain Text (mocked LLM):")
    if extracted_json:
        print(json.dumps(json.loads(extracted_json), indent=2))

if __name__ == "__main__":
    extract_from_plain_text()
```
---

### 3.8. Forcing JSON Response (`force_json_response`)

#### 3.8.1. Example: Using `force_json_response=True` with `extraction_type="schema"` or `"schema_from_instruction"`.
This is particularly useful with LLMs that might not strictly adhere to JSON output, or when using providers that support JSON mode.

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig
from pydantic import BaseModel
from unittest.mock import patch, MagicMock
import json

class UserProfile(BaseModel):
    username: str
    email: str

# Mock LLM: simulate it trying to return JSON but maybe with extra text
# if force_json_response was False. With True, it should ensure clean JSON.
mock_llm_force_json = MagicMock()
mock_llm_force_json.choices = [MagicMock()]
# LiteLLM's JSON mode (which force_json_response=True often enables)
# typically ensures the LLM's output is directly the JSON object string.
mock_llm_force_json.choices[0].message.content = json.dumps(
    {"username": "testuser", "email": "test@example.com"}
)
mock_llm_force_json.usage = MagicMock(completion_tokens=15, prompt_tokens=70, total_tokens=85)
mock_llm_force_json.usage.completion_tokens_details = {}; mock_llm_force_json.usage.prompt_tokens_details = {}


@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', return_value=mock_llm_force_json)
def force_json_response_example(mock_perform_completion):
    try:
        # Note: Some providers/models have better native JSON mode support.
        # OpenAI models often benefit from this.
        strategy = LLMExtractionStrategy(
            llm_config=LLMConfig(provider="openai/gpt-3.5-turbo", api_token=os.getenv("OPENAI_API_KEY","mock_key")), # Using OpenAI example
            schema=UserProfile.model_json_schema(),
            extraction_type="schema",
            force_json_response=True # Enable JSON mode
        )
        if not os.getenv("OPENAI_API_KEY"):
            print("Warning: OPENAI_API_KEY not set. Mocking will proceed, but real behavior might differ.")

    except:
        print("LLM provider not available, skipping force_json_response test.")
        return

    sample_content = "User: testuser, Email: test@example.com"
    extracted_json_string = strategy.extract(url="http://dummy.com/user", html_content=sample_content)
    
    print("Force JSON Response Example (mocked LLM):")
    if extracted_json_string:
        print(f"Raw output from LLM (should be clean JSON string): {extracted_json_string}")
        try:
            extracted_data = json.loads(extracted_json_string)
            print("Parsed data:", json.dumps(extracted_data, indent=2))
            UserProfile(**extracted_data) # Validate
            print("Successfully parsed and validated JSON.")
        except json.JSONDecodeError as e:
            print(f"Failed to parse JSON even with force_json_response: {e}")
            print("This might indicate an issue with the LLM's JSON mode or the mock setup.")
    else:
        print("No data extracted.")
    
    # Check if the 'response_format' was passed to litellm
    # This depends on the internal implementation detail of how force_json_response is passed.
    # Assuming it sets 'response_format': {'type': 'json_object'} in extra_args for litellm.
    # mock_perform_completion.assert_called_once()
    # call_kwargs = mock_perform_completion.call_args.kwargs
    # assert call_kwargs.get("extra_args", {}).get("response_format") == {"type": "json_object"}
    # print("LLM call included JSON response format.")


if __name__ == "__main__":
    force_json_response_example()
```

#### 3.8.2. Example: Comparing LLM output with and without `force_json_response=True` to show its effect on non-JSON-compliant LLMs.
This example requires an LLM that is known to sometimes produce non-JSON output or a more sophisticated mock.

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig
from pydantic import BaseModel
from unittest.mock import patch, MagicMock
import json
import os

class SimpleData(BaseModel):
    key: str

# Mock 1: LLM returns non-JSON compliant string
mock_llm_non_json = MagicMock()
mock_llm_non_json.choices = [MagicMock()]
mock_llm_non_json.choices[0].message.content = "Here is the JSON you asked for: ```json\n{\"key\": \"value_one\"}\n``` Some extra text."
mock_llm_non_json.usage = MagicMock(completion_tokens=30, prompt_tokens=80, total_tokens=110)
mock_llm_non_json.usage.completion_tokens_details = {}; mock_llm_non_json.usage.prompt_tokens_details = {}


# Mock 2: LLM returns clean JSON (as if force_json_response worked)
mock_llm_forced_json = MagicMock()
mock_llm_forced_json.choices = [MagicMock()]
mock_llm_forced_json.choices[0].message.content = json.dumps({"key": "value_one"})
mock_llm_forced_json.usage = MagicMock(completion_tokens=10, prompt_tokens=80, total_tokens=90)
mock_llm_forced_json.usage.completion_tokens_details = {}; mock_llm_forced_json.usage.prompt_tokens_details = {}


@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff')
def compare_force_json_response(mock_perform_completion):
    sample_content = "The key is value_one."
    schema_def = SimpleData.model_json_schema()
    
    try:
        # Using a provider that might benefit from force_json_response
        llm_config_for_comparison = LLMConfig(provider="openai/gpt-3.5-turbo", api_token=os.getenv("OPENAI_API_KEY","mock_key_compare"))
        if not os.getenv("OPENAI_API_KEY"):
            print("Warning: OPENAI_API_KEY not set for comparison. Mocking will show intended difference.")
    except:
        print("LLM provider not available, skipping force_json comparison test.")
        return

    # Case 1: force_json_response = False (default)
    mock_perform_completion.return_value = mock_llm_non_json
    strategy_no_force = LLMExtractionStrategy(
        llm_config=llm_config_for_comparison,
        schema=schema_def,
        extraction_type="schema",
        force_json_response=False
    )
    print("--- Without force_json_response ---")
    result_no_force_json_str = strategy_no_force.extract("url", sample_content)
    print(f"Raw output: {result_no_force_json_str}")
    try:
        data_no_force = json.loads(result_no_force_json_str) # This would likely fail with the mock
        print(f"Parsed data: {data_no_force}")
    except json.JSONDecodeError as e:
        print(f"Failed to parse as JSON (expected for this mock): {e}")

    # Case 2: force_json_response = True
    mock_perform_completion.return_value = mock_llm_forced_json
    strategy_with_force = LLMExtractionStrategy(
        llm_config=llm_config_for_comparison,
        schema=schema_def,
        extraction_type="schema",
        force_json_response=True
    )
    print("\n--- With force_json_response = True ---")
    result_with_force_json_str = strategy_with_force.extract("url", sample_content)
    print(f"Raw output: {result_with_force_json_str}")
    try:
        data_with_force = json.loads(result_with_force_json_str)
        SimpleData(**data_with_force) # Validate
        print(f"Parsed data: {data_with_force} (Successfully parsed and validated)")
    except json.JSONDecodeError as e:
        print(f"Failed to parse as JSON (unexpected with good JSON mode): {e}")

if __name__ == "__main__":
    compare_force_json_response()
```
---

### 3.9. Verbosity and Logging

#### 3.9.1. Example: Using `verbose=True` to see detailed LLM interaction logs.
Setting `verbose=True` in `LLMExtractionStrategy` enables detailed logging of prompts sent to and responses received from the LLM.

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig, DefaultLogger
from unittest.mock import patch, MagicMock
import json
import io
import sys

# Mock LLM response
mock_llm_verbose = MagicMock()
mock_llm_verbose.choices = [MagicMock(message=MagicMock(content=json.dumps({"data": "verbose example"})))]
mock_llm_verbose.usage = MagicMock(completion_tokens=5, prompt_tokens=10, total_tokens=15)
mock_llm_verbose.usage.completion_tokens_details = {}; mock_llm_verbose.usage.prompt_tokens_details = {}


@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', return_value=mock_llm_verbose)
def verbose_logging_example(mock_perform_completion):
    # Capture stdout to check for verbose logs
    old_stdout = sys.stdout
    sys.stdout = captured_output = io.StringIO()

    try:
        # Use a simple logger for this example that prints to stdout
        logger = DefaultLogger(verbose=True) 
        
        strategy = LLMExtractionStrategy(
            llm_config=LLMConfig(provider="ollama/llama3", api_token="ollama"),
            verbose=True, # Enable verbose logging in the strategy
            logger=logger,  # Pass the logger
            extraction_type="schema_from_instruction",
            instruction="Extract something."
        )
    except: # Fallback if ollama/logger setup fails for some reason in test env
        sys.stdout = old_stdout
        print("Ollama/Logger not available, skipping verbose logging test.")
        return


    strategy.extract(url="http://dummy.com/verbose", html_content="Some sample content.")
    
    sys.stdout = old_stdout # Restore stdout
    output_log = captured_output.getvalue()
    
    print("\n--- Captured Verbose Log Output (should contain LLM prompt/response details) ---")
    print(output_log)

    # Check for typical verbose log messages (actual messages might vary)
    assert "LLM Request" in output_log or "Prompt for LLM" in output_log
    assert "LLM Response" in output_log or "Response from LLM" in output_log
    print("\nVerbose logging appeared to work.")

if __name__ == "__main__":
    verbose_logging_example()
```

#### 3.9.2. Example: Providing a custom `logger` instance to `LLMExtractionStrategy`.
You can integrate `LLMExtractionStrategy` with your existing logging setup.

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig, DefaultLogger
import logging
import io

# Setup a custom Python logger
custom_logger = logging.getLogger("MyCustomExtractorLogger")
custom_logger.setLevel(logging.INFO)
log_capture_string = io.StringIO()
ch = logging.StreamHandler(log_capture_string)
ch.setLevel(logging.INFO)
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
ch.setFormatter(formatter)
custom_logger.addHandler(ch)
custom_logger.propagate = False # Prevent duplicate logs if root logger also has a handler

# For LLMExtractionStrategy, we need to wrap this in a Crawl4ai compatible logger
class CustomCrawl4aiLogger(DefaultLogger):
    def __init__(self, py_logger, verbose=False):
        super().__init__(verbose=verbose)
        self.py_logger = py_logger

    def _log(self, level_str, message, tag=None, params=None, colors=None):
        # You can customize how messages are formatted and logged here
        log_message = f"[{tag or 'C4AI'}] {message}"
        if params:
            log_message = log_message.format(**params)
        
        if level_str.lower() == "info":
            self.py_logger.info(log_message)
        elif level_str.lower() == "error":
            self.py_logger.error(log_message)
        elif level_str.lower() == "warning":
            self.py_logger.warning(log_message)
        elif self.verbose and level_str.lower() == "debug": # Only log debug if verbose
             self.py_logger.debug(log_message)


# Mock the LLM call for this example to focus on logging
from unittest.mock import patch, MagicMock
import json
mock_llm_custom_log = MagicMock()
mock_llm_custom_log.choices = [MagicMock(message=MagicMock(content=json.dumps({"info":"logged"})))]
mock_llm_custom_log.usage = MagicMock(completion_tokens=3, prompt_tokens=10, total_tokens=13)
mock_llm_custom_log.usage.completion_tokens_details = {}; mock_llm_custom_log.usage.prompt_tokens_details = {}


@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', return_value=mock_llm_custom_log)
def custom_logger_example(mock_perform_completion):
    crawl4ai_custom_logger = CustomCrawl4aiLogger(custom_logger, verbose=True)
    
    try:
        strategy = LLMExtractionStrategy(
            llm_config=LLMConfig(provider="ollama/llama3", api_token="ollama"),
            logger=crawl4ai_custom_logger, # Pass the custom logger instance
            verbose=True, # Ensure strategy attempts to log debug messages too
            extraction_type="schema_from_instruction",
            instruction="Log this."
        )
    except:
        print("Ollama not available, skipping custom logger test.")
        return

    strategy.extract(url="http://dummy.com/custom_log", html_content="Content for custom logger.")
    
    log_contents = log_capture_string.getvalue()
    print("\n--- Captured Log Output (via custom Python logger) ---")
    print(log_contents)
    
    assert "MyCustomExtractorLogger" in log_contents # Check if our logger's name is in output
    assert "[LLM_REQ]" in log_contents or "[LLM_RESP]" in log_contents # Check for common strategy tags

if __name__ == "__main__":
    custom_logger_example()
```
---

### 3.10. Practical Extraction Scenarios
These examples use `AsyncWebCrawler` and might require actual internet access and potentially API keys for the LLMs. They will be mocked for consistency in testing, but the setup shows real-world usage.

#### 3.10.1. Example: Extracting product names, prices, and descriptions from an e-commerce page.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, LLMConfig
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from pydantic import BaseModel, Field
from typing import List, Optional
from unittest.mock import patch, MagicMock
import json
import os

class ProductInfo(BaseModel):
    name: str = Field(..., description="The name of the product")
    price: Optional[float] = Field(None, description="The price of the product, as a float")
    description_snippet: Optional[str] = Field(None, description="A short snippet of the product description")

class ProductPageExtract(BaseModel):
    products: List[ProductInfo] = Field(description="List of products found on the page")

# Mock the LLM call
mock_ecommerce_response = MagicMock()
mock_ecommerce_response.choices = [MagicMock()]
mock_ecommerce_response.choices[0].message.content = json.dumps({
    "products": [
        {"name": "Super Widget X1000", "price": 99.99, "description_snippet": "The best widget ever."},
        {"name": "Basic Widget B50", "price": 19.99, "description_snippet": "A simple, reliable widget."}
    ]
})
mock_ecommerce_response.usage = MagicMock(completion_tokens=50, prompt_tokens=300, total_tokens=350)
mock_ecommerce_response.usage.completion_tokens_details = {}; mock_ecommerce_response.usage.prompt_tokens_details = {}


@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', return_value=mock_ecommerce_response)
async def extract_ecommerce_products(mock_perform_completion):
    # This URL is a placeholder; a real e-commerce page would be used.
    # For CI/testing, we use a simple example.com which won't have products.
    # The key is to show the setup.
    ecommerce_url = "http://example.com" 
    
    try:
        llm_conf = LLMConfig(provider="openai/gpt-4o-mini", api_token=os.getenv("OPENAI_API_KEY", "mock_key_ecommerce"))
        if not os.getenv("OPENAI_API_KEY"): print("Warning: OPENAI_API_KEY not set. Mock will be used.")
        
        extraction_strat = LLMExtractionStrategy(
            llm_config=llm_conf,
            schema=ProductPageExtract.model_json_schema(),
            extraction_type="schema",
            instruction="Extract all product names, their prices, and a short description snippet from the page content."
        )
    except Exception as e:
        print(f"LLM setup failed for e-commerce example: {e}. Skipping.")
        return

    run_config = CrawlerRunConfig(
        extraction_strategy=extraction_strat,
        # word_count_threshold=5 # Lower for example.com if testing live
    )

    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(url=ecommerce_url, config=run_config)

    print(f"--- Extraction from E-commerce like page ({ecommerce_url}) ---")
    if result.success and result.extracted_content:
        extracted_data = json.loads(result.extracted_content)
        print(json.dumps(extracted_data, indent=2))
        
        # Validate with Pydantic
        page_data = ProductPageExtract(**extracted_data)
        for product in page_data.products:
            print(f"Product: {product.name}, Price: {product.price}")
    elif not result.success:
        print(f"Crawl failed: {result.error_message}")
    else:
        print("No structured data extracted or extraction failed.")
    
    assert mock_perform_completion.called

if __name__ == "__main__":
    asyncio.run(extract_ecommerce_products())
```

#### 3.10.2. Example: Extracting article headlines, authors, and publication dates from a news site.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, LLMConfig
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from pydantic import BaseModel, Field
from typing import Optional
from unittest.mock import patch, MagicMock
import json
import os

class NewsArticle(BaseModel):
    headline: str = Field(..., description="The main headline of the news article")
    author: Optional[str] = Field(None, description="The author(s) of the article")
    publication_date: Optional[str] = Field(None, description="The date the article was published (e.g., YYYY-MM-DD)")

# Mock the LLM call
mock_news_response = MagicMock()
mock_news_response.choices = [MagicMock()]
mock_news_response.choices[0].message.content = json.dumps({
    "headline": "AI Breakthrough Announced", 
    "author": "Reporter Bot", 
    "publication_date": "2024-05-24"
})
mock_news_response.usage = MagicMock(completion_tokens=30, prompt_tokens=250, total_tokens=280)
mock_news_response.usage.completion_tokens_details = {}; mock_news_response.usage.prompt_tokens_details = {}

@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', return_value=mock_news_response)
async def extract_news_article_details(mock_perform_completion):
    # Using Wikipedia for a stable, public news-like article structure
    news_url = "https://en.wikipedia.org/wiki/Artificial_intelligence" 
    
    try:
        llm_conf = LLMConfig(provider="openai/gpt-4o-mini", api_token=os.getenv("OPENAI_API_KEY", "mock_key_news"))
        if not os.getenv("OPENAI_API_KEY"): print("Warning: OPENAI_API_KEY not set. Mock will be used.")

        extraction_strat = LLMExtractionStrategy(
            llm_config=llm_conf,
            schema=NewsArticle.model_json_schema(),
            extraction_type="schema",
            instruction="From the provided news article content, extract the main headline, the author(s), and the publication date."
        )
    except Exception as e:
        print(f"LLM setup failed for news example: {e}. Skipping.")
        return


    run_config = CrawlerRunConfig(extraction_strategy=extraction_strat)

    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(url=news_url, config=run_config)

    print(f"--- Extraction from News Article ({news_url}) ---")
    if result.success and result.extracted_content:
        extracted_data = json.loads(result.extracted_content)
        print(json.dumps(extracted_data, indent=2))
        article_data = NewsArticle(**extracted_data)
        print(f"Headline: {article_data.headline}")
    elif not result.success:
        print(f"Crawl failed: {result.error_message}")
    else:
        print("No structured data extracted or extraction failed.")
    
    assert mock_perform_completion.called

if __name__ == "__main__":
    asyncio.run(extract_news_article_details())
```

#### 3.10.3. Example: Extracting frequently asked questions (FAQs) and their answers from a support page.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, LLMConfig
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from pydantic import BaseModel, Field
from typing import List
from unittest.mock import patch, MagicMock
import json
import os

class FAQItem(BaseModel):
    question: str
    answer: str

class FAQPage(BaseModel):
    faqs: List[FAQItem]

# Mock the LLM call
mock_faq_response = MagicMock()
mock_faq_response.choices = [MagicMock()]
mock_faq_response.choices[0].message.content = json.dumps({
    "faqs": [
        {"question": "What is Crawl4ai?", "answer": "An awesome web crawler."},
        {"question": "How to install?", "answer": "pip install crawl4ai"}
    ]
})
mock_faq_response.usage = MagicMock(completion_tokens=60, prompt_tokens=300, total_tokens=360)
mock_faq_response.usage.completion_tokens_details = {}; mock_faq_response.usage.prompt_tokens_details = {}


@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', return_value=mock_faq_response)
async def extract_faqs(mock_perform_completion):
    # Placeholder URL - a real FAQ page would be used
    faq_url = "http://example.com/faq" 

    try:
        llm_conf = LLMConfig(provider="openai/gpt-4o-mini", api_token=os.getenv("OPENAI_API_KEY", "mock_key_faq"))
        if not os.getenv("OPENAI_API_KEY"): print("Warning: OPENAI_API_KEY not set. Mock will be used.")
        
        extraction_strat = LLMExtractionStrategy(
            llm_config=llm_conf,
            schema=FAQPage.model_json_schema(),
            extraction_type="schema",
            instruction="Extract all question and answer pairs from the FAQ section of this page."
        )
    except Exception as e:
        print(f"LLM setup failed for FAQ example: {e}. Skipping.")
        return

    run_config = CrawlerRunConfig(extraction_strategy=extraction_strat)

    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(url=faq_url, config=run_config)

    print(f"--- Extraction from FAQ Page ({faq_url}) ---")
    if result.success and result.extracted_content:
        extracted_data = json.loads(result.extracted_content)
        print(json.dumps(extracted_data, indent=2))
        faq_page_data = FAQPage(**extracted_data)
        for faq_item in faq_page_data.faqs:
            print(f"Q: {faq_item.question}\nA: {faq_item.answer}\n")
    elif not result.success:
        print(f"Crawl failed: {result.error_message}")
    else:
        print("No structured data extracted or extraction failed.")

    assert mock_perform_completion.called

if __name__ == "__main__":
    asyncio.run(extract_faqs())
```

#### 3.10.4. Example: Extracting contact information (email, phone, address) from a company's "Contact Us" page.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, LLMConfig
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from pydantic import BaseModel, Field
from typing import Optional
from unittest.mock import patch, MagicMock
import json
import os

class ContactInfo(BaseModel):
    email: Optional[str] = Field(None, description="Company contact email address")
    phone: Optional[str] = Field(None, description="Company contact phone number")
    address: Optional[str] = Field(None, description="Company physical address")

# Mock the LLM call
mock_contact_response = MagicMock()
mock_contact_response.choices = [MagicMock()]
mock_contact_response.choices[0].message.content = json.dumps({
    "email": "support@example.com", 
    "phone": "1-800-555-1234", 
    "address": "123 Main St, Anytown, USA"
})
mock_contact_response.usage = MagicMock(completion_tokens=40, prompt_tokens=200, total_tokens=240)
mock_contact_response.usage.completion_tokens_details = {}; mock_contact_response.usage.prompt_tokens_details = {}


@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', return_value=mock_contact_response)
async def extract_contact_info(mock_perform_completion):
    contact_url = "http://example.com/contact" 

    try:
        llm_conf = LLMConfig(provider="openai/gpt-4o-mini", api_token=os.getenv("OPENAI_API_KEY", "mock_key_contact"))
        if not os.getenv("OPENAI_API_KEY"): print("Warning: OPENAI_API_KEY not set. Mock will be used.")
        
        extraction_strat = LLMExtractionStrategy(
            llm_config=llm_conf,
            schema=ContactInfo.model_json_schema(),
            extraction_type="schema",
            instruction="Extract the primary email, phone number, and physical address from this contact page."
        )
    except Exception as e:
        print(f"LLM setup failed for contact info example: {e}. Skipping.")
        return

    run_config = CrawlerRunConfig(extraction_strategy=extraction_strat)

    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(url=contact_url, config=run_config)

    print(f"--- Extraction from Contact Page ({contact_url}) ---")
    if result.success and result.extracted_content:
        extracted_data = json.loads(result.extracted_content)
        print(json.dumps(extracted_data, indent=2))
        contact_data = ContactInfo(**extracted_data)
        print(f"Email: {contact_data.email}, Phone: {contact_data.phone}")
    elif not result.success:
        print(f"Crawl failed: {result.error_message}")
    else:
        print("No structured data extracted or extraction failed.")
    
    assert mock_perform_completion.called

if __name__ == "__main__":
    asyncio.run(extract_contact_info())
```

#### 3.10.5. Example: Extracting key entities (people, organizations, locations) from a block of text using `extraction_type="block"` and a specific instruction.
This uses "block" extraction but with an instruction to guide the LLM to tag specific entities.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, LLMConfig
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from unittest.mock import patch, MagicMock
import json
import os

# Mock LLM response - for block extraction with entity tagging
mock_entity_response = MagicMock()
mock_entity_response.choices = [MagicMock()]
mock_entity_response.choices[0].message.content = """
<blocks>
  <block>
    <content>Apple Inc. is headquartered in Cupertino.</content>
    <tags><tag>sentence</tag><tag>ORG:Apple Inc.</tag><tag>LOC:Cupertino</tag></tags>
  </block>
  <block>
    <content>Tim Cook is the CEO.</content>
    <tags><tag>sentence</tag><tag>PER:Tim Cook</tag></tags>
  </block>
</blocks>
"""
mock_entity_response.usage = MagicMock(completion_tokens=50, prompt_tokens=150, total_tokens=200)
mock_entity_response.usage.completion_tokens_details = {}; mock_entity_response.usage.prompt_tokens_details = {}


@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', return_value=mock_entity_response)
async def extract_entities_with_block(mock_perform_completion):
    entity_url = "http://example.com/about-us" # Placeholder
    
    try:
        llm_conf = LLMConfig(provider="openai/gpt-4o-mini", api_token=os.getenv("OPENAI_API_KEY", "mock_key_entities"))
        if not os.getenv("OPENAI_API_KEY"): print("Warning: OPENAI_API_KEY not set. Mock will be used.")
        
        extraction_strat = LLMExtractionStrategy(
            llm_config=llm_conf,
            extraction_type="block",
            instruction="Extract each sentence as a block. For each block, identify and add tags for People (PER:Name), Organizations (ORG:Name), and Locations (LOC:Name) found within that sentence."
        )
    except Exception as e:
        print(f"LLM setup failed for entity extraction example: {e}. Skipping.")
        return

    run_config = CrawlerRunConfig(extraction_strategy=extraction_strat)

    # For this demo, we'll use direct content instead of crawling a URL
    sample_text_for_entities = "Apple Inc. is headquartered in Cupertino. Tim Cook is the CEO."

    # Normally you'd use crawler.arun(url=..., config=...).
    # Here, we call the strategy directly for simplicity with local text.
    extracted_blocks = extraction_strat.extract(url=entity_url, html_content=sample_text_for_entities)


    print(f"--- Entity Extraction using extraction_type='block' ---")
    if extracted_blocks:
        print(json.dumps(extracted_blocks, indent=2))
        for block in extracted_blocks:
            print(f"Content: {block['content']}")
            print(f"  Tags: {block['tags']}")
    else:
        print("No blocks extracted or extraction failed.")
    
    assert mock_perform_completion.called

if __name__ == "__main__":
    asyncio.run(extract_entities_with_block())
```
---

## 4. Integration with `AsyncWebCrawler`

#### 4.1. Example: Basic `AsyncWebCrawler` run with `LLMExtractionStrategy` configured in `CrawlerRunConfig`.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, LLMConfig
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from pydantic import BaseModel
from unittest.mock import patch, MagicMock
import json
import os

class PageSummary(BaseModel):
    summary: str
    keywords: List[str]

mock_llm_crawler_run = MagicMock()
mock_llm_crawler_run.choices = [MagicMock()]
mock_llm_crawler_run.choices[0].message.content = json.dumps({
    "summary": "Example.com is a domain for use in illustrative examples.",
    "keywords": ["example", "domain", "documentation"]
})
mock_llm_crawler_run.usage = MagicMock(completion_tokens=30, prompt_tokens=100, total_tokens=130)
mock_llm_crawler_run.usage.completion_tokens_details = {}; mock_llm_crawler_run.usage.prompt_tokens_details = {}


@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', return_value=mock_llm_crawler_run)
async def crawler_with_llm_extraction(mock_perform_completion):
    try:
        llm_conf = LLMConfig(provider="openai/gpt-3.5-turbo", api_token=os.getenv("OPENAI_API_KEY", "mock_key_crawler"))
        if not os.getenv("OPENAI_API_KEY"): print("Warning: OPENAI_API_KEY not set. Mock will be used.")

        extraction_strat = LLMExtractionStrategy(
            llm_config=llm_conf,
            schema=PageSummary.model_json_schema(),
            extraction_type="schema",
            instruction="Provide a one-sentence summary of the page and list up to 3 main keywords."
        )
    except Exception as e:
        print(f"LLM setup failed for crawler integration example: {e}. Skipping.")
        return

    run_config = CrawlerRunConfig(
        extraction_strategy=extraction_strat,
        word_count_threshold=5 # Ensure example.com content is processed
    )

    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(url="http://example.com", config=run_config)

    print(f"--- Crawler run with LLMExtractionStrategy ---")
    if result.success and result.extracted_content:
        extracted_data = json.loads(result.extracted_content)
        print("Extracted Data:")
        print(json.dumps(extracted_data, indent=2))
        
        summary_instance = PageSummary(**extracted_data)
        print(f"\nValidated Summary: {summary_instance.summary}")
        print(f"Keywords: {summary_instance.keywords}")
    elif not result.success:
        print(f"Crawl failed: {result.error_message}")
    else:
        print("No structured data extracted.")
    
    assert mock_perform_completion.called

if __name__ == "__main__":
    asyncio.run(crawler_with_llm_extraction())
```

#### 4.2. Example: `AsyncWebCrawler` processing multiple URLs, each with the same `LLMExtractionStrategy`.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, LLMConfig
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from pydantic import BaseModel
from unittest.mock import patch, MagicMock
import json
import os

class SiteInfo(BaseModel):
    site_name: str
    main_purpose: str

# Mock LLM to return different info based on URL (simplified by just returning same mock structure)
mock_llm_multi_url = MagicMock()
mock_llm_multi_url.choices = [MagicMock()]
# We'll have the mock return slightly different content for each call
responses_for_multi_url = [
    json.dumps({"site_name": "Example Domain", "main_purpose": "Illustrative examples"}),
    json.dumps({"site_name": "IANA", "main_purpose": "Managing global IP addressing"})
]
call_count_multi = 0
def side_effect_multi_url(*args, **kwargs):
    global call_count_multi
    mock_llm_multi_url.choices[0].message.content = responses_for_multi_url[call_count_multi % len(responses_for_multi_url)]
    call_count_multi +=1
    return mock_llm_multi_url

mock_llm_multi_url.usage = MagicMock(completion_tokens=20, prompt_tokens=80, total_tokens=100) # Generic usage
mock_llm_multi_url.usage.completion_tokens_details = {}; mock_llm_multi_url.usage.prompt_tokens_details = {}

@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', side_effect=side_effect_multi_url)
async def crawler_many_with_llm_extraction(mock_perform_completion):
    urls_to_crawl = [
        "http://example.com",
        "https://www.iana.org/domains/reserved" # Another simple, stable page
    ]
    
    try:
        llm_conf = LLMConfig(provider="openai/gpt-3.5-turbo", api_token=os.getenv("OPENAI_API_KEY", "mock_key_many"))
        if not os.getenv("OPENAI_API_KEY"): print("Warning: OPENAI_API_KEY not set. Mock will be used.")

        extraction_strat = LLMExtractionStrategy(
            llm_config=llm_conf,
            schema=SiteInfo.model_json_schema(),
            extraction_type="schema",
            instruction="Identify the site name and its main purpose."
        )
    except Exception as e:
        print(f"LLM setup failed for arun_many example: {e}. Skipping.")
        return

    run_config = CrawlerRunConfig(
        extraction_strategy=extraction_strat,
        word_count_threshold=10 # Adjust as needed for the test URLs
    )

    async with AsyncWebCrawler() as crawler:
        # arun_many returns an async generator if stream=True, or list if stream=False (default)
        results = await crawler.arun_many(urls=urls_to_crawl, config=run_config) 

    print(f"--- Crawler arun_many with LLMExtractionStrategy ---")
    for result in results:
        if result.success and result.extracted_content:
            extracted_data = json.loads(result.extracted_content)
            print(f"\nURL: {result.url}")
            print("Extracted Data:")
            print(json.dumps(extracted_data, indent=2))
        elif not result.success:
            print(f"\nCrawl for {result.url} failed: {result.error_message}")
        else:
            print(f"\nNo structured data extracted for {result.url}.")
    
    assert mock_perform_completion.call_count == len(urls_to_crawl)

if __name__ == "__main__":
    asyncio.run(crawler_many_with_llm_extraction())
```

#### 4.3. Example: `AsyncWebCrawler` where `CrawlerRunConfig` is dynamically changed per URL to use different extraction schemas or instructions.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, LLMConfig
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from pydantic import BaseModel, Field
from typing import List, Optional, Dict, Any
from unittest.mock import patch, MagicMock
import json
import os

class TechArticleInfo(BaseModel):
    title: str
    primary_topic: str

class CompanyInfo(BaseModel):
    company_name: str
    services_offered: List[str]

# Mocks for different schemas
mock_tech_article_response = MagicMock()
mock_tech_article_response.choices = [MagicMock(message=MagicMock(content=json.dumps({"title": "Intro to Crawling", "primary_topic": "Web Scraping"})))]
mock_tech_article_response.usage = MagicMock(completion_tokens=15, prompt_tokens=70, total_tokens=85)
mock_tech_article_response.usage.completion_tokens_details = {}; mock_tech_article_response.usage.prompt_tokens_details = {}


mock_company_response = MagicMock()
mock_company_response.choices = [MagicMock(message=MagicMock(content=json.dumps({"company_name": "Example Corp", "services_offered": ["Web Hosting", "Domain Registration"]})))]
mock_company_response.usage = MagicMock(completion_tokens=20, prompt_tokens=90, total_tokens=110)
mock_company_response.usage.completion_tokens_details = {}; mock_company_response.usage.prompt_tokens_details = {}


# Side effect function to return different mocks based on schema/instruction
def dynamic_llm_side_effect(*args, **kwargs):
    # Heuristic: check prompt content for clues about which schema is expected
    prompt_content = args[1] # The prompt string is the second argument to perform_completion_with_backoff
    if "TechArticleInfo" in prompt_content or "primary_topic" in prompt_content:
        return mock_tech_article_response
    elif "CompanyInfo" in prompt_content or "services_offered" in prompt_content:
        return mock_company_response
    return MagicMock() # Default generic mock if no match

@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', side_effect=dynamic_llm_side_effect)
async def crawler_dynamic_configs(mock_perform_completion):
    urls_and_configs: List[Dict[str, Any]] = [
        {
            "url": "https://en.wikipedia.org/wiki/Web_scraping", # Tech article like
            "schema_model": TechArticleInfo,
            "instruction": "Extract title and primary topic of this technical article."
        },
        {
            "url": "http://example.com", # Generic company like
            "schema_model": CompanyInfo,
            "instruction": "Identify the company name and list its main services."
        }
    ]
    
    try:
        llm_conf = LLMConfig(provider="openai/gpt-3.5-turbo", api_token=os.getenv("OPENAI_API_KEY", "mock_key_dynamic"))
        if not os.getenv("OPENAI_API_KEY"): print("Warning: OPENAI_API_KEY not set. Mock will be used.")
    except Exception as e:
        print(f"LLM setup failed for dynamic config example: {e}. Skipping.")
        return

    all_results_data = []

    async with AsyncWebCrawler() as crawler:
        for item in urls_and_configs:
            current_url = item["url"]
            CurrentSchemaModel = item["schema_model"]
            current_instruction = item["instruction"]

            extraction_strat = LLMExtractionStrategy(
                llm_config=llm_conf,
                schema=CurrentSchemaModel.model_json_schema(),
                extraction_type="schema",
                instruction=current_instruction
            )
            run_config = CrawlerRunConfig(
                extraction_strategy=extraction_strat, 
                word_count_threshold=10
            )
            
            print(f"\nCrawling {current_url} with schema {CurrentSchemaModel.__name__}...")
            result = await crawler.arun(url=current_url, config=run_config)
            
            if result.success and result.extracted_content:
                extracted_data = json.loads(result.extracted_content)
                print(f"Extracted for {current_url}:")
                print(json.dumps(extracted_data, indent=2))
                all_results_data.append(extracted_data)
            else:
                print(f"Failed or no extraction for {current_url}: {result.error_message}")
                all_results_data.append({"error": result.error_message, "url": current_url})
                
    assert mock_perform_completion.call_count == len(urls_and_configs)
    assert all_results_data[0].get("primary_topic") == "Web Scraping"
    assert "Web Hosting" in all_results_data[1].get("services_offered", [])


if __name__ == "__main__":
    asyncio.run(crawler_dynamic_configs())
```
---

## 5. Combining Extraction with Other Crawl4ai Features

### 5.1. **Extraction from PDF Content:**

#### 5.1.1. Example: Using `PDFCrawerStrategy` and `PDFContentScrapingStrategy` to get HTML/text from a PDF, then using `LLMExtractionStrategy` on that content.
This example requires `crawl4ai[pdf]` to be installed. We'll use a public PDF URL.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, LLMConfig, BrowserConfig
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.content_scraping_strategy import PDFContentScrapingStrategy # For PDF processing
from crawl4ai.async_crawler_strategy import PDFCrawlerStrategy # To handle PDF URLs
from pydantic import BaseModel, Field
from typing import List, Optional
from unittest.mock import patch, MagicMock
import json
import os

# Note: This example requires PyPDF2. Install with: pip install crawl4ai[pdf]

class PDFSummary(BaseModel):
    title: Optional[str] = Field(None, description="The title of the PDF document, if discernible.")
    first_paragraph_summary: str = Field(..., description="A brief summary of the first main paragraph of text.")

# Mock for LLM
mock_llm_pdf_extract = MagicMock()
mock_llm_pdf_extract.choices = [MagicMock(message=MagicMock(content=json.dumps({
    "title": "Sample PDF Document",
    "first_paragraph_summary": "This PDF discusses important topics related to data."
})))]
mock_llm_pdf_extract.usage = MagicMock(completion_tokens=20, prompt_tokens=150, total_tokens=170)
mock_llm_pdf_extract.usage.completion_tokens_details = {}; mock_llm_pdf_extract.usage.prompt_tokens_details = {}

@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', return_value=mock_llm_pdf_extract)
async def extract_from_pdf_content(mock_perform_completion):
    # A public, simple PDF for testing. (e.g., a small, text-based PDF)
    # Using a known, stable PDF URL: an RFC document (plain text heavy)
    pdf_url = "https://www.rfc-editor.org/rfc/rfc2616.txt.pdf" # This is a PDF version of an RFC text file
    # pdf_url = "https://www.w3.org/WAI/ER/tests/xhtml/testfiles/resources/pdf/dummy.pdf" # A very simple dummy PDF

    try:
        llm_conf = LLMConfig(provider="openai/gpt-3.5-turbo", api_token=os.getenv("OPENAI_API_KEY", "mock_key_pdf"))
        if not os.getenv("OPENAI_API_KEY"): print("Warning: OPENAI_API_KEY not set for PDF example. Mock will be used.")

        extraction_strat = LLMExtractionStrategy(
            llm_config=llm_conf,
            schema=PDFSummary.model_json_schema(),
            extraction_type="schema",
            instruction="From the provided PDF text, extract the document title if available, and summarize the first main paragraph.",
            input_format="text" # PDFContentScrapingStrategy will provide text
        )
    except Exception as e:
        print(f"LLM setup failed for PDF extraction example: {e}. Skipping.")
        return

    # BrowserConfig might be needed if the PDF is rendered in-browser and not a direct link
    browser_cfg = BrowserConfig() 

    # CrawlerRunConfig for PDF processing and then LLM extraction
    run_config = CrawlerRunConfig(
        extraction_strategy=extraction_strat,
        # The PDFContentScrapingStrategy will be used by PDFCrawlerStrategy
        # to convert PDF to text/HTML for the LLMExtractionStrategy.
        # LLMExtractionStrategy expects text if input_format="text".
        scraping_strategy=PDFContentScrapingStrategy(output_format="text") # Ensure text output for LLM
    )
    
    # Use PDFCrawlerStrategy to handle the PDF URL
    # Note: For PDFCrawlerStrategy, the 'browser_config' of AsyncWebCrawler is not directly used for PDF fetching.
    # PDFCrawlerStrategy uses requests library directly.
    async with AsyncWebCrawler(crawler_strategy=PDFCrawlerStrategy(), browser_config=browser_cfg) as crawler:
        print(f"Crawling PDF: {pdf_url}")
        result = await crawler.arun(url=pdf_url, config=run_config)

    print(f"--- Extraction from PDF Content ({pdf_url}) ---")
    if result.success:
        if result.extracted_content:
            extracted_data = json.loads(result.extracted_content)
            print("Extracted Data from PDF:")
            print(json.dumps(extracted_data, indent=2))
            pdf_summary_instance = PDFSummary(**extracted_data)
            print(f"\nValidated Title: {pdf_summary_instance.title}")
            print(f"Summary: {pdf_summary_instance.first_paragraph_summary}")
        else:
            print("PDF content processed, but no structured data extracted by LLM.")
            print(f"Raw Markdown/Text from PDF (first 300 chars): {result.markdown.raw_markdown[:300] if result.markdown else 'N/A'}...")
    else:
        print(f"Crawl/Processing of PDF failed: {result.error_message}")
    
    if extraction_strat.llm_config.api_token != "mock_key_pdf": # Only assert if not fully mocked
         assert mock_perform_completion.called

if __name__ == "__main__":
    # This example needs `pip install crawl4ai[pdf]`
    try:
        import PyPDF2 # Check if PyPDF2 is installed
        asyncio.run(extract_from_pdf_content())
    except ImportError:
        print("PyPDF2 not found. Please install it with `pip install crawl4ai[pdf]` to run this example.")
    except Exception as e:
        print(f"An error occurred: {e}")
```

### 5.2. **Extraction After Content Filtering (Illustrative):**

#### 5.2.1. Example: Manually running a `ContentFilterStrategy` on HTML, then passing the filtered HTML to `LLMExtractionStrategy.extract(input_format="html")`.

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.content_filter_strategy import PruningContentFilter # Example filter
from crawl4ai.utils import LLMConfig
from bs4 import BeautifulSoup
from unittest.mock import patch, MagicMock
import json

# Mock for LLM
mock_llm_filtered_html = MagicMock()
mock_llm_filtered_html.choices = [MagicMock(message=MagicMock(content=json.dumps({"main_idea": "The core idea is about X."})))]
mock_llm_filtered_html.usage = MagicMock(completion_tokens=10, prompt_tokens=40, total_tokens=50)
mock_llm_filtered_html.usage.completion_tokens_details = {}; mock_llm_filtered_html.usage.prompt_tokens_details = {}


@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', return_value=mock_llm_filtered_html)
def extract_after_manual_filter(mock_perform_completion):
    sample_raw_html = """
    <html><head><title>Test Page</title></head><body>
        <header>Site Navigation Links...</header>
        <main id='content'>
            <h1>Article Title</h1>
            <p>This is the first important paragraph.</p>
            <div class='ad-banner'>Advertisement here</div>
            <p>This is the second important paragraph after an ad.</p>
        </main>
        <footer>Copyright 2024. All rights reserved.</footer>
    </body></html>
    """
    
    # Step 1: Manually apply a content filter
    # PruningContentFilter modifies the soup in-place
    soup = BeautifulSoup(sample_raw_html, "lxml")
    
    # Example of direct filter usage (conceptual, PruningContentFilter works on soup)
    # PruningContentFilter might typically be used within MarkdownGenerator
    # For this example, let's simulate its effect by selecting main content
    # and removing a known ad-like element.
    
    # Simulate pruning effect:
    header = soup.find("header")
    if header: header.decompose()
    footer = soup.find("footer")
    if footer: footer.decompose()
    ad_banner = soup.find("div", class_="ad-banner")
    if ad_banner: ad_banner.decompose()
        
    filtered_html_content = str(soup.find("main")) # Get HTML of the main content after pruning
    
    print(f"--- Filtered HTML (simulated) ---\n{filtered_html_content}\n---------------------------------")

    # Step 2: Pass filtered HTML to LLMExtractionStrategy
    try:
        strategy = LLMExtractionStrategy(
            llm_config=LLMConfig(provider="ollama/llama3", api_token="ollama"),
            input_format="html", # The filtered content is still HTML
            extraction_type="schema_from_instruction",
            instruction="What is the main idea of this content?"
        )
    except:
        print("Ollama not available, skipping manual filter extraction test.")
        return

    extracted_json = strategy.extract(url="http://dummy.com/filtered", html_content=filtered_html_content)
    
    print("\nExtraction from Manually Filtered HTML (mocked LLM):")
    if extracted_json:
        print(json.dumps(json.loads(extracted_json), indent=2))
    
    assert mock_perform_completion.called

if __name__ == "__main__":
    extract_after_manual_filter()
```
---

## 6. Advanced Techniques and Edge Cases

#### 6.1. Example: Handling cases where the LLM fails to extract data or returns an unexpected format (and how `force_json_response` might help).

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig
from pydantic import BaseModel
from unittest.mock import patch, MagicMock
import json
import os

class SimpleSchema(BaseModel):
    name: str

# Mock 1: LLM returns malformed JSON (e.g., with extra text, missing comma)
mock_malformed_json_response = MagicMock()
mock_malformed_json_response.choices = [MagicMock(message=MagicMock(content='Sure, here is the JSON: {"name": "Test" // oops, a comment'))]
mock_malformed_json_response.usage = MagicMock(completion_tokens=10, prompt_tokens=50, total_tokens=60)
mock_malformed_json_response.usage.completion_tokens_details = {}; mock_malformed_json_response.usage.prompt_tokens_details = {}


# Mock 2: LLM returns clean JSON (simulating successful force_json_response)
mock_clean_json_response = MagicMock()
mock_clean_json_response.choices = [MagicMock(message=MagicMock(content=json.dumps({"name": "Test"})))]
mock_clean_json_response.usage = MagicMock(completion_tokens=8, prompt_tokens=50, total_tokens=58)
mock_clean_json_response.usage.completion_tokens_details = {}; mock_clean_json_response.usage.prompt_tokens_details = {}


@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff')
def handle_llm_failures(mock_perform_completion):
    sample_content = "The name is Test."
    schema_def = SimpleSchema.model_json_schema()
    
    try:
        llm_conf = LLMConfig(provider="openai/gpt-3.5-turbo", api_token=os.getenv("OPENAI_API_KEY", "mock_key_failure"))
        if not os.getenv("OPENAI_API_KEY"): print("Warning: OPENAI_API_KEY not set for failure handling example.")
    except:
        print("LLM provider setup failed. Skipping LLM failure handling test.")
        return

    # Scenario 1: LLM returns malformed JSON, force_json_response=False
    print("--- Scenario 1: Malformed JSON, force_json_response=False ---")
    mock_perform_completion.return_value = mock_malformed_json_response
    strategy_no_force = LLMExtractionStrategy(
        llm_config=llm_conf, schema=schema_def, extraction_type="schema", force_json_response=False
    )
    result_str_no_force = strategy_no_force.extract("url", sample_content)
    print(f"LLM raw output: {result_str_no_force}")
    try:
        # This should ideally fail or be handled by LiteLLM's built-in JSON parsing attempts
        parsed = json.loads(result_str_no_force) 
        SimpleSchema(**parsed) # Validate
        print(f"Parsed (unexpectedly successful for this mock): {parsed}")
    except Exception as e:
        print(f"Expected parsing/validation error: {e}")

    # Scenario 2: LLM returns malformed JSON, force_json_response=True
    # The mock for this scenario simulates that force_json_response helped the LLM return clean JSON
    print("\n--- Scenario 2: Malformed JSON (conceptually), force_json_response=True ---")
    mock_perform_completion.return_value = mock_clean_json_response 
    strategy_with_force = LLMExtractionStrategy(
        llm_config=llm_conf, schema=schema_def, extraction_type="schema", force_json_response=True
    )
    result_str_with_force = strategy_with_force.extract("url", sample_content)
    print(f"LLM raw output (should be clean JSON): {result_str_with_force}")
    try:
        parsed_forced = json.loads(result_str_with_force)
        SimpleSchema(**parsed_forced) # Validate
        print(f"Parsed successfully with force_json_response: {parsed_forced}")
    except Exception as e:
        print(f"Error parsing/validating even with force_json_response: {e}")

if __name__ == "__main__":
    handle_llm_failures()
```

#### 6.2. Example: Strategies for dealing with very long content that might exceed single LLM context windows even after chunking (e.g., iterative extraction or summarization prior to extraction).
This is a conceptual example. Real implementation would be more complex.
We'll show a simplified version where we first "summarize" chunks then extract from summaries.

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig
from pydantic import BaseModel
from unittest.mock import patch, MagicMock
import json
import os

class DocumentExtract(BaseModel):
    overall_summary: str
    key_points: List[str]

# Mocks
# 1. For summarizing chunks
mock_summarize_chunk = MagicMock()
mock_summarize_chunk.choices = [MagicMock()]
mock_summarize_chunk.usage = MagicMock(completion_tokens=20, prompt_tokens=50, total_tokens=70)
mock_summarize_chunk.usage.completion_tokens_details = {}; mock_summarize_chunk.usage.prompt_tokens_details = {}


# 2. For final extraction from combined summaries
mock_final_extract = MagicMock()
mock_final_extract.choices = [MagicMock()]
mock_final_extract.choices[0].message.content = json.dumps({
    "overall_summary": "The document discusses AI advancements and their societal impact.",
    "key_points": ["AI is evolving fast.", "Ethics are important.", "Future is exciting."]
})
mock_final_extract.usage = MagicMock(completion_tokens=40, prompt_tokens=100, total_tokens=140)
mock_final_extract.usage.completion_tokens_details = {}; mock_final_extract.usage.prompt_tokens_details = {}


def mock_llm_router(*args, **kwargs):
    # Simplistic router based on instruction
    instruction = kwargs.get('instruction', '')
    if "summarize this chunk" in instruction.lower():
        # The content of the mock is less important here than the call itself
        mock_summarize_chunk.choices[0].message.content = json.dumps({"summary_of_chunk": "Chunk summary text."})
        return mock_summarize_chunk
    elif "overall document summary" in instruction.lower():
        return mock_final_extract
    return MagicMock() # Default

@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', side_effect=mock_llm_router)
def iterative_extraction_for_long_content(mock_perform_completion):
    very_long_document_content = ("This is part of a very long document. " * 100) # Simulate long content
    
    try:
        llm_conf = LLMConfig(provider="openai/gpt-3.5-turbo", api_token=os.getenv("OPENAI_API_KEY", "mock_key_iterative"))
        if not os.getenv("OPENAI_API_KEY"): print("Warning: OPENAI_API_KEY not set for iterative example.")
    except:
        print("LLM setup failed. Skipping iterative extraction test.")
        return

    # Step 1: Create strategy for summarizing chunks (block extraction for simplicity)
    summarizer_strategy = LLMExtractionStrategy(
        llm_config=llm_conf,
        extraction_type="block", # Could be schema { "summary": "..." }
        instruction="Summarize this chunk of text concisely.",
        chunk_token_threshold=60, # Smaller chunks for summarization
        word_token_rate=0.75,
        apply_chunking=True
    )

    print("--- Step 1: Summarizing chunks of the long document ---")
    # LLMExtractionStrategy.extract returns list of blocks if extraction_type="block"
    chunk_summaries_blocks = summarizer_strategy.extract("url", very_long_document_content) 
    
    # We'd expect chunk_summaries_blocks to be a list of dicts like [{'content': 'summary1', 'tags':[]}, ...]
    # For this mock, we'll just take the mocked 'content'
    chunk_summaries = [block.get("content", "") for block in chunk_summaries_blocks if isinstance(block, dict)]
    combined_summary_text = "\n".join(chunk_summaries)
    
    print(f"Summarized {len(chunk_summaries_blocks)} chunks into combined text of length {len(combined_summary_text)}.")
    print(f"Combined summary (first 100 chars): {combined_summary_text[:100]}...")

    # Step 2: Create strategy for final extraction from combined summaries
    final_extraction_strategy = LLMExtractionStrategy(
        llm_config=llm_conf,
        schema=DocumentExtract.model_json_schema(),
        extraction_type="schema",
        instruction="From the provided combined summaries, create an overall document summary and list key points."
    )

    print("\n--- Step 2: Final extraction from combined summaries ---")
    final_extracted_json_str = final_extraction_strategy.extract("url_summary", combined_summary_text)

    if final_extracted_json_str:
        final_data = json.loads(final_extracted_json_str)
        print(json.dumps(final_data, indent=2))
        doc_extract_instance = DocumentExtract(**final_data)
        assert "AI is evolving fast" in doc_extract_instance.key_points
    else:
        print("Final extraction failed.")

if __name__ == "__main__":
    iterative_extraction_for_long_content()
```

#### 6.3. Example: Showing how to access `TokenUsage` statistics after an LLM extraction.

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig
from unittest.mock import patch, MagicMock
import json

# Mock LLM response with usage data
mock_llm_usage = MagicMock()
mock_llm_usage.choices = [MagicMock(message=MagicMock(content=json.dumps({"extracted": "data"})))]
mock_llm_usage.usage = MagicMock()
mock_llm_usage.usage.completion_tokens = 50
mock_llm_usage.usage.prompt_tokens = 150
mock_llm_usage.usage.total_tokens = 200
# Example of detailed token usage (if provider supports it)
mock_llm_usage.usage.completion_tokens_details = {"gpt-3.5-turbo": 50} 
mock_llm_usage.usage.prompt_tokens_details = {"gpt-3.5-turbo": 150}

@patch('crawl4ai.extraction_strategy.perform_completion_with_backoff', return_value=mock_llm_usage)
def show_token_usage(mock_perform_completion):
    try:
        strategy = LLMExtractionStrategy(
            llm_config=LLMConfig(provider="ollama/llama3", api_token="ollama"), # Or any provider
            extraction_type="schema_from_instruction",
            instruction="Extract data."
        )
    except:
        print("Ollama not available, skipping token usage test.")
        return

    strategy.extract(url="http://dummy.com/usage_test", html_content="Some content to extract from.")
    
    print("--- Token Usage Statistics ---")
    
    # Total usage accumulated by the strategy instance across all its .extract() calls
    print(f"Total Accumulated Usage for this strategy instance:")
    print(f"  Completion Tokens: {strategy.total_usage.completion_tokens}")
    print(f"  Prompt Tokens: {strategy.total_usage.prompt_tokens}")
    print(f"  Total Tokens: {strategy.total_usage.total_tokens}")

    # Usage for the last .extract() call (or list of calls if chunking happened)
    if strategy.usages:
        print(f"\nUsage for the last .extract() operation (may include multiple LLM calls if chunked):")
        for i, usage_info in enumerate(strategy.usages[-mock_perform_completion.call_count:]): # Show for calls made in this run
            print(f"  LLM Call {i+1}:")
            print(f"    Completion Tokens: {usage_info.completion_tokens}")
            print(f"    Prompt Tokens: {usage_info.prompt_tokens}")
            print(f"    Total Tokens: {usage_info.total_tokens}")
            if usage_info.completion_tokens_details:
                 print(f"    Completion Details: {usage_info.completion_tokens_details}")
            if usage_info.prompt_tokens_details:
                 print(f"    Prompt Details: {usage_info.prompt_tokens_details}")
    else:
        print("No usage data recorded for the last operation (might be an issue or first run).")
        
    assert strategy.total_usage.total_tokens == 200 # Based on mock

if __name__ == "__main__":
    show_token_usage()
```
---

## 7. Deprecated Parameter Usage (for backward compatibility reference, if necessary)

#### 7.1. Example: Initializing `LLMExtractionStrategy` using deprecated `provider`, `api_token`, `base_url` and showing equivalence with `llm_config`. (Mark as deprecated in example comments).
This example shows the old way of passing LLM configuration directly to `LLMExtractionStrategy` and the new, recommended way using `LLMConfig`.

```python
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai.utils import LLMConfig
import os
import warnings

# Suppress deprecation warnings for this specific example
warnings.filterwarnings("ignore", category=DeprecationWarning, module="crawl4ai.extraction_strategy")


print("--- Demonstrating LLM Configuration: Deprecated vs. LLMConfig ---")

# Parameters for the example
provider_name = "openai/gpt-3.5-turbo"
api_key = os.getenv("OPENAI_API_KEY", "YOUR_OPENAI_API_KEY_PLACEHOLDER")
# base_api_url = "https://api.example.com/v1" # Example custom base URL

# --- Deprecated Way (for illustration) ---
# Note: This will raise DeprecationWarnings if not suppressed.
# It's included here to show users how to migrate.
print("\nAttempting initialization with DEPRECATED direct parameters:")
try:
    strategy_deprecated = LLMExtractionStrategy(
        provider=provider_name,
        api_token=api_key,
        # base_url=base_api_url, # If you had a custom base_url
        instruction="This is a test." # Need some instruction for it to init provider
    )
    print(f"  Deprecated way: Provider='{strategy_deprecated.llm_config.provider}', Token (first 5)='{strategy_deprecated.llm_config.api_token[:5] if strategy_deprecated.llm_config.api_token else None}', BaseURL='{strategy_deprecated.llm_config.base_url}'")
    assert strategy_deprecated.llm_config.provider == provider_name
except Exception as e:
    print(f"  Error with deprecated init (as expected if params fully removed or strict checks): {e}")


# --- New Recommended Way (using LLMConfig) ---
print("\nAttempting initialization with NEW LLMConfig object:")
llm_configuration = LLMConfig(
    provider=provider_name,
    api_token=api_key,
    # base_url=base_api_url # If you have a custom base_url
)
strategy_new = LLMExtractionStrategy(
    llm_config=llm_configuration,
    instruction="This is a test." # Need some instruction
)
print(f"  New way (LLMConfig): Provider='{strategy_new.llm_config.provider}', Token (first 5)='{strategy_new.llm_config.api_token[:5] if strategy_new.llm_config.api_token else None}', BaseURL='{strategy_new.llm_config.base_url}'")
assert strategy_new.llm_config.provider == provider_name

print("\nComparison:")
if 'strategy_deprecated' in locals() and strategy_deprecated.llm_config.provider == strategy_new.llm_config.provider:
    print("Both methods (deprecated and new) resulted in the same LLM provider configuration (provider name matches).")
else:
    print("There was a difference or the deprecated method failed as expected.")

# Restore default warning behavior if needed for other tests
warnings.resetwarnings()
```
---

**Note:** Many examples involving `LLMExtractionStrategy` use mocked LLM calls for simplicity and to avoid API key dependencies during automated testing or casual runs. When adapting these examples for real use, ensure you have a valid `LLMConfig` with appropriate provider details and API tokens, and remove or adapt the `@patch` decorators. For local testing, consider using Ollama with a model like `ollama/llama3`.

```