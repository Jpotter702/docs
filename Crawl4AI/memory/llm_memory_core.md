# Detailed Outline for crawl4ai - core Component

**Target Document Type:** memory
**Target Output Filename Suggestion:** `llm_memory_core.md`
**Library Version Context:** 0.6.3
**Outline Generation Date:** 2025-05-24
---

## 1. Introduction to Core Components
    * 1.1. Purpose: Provides the foundational classes, configurations, and data models for web crawling and scraping operations within the `crawl4ai` library.
    * 1.2. Key Functionalities:
        *   Orchestration of asynchronous web crawling (`AsyncWebCrawler`).
        *   Configuration of browser behavior and specific crawl runs (`BrowserConfig`, `CrawlerRunConfig`).
        *   Standardized data structures for crawl results and associated data (`CrawlResult`, `Media`, `Links`, etc.).
        *   Strategies for fetching web content (`AsyncPlaywrightCrawlerStrategy`, `AsyncHTTPCrawlerStrategy`).
        *   Management of browser instances and sessions (`BrowserManager`, `ManagedBrowser`).
        *   Asynchronous logging (`AsyncLogger`).
    * 1.3. Relationship with other `crawl4ai` components:
        *   The `core` component serves as the foundation upon which specialized strategies (e.g., PDF processing, Markdown generation, content extraction, chunking, content filtering) are built and integrated.

## 2. Main Class: `AsyncWebCrawler`
    * 2.1. Purpose: The primary class for orchestrating asynchronous web crawling operations. It manages browser instances (via a `BrowserManager`), applies crawling strategies, and processes HTML content to produce structured results.
    * 2.2. Initialization (`__init__`)
        * 2.2.1. Signature:
            ```python
            def __init__(
                self,
                crawler_strategy: Optional[AsyncCrawlerStrategy] = None,
                config: Optional[BrowserConfig] = None,
                base_directory: str = str(os.getenv("CRAWL4_AI_BASE_DIRECTORY", Path.home())),
                thread_safe: bool = False,
                logger: Optional[AsyncLoggerBase] = None,
                **kwargs,
            ):
            ```
        * 2.2.2. Parameters:
            * `crawler_strategy (Optional[AsyncCrawlerStrategy])`: The strategy to use for fetching web content. If `None`, defaults to `AsyncPlaywrightCrawlerStrategy` initialized with `config` and `logger`.
            * `config (Optional[BrowserConfig])`: Configuration object for browser settings. If `None`, a default `BrowserConfig()` is created.
            * `base_directory (str)`: The base directory for storing crawl4ai related files, such as cache and logs. Defaults to `os.getenv("CRAWL4_AI_BASE_DIRECTORY", Path.home())`.
            * `thread_safe (bool)`: If `True`, uses an `asyncio.Lock` for thread-safe operations, particularly relevant for `arun_many`. Default: `False`.
            * `logger (Optional[AsyncLoggerBase])`: An instance of a logger. If `None`, a default `AsyncLogger` is initialized using `base_directory` and `config.verbose`.
            * `**kwargs`: Additional keyword arguments, primarily for backward compatibility, passed to the `AsyncPlaywrightCrawlerStrategy` if `crawler_strategy` is not provided.
    * 2.3. Key Public Attributes/Properties:
        * `browser_config (BrowserConfig)`: Read-only. The browser configuration object used by the crawler.
        * `crawler_strategy (AsyncCrawlerStrategy)`: Read-only. The active crawling strategy instance.
        * `logger (AsyncLoggerBase)`: Read-only. The logger instance used by the crawler.
        * `ready (bool)`: Read-only. `True` if the crawler has been started and is ready to perform crawl operations, `False` otherwise.
    * 2.4. Lifecycle Methods:
        * 2.4.1. `async start() -> AsyncWebCrawler`:
            * Purpose: Asynchronously initializes the crawler strategy (e.g., launches the browser). This must be called before `arun` or `arun_many` if the crawler is not used as an asynchronous context manager.
            * Returns: The `AsyncWebCrawler` instance (`self`).
        * 2.4.2. `async close() -> None`:
            * Purpose: Asynchronously closes the crawler strategy and cleans up resources (e.g., closes the browser). This should be called if `start()` was used explicitly.
        * 2.4.3. `async __aenter__() -> AsyncWebCrawler`:
            * Purpose: Entry point for asynchronous context management. Calls `self.start()`.
            * Returns: The `AsyncWebCrawler` instance (`self`).
        * 2.4.4. `async __aexit__(exc_type, exc_val, exc_tb) -> None`:
            * Purpose: Exit point for asynchronous context management. Calls `self.close()`.
    * 2.5. Primary Crawl Methods:
        * 2.5.1. `async arun(url: str, config: Optional[CrawlerRunConfig] = None, **kwargs) -> RunManyReturn`:
            * Purpose: Performs a single crawl operation for the given URL or raw HTML content.
            * Parameters:
                * `url (str)`: The URL to crawl (e.g., "http://example.com", "file:///path/to/file.html") or raw HTML content prefixed with "raw:" (e.g., "raw:<html>...</html>").
                * `config (Optional[CrawlerRunConfig])`: Configuration for this specific crawl run. If `None`, a default `CrawlerRunConfig()` is used.
                * `**kwargs`: Additional parameters passed to the underlying `aprocess_html` method, can be used to override settings in `config`.
            * Returns: `RunManyReturn` (which resolves to `CrawlResultContainer` containing a single `CrawlResult`).
        * 2.5.2. `async arun_many(urls: List[str], config: Optional[CrawlerRunConfig] = None, dispatcher: Optional[BaseDispatcher] = None, **kwargs) -> RunManyReturn`:
            * Purpose: Crawls multiple URLs concurrently using a specified or default dispatcher strategy.
            * Parameters:
                * `urls (List[str])`: A list of URLs to crawl.
                * `config (Optional[CrawlerRunConfig])`: Configuration applied to all crawl runs in this batch.
                * `dispatcher (Optional[BaseDispatcher])`: The dispatcher strategy to manage concurrent crawls. Defaults to `MemoryAdaptiveDispatcher`.
                * `**kwargs`: Additional parameters passed to the underlying `arun` method for each URL.
            * Returns: `RunManyReturn`. If `config.stream` is `True`, returns an `AsyncGenerator[CrawlResult, None]`. Otherwise, returns a `CrawlResultContainer` (list-like) of `CrawlResult` objects.
    * 2.6. Internal Processing Method (User-Facing Effects):
        * 2.6.1. `async aprocess_html(url: str, html: str, extracted_content: Optional[str], config: CrawlerRunConfig, screenshot_data: Optional[str], pdf_data: Optional[bytes], verbose: bool, **kwargs) -> CrawlResult`:
            * Purpose: Processes the fetched HTML content. This method is called internally by `arun` after content is fetched (either from a live crawl or cache). It applies scraping strategies, content filtering, and Markdown generation based on the `config`.
            * Parameters:
                * `url (str)`: The URL of the content being processed.
                * `html (str)`: The raw HTML content.
                * `extracted_content (Optional[str])`: Pre-extracted content from a previous step or cache.
                * `config (CrawlerRunConfig)`: Configuration for this processing run.
                * `screenshot_data (Optional[str])`: Base64 encoded screenshot data, if available.
                * `pdf_data (Optional[bytes])`: PDF data, if available.
                * `verbose (bool)`: Verbosity setting for logging during processing.
                * `**kwargs`: Additional parameters, including `is_raw_html` and `redirected_url`.
            * Returns: A `CrawlResult` object containing the processed data.

## 3. Core Configuration Objects

    * 3.1. Class `BrowserConfig` (from `crawl4ai.async_configs`)
        * 3.1.1. Purpose: Configures the browser instance launched by Playwright, including its type, mode, display settings, proxy, user agent, and persistent storage options.
        * 3.1.2. Initialization (`__init__`)
            * Signature:
            ```python
            def __init__(
                self,
                browser_type: str = "chromium",
                headless: bool = True,
                browser_mode: str = "dedicated",
                use_managed_browser: bool = False,
                cdp_url: Optional[str] = None,
                use_persistent_context: bool = False,
                user_data_dir: Optional[str] = None,
                channel: Optional[str] = "chromium", # Note: 'channel' from code, outline had 'chrome_channel'
                proxy: Optional[str] = None, # Note: 'proxy' from code, outline had 'proxy_config' for this level
                proxy_config: Optional[Union[ProxyConfig, dict, None]] = None,
                viewport_width: int = 1080,
                viewport_height: int = 600,
                viewport: Optional[dict] = None,
                accept_downloads: bool = False,
                downloads_path: Optional[str] = None,
                storage_state: Optional[Union[str, dict, None]] = None,
                ignore_https_errors: bool = True,
                java_script_enabled: bool = True,
                sleep_on_close: bool = False,
                verbose: bool = True,
                cookies: Optional[list] = None,
                headers: Optional[dict] = None,
                user_agent: str = "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 Chrome/116.0.0.0 Safari/537.36",
                user_agent_mode: str = "",
                user_agent_generator_config: Optional[dict] = None, # Note: 'user_agent_generator_config' from code
                text_mode: bool = False,
                light_mode: bool = False,
                extra_args: Optional[list] = None,
                debugging_port: int = 9222,
                host: str = "localhost",
            ):
            ```
            * Key Parameters:
                * `browser_type (str)`: Type of browser to launch ("chromium", "firefox", "webkit"). Default: "chromium".
                * `headless (bool)`: Whether to run the browser in headless mode. Default: `True`.
                * `browser_mode (str)`: How the browser should be initialized ("builtin", "dedicated", "cdp", "docker"). Default: "dedicated".
                * `use_managed_browser (bool)`: Whether to launch the browser using a managed approach (e.g., via CDP). Default: `False`.
                * `cdp_url (Optional[str])`: URL for Chrome DevTools Protocol endpoint. Default: `None`.
                * `use_persistent_context (bool)`: Use a persistent browser context (profile). Default: `False`.
                * `user_data_dir (Optional[str])`: Path to user data directory for persistent sessions. Default: `None`.
                * `channel (Optional[str])`: Browser channel (e.g., "chromium", "chrome", "msedge"). Default: "chromium".
                * `proxy (Optional[str])`: Simple proxy server URL string.
                * `proxy_config (Optional[Union[ProxyConfig, dict, None]])`: Detailed proxy configuration object or dictionary. Takes precedence over `proxy`.
                * `viewport_width (int)`: Default viewport width. Default: `1080`.
                * `viewport_height (int)`: Default viewport height. Default: `600`.
                * `viewport (Optional[dict])`: Dictionary to set viewport dimensions, overrides `viewport_width` and `viewport_height` if set (e.g., `{"width": 1920, "height": 1080}`). Default: `None`.
                * `accept_downloads (bool)`: Whether to allow file downloads. Default: `False`.
                * `downloads_path (Optional[str])`: Directory to store downloaded files. Default: `None`.
                * `storage_state (Optional[Union[str, dict, None]])`: Path to a file or a dictionary containing browser storage state (cookies, localStorage). Default: `None`.
                * `ignore_https_errors (bool)`: Ignore HTTPS certificate errors. Default: `True`.
                * `java_script_enabled (bool)`: Enable JavaScript execution. Default: `True`.
                * `user_agent (str)`: Custom User-Agent string. Default: "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 Chrome/116.0.0.0 Safari/537.36".
                * `user_agent_mode (str)`: Mode for generating User-Agent (e.g., "random"). Default: `""` (uses provided `user_agent`).
                * `user_agent_generator_config (Optional[dict])`: Configuration for User-Agent generation if `user_agent_mode` is active. Default: `{}`.
                * `text_mode (bool)`: If `True`, disables images and rich content for faster loading. Default: `False`.
                * `light_mode (bool)`: Disables certain background features for performance. Default: `False`.
                * `extra_args (Optional[list])`: Additional command-line arguments for the browser. Default: `None` (resolves to `[]`).
                * `debugging_port (int)`: Port for browser debugging protocol. Default: `9222`.
                * `host (str)`: Host for browser debugging protocol. Default: "localhost".
        * 3.1.3. Key Public Methods:
            * `clone(**kwargs) -> BrowserConfig`: Creates a new `BrowserConfig` instance as a copy of the current one, with specified keyword arguments overriding existing values.
            * `to_dict() -> dict`: Returns a dictionary representation of the configuration object's attributes.
            * `dump() -> dict`: Serializes the configuration object to a JSON-serializable dictionary, including nested objects.
            * `static load(data: dict) -> BrowserConfig`: Deserializes a `BrowserConfig` instance from a dictionary (previously created by `dump`).
            * `static from_kwargs(kwargs: dict) -> BrowserConfig`: Creates a `BrowserConfig` instance directly from a dictionary of keyword arguments.

    * 3.2. Class `CrawlerRunConfig` (from `crawl4ai.async_configs`)
        * 3.2.1. Purpose: Specifies settings for an individual crawl operation initiated by `arun()` or `arun_many()`. These settings can override or augment the global `BrowserConfig`.
        * 3.2.2. Initialization (`__init__`)
            * Signature:
            ```python
            def __init__(
                self,
                # Content Processing Parameters
                word_count_threshold: int = MIN_WORD_THRESHOLD,
                extraction_strategy: Optional[ExtractionStrategy] = None,
                chunking_strategy: ChunkingStrategy = RegexChunking(),
                markdown_generator: MarkdownGenerationStrategy = DefaultMarkdownGenerator(),
                only_text: bool = False,
                css_selector: Optional[str] = None,
                target_elements: Optional[List[str]] = None,
                excluded_tags: Optional[list] = None,
                excluded_selector: Optional[str] = None,
                keep_data_attributes: bool = False,
                keep_attrs: Optional[list] = None,
                remove_forms: bool = False,
                prettify: bool = False,
                parser_type: str = "lxml",
                scraping_strategy: ContentScrapingStrategy = None, # Will default to WebScrapingStrategy
                proxy_config: Optional[Union[ProxyConfig, dict, None]] = None,
                proxy_rotation_strategy: Optional[ProxyRotationStrategy] = None,
                # Browser Location and Identity Parameters
                locale: Optional[str] = None,
                timezone_id: Optional[str] = None,
                geolocation: Optional[GeolocationConfig] = None,
                # SSL Parameters
                fetch_ssl_certificate: bool = False,
                # Caching Parameters
                cache_mode: CacheMode = CacheMode.BYPASS,
                session_id: Optional[str] = None,
                bypass_cache: bool = False, # Legacy
                disable_cache: bool = False, # Legacy
                no_cache_read: bool = False, # Legacy
                no_cache_write: bool = False, # Legacy
                shared_data: Optional[dict] = None,
                # Page Navigation and Timing Parameters
                wait_until: str = "domcontentloaded",
                page_timeout: int = PAGE_TIMEOUT,
                wait_for: Optional[str] = None,
                wait_for_timeout: Optional[int] = None,
                wait_for_images: bool = False,
                delay_before_return_html: float = 0.1,
                mean_delay: float = 0.1,
                max_range: float = 0.3,
                semaphore_count: int = 5,
                # Page Interaction Parameters
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
                # Media Handling Parameters
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
                # Link and Domain Handling Parameters
                exclude_social_media_domains: Optional[list] = None, # Note: 'exclude_social_media_domains' from code
                exclude_external_links: bool = False,
                exclude_social_media_links: bool = False,
                exclude_domains: Optional[list] = None,
                exclude_internal_links: bool = False,
                # Debugging and Logging Parameters
                verbose: bool = True,
                log_console: bool = False,
                # Network and Console Capturing Parameters
                capture_network_requests: bool = False,
                capture_console_messages: bool = False,
                # Connection Parameters (for HTTPCrawlerStrategy)
                method: str = "GET",
                stream: bool = False,
                url: Optional[str] = None,
                # Robots.txt Handling
                check_robots_txt: bool = False,
                # User Agent Parameters
                user_agent: Optional[str] = None,
                user_agent_mode: Optional[str] = None,
                user_agent_generator_config: Optional[dict] = None, # Note: 'user_agent_generator_config' from code
                # Deep Crawl Parameters
                deep_crawl_strategy: Optional[DeepCrawlStrategy] = None,
                # Experimental Parameters
                experimental: Optional[Dict[str, Any]] = None,
            ):
            ```
            * Key Parameters:
                * `word_count_threshold (int)`: Minimum word count for a content block to be considered. Default: `MIN_WORD_THRESHOLD` (200).
                * `extraction_strategy (Optional[ExtractionStrategy])`: Strategy for structured data extraction (e.g., `LLMExtractionStrategy`, `JsonCssExtractionStrategy`). Default: `None` (falls back to `NoExtractionStrategy`).
                * `chunking_strategy (ChunkingStrategy)`: Strategy for splitting content into chunks before extraction. Default: `RegexChunking()`.
                * `markdown_generator (MarkdownGenerationStrategy)`: Strategy for converting HTML to Markdown. Default: `DefaultMarkdownGenerator()`.
                * `cache_mode (CacheMode)`: Caching behavior for this run. Default: `CacheMode.BYPASS`.
                * `session_id (Optional[str])`: ID for session persistence (reusing browser tabs/contexts). Default: `None`.
                * `js_code (Optional[Union[str, List[str]]])`: JavaScript code snippets to execute on the page. Default: `None`.
                * `wait_for (Optional[str])`: CSS selector or JS condition (prefixed with "js:") to wait for before proceeding. Default: `None`.
                * `page_timeout (int)`: Timeout for page operations (e.g., navigation) in milliseconds. Default: `PAGE_TIMEOUT` (60000ms).
                * `screenshot (bool)`: If `True`, capture a screenshot of the page. Default: `False`.
                * `pdf (bool)`: If `True`, generate a PDF of the page. Default: `False`.
                * `capture_mhtml (bool)`: If `True`, capture an MHTML snapshot of the page. Default: `False`.
                * `exclude_external_links (bool)`: If `True`, exclude external links from results. Default: `False`.
                * `stream (bool)`: If `True` (used with `arun_many`), results are yielded as an `AsyncGenerator`. Default: `False`.
                * `check_robots_txt (bool)`: If `True`, crawler will check and respect `robots.txt` rules. Default: `False`.
                * `user_agent (Optional[str])`: Override the browser's User-Agent for this specific run.
        * 3.2.3. Key Public Methods:
            * `clone(**kwargs) -> CrawlerRunConfig`: Creates a new `CrawlerRunConfig` instance as a copy of the current one, with specified keyword arguments overriding existing values.
            * `to_dict() -> dict`: Returns a dictionary representation of the configuration object's attributes.
            * `dump() -> dict`: Serializes the configuration object to a JSON-serializable dictionary, including nested objects.
            * `static load(data: dict) -> CrawlerRunConfig`: Deserializes a `CrawlerRunConfig` instance from a dictionary (previously created by `dump`).
            * `static from_kwargs(kwargs: dict) -> CrawlerRunConfig`: Creates a `CrawlerRunConfig` instance directly from a dictionary of keyword arguments.

    * 3.3. Supporting Configuration Objects (from `crawl4ai.async_configs`)
        * 3.3.1. Class `GeolocationConfig`
            * Purpose: Defines geolocation (latitude, longitude, accuracy) to be emulated by the browser.
            * Initialization (`__init__`):
                ```python
                def __init__(
                    self,
                    latitude: float,
                    longitude: float,
                    accuracy: Optional[float] = 0.0
                ):
                ```
            * Parameters:
                * `latitude (float)`: Latitude coordinate (e.g., 37.7749).
                * `longitude (float)`: Longitude coordinate (e.g., -122.4194).
                * `accuracy (Optional[float])`: Accuracy in meters. Default: `0.0`.
            * Methods:
                * `static from_dict(geo_dict: Dict) -> GeolocationConfig`: Creates an instance from a dictionary.
                * `to_dict() -> Dict`: Converts the instance to a dictionary.
                * `clone(**kwargs) -> GeolocationConfig`: Creates a copy with updated values.
        * 3.3.2. Class `ProxyConfig`
            * Purpose: Defines the settings for a single proxy server, including server address, authentication credentials, and optional IP.
            * Initialization (`__init__`):
                ```python
                def __init__(
                    self,
                    server: str,
                    username: Optional[str] = None,
                    password: Optional[str] = None,
                    ip: Optional[str] = None,
                ):
                ```
            * Parameters:
                * `server (str)`: Proxy server URL (e.g., "http://127.0.0.1:8080", "socks5://user:pass@host:port").
                * `username (Optional[str])`: Username for proxy authentication.
                * `password (Optional[str])`: Password for proxy authentication.
                * `ip (Optional[str])`: Optional IP address associated with the proxy for verification.
            * Methods:
                * `static from_string(proxy_str: str) -> ProxyConfig`: Creates an instance from a string (e.g., "ip:port:username:password" or "ip:port").
                * `static from_dict(proxy_dict: Dict) -> ProxyConfig`: Creates an instance from a dictionary.
                * `static from_env(env_var: str = "PROXIES") -> List[ProxyConfig]`: Loads a list of proxies from a comma-separated environment variable.
                * `to_dict() -> Dict`: Converts the instance to a dictionary.
                * `clone(**kwargs) -> ProxyConfig`: Creates a copy with updated values.
        * 3.3.3. Class `HTTPCrawlerConfig`
            * Purpose: Configuration for the `AsyncHTTPCrawlerStrategy`, specifying HTTP method, headers, data/JSON payload, and redirect/SSL verification behavior.
            * Initialization (`__init__`):
                ```python
                def __init__(
                    self,
                    method: str = "GET",
                    headers: Optional[Dict[str, str]] = None,
                    data: Optional[Dict[str, Any]] = None,
                    json: Optional[Dict[str, Any]] = None,
                    follow_redirects: bool = True,
                    verify_ssl: bool = True,
                ):
                ```
            * Parameters:
                * `method (str)`: HTTP method (e.g., "GET", "POST"). Default: "GET".
                * `headers (Optional[Dict[str, str]])`: Dictionary of HTTP request headers. Default: `None`.
                * `data (Optional[Dict[str, Any]])`: Dictionary of form data to send in the request body. Default: `None`.
                * `json (Optional[Dict[str, Any]])`: JSON data to send in the request body. Default: `None`.
                * `follow_redirects (bool)`: Whether to automatically follow HTTP redirects. Default: `True`.
                * `verify_ssl (bool)`: Whether to verify SSL certificates. Default: `True`.
            * Methods:
                * `static from_kwargs(kwargs: dict) -> HTTPCrawlerConfig`: Creates an instance from keyword arguments.
                * `to_dict() -> dict`: Converts config to a dictionary.
                * `clone(**kwargs) -> HTTPCrawlerConfig`: Creates a copy with updated values.
                * `dump() -> dict`: Serializes the config to a dictionary.
                * `static load(data: dict) -> HTTPCrawlerConfig`: Deserializes from a dictionary.
        * 3.3.4. Class `LLMConfig`
            * Purpose: Configures settings for interacting with Large Language Models, including provider choice, API credentials, and generation parameters.
            * Initialization (`__init__`):
                ```python
                def __init__(
                    self,
                    provider: str = DEFAULT_PROVIDER,
                    api_token: Optional[str] = None,
                    base_url: Optional[str] = None,
                    temperature: Optional[float] = None,
                    max_tokens: Optional[int] = None,
                    top_p: Optional[float] = None,
                    frequency_penalty: Optional[float] = None,
                    presence_penalty: Optional[float] = None,
                    stop: Optional[List[str]] = None,
                    n: Optional[int] = None,
                ):
                ```
            * Key Parameters:
                * `provider (str)`: Name of the LLM provider (e.g., "openai/gpt-4o", "ollama/llama3.3", "groq/llama3-8b-8192"). Default: `DEFAULT_PROVIDER` (from `crawl4ai.config`).
                * `api_token (Optional[str])`: API token for the LLM provider. If prefixed with "env:", it reads from the specified environment variable (e.g., "env:OPENAI_API_KEY"). If not provided, it attempts to load from default environment variables based on the provider.
                * `base_url (Optional[str])`: Custom base URL for the LLM API endpoint.
                * `temperature (Optional[float])`: Sampling temperature for generation.
                * `max_tokens (Optional[int])`: Maximum number of tokens to generate.
                * `top_p (Optional[float])`: Nucleus sampling parameter.
                * `frequency_penalty (Optional[float])`: Penalty for token frequency.
                * `presence_penalty (Optional[float])`: Penalty for token presence.
                * `stop (Optional[List[str]])`: List of stop sequences for generation.
                * `n (Optional[int])`: Number of completions to generate.
            * Methods:
                * `static from_kwargs(kwargs: dict) -> LLMConfig`: Creates an instance from keyword arguments.
                * `to_dict() -> dict`: Converts config to a dictionary.
                * `clone(**kwargs) -> LLMConfig`: Creates a copy with updated values.

## 4. Core Data Models (Results & Payloads from `crawl4ai.models`)

    * 4.1. Class `CrawlResult(BaseModel)`
        * Purpose: A Pydantic model representing the comprehensive result of a single crawl and processing operation.
        * Key Fields:
            * `url (str)`: The final URL that was crawled (after any redirects).
            * `html (str)`: The raw HTML content fetched from the URL.
            * `success (bool)`: `True` if the crawl operation (fetching and initial processing) was successful, `False` otherwise.
            * `cleaned_html (Optional[str])`: HTML content after sanitization and removal of unwanted tags/attributes as per configuration. Default: `None`.
            * `_markdown (Optional[MarkdownGenerationResult])`: (Private Attribute) Holds the `MarkdownGenerationResult` object if Markdown generation was performed. Use the `markdown` property to access. Default: `None`.
            * `markdown (Optional[Union[str, MarkdownGenerationResult]])`: (Property) Provides access to Markdown content. Behaves as a string (raw markdown) by default but allows access to `MarkdownGenerationResult` attributes (e.g., `result.markdown.fit_markdown`).
            * `extracted_content (Optional[str])`: JSON string representation of structured data extracted by an `ExtractionStrategy`. Default: `None`.
            * `media (Media)`: An object containing lists of `MediaItem` for images, videos, audio, and extracted tables. Default: `Media()`.
            * `links (Links)`: An object containing lists of `Link` for internal and external hyperlinks found on the page. Default: `Links()`.
            * `downloaded_files (Optional[List[str]])`: A list of file paths if any files were downloaded during the crawl. Default: `None`.
            * `js_execution_result (Optional[Dict[str, Any]])`: The result of any JavaScript code executed on the page. Default: `None`.
            * `screenshot (Optional[str])`: Base64 encoded string of the page screenshot, if `screenshot=True` was set. Default: `None`.
            * `pdf (Optional[bytes])`: Raw bytes of the PDF generated from the page, if `pdf=True` was set. Default: `None`.
            * `mhtml (Optional[str])`: MHTML snapshot of the page, if `capture_mhtml=True` was set. Default: `None`.
            * `metadata (Optional[dict])`: Dictionary of metadata extracted from the page (e.g., title, description, OpenGraph tags, Twitter card data). Default: `None`.
            * `error_message (Optional[str])`: A message describing the error if `success` is `False`. Default: `None`.
            * `session_id (Optional[str])`: The session ID used for this crawl, if applicable. Default: `None`.
            * `response_headers (Optional[dict])`: HTTP response headers from the server. Default: `None`.
            * `status_code (Optional[int])`: HTTP status code of the response. Default: `None`.
            * `ssl_certificate (Optional[SSLCertificate])`: Information about the SSL certificate if `fetch_ssl_certificate=True`. Default: `None`.
            * `dispatch_result (Optional[DispatchResult])`: Metadata about the task execution from the dispatcher (e.g., timings, memory usage). Default: `None`.
            * `redirected_url (Optional[str])`: The original URL if the request was redirected. Default: `None`.
            * `network_requests (Optional[List[Dict[str, Any]]])`: List of captured network requests if `capture_network_requests=True`. Default: `None`.
            * `console_messages (Optional[List[Dict[str, Any]]])`: List of captured browser console messages if `capture_console_messages=True`. Default: `None`.
        * Methods:
            * `model_dump(*args, **kwargs)`: Serializes the `CrawlResult` model to a dictionary, ensuring the `_markdown` private attribute is correctly handled and included as "markdown" in the output if present.

    * 4.2. Class `MarkdownGenerationResult(BaseModel)`
        * Purpose: A Pydantic model that holds various forms of Markdown generated from HTML content.
        * Fields:
            * `raw_markdown (str)`: The basic, direct conversion of HTML to Markdown.
            * `markdown_with_citations (str)`: Markdown content with inline citations (e.g., [^1^]) and a references section.
            * `references_markdown (str)`: The Markdown content for the "References" section, listing all cited links.
            * `fit_markdown (Optional[str])`: Markdown generated specifically from content deemed "relevant" by a content filter (like `PruningContentFilter` or `LLMContentFilter`), if such a filter was applied. Default: `None`.
            * `fit_html (Optional[str])`: The filtered HTML content that was used to generate `fit_markdown`. Default: `None`.
        * Methods:
            * `__str__(self) -> str`: Returns `self.raw_markdown` when the object is cast to a string.

    * 4.3. Class `ScrapingResult(BaseModel)`
        * Purpose: A Pydantic model representing a standardized output from content scraping strategies.
        * Fields:
            * `cleaned_html (str)`: The primary sanitized and processed HTML content.
            * `success (bool)`: Indicates if the scraping operation was successful.
            * `media (Media)`: A `Media` object containing extracted images, videos, audio, and tables.
            * `links (Links)`: A `Links` object containing extracted internal and external links.
            * `metadata (Dict[str, Any])`: A dictionary of metadata extracted from the page (e.g., title, description).

    * 4.4. Class `MediaItem(BaseModel)`
        * Purpose: A Pydantic model representing a generic media item like an image, video, or audio file.
        * Fields:
            * `src (Optional[str])`: The source URL of the media item. Default: `""`.
            * `data (Optional[str])`: Base64 encoded data for inline media. Default: `""`.
            * `alt (Optional[str])`: Alternative text for the media item (e.g., image alt text). Default: `""`.
            * `desc (Optional[str])`: A description or surrounding text related to the media item. Default: `""`.
            * `score (Optional[int])`: A relevance or importance score, if calculated by a strategy. Default: `0`.
            * `type (str)`: The type of media (e.g., "image", "video", "audio"). Default: "image".
            * `group_id (Optional[int])`: An identifier to group related media variants (e.g., different resolutions of the same image from a srcset). Default: `0`.
            * `format (Optional[str])`: The detected file format (e.g., "jpeg", "png", "mp4"). Default: `None`.
            * `width (Optional[int])`: The width of the media item in pixels, if available. Default: `None`.

    * 4.5. Class `Link(BaseModel)`
        * Purpose: A Pydantic model representing an extracted hyperlink.
        * Fields:
            * `href (Optional[str])`: The URL (href attribute) of the link. Default: `""`.
            * `text (Optional[str])`: The anchor text of the link. Default: `""`.
            * `title (Optional[str])`: The title attribute of the link, if present. Default: `""`.
            * `base_domain (Optional[str])`: The base domain extracted from the `href`. Default: `""`.

    * 4.6. Class `Media(BaseModel)`
        * Purpose: A Pydantic model that acts as a container for lists of different types of media items found on a page.
        * Fields:
            * `images (List[MediaItem])`: A list of `MediaItem` objects representing images. Default: `[]`.
            * `videos (List[MediaItem])`: A list of `MediaItem` objects representing videos. Default: `[]`.
            * `audios (List[MediaItem])`: A list of `MediaItem` objects representing audio files. Default: `[]`.
            * `tables (List[Dict])`: A list of dictionaries, where each dictionary represents an extracted HTML table with keys like "headers", "rows", "caption", "summary". Default: `[]`.

    * 4.7. Class `Links(BaseModel)`
        * Purpose: A Pydantic model that acts as a container for lists of internal and external links.
        * Fields:
            * `internal (List[Link])`: A list of `Link` objects considered internal to the crawled site. Default: `[]`.
            * `external (List[Link])`: A list of `Link` objects pointing to external sites. Default: `[]`.

    * 4.8. Class `AsyncCrawlResponse(BaseModel)`
        * Purpose: A Pydantic model representing the raw response from a crawler strategy's `crawl` method. This data is then processed further to create a `CrawlResult`.
        * Fields:
            * `html (str)`: The raw HTML content of the page.
            * `response_headers (Dict[str, str])`: A dictionary of HTTP response headers.
            * `js_execution_result (Optional[Dict[str, Any]])`: The result from any JavaScript code executed on the page. Default: `None`.
            * `status_code (int)`: The HTTP status code of the response.
            * `screenshot (Optional[str])`: Base64 encoded screenshot data, if captured. Default: `None`.
            * `pdf_data (Optional[bytes])`: Raw PDF data, if captured. Default: `None`.
            * `mhtml_data (Optional[str])`: MHTML snapshot data, if captured. Default: `None`.
            * `downloaded_files (Optional[List[str]])`: A list of local file paths for any files downloaded during the crawl. Default: `None`.
            * `ssl_certificate (Optional[SSLCertificate])`: SSL certificate information for the site. Default: `None`.
            * `redirected_url (Optional[str])`: The original URL requested if the final URL is a result of redirection. Default: `None`.
            * `network_requests (Optional[List[Dict[str, Any]]])`: Captured network requests if enabled. Default: `None`.
            * `console_messages (Optional[List[Dict[str, Any]]])`: Captured console messages if enabled. Default: `None`.

    * 4.9. Class `TokenUsage(BaseModel)`
        * Purpose: A Pydantic model to track token usage statistics for interactions with Large Language Models.
        * Fields:
            * `completion_tokens (int)`: Number of tokens used for the LLM's completion/response. Default: `0`.
            * `prompt_tokens (int)`: Number of tokens used for the input prompt to the LLM. Default: `0`.
            * `total_tokens (int)`: Total number of tokens used (prompt + completion). Default: `0`.
            * `completion_tokens_details (Optional[dict])`: Provider-specific detailed breakdown of completion tokens. Default: `None`.
            * `prompt_tokens_details (Optional[dict])`: Provider-specific detailed breakdown of prompt tokens. Default: `None`.

    * 4.10. Class `SSLCertificate(dict)` (from `crawl4ai.ssl_certificate`)
        * Purpose: Represents an SSL certificate's information, behaving like a dictionary for direct JSON serialization and easy access to its fields.
        * Key Fields (accessed as dictionary keys):
            * `subject (dict)`: Dictionary of subject fields (e.g., `{"CN": "example.com", "O": "Example Inc."}`).
            * `issuer (dict)`: Dictionary of issuer fields.
            * `version (int)`: Certificate version number.
            * `serial_number (str)`: Certificate serial number (hexadecimal string).
            * `not_before (str)`: Validity start date and time (ASN.1/UTC format string, e.g., "YYYYMMDDHHMMSSZ").
            * `not_after (str)`: Validity end date and time (ASN.1/UTC format string).
            * `fingerprint (str)`: SHA-256 fingerprint of the certificate (lowercase hex string).
            * `signature_algorithm (str)`: The algorithm used to sign the certificate (e.g., "sha256WithRSAEncryption").
            * `raw_cert (str)`: Base64 encoded string of the raw DER-encoded certificate.
            * `extensions (List[dict])`: A list of dictionaries, each representing a certificate extension with "name" and "value" keys.
        * Static Methods:
            * `from_url(url: str, timeout: int = 10) -> Optional[SSLCertificate]`: Fetches the SSL certificate from the given URL and returns an `SSLCertificate` instance, or `None` on failure.
        * Instance Methods:
            * `to_json(filepath: Optional[str] = None) -> Optional[str]`: Exports the certificate information as a JSON string. If `filepath` is provided, writes to the file and returns `None`.
            * `to_pem(filepath: Optional[str] = None) -> Optional[str]`: Exports the certificate in PEM format as a string. If `filepath` is provided, writes to the file and returns `None`.
            * `to_der(filepath: Optional[str] = None) -> Optional[bytes]`: Exports the raw certificate in DER format as bytes. If `filepath` is provided, writes to the file and returns `None`.
        * Example:
            ```python
            # Assuming 'cert' is an SSLCertificate instance
            # print(cert["subject"]["CN"])
            # cert.to_pem("my_cert.pem")
            ```

    * 4.11. Class `DispatchResult(BaseModel)`
        * Purpose: Contains metadata about a task's execution when processed by a dispatcher (e.g., in `arun_many`).
        * Fields:
            * `task_id (str)`: A unique identifier for the dispatched task.
            * `memory_usage (float)`: Memory usage (in MB) recorded during the task's execution.
            * `peak_memory (float)`: Peak memory usage (in MB) recorded during the task's execution.
            * `start_time (Union[datetime, float])`: The start time of the task (can be a `datetime` object or a Unix timestamp float).
            * `end_time (Union[datetime, float])`: The end time of the task.
            * `error_message (str)`: Any error message if the task failed during dispatch or execution. Default: `""`.

    * 4.12. `CrawlResultContainer(Generic[CrawlResultT])`
        * Purpose: A generic container for `CrawlResult` objects, primarily used as the return type for `arun_many` when `stream=False`. It behaves like a list, allowing iteration, indexing, and length checking.
        * Methods:
            * `__iter__(self)`: Allows iteration over the contained `CrawlResult` objects.
            * `__getitem__(self, index)`: Allows accessing `CrawlResult` objects by index.
            * `__len__(self)`: Returns the number of `CrawlResult` objects contained.
            * `__repr__(self)`: Provides a string representation of the container.
        * Attribute:
            * `_results (List[CrawlResultT])`: The internal list holding the `CrawlResult` objects.

    * 4.13. `RunManyReturn` (Type Alias from `crawl4ai.models`)
        * Purpose: A type alias defining the possible return types for the `arun_many` method of `AsyncWebCrawler`.
        * Definition: `Union[CrawlResultContainer[CrawlResult], AsyncGenerator[CrawlResult, None]]`
            * This means `arun_many` will return a `CrawlResultContainer` (a list-like object of all `CrawlResult` instances) if `CrawlerRunConfig.stream` is `False` (the default).
            * It will return an `AsyncGenerator` yielding individual `CrawlResult` instances if `CrawlerRunConfig.stream` is `True`.

## 5. Core Crawler Strategies (from `crawl4ai.async_crawler_strategy`)

    * 5.1. Abstract Base Class `AsyncCrawlerStrategy(ABC)`
        * Purpose: Defines the common interface that all asynchronous crawler strategies must implement. This allows `AsyncWebCrawler` to use different fetching mechanisms (e.g., Playwright, HTTP requests) interchangeably.
        * Initialization (`__init__`):
            ```python
            def __init__(self, browser_config: BrowserConfig, logger: AsyncLoggerBase):
            ```
            * Parameters:
                * `browser_config (BrowserConfig)`: The browser configuration to be used by the strategy.
                * `logger (AsyncLoggerBase)`: The logger instance for logging strategy-specific events.
        * Key Abstract Methods (must be implemented by concrete subclasses):
            * `async crawl(self, url: str, config: CrawlerRunConfig) -> AsyncCrawlResponse`:
                * Purpose: Fetches the content from the given URL according to the `config`.
                * Returns: An `AsyncCrawlResponse` object containing the raw fetched data.
            * `async __aenter__(self)`:
                * Purpose: Asynchronous context manager entry, typically for initializing resources (e.g., launching a browser).
            * `async __aexit__(self, exc_type, exc_val, exc_tb)`:
                * Purpose: Asynchronous context manager exit, for cleaning up resources.
        * Key Concrete Methods (available to all strategies):
            * `set_custom_headers(self, headers: dict) -> None`:
                * Purpose: Sets custom HTTP headers to be used by the strategy for subsequent requests.
            * `update_user_agent(self, user_agent: str) -> None`:
                * Purpose: Updates the User-Agent string used by the strategy.
            * `set_hook(self, hook_name: str, callback: Callable) -> None`:
                * Purpose: Registers a callback function for a specific hook point in the crawling lifecycle.
            * `async_run_hook(self, hook_name: str, *args, **kwargs) -> Any`:
                * Purpose: Executes a registered hook with the given arguments.
            * `async_get_default_context(self) -> BrowserContext`:
                * Purpose: Retrieves the default browser context (Playwright specific, might raise `NotImplementedError` in non-Playwright strategies).
            * `async_create_new_page(self, context: BrowserContext) -> Page`:
                * Purpose: Creates a new page within a given browser context (Playwright specific).
            * `async_get_page(self, url: str, config: CrawlerRunConfig, session_id: Optional[str]) -> Tuple[Page, BrowserContext]`:
                * Purpose: Gets an existing page/context for a session or creates a new one (Playwright specific, managed by `BrowserManager`).
            * `async_close_page(self, page: Page, session_id: Optional[str]) -> None`:
                * Purpose: Closes a page, potentially keeping the associated context/session alive (Playwright specific).
            * `async_kill_session(self, session_id: str) -> None`:
                * Purpose: Kills (closes) a specific browser session, including its page and context (Playwright specific).

    * 5.2. Class `AsyncPlaywrightCrawlerStrategy(AsyncCrawlerStrategy)`
        * Purpose: The default crawler strategy, using Playwright to control a web browser for fetching and interacting with web pages. It supports complex JavaScript execution and provides hooks for various stages of the crawl.
        * Initialization (`__init__`):
            ```python
            def __init__(
                self,
                browser_config: Optional[BrowserConfig] = None,
                logger: Optional[AsyncLoggerBase] = None,
                browser_manager: Optional[BrowserManager] = None
            ):
            ```
            * Parameters:
                * `browser_config (Optional[BrowserConfig])`: Browser configuration. Defaults to a new `BrowserConfig()` if not provided.
                * `logger (Optional[AsyncLoggerBase])`: Logger instance. Defaults to a new `AsyncLogger()`.
                * `browser_manager (Optional[BrowserManager])`: An instance of `BrowserManager` to manage browser lifecycles and contexts. If `None`, a new `BrowserManager` is created internally.
        * Key Overridden/Implemented Methods:
            * `async crawl(self, url: str, config: CrawlerRunConfig) -> AsyncCrawlResponse`:
                * Purpose: Implements the crawling logic using Playwright. It navigates to the URL, executes JavaScript if specified, waits for conditions, captures screenshots/PDFs if requested, and returns the page content and other metadata.
            * `async aprocess_html(self, url: str, html: str, config: CrawlerRunConfig, **kwargs) -> CrawlResult`:
                * Purpose: (Note: While `AsyncWebCrawler` calls this, the default implementation is in `AsyncPlaywrightCrawlerStrategy` for convenience, acting as a bridge to the scraping strategy.) Processes the fetched HTML to produce a `CrawlResult`. This involves using the `scraping_strategy` from the `config` (defaults to `WebScrapingStrategy`) to clean HTML, extract media/links, and then uses the `markdown_generator` to produce Markdown.
        * Specific Public Methods:
            * `async_create_new_context(self, config: Optional[CrawlerRunConfig] = None) -> BrowserContext`:
                * Purpose: Creates a new Playwright `BrowserContext` based on the global `BrowserConfig` and optional overrides from `CrawlerRunConfig`.
            * `async_setup_context_default(self, context: BrowserContext, config: Optional[CrawlerRunConfig] = None) -> None`:
                * Purpose: Applies default settings to a `BrowserContext`, such as viewport size, user agent, custom headers, locale, timezone, and geolocation, based on `BrowserConfig` and `CrawlerRunConfig`.
            * `async_setup_context_hooks(self, context: BrowserContext, config: CrawlerRunConfig) -> None`:
                * Purpose: Sets up event listeners on the context for capturing network requests and console messages if `config.capture_network_requests` or `config.capture_console_messages` is `True`.
            * `async_handle_storage_state(self, context: BrowserContext, config: CrawlerRunConfig) -> None`:
                * Purpose: Loads cookies and localStorage from a `storage_state` file or dictionary (specified in `BrowserConfig` or `CrawlerRunConfig`) into the given `BrowserContext`.
        * Hooks (Callable via `set_hook(hook_name, callback)` and executed by `async_run_hook`):
            * `on_browser_created`: Called after the Playwright browser instance is launched or connected. Callback receives `(browser, **kwargs)`.
            * `on_page_context_created`: Called after a new Playwright `BrowserContext` and `Page` are created. Callback receives `(page, context, **kwargs)`.
            * `before_goto`: Called just before `page.goto(url)` is executed. Callback receives `(page, context, url, **kwargs)`.
            * `after_goto`: Called after `page.goto(url)` completes successfully. Callback receives `(page, context, url, response, **kwargs)`.
            * `on_user_agent_updated`: Called when the User-Agent string is updated for a context. Callback receives `(page, context, user_agent, **kwargs)`.
            * `on_execution_started`: Called when `js_code` execution begins on a page. Callback receives `(page, context, **kwargs)`.
            * `before_retrieve_html`: Called just before the final HTML content is retrieved from the page. Callback receives `(page, context, **kwargs)`.
            * `before_return_html`: Called just before the `AsyncCrawlResponse` is returned by the `crawl()` method of the strategy. Callback receives `(page, context, html_content, **kwargs)`.

    * 5.3. Class `AsyncHTTPCrawlerStrategy(AsyncCrawlerStrategy)`
        * Purpose: A lightweight crawler strategy that uses direct HTTP requests (via `httpx`) instead of a full browser. Suitable for static sites or when JavaScript execution is not needed.
        * Initialization (`__init__`):
            ```python
            def __init__(self, http_config: Optional[HTTPCrawlerConfig] = None, logger: Optional[AsyncLoggerBase] = None):
            ```
            * Parameters:
                * `http_config (Optional[HTTPCrawlerConfig])`: Configuration for HTTP requests (method, headers, data, etc.). Defaults to a new `HTTPCrawlerConfig()`.
                * `logger (Optional[AsyncLoggerBase])`: Logger instance. Defaults to a new `AsyncLogger()`.
        * Key Overridden/Implemented Methods:
            * `async crawl(self, url: str, http_config: Optional[HTTPCrawlerConfig] = None, **kwargs) -> AsyncCrawlResponse`:
                * Purpose: Fetches content from the URL using an HTTP GET or POST request via `httpx`. Does not execute JavaScript. Returns an `AsyncCrawlResponse` with HTML, status code, and headers. Screenshot, PDF, and MHTML capabilities are not available with this strategy.

## 6. Browser Management (from `crawl4ai.browser_manager`)

    * 6.1. Class `BrowserManager`
        * Purpose: Manages the lifecycle of Playwright browser instances and their contexts. It handles launching/connecting to browsers, creating new contexts with specific configurations, managing sessions for page reuse, and cleaning up resources.
        * Initialization (`__init__`):
            ```python
            def __init__(self, browser_config: BrowserConfig, logger: Optional[AsyncLoggerBase] = None):
            ```
            * Parameters:
                * `browser_config (BrowserConfig)`: The global browser configuration settings.
                * `logger (Optional[AsyncLoggerBase])`: Logger instance for browser management events.
        * Key Methods:
            * `async start() -> None`: Initializes the Playwright instance and launches or connects to the browser based on `browser_config` (e.g., launches a new browser instance or connects to an existing CDP endpoint via `ManagedBrowser`).
            * `async create_browser_context(self, crawlerRunConfig: Optional[CrawlerRunConfig] = None) -> playwright.async_api.BrowserContext`: Creates a new browser context. If `crawlerRunConfig` is provided, its settings (e.g., locale, viewport, proxy) can override the global `BrowserConfig`.
            * `async setup_context(self, context: playwright.async_api.BrowserContext, crawlerRunConfig: Optional[CrawlerRunConfig] = None, is_default: bool = False) -> None`: Applies various settings to a given browser context, including headers, cookies, viewport, geolocation, permissions, and storage state, based on `BrowserConfig` and `CrawlerRunConfig`.
            * `async get_page(self, crawlerRunConfig: CrawlerRunConfig) -> Tuple[playwright.async_api.Page, playwright.async_api.BrowserContext]`: Retrieves an existing page and context for a given `session_id` (if present in `crawlerRunConfig` and the session is active) or creates a new page and context. Manages context reuse based on a signature derived from `CrawlerRunConfig` to ensure contexts with different core settings (like proxy, locale) are isolated.
            * `async kill_session(self, session_id: str) -> None`: Closes the page and browser context associated with the given `session_id`, effectively ending that session.
            * `async close() -> None`: Closes all managed browser contexts and the main browser instance.

    * 6.2. Class `ManagedBrowser`
        * Purpose: Manages the lifecycle of a single, potentially persistent, browser process. It's used when `BrowserConfig.use_managed_browser` is `True` or `BrowserConfig.use_persistent_context` is `True`. It handles launching the browser with a specific user data directory and connecting via CDP.
        * Initialization (`__init__`):
            ```python
            def __init__(
                self,
                browser_type: str = "chromium",
                user_data_dir: Optional[str] = None,
                headless: bool = False,
                logger=None,
                host: str = "localhost",
                debugging_port: int = 9222,
                cdp_url: Optional[str] = None, # Added as per code_analysis
                browser_config: Optional[BrowserConfig] = None # Added as per code_analysis
            ):
            ```
            * Parameters:
                * `browser_type (str)`: "chromium", "firefox", or "webkit". Default: "chromium".
                * `user_data_dir (Optional[str])`: Path to the user data directory for the browser profile. If `None`, a temporary directory might be created.
                * `headless (bool)`: Whether to launch the browser in headless mode. Default: `False` (typically for managed/persistent scenarios).
                * `logger`: Logger instance.
                * `host (str)`: Host for the debugging port. Default: "localhost".
                * `debugging_port (int)`: Port for the Chrome DevTools Protocol. Default: `9222`.
                * `cdp_url (Optional[str])`: If provided, attempts to connect to an existing browser at this CDP URL instead of launching a new one.
                * `browser_config (Optional[BrowserConfig])`: The `BrowserConfig` object providing overall browser settings.
        * Key Methods:
            * `async start() -> str`: Starts the browser process (if not connecting to an existing `cdp_url`). If a new browser is launched, it uses the specified `user_data_dir` and `debugging_port`.
            * Returns: The CDP endpoint URL (e.g., "http://localhost:9222").
            * `async cleanup() -> None`: Terminates the browser process (if launched by this instance) and removes any temporary user data directory created by it.
        * Static Methods:
            * `async create_profile(cls, browser_config: Optional[BrowserConfig] = None, profile_name: Optional[str] = None, logger=None) -> str`:
                * Purpose: Launches a browser instance with a new or existing user profile, allowing interactive setup (e.g., manual login, cookie acceptance). The browser remains open until the user closes it.
                * Parameters:
                    * `browser_config (Optional[BrowserConfig])`: Optional browser configuration to use.
                    * `profile_name (Optional[str])`: Name for the profile. If `None`, a default name is used.
                    * `logger`: Logger instance.
                * Returns: The path to the created/used user data directory, which can then be passed to `BrowserConfig.user_data_dir`.
            * `list_profiles(cls) -> List[str]`:
                * Purpose: Lists the names of all browser profiles stored in the default Crawl4AI profiles directory (`~/.crawl4ai/profiles`).
                * Returns: A list of profile name strings.
            * `delete_profile(cls, profile_name_or_path: str) -> bool`:
                * Purpose: Deletes a browser profile either by its name (if in the default directory) or by its full path.
                * Returns: `True` if deletion was successful, `False` otherwise.

    * 6.3. Function `clone_runtime_state(src: BrowserContext, dst: BrowserContext, crawlerRunConfig: Optional[CrawlerRunConfig] = None, browserConfig: Optional[BrowserConfig] = None) -> None`
        * Purpose: Asynchronously copies runtime state (cookies, localStorage, session storage) from a source `BrowserContext` to a destination `BrowserContext`. Can also apply headers and geolocation from `CrawlerRunConfig` or `BrowserConfig` to the destination context.
        * Parameters:
            * `src (BrowserContext)`: The source browser context.
            * `dst (BrowserContext)`: The destination browser context.
            * `crawlerRunConfig (Optional[CrawlerRunConfig])`: Optional run configuration to apply to `dst`.
            * `browserConfig (Optional[BrowserConfig])`: Optional browser configuration to apply to `dst`.

## 7. Proxy Rotation Strategies (from `crawl4ai.proxy_strategy`)

    * 7.1. Abstract Base Class `ProxyRotationStrategy(ABC)`
        * Purpose: Defines the interface for strategies that provide a sequence of proxy configurations, enabling proxy rotation.
        * Abstract Methods:
            * `async get_next_proxy(self) -> Optional[ProxyConfig]`:
                * Purpose: Asynchronously retrieves the next `ProxyConfig` from the strategy.
                * Returns: A `ProxyConfig` object or `None` if no more proxies are available or an error occurs.
            * `add_proxies(self, proxies: List[ProxyConfig]) -> None`:
                * Purpose: Adds a list of `ProxyConfig` objects to the strategy's pool of proxies.

    * 7.2. Class `RoundRobinProxyStrategy(ProxyRotationStrategy)`
        * Purpose: A simple proxy rotation strategy that cycles through a list of provided proxies in a round-robin fashion.
        * Initialization (`__init__`):
            ```python
            def __init__(self, proxies: Optional[List[ProxyConfig]] = None):
            ```
            * Parameters:
                * `proxies (Optional[List[ProxyConfig]])`: An initial list of `ProxyConfig` objects. If `None`, the list is empty and proxies must be added via `add_proxies`.
        * Methods:
            * `add_proxies(self, proxies: List[ProxyConfig]) -> None`: Adds new `ProxyConfig` objects to the internal list of proxies and reinitializes the cycle.
            * `async get_next_proxy(self) -> Optional[ProxyConfig]`: Returns the next `ProxyConfig` from the list, cycling back to the beginning when the end is reached. Returns `None` if the list is empty.

## 8. Logging (from `crawl4ai.async_logger`)

    * 8.1. Abstract Base Class `AsyncLoggerBase(ABC)`
        * Purpose: Defines the basic interface for an asynchronous logger. Concrete implementations should provide methods for logging messages at different levels.
    * 8.2. Class `AsyncLogger(AsyncLoggerBase)`
        * Purpose: The default asynchronous logger for `crawl4ai`. It provides structured logging to both the console and optionally to a file, with customizable icons, colors, and verbosity levels.
        * Initialization (`__init__`):
            ```python
            def __init__(
                self,
                log_file: Optional[str] = None,
                verbose: bool = True,
                tag_width: int = 15, # outline had 10, code has 15
                icons: Optional[Dict[str, str]] = None,
                colors: Optional[Dict[LogLevel, LogColor]] = None, # Corrected type annotation
                log_level: LogLevel = LogLevel.INFO # Assuming LogLevel.INFO is a typical default
            ):
            ```
            * Parameters:
                * `log_file (Optional[str])`: Path to a file where logs should be written. If `None`, logs only to console.
                * `verbose (bool)`: If `True`, enables more detailed logging (DEBUG level). Default: `True`.
                * `tag_width (int)`: Width for the tag part of the log message. Default: `15`.
                * `icons (Optional[Dict[str, str]])`: Custom icons for different log tags.
                * `colors (Optional[Dict[LogLevel, LogColor]])`: Custom colors for different log levels.
                * `log_level (LogLevel)`: Minimum log level to output.
        * Key Methods (for logging):
            * `info(self, message: str, tag: Optional[str] = None, **params) -> None`: Logs an informational message.
            * `warning(self, message: str, tag: Optional[str] = None, **params) -> None`: Logs a warning message.
            * `error(self, message: str, tag: Optional[str] = None, **params) -> None`: Logs an error message.
            * `debug(self, message: str, tag: Optional[str] = None, **params) -> None`: Logs a debug message (only if `verbose=True` or `log_level` is DEBUG).
            * `url_status(self, url: str, success: bool, timing: float, tag: str = "FETCH", **params) -> None`: Logs the status of a URL fetch operation, including success/failure and timing.
            * `error_status(self, url: str, error: str, tag: str = "ERROR", **params) -> None`: Logs an error encountered for a specific URL.

## 9. Core Utility Functions (from `crawl4ai.async_configs`)
    * 9.1. `to_serializable_dict(obj: Any, ignore_default_value: bool = False) -> Dict`
        * Purpose: Recursively converts a Python object (often a Pydantic model or a dataclass instance used for configuration) into a dictionary that is safe for JSON serialization. It handles nested objects, enums, and basic types.
        * Parameters:
            * `obj (Any)`: The object to be serialized.
            * `ignore_default_value (bool)`: If `True`, fields whose current value is the same as their default value (if applicable, e.g., for Pydantic models) might be omitted from the resulting dictionary. Default: `False`.
        * Returns: `Dict` - A JSON-serializable dictionary representation of the object.
    * 9.2. `from_serializable_dict(data: Any) -> Any`
        * Purpose: Recursively reconstructs Python objects from a dictionary representation (typically one created by `to_serializable_dict`). It attempts to instantiate classes based on a "type" key in the dictionary if present.
        * Parameters:
            * `data (Any)`: The dictionary (or basic type) to be deserialized.
        * Returns: `Any` - The reconstructed Python object or the original data if no special deserialization rule applies.
    * 9.3. `is_empty_value(value: Any) -> bool`
        * Purpose: Checks if a given value is considered "empty" (e.g., `None`, an empty string, an empty list, an empty dictionary).
        * Returns: `bool` - `True` if the value is empty, `False` otherwise.

## 10. Enumerations (Key Enums used in Core)
    * 10.1. `CacheMode` (from `crawl4ai.cache_context`, defined in `crawl4ai.async_configs` as per provided code)
        * Purpose: Defines the caching behavior for crawl operations.
        * Members:
            * `ENABLE`: (Value: "enable") Normal caching behavior; read from cache if available, write to cache after fetching.
            * `DISABLE`: (Value: "disable") No caching at all; always fetch fresh content and do not write to cache.
            * `READ_ONLY`: (Value: "read_only") Only read from the cache; do not write new or updated content to the cache.
            * `WRITE_ONLY`: (Value: "write_only") Only write to the cache after fetching; do not read from the cache.
            * `BYPASS`: (Value: "bypass") Skip the cache entirely for this specific operation; fetch fresh content and do not write to cache. This is often the default for individual `CrawlerRunConfig` instances.
    * 10.2. `DisplayMode` (from `crawl4ai.models`, used by `CrawlerMonitor`)
        * Purpose: Defines the display mode for the `CrawlerMonitor`.
        * Members:
            * `DETAILED`: Shows detailed information for each task.
            * `AGGREGATED`: Shows summary statistics and overall progress.
    * 10.3. `CrawlStatus` (from `crawl4ai.models`, used by `CrawlStats`)
        * Purpose: Represents the status of a crawl task.
        * Members:
            * `QUEUED`: Task is waiting to be processed.
            * `IN_PROGRESS`: Task is currently being processed.
            * `COMPLETED`: Task finished successfully.
            * `FAILED`: Task failed.

## 11. Versioning
    * 11.1. Accessing Library Version:
        * The current version of the `crawl4ai` library can be accessed programmatically via the `__version__` attribute of the top-level `crawl4ai` package.
        * Example:
            ```python
            from crawl4ai import __version__ as crawl4ai_version
            print(f"Crawl4AI Version: {crawl4ai_version}")
            # Expected output based on provided code: Crawl4AI Version: 0.6.3
            ```

## 12. Basic Usage Examples

    * 12.1. Minimal Crawl:
        ```python
        import asyncio
        from crawl4ai import AsyncWebCrawler

        async def main():
            async with AsyncWebCrawler() as crawler:
                result = await crawler.arun(url="http://example.com")
                if result.success:
                    print("Markdown (first 300 chars):")
                    print(result.markdown.raw_markdown[:300]) # Accessing raw_markdown
                else:
                    print(f"Error: {result.error_message}")

        if __name__ == "__main__":
            asyncio.run(main())
        ```

    * 12.2. Crawl with Basic Configuration:
        ```python
        import asyncio
        from crawl4ai import AsyncWebCrawler, BrowserConfig, CrawlerRunConfig, CacheMode

        async def main():
            browser_cfg = BrowserConfig(headless=True, browser_type="firefox")
            run_cfg = CrawlerRunConfig(
                cache_mode=CacheMode.BYPASS,
                word_count_threshold=50
            )
            async with AsyncWebCrawler(config=browser_cfg) as crawler:
                result = await crawler.arun(url="http://example.com", config=run_cfg)
                if result.success:
                    print(f"Status Code: {result.status_code}")
                    print(f"Cleaned HTML length: {len(result.cleaned_html)}")
                else:
                    print(f"Error: {result.error_message}")
        
        if __name__ == "__main__":
            asyncio.run(main())
        ```

    * 12.3. Accessing Links and Images from Result:
        ```python
        import asyncio
        from crawl4ai import AsyncWebCrawler

        async def main():
            async with AsyncWebCrawler() as crawler:
                result = await crawler.arun(url="http://example.com")
                if result.success:
                    print(f"Found {len(result.links.internal)} internal links.")
                    if result.links.internal:
                        print(f"First internal link: {result.links.internal[0].href}")
                    
                    print(f"Found {len(result.media.images)} images.")
                    if result.media.images:
                        print(f"First image src: {result.media.images[0].src}")
                else:
                    print(f"Error: {result.error_message}")

        if __name__ == "__main__":
            asyncio.run(main())
        ```
```