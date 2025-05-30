
# Detailed Outline for crawl4ai - config_objects Component

**Target Document Type:** memory
**Target Output Filename Suggestion:** `llm_memory_config_objects.md`
**Library Version Context:** 0.6.3
**Outline Generation Date:** 2024-05-24
---

## 1. Introduction to Configuration Objects in Crawl4ai

*   **1.1. Purpose of Configuration Objects**
    *   Explanation: Configuration objects in `crawl4ai` serve to centralize and manage settings for various components and behaviors of the library. This includes browser setup, individual crawl run parameters, LLM provider interactions, proxy settings, and more.
    *   Benefit: This approach enhances code readability by grouping related settings, improves maintainability by providing a clear structure for configurations, and offers ease of customization for users to tailor the library's behavior to their specific needs.
*   **1.2. General Principles and Usage**
    *   **1.2.1. Immutability/Cloning:**
        *   Concept: Most configuration objects are designed with a `clone()` method, allowing users to create modified copies without altering the original configuration instance. This promotes safer state management, especially when reusing base configurations for multiple tasks.
        *   Method: `clone(**kwargs)` on most configuration objects.
    *   **1.2.2. Serialization and Deserialization:**
        *   Concept: `crawl4ai` configuration objects can be serialized to dictionary format (e.g., for saving to JSON) and deserialized back into their respective class instances.
        *   Methods:
            *   `dump() -> dict`: Serializes the object to a dictionary suitable for JSON, often using the internal `to_serializable_dict` helper.
            *   `load(data: dict) -> ConfigClass` (Static Method): Deserializes an object from a dictionary, often using the internal `from_serializable_dict` helper.
            *   `to_dict() -> dict`: Converts the object to a standard Python dictionary.
            *   `from_dict(data: dict) -> ConfigClass` (Static Method): Creates an instance from a standard Python dictionary.
        *   Helper Functions:
            *   `crawl4ai.async_configs.to_serializable_dict(obj: Any, ignore_default_value: bool = False) -> Dict`: Recursively converts objects into a serializable dictionary format, handling complex types like enums and nested objects.
            *   `crawl4ai.async_configs.from_serializable_dict(data: Any) -> Any`: Reconstructs Python objects from the serializable dictionary format.
*   **1.3. Scope of this Document**
    *   Statement: This document provides a factual API reference for the primary configuration objects within the `crawl4ai` library, detailing their purpose, initialization parameters, attributes, and key methods.

## 2. Core Configuration Objects

### 2.1. `BrowserConfig`
Located in `crawl4ai.async_configs`.

*   **2.1.1. Purpose:**
    *   Description: The `BrowserConfig` class is used to configure the settings for a browser instance and its associated contexts when using browser-based crawler strategies like `AsyncPlaywrightCrawlerStrategy`. It centralizes all parameters that affect the creation and behavior of the browser.
*   **2.1.2. Initialization (`__init__`)**
    *   Signature:
        ```python
        class BrowserConfig:
            def __init__(
                self,
                browser_type: str = "chromium",
                headless: bool = True,
                browser_mode: str = "dedicated",
                use_managed_browser: bool = False,
                cdp_url: Optional[str] = None,
                use_persistent_context: bool = False,
                user_data_dir: Optional[str] = None,
                chrome_channel: Optional[str] = "chromium", # Note: 'channel' is preferred
                channel: Optional[str] = "chromium",
                proxy: Optional[str] = None,
                proxy_config: Optional[Union[ProxyConfig, dict]] = None,
                viewport_width: int = 1080,
                viewport_height: int = 600,
                viewport: Optional[dict] = None,
                accept_downloads: bool = False,
                downloads_path: Optional[str] = None,
                storage_state: Optional[Union[str, dict]] = None,
                ignore_https_errors: bool = True,
                java_script_enabled: bool = True,
                sleep_on_close: bool = False,
                verbose: bool = True,
                cookies: Optional[List[dict]] = None,
                headers: Optional[dict] = None,
                user_agent: Optional[str] = "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 Chrome/116.0.0.0 Safari/537.36",
                user_agent_mode: Optional[str] = "",
                user_agent_generator_config: Optional[dict] = None, # Default is {} in __init__
                text_mode: bool = False,
                light_mode: bool = False,
                extra_args: Optional[List[str]] = None,
                debugging_port: int = 9222,
                host: str = "localhost"
            ): ...
        ```
    *   Parameters:
        *   `browser_type (str, default: "chromium")`: Specifies the browser engine to use. Supported values: `"chromium"`, `"firefox"`, `"webkit"`.
        *   `headless (bool, default: True)`: If `True`, runs the browser without a visible GUI. Set to `False` for debugging or visual interaction.
        *   `browser_mode (str, default: "dedicated")`: Defines how the browser is initialized. Options: `"builtin"` (uses built-in CDP), `"dedicated"` (new instance each time), `"cdp"` (connects to an existing CDP endpoint specified by `cdp_url`), `"docker"` (runs browser in a Docker container).
        *   `use_managed_browser (bool, default: False)`: If `True`, launches the browser using a managed approach (e.g., via CDP or Docker), allowing for more advanced control. Automatically set to `True` if `browser_mode` is `"builtin"`, `"docker"`, or if `cdp_url` is provided, or if `use_persistent_context` is `True`.
        *   `cdp_url (Optional[str], default: None)`: The URL for the Chrome DevTools Protocol (CDP) endpoint. If not provided and `use_managed_browser` is active, it might be set by an internal browser manager.
        *   `use_persistent_context (bool, default: False)`: If `True`, uses a persistent browser context (profile), saving cookies, localStorage, etc., across sessions. Requires `user_data_dir`. Sets `use_managed_browser=True`.
        *   `user_data_dir (Optional[str], default: None)`: Path to a directory for storing user data for persistent sessions. If `None` and `use_persistent_context` is `True`, a temporary directory might be used.
        *   `chrome_channel (Optional[str], default: "chromium")`: Specifies the Chrome channel (e.g., "chrome", "msedge", "chromium-beta"). Only applicable if `browser_type` is "chromium".
        *   `channel (Optional[str], default: "chromium")`: Preferred alias for `chrome_channel`. Set to `""` for Firefox or WebKit.
        *   `proxy (Optional[str], default: None)`: A string representing the proxy server URL (e.g., "http://username:password@proxy.example.com:8080").
        *   `proxy_config (Optional[Union[ProxyConfig, dict]], default: None)`: A `ProxyConfig` object or a dictionary specifying detailed proxy settings. Overrides the `proxy` string if both are provided.
        *   `viewport_width (int, default: 1080)`: Default width of the browser viewport in pixels.
        *   `viewport_height (int, default: 600)`: Default height of the browser viewport in pixels.
        *   `viewport (Optional[dict], default: None)`: A dictionary specifying viewport dimensions, e.g., `{"width": 1920, "height": 1080}`. If set, overrides `viewport_width` and `viewport_height`.
        *   `accept_downloads (bool, default: False)`: If `True`, allows files to be downloaded by the browser.
        *   `downloads_path (Optional[str], default: None)`: Directory path where downloaded files will be stored. Required if `accept_downloads` is `True`.
        *   `storage_state (Optional[Union[str, dict]], default: None)`: Path to a JSON file or a dictionary containing the browser's storage state (cookies, localStorage, etc.) to load.
        *   `ignore_https_errors (bool, default: True)`: If `True`, HTTPS certificate errors will be ignored.
        *   `java_script_enabled (bool, default: True)`: If `True`, JavaScript execution is enabled on web pages.
        *   `sleep_on_close (bool, default: False)`: If `True`, introduces a small delay before the browser is closed.
        *   `verbose (bool, default: True)`: If `True`, enables verbose logging for browser operations.
        *   `cookies (Optional[List[dict]], default: None)`: A list of cookie dictionaries to be set in the browser context. Each dictionary should conform to Playwright's cookie format.
        *   `headers (Optional[dict], default: None)`: A dictionary of additional HTTP headers to be sent with every request made by the browser.
        *   `user_agent (Optional[str], default: "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 Chrome/116.0.0.0 Safari/537.36")`: The User-Agent string the browser will use.
        *   `user_agent_mode (Optional[str], default: "")`: Mode for generating the User-Agent string. If set (e.g., to "random"), `user_agent_generator_config` can be used.
        *   `user_agent_generator_config (Optional[dict], default: {})`: Configuration dictionary for the User-Agent generator if `user_agent_mode` is active.
        *   `text_mode (bool, default: False)`: If `True`, attempts to disable images and other rich content to potentially speed up loading for text-focused crawls.
        *   `light_mode (bool, default: False)`: If `True`, disables certain background browser features for potential performance gains.
        *   `extra_args (Optional[List[str]], default: None)`: A list of additional command-line arguments to pass to the browser executable upon launch.
        *   `debugging_port (int, default: 9222)`: The port to use for the browser's remote debugging protocol (CDP).
        *   `host (str, default: "localhost")`: The host on which the browser's remote debugging protocol will listen.
*   **2.1.3. Key Public Attributes/Properties:**
    *   All parameters listed in `__init__` are available as public attributes with the same names and types.
    *   `browser_hint (str)`: [Read-only] - A string representing client hints (Sec-CH-UA) generated based on the `user_agent` string. This is automatically set during initialization.
*   **2.1.4. Key Public Methods:**
    *   `from_kwargs(cls, kwargs: dict) -> BrowserConfig` (Static Method):
        *   Purpose: Creates a `BrowserConfig` instance from a dictionary of keyword arguments.
    *   `to_dict(self) -> dict`:
        *   Purpose: Converts the `BrowserConfig` instance into a dictionary representation.
    *   `clone(self, **kwargs) -> BrowserConfig`:
        *   Purpose: Creates a deep copy of the current `BrowserConfig` instance. Keyword arguments can be provided to override specific attributes in the new instance.
    *   `dump(self) -> dict`:
        *   Purpose: Serializes the `BrowserConfig` object into a dictionary format that is suitable for JSON storage or transmission, utilizing the `to_serializable_dict` helper.
    *   `load(cls, data: dict) -> BrowserConfig` (Static Method):
        *   Purpose: Deserializes a `BrowserConfig` object from a dictionary (typically one created by `dump()`), utilizing the `from_serializable_dict` helper.

### 2.2. `CrawlerRunConfig`
Located in `crawl4ai.async_configs`.

*   **2.2.1. Purpose:**
    *   Description: The `CrawlerRunConfig` class encapsulates all settings that control the behavior of a single crawl operation performed by `AsyncWebCrawler.arun()` or multiple operations within `AsyncWebCrawler.arun_many()`. This includes parameters for content processing, page interaction, caching, and media handling.
*   **2.2.2. Initialization (`__init__`)**
    *   Signature:
        ```python
        class CrawlerRunConfig:
            def __init__(
                self,
                url: Optional[str] = None,
                word_count_threshold: int = MIN_WORD_THRESHOLD,
                extraction_strategy: Optional[ExtractionStrategy] = None,
                chunking_strategy: Optional[ChunkingStrategy] = RegexChunking(),
                markdown_generator: Optional[MarkdownGenerationStrategy] = DefaultMarkdownGenerator(),
                only_text: bool = False,
                css_selector: Optional[str] = None,
                target_elements: Optional[List[str]] = None, # Default is [] in __init__
                excluded_tags: Optional[List[str]] = None, # Default is [] in __init__
                excluded_selector: Optional[str] = "", # Default is "" in __init__
                keep_data_attributes: bool = False,
                keep_attrs: Optional[List[str]] = None, # Default is [] in __init__
                remove_forms: bool = False,
                prettify: bool = False,
                parser_type: str = "lxml",
                scraping_strategy: Optional[ContentScrapingStrategy] = None, # Instantiated with WebScrapingStrategy() if None
                proxy_config: Optional[Union[ProxyConfig, dict]] = None,
                proxy_rotation_strategy: Optional[ProxyRotationStrategy] = None,
                locale: Optional[str] = None,
                timezone_id: Optional[str] = None,
                geolocation: Optional[GeolocationConfig] = None,
                fetch_ssl_certificate: bool = False,
                cache_mode: CacheMode = CacheMode.BYPASS,
                session_id: Optional[str] = None,
                shared_data: Optional[dict] = None,
                wait_until: str = "domcontentloaded",
                page_timeout: int = PAGE_TIMEOUT,
                wait_for: Optional[str] = None,
                wait_for_timeout: Optional[int] = None,
                wait_for_images: bool = False,
                delay_before_return_html: float = 0.1,
                mean_delay: float = 0.1,
                max_range: float = 0.3,
                semaphore_count: int = 5,
                js_code: Optional[Union[str, List[str]]] = None,
                js_only: bool = False,
                ignore_body_visibility: bool = True,
                scan_full_page: bool = False,
                scroll_delay: float = 0.2,
                process_iframes: bool = False,
                remove_overlay_elements: bool = False,
                simulate_user: bool = False,
                override_navigator: bool = False,
                magic: bool = False,
                adjust_viewport_to_content: bool = False,
                screenshot: bool = False,
                screenshot_wait_for: Optional[float] = None,
                screenshot_height_threshold: int = SCREENSHOT_HEIGHT_THRESHOLD,
                pdf: bool = False,
                capture_mhtml: bool = False,
                image_description_min_word_threshold: int = IMAGE_DESCRIPTION_MIN_WORD_THRESHOLD,
                image_score_threshold: int = IMAGE_SCORE_THRESHOLD,
                table_score_threshold: int = 7,
                exclude_external_images: bool = False,
                exclude_all_images: bool = False,
                exclude_social_media_domains: Optional[List[str]] = None, # Uses SOCIAL_MEDIA_DOMAINS if None
                exclude_external_links: bool = False,
                exclude_social_media_links: bool = False,
                exclude_domains: Optional[List[str]] = None, # Default is [] in __init__
                exclude_internal_links: bool = False,
                verbose: bool = True,
                log_console: bool = False,
                capture_network_requests: bool = False,
                capture_console_messages: bool = False,
                method: str = "GET",
                stream: bool = False,
                check_robots_txt: bool = False,
                user_agent: Optional[str] = None,
                user_agent_mode: Optional[str] = None,
                user_agent_generator_config: Optional[dict] = None, # Default is {} in __init__
                deep_crawl_strategy: Optional[DeepCrawlStrategy] = None,
                experimental: Optional[Dict[str, Any]] = None # Default is {} in __init__
            ): ...
        ```
    *   Parameters:
        *   `url (Optional[str], default: None)`: The target URL for this specific crawl run.
        *   `word_count_threshold (int, default: MIN_WORD_THRESHOLD)`: Minimum word count for a text block to be considered significant during content processing.
        *   `extraction_strategy (Optional[ExtractionStrategy], default: None)`: Strategy for extracting structured data from the page. If `None`, `NoExtractionStrategy` is used.
        *   `chunking_strategy (Optional[ChunkingStrategy], default: RegexChunking())`: Strategy to split content into chunks before extraction.
        *   `markdown_generator (Optional[MarkdownGenerationStrategy], default: DefaultMarkdownGenerator())`: Strategy for converting HTML to Markdown.
        *   `only_text (bool, default: False)`: If `True`, attempts to extract only textual content, potentially ignoring structural elements beneficial for rich Markdown.
        *   `css_selector (Optional[str], default: None)`: A CSS selector defining the primary region of the page to focus on for content extraction. The raw HTML is reduced to this region.
        *   `target_elements (Optional[List[str]], default: [])`: A list of CSS selectors. If provided, only the content within these elements will be considered for Markdown generation and structured data extraction. Unlike `css_selector`, this does not reduce the raw HTML but scopes the processing.
        *   `excluded_tags (Optional[List[str]], default: [])`: A list of HTML tag names (e.g., "nav", "footer") to be removed from the HTML before processing.
        *   `excluded_selector (Optional[str], default: "")`: A CSS selector specifying elements to be removed from the HTML before processing.
        *   `keep_data_attributes (bool, default: False)`: If `True`, `data-*` attributes on HTML elements are preserved during cleaning.
        *   `keep_attrs (Optional[List[str]], default: [])`: A list of specific HTML attribute names to preserve during HTML cleaning.
        *   `remove_forms (bool, default: False)`: If `True`, all `<form>` elements are removed from the HTML.
        *   `prettify (bool, default: False)`: If `True`, the cleaned HTML output is "prettified" for better readability.
        *   `parser_type (str, default: "lxml")`: The HTML parser to be used by the scraping strategy (e.g., "lxml", "html.parser").
        *   `scraping_strategy (Optional[ContentScrapingStrategy], default: WebScrapingStrategy())`: The strategy for scraping content from the HTML.
        *   `proxy_config (Optional[Union[ProxyConfig, dict]], default: None)`: Proxy configuration for this specific run. Overrides any proxy settings in `BrowserConfig`.
        *   `proxy_rotation_strategy (Optional[ProxyRotationStrategy], default: None)`: Strategy to use for rotating proxies if multiple are available.
        *   `locale (Optional[str], default: None)`: Locale to set for the browser context (e.g., "en-US", "fr-FR"). Affects `Accept-Language` header and JavaScript `navigator.language`.
        *   `timezone_id (Optional[str], default: None)`: Timezone ID to set for the browser context (e.g., "America/New_York", "Europe/Paris"). Affects JavaScript `Date` objects.
        *   `geolocation (Optional[GeolocationConfig], default: None)`: A `GeolocationConfig` object or dictionary to set the browser's mock geolocation.
        *   `fetch_ssl_certificate (bool, default: False)`: If `True`, the SSL certificate information for the main URL will be fetched and included in the `CrawlResult`.
        *   `cache_mode (CacheMode, default: CacheMode.BYPASS)`: Defines caching behavior for this run. See `CacheMode` enum for options.
        *   `session_id (Optional[str], default: None)`: An identifier for a browser session. If provided, `crawl4ai` will attempt to reuse an existing page/context associated with this ID, or create a new one and associate it.
        *   `shared_data (Optional[dict], default: None)`: A dictionary for passing custom data between hooks during the crawl lifecycle.
        *   `wait_until (str, default: "domcontentloaded")`: Playwright's page navigation wait condition (e.g., "load", "domcontentloaded", "networkidle", "commit").
        *   `page_timeout (int, default: PAGE_TIMEOUT)`: Maximum time in milliseconds for page navigation and other page operations.
        *   `wait_for (Optional[str], default: None)`: A CSS selector or a JavaScript expression (prefixed with "js:"). The crawler will wait until this condition is met before proceeding.
        *   `wait_for_timeout (Optional[int], default: None)`: Specific timeout in milliseconds for the `wait_for` condition. If `None`, `page_timeout` is used.
        *   `wait_for_images (bool, default: False)`: If `True`, attempts to wait for all images on the page to finish loading.
        *   `delay_before_return_html (float, default: 0.1)`: Delay in seconds to wait just before the final HTML content is retrieved from the page.
        *   `mean_delay (float, default: 0.1)`: Used with `arun_many`. The mean base delay in seconds between processing URLs.
        *   `max_range (float, default: 0.3)`: Used with `arun_many`. The maximum additional random delay (added to `mean_delay`) between processing URLs.
        *   `semaphore_count (int, default: 5)`: Used with `arun_many` and semaphore-based dispatchers. The maximum number of concurrent crawl operations.
        *   `js_code (Optional[Union[str, List[str]]], default: None)`: A string or list of strings containing JavaScript code to be executed on the page after it loads.
        *   `js_only (bool, default: False)`: If `True`, indicates that this `arun` call is primarily for JavaScript execution on an already loaded page (within a session) and a full page navigation might not be needed.
        *   `ignore_body_visibility (bool, default: True)`: If `True`, proceeds with content extraction even if the `<body>` element is not deemed visible by Playwright.
        *   `scan_full_page (bool, default: False)`: If `True`, the crawler will attempt to scroll through the entire page to trigger lazy-loaded content.
        *   `scroll_delay (float, default: 0.2)`: Delay in seconds between each scroll step when `scan_full_page` is `True`.
        *   `process_iframes (bool, default: False)`: If `True`, attempts to extract and inline content from `<iframe>` elements.
        *   `remove_overlay_elements (bool, default: False)`: If `True`, attempts to identify and remove common overlay elements (popups, cookie banners) before content extraction.
        *   `simulate_user (bool, default: False)`: If `True`, enables heuristics to simulate user interactions (like mouse movements) to potentially bypass some anti-bot measures.
        *   `override_navigator (bool, default: False)`: If `True`, overrides certain JavaScript `navigator` properties to appear more like a standard browser.
        *   `magic (bool, default: False)`: If `True`, enables a combination of techniques (like `remove_overlay_elements`, `simulate_user`) to try and handle dynamic/obfuscated sites.
        *   `adjust_viewport_to_content (bool, default: False)`: If `True`, attempts to adjust the browser viewport size to match the dimensions of the page content.
        *   `screenshot (bool, default: False)`: If `True`, a screenshot of the page will be taken and included in `CrawlResult.screenshot`.
        *   `screenshot_wait_for (Optional[float], default: None)`: Additional delay in seconds to wait before taking the screenshot.
        *   `screenshot_height_threshold (int, default: SCREENSHOT_HEIGHT_THRESHOLD)`: If page height exceeds this, a full-page screenshot strategy might be different.
        *   `pdf (bool, default: False)`: If `True`, a PDF version of the page will be generated and included in `CrawlResult.pdf`.
        *   `capture_mhtml (bool, default: False)`: If `True`, an MHTML archive of the page will be captured and included in `CrawlResult.mhtml`.
        *   `image_description_min_word_threshold (int, default: IMAGE_DESCRIPTION_MIN_WORD_THRESHOLD)`: Minimum word count for surrounding text to be considered as an image description.
        *   `image_score_threshold (int, default: IMAGE_SCORE_THRESHOLD)`: Heuristic score threshold for an image to be included in `CrawlResult.media`.
        *   `table_score_threshold (int, default: 7)`: Heuristic score threshold for an HTML table to be considered a data table and included in `CrawlResult.media`.
        *   `exclude_external_images (bool, default: False)`: If `True`, images hosted on different domains than the main page URL are excluded.
        *   `exclude_all_images (bool, default: False)`: If `True`, all images are excluded from `CrawlResult.media`.
        *   `exclude_social_media_domains (Optional[List[str]], default: SOCIAL_MEDIA_DOMAINS from config)`: List of social media domains whose links should be excluded.
        *   `exclude_external_links (bool, default: False)`: If `True`, all links pointing to external domains are excluded from `CrawlResult.links`.
        *   `exclude_social_media_links (bool, default: False)`: If `True`, links to domains in `exclude_social_media_domains` are excluded.
        *   `exclude_domains (Optional[List[str]], default: [])`: A list of specific domains whose links should be excluded.
        *   `exclude_internal_links (bool, default: False)`: If `True`, all links pointing to the same domain are excluded.
        *   `verbose (bool, default: True)`: Enables verbose logging for this specific crawl run. Overrides `BrowserConfig.verbose`.
        *   `log_console (bool, default: False)`: If `True`, browser console messages are captured (requires `capture_console_messages=True` to be effective).
        *   `capture_network_requests (bool, default: False)`: If `True`, captures details of network requests and responses made by the page.
        *   `capture_console_messages (bool, default: False)`: If `True`, captures messages logged to the browser's console.
        *   `method (str, default: "GET")`: HTTP method to use, primarily for `AsyncHTTPCrawlerStrategy`.
        *   `stream (bool, default: False)`: If `True` when using `arun_many`, results are yielded as an async generator instead of returned as a list at the end.
        *   `check_robots_txt (bool, default: False)`: If `True`, `robots.txt` rules for the domain will be checked and respected.
        *   `user_agent (Optional[str], default: None)`: User-Agent string for this specific run. Overrides `BrowserConfig.user_agent`.
        *   `user_agent_mode (Optional[str], default: None)`: User-Agent generation mode for this specific run.
        *   `user_agent_generator_config (Optional[dict], default: {})`: Configuration for User-Agent generator for this run.
        *   `deep_crawl_strategy (Optional[DeepCrawlStrategy], default: None)`: Strategy to use for deep crawling beyond the initial URL.
        *   `experimental (Optional[Dict[str, Any]], default: {})`: A dictionary for passing experimental or beta parameters.
*   **2.2.3. Key Public Attributes/Properties:**
    *   All parameters listed in `__init__` are available as public attributes with the same names and types.
*   **2.2.4. Deprecated Property Handling (`__getattr__`, `_UNWANTED_PROPS`)**
    *   Behavior: Attempting to access a deprecated property (e.g., `bypass_cache`, `disable_cache`, `no_cache_read`, `no_cache_write`) raises an `AttributeError`. The error message directs the user to use the `cache_mode` parameter with the appropriate `CacheMode` enum member instead.
    *   List of Deprecated Properties and their `CacheMode` Equivalents:
        *   `bypass_cache`: Use `cache_mode=CacheMode.BYPASS`.
        *   `disable_cache`: Use `cache_mode=CacheMode.DISABLE`.
        *   `no_cache_read`: Use `cache_mode=CacheMode.WRITE_ONLY`.
        *   `no_cache_write`: Use `cache_mode=CacheMode.READ_ONLY`.
*   **2.2.5. Key Public Methods:**
    *   `from_kwargs(cls, kwargs: dict) -> CrawlerRunConfig` (Static Method):
        *   Purpose: Creates a `CrawlerRunConfig` instance from a dictionary of keyword arguments.
    *   `dump(self) -> dict`:
        *   Purpose: Serializes the `CrawlerRunConfig` object to a dictionary suitable for JSON storage, handling complex nested objects using `to_serializable_dict`.
    *   `load(cls, data: dict) -> CrawlerRunConfig` (Static Method):
        *   Purpose: Deserializes a `CrawlerRunConfig` object from a dictionary (typically one created by `dump()`), using `from_serializable_dict`.
    *   `to_dict(self) -> dict`:
        *   Purpose: Converts the `CrawlerRunConfig` instance into a dictionary representation. Complex objects like strategies are typically represented by their class name or a simplified form.
    *   `clone(self, **kwargs) -> CrawlerRunConfig`:
        *   Purpose: Creates a deep copy of the current `CrawlerRunConfig` instance. Keyword arguments can be provided to override specific attributes in the new instance.

### 2.3. `LLMConfig`
Located in `crawl4ai.async_configs`.

*   **2.3.1. Purpose:**
    *   Description: The `LLMConfig` class provides configuration for interacting with Large Language Model (LLM) providers. It includes settings for the provider name, API token, base URL, and various model-specific parameters like temperature and max tokens.
*   **2.3.2. Initialization (`__init__`)**
    *   Signature:
        ```python
        class LLMConfig:
            def __init__(
                self,
                provider: str = DEFAULT_PROVIDER, # e.g., "openai/gpt-4o-mini"
                api_token: Optional[str] = None,
                base_url: Optional[str] = None,
                temperature: Optional[float] = None,
                max_tokens: Optional[int] = None,
                top_p: Optional[float] = None,
                frequency_penalty: Optional[float] = None,
                presence_penalty: Optional[float] = None,
                stop: Optional[List[str]] = None,
                n: Optional[int] = None,
            ): ...
        ```
    *   Parameters:
        *   `provider (str, default: DEFAULT_PROVIDER)`: The identifier for the LLM provider and model (e.g., "openai/gpt-4o-mini", "ollama/llama3.3", "gemini/gemini-1.5-pro").
        *   `api_token (Optional[str], default: None)`: The API token for authenticating with the LLM provider. If `None`, it attempts to load from environment variables based on the provider (e.g., `OPENAI_API_KEY` for OpenAI, `GEMINI_API_KEY` for Gemini). Can also be set as "env:YOUR_ENV_VAR_NAME".
        *   `base_url (Optional[str], default: None)`: A custom base URL for the LLM API endpoint, useful for self-hosted models or proxies.
        *   `temperature (Optional[float], default: None)`: Controls the randomness of the LLM's output. Higher values (e.g., 0.8) make output more random, lower values (e.g., 0.2) make it more deterministic.
        *   `max_tokens (Optional[int], default: None)`: The maximum number of tokens the LLM should generate in its response.
        *   `top_p (Optional[float], default: None)`: Nucleus sampling parameter. The model considers only tokens with cumulative probability mass up to `top_p`.
        *   `frequency_penalty (Optional[float], default: None)`: Penalizes new tokens based on their existing frequency in the text so far, decreasing the model's likelihood to repeat the same line verbatim.
        *   `presence_penalty (Optional[float], default: None)`: Penalizes new tokens based on whether they have appeared in the text so far, increasing the model's likelihood to talk about new topics.
        *   `stop (Optional[List[str]], default: None)`: A list of sequences where the API will stop generating further tokens.
        *   `n (Optional[int], default: None)`: The number of completions to generate for each prompt.
*   **2.3.3. Key Public Attributes/Properties:**
    *   All parameters listed in `__init__` are available as public attributes with the same names and types.
*   **2.3.4. Key Public Methods:**
    *   `from_kwargs(cls, kwargs: dict) -> LLMConfig` (Static Method):
        *   Purpose: Creates an `LLMConfig` instance from a dictionary of keyword arguments.
    *   `to_dict(self) -> dict`:
        *   Purpose: Converts the `LLMConfig` instance into a dictionary representation.
    *   `clone(self, **kwargs) -> LLMConfig`:
        *   Purpose: Creates a deep copy of the current `LLMConfig` instance. Keyword arguments can be provided to override specific attributes in the new instance.

### 2.4. `GeolocationConfig`
Located in `crawl4ai.async_configs`.

*   **2.4.1. Purpose:**
    *   Description: The `GeolocationConfig` class stores settings for mocking the browser's geolocation, including latitude, longitude, and accuracy.
*   **2.4.2. Initialization (`__init__`)**
    *   Signature:
        ```python
        class GeolocationConfig:
            def __init__(
                self,
                latitude: float,
                longitude: float,
                accuracy: Optional[float] = 0.0
            ): ...
        ```
    *   Parameters:
        *   `latitude (float)`: The latitude coordinate (e.g., 37.7749 for San Francisco).
        *   `longitude (float)`: The longitude coordinate (e.g., -122.4194 for San Francisco).
        *   `accuracy (Optional[float], default: 0.0)`: The accuracy of the geolocation in meters.
*   **2.4.3. Key Public Attributes/Properties:**
    *   `latitude (float)`: Stores the latitude.
    *   `longitude (float)`: Stores the longitude.
    *   `accuracy (Optional[float])`: Stores the accuracy.
*   **2.4.4. Key Public Methods:**
    *   `from_dict(cls, geo_dict: dict) -> GeolocationConfig` (Static Method):
        *   Purpose: Creates a `GeolocationConfig` instance from a dictionary.
    *   `to_dict(self) -> dict`:
        *   Purpose: Converts the `GeolocationConfig` instance to a dictionary: `{"latitude": ..., "longitude": ..., "accuracy": ...}`.
    *   `clone(self, **kwargs) -> GeolocationConfig`:
        *   Purpose: Creates a copy of the `GeolocationConfig` instance, allowing for overriding specific attributes with `kwargs`.

### 2.5. `ProxyConfig`
Located in `crawl4ai.async_configs` (and `crawl4ai.proxy_strategy`).

*   **2.5.1. Purpose:**
    *   Description: The `ProxyConfig` class encapsulates the configuration for a single proxy server, including its address, authentication credentials (if any), and optionally its public IP address.
*   **2.5.2. Initialization (`__init__`)**
    *   Signature:
        ```python
        class ProxyConfig:
            def __init__(
                self,
                server: str,
                username: Optional[str] = None,
                password: Optional[str] = None,
                ip: Optional[str] = None,
            ): ...
        ```
    *   Parameters:
        *   `server (str)`: The proxy server URL, including protocol and port (e.g., "http://127.0.0.1:8080", "socks5://proxy.example.com:1080").
        *   `username (Optional[str], default: None)`: The username for proxy authentication, if required.
        *   `password (Optional[str], default: None)`: The password for proxy authentication, if required.
        *   `ip (Optional[str], default: None)`: The public IP address of the proxy server. If not provided, it will be automatically extracted from the `server` string if possible.
*   **2.5.3. Key Public Attributes/Properties:**
    *   `server (str)`: The proxy server URL.
    *   `username (Optional[str])`: The username for proxy authentication.
    *   `password (Optional[str])`: The password for proxy authentication.
    *   `ip (Optional[str])`: The public IP address of the proxy. This is either user-provided or automatically extracted from the `server` string during initialization via the internal `_extract_ip_from_server` method.
*   **2.5.4. Key Public Methods:**
    *   `_extract_ip_from_server(self) -> Optional[str]` (Internal method):
        *   Purpose: Extracts the IP address component from the `self.server` URL string.
    *   `from_string(cls, proxy_str: str) -> ProxyConfig` (Static Method):
        *   Purpose: Creates a `ProxyConfig` instance from a string.
        *   Formats:
            *   `'ip:port:username:password'`
            *   `'ip:port'` (no authentication)
    *   `from_dict(cls, proxy_dict: dict) -> ProxyConfig` (Static Method):
        *   Purpose: Creates a `ProxyConfig` instance from a dictionary with keys "server", "username", "password", and "ip".
    *   `from_env(cls, env_var: str = "PROXIES") -> List[ProxyConfig]` (Static Method):
        *   Purpose: Loads a list of `ProxyConfig` objects from a comma-separated environment variable. Each proxy string in the variable should conform to the format accepted by `from_string`.
    *   `to_dict(self) -> dict`:
        *   Purpose: Converts the `ProxyConfig` instance to a dictionary: `{"server": ..., "username": ..., "password": ..., "ip": ...}`.
    *   `clone(self, **kwargs) -> ProxyConfig`:
        *   Purpose: Creates a copy of the `ProxyConfig` instance, allowing for overriding specific attributes with `kwargs`.

### 2.6. `HTTPCrawlerConfig`
Located in `crawl4ai.async_configs`.

*   **2.6.1. Purpose:**
    *   Description: The `HTTPCrawlerConfig` class holds configuration settings specific to direct HTTP-based crawling strategies (e.g., `AsyncHTTPCrawlerStrategy`), which do not use a full browser environment.
*   **2.6.2. Initialization (`__init__`)**
    *   Signature:
        ```python
        class HTTPCrawlerConfig:
            def __init__(
                self,
                method: str = "GET",
                headers: Optional[Dict[str, str]] = None,
                data: Optional[Dict[str, Any]] = None,
                json: Optional[Dict[str, Any]] = None,
                follow_redirects: bool = True,
                verify_ssl: bool = True,
            ): ...
        ```
    *   Parameters:
        *   `method (str, default: "GET")`: The HTTP method to use for the request (e.g., "GET", "POST", "PUT").
        *   `headers (Optional[Dict[str, str]], default: None)`: A dictionary of custom HTTP headers to send with the request.
        *   `data (Optional[Dict[str, Any]], default: None)`: Data to be sent in the body of the request, typically for "POST" or "PUT" requests (e.g., form data).
        *   `json (Optional[Dict[str, Any]], default: None)`: JSON data to be sent in the body of the request. If provided, the `Content-Type` header is typically set to `application/json`.
        *   `follow_redirects (bool, default: True)`: If `True`, the crawler will automatically follow HTTP redirects.
        *   `verify_ssl (bool, default: True)`: If `True`, SSL certificates will be verified. Set to `False` to ignore SSL errors (use with caution).
*   **2.6.3. Key Public Attributes/Properties:**
    *   All parameters listed in `__init__` are available as public attributes with the same names and types.
*   **2.6.4. Key Public Methods:**
    *   `from_kwargs(cls, kwargs: dict) -> HTTPCrawlerConfig` (Static Method):
        *   Purpose: Creates an `HTTPCrawlerConfig` instance from a dictionary of keyword arguments.
    *   `to_dict(self) -> dict`:
        *   Purpose: Converts the `HTTPCrawlerConfig` instance into a dictionary representation.
    *   `clone(self, **kwargs) -> HTTPCrawlerConfig`:
        *   Purpose: Creates a deep copy of the current `HTTPCrawlerConfig` instance. Keyword arguments can be provided to override specific attributes in the new instance.
    *   `dump(self) -> dict`:
        *   Purpose: Serializes the `HTTPCrawlerConfig` object to a dictionary.
    *   `load(cls, data: dict) -> HTTPCrawlerConfig` (Static Method):
        *   Purpose: Deserializes an `HTTPCrawlerConfig` object from a dictionary.

## 3. Enumerations and Helper Constants

### 3.1. `CacheMode` (Enum)
Located in `crawl4ai.cache_context`.

*   **3.1.1. Purpose:**
    *   Description: The `CacheMode` enumeration defines the different caching behaviors that can be applied to a crawl operation. It is used in `CrawlerRunConfig` to control how results are read from and written to the cache.
*   **3.1.2. Enum Members:**
    *   `ENABLE (str)`: Value: "ENABLE". Description: Enables normal caching behavior. The crawler will attempt to read from the cache first, and if a result is not found or is stale, it will perform the crawl and write the new result to the cache.
    *   `DISABLE (str)`: Value: "DISABLE". Description: Disables all caching. The crawler will not read from or write to the cache. Every request will be a fresh crawl.
    *   `READ_ONLY (str)`: Value: "READ_ONLY". Description: The crawler will only attempt to read from the cache. If a result is found, it will be used. If not, the crawl will not proceed further for that URL, and no new data will be written to the cache.
    *   `WRITE_ONLY (str)`: Value: "WRITE_ONLY". Description: The crawler will not attempt to read from the cache. It will always perform a fresh crawl and then write the result to the cache.
    *   `BYPASS (str)`: Value: "BYPASS". Description: The crawler will skip reading from the cache for this specific operation and will perform a fresh crawl. The result of this crawl *will* be written to the cache. This is the default `cache_mode` for `CrawlerRunConfig`.
*   **3.1.3. Usage:**
    *   Example:
        ```python
        from crawl4ai import CrawlerRunConfig, CacheMode
        config = CrawlerRunConfig(cache_mode=CacheMode.ENABLE) # Use cache fully
        config_bypass = CrawlerRunConfig(cache_mode=CacheMode.BYPASS) # Force fresh crawl, then cache
        ```

## 4. Serialization Helper Functions
Located in `crawl4ai.async_configs`.

### 4.1. `to_serializable_dict(obj: Any, ignore_default_value: bool = False) -> Dict`

*   **4.1.1. Purpose:**
    *   Description: This utility function recursively converts various Python objects, including `crawl4ai` configuration objects, into a dictionary format that is suitable for JSON serialization. It uses a `{ "type": "ClassName", "params": { ... } }` structure for custom class instances to enable proper deserialization later.
*   **4.1.2. Parameters:**
    *   `obj (Any)`: The Python object to be serialized.
    *   `ignore_default_value (bool, default: False)`: If `True`, when serializing class instances, parameters whose current values match their `__init__` default values might be excluded from the "params" dictionary. (Note: The exact behavior depends on the availability of default values in the class signature and handling of empty/None values).
*   **4.1.3. Returns:**
    *   `Dict`: A dictionary representation of the input object, structured for easy serialization (e.g., to JSON) and later deserialization by `from_serializable_dict`.
*   **4.1.4. Key Behaviors:**
    *   **Basic Types:** `str`, `int`, `float`, `bool`, `None` are returned as is.
    *   **Enums:** Serialized as `{"type": "EnumClassName", "params": enum_member.value}`.
    *   **Datetime Objects:** Serialized to their ISO 8601 string representation.
    *   **Lists, Tuples, Sets, Frozensets:** Serialized by recursively calling `to_serializable_dict` on each of their elements, returning a list.
    *   **Plain Dictionaries:** Serialized as `{"type": "dict", "value": {key: serialized_value, ...}}`.
    *   **Class Instances (e.g., Config Objects):**
        *   The object's class name is stored in the "type" field.
        *   Parameters from the `__init__` signature and attributes from `__slots__` (if defined) are collected.
        *   Their current values are recursively serialized and stored in the "params" dictionary.
        *   The structure is `{"type": "ClassName", "params": {"param_name": serialized_param_value, ...}}`.

### 4.2. `from_serializable_dict(data: Any) -> Any`

*   **4.2.1. Purpose:**
    *   Description: This utility function reconstructs Python objects, including `crawl4ai` configuration objects, from the serializable dictionary format previously created by `to_serializable_dict`.
*   **4.2.2. Parameters:**
    *   `data (Any)`: The dictionary (or basic data type) to be deserialized. This is typically the output of `to_serializable_dict` after being, for example, loaded from a JSON string.
*   **4.2.3. Returns:**
    *   `Any`: The reconstructed Python object (e.g., an instance of `BrowserConfig`, `LLMConfig`, a list, a plain dictionary, etc.).
*   **4.2.4. Key Behaviors:**
    *   **Basic Types:** `str`, `int`, `float`, `bool`, `None` are returned as is.
    *   **Typed Dictionaries (from `to_serializable_dict`):**
        *   If `data` is a dictionary and contains a "type" key:
            *   If `data["type"] == "dict"`, it reconstructs a plain Python dictionary from `data["value"]` by recursively deserializing its items.
            *   Otherwise, it attempts to locate the class specified by `data["type"]` within the `crawl4ai` module.
                *   If the class is an `Enum`, it instantiates the enum member using `data["params"]` (the enum value).
                *   If it's a regular class, it recursively deserializes the items in `data["params"]` and uses them as keyword arguments (`**kwargs`) to instantiate the class.
    *   **Lists:** If `data` is a list, it reconstructs a list by recursively calling `from_serializable_dict` on each of its elements.
    *   **Legacy Dictionaries:** If `data` is a dictionary but does not conform to the "type" key structure (for backward compatibility), it attempts to deserialize its values.

## 5. Cross-References and Relationships

*   **5.1. `BrowserConfig` Usage:**
    *   Typically instantiated once and passed to the `AsyncWebCrawler` constructor via its `config` parameter.
    *   `browser_config = BrowserConfig(headless=False)`
    *   `crawler = AsyncWebCrawler(config=browser_config)`
    *   It defines the global browser settings that will be used for all subsequent crawl operations unless overridden by `CrawlerRunConfig` on a per-run basis.
*   **5.2. `CrawlerRunConfig` Usage:**
    *   Passed to the `arun()` or `arun_many()` methods of `AsyncWebCrawler`.
    *   `run_config = CrawlerRunConfig(screenshot=True, cache_mode=CacheMode.BYPASS)`
    *   `result = await crawler.arun(url="https://example.com", config=run_config)`
    *   Allows for fine-grained control over individual crawl requests, overriding global settings from `BrowserConfig` or `AsyncWebCrawler`'s defaults where applicable (e.g., `user_agent`, `proxy_config`, `cache_mode`).
*   **5.3. `LLMConfig` Usage:**
    *   Instantiated and passed to LLM-based extraction strategies (e.g., `LLMExtractionStrategy`) or content filters (`LLMContentFilter`) during their initialization.
    *   `llm_conf = LLMConfig(provider="openai/gpt-4o-mini", api_token="sk-...")`
    *   `extraction_strategy = LLMExtractionStrategy(llm_config=llm_conf, schema=my_schema)`
*   **5.4. `GeolocationConfig` and `ProxyConfig` Usage:**
    *   `GeolocationConfig` is typically instantiated and assigned to the `geolocation` parameter of `CrawlerRunConfig`.
        *   `geo_conf = GeolocationConfig(latitude=34.0522, longitude=-118.2437)`
        *   `run_config = CrawlerRunConfig(geolocation=geo_conf)`
    *   `ProxyConfig` can be assigned to the `proxy_config` parameter of `BrowserConfig` (for a global proxy applied to all contexts) or `CrawlerRunConfig` (for a proxy specific to a single crawl run).
        *   `proxy_conf = ProxyConfig(server="http://myproxy:8080")`
        *   `browser_config = BrowserConfig(proxy_config=proxy_conf)` (global)
        *   `run_config = CrawlerRunConfig(proxy_config=proxy_conf)` (per-run)
*   **5.5. `HTTPCrawlerConfig` Usage:**
    *   Used when the `crawler_strategy` for `AsyncWebCrawler` is set to `AsyncHTTPCrawlerStrategy` (for non-browser-based HTTP requests).
    *   `http_conf = HTTPCrawlerConfig(method="POST", json={"key": "value"})`
    *   `http_strategy = AsyncHTTPCrawlerStrategy(http_crawler_config=http_conf)`
    *   `crawler = AsyncWebCrawler(crawler_strategy=http_strategy)`
    *   Alternatively, parameters like `method`, `data`, `json` can be passed directly to `arun()` when using `AsyncHTTPCrawlerStrategy` if they are part of the `CrawlerRunConfig`.