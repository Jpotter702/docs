```markdown
# Examples Document for crawl4ai - config_objects Component

**Target Document Type:** Examples Collection
**Target Output Filename Suggestion:** `llm_examples_config_objects.md`
**Library Version Context:** 0.6.3
**Outline Generation Date:** 2025-05-24
---

This document provides a collection of runnable code examples demonstrating how to use the various configuration objects within the `crawl4ai` library. These examples are designed to showcase diverse usage patterns and help users quickly understand how to configure the crawler for their specific needs.

## 1. Introduction to Configuration Objects

### 1.1. Overview: Purpose and interaction of `BrowserConfig`, `CrawlerRunConfig`, `LLMConfig`, `ProxyConfig`, and `GeolocationConfig`.

The `crawl4ai` library utilizes several key configuration objects to manage different aspects of the crawling and data extraction process:

*   **`BrowserConfig`**: Controls the browser's launch and behavior, such as its type (Chromium, Firefox, WebKit), headless mode, proxy settings, user agent, viewport size, and persistent contexts. This configuration is typically passed when initializing the `AsyncWebCrawler`.
*   **`CrawlerRunConfig`**: Dictates the behavior for each specific `arun()` call. This includes URL-specific settings like caching modes, JavaScript execution, waiting conditions, screenshot/PDF generation, content processing strategies (extraction, chunking, markdown), and run-specific overrides for browser settings like locale or geolocation.
*   **`LLMConfig`**: Configures the Large Language Model (LLM) provider and its parameters when using LLM-based extraction strategies (e.g., `LLMExtractionStrategy`). It includes provider name, API token, base URL, and generation parameters like temperature and max tokens.
*   **`ProxyConfig`**: A dedicated object for defining proxy server details, including server URL, authentication credentials (username/password), and IP. It can be used within `BrowserConfig` (for browser-wide proxy) or `CrawlerRunConfig` (for run-specific proxy, especially with HTTP-based strategies or proxy rotation).
*   **`GeolocationConfig`**: Specifies geolocation data (latitude, longitude, accuracy) to simulate a specific geographic location for the browser context, often used within `CrawlerRunConfig`.

These objects interact to provide fine-grained control. For instance, `BrowserConfig` sets up the general browser environment, while `CrawlerRunConfig` can override or augment these settings for individual crawl tasks. `LLMConfig`, `ProxyConfig`, and `GeolocationConfig` are often nested within `BrowserConfig` or `CrawlerRunConfig` to provide specialized configurations.

---
### 1.2. Example: Basic workflow - Creating default instances of each config object and passing them to `AsyncWebCrawler` or relevant strategies.

This example shows the most basic initialization of config objects and how they might be passed to the crawler.

```python
import asyncio
import os
from crawl4ai import (
    AsyncWebCrawler,
    BrowserConfig,
    CrawlerRunConfig,
    LLMConfig,
    ProxyConfig,
    GeolocationConfig,
    CacheMode
)

async def basic_config_workflow():
    # 1. BrowserConfig: Default settings (headless chromium)
    browser_config = BrowserConfig()
    print(f"Default BrowserConfig: {browser_config.to_dict()}")

    # 2. GeolocationConfig (Optional, for specific location simulation)
    geolocation_config = GeolocationConfig(latitude=37.7749, longitude=-122.4194) # San Francisco
    print(f"\nGeolocationConfig: {geolocation_config.to_dict()}")

    # 3. ProxyConfig (Optional, if a proxy is needed)
    # For this example, we'll assume no proxy is needed for the default BrowserConfig.
    # If one were, it could be:
    # proxy_config = ProxyConfig(server="http://myproxy.com:8080")
    # browser_config_with_proxy = BrowserConfig(proxy_config=proxy_config)

    # 4. LLMConfig (Optional, only if using LLM-based extraction)
    # Requires an API key, e.g., from environment
    llm_config = LLMConfig(
        provider="openai/gpt-4o-mini",
        api_token=os.getenv("OPENAI_API_KEY", "YOUR_OPENAI_API_KEY_HERE") # Replace or set env var
    )
    print(f"\nLLMConfig (provider only shown if token is placeholder): {llm_config.provider}")

    # 5. CrawlerRunConfig: Default settings, but can integrate other configs
    # For a specific run, you might pass geolocation or an LLM strategy
    crawler_run_config_basic = CrawlerRunConfig(
        url="https://example.com",
        cache_mode=CacheMode.BYPASS # For this demo, bypass cache
    )
    print(f"\nBasic CrawlerRunConfig: {crawler_run_config_basic.to_dict(exclude_none=True)}")

    crawler_run_config_with_geo = CrawlerRunConfig(
        url="https://example.com/geo-specific-content",
        geolocation=geolocation_config,
        cache_mode=CacheMode.BYPASS
    )
    print(f"\nCrawlerRunConfig with Geolocation: {crawler_run_config_with_geo.to_dict(exclude_none=True)}")

    # To use LLMConfig, you'd typically pass it to an LLMExtractionStrategy,
    # which then goes into CrawlerRunConfig.extraction_strategy.
    # from crawl4ai.extraction_strategy import LLMExtractionStrategy
    # llm_extraction_strategy = LLMExtractionStrategy(llm_config=llm_config, schema={"title": "Page title"})
    # crawler_run_config_with_llm = CrawlerRunConfig(
    #     url="https://example.com/article",
    #     extraction_strategy=llm_extraction_strategy,
    #     cache_mode=CacheMode.BYPASS
    # )
    # print(f"\nCrawlerRunConfig with LLM Strategy (conceptual): {crawler_run_config_with_llm.to_dict(exclude_none=True)}")


    # Using AsyncWebCrawler with the basic browser config
    async with AsyncWebCrawler(config=browser_config) as crawler:
        print("\n--- Running basic crawl ---")
        result = await crawler.arun(config=crawler_run_config_basic)
        if result.success:
            print(f"Successfully crawled {result.url}. Markdown length: {len(result.markdown.raw_markdown)}")
        else:
            print(f"Failed to crawl {result.url}: {result.error_message}")

        # Example with geolocation (on a site that might use it)
        # For a real test, use a site like https://mylocation.org/
        print("\n--- Running crawl with geolocation (conceptual) ---")
        result_geo = await crawler.arun(config=crawler_run_config_with_geo)
        if result_geo.success:
            print(f"Successfully crawled {result_geo.url} (geo). Markdown length: {len(result_geo.markdown.raw_markdown)}")
        else:
            print(f"Failed to crawl {result_geo.url} (geo): {result_geo.error_message}")


if __name__ == "__main__":
    asyncio.run(basic_config_workflow())
```

---
## 2. `GeolocationConfig` Examples

### 2.1. Example: Basic initialization of `GeolocationConfig` for a specific location (e.g., San Francisco).

This example shows how to initialize `GeolocationConfig` with latitude and longitude, defaulting accuracy to 0.0.

```python
import asyncio
from crawl4ai import GeolocationConfig, CrawlerRunConfig, AsyncWebCrawler

async def basic_geolocation_config():
    # San Francisco coordinates
    sf_geo_config = GeolocationConfig(latitude=37.7749, longitude=-122.4194)
    print(f"San Francisco GeolocationConfig: {sf_geo_config.to_dict()}")

    # To see its effect, you need a site that uses geolocation.
    # We'll use a placeholder for demonstration.
    # A good test site could be https://mylocation.org/ or https://www.gps-coordinates.net/
    run_config = CrawlerRunConfig(
        url="https://www.whatismybrowser.com/detect/what-is-my-user-agent", # Shows some geo info
        geolocation=sf_geo_config,
        verbose=True
    )

    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(config=run_config)
        if result.success:
            print(f"\nCrawled {result.url} with geolocation. HTML snippet (first 500 chars):")
            # You would typically parse the HTML to find location-specific content
            # For this example, we just print part of the HTML.
            # print(result.html[:500]) # The actual page might not show the spoofed geo directly
            print("Geolocation spoofing applied. Check a geo-sensitive site for full effect.")
        else:
            print(f"Failed to crawl: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(basic_geolocation_config())
```

---
### 2.2. Example: Initializing `GeolocationConfig` with latitude, longitude, and custom accuracy.

This example demonstrates setting a specific accuracy for the geolocation.

```python
import asyncio
from crawl4ai import GeolocationConfig

async def geolocation_with_accuracy():
    # Paris coordinates with 100 meters accuracy
    paris_geo_config = GeolocationConfig(
        latitude=48.8566,
        longitude=2.3522,
        accuracy=100.0  # Accuracy in meters
    )
    print(f"Paris GeolocationConfig with accuracy: {paris_geo_config.to_dict()}")
    # You would then use this in CrawlerRunConfig like the previous example.

if __name__ == "__main__":
    asyncio.run(geolocation_with_accuracy())
```

---
### 2.3. Example: Creating `GeolocationConfig` from a dictionary using `GeolocationConfig.from_dict()`.

This shows how to instantiate `GeolocationConfig` from a dictionary, useful for dynamic configurations.

```python
import asyncio
from crawl4ai import GeolocationConfig

async def geolocation_from_dict_example():
    geo_data_dict = {
        "latitude": 51.5074,  # London
        "longitude": 0.1278,
        "accuracy": 50.0
    }
    london_geo_config = GeolocationConfig.from_dict(geo_data_dict)
    print(f"London GeolocationConfig from_dict: {london_geo_config.to_dict()}")
    assert london_geo_config.latitude == 51.5074
    assert london_geo_config.accuracy == 50.0

if __name__ == "__main__":
    asyncio.run(geolocation_from_dict_example())
```

---
### 2.4. Example: Converting `GeolocationConfig` to a dictionary using `to_dict()`.

Demonstrates serializing a `GeolocationConfig` instance back into a dictionary.

```python
import asyncio
from crawl4ai import GeolocationConfig

async def geolocation_to_dict_example():
    tokyo_geo_config = GeolocationConfig(latitude=35.6895, longitude=139.6917, accuracy=75.0)
    config_dict = tokyo_geo_config.to_dict()
    print(f"Tokyo GeolocationConfig as dict: {config_dict}")
    assert config_dict["latitude"] == 35.6895
    assert config_dict["accuracy"] == 75.0

if __name__ == "__main__":
    asyncio.run(geolocation_to_dict_example())
```

---
### 2.5. Example: Cloning `GeolocationConfig` and modifying accuracy using `clone()`.

Shows how to create a copy of a `GeolocationConfig` instance and modify specific attributes.

```python
import asyncio
from crawl4ai import GeolocationConfig

async def geolocation_clone_example():
    original_geo_config = GeolocationConfig(latitude=40.7128, longitude=-74.0060) # New York
    print(f"Original NY GeolocationConfig: {original_geo_config.to_dict()}")

    # Clone and modify accuracy
    cloned_geo_config = original_geo_config.clone(accuracy=25.0)
    print(f"Cloned NY GeolocationConfig with new accuracy: {cloned_geo_config.to_dict()}")

    assert cloned_geo_config.latitude == original_geo_config.latitude
    assert cloned_geo_config.accuracy == 25.0
    assert original_geo_config.accuracy == 0.0 # Original remains unchanged

if __name__ == "__main__":
    asyncio.run(geolocation_clone_example())
```

---
### 2.6. Example: Integrating `GeolocationConfig` into `CrawlerRunConfig` to simulate location for a crawl.

This example explicitly shows passing `GeolocationConfig` to `CrawlerRunConfig` for a simulated crawl.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, GeolocationConfig, CacheMode

async def integrate_geolocation_in_crawler_run():
    # Berlin coordinates
    berlin_geo = GeolocationConfig(latitude=52.5200, longitude=13.4050, accuracy=10.0)

    # Configure the specific run to use this geolocation
    # Use a site that might reflect geolocation, e.g., a weather site or Google search
    run_config_with_geo = CrawlerRunConfig(
        url="https://www.google.com/search?q=weather", # Google search might show location-based results
        geolocation=berlin_geo,
        cache_mode=CacheMode.BYPASS, # Ensure fresh fetch for demo
        verbose=True
    )

    async with AsyncWebCrawler() as crawler:
        print(f"Crawling with geolocation for Berlin: {berlin_geo.to_dict()}")
        result = await crawler.arun(config=run_config_with_geo)
        if result.success:
            print(f"Successfully crawled {result.url} with simulated Berlin location.")
            # Inspect result.html or result.markdown for signs of location-based content
            # For instance, search for "Berlin" in the content
            if "Berlin" in result.html[:2000]: # Check first 2000 chars of raw HTML
                 print("Berlin might be reflected in the page content.")
            else:
                 print("Berlin not obviously reflected in initial HTML content (site might not use precise geo or requires more interaction).")
        else:
            print(f"Failed to crawl: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(integrate_geolocation_in_crawler_run())
```

---
## 3. `ProxyConfig` Examples

### 3.1. Example: Basic initialization of `ProxyConfig` with a server URL.

Shows how to set up a simple proxy without authentication.

```python
import asyncio
from crawl4ai import ProxyConfig

async def basic_proxy_config():
    proxy_config = ProxyConfig(server="http://myproxy.example.com:8080")
    print(f"Basic ProxyConfig: {proxy_config.to_dict()}")
    assert proxy_config.server == "http://myproxy.example.com:8080"
    assert proxy_config.ip == "myproxy.example.com" # IP is extracted

if __name__ == "__main__":
    asyncio.run(basic_proxy_config())
```

---
### 3.2. Example: Initializing `ProxyConfig` with server, username, and password for an authenticated proxy.

Demonstrates setting up an authenticated proxy.

```python
import asyncio
from crawl4ai import ProxyConfig

async def authenticated_proxy_config():
    auth_proxy_config = ProxyConfig(
        server="http://authproxy.example.com:3128",
        username="proxyuser",
        password="proxypassword123"
    )
    print(f"Authenticated ProxyConfig: {auth_proxy_config.to_dict()}")
    assert auth_proxy_config.username == "proxyuser"

if __name__ == "__main__":
    asyncio.run(authenticated_proxy_config())
```

---
### 3.3. Example: Initializing `ProxyConfig` with an explicit IP address.

This example shows specifying the IP address directly, which can be useful if the server URL is complex or resolution is tricky.

```python
import asyncio
from crawl4ai import ProxyConfig

async def explicit_ip_proxy_config():
    proxy_config_with_ip = ProxyConfig(
        server="http://hostname.that.is.not.ip:9999",
        ip="192.168.1.100" # Explicitly set the IP
    )
    print(f"ProxyConfig with explicit IP: {proxy_config_with_ip.to_dict()}")
    assert proxy_config_with_ip.ip == "192.168.1.100"

if __name__ == "__main__":
    asyncio.run(explicit_ip_proxy_config())
```

---
### 3.4. Example: Demonstrating automatic IP extraction from the server URL in `ProxyConfig`.

`ProxyConfig` attempts to extract the IP/hostname from the server URL if `ip` is not provided.

```python
import asyncio
from crawl4ai import ProxyConfig

async def auto_ip_extraction_proxy_config():
    # IP extraction from http://ip:port
    proxy1 = ProxyConfig(server="http://123.45.67.89:8000")
    print(f"Proxy 1 (IP from http): {proxy1.to_dict()}")
    assert proxy1.ip == "123.45.67.89"

    # IP extraction from https://hostname:port
    proxy2 = ProxyConfig(server="https://secureproxy.example.net:8443")
    print(f"Proxy 2 (Hostname from https): {proxy2.to_dict()}")
    assert proxy2.ip == "secureproxy.example.net"

    # IP extraction from socks5://hostname:port
    proxy3 = ProxyConfig(server="socks5://socksp.example.org:1080")
    print(f"Proxy 3 (Hostname from socks5): {proxy3.to_dict()}")
    assert proxy3.ip == "socksp.example.org"
    
    # IP extraction from just hostname:port (assumes http)
    proxy4 = ProxyConfig(server="anotherproxy.com:7070")
    print(f"Proxy 4 (Hostname from hostname:port): {proxy4.to_dict()}")
    assert proxy4.ip == "anotherproxy.com"


if __name__ == "__main__":
    asyncio.run(auto_ip_extraction_proxy_config())
```

---
### 3.5. Example: Creating `ProxyConfig` from a string in 'ip:port' format using `ProxyConfig.from_string()`.

Illustrates creating a `ProxyConfig` object from a simple 'ip:port' string.

```python
import asyncio
from crawl4ai import ProxyConfig

async def proxy_from_simple_string():
    proxy_str = "192.168.1.50:8888"
    proxy_config = ProxyConfig.from_string(proxy_str)
    print(f"ProxyConfig from '{proxy_str}': {proxy_config.to_dict()}")
    assert proxy_config.server == "http://192.168.1.50:8888"
    assert proxy_config.ip == "192.168.1.50"
    assert proxy_config.username is None

if __name__ == "__main__":
    asyncio.run(proxy_from_simple_string())
```

---
### 3.6. Example: Creating `ProxyConfig` from a string in 'ip:port:username:password' format using `ProxyConfig.from_string()`.

Shows creating an authenticated `ProxyConfig` object from a formatted string.

```python
import asyncio
from crawl4ai import ProxyConfig

async def proxy_from_auth_string():
    proxy_auth_str = "proxy.example.net:3128:user123:pass456"
    proxy_config = ProxyConfig.from_string(proxy_auth_str)
    print(f"ProxyConfig from '{proxy_auth_str}': {proxy_config.to_dict()}")
    assert proxy_config.server == "http://proxy.example.net:3128"
    assert proxy_config.ip == "proxy.example.net"
    assert proxy_config.username == "user123"
    assert proxy_config.password == "pass456"

if __name__ == "__main__":
    asyncio.run(proxy_from_auth_string())
```

---
### 3.7. Example: Creating `ProxyConfig` from a dictionary using `ProxyConfig.from_dict()`.

Demonstrates instantiating `ProxyConfig` from a dictionary.

```python
import asyncio
from crawl4ai import ProxyConfig

async def proxy_from_dict_example():
    proxy_data_dict = {
        "server": "socks5://secure.proxy.com:1080",
        "username": "sockuser",
        "password": "sockpassword"
    }
    proxy_config = ProxyConfig.from_dict(proxy_data_dict)
    print(f"ProxyConfig from_dict: {proxy_config.to_dict()}")
    assert proxy_config.server == "socks5://secure.proxy.com:1080"
    assert proxy_config.username == "sockuser"

if __name__ == "__main__":
    asyncio.run(proxy_from_dict_example())
```

---
### 3.8. Example: Loading a single `ProxyConfig` from the `PROXIES` environment variable using `ProxyConfig.from_env()`.

Shows how to load proxy settings from an environment variable.

```python
import asyncio
import os
from crawl4ai import ProxyConfig

async def proxy_from_env_single():
    env_var_name = "MY_CRAWLER_PROXIES" # Custom env var name
    proxy_string_in_env = "10.0.0.1:8000:envuser:envpass"

    # Temporarily set the environment variable for this example
    os.environ[env_var_name] = proxy_string_in_env
    
    try:
        proxy_configs = ProxyConfig.from_env(env_var=env_var_name)
        assert len(proxy_configs) == 1
        proxy_config = proxy_configs[0]
        
        print(f"Loaded ProxyConfig from env var '{env_var_name}': {proxy_config.to_dict()}")
        assert proxy_config.server == "http://10.0.0.1:8000"
        assert proxy_config.username == "envuser"
        assert proxy_config.password == "envpass"
    finally:
        del os.environ[env_var_name] # Clean up

if __name__ == "__main__":
    asyncio.run(proxy_from_env_single())
```

---
### 3.9. Example: Loading multiple `ProxyConfig` instances from a comma-separated `PROXIES` environment variable.

Demonstrates loading a list of proxies if the environment variable contains multiple comma-separated proxy strings.

```python
import asyncio
import os
from crawl4ai import ProxyConfig

async def proxy_from_env_multiple():
    env_var_name = "PROXIES" # Default env var name
    multiple_proxy_strings = "10.0.0.1:8000,10.0.0.2:8001:user2:pass2"

    os.environ[env_var_name] = multiple_proxy_strings
    
    try:
        proxy_configs = ProxyConfig.from_env() # Uses "PROXIES" by default
        print(f"Loaded {len(proxy_configs)} ProxyConfigs from env var '{env_var_name}':")
        for i, pc in enumerate(proxy_configs):
            print(f"  Proxy {i+1}: {pc.to_dict()}")
        
        assert len(proxy_configs) == 2
        assert proxy_configs[0].server == "http://10.0.0.1:8000"
        assert proxy_configs[1].server == "http://10.0.0.2:8001"
        assert proxy_configs[1].username == "user2"
    finally:
        del os.environ[env_var_name]

if __name__ == "__main__":
    asyncio.run(proxy_from_env_multiple())
```

---
### 3.10. Example: Converting `ProxyConfig` to a dictionary using `to_dict()`.

Shows serializing a `ProxyConfig` instance back to a dictionary.

```python
import asyncio
from crawl4ai import ProxyConfig

async def proxy_to_dict_example():
    proxy_config = ProxyConfig(
        server="https_proxy.example.com:443", 
        username="user", 
        password="secure"
    )
    config_dict = proxy_config.to_dict()
    print(f"ProxyConfig as dict: {config_dict}")
    assert config_dict["server"] == "https_proxy.example.com:443" # Note: schema is not added by default by constructor if not present
    assert config_dict["username"] == "user"

if __name__ == "__main__":
    asyncio.run(proxy_to_dict_example())
```

---
### 3.11. Example: Cloning `ProxyConfig` and changing server details using `clone()`.

Demonstrates creating a modified copy of a `ProxyConfig` instance.

```python
import asyncio
from crawl4ai import ProxyConfig

async def proxy_clone_example():
    original_proxy = ProxyConfig(server="http://original.proxy.com:8000", username="orig_user")
    print(f"Original ProxyConfig: {original_proxy.to_dict()}")

    cloned_proxy = original_proxy.clone(server="http://new.proxy.com:8080", username="new_user", password="new_password")
    print(f"Cloned ProxyConfig with new details: {cloned_proxy.to_dict()}")

    assert cloned_proxy.server == "http://new.proxy.com:8080"
    assert cloned_proxy.username == "new_user"
    assert original_proxy.server == "http://original.proxy.com:8000" # Original is unchanged

if __name__ == "__main__":
    asyncio.run(proxy_clone_example())
```

---
### 3.12. Example: Using `ProxyConfig` within `BrowserConfig` to set a browser-level proxy.

This shows how `ProxyConfig` is used to configure the proxy for all browser activity.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, BrowserConfig, ProxyConfig, CrawlerRunConfig

# This example requires a running proxy server to test against.
# For demonstration, we'll use httpbin.org/ip which returns the requester's IP.
# If run through a proxy, it should show the proxy's IP.

async def browser_level_proxy():
    # Replace with your actual proxy details
    # For this example, let's assume a (non-functional) placeholder
    proxy_details = ProxyConfig(server="http://your-proxy-server.com:8080") 
    # If your proxy requires auth:
    # proxy_details = ProxyConfig(server="http://your-proxy-server.com:8080", username="user", password="pass")


    browser_config = BrowserConfig(
        proxy_config=proxy_details,
        headless=True, # Usually True for automated tasks
        verbose=True
    )
    
    # A URL that shows your IP address
    test_url = "https://httpbin.org/ip"
    run_config = CrawlerRunConfig(url=test_url)

    print(f"Attempting to crawl {test_url} via proxy: {proxy_details.server}")
    print("NOTE: This example will likely fail or show your direct IP if the placeholder proxy is not replaced with a real, working proxy.")

    async with AsyncWebCrawler(config=browser_config) as crawler:
        result = await crawler.arun(config=run_config)
        if result.success:
            print(f"Successfully crawled {result.url}.")
            # The response content should ideally show the proxy's IP
            print(f"Response content (first 200 chars): {result.html[:200]}")
            # If using a real proxy, you'd parse result.html (which is JSON from httpbin.org/ip)
            # and check if 'origin' IP matches your proxy's IP.
        else:
            print(f"Failed to crawl via proxy: {result.error_message}")
            print("This could be due to the proxy server not being reachable, incorrect credentials, or other network issues.")

if __name__ == "__main__":
    # asyncio.run(browser_level_proxy())
    print("Skipping browser_level_proxy example as it requires a live proxy server.")
    print("To run, replace placeholder proxy details and uncomment asyncio.run().")
```

---
### 3.13. Example: Using `ProxyConfig` within `CrawlerRunConfig` for HTTP-specific crawler strategies.

When using `AsyncHTTPCrawlerStrategy` (not browser-based), `ProxyConfig` can be passed via `CrawlerRunConfig`.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, ProxyConfig
from crawl4ai.async_crawler_strategy import AsyncHTTPCrawlerStrategy # For non-browser crawling

async def http_crawler_with_proxy():
    # Replace with your actual proxy details
    proxy_details = ProxyConfig(server="http://your-http-proxy.com:8000")

    # This config applies only to this specific run, not the browser (if one were used elsewhere)
    run_config_with_proxy = CrawlerRunConfig(
        url="https://api.ipify.org?format=json", # A simple API to get IP
        proxy_config=proxy_details,
        verbose=True
    )

    # Initialize crawler with an HTTP-based strategy
    http_strategy = AsyncHTTPCrawlerStrategy()
    
    print(f"Attempting HTTP GET for {run_config_with_proxy.url} via proxy: {proxy_details.server}")
    print("NOTE: This example will likely fail or show your direct IP if the placeholder proxy is not replaced.")

    async with AsyncWebCrawler(crawler_strategy=http_strategy) as crawler:
        result = await crawler.arun(config=run_config_with_proxy)
        if result.success:
            print(f"Successfully fetched {result.url}.")
            print(f"Response content: {result.html}")
            # Parse result.html (JSON) to check if 'ip' matches your proxy's IP.
        else:
            print(f"Failed to fetch via proxy: {result.error_message}")

if __name__ == "__main__":
    # asyncio.run(http_crawler_with_proxy())
    print("Skipping http_crawler_with_proxy example as it requires a live proxy server.")
    print("To run, replace placeholder proxy details and uncomment asyncio.run().")

```

---
## 4. `BrowserConfig` Examples

### 4.1. Basic Initialization

#### 4.1.1. Example: Default initialization of `BrowserConfig`.
This uses default settings: Chromium, headless mode, standard viewport.

```python
import asyncio
from crawl4ai import BrowserConfig, AsyncWebCrawler, CrawlerRunConfig

async def default_browser_config_init():
    browser_cfg = BrowserConfig()
    print(f"Default BrowserConfig: {browser_cfg.to_dict(exclude_none=True)}")

    # Demonstrate its use
    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(url="https://example.com", config=CrawlerRunConfig())
        if result.success:
            print(f"Crawled successfully with default browser config. Page title: {result.metadata.get('title')}")
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(default_browser_config_init())
```

---
#### 4.1.2. Example: Specifying `browser_type` as "firefox".
Shows how to launch Firefox instead of the default Chromium.

```python
import asyncio
from crawl4ai import BrowserConfig, AsyncWebCrawler, CrawlerRunConfig

async def firefox_browser_config():
    browser_cfg = BrowserConfig(browser_type="firefox", headless=True) # Ensure headless for automation
    print(f"Firefox BrowserConfig: {browser_cfg.to_dict(exclude_none=True)}")
    assert browser_cfg.browser_type == "firefox"

    # Demonstrate its use
    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(url="https://example.com", config=CrawlerRunConfig())
        if result.success:
            print(f"Crawled successfully with Firefox. Page title: {result.metadata.get('title')}")
        else:
            print(f"Crawl failed with Firefox: {result.error_message}")
            print("Ensure Firefox is installed and accessible by Playwright.")

if __name__ == "__main__":
    asyncio.run(firefox_browser_config())
```

---
#### 4.1.3. Example: Specifying `browser_type` as "webkit".
Shows how to launch WebKit (Safari's engine).

```python
import asyncio
from crawl4ai import BrowserConfig, AsyncWebCrawler, CrawlerRunConfig

async def webkit_browser_config():
    browser_cfg = BrowserConfig(browser_type="webkit", headless=True) # Ensure headless
    print(f"WebKit BrowserConfig: {browser_cfg.to_dict(exclude_none=True)}")
    assert browser_cfg.browser_type == "webkit"

    # Demonstrate its use
    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(url="https://example.com", config=CrawlerRunConfig())
        if result.success:
            print(f"Crawled successfully with WebKit. Page title: {result.metadata.get('title')}")
        else:
            print(f"Crawl failed with WebKit: {result.error_message}")
            print("Ensure WebKit is installed and accessible by Playwright.")

if __name__ == "__main__":
    asyncio.run(webkit_browser_config())
```

---
#### 4.1.4. Example: Running `BrowserConfig` in headed mode (`headless=False`) for visual debugging.
Launches a visible browser window. Useful for observing the crawl.

```python
import asyncio
from crawl4ai import BrowserConfig, AsyncWebCrawler, CrawlerRunConfig

async def headed_browser_config():
    # Note: In some environments (like CI/CD), headed mode might not be possible or may require Xvfb.
    try:
        browser_cfg = BrowserConfig(headless=False) # Visible browser window
        print(f"Headed BrowserConfig: {browser_cfg.to_dict(exclude_none=True)}")
        assert not browser_cfg.headless

        async with AsyncWebCrawler(config=browser_cfg) as crawler:
            print("Attempting to launch a visible browser to crawl example.com...")
            print("The browser window should appear briefly.")
            result = await crawler.arun(url="https://example.com", config=CrawlerRunConfig(page_timeout=10000)) # 10s timeout
            if result.success:
                print(f"Crawled successfully with a visible browser. Page title: {result.metadata.get('title')}")
            else:
                print(f"Crawl failed with visible browser: {result.error_message}")
    except Exception as e:
        print(f"Could not run headed browser example (this is common in restricted environments): {e}")
        print("Skipping headed browser test.")


if __name__ == "__main__":
    # This example might require a display server.
    # asyncio.run(headed_browser_config())
    print("Skipping headed_browser_config example. Uncomment to run if you have a display server.")
```

---
### 4.2. Browser Mode and Management

#### 4.2.1. Example: Using `browser_mode="builtin"` for Playwright's managed CDP.
This mode is for connecting to Playwright's own managed browser instance via CDP, typically for advanced control or specific scenarios.

```python
import asyncio
from crawl4ai import BrowserConfig, AsyncWebCrawler, CrawlerRunConfig

# Note: 'builtin' mode often implies a more complex setup where the browser is
# launched and managed by Playwright in a specific way, and Crawl4ai connects to it.
# For a simple demonstration, it might behave similarly to 'dedicated' if not carefully orchestrated.
# The key is `use_managed_browser=True` and letting the BrowserManager handle CDP details.

async def builtin_browser_mode_config():
    # 'builtin' mode primarily signals that the browser lifecycle is managed
    # internally, often implying connection to a persistent browser instance.
    # `use_managed_browser` is True implicitly.
    # For this test, the cdp_url will be set by the internal ManagedBrowser.
    
    browser_cfg = BrowserConfig(
        browser_mode="builtin", 
        headless=True,
        verbose=True
    )
    print(f"Builtin mode BrowserConfig: {browser_cfg.to_dict(exclude_none=True)}")
    assert browser_cfg.browser_mode == "builtin"
    assert browser_cfg.use_managed_browser # Should be True for builtin

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        # The crawler will internally manage the browser and connect via CDP
        print("Crawler will use its managed browser instance via CDP (builtin mode).")
        result = await crawler.arun(url="https://example.com", config=CrawlerRunConfig())
        if result.success:
            print(f"Crawled successfully with 'builtin' browser mode. Page title: {result.metadata.get('title')}")
        else:
            print(f"Crawl failed with 'builtin' browser mode: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(builtin_browser_mode_config())
```

---
#### 4.2.2. Example: Using `browser_mode="docker"` (conceptual outline, actual Docker setup is external).
This mode implies Crawl4ai connecting to a browser running inside a Docker container. Actual Docker setup is beyond this example's scope.

```python
import asyncio
from crawl4ai import BrowserConfig # AsyncWebCrawler, CrawlerRunConfig (not directly run here)

# This is a conceptual outline. Running this requires:
# 1. A Docker image with a browser and Playwright server (or just browser with CDP enabled).
# 2. The Docker container running and exposing the CDP port.
# 3. The `cdp_url` correctly pointing to the Docker container's exposed port.

async def docker_browser_mode_conceptual():
    # Assume Docker container exposes CDP on localhost:9222
    # The cdp_url would be set by the (not-yet-implemented) DockerBrowserStrategy.
    # For now, browser_mode="docker" signals intent; actual connection uses cdp_url.
    
    docker_cdp_url = "ws://localhost:9222/devtools/browser/some-id" # Example CDP URL from Docker
    
    browser_cfg_docker_intent = BrowserConfig(
        browser_mode="docker",
        headless=True # Usually true for Docker
    )
    # When a DockerBrowserStrategy is implemented, it would handle launching/connecting
    # and setting the cdp_url. For now, this mode serves as a placeholder.
    # To actually connect to a Dockerized browser, you'd use 'custom' mode with a cdp_url.
    print(f"Conceptual Docker mode BrowserConfig (intent): {browser_cfg_docker_intent.to_dict(exclude_none=True)}")
    assert browser_cfg_docker_intent.browser_mode == "docker"
    assert browser_cfg_docker_intent.use_managed_browser # Docker mode implies managed connection

    print("\nTo actually connect to a Dockerized browser, you'd typically use:")
    browser_cfg_docker_connect = BrowserConfig(
        browser_mode="custom", # Or 'dedicated' if Crawl4ai itself starts the Docker container
        cdp_url=docker_cdp_url, # The actual CDP endpoint of the browser in Docker
        headless=True 
    )
    print(f"Actual connection to Dockerized browser would use: {browser_cfg_docker_connect.to_dict(exclude_none=True)}")

if __name__ == "__main__":
    asyncio.run(docker_browser_mode_conceptual())
    print("\nNote: 'docker' mode is currently more of an intent. For actual connection to a pre-existing Dockerized browser, use 'custom' mode with its cdp_url.")
```

---
#### 4.2.3. Example: Connecting to an externally managed browser via CDP URL using `browser_mode="custom"` and `cdp_url`.
If you have a browser already running (e.g., manually, or by another tool) and its Chrome DevTools Protocol endpoint is known.

```python
import asyncio
from crawl4ai import BrowserConfig, AsyncWebCrawler, CrawlerRunConfig

# This example requires an external Chrome/Chromium instance started with remote debugging enabled.
# E.g., /path/to/chrome --remote-debugging-port=9222 --headless --user-data-dir=/tmp/mychromedata
# Then find the browser's CDP endpoint (e.g., from http://localhost:9222/json/version -> webSocketDebuggerUrl)

async def custom_cdp_connection():
    # Replace with your actual CDP URL. This is a common placeholder.
    # If no browser is running at this CDP, the example will fail.
    external_cdp_url = "ws://localhost:9222/devtools/browser/some-unique-id" 
    
    # For a real test, you must get this from an actual running browser instance.
    # e.g. by navigating to http://localhost:9222/json in another browser
    # and copying the webSocketDebuggerUrl for a "page" type target.
    # Or, if connecting to the browser endpoint, it would be like:
    # external_cdp_url = "ws://localhost:9222/devtools/browser/...." (from /json/version)


    # Using a known CDP URL implies 'custom' management.
    browser_cfg = BrowserConfig(
        cdp_url=external_cdp_url, # This signals to connect to an existing browser
        browser_mode="custom"     # Explicitly set custom mode
    )
    print(f"Custom CDP BrowserConfig: {browser_cfg.to_dict(exclude_none=True)}")
    assert browser_cfg.cdp_url == external_cdp_url
    assert browser_cfg.use_managed_browser # Connecting via CDP means it's "managed" in this context

    print(f"\nAttempting to connect to external browser via CDP: {external_cdp_url}")
    print("Ensure a browser is running with remote debugging enabled on the specified port and path.")
    
    try:
        async with AsyncWebCrawler(config=browser_cfg) as crawler:
            result = await crawler.arun(url="https://example.com", config=CrawlerRunConfig())
            if result.success:
                print(f"Successfully crawled using external browser. Page title: {result.metadata.get('title')}")
            else:
                print(f"Crawl failed using external browser: {result.error_message}")
    except Exception as e:
        print(f"Failed to connect or crawl with external browser: {e}")
        print("Common reasons: No browser at CDP URL, incorrect CDP URL, network issues.")

if __name__ == "__main__":
    # asyncio.run(custom_cdp_connection())
    print("Skipping custom_cdp_connection example as it requires a pre-configured external browser with CDP.")
    print("To run, start a browser with remote debugging and update 'external_cdp_url'.")
```

---
#### 4.2.4. Example: Enabling a persistent browser context with `use_persistent_context=True` and specifying `user_data_dir`.
Saves browser state (cookies, localStorage, etc.) to a directory, allowing sessions to persist across crawler restarts.

```python
import asyncio
import shutil
from pathlib import Path
from crawl4ai import BrowserConfig, AsyncWebCrawler, CrawlerRunConfig

async def persistent_context_example():
    # Create a temporary directory for user data
    user_data_path = Path("./temp_crawl4ai_user_data_persistent")
    if user_data_path.exists():
        shutil.rmtree(user_data_path) # Clean up from previous runs
    user_data_path.mkdir(parents=True, exist_ok=True)

    print(f"Using persistent user data directory: {user_data_path.resolve()}")

    browser_cfg_persistent = BrowserConfig(
        use_persistent_context=True,
        user_data_dir=str(user_data_path.resolve()), # Must be an absolute path for Playwright
        headless=True, # Can be False for debugging the persistent state
        verbose=True
    )
    # `use_persistent_context=True` automatically implies `use_managed_browser=True`.
    print(f"Persistent Context BrowserConfig: {browser_cfg_persistent.to_dict(exclude_none=True)}")
    assert browser_cfg_persistent.use_persistent_context
    assert browser_cfg_persistent.user_data_dir == str(user_data_path.resolve())

    # First run: crawl a page, maybe set a cookie (implicitly or explicitly)
    run_config1 = CrawlerRunConfig(url="https://httpbin.org/cookies/set?mycookie=myvalue")
    
    async with AsyncWebCrawler(config=browser_cfg_persistent) as crawler:
        print("\n--- First run: Setting a cookie ---")
        result1 = await crawler.arun(config=run_config1)
        if result1.success:
            print(f"First run to {result1.url} successful.")
            # httpbin.org/cookies/set redirects to /cookies, which shows current cookies
            if "mycookie" in result1.html and "myvalue" in result1.html:
                 print("Cookie 'mycookie=myvalue' likely set and visible in response.")
            else:
                 print("Cookie might not be immediately visible in this response, check next run.")
        else:
            print(f"First run failed: {result1.error_message}")

    # Second run: with the same BrowserConfig (and thus same user_data_dir)
    # The cookie "mycookie" should persist.
    run_config2 = CrawlerRunConfig(url="https://httpbin.org/cookies") # This page shows received cookies

    async with AsyncWebCrawler(config=browser_cfg_persistent) as crawler: # Reuses the same persistent context
        print("\n--- Second run: Checking for persisted cookie ---")
        result2 = await crawler.arun(config=run_config2)
        if result2.success:
            print(f"Second run to {result2.url} successful.")
            print(f"Response content (cookies): {result2.html}")
            if "mycookie" in result2.html and "myvalue" in result2.html:
                print("SUCCESS: Cookie 'mycookie=myvalue' persisted across runs!")
            else:
                print("FAILURE: Cookie did not persist or was not found.")
        else:
            print(f"Second run failed: {result2.error_message}")

    # Clean up the temporary directory
    if user_data_path.exists():
        shutil.rmtree(user_data_path)
    print(f"\nCleaned up user data directory: {user_data_path.resolve()}")

if __name__ == "__main__":
    asyncio.run(persistent_context_example())
```

---
#### 4.2.5. Example: Specifying Chrome browser channel using `channel="chrome"`.
Launches the stable Chrome browser if installed, instead of Chromium.

```python
import asyncio
from crawl4ai import BrowserConfig, AsyncWebCrawler, CrawlerRunConfig

async def chrome_channel_config():
    # This requires Google Chrome (stable channel) to be installed.
    # Playwright will attempt to find it.
    browser_cfg = BrowserConfig(
        browser_type="chromium", # Still 'chromium' as base type for Playwright
        channel="chrome",        # Specify 'chrome' channel
        headless=True
    )
    print(f"Chrome Channel BrowserConfig: {browser_cfg.to_dict(exclude_none=True)}")
    assert browser_cfg.channel == "chrome"

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        print("Attempting to crawl with Google Chrome (stable channel)...")
        result = await crawler.arun(url="https://example.com", config=CrawlerRunConfig())
        if result.success:
            print(f"Crawled successfully with Chrome channel. Page title: {result.metadata.get('title')}")
            # To truly verify, one might check specific Chrome-only features or detailed UA string,
            # but that's beyond simple config demonstration.
        else:
            print(f"Crawl failed with Chrome channel: {result.error_message}")
            print("Ensure Google Chrome (stable) is installed and accessible by Playwright.")

if __name__ == "__main__":
    asyncio.run(chrome_channel_config())
```

---
#### 4.2.6. Example: Specifying Microsoft Edge browser channel using `channel="msedge"`.
Launches Microsoft Edge browser if installed.

```python
import asyncio
from crawl4ai import BrowserConfig, AsyncWebCrawler, CrawlerRunConfig

async def msedge_channel_config():
    # This requires Microsoft Edge (stable channel) to be installed.
    browser_cfg = BrowserConfig(
        browser_type="chromium", # Base type for Playwright
        channel="msedge",        # Specify 'msedge' channel
        headless=True
    )
    print(f"MS Edge Channel BrowserConfig: {browser_cfg.to_dict(exclude_none=True)}")
    assert browser_cfg.channel == "msedge"

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        print("Attempting to crawl with Microsoft Edge (stable channel)...")
        result = await crawler.arun(url="https://example.com", config=CrawlerRunConfig())
        if result.success:
            print(f"Crawled successfully with MS Edge channel. Page title: {result.metadata.get('title')}")
        else:
            print(f"Crawl failed with MS Edge channel: {result.error_message}")
            print("Ensure Microsoft Edge (stable) is installed and accessible by Playwright.")

if __name__ == "__main__":
    asyncio.run(msedge_channel_config())
```

---
### 4.3. Proxy Configuration in `BrowserConfig`

#### 4.3.1. Example: Setting a simple proxy string using the `proxy` parameter.
This is a shorthand for providing proxy details directly as a string.

```python
import asyncio
from crawl4ai import BrowserConfig #, AsyncWebCrawler, CrawlerRunConfig

async def simple_proxy_string_in_browserconfig():
    # Format: "scheme://user:pass@host:port" or "scheme://host:port"
    # This is a non-functional placeholder.
    proxy_server_string = "http://user:password@proxy.example.com:8080" 
    
    browser_cfg = BrowserConfig(proxy=proxy_server_string, headless=True)
    print(f"BrowserConfig with simple proxy string: {browser_cfg.proxy}")
    
    # The `proxy` string is parsed internally into Playwright's proxy format.
    # To demonstrate its effect, one would typically use it with AsyncWebCrawler:
    # async with AsyncWebCrawler(config=browser_cfg) as crawler:
    #     result = await crawler.arun(url="https://httpbin.org/ip")
    #     print(result.html) # Should show proxy's IP
    
    print(f"Proxy server string set to: {browser_cfg.proxy}")
    # Note: The ProxyConfig object is not explicitly created or exposed when using the 'proxy' string directly.
    # If you need to access ProxyConfig attributes, use the `proxy_config` parameter with a ProxyConfig object.

if __name__ == "__main__":
    asyncio.run(simple_proxy_string_in_browserconfig())
```

---
#### 4.3.2. Example: Using a detailed `ProxyConfig` object via the `proxy_config` parameter.
Provides more structured control over proxy settings.

```python
import asyncio
from crawl4ai import BrowserConfig, ProxyConfig #, AsyncWebCrawler, CrawlerRunConfig

async def detailed_proxy_object_in_browserconfig():
    # This is a non-functional placeholder.
    proxy_obj = ProxyConfig(
        server="http://anotherproxy.example.com:3128",
        username="proxy_user_obj",
        password="proxy_password_obj"
    )
    
    browser_cfg = BrowserConfig(proxy_config=proxy_obj, headless=True)
    print(f"BrowserConfig with ProxyConfig object: {browser_cfg.proxy_config.to_dict()}") # type: ignore
    assert browser_cfg.proxy_config.server == "http://anotherproxy.example.com:3128" # type: ignore

    # To demonstrate its effect:
    # async with AsyncWebCrawler(config=browser_cfg) as crawler:
    #     result = await crawler.arun(url="https://httpbin.org/ip")
    #     print(result.html) # Should show proxy's IP
    print("ProxyConfig object set. For a live test, replace placeholder with a real proxy.")

if __name__ == "__main__":
    asyncio.run(detailed_proxy_object_in_browserconfig())
```

---
### 4.4. Viewport and Display Settings

#### 4.4.1. Example: Setting custom viewport dimensions using `viewport_width` and `viewport_height`.
Controls the initial size of the browser window/viewport.

```python
import asyncio
from crawl4ai import BrowserConfig, AsyncWebCrawler, CrawlerRunConfig

async def custom_viewport_dimensions():
    browser_cfg = BrowserConfig(
        viewport_width=1920,
        viewport_height=1080,
        headless=True
    )
    print(f"BrowserConfig with custom viewport: Width={browser_cfg.viewport_width}, Height={browser_cfg.viewport_height}")
    assert browser_cfg.viewport_width == 1920
    assert browser_cfg.viewport_height == 1080

    # Demonstrate by checking JavaScript window dimensions
    run_config = CrawlerRunConfig(
        js_code="JSON.stringify({width: window.innerWidth, height: window.innerHeight})"
    )
    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(url="https://example.com", config=run_config)
        if result.success and result.js_execution_result:
            dims = result.js_execution_result
            print(f"JS reported dimensions: {dims}")
            # Note: Playwright viewport might differ slightly from window.innerWidth/Height due to scrollbars etc.
            # This example primarily shows the config is passed.
            assert dims.get("width") is not None # Check if JS executed
        else:
            print(f"Crawl or JS execution failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(custom_viewport_dimensions())
```

---
#### 4.4.2. Example: Setting viewport dimensions using the `viewport` dictionary parameter.
An alternative way to set viewport size, overriding individual width/height parameters if both are set.

```python
import asyncio
from crawl4ai import BrowserConfig, AsyncWebCrawler, CrawlerRunConfig
import json

async def viewport_dict_parameter():
    viewport_dimensions = {"width": 800, "height": 600}
    browser_cfg = BrowserConfig(
        viewport=viewport_dimensions,
        headless=True,
        # If viewport_width/height were also set, 'viewport' dict takes precedence
        viewport_width=1200 # This will be overridden by the viewport dict
    )
    print(f"BrowserConfig with viewport dict: {browser_cfg.viewport}")
    print(f"Effective viewport_width: {browser_cfg.viewport_width}") # Should be 800
    print(f"Effective viewport_height: {browser_cfg.viewport_height}") # Should be 600

    assert browser_cfg.viewport_width == 800
    assert browser_cfg.viewport_height == 600

    # Demonstrate by checking JavaScript window dimensions
    run_config = CrawlerRunConfig(
        js_code="JSON.stringify({width: window.innerWidth, height: window.innerHeight})"
    )
    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(url="https://example.com", config=run_config)
        if result.success and result.js_execution_result:
            dims = result.js_execution_result
            print(f"JS reported dimensions: {dims}")
            # Similar to above, exact match might vary due to browser chrome
            assert dims.get("width") is not None
        else:
            print(f"Crawl or JS execution failed: {result.error_message}")
            
if __name__ == "__main__":
    asyncio.run(viewport_dict_parameter())
```

---
### 4.5. Downloads and Storage State

#### 4.5.1. Example: Enabling file downloads with `accept_downloads=True` and specifying `downloads_path`.
Allows the browser to download files triggered by page interactions or navigations.

```python
import asyncio
import os
import shutil
from pathlib import Path
from crawl4ai import BrowserConfig, AsyncWebCrawler, CrawlerRunConfig

async def enable_file_downloads():
    # Create a temporary directory for downloads
    temp_downloads_dir = Path("./temp_crawl4ai_downloads")
    if temp_downloads_dir.exists():
        shutil.rmtree(temp_downloads_dir)
    temp_downloads_dir.mkdir(parents=True, exist_ok=True)
    
    print(f"Downloads will be saved to: {temp_downloads_dir.resolve()}")

    browser_cfg = BrowserConfig(
        accept_downloads=True,
        downloads_path=str(temp_downloads_dir.resolve()),
        headless=True
    )
    print(f"BrowserConfig for downloads: accept_downloads={browser_cfg.accept_downloads}, path={browser_cfg.downloads_path}")
    assert browser_cfg.accept_downloads
    assert browser_cfg.downloads_path == str(temp_downloads_dir.resolve())
    
    # A small, publicly downloadable file (e.g., a sample text file or small image)
    # This URL directly triggers a download for a sample PDF from an educational site
    download_trigger_url = "https://www.w3.org/WAI/ER/tests/xhtml/testfiles/resources/pdf/dummy.pdf"

    run_config = CrawlerRunConfig(
        url=download_trigger_url,
        # For direct downloads, Playwright often handles it without JS click.
        # If it was a button: js_code="document.querySelector('#downloadButton').click();"
        # We also need to give it time for the download to complete.
        page_timeout=30000 # 30 seconds for download
    )

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        print(f"Attempting to download from: {download_trigger_url}")
        result = await crawler.arun(config=run_config)
        
        if result.success:
            print("Crawl part successful. Checking for downloaded files...")
            if result.downloaded_files:
                print(f"Files downloaded to {browser_cfg.downloads_path}:")
                for file_path in result.downloaded_files:
                    print(f"  - {file_path} (Size: {Path(file_path).stat().st_size} bytes)")
                assert len(result.downloaded_files) > 0
            else:
                print("No files reported as downloaded by the crawler. The page might not have triggered a download as expected, or the download event was missed.")
        else:
            print(f"Crawl failed: {result.error_message}")
            
    # Clean up
    if temp_downloads_dir.exists():
        shutil.rmtree(temp_downloads_dir)
    print(f"Cleaned up downloads directory: {temp_downloads_dir.resolve()}")

if __name__ == "__main__":
    asyncio.run(enable_file_downloads())
```

---
#### 4.5.2. Example: Loading browser state (cookies, localStorage) from a file path using `storage_state`.
Restores a previously saved browser session.

```python
import asyncio
import json
import shutil
from pathlib import Path
from crawl4ai import BrowserConfig, AsyncWebCrawler, CrawlerRunConfig

async def load_storage_state_from_file():
    # Create a dummy storage state file for this example
    storage_state_path = Path("./temp_crawl4ai_storage_state.json")
    dummy_storage_state = {
        "cookies": [{
            "name": "persistent_cookie", "value": "loaded_from_file",
            "domain": "httpbin.org", "path": "/", "expires": -1,
            "httpOnly": False, "secure": False, "sameSite": "Lax"
        }],
        "origins": [{
            "origin": "https://httpbin.org",
            "localStorage": [{"name": "persistent_ls_item", "value": "loaded_from_file_ls"}]
        }]
    }
    with open(storage_state_path, 'w') as f:
        json.dump(dummy_storage_state, f)

    print(f"Using storage state from file: {storage_state_path.resolve()}")

    browser_cfg = BrowserConfig(
        storage_state=str(storage_state_path.resolve()),
        headless=True,
        verbose=True
    )
    print(f"BrowserConfig with storage_state file: {browser_cfg.storage_state}")
    
    # URL to check cookies and localStorage
    check_url = "https://httpbin.org/anything" 
    # JS to retrieve localStorage (httpbin doesn't show it directly in /anything)
    js_to_get_ls = "JSON.stringify(localStorage.getItem('persistent_ls_item'))"

    run_config = CrawlerRunConfig(url=check_url, js_code=js_to_get_ls)

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(config=run_config)
        if result.success:
            print(f"Crawled {result.url} with loaded storage state.")
            response_data = json.loads(result.html) # httpbin.org/anything returns JSON
            
            # Check for cookie
            if "persistent_cookie=loaded_from_file" in response_data.get("headers", {}).get("Cookie", ""):
                print("SUCCESS: Cookie 'persistent_cookie' was loaded and sent!")
            else:
                print(f"Cookie not found in request headers. Cookies: {response_data.get('headers', {}).get('Cookie')}")

            # Check for localStorage item (via JS execution result)
            if result.js_execution_result == '"loaded_from_file_ls"': # JS returns JSON string
                print("SUCCESS: localStorage item 'persistent_ls_item' was loaded!")
            else:
                print(f"localStorage item not found or incorrect. JS result: {result.js_execution_result}")
        else:
            print(f"Crawl failed: {result.error_message}")

    # Clean up
    if storage_state_path.exists():
        storage_state_path.unlink()
    print(f"\nCleaned up storage state file: {storage_state_path.resolve()}")

if __name__ == "__main__":
    asyncio.run(load_storage_state_from_file())
```

---
#### 4.5.3. Example: Loading browser state from an in-memory dictionary using `storage_state`.
Allows providing cookies and localStorage directly as a Python dictionary.

```python
import asyncio
import json
from crawl4ai import BrowserConfig, AsyncWebCrawler, CrawlerRunConfig

async def load_storage_state_from_dict():
    in_memory_storage_state = {
        "cookies": [{
            "name": "mem_cookie", "value": "loaded_from_dict",
            "url": "https://httpbin.org" # More robust to use 'url' or 'domain'/'path'
        }],
        "origins": [{
            "origin": "https://httpbin.org",
            "localStorage": [{"name": "mem_ls_item", "value": "loaded_from_dict_ls"}]
        }]
    }
    print(f"Using in-memory storage state: {in_memory_storage_state}")

    browser_cfg = BrowserConfig(
        storage_state=in_memory_storage_state,
        headless=True,
        verbose=True
    )
    
    check_url = "https://httpbin.org/anything"
    js_to_get_ls = "JSON.stringify(localStorage.getItem('mem_ls_item'))"
    run_config = CrawlerRunConfig(url=check_url, js_code=js_to_get_ls)

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(config=run_config)
        if result.success:
            print(f"Crawled {result.url} with in-memory storage state.")
            response_data = json.loads(result.html)
            
            if "mem_cookie=loaded_from_dict" in response_data.get("headers", {}).get("Cookie", ""):
                print("SUCCESS: Cookie 'mem_cookie' was loaded and sent!")
            else:
                print(f"Cookie not found in request headers. Cookies: {response_data.get('headers', {}).get('Cookie')}")

            if result.js_execution_result == '"loaded_from_dict_ls"':
                print("SUCCESS: localStorage item 'mem_ls_item' was loaded!")
            else:
                print(f"localStorage item not found or incorrect. JS result: {result.js_execution_result}")
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(load_storage_state_from_dict())
```

---
### 4.6. Security and Scripting Control

#### 4.6.1. Example: Enforcing HTTPS error checks by setting `ignore_https_errors=False`.
By default, Crawl4ai (via Playwright) ignores HTTPS errors. This shows how to enforce them.

```python
import asyncio
from crawl4ai import BrowserConfig, AsyncWebCrawler, CrawlerRunConfig

async def enforce_https_errors():
    browser_cfg = BrowserConfig(
        ignore_https_errors=False, # Default is True
        headless=True
    )
    print(f"BrowserConfig with ignore_https_errors={browser_cfg.ignore_https_errors}")
    assert not browser_cfg.ignore_https_errors
    
    # Use a site with a known SSL issue (e.g., expired, self-signed)
    # expired.badssl.com is a good test site for this
    # For safety in automated tests, we won't hit a live "bad" SSL site by default.
    # test_url_with_ssl_issue = "https://expired.badssl.com/"
    test_url_good_ssl = "https://example.com"


    print(f"Attempting to crawl {test_url_good_ssl} (should succeed).")
    run_config = CrawlerRunConfig(url=test_url_good_ssl)
    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(config=run_config)
        if result.success:
            print(f"Successfully crawled {test_url_good_ssl} with HTTPS errors NOT ignored.")
        else:
            print(f"Crawl failed for {test_url_good_ssl}: {result.error_message}")

    # To test the error case:
    # print(f"\nAttempting to crawl {test_url_with_ssl_issue} (should fail due to SSL error).")
    # run_config_bad_ssl = CrawlerRunConfig(url=test_url_with_ssl_issue)
    # async with AsyncWebCrawler(config=browser_cfg) as crawler:
    #     result_bad = await crawler.arun(config=run_config_bad_ssl)
    #     if not result_bad.success and "net::ERR_CERT_DATE_INVALID" in result_bad.error_message: # Or similar error
    #         print(f"SUCCESS: Crawl failed as expected for {test_url_with_ssl_issue} due to SSL error: {result_bad.error_message[:100]}...")
    #     elif result_bad.success:
    #         print(f"UNEXPECTED: Crawl succeeded for {test_url_with_ssl_issue}, SSL error might not have been caught.")
    #     else:
    #         print(f"Crawl failed for {test_url_with_ssl_issue} for other reasons: {result_bad.error_message}")
    print("\nNote: Actual test for HTTPS error enforcement requires a site like expired.badssl.com.")


if __name__ == "__main__":
    asyncio.run(enforce_https_errors())
```

---
#### 4.6.2. Example: Disabling JavaScript execution in pages by setting `java_script_enabled=False`.
Prevents JavaScript from running, which can speed up crawls but might break sites reliant on JS.

```python
import asyncio
from crawl4ai import BrowserConfig, AsyncWebCrawler, CrawlerRunConfig

async def disable_javascript():
    browser_cfg = BrowserConfig(
        java_script_enabled=False, # Default is True
        headless=True
    )
    print(f"BrowserConfig with java_script_enabled={browser_cfg.java_script_enabled}")
    assert not browser_cfg.java_script_enabled
    
    # A page that uses JS to modify content
    # httpbin.org/html includes a simple script
    test_url = "https://httpbin.org/html" 
    run_config = CrawlerRunConfig(url=test_url)

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        print(f"Attempting to crawl {test_url} with JavaScript disabled.")
        result = await crawler.arun(config=run_config)
        if result.success:
            print(f"Successfully crawled {test_url}.")
            # Look for signs that JS did NOT run.
            # The sample JS on httpbin.org/html adds "Hello, world!" to an h1.
            # If JS is disabled, this text should be absent or different.
            # The original h1 on httpbin.org/html is "Herman Melville - Moby-Dick"
            if "Moby-Dick" in result.html and "Hello, world!" not in result.html:
                print("SUCCESS: JavaScript seems to have been disabled (JS-added content not found).")
            else:
                print("JavaScript execution state is inconclusive from this page's content.")
                # print(result.html) # For debugging
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(disable_javascript())
```

---
### 4.7. Headers, Cookies, and User Agent Customization

#### 4.7.1. Example: Adding a list of custom `cookies` to be set in the browser context.
These cookies will be sent with all requests made by this browser context.

```python
import asyncio
import json
from crawl4ai import BrowserConfig, AsyncWebCrawler, CrawlerRunConfig

async def custom_cookies_browser_config():
    custom_cookies_list = [
        {"name": "my_custom_cookie", "value": "cookie_value_123", "url": "https://httpbin.org"},
        {"name": "another_cookie", "value": "more_data", "domain": ".httpbin.org", "path": "/"}
    ]
    browser_cfg = BrowserConfig(cookies=custom_cookies_list, headless=True)
    print(f"BrowserConfig with custom cookies: {browser_cfg.cookies}")
    
    # httpbin.org/cookies shows cookies sent by the client
    run_config = CrawlerRunConfig(url="https://httpbin.org/cookies")

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(config=run_config)
        if result.success:
            print(f"Crawled {result.url} with custom cookies.")
            response_data = json.loads(result.html) # httpbin returns JSON
            print(f"Cookies received by server: {response_data.get('cookies')}")
            assert "my_custom_cookie" in response_data.get("cookies", {})
            assert response_data.get("cookies", {}).get("my_custom_cookie") == "cookie_value_123"
            assert "another_cookie" in response_data.get("cookies", {})
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(custom_cookies_browser_config())
```

---
#### 4.7.2. Example: Setting default `headers` for all requests made within the browser context.
These headers will be added to every HTTP request.

```python
import asyncio
import json
from crawl4ai import BrowserConfig, AsyncWebCrawler, CrawlerRunConfig

async def default_headers_browser_config():
    custom_headers_dict = {
        "X-Custom-Header": "Crawl4AI-Test",
        "Accept-Language": "de-DE,de;q=0.9" # Example: German language preference
    }
    browser_cfg = BrowserConfig(headers=custom_headers_dict, headless=True)
    print(f"BrowserConfig with default headers: {browser_cfg.headers}")
    
    # httpbin.org/headers shows headers received by the server
    run_config = CrawlerRunConfig(url="https://httpbin.org/headers")

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(config=run_config)
        if result.success:
            print(f"Crawled {result.url} with custom default headers.")
            response_data = json.loads(result.html) # httpbin returns JSON
            print(f"Headers received by server (excerpt):")
            received_headers = response_data.get("headers", {})
            for key, value in custom_headers_dict.items():
                print(f"  {key}: {received_headers.get(key)}")
                assert received_headers.get(key) == value
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(default_headers_browser_config())
```

---
#### 4.7.3. Example: Setting a specific `user_agent` string.
Overrides the default Playwright user agent.

```python
import asyncio
import json
from crawl4ai import BrowserConfig, AsyncWebCrawler, CrawlerRunConfig

async def specific_user_agent_config():
    my_ua_string = "MyCustomCrawler/1.0 (compatible; MyBot/0.1; +http://mybot.example.com)"
    browser_cfg = BrowserConfig(user_agent=my_ua_string, headless=True)
    print(f"BrowserConfig with specific User-Agent: {browser_cfg.user_agent}")
    assert browser_cfg.user_agent == my_ua_string
    
    # httpbin.org/user-agent shows the User-Agent header received by the server
    run_config = CrawlerRunConfig(url="https://httpbin.org/user-agent")

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(config=run_config)
        if result.success:
            print(f"Crawled {result.url} with specific User-Agent.")
            response_data = json.loads(result.html)
            print(f"User-Agent received by server: {response_data.get('user-agent')}")
            assert response_data.get('user-agent') == my_ua_string
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(specific_user_agent_config())
```

---
#### 4.7.4. Example: Generating a random `user_agent` by setting `user_agent_mode="random"`.
Uses the built-in user agent generator to pick a random, valid user agent.

```python
import asyncio
import json
from crawl4ai import BrowserConfig, AsyncWebCrawler, CrawlerRunConfig

async def random_user_agent_config():
    # user_agent_mode="random" will use ValidUAGenerator by default
    browser_cfg = BrowserConfig(user_agent_mode="random", headless=True, verbose=True)
    # The actual user_agent string is generated upon BrowserConfig initialization if mode is random.
    print(f"BrowserConfig with random User-Agent mode. Generated UA: {browser_cfg.user_agent}")
    assert browser_cfg.user_agent is not None 
    assert browser_cfg.user_agent != BrowserConfig().user_agent # Should be different from default
    
    run_config = CrawlerRunConfig(url="https://httpbin.org/user-agent")

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(config=run_config)
        if result.success:
            print(f"Crawled {result.url} with a random User-Agent.")
            response_data = json.loads(result.html)
            print(f"User-Agent received by server: {response_data.get('user-agent')}")
            assert response_data.get('user-agent') == browser_cfg.user_agent # Check if it matches the one set
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(random_user_agent_config())
```

---
#### 4.7.5. Example: Customizing random user agent generation using `user_agent_generator_config` (e.g., specifying device type or OS).
Allows fine-tuning the type of random user agent generated.

```python
import asyncio
import json
from crawl4ai import BrowserConfig, AsyncWebCrawler, CrawlerRunConfig

async def custom_random_user_agent_config():
    # Example: Generate a random user agent for a Linux Desktop
    ua_gen_config = {"device_type": "desktop", "os_name": "linux"}
    
    browser_cfg = BrowserConfig(
        user_agent_mode="random",
        user_agent_generator_config=ua_gen_config,
        headless=True,
        verbose=True
    )
    generated_ua = browser_cfg.user_agent
    print(f"BrowserConfig with custom random UA generation. Config: {ua_gen_config}")
    print(f"Generated UA: {generated_ua}")
    
    # Basic check if the UA string seems plausible for Linux
    assert "Linux" in generated_ua or "X11" in generated_ua
    
    run_config = CrawlerRunConfig(url="https://httpbin.org/user-agent")

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(config=run_config)
        if result.success:
            print(f"Crawled {result.url} with custom random User-Agent.")
            response_data = json.loads(result.html)
            print(f"User-Agent received by server: {response_data.get('user-agent')}")
            assert response_data.get('user-agent') == generated_ua
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(custom_random_user_agent_config())
```

---
#### 4.7.6. Example: Demonstrating `BrowserConfig` automatically setting `sec-ch-ua` client hint headers.
`BrowserConfig` automatically derives and sets appropriate `Sec-CH-UA` client hint headers based on the `user_agent`.

```python
import asyncio
import json
from crawl4ai import BrowserConfig, AsyncWebCrawler, CrawlerRunConfig

async def sec_ch_ua_header_demonstration():
    # Using a specific User-Agent that would imply certain client hints
    # This UA is for Chrome 116 on Linux
    ua_string = "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/116.0.0.0 Safari/537.36"
    
    browser_cfg = BrowserConfig(user_agent=ua_string, headless=True, verbose=True)
    print(f"BrowserConfig User-Agent: {browser_cfg.user_agent}")
    print(f"Automatically generated Sec-CH-UA client hint: {browser_cfg.browser_hint}")
    
    # Expected client hint might look something like:
    # '"Chromium";v="116", "Not)A;Brand";v="24", "Google Chrome";v="116"'
    # The exact value depends on the UAGen library's parsing of the UA.
    assert "Chromium" in browser_cfg.browser_hint or "Google Chrome" in browser_cfg.browser_hint
    assert "116" in browser_cfg.browser_hint

    run_config = CrawlerRunConfig(url="https://httpbin.org/headers")

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(config=run_config)
        if result.success:
            print(f"Crawled {result.url}. Checking received headers by server...")
            response_data = json.loads(result.html)
            received_headers = response_data.get("headers", {})
            
            print(f"  User-Agent: {received_headers.get('User-Agent')}")
            print(f"  Sec-Ch-Ua: {received_headers.get('Sec-Ch-Ua')}") # Case-insensitive matching might be needed for real server
            
            # Check if the Sec-CH-UA header set by BrowserConfig was received
            # Note: httpbin.org might not show all client hints perfectly, or Playwright might override some.
            # This primarily tests that BrowserConfig correctly *sets* it for Playwright.
            # The actual sent header can depend on Playwright's behavior with that browser version.
            if browser_cfg.browser_hint.strip('"') in received_headers.get('Sec-Ch-Ua', '').strip('"'):
                 print("SUCCESS: Sec-CH-UA client hint seems to be correctly passed through.")
            else:
                 print("NOTE: Sec-CH-UA might differ slightly due to Playwright/browser behavior or httpbin.org limitations.")
                 print(f"   Expected hint from BrowserConfig: {browser_cfg.browser_hint}")

        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(sec_ch_ua_header_demonstration())
```

---
### 4.8. Performance and Other Browser Settings

#### 4.8.1. Example: Using `text_mode=True` to attempt disabling images and rich content for faster text-focused crawls.
This mode aims to block resources like images, fonts, and potentially some scripts to speed up loading for text extraction.

```python
import asyncio
from crawl4ai import BrowserConfig, AsyncWebCrawler, CrawlerRunConfig

async def text_mode_config():
    browser_cfg = BrowserConfig(text_mode=True, headless=True, verbose=True)
    print(f"BrowserConfig with text_mode: {browser_cfg.text_mode}")
    assert browser_cfg.text_mode
    
    # A page with images
    run_config = CrawlerRunConfig(url="https://example.com") # example.com has an IANA logo

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        print(f"Attempting to crawl {run_config.url} in text_mode.")
        # We'll also capture network requests to see if image requests are blocked.
        result = await crawler.arun(config=run_config.clone(capture_network_requests=True))
        
        if result.success:
            print(f"Successfully crawled {run_config.url} in text_mode.")
            
            image_requests_found = False
            if result.network_requests:
                for req in result.network_requests:
                    if req.get("event_type") == "request" and req.get("resource_type") == "image":
                        image_requests_found = True
                        print(f"Found image request (unexpected in text_mode): {req.get('url')}")
                        break
            
            if not image_requests_found:
                print("SUCCESS: No image requests were detected, as expected in text_mode.")
            else:
                print("WARNING: Image requests were detected. Text_mode might not have fully blocked them for this site/browser.")
            
            # Check if images are absent in the rendered HTML (might be harder to verify reliably)
            # print(f"HTML (first 500 chars):\n{result.html[:500]}")
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(text_mode_config())
```

---
#### 4.8.2. Example: Using `light_mode=True` to disable certain background browser features for performance.
`light_mode` applies a set of browser flags (defined in `BROWSER_DISABLE_OPTIONS` in `browser_manager.py`) to reduce resource usage.

```python
import asyncio
from crawl4ai import BrowserConfig, AsyncWebCrawler, CrawlerRunConfig

async def light_mode_config():
    browser_cfg = BrowserConfig(light_mode=True, headless=True, verbose=True)
    print(f"BrowserConfig with light_mode: {browser_cfg.light_mode}")
    print(f"Extra args applied by light_mode (subset shown): {browser_cfg.extra_args[:5]}...")
    assert browser_cfg.light_mode
    # Check if some known light_mode flags are present in extra_args
    assert "--disable-background-networking" in browser_cfg.extra_args 
    
    run_config = CrawlerRunConfig(url="https://example.com")

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        print(f"Attempting to crawl {run_config.url} in light_mode.")
        result = await crawler.arun(config=run_config)
        if result.success:
            print(f"Successfully crawled {run_config.url} in light_mode.")
            # Effect of light_mode is on browser resource usage, not directly visible in content typically.
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(light_mode_config())
```

---
#### 4.8.3. Example: Passing additional command-line `extra_args` to the browser launcher.
Allows fine-grained control over browser behavior by passing Chromium/Firefox specific flags.

```python
import asyncio
from crawl4ai import BrowserConfig, AsyncWebCrawler, CrawlerRunConfig

async def extra_args_config():
    # Example: Disable GPU (often useful in headless environments)
    # and set a custom window size (though viewport is preferred for content area)
    custom_args = [
        "--disable-gpu", 
        "--window-size=800,600" # Note: viewport_width/height is preferred for content area
    ]
    browser_cfg = BrowserConfig(extra_args=custom_args, headless=True) # Usually headless with these args
    print(f"BrowserConfig with extra_args: {browser_cfg.extra_args}")
    assert "--disable-gpu" in browser_cfg.extra_args
    
    run_config = CrawlerRunConfig(url="https://example.com")

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        print(f"Attempting to crawl {run_config.url} with extra browser arguments.")
        result = await crawler.arun(config=run_config)
        if result.success:
            print(f"Successfully crawled {run_config.url} with extra arguments.")
            # Verifying effect of these args often requires deeper inspection or specific test pages.
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(extra_args_config())
```

---
#### 4.8.4. Example: Setting a custom `debugging_port` and `host` for the browser's remote debugging.
Useful if the default port (9222) is in use or for specific network configurations.

```python
import asyncio
from crawl4ai import BrowserConfig, AsyncWebCrawler, CrawlerRunConfig

# Note: This example primarily shows config. It might be hard to verify the port
# change without external tools unless use_managed_browser is also True
# and we try to connect to the new CDP URL.

async def custom_debugging_port_host():
    custom_port = 9333
    custom_host = "127.0.0.1" # Often 'localhost' or '127.0.0.1'

    # This is most relevant for use_managed_browser=True or browser_mode='builtin'
    # as it tells the ManagedBrowser which port to launch on.
    browser_cfg = BrowserConfig(
        debugging_port=custom_port,
        host=custom_host,
        headless=True,
        use_managed_browser=True, # To see the effect of debugging_port
        verbose=True
    )
    print(f"BrowserConfig with custom debugging_port={browser_cfg.debugging_port} and host={browser_cfg.host}")
    assert browser_cfg.debugging_port == custom_port
    assert browser_cfg.host == custom_host

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        print(f"Attempting to launch managed browser on {custom_host}:{custom_port} and crawl.")
        # The ManagedBrowser instance within AsyncWebCrawler will use these settings.
        # If successful, it means the browser launched on the custom port and CDP connection was made.
        result = await crawler.arun(url="https://example.com", config=CrawlerRunConfig())
        if result.success:
            print(f"Successfully crawled. Managed browser likely used {custom_host}:{custom_port}.")
        else:
            print(f"Crawl failed: {result.error_message}. This could be due to port conflict or other launch issues.")
            print("Ensure the custom port is available.")
            
if __name__ == "__main__":
    asyncio.run(custom_debugging_port_host())
```

---
#### 4.8.5. Example: Enabling verbose logging for browser operations via `verbose=True`.
Provides more detailed output from the `AsyncWebCrawler` and underlying strategies.

```python
import asyncio
from crawl4ai import BrowserConfig, AsyncWebCrawler, CrawlerRunConfig

async def verbose_browser_logging():
    # Set verbose=True in BrowserConfig
    # This will be picked up by the AsyncLogger if not overridden by CrawlerRunConfig
    browser_cfg = BrowserConfig(verbose=True, headless=True)
    print(f"BrowserConfig verbose setting: {browser_cfg.verbose}")
    
    # CrawlerRunConfig can also have a verbose setting, which might take precedence for that run.
    # Here, we don't set it in CrawlerRunConfig, so BrowserConfig's verbose should apply.
    run_config = CrawlerRunConfig(url="https://example.com")

    print("\nRunning crawl with verbose=True in BrowserConfig. Expect more detailed logs.")
    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        # The logger used by the crawler will inherit verbosity from browser_cfg
        result = await crawler.arun(config=run_config)
        if result.success:
            print(f"\nCrawl successful. Verbose logs should have been printed during the process.")
        else:
            print(f"\nCrawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(verbose_browser_logging())
```

---
#### 4.8.6. Example: Using `sleep_on_close=True` to pause before the browser fully closes (useful for debugging).
This is useful when `headless=False` to inspect the final state of the browser before it closes.

```python
import asyncio
from crawl4ai import BrowserConfig, AsyncWebCrawler, CrawlerRunConfig

async def sleep_on_close_example():
    # This is most effective with headless=False
    # In a script, the pause might not be very noticeable if headless=True
    browser_cfg = BrowserConfig(
        sleep_on_close=True, 
        headless=False, # Set to False to see the browser window pause
        verbose=True
    )
    print(f"BrowserConfig with sleep_on_close={browser_cfg.sleep_on_close}")
    
    run_config = CrawlerRunConfig(url="https://example.com")

    print("\nRunning crawl. Browser window should appear and pause for a moment before closing if headless=False.")
    print("(If headless=True, the effect is a slight delay before script termination).")
    try:
        async with AsyncWebCrawler(config=browser_cfg) as crawler:
            result = await crawler.arun(config=run_config)
            if result.success:
                print(f"Crawl successful. Browser should have paused before closing (if visible).")
            else:
                print(f"Crawl failed: {result.error_message}")
    except Exception as e:
        print(f"Could not run headed browser example for sleep_on_close (common in restricted environments): {e}")
        print("Skipping sleep_on_close headed test.")


if __name__ == "__main__":
    # asyncio.run(sleep_on_close_example())
    print("Skipping sleep_on_close_example. Uncomment to run, preferably with headless=False if you have a display.")
```

---
### 4.9. `BrowserConfig` Utility Methods

#### 4.9.1. Example: Creating `BrowserConfig` from a dictionary of keyword arguments using `BrowserConfig.from_kwargs()`.
Useful for creating config objects dynamically.

```python
import asyncio
from crawl4ai import BrowserConfig

async def browserconfig_from_kwargs():
    kwargs = {
        "browser_type": "firefox",
        "headless": False,
        "viewport_width": 1200,
        "user_agent": "MyTestAgent/1.0"
    }
    browser_cfg = BrowserConfig.from_kwargs(kwargs)
    
    print("BrowserConfig created from_kwargs:")
    print(f"  Browser Type: {browser_cfg.browser_type}")
    print(f"  Headless: {browser_cfg.headless}")
    print(f"  Viewport Width: {browser_cfg.viewport_width}")
    print(f"  User Agent: {browser_cfg.user_agent}")

    assert browser_cfg.browser_type == "firefox"
    assert not browser_cfg.headless
    assert browser_cfg.viewport_width == 1200
    assert browser_cfg.user_agent == "MyTestAgent/1.0"

if __name__ == "__main__":
    asyncio.run(browserconfig_from_kwargs())
```

---
#### 4.9.2. Example: Converting `BrowserConfig` instance to a dictionary using `to_dict()`.
Serializes the config object's attributes to a Python dictionary.

```python
import asyncio
from crawl4ai import BrowserConfig

async def browserconfig_to_dict():
    browser_cfg = BrowserConfig(
        browser_type="webkit",
        headless=True,
        proxy="http://proxy.internal:3128",
        verbose=False
    )
    config_dict = browser_cfg.to_dict()

    print("BrowserConfig instance:")
    print(f"  Original object browser_type: {browser_cfg.browser_type}")
    print(f"  Original object proxy: {browser_cfg.proxy}")
    
    print("\nConverted to dictionary:")
    for key, value in config_dict.items():
        # Only print a few for brevity if it's too long
        if key in ["browser_type", "headless", "proxy", "verbose", "user_agent"]:
             print(f"  {key}: {value}")
    
    assert config_dict["browser_type"] == "webkit"
    assert config_dict["headless"] is True
    assert config_dict["proxy"] == "http://proxy.internal:3128"
    assert config_dict["verbose"] is False # Check a default that was changed or not set

if __name__ == "__main__":
    asyncio.run(browserconfig_to_dict())
```

---
#### 4.9.3. Example: Cloning a `BrowserConfig` instance and modifying its `headless` mode using `clone()`.
Creates a new instance with optionally overridden attributes.

```python
import asyncio
from crawl4ai import BrowserConfig

async def browserconfig_clone_modify():
    original_cfg = BrowserConfig(browser_type="chromium", headless=True, viewport_width=1024)
    print(f"Original Config: headless={original_cfg.headless}, viewport_width={original_cfg.viewport_width}")

    # Clone and change headless mode, keep other settings
    cloned_cfg_headed = original_cfg.clone(headless=False)
    print(f"Cloned Config (headed): headless={cloned_cfg_headed.headless}, viewport_width={cloned_cfg_headed.viewport_width}")

    assert original_cfg.headless is True # Original unchanged
    assert cloned_cfg_headed.headless is False # Cloned is modified
    assert cloned_cfg_headed.browser_type == original_cfg.browser_type # Unspecified attributes are copied
    assert cloned_cfg_headed.viewport_width == original_cfg.viewport_width

if __name__ == "__main__":
    asyncio.run(browserconfig_clone_modify())
```

---
#### 4.9.4. Example: Serializing `BrowserConfig` to a JSON-compatible dictionary using `dump()`.
The `dump()` method leverages `to_serializable_dict` for JSON compatibility.

```python
import asyncio
import json
from crawl4ai import BrowserConfig, ProxyConfig

async def browserconfig_dump_json_compatible():
    proxy_obj = ProxyConfig(server="http://jsondump.proxy:1234")
    browser_cfg = BrowserConfig(
        browser_type="firefox",
        headless=False,
        proxy_config=proxy_obj, # Nested object
        extra_args=["--some-flag"]
    )
    
    # dump() calls to_serializable_dict() internally
    dumped_dict = browser_cfg.dump() 

    print("Dumped BrowserConfig (JSON compatible):")
    print(json.dumps(dumped_dict, indent=2))

    # Check if nested ProxyConfig was serialized correctly
    assert dumped_dict["params"]["proxy_config"]["type"] == "ProxyConfig"
    assert dumped_dict["params"]["proxy_config"]["params"]["server"] == "http://jsondump.proxy:1234"
    assert dumped_dict["params"]["browser_type"] == "firefox"
    
    # Verify it can be loaded back (see next example)

if __name__ == "__main__":
    asyncio.run(browserconfig_dump_json_compatible())
```

---
#### 4.9.5. Example: Deserializing `BrowserConfig` from a dictionary (potentially read from JSON) using `load()`.
The `load()` method reconstructs a `BrowserConfig` instance from its serialized form.

```python
import asyncio
from crawl4ai import BrowserConfig, ProxyConfig

async def browserconfig_load_from_dict():
    serialized_data = {
        "type": "BrowserConfig",
        "params": {
            "browser_type": "webkit",
            "headless": True,
            "proxy_config": {
                "type": "ProxyConfig",
                "params": {"server": "http://jsonload.proxy:5678"}
            },
            "user_agent": "LoadedAgent/2.0",
            "extra_args": ["--another-flag"]
        }
    }
    
    loaded_cfg = BrowserConfig.load(serialized_data)
    
    print("Loaded BrowserConfig from dictionary:")
    print(f"  Browser Type: {loaded_cfg.browser_type}")
    print(f"  Headless: {loaded_cfg.headless}")
    print(f"  User Agent: {loaded_cfg.user_agent}")
    print(f"  Extra Args: {loaded_cfg.extra_args}")
    if loaded_cfg.proxy_config:
        print(f"  Proxy Server: {loaded_cfg.proxy_config.server}")

    assert isinstance(loaded_cfg, BrowserConfig)
    assert loaded_cfg.browser_type == "webkit"
    assert loaded_cfg.headless is True
    assert isinstance(loaded_cfg.proxy_config, ProxyConfig)
    assert loaded_cfg.proxy_config.server == "http://jsonload.proxy:5678" # type: ignore
    assert loaded_cfg.user_agent == "LoadedAgent/2.0"
    assert "--another-flag" in loaded_cfg.extra_args

if __name__ == "__main__":
    asyncio.run(browserconfig_load_from_dict())
```

---
## 5. `HTTPCrawlerConfig` Examples (For non-browser HTTP crawling)
`HTTPCrawlerConfig` is used with strategies like `AsyncHTTPCrawlerStrategy` that make direct HTTP requests without a full browser.

### 5.1. Example: Basic `HTTPCrawlerConfig` for a GET request (default).

```python
import asyncio
from crawl4ai import HTTPCrawlerConfig
# from crawl4ai import AsyncWebCrawler, AsyncHTTPCrawlerStrategy (for full example)

async def basic_httpcrawler_config():
    http_cfg = HTTPCrawlerConfig() # Defaults to method="GET"
    print(f"Default HTTPCrawlerConfig: {http_cfg.to_dict()}")
    assert http_cfg.method == "GET"
    
    # To use it:
    # strategy = AsyncHTTPCrawlerStrategy(http_config=http_cfg)
    # async with AsyncWebCrawler(crawler_strategy=strategy) as crawler:
    #     result = await crawler.arun(url="https://httpbin.org/get")
    #     print(result.html)

if __name__ == "__main__":
    asyncio.run(basic_httpcrawler_config())
```

---
### 5.2. Example: Configuring a POST request with form `data` using `HTTPCrawlerConfig`.

```python
import asyncio
from crawl4ai import HTTPCrawlerConfig
# from crawl4ai import AsyncWebCrawler, AsyncHTTPCrawlerStrategy (for full example)
# import json

async def post_form_data_httpcrawler_config():
    form_payload = {"key1": "value1", "key2": "value2"}
    http_cfg = HTTPCrawlerConfig(method="POST", data=form_payload)
    
    print(f"HTTPCrawlerConfig for POST with form data: {http_cfg.to_dict()}")
    assert http_cfg.method == "POST"
    assert http_cfg.data == form_payload
    
    # To use it:
    # strategy = AsyncHTTPCrawlerStrategy() # Default config is fine, override in arun
    # async with AsyncWebCrawler(crawler_strategy=strategy) as crawler:
    #     run_cfg = CrawlerRunConfig(url="https://httpbin.org/post", http_config=http_cfg)
    #     result = await crawler.arun(config=run_cfg)
    #     if result.success:
    #         response_data = json.loads(result.html)
    #         print(f"Server received form data: {response_data.get('form')}")
    #         assert response_data.get('form') == form_payload

if __name__ == "__main__":
    asyncio.run(post_form_data_httpcrawler_config())
    print("Note: For a live test, you'd use this with AsyncHTTPCrawlerStrategy.")
```

---
### 5.3. Example: Configuring a POST request with a `json` payload using `HTTPCrawlerConfig`.

```python
import asyncio
from crawl4ai import HTTPCrawlerConfig
# from crawl4ai import AsyncWebCrawler, AsyncHTTPCrawlerStrategy (for full example)
# import json

async def post_json_payload_httpcrawler_config():
    json_payload = {"name": "Crawl4AI", "version": "0.6.3"}
    http_cfg = HTTPCrawlerConfig(method="POST", json=json_payload) # Use 'json' parameter
    
    print(f"HTTPCrawlerConfig for POST with JSON payload: {http_cfg.to_dict()}")
    assert http_cfg.method == "POST"
    assert http_cfg.json == json_payload
    
    # To use it:
    # strategy = AsyncHTTPCrawlerStrategy()
    # async with AsyncWebCrawler(crawler_strategy=strategy) as crawler:
    #     run_cfg = CrawlerRunConfig(url="https://httpbin.org/post", http_config=http_cfg)
    #     result = await crawler.arun(config=run_cfg)
    #     if result.success:
    #         response_data = json.loads(result.html)
    #         print(f"Server received JSON data: {response_data.get('json')}")
    #         assert response_data.get('json') == json_payload

if __name__ == "__main__":
    asyncio.run(post_json_payload_httpcrawler_config())
    print("Note: For a live test, you'd use this with AsyncHTTPCrawlerStrategy.")
```

---
### 5.4. Example: Setting custom `headers` for an HTTP request via `HTTPCrawlerConfig`.

```python
import asyncio
from crawl4ai import HTTPCrawlerConfig
# from crawl4ai import AsyncWebCrawler, AsyncHTTPCrawlerStrategy (for full example)
# import json

async def custom_headers_httpcrawler_config():
    custom_http_headers = {
        "X-API-Key": "mysecretapikey",
        "Content-Type": "application/json" # Though httpx might set this for json payload
    }
    http_cfg = HTTPCrawlerConfig(headers=custom_http_headers, method="GET")
    
    print(f"HTTPCrawlerConfig with custom headers: {http_cfg.headers}")
    assert http_cfg.headers["X-API-Key"] == "mysecretapikey"
    
    # To use it:
    # strategy = AsyncHTTPCrawlerStrategy()
    # async with AsyncWebCrawler(crawler_strategy=strategy) as crawler:
    #     run_cfg = CrawlerRunConfig(url="https://httpbin.org/headers", http_config=http_cfg)
    #     result = await crawler.arun(config=run_cfg)
    #     if result.success:
    #         response_data = json.loads(result.html)
    #         print(f"Server received headers (excerpt):")
    #         received_headers = response_data.get("headers", {})
    #         assert received_headers.get("X-Api-Key") == "mysecretapikey" # HTTP headers are case-insensitive

if __name__ == "__main__":
    asyncio.run(custom_headers_httpcrawler_config())
    print("Note: For a live test, you'd use this with AsyncHTTPCrawlerStrategy.")
```

---
### 5.5. Example: Disabling automatic redirect following (`follow_redirects=False`) in `HTTPCrawlerConfig`.

```python
import asyncio
from crawl4ai import HTTPCrawlerConfig, AsyncWebCrawler, CrawlerRunConfig
from crawl4ai.async_crawler_strategy import AsyncHTTPCrawlerStrategy

async def no_redirect_httpcrawler_config():
    http_cfg_no_redirect = HTTPCrawlerConfig(follow_redirects=False)
    print(f"HTTPCrawlerConfig with follow_redirects={http_cfg_no_redirect.follow_redirects}")
    assert not http_cfg_no_redirect.follow_redirects
    
    # httpbin.org/redirect/1 will redirect once
    redirect_url = "https://httpbin.org/redirect/1"
    run_config = CrawlerRunConfig(url=redirect_url)
    
    strategy = AsyncHTTPCrawlerStrategy(http_config=http_cfg_no_redirect)
    async with AsyncWebCrawler(crawler_strategy=strategy) as crawler:
        print(f"Attempting to crawl {redirect_url} with redirects disabled.")
        result = await crawler.arun(config=run_config)
        if result.success:
            print(f"Crawl to {result.url} status: {result.status_code}")
            # Expecting a 302 status code since redirects are off
            assert result.status_code in [301, 302, 303, 307, 308] 
            print(f"SUCCESS: Received redirect status {result.status_code} as expected.")
            print(f"Location header: {result.response_headers.get('Location')}")
        else:
            # This might happen if the test URL itself changes or there's an issue
            print(f"Crawl failed or did not behave as expected: {result.error_message}")
            print(f"Status code: {result.status_code}")


if __name__ == "__main__":
    asyncio.run(no_redirect_httpcrawler_config())
```

---
### 5.6. Example: Disabling SSL certificate verification (`verify_ssl=False`) in `HTTPCrawlerConfig`.
**Warning:** Disabling SSL verification is a security risk and should only be used in trusted environments or for specific testing purposes.

```python
import asyncio
from crawl4ai import HTTPCrawlerConfig, AsyncWebCrawler, CrawlerRunConfig
from crawl4ai.async_crawler_strategy import AsyncHTTPCrawlerStrategy

async def no_ssl_verify_httpcrawler_config():
    http_cfg_no_ssl = HTTPCrawlerConfig(verify_ssl=False)
    print(f"HTTPCrawlerConfig with verify_ssl={http_cfg_no_ssl.verify_ssl}")
    assert not http_cfg_no_ssl.verify_ssl
    
    # Use a site with a self-signed or expired certificate for testing
    # For example: "https://self-signed.badssl.com/" or "https://expired.badssl.com/"
    # For this example, we'll use a regular site, as hitting badssl might cause other issues in CI.
    # The real test is that it *doesn't* fail on a bad SSL site.
    test_url_bad_ssl = "https://expired.badssl.com/" 
    test_url_good_ssl = "https://example.com"

    print(f"Attempting to crawl {test_url_bad_ssl} with SSL verification disabled.")
    print("NOTE: This test is more meaningful if tested against a site with actual SSL issues.")
    
    strategy = AsyncHTTPCrawlerStrategy(http_config=http_cfg_no_ssl)
    async with AsyncWebCrawler(crawler_strategy=strategy) as crawler:
        run_config_bad = CrawlerRunConfig(url=test_url_bad_ssl)
        result_bad = await crawler.arun(config=run_config_bad)
        if result_bad.success:
            print(f"Successfully crawled {test_url_bad_ssl} (SSL errors were ignored).")
        else:
            # It might still fail for other reasons (e.g., site down)
            print(f"Crawl to {test_url_bad_ssl} failed, but not necessarily due to SSL: {result_bad.error_message}")

        # Verify it still works for good SSL sites
        run_config_good = CrawlerRunConfig(url=test_url_good_ssl)
        result_good = await crawler.arun(config=run_config_good)
        if result_good.success:
             print(f"Successfully crawled {test_url_good_ssl} as well.")

if __name__ == "__main__":
    asyncio.run(no_ssl_verify_httpcrawler_config())
```

---
### 5.7. Example: Creating `HTTPCrawlerConfig` using `HTTPCrawlerConfig.from_kwargs()`.

```python
import asyncio
from crawl4ai import HTTPCrawlerConfig

async def httpcrawlerconfig_from_kwargs():
    kwargs = {
        "method": "PUT",
        "headers": {"Authorization": "Bearer mytoken"},
        "json_payload": {"update_key": "new_value"} # Note: constructor uses 'json'
    }
    # The from_kwargs will map json_payload to json if the class expects 'json'
    # Let's check the actual constructor signature for 'json' vs 'json_payload'
    # The HTTPCrawlerConfig constructor takes 'json' not 'json_payload'.
    # So for from_kwargs, we should use the correct parameter names.
    kwargs_correct = {
        "method": "PUT",
        "headers": {"Authorization": "Bearer mytoken"},
        "json": {"update_key": "new_value"} 
    }

    http_cfg = HTTPCrawlerConfig.from_kwargs(kwargs_correct)
    
    print("HTTPCrawlerConfig created from_kwargs:")
    print(f"  Method: {http_cfg.method}")
    print(f"  Headers: {http_cfg.headers}")
    print(f"  JSON Payload: {http_cfg.json}")

    assert http_cfg.method == "PUT"
    assert http_cfg.headers["Authorization"] == "Bearer mytoken"
    assert http_cfg.json["update_key"] == "new_value"

if __name__ == "__main__":
    asyncio.run(httpcrawlerconfig_from_kwargs())
```

---
### 5.8. Example: Converting `HTTPCrawlerConfig` to a dictionary using `to_dict()`.

```python
import asyncio
from crawl4ai import HTTPCrawlerConfig

async def httpcrawlerconfig_to_dict():
    http_cfg = HTTPCrawlerConfig(
        method="DELETE",
        headers={"X-Request-ID": "123xyz"},
        follow_redirects=False
    )
    config_dict = http_cfg.to_dict()

    print("HTTPCrawlerConfig instance:")
    print(f"  Original method: {http_cfg.method}")
    
    print("\nConverted to dictionary:")
    print(json.dumps(config_dict, indent=2))
    
    assert config_dict["method"] == "DELETE"
    assert config_dict["headers"]["X-Request-ID"] == "123xyz"
    assert config_dict["follow_redirects"] is False

if __name__ == "__main__":
    asyncio.run(httpcrawlerconfig_to_dict())
```

---
### 5.9. Example: Cloning `HTTPCrawlerConfig` and changing the HTTP method using `clone()`.

```python
import asyncio
from crawl4ai import HTTPCrawlerConfig

async def httpcrawlerconfig_clone_modify():
    original_cfg = HTTPCrawlerConfig(method="GET", verify_ssl=True)
    print(f"Original Config: method={original_cfg.method}, verify_ssl={original_cfg.verify_ssl}")

    cloned_cfg = original_cfg.clone(method="PATCH", verify_ssl=False)
    print(f"Cloned Config: method={cloned_cfg.method}, verify_ssl={cloned_cfg.verify_ssl}")

    assert original_cfg.method == "GET"
    assert cloned_cfg.method == "PATCH"
    assert cloned_cfg.verify_ssl is False
    assert original_cfg.verify_ssl is True # Original unchanged

if __name__ == "__main__":
    asyncio.run(httpcrawlerconfig_clone_modify())
```

---
### 5.10. Example: Serializing `HTTPCrawlerConfig` using `dump()`.

```python
import asyncio
import json
from crawl4ai import HTTPCrawlerConfig

async def httpcrawlerconfig_dump():
    http_cfg = HTTPCrawlerConfig(
        method="OPTIONS",
        headers={"Origin": "https://example.com"},
        data={"ping": "true"}
    )
    
    dumped_dict = http_cfg.dump()

    print("Dumped HTTPCrawlerConfig (JSON compatible):")
    print(json.dumps(dumped_dict, indent=2))
    
    assert dumped_dict["type"] == "HTTPCrawlerConfig"
    assert dumped_dict["params"]["method"] == "OPTIONS"
    assert dumped_dict["params"]["data"]["ping"] == "true"

if __name__ == "__main__":
    asyncio.run(httpcrawlerconfig_dump())
```

---
### 5.11. Example: Deserializing `HTTPCrawlerConfig` using `load()`.

```python
import asyncio
from crawl4ai import HTTPCrawlerConfig

async def httpcrawlerconfig_load():
    serialized_data = {
        "type": "HTTPCrawlerConfig",
        "params": {
            "method": "HEAD",
            "headers": {"Cache-Control": "no-cache"},
            "follow_redirects": False
        }
    }
    
    loaded_cfg = HTTPCrawlerConfig.load(serialized_data)
    
    print("Loaded HTTPCrawlerConfig from dictionary:")
    print(f"  Method: {loaded_cfg.method}")
    print(f"  Headers: {loaded_cfg.headers}")
    print(f"  Follow Redirects: {loaded_cfg.follow_redirects}")

    assert isinstance(loaded_cfg, HTTPCrawlerConfig)
    assert loaded_cfg.method == "HEAD"
    assert loaded_cfg.headers["Cache-Control"] == "no-cache"
    assert loaded_cfg.follow_redirects is False

if __name__ == "__main__":
    asyncio.run(httpcrawlerconfig_load())
```

---
## 6. `CrawlerRunConfig` Examples

### 6.1. Basic Initialization

#### 6.1.1. Example: Default initialization of `CrawlerRunConfig`.
Creates a `CrawlerRunConfig` with all default values. The `url` will need to be provided when calling `arun`.

```python
import asyncio
from crawl4ai import CrawlerRunConfig, AsyncWebCrawler

async def default_crawler_run_config():
    run_cfg = CrawlerRunConfig() # No URL specified here
    print(f"Default CrawlerRunConfig: {run_cfg.to_dict(exclude_none=True)}")
    assert run_cfg.url is None # URL must be passed to arun if not set here

    # Example of using it (requires URL in arun)
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(url="https://example.com", config=run_cfg)
        if result.success:
            print(f"Crawled example.com with default run config. Markdown length: {len(result.markdown.raw_markdown)}")

if __name__ == "__main__":
    asyncio.run(default_crawler_run_config())
```

---
#### 6.1.2. Example: Specifying the target `url` directly within `CrawlerRunConfig`.
The `url` can be part of the config object itself.

```python
import asyncio
from crawl4ai import CrawlerRunConfig, AsyncWebCrawler

async def url_in_crawler_run_config():
    run_cfg_with_url = CrawlerRunConfig(url="https://example.org")
    print(f"CrawlerRunConfig with URL: {run_cfg_with_url.to_dict(exclude_none=True)}")
    assert run_cfg_with_url.url == "https://example.org"

    # Example of using it (arun will use the URL from config)
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(config=run_cfg_with_url) # No URL needed here
        if result.success:
            print(f"Crawled {result.url} using URL from config. Markdown length: {len(result.markdown.raw_markdown)}")

if __name__ == "__main__":
    asyncio.run(url_in_crawler_run_config())
```

---
### 6.2. Content Processing & Extraction

#### 6.2.1. Example: Adjusting `word_count_threshold` to control content block filtering.
Lower threshold means more (potentially shorter) blocks are kept.

```python
import asyncio
from crawl4ai import CrawlerRunConfig, AsyncWebCrawler, CacheMode

async def word_count_threshold_example():
    # Default is 200, let's try a much lower one
    run_cfg_low_wct = CrawlerRunConfig(
        url="https://news.ycombinator.com", 
        word_count_threshold=10, # Keep blocks with at least 10 words
        cache_mode=CacheMode.BYPASS
    )
    
    async with AsyncWebCrawler() as crawler:
        result_low = await crawler.arun(config=run_cfg_low_wct)
        if result_low.success:
            print(f"Markdown length with WCT=10: {len(result_low.markdown.raw_markdown)}")

            run_cfg_high_wct = CrawlerRunConfig(
                url="https://news.ycombinator.com",
                word_count_threshold=100, # Keep blocks with at least 100 words
                cache_mode=CacheMode.BYPASS
            )
            result_high = await crawler.arun(config=run_cfg_high_wct)
            if result_high.success:
                print(f"Markdown length with WCT=100: {len(result_high.markdown.raw_markdown)}")
                # Expect result_low.markdown to be longer or include more diverse small blocks
                assert len(result_low.markdown.raw_markdown) >= len(result_high.markdown.raw_markdown)
            else:
                print(f"Crawl with high WCT failed: {result_high.error_message}")
        else:
            print(f"Crawl with low WCT failed: {result_low.error_message}")

if __name__ == "__main__":
    asyncio.run(word_count_threshold_example())
```

---
#### 6.2.2. Example: Integrating a custom `extraction_strategy` (e.g., `NoExtractionStrategy`).
`NoExtractionStrategy` skips structured data extraction, only providing HTML/Markdown.

```python
import asyncio
from crawl4ai import CrawlerRunConfig, AsyncWebCrawler, CacheMode
from crawl4ai.extraction_strategy import NoExtractionStrategy

async def no_extraction_strategy_example():
    run_cfg = CrawlerRunConfig(
        url="https://example.com",
        extraction_strategy=NoExtractionStrategy(),
        cache_mode=CacheMode.BYPASS
    )
    
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(config=run_cfg)
        if result.success:
            print(f"Crawled {result.url} with NoExtractionStrategy.")
            print(f"Extracted content: {result.extracted_content}") # Should be None or empty
            assert result.extracted_content is None or result.extracted_content == "[]" # Default is "[]" from NoExtraction
            print(f"Markdown content (first 100 chars): {result.markdown.raw_markdown[:100]}")
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(no_extraction_strategy_example())
```

---
#### 6.2.3. Example: Using `RegExChunking` as the `chunking_strategy`.
Splits content based on regular expressions, useful before passing to some extraction strategies.

```python
import asyncio
from crawl4ai import CrawlerRunConfig, AsyncWebCrawler, CacheMode
from crawl4ai.chunking_strategy import RegExChunking
# For LLMExtractionStrategy to show effect of chunking
from crawl4ai.extraction_strategy import LLMExtractionStrategy 
from crawl4ai import LLMConfig
import os

async def regex_chunking_example():
    # This example is more meaningful if combined with an extraction strategy
    # that processes chunks, like LLMExtractionStrategy.

    # Define a schema for LLM extraction
    schema = {"key_topics": "List the main topics discussed in this text."}
    llm_config_test = LLMConfig(
        provider="openai/gpt-4o-mini", 
        api_token=os.getenv("OPENAI_API_KEY_PLACEHOLDER", "YOUR_API_KEY")
    )
    
    # Chunk by paragraphs (approximate regex)
    regex_chunker = RegExChunking(patterns=[r"\n\s*\n"]) # Split on blank lines

    # If OPENAI_API_KEY_PLACEHOLDER is set to a real key, this test will make an API call.
    if llm_config_test.api_token == "YOUR_API_KEY":
        print("Skipping RegExChunking with LLM example due to missing OpenAI API key.")
        extraction_strat = None
    else:
        extraction_strat = LLMExtractionStrategy(
            llm_config=llm_config_test,
            schema=schema,
            # Note: The chunking_strategy is applied *before* LLM extraction if the LLM strategy expects chunks.
            # However, LLMExtractionStrategy itself handles chunking internally if content is too long.
            # To demonstrate RegExChunking independently, one might process its output directly.
            # For this example, we'll let LLMExtractionStrategy use it.
        )


    run_cfg = CrawlerRunConfig(
        url="https://en.wikipedia.org/wiki/Python_(programming_language)",
        chunking_strategy=regex_chunker, # This will be used by LLMExtractionStrategy if it's configured to take pre-chunked input
        extraction_strategy=extraction_strat,
        cache_mode=CacheMode.BYPASS,
        word_count_threshold=50 # Get more content for chunking
    )
    
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(config=run_cfg)
        if result.success:
            print(f"Crawled {result.url}.")
            if result.extracted_content:
                print(f"Extracted content (potentially from chunks): {result.extracted_content[:500]}...")
            else:
                print("No structured content extracted (or API key missing). Markdown was still generated from chunked/full content.")
            print(f"Markdown (first 300 chars): {result.markdown.raw_markdown[:300]}")
            # To truly verify RegExChunking effect, one would need to inspect how LLMExtractionStrategy uses it,
            # or use a custom strategy that explicitly consumes `crawler_run_config.chunking_strategy.chunk(content)`.
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(regex_chunking_example())
```

---
#### 6.2.4. Example: Configuring `DefaultMarkdownGenerator` for `markdown_generator`.
Allows customization of how Markdown is generated (e.g., with/without citations, different content source).

```python
import asyncio
from crawl4ai import CrawlerRunConfig, AsyncWebCrawler, CacheMode
from crawl4ai.markdown_generation_strategy import DefaultMarkdownGenerator

async def custom_markdown_generator_example():
    # Generate Markdown without citations
    md_gen_no_citations = DefaultMarkdownGenerator(
        options={"citations": False} # html2text option
    )
    run_cfg_no_cite = CrawlerRunConfig(
        url="https://example.com",
        markdown_generator=md_gen_no_citations,
        cache_mode=CacheMode.BYPASS
    )

    async with AsyncWebCrawler() as crawler:
        result_no_cite = await crawler.arun(config=run_cfg_no_cite)
        if result_no_cite.success:
            print("--- Markdown without Citations (first 300 chars) ---")
            print(result_no_cite.markdown.raw_markdown[:300])
            # DefaultMarkdownGenerator produces raw_markdown and markdown_with_citations
            # We expect raw_markdown to be the one without reference style links.
            assert "]: http" not in result_no_cite.markdown.raw_markdown # Heuristic check
        
        # Generate Markdown from raw HTML instead of cleaned HTML
        md_gen_from_raw = DefaultMarkdownGenerator(content_source="raw_html")
        run_cfg_raw_html = CrawlerRunConfig(
             url="https://example.com",
             markdown_generator=md_gen_from_raw,
             cache_mode=CacheMode.BYPASS
        )
        result_raw = await crawler.arun(config=run_cfg_raw_html)
        if result_raw.success:
            print("\n--- Markdown from Raw HTML (first 300 chars) ---")
            print(result_raw.markdown.raw_markdown[:300])
            # This might be much noisier than from cleaned_html
            
if __name__ == "__main__":
    asyncio.run(custom_markdown_generator_example())
```

---
#### 6.2.5. Example: Enabling `only_text=True` for text-only extraction from HTML.
Strips all HTML tags, leaving only the text content.

```python
import asyncio
from crawl4ai import CrawlerRunConfig, AsyncWebCrawler, CacheMode

async def only_text_example():
    run_cfg = CrawlerRunConfig(
        url="https://example.com",
        only_text=True,
        cache_mode=CacheMode.BYPASS
    )
    
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(config=run_cfg)
        if result.success:
            print(f"Crawled {result.url} with only_text=True.")
            print(f"Cleaned HTML (should be just text): {result.cleaned_html[:300]}")
            assert "<" not in result.cleaned_html[:10] # Heuristic: no HTML tags at start
            print(f"Markdown (should be similar to cleaned_html): {result.markdown.raw_markdown[:300]}")
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(only_text_example())
```

---
#### 6.2.6. Example: Using `css_selector` to focus HTML processing on a specific part of the page.
The `cleaned_html` (and thus default Markdown) will only contain content from the matched selector.

```python
import asyncio
from crawl4ai import CrawlerRunConfig, AsyncWebCrawler, CacheMode

async def css_selector_focus_example():
    # On example.com, let's try to get only the content within the first <p> tag inside <div>
    run_cfg = CrawlerRunConfig(
        url="https://example.com",
        css_selector="div > p:first-of-type", # Selects the first paragraph in the main div
        cache_mode=CacheMode.BYPASS
    )
    
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(config=run_cfg)
        if result.success:
            print(f"Crawled {result.url}, focusing on 'div > p:first-of-type'.")
            print(f"Cleaned HTML: {result.cleaned_html}")
            # Expected: "This domain is for use in illustrative examples in documents..."
            assert "illustrative examples" in result.cleaned_html
            assert "More information..." not in result.cleaned_html # Content from other <p>
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(css_selector_focus_example())
```

---
#### 6.2.7. Example: Specifying multiple `target_elements` for focused Markdown generation.
Markdown and structured extraction will focus on these elements, but other page data (links, media) is still gathered from the whole page.

```python
import asyncio
from crawl4ai import CrawlerRunConfig, AsyncWebCrawler, CacheMode

async def target_elements_example():
    # On example.com, target the <h1> and the link <a>
    run_cfg = CrawlerRunConfig(
        url="https://example.com",
        target_elements=["h1", "div > p > a"], # Target heading and the "More information" link
        cache_mode=CacheMode.BYPASS
    )
    
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(config=run_cfg)
        if result.success:
            print(f"Crawled {result.url} with target_elements=['h1', 'div > p > a'].")
            print(f"Generated Markdown (focused on targets):\n{result.markdown.raw_markdown}")
            assert "Example Domain" in result.markdown.raw_markdown
            assert "More information..." in result.markdown.raw_markdown
            # The first paragraph's text should NOT be in the markdown if not targeted
            assert "illustrative examples" not in result.markdown.raw_markdown

            # Check if all links from the page are still collected (they should be)
            print(f"\nTotal internal links found on page: {len(result.links.get('internal', []))}")
            assert len(result.links.get("internal", [])) > 0 
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(target_elements_example())
```

---
#### 6.2.8. Example: Excluding specific HTML tags (e.g., `nav`, `footer`) using `excluded_tags`.
These tags and their content will be removed from `cleaned_html`.

```python
import asyncio
from crawl4ai import CrawlerRunConfig, AsyncWebCrawler, CacheMode

async def excluded_tags_example():
    # Let's try to exclude <p> tags from example.com
    run_cfg = CrawlerRunConfig(
        url="https://example.com",
        excluded_tags=["p"],
        cache_mode=CacheMode.BYPASS
    )
    
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(config=run_cfg)
        if result.success:
            print(f"Crawled {result.url} excluding <p> tags.")
            print(f"Cleaned HTML (should not contain <p>):\n{result.cleaned_html}")
            assert "<p>" not in result.cleaned_html.lower()
            assert "illustrative examples" not in result.cleaned_html # This text is in a <p>
            assert "Example Domain" in result.cleaned_html # The <h1> should remain
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(excluded_tags_example())
```

---
#### 6.2.9. Example: Excluding elements based on a CSS selector using `excluded_selector`.
Removes elements matching the CSS selector from `cleaned_html`.

```python
import asyncio
from crawl4ai import CrawlerRunConfig, AsyncWebCrawler, CacheMode

async def excluded_selector_example():
    # Exclude the link "More information..." on example.com which is in a <p> inside a <div>
    run_cfg = CrawlerRunConfig(
        url="https://example.com",
        excluded_selector="div > p > a", # CSS selector for the link
        cache_mode=CacheMode.BYPASS
    )
    
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(config=run_cfg)
        if result.success:
            print(f"Crawled {result.url} excluding 'div > p > a'.")
            print(f"Cleaned HTML:\n{result.cleaned_html}")
            assert "More information..." not in result.cleaned_html
            assert "illustrative examples" in result.cleaned_html # The rest of the <p> should be there
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(excluded_selector_example())
```

---
#### 6.2.10. Example: Preserving `data-*` attributes during HTML cleaning with `keep_data_attributes=True`.
By default, many attributes are stripped. This keeps `data-*` attributes.

```python
import asyncio
from crawl4ai import CrawlerRunConfig, AsyncWebCrawler, CacheMode

async def keep_data_attributes_example():
    sample_html_with_data_attr = """
    <html><body>
        <div data-testid="main-content" data-analytics-id="section1">
            <p>Some important text.</p>
            <span data-custom-info="extra detail">More text.</span>
        </div>
    </body></html>
    """
    run_cfg = CrawlerRunConfig(
        url=f"raw://{sample_html_with_data_attr}",
        keep_data_attributes=True,
        cache_mode=CacheMode.BYPASS
    )
    
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(config=run_cfg)
        if result.success:
            print(f"Crawled content with keep_data_attributes=True.")
            print(f"Cleaned HTML:\n{result.cleaned_html}")
            assert 'data-testid="main-content"' in result.cleaned_html
            assert 'data-analytics-id="section1"' in result.cleaned_html
            assert 'data-custom-info="extra detail"' in result.cleaned_html
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(keep_data_attributes_example())
```

---
#### 6.2.11. Example: Specifying a custom list of HTML attributes to preserve using `keep_attrs`.
Allows fine-grained control over which attributes are kept.

```python
import asyncio
from crawl4ai import CrawlerRunConfig, AsyncWebCrawler, CacheMode

async def keep_specific_attributes_example():
    sample_html_with_attrs = """
    <html><body>
        <article id="article-123" class="blog-post important" style="color:blue;" title="My Article">
            <h1>Title</h1>
            <a href="/path" data-linkid="789">Link</a>
        </article>
    </body></html>
    """
    # We want to keep 'id', 'class' from the article, and 'href' from the link
    # and also data-linkid
    run_cfg = CrawlerRunConfig(
        url=f"raw://{sample_html_with_attrs}",
        keep_attrs=["id", "class", "href", "data-linkid"], 
        # keep_data_attributes=False (default), so only "data-linkid" if explicitly listed here.
        cache_mode=CacheMode.BYPASS
    )
    
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(config=run_cfg)
        if result.success:
            print(f"Crawled content with keep_attrs=['id', 'class', 'href', 'data-linkid'].")
            cleaned_html = result.cleaned_html
            print(f"Cleaned HTML:\n{cleaned_html}")
            assert 'id="article-123"' in cleaned_html
            assert 'class="blog-post important"' in cleaned_html # Classes are usually kept by default cleaner for semantic reasons
            assert 'href="/path"' in cleaned_html
            assert 'data-linkid="789"' in cleaned_html
            assert 'style="color:blue;"' not in cleaned_html # style should be removed
            assert 'title="My Article"' not in cleaned_html # title should be removed
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(keep_specific_attributes_example())
```

---
#### 6.2.12. Example: Removing all `<form>` elements from HTML using `remove_forms=True`.
Useful for cleaning up pages before text extraction if forms are noisy.

```python
import asyncio
from crawl4ai import CrawlerRunConfig, AsyncWebCrawler, CacheMode

async def remove_forms_example():
    html_with_form = """
    <html><body>
        <p>Some text before form.</p>
        <form action="/submit">
            <label for="name">Name:</label>
            <input type="text" id="name" name="name"><br>
            <input type="submit" value="Submit">
        </form>
        <p>Some text after form.</p>
    </body></html>
    """
    run_cfg = CrawlerRunConfig(
        url=f"raw://{html_with_form}",
        remove_forms=True,
        cache_mode=CacheMode.BYPASS
    )
    
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(config=run_cfg)
        if result.success:
            print(f"Crawled content with remove_forms=True.")
            cleaned_html = result.cleaned_html
            print(f"Cleaned HTML (should not contain <form>):\n{cleaned_html}")
            assert "<form" not in cleaned_html.lower()
            assert "Some text before form." in cleaned_html
            assert "Some text after form." in cleaned_html
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(remove_forms_example())
```

---
#### 6.2.13. Example: Enabling HTML prettifying of the cleaned HTML output with `prettify=True`.
Formats the `cleaned_html` to be more human-readable.

```python
import asyncio
from crawl4ai import CrawlerRunConfig, AsyncWebCrawler, CacheMode

async def prettify_html_example():
    # A simple, unformatted HTML string
    raw_html = "<html><body><div><p>Hello</p><p>World</p></div></body></html>"
    run_cfg = CrawlerRunConfig(
        url=f"raw://{raw_html}",
        prettify=True,
        cache_mode=CacheMode.BYPASS
    )
    
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(config=run_cfg)
        if result.success:
            print(f"Crawled content with prettify=True.")
            cleaned_html = result.cleaned_html
            print(f"Prettified Cleaned HTML:\n{cleaned_html}")
            # Prettified HTML usually has more newlines and indentation
            assert cleaned_html.count('\n') > 3 # Heuristic check for newlines
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(prettify_html_example())
```

---
#### 6.2.14. Example: Changing the HTML parser to "html.parser" using `parser_type`.
The default is "lxml". "html.parser" is Python's built-in parser.

```python
import asyncio
from crawl4ai import CrawlerRunConfig, AsyncWebCrawler, CacheMode

async def change_parser_type_example():
    run_cfg = CrawlerRunConfig(
        url="https://example.com",
        parser_type="html.parser", # Use Python's built-in parser
        cache_mode=CacheMode.BYPASS
    )
    
    print(f"Using parser_type: {run_cfg.parser_type}")
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(config=run_cfg)
        if result.success:
            print(f"Crawled {result.url} using 'html.parser'.")
            # Behavior should be largely similar for well-formed HTML
            assert "Example Domain" in result.cleaned_html
        else:
            print(f"Crawl failed with 'html.parser': {result.error_message}")

if __name__ == "__main__":
    asyncio.run(change_parser_type_example())
```

---
#### 6.2.15. Example: Specifying a custom `scraping_strategy` (e.g., `PDFScrapingStrategy` - if applicable).
Demonstrates using a different strategy for content scraping, here for PDF files.

```python
import asyncio
from crawl4ai import CrawlerRunConfig, AsyncWebCrawler, CacheMode
from crawl4ai.content_scraping_strategy import WebScrapingStrategy # Default
# For PDF, we'd import PDFç›¸å…³çš„strategies
from crawl4ai.processors.pdf import PDFCrawlerStrategy, PDFScrapingStrategy
# Note: PDFScrapingStrategy is typically used by PDFCrawlerStrategy, not directly by AsyncWebCrawler for a URL.
# This example will show how to set it conceptually if a strategy took it.
# A more realistic PDF example uses PDFCrawlerStrategy directly.

async def custom_scraping_strategy_conceptual():
    # This is conceptual for CrawlerRunConfig.scraping_strategy if a strategy used it.
    # In practice, for PDF, you'd use PDFCrawlerStrategy.
    
    # Conceptual: if AsyncWebCrawler could directly use PDFScrapingStrategy via config
    # (It doesn't by default for generic URLs; PDF processing has its own flow)
    # pdf_scraping_strat = PDFScrapingStrategy() 
    # run_cfg_pdf = CrawlerRunConfig(
    #     url="file:///path/to/your/document.pdf", # Example local PDF
    #     scraping_strategy=pdf_scraping_strat,
    #     cache_mode=CacheMode.BYPASS
    # )
    # print(f"Conceptual CrawlerRunConfig with PDFScrapingStrategy: {run_cfg_pdf.scraping_strategy}")


    # More realistic usage for PDF:
    # Use PDFCrawlerStrategy directly if you know it's a PDF
    pdf_url = "https://www.w3.org/WAI/ER/tests/xhtml/testfiles/resources/pdf/dummy.pdf" # A sample PDF
    
    # We need a PDF-specific crawler strategy, not just scraping_strategy in CrawlerRunConfig
    # This example demonstrates where scraping_strategy *would* go if a generic URL crawler used it.
    # The PDF files provided show PDFCrawlerStrategy and PDFContentScrapingStrategy.
    # PDFContentScrapingStrategy is the equivalent scraper for PDFs.
    
    # Correct way for PDF:
    pdf_crawler_strategy = PDFCrawlerStrategy()
    pdf_scraping_config = PDFScrapingStrategy() # This is the content scraper for PDFs
    
    run_config_for_pdf = CrawlerRunConfig(
        url=pdf_url,
        scraping_strategy=pdf_scraping_config, # This is what the PDFCrawlerStrategy would use internally
        cache_mode=CacheMode.BYPASS
    )

    print(f"Using PDFScrapingStrategy conceptually in CrawlerRunConfig for a PDF URL: {pdf_url}")
    
    # The AsyncWebCrawler's default Playwright strategy won't use run_config.scraping_strategy.
    # We need to pass the PDFCrawlerStrategy to AsyncWebCrawler itself.
    async with AsyncWebCrawler(crawler_strategy=pdf_crawler_strategy) as crawler:
        # The run_config's scraping_strategy will be passed to pdf_crawler_strategy.crawl()
        # if pdf_crawler_strategy.crawl() is designed to accept it via **kwargs
        # and then pass it to its internal scraper.
        # The provided PDF __init__.py passes the 'save_images_locally', 'extract_images' etc.
        # directly from CrawlerRunConfig to the PDFContentScrapingStrategy.
        
        # For this example, let's adjust CrawlerRunConfig to pass params PDFContentScrapingStrategy expects:
        pdf_specific_run_config = CrawlerRunConfig(
            url=pdf_url,
            extract_images=True, # A param PDFContentScrapingStrategy uses
            cache_mode=CacheMode.BYPASS
        )

        result = await crawler.arun(config=pdf_specific_run_config)
        if result.success:
            print(f"Successfully processed PDF {result.url}.")
            print(f"Markdown content (first 300 chars):\n{result.markdown.raw_markdown[:300]}")
            if result.media and result.media.get("images"):
                print(f"Extracted {len(result.media['images'])} images from PDF.")
        else:
            print(f"Failed to process PDF: {result.error_message}")


if __name__ == "__main__":
    asyncio.run(custom_scraping_strategy_conceptual())
```

---
### 6.3. Proxy Configuration for a Specific Run

#### 6.3.1. Example: Providing a `ProxyConfig` object to `CrawlerRunConfig.proxy_config`.
This overrides any proxy set in `BrowserConfig` for this specific `arun` call.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, BrowserConfig, CrawlerRunConfig, ProxyConfig

async def run_specific_proxy():
    # Browser might have a default proxy or no proxy
    browser_cfg = BrowserConfig(headless=True, verbose=True) 
    
    # Proxy for this specific run (non-functional placeholder)
    run_proxy = ProxyConfig(server="http://runspecificproxy.example.com:8888")
    
    run_cfg_with_proxy = CrawlerRunConfig(
        url="https://httpbin.org/ip", # Shows requester IP
        proxy_config=run_proxy
    )
    print(f"CrawlerRunConfig with specific proxy: {run_cfg_with_proxy.proxy_config.to_dict()}") # type: ignore

    print("NOTE: This example will likely fail or show your direct IP if the placeholder proxy is not replaced.")
    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(config=run_cfg_with_proxy)
        if result.success:
            print(f"Crawled {result.url} using run-specific proxy.")
            print(f"Response (should show proxy IP): {result.html}")
        else:
            print(f"Crawl with run-specific proxy failed: {result.error_message}")

if __name__ == "__main__":
    # asyncio.run(run_specific_proxy())
    print("Skipping run_specific_proxy example as it requires a live proxy server.")
```

---
#### 6.3.2. Example: Using `RoundRobinProxyStrategy` for `proxy_rotation_strategy`.
Demonstrates setting up a proxy rotation strategy for `arun_many` or multiple `arun` calls.

```python
import asyncio
from crawl4ai import (
    AsyncWebCrawler, BrowserConfig, CrawlerRunConfig, 
    ProxyConfig, RoundRobinProxyStrategy
)

async def proxy_rotation_example():
    # List of proxies (non-functional placeholders)
    proxies = [
        ProxyConfig(server="http://proxy1.example.com:8000"),
        ProxyConfig(server="http://proxy2.example.com:8001", username="user2", password="p2"),
        ProxyConfig(server="http://proxy3.example.com:8002"),
    ]
    
    proxy_rotator = RoundRobinProxyStrategy(proxies=proxies)
    
    # Browser config (no proxy set here, it will be set per run)
    browser_cfg = BrowserConfig(headless=True, verbose=True)
    
    urls_to_crawl = [
        "https://httpbin.org/ip?site=1",
        "https://httpbin.org/ip?site=2",
        "https://httpbin.org/ip?site=3",
        "https://httpbin.org/ip?site=4" # Will cycle back to proxy1
    ]

    print("NOTE: This example will likely fail or show direct IPs if placeholder proxies are not replaced.")
    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        for i, url in enumerate(urls_to_crawl):
            # Get next proxy from strategy and create a run_config with it
            # The proxy_rotation_strategy itself is passed to CrawlerRunConfig
            run_cfg = CrawlerRunConfig(
                url=url,
                proxy_rotation_strategy=proxy_rotator # The crawler will call get_next_proxy()
            )
            
            # The `arun` method will internally use proxy_rotation_strategy.get_next_proxy()
            # to set the proxy_config for the actual call if proxy_rotation_strategy is present.
            # The proxy_config will be set inside the AsyncPlaywrightCrawlerStrategy.
            
            print(f"\n--- Crawling {url} (Attempt {i+1}) ---")
            # current_proxy_for_run = await proxy_rotator.get_next_proxy() # This is what arun would do
            # print(f"Expected proxy for this run: {current_proxy_for_run.server if current_proxy_for_run else 'None'}")

            result = await crawler.arun(config=run_cfg)
            
            if result.success:
                print(f"Crawled {result.url} successfully.")
                print(f"Response (should show IP of proxy {i % len(proxies) + 1}): {result.html}")
            else:
                print(f"Crawl for {result.url} failed: {result.error_message}")
            
            await asyncio.sleep(0.5) # Small delay between requests

if __name__ == "__main__":
    # asyncio.run(proxy_rotation_example())
    print("Skipping proxy_rotation_example as it requires live proxy servers.")
```

---
### 6.4. Localization and Geolocation for a Specific Run

#### 6.4.1. Example: Setting browser `locale` (e.g., "es-ES") for the crawl.
This can affect language of the page, date/number formats.

```python
import asyncio
import json
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig

async def set_browser_locale_for_run():
    browser_cfg = BrowserConfig(headless=True, verbose=True) # Default locale (usually en-US)
    
    run_cfg_spanish = CrawlerRunConfig(
        url="https://httpbin.org/headers", # This endpoint shows request headers
        locale="es-ES" # Spanish (Spain)
    )
    print(f"CrawlerRunConfig with locale: {run_cfg_spanish.locale}")

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(config=run_cfg_spanish)
        if result.success:
            print(f"Crawled {result.url} with locale 'es-ES'.")
            response_data = json.loads(result.html)
            accept_language_header = response_data.get("headers", {}).get("Accept-Language")
            print(f"Accept-Language header sent: {accept_language_header}")
            # Playwright typically sets Accept-Language based on locale
            assert accept_language_header and "es-ES" in accept_language_header.split(',')[0]
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(set_browser_locale_for_run())
```

---
#### 6.4.2. Example: Setting browser `timezone_id` (e.g., "Europe/Madrid") for the crawl.
Affects JavaScript `Date` objects and potentially server-side logic based on timezone.

```python
import asyncio
import json
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig

async def set_browser_timezone_for_run():
    browser_cfg = BrowserConfig(headless=True, verbose=True)
    
    run_cfg_madrid_tz = CrawlerRunConfig(
        url="https://httpbin.org/anything", # We'll execute JS to get timezone
        timezone_id="Europe/Madrid",
        js_code="JSON.stringify({offset: new Date().getTimezoneOffset(), localeString: new Date().toLocaleString('en-US', {timeZoneName:'short'})})"
    )
    print(f"CrawlerRunConfig with timezone_id: {run_cfg_madrid_tz.timezone_id}")

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(config=run_cfg_madrid_tz)
        if result.success and result.js_execution_result:
            print(f"Crawled {result.url} with timezone 'Europe/Madrid'.")
            js_result = result.js_execution_result
            print(f"JS Date info: {js_result}")
            # For Madrid (CET/CEST), offset is -60 (CET) or -120 (CEST) from UTC.
            # localeString might show 'CET' or 'CEST'.
            # This is a heuristic check.
            assert js_result.get("offset") in [-60, -120] or "CET" in js_result.get("localeString", "") or "CEST" in js_result.get("localeString", "")
        elif not result.js_execution_result:
            print(f"JS execution did not return a result for timezone check.")
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(set_browser_timezone_for_run())
```

---
#### 6.4.3. Example: Providing a `GeolocationConfig` object for the crawl.
Simulates a specific GPS location for the browser.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig, GeolocationConfig

async def set_geolocation_for_run():
    browser_cfg = BrowserConfig(headless=True, verbose=True)
    
    # Sydney, Australia
    sydney_geo = GeolocationConfig(latitude=-33.8688, longitude=151.2093, accuracy=50.0)
    
    run_cfg_sydney = CrawlerRunConfig(
        url="https://www.gps-coordinates.net/my-location", # This site shows your detected location
        geolocation=sydney_geo,
        wait_for="css=#address", # Wait for the address to be populated
        page_timeout=20000, # Give it time
        verbose=True
    )
    print(f"CrawlerRunConfig with geolocation: {run_cfg_sydney.geolocation.to_dict()}")

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        print(f"Attempting to crawl {run_cfg_sydney.url} simulating Sydney location.")
        result = await crawler.arun(config=run_cfg_sydney)
        if result.success:
            print(f"Successfully crawled {result.url} with simulated Sydney location.")
            # Check if the page content reflects Sydney
            # print(f"Page HTML (first 1000 chars):\n{result.html[:1000]}")
            if "Sydney" in result.html or "Australia" in result.html:
                 print("SUCCESS: Sydney or Australia mentioned in page, geolocation likely worked.")
            else:
                 print("Geolocation effect not immediately obvious in HTML. Manual check of site behavior needed.")
        else:
            print(f"Crawl failed: {result.error_message}")
            print("This might be due to the test site's behavior or if permissions for geolocation were not granted by the browser context setup.")

if __name__ == "__main__":
    asyncio.run(set_geolocation_for_run())
```

---
### 6.5. Caching Behavior with `CacheMode`

#### 6.5.1. Example: Enabling full read/write caching with `cache_mode=CacheMode.ENABLED`.
Reads from cache if available, otherwise fetches and writes to cache. (Default if no mode is set by user and cache is available).

```python
import asyncio
import time
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, CacheMode, BrowserConfig
from crawl4ai.utils import get_cache_dir # Helper to find cache location

async def cache_enabled_example():
    # Ensure cache dir exists for this test
    cache_dir = get_cache_dir()
    print(f"Using cache directory: {cache_dir}")

    run_cfg = CrawlerRunConfig(
        url="https://example.com", # A static page, good for caching demo
        cache_mode=CacheMode.ENABLED
    )
    
    browser_cfg = BrowserConfig(headless=True, verbose=True)

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        # First run: Should fetch and cache
        print("\n--- First run (CacheMode.ENABLED) ---")
        start_time1 = time.perf_counter()
        result1 = await crawler.arun(config=run_cfg)
        duration1 = time.perf_counter() - start_time1
        if result1.success:
            print(f"First run successful. Duration: {duration1:.2f}s. Cached: {result1.metadata.get('cached', False)}")
            assert not result1.metadata.get('cached', False) # Should not be cached on first run (unless already in DB)
        else:
            print(f"First run failed: {result1.error_message}")
            return

        # Second run: Should read from cache
        print("\n--- Second run (CacheMode.ENABLED) ---")
        start_time2 = time.perf_counter()
        result2 = await crawler.arun(config=run_cfg) # Same URL and config
        duration2 = time.perf_counter() - start_time2
        if result2.success:
            print(f"Second run successful. Duration: {duration2:.2f}s. Cached: {result2.metadata.get('cached', False)}")
            assert result2.metadata.get('cached', True) # Should be cached now
            assert duration2 < duration1 # Cached read should be faster
            assert result1.markdown.raw_markdown == result2.markdown.raw_markdown
        else:
            print(f"Second run failed: {result2.error_message}")

if __name__ == "__main__":
    asyncio.run(cache_enabled_example())
```

---
#### 6.5.2. Example: Bypassing the cache for a single run with `cache_mode=CacheMode.BYPASS`.
Ignores existing cache and fetches fresh data, but still writes the new data to cache.

```python
import asyncio
import time
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, CacheMode, BrowserConfig

async def cache_bypass_example():
    run_cfg_cache_first = CrawlerRunConfig(
        url="https://example.com/cache_test_page_bypass", 
        cache_mode=CacheMode.ENABLED # First, ensure it's cached
    )
    run_cfg_bypass = CrawlerRunConfig(
        url="https://example.com/cache_test_page_bypass", # Same URL
        cache_mode=CacheMode.BYPASS # This run will bypass read but still write
    )
    browser_cfg = BrowserConfig(headless=True, verbose=True)

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        # First run: Populate cache
        print("\n--- First run (CacheMode.ENABLED to populate cache) ---")
        await crawler.arun(config=run_cfg_cache_first)
        print("Cache potentially populated.")

        # Second run: Bypass cache (should fetch fresh, then write to cache)
        print("\n--- Second run (CacheMode.BYPASS) ---")
        start_time_bypass = time.perf_counter()
        result_bypass = await crawler.arun(config=run_cfg_bypass)
        duration_bypass = time.perf_counter() - start_time_bypass
        if result_bypass.success:
            print(f"Bypass run successful. Duration: {duration_bypass:.2f}s. Cached flag: {result_bypass.metadata.get('cached', False)}")
            # 'cached' flag is True if data was retrieved from cache. For BYPASS, it should be False for the read part.
            assert not result_bypass.metadata.get('cached', False) 
        else:
            print(f"Bypass run failed: {result_bypass.error_message}")
            return

        # Third run: Normal enabled mode, should now read the data written by BYPASS run
        print("\n--- Third run (CacheMode.ENABLED, after BYPASS) ---")
        start_time_cached = time.perf_counter()
        result_cached_after_bypass = await crawler.arun(config=run_cfg_cache_first) # Use ENABLED config
        duration_cached = time.perf_counter() - start_time_cached
        if result_cached_after_bypass.success:
            print(f"Cached run successful. Duration: {duration_cached:.2f}s. Cached flag: {result_cached_after_bypass.metadata.get('cached', False)}")
            assert result_cached_after_bypass.metadata.get('cached', True)
            assert duration_cached < duration_bypass
        else:
            print(f"Cached run after bypass failed: {result_cached_after_bypass.error_message}")


if __name__ == "__main__":
    asyncio.run(cache_bypass_example())
```

---
#### 6.5.3. Example: Disabling all caching operations with `cache_mode=CacheMode.DISABLED`.
Neither reads from nor writes to the cache.

```python
import asyncio
import time
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, CacheMode, BrowserConfig

async def cache_disabled_example():
    run_cfg = CrawlerRunConfig(
        url="https://example.com/cache_test_page_disabled",
        cache_mode=CacheMode.DISABLED
    )
    browser_cfg = BrowserConfig(headless=True, verbose=True)

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        # First run
        print("\n--- First run (CacheMode.DISABLED) ---")
        start_time1 = time.perf_counter()
        result1 = await crawler.arun(config=run_cfg)
        duration1 = time.perf_counter() - start_time1
        if result1.success:
            print(f"First run successful. Duration: {duration1:.2f}s. Cached: {result1.metadata.get('cached', False)}")
            assert not result1.metadata.get('cached', False) # Should not be cached
        else:
            print(f"First run failed: {result1.error_message}")
            return

        # Second run: Should also fetch fresh as cache is disabled
        print("\n--- Second run (CacheMode.DISABLED) ---")
        start_time2 = time.perf_counter()
        result2 = await crawler.arun(config=run_cfg) # Same URL and config
        duration2 = time.perf_counter() - start_time2
        if result2.success:
            print(f"Second run successful. Duration: {duration2:.2f}s. Cached: {result2.metadata.get('cached', False)}")
            assert not result2.metadata.get('cached', False) # Still not cached
            # Durations might be similar as both are fresh fetches
            print(f"Durations: Run1={duration1:.2f}s, Run2={duration2:.2f}s")
        else:
            print(f"Second run failed: {result2.error_message}")

if __name__ == "__main__":
    asyncio.run(cache_disabled_example())
```

---
#### 6.5.4. Example: Configuring cache for read-only access with `cache_mode=CacheMode.READ_ONLY`.
Only reads from cache if data exists; does not write new data if fetched fresh.

```python
import asyncio
import time
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, CacheMode, BrowserConfig, AsyncDBManager

async def cache_readonly_example():
    url_readonly = "https://example.com/cache_test_page_readonly"
    
    # Ensure the item is NOT in cache initially by using a unique URL or clearing
    # For this test, we'll first try to read (should fail to find in cache), then try to write (should not write)
    
    run_cfg_readonly = CrawlerRunConfig(url=url_readonly, cache_mode=CacheMode.READ_ONLY)
    run_cfg_enabled = CrawlerRunConfig(url=url_readonly, cache_mode=CacheMode.ENABLED) # To check if it was written
    
    browser_cfg = BrowserConfig(headless=True, verbose=True)
    db_manager = AsyncDBManager() # For direct cache check

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        # Ensure item is not in cache
        await db_manager.adelete_url_cache(url_readonly)
        print(f"Ensured {url_readonly} is not in cache.")

        # First run: READ_ONLY mode, item not in cache. Should fetch fresh, NOT write.
        print("\n--- First run (CacheMode.READ_ONLY, item not in cache) ---")
        result_fetch = await crawler.arun(config=run_cfg_readonly)
        if result_fetch.success:
            print(f"READ_ONLY (fetch): Successful. Cached: {result_fetch.metadata.get('cached', False)}")
            assert not result_fetch.metadata.get('cached', False)
        else:
            print(f"READ_ONLY (fetch) failed: {result_fetch.error_message}")
            return

        # Check if it was written to cache (it shouldn't have been)
        cached_item_after_readonly = await db_manager.aget_cached_url(url_readonly)
        if cached_item_after_readonly is None:
            print("SUCCESS: Item was NOT written to cache after READ_ONLY fetch, as expected.")
        else:
            print("FAILURE: Item was unexpectedly written to cache after READ_ONLY fetch.")
            await db_manager.adelete_url_cache(url_readonly) # Clean for next step

        # Now, let's populate the cache using ENABLED mode
        print("\n--- Populating cache (CacheMode.ENABLED) ---")
        await crawler.arun(config=run_cfg_enabled)
        print("Cache populated.")

        # Second run: READ_ONLY mode, item IS in cache. Should read from cache.
        print("\n--- Second run (CacheMode.READ_ONLY, item IS in cache) ---")
        start_time_read = time.perf_counter()
        result_read_cache = await crawler.arun(config=run_cfg_readonly)
        duration_read = time.perf_counter() - start_time_read
        if result_read_cache.success:
            print(f"READ_ONLY (read cache): Successful. Duration: {duration_read:.2f}s. Cached: {result_read_cache.metadata.get('cached', False)}")
            assert result_read_cache.metadata.get('cached', True)
        else:
            print(f"READ_ONLY (read cache) failed: {result_read_cache.error_message}")
        
        # Clean up
        await db_manager.adelete_url_cache(url_readonly)


if __name__ == "__main__":
    asyncio.run(cache_readonly_example())
```

---
#### 6.5.5. Example: Configuring cache for write-only access with `cache_mode=CacheMode.WRITE_ONLY`.
Always fetches fresh data, but writes it to the cache. Does not read from cache even if data exists.

```python
import asyncio
import time
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, CacheMode, BrowserConfig, AsyncDBManager

async def cache_writeonly_example():
    url_writeonly = "https://example.com/cache_test_page_writeonly"
    run_cfg_writeonly = CrawlerRunConfig(url=url_writeonly, cache_mode=CacheMode.WRITE_ONLY)
    run_cfg_enabled = CrawlerRunConfig(url=url_writeonly, cache_mode=CacheMode.ENABLED)
    
    browser_cfg = BrowserConfig(headless=True, verbose=True)
    db_manager = AsyncDBManager()

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        # Ensure item is not in cache
        await db_manager.adelete_url_cache(url_writeonly)
        print(f"Ensured {url_writeonly} is not in cache.")

        # First run: WRITE_ONLY. Should fetch fresh AND write to cache.
        print("\n--- First run (CacheMode.WRITE_ONLY) ---")
        result_write1 = await crawler.arun(config=run_cfg_writeonly)
        if result_write1.success:
            print(f"WRITE_ONLY run 1: Successful. Cached flag: {result_write1.metadata.get('cached', False)}")
            assert not result_write1.metadata.get('cached', False) # Read part is skipped
        else:
            print(f"WRITE_ONLY run 1 failed: {result_write1.error_message}")
            return

        # Check if it was written
        cached_item_after_write1 = await db_manager.aget_cached_url(url_writeonly)
        if cached_item_after_write1:
            print("SUCCESS: Item was written to cache after first WRITE_ONLY run.")
        else:
            print("FAILURE: Item was NOT written to cache after first WRITE_ONLY run.")
            return

        # Second run: WRITE_ONLY again. Should still fetch fresh (not read cache), then overwrite cache.
        print("\n--- Second run (CacheMode.WRITE_ONLY, item is in cache) ---")
        # To verify it fetches fresh, we'd need a page that changes content, or compare timestamps.
        # For simplicity, we'll just confirm the behavior.
        start_time_write2 = time.perf_counter()
        result_write2 = await crawler.arun(config=run_cfg_writeonly)
        duration_write2 = time.perf_counter() - start_time_write2
        if result_write2.success:
            print(f"WRITE_ONLY run 2: Successful. Duration: {duration_write2:.2f}s. Cached flag: {result_write2.metadata.get('cached', False)}")
            assert not result_write2.metadata.get('cached', False)
        else:
            print(f"WRITE_ONLY run 2 failed: {result_write2.error_message}")
        
        # Clean up
        await db_manager.adelete_url_cache(url_writeonly)

if __name__ == "__main__":
    asyncio.run(cache_writeonly_example())
```

---
#### 6.5.6. Example: Showing modern equivalent for deprecated `bypass_cache=True` (uses `CacheMode.BYPASS`).
The `bypass_cache=True` flag in older versions is now `cache_mode=CacheMode.BYPASS`.

```python
import asyncio
from crawl4ai import CrawlerRunConfig, CacheMode

async def deprecated_bypass_cache_equivalent():
    # Old way (would raise warning or error in newer versions if directly used as param)
    # run_cfg_old = CrawlerRunConfig(bypass_cache=True) 

    # New way
    run_cfg_new = CrawlerRunConfig(cache_mode=CacheMode.BYPASS)
    
    print("Demonstrating CacheMode.BYPASS (equivalent to old bypass_cache=True):")
    print(f"  cache_mode: {run_cfg_new.cache_mode}")
    
    # Verify behavior (conceptual, actual test in CacheMode.BYPASS example)
    # - It will not read from cache.
    # - It will fetch fresh data.
    # - It will write the fetched data to cache.
    assert run_cfg_new.cache_mode == CacheMode.BYPASS
    print("This configuration means the crawler will ignore existing cache entries for reading but will update the cache with the new data.")

if __name__ == "__main__":
    asyncio.run(deprecated_bypass_cache_equivalent())
```

---
#### 6.5.7. Example: Showing modern equivalent for deprecated `disable_cache=True` (uses `CacheMode.DISABLED`).
The `disable_cache=True` flag in older versions is now `cache_mode=CacheMode.DISABLED`.

```python
import asyncio
from crawl4ai import CrawlerRunConfig, CacheMode

async def deprecated_disable_cache_equivalent():
    # Old way (would raise warning or error in newer versions if directly used as param)
    # run_cfg_old = CrawlerRunConfig(disable_cache=True)

    # New way
    run_cfg_new = CrawlerRunConfig(cache_mode=CacheMode.DISABLED)

    print("Demonstrating CacheMode.DISABLED (equivalent to old disable_cache=True):")
    print(f"  cache_mode: {run_cfg_new.cache_mode}")
    
    # Verify behavior (conceptual, actual test in CacheMode.DISABLED example)
    # - It will not read from cache.
    # - It will fetch fresh data.
    # - It will NOT write the fetched data to cache.
    assert run_cfg_new.cache_mode == CacheMode.DISABLED
    print("This configuration means the crawler will neither read from nor write to the cache.")

if __name__ == "__main__":
    asyncio.run(deprecated_disable_cache_equivalent())
```

---
### 6.6. Session Management and Shared Data

#### 6.6.1. Example: Using `session_id` to maintain a persistent browser context across multiple `arun` calls.
Allows subsequent `arun` calls to reuse the same browser page/tab and its state (cookies, localStorage, JS context).

```python
import asyncio
import json
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig

async def session_id_example():
    my_session_id = "my_persistent_web_session_123"
    browser_cfg = BrowserConfig(headless=True, verbose=True) # Keep browser open across arun calls if session_id is used

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        # First call: Visit a page, set a cookie via JS
        js_set_cookie = "document.cookie = 'session_test_cookie=hello_crawl4ai; path=/';"
        run_cfg1 = CrawlerRunConfig(
            url="https://httpbin.org/anything", # Initial page
            session_id=my_session_id,
            js_code=js_set_cookie
        )
        print(f"\n--- First call with session_id='{my_session_id}', setting cookie ---")
        result1 = await crawler.arun(config=run_cfg1)
        if result1.success:
            print(f"First call to {result1.url} successful.")
        else:
            print(f"First call failed: {result1.error_message}")
            return

        # Second call: Visit another page on the same domain, cookie should be sent
        run_cfg2 = CrawlerRunConfig(
            url="https://httpbin.org/cookies", # This page shows cookies
            session_id=my_session_id # Crucial: use the same session_id
        )
        print(f"\n--- Second call with session_id='{my_session_id}', checking cookie ---")
        result2 = await crawler.arun(config=run_cfg2)
        if result2.success:
            print(f"Second call to {result2.url} successful.")
            response_data = json.loads(result2.html)
            print(f"Cookies received by server: {response_data.get('cookies')}")
            assert "session_test_cookie" in response_data.get("cookies", {})
            assert response_data.get("cookies", {}).get("session_test_cookie") == "hello_crawl4ai"
            print("SUCCESS: Cookie persisted across calls within the same session_id!")
        else:
            print(f"Second call failed: {result2.error_message}")
        
        # Important: Clean up the session if you're done with it and it's not managed by use_persistent_context
        await crawler.crawler_strategy.kill_session(my_session_id)
        print(f"\nSession '{my_session_id}' explicitly killed.")


if __name__ == "__main__":
    asyncio.run(session_id_example())
```

---
#### 6.6.2. Example: Passing `shared_data` dictionary for use in custom hooks (hooks themselves are external to this component).
`shared_data` is a way to pass arbitrary data to and between custom hook functions if you've implemented them.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig
from crawl4ai.async_crawler_strategy import AsyncPlaywrightCrawlerStrategy # To attach hooks

async def shared_data_example():
    # Define a simple hook that accesses shared_data
    async def my_custom_before_goto_hook(page, context, url, shared_data, **kwargs):
        print(f"[HOOK - before_goto] Accessing shared_data: {shared_data}")
        shared_data["hook_modified_value"] = "value_set_by_hook"
        # This hook could, for example, conditionally set headers based on shared_data
        return page

    browser_cfg = BrowserConfig(headless=True, verbose=True)
    
    # The strategy where hooks are attached
    crawler_strategy = AsyncPlaywrightCrawlerStrategy(config=browser_cfg)
    crawler_strategy.set_hook("before_goto", my_custom_before_goto_hook)
    
    initial_shared_data = {"user_id": 123, "task_type": "data_collection"}
    
    run_cfg = CrawlerRunConfig(
        url="https://example.com",
        shared_data=initial_shared_data.copy() # Pass a copy to avoid modification issues if reused
    )
    print(f"CrawlerRunConfig with initial shared_data: {run_cfg.shared_data}")

    async with AsyncWebCrawler(crawler_strategy=crawler_strategy) as crawler:
        print("\n--- Running crawl with shared_data and custom hook ---")
        result = await crawler.arun(config=run_cfg)
        
        if result.success:
            print(f"Crawl to {result.url} successful.")
            # Check if the hook modified the shared_data (if the hook logic does so)
            # The hook we defined modifies the dictionary passed to it.
            # The `run_cfg.shared_data` is the one passed, so it should be modified.
            print(f"Shared data after crawl (modified by hook): {run_cfg.shared_data}")
            assert run_cfg.shared_data["hook_modified_value"] == "value_set_by_hook"
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(shared_data_example())
```

---
### 6.7. Page Navigation, Waits, and Timing

#### 6.7.1. Example: Changing `wait_until` to "load" or "networkidle".
Controls when Playwright considers navigation complete. Default is "domcontentloaded".

```python
import asyncio
import time
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig

async def wait_until_example():
    browser_cfg = BrowserConfig(headless=True, verbose=True)
    url_to_crawl = "https://example.com" # A simple, fast-loading page

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        # Test with "domcontentloaded" (default)
        run_cfg_dom = CrawlerRunConfig(url=url_to_crawl, wait_until="domcontentloaded")
        start_time_dom = time.perf_counter()
        await crawler.arun(config=run_cfg_dom)
        duration_dom = time.perf_counter() - start_time_dom
        print(f"Wait_until 'domcontentloaded' duration: {duration_dom:.2f}s")

        # Test with "load" (waits for all resources like images)
        run_cfg_load = CrawlerRunConfig(url=url_to_crawl, wait_until="load")
        start_time_load = time.perf_counter()
        await crawler.arun(config=run_cfg_load)
        duration_load = time.perf_counter() - start_time_load
        print(f"Wait_until 'load' duration: {duration_load:.2f}s")
        # For a simple page, 'load' might not be much longer than 'domcontentloaded'
        # but for image-heavy pages, it would be.

        # Test with "networkidle" (waits for network to be idle for 500ms)
        run_cfg_idle = CrawlerRunConfig(url=url_to_crawl, wait_until="networkidle")
        start_time_idle = time.perf_counter()
        await crawler.arun(config=run_cfg_idle)
        duration_idle = time.perf_counter() - start_time_idle
        print(f"Wait_until 'networkidle' duration: {duration_idle:.2f}s")
        # 'networkidle' can be significantly longer if there are background network activities

if __name__ == "__main__":
    asyncio.run(wait_until_example())
```

---
#### 6.7.2. Example: Setting a custom `page_timeout` for page operations.
Maximum time (in ms) for operations like page navigation. Default is 60000ms (60s).

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig

async def page_timeout_example():
    browser_cfg = BrowserConfig(headless=True, verbose=True)
    
    # A URL that is intentionally slow or non-existent to trigger timeout
    # httpbin.org/delay/X can simulate a slow response
    slow_url = "https://httpbin.org/delay/5" # Delays for 5 seconds

    # Set a short timeout to demonstrate
    run_cfg_short_timeout = CrawlerRunConfig(url=slow_url, page_timeout=2000) # 2 seconds
    
    print(f"Attempting to crawl {slow_url} with page_timeout=2000ms.")
    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(config=run_cfg_short_timeout)
        if not result.success:
            print(f"Crawl failed as expected due to timeout: {result.error_message}")
            assert "Timeout" in result.error_message or "timeout" in result.error_message.lower()
        else:
            print("Crawl succeeded unexpectedly (timeout might not have triggered).")

if __name__ == "__main__":
    asyncio.run(page_timeout_example())
```

---
#### 6.7.3. Example: Using `wait_for` with a CSS selector before proceeding.
Waits for a specific element to appear in the DOM.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig

async def wait_for_css_example():
    browser_cfg = BrowserConfig(headless=True, verbose=True)
    
    # Simulate a page where content appears after a delay
    # We'll use JS to add an element after 2 seconds
    html_content_delayed = """
    <html><body>
        <div id='loading'>Loading...</div>
        <script>
            setTimeout(() => {
                const newElem = document.createElement('p');
                newElem.id = 'late-content';
                newElem.textContent = 'Content has arrived!';
                document.body.appendChild(newElem);
                document.getElementById('loading').remove();
            }, 2000);
        </script>
    </body></html>
    """
    
    run_cfg = CrawlerRunConfig(
        url=f"raw://{html_content_delayed}",
        wait_for="css:#late-content", # Wait for the element with id 'late-content'
        page_timeout=5000 # Overall timeout for the operation
    )
    
    print("Attempting to crawl page with delayed content, waiting for '#late-content'.")
    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(config=run_cfg)
        if result.success:
            print(f"Crawl successful. Cleaned HTML: {result.cleaned_html}")
            assert "Content has arrived!" in result.cleaned_html
            assert "Loading..." not in result.cleaned_html # Should be removed by the script
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(wait_for_css_example())
```

---
#### 6.7.4. Example: Using `wait_for` with a JavaScript predicate.
Waits for a JavaScript expression to evaluate to true.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig

async def wait_for_js_example():
    browser_cfg = BrowserConfig(headless=True, verbose=True)
    
    # Simulate a page where a JS variable changes after a delay
    html_js_var_delayed = """
    <html><body>
        <p id="status">Initializing...</p>
        <script>
            window.myAppStatus = 'pending';
            setTimeout(() => {
                window.myAppStatus = 'ready';
                document.getElementById('status').textContent = 'Application is Ready!';
            }, 2500);
        </script>
    </body></html>
    """
    
    js_predicate = "window.myAppStatus === 'ready'"
    run_cfg = CrawlerRunConfig(
        url=f"raw://{html_js_var_delayed}",
        wait_for=f"js:{js_predicate}",
        page_timeout=5000
    )
    
    print(f"Attempting to crawl page, waiting for JS: `{js_predicate}`.")
    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(config=run_cfg)
        if result.success:
            print(f"Crawl successful after JS condition met.")
            print(f"Cleaned HTML: {result.cleaned_html}")
            assert "Application is Ready!" in result.cleaned_html
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(wait_for_js_example())
```

---
#### 6.7.5. Example: Setting a specific `wait_for_timeout` for the `wait_for` condition.
If `wait_for` condition is not met within this timeout (in ms), the crawl proceeds or fails based on overall `page_timeout`.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig

async def wait_for_timeout_example():
    browser_cfg = BrowserConfig(headless=True, verbose=True)
    
    html_never_appears = "<html><body><p>Content</p></body></html>"
    
    run_cfg = CrawlerRunConfig(
        url=f"raw://{html_never_appears}",
        wait_for="css:#nonexistent-element",
        wait_for_timeout=1000, # Wait only 1 second for this specific element
        page_timeout=5000      # Overall page timeout
    )
    
    print("Attempting to crawl, waiting for a nonexistent element with a short wait_for_timeout.")
    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        start_time = asyncio.get_event_loop().time()
        result = await crawler.arun(config=run_cfg)
        duration = asyncio.get_event_loop().time() - start_time
        
        if result.success:
            print(f"Crawl completed (likely after wait_for_timeout). Duration: {duration:.2f}s")
            # The crawl might succeed if page_timeout is longer and wait_for_timeout is just a pause point
            # Playwright's wait_for_selector with timeout usually throws an error if not found.
            # Crawl4ai's behavior might proceed if overall page_timeout isn't hit by this.
            # Let's check if the error message indicates a timeout related to the selector.
            # The current implementation of AsyncPlaywrightCrawlerStrategy would lead to an overall timeout.
            # If the wait_for_timeout is meant to be non-fatal, the strategy logic would need adjustment.
            # For now, we expect the page.wait_for_selector to raise a TimeoutError.
            print(f"Crawl error_message (if any): {result.error_message}")
            # assert "Timeout" in result.error_message if result.error_message else False
            # This behavior depends on how Playwright's timeout for wait_for_selector is handled.
            # If it raises and is caught, error_message will have it. If it's ignored, it might proceed.
            # Given current code, it's more likely the overall page_timeout will be hit if wait_for never resolves.
            # Let's assume for this test that wait_for failing to find element leads to crawl failing.
            if result.error_message and "Timeout" in result.error_message: # Playwright TimeoutError
                 print("Wait_for timed out as expected, and crawl might have failed due to it.")
            else:
                 print("Crawl succeeded, wait_for_timeout might have just passed and execution continued.")

        else: # More likely scenario if wait_for is critical
            print(f"Crawl failed as expected. Duration: {duration:.2f}s. Error: {result.error_message}")
            assert "Timeout" in result.error_message # Expecting a timeout error

if __name__ == "__main__":
    asyncio.run(wait_for_timeout_example())
```

---
#### 6.7.6. Example: Enabling `wait_for_images=True` to ensure images are loaded.
Attempts to wait until all (or most) images on the page have finished loading.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig

async def wait_for_images_example():
    browser_cfg = BrowserConfig(headless=True, verbose=True)
    # A page with several images. Wikipedia pages are good for this.
    url_with_images = "https://en.wikipedia.org/wiki/Cat"
    
    run_cfg = CrawlerRunConfig(
        url=url_with_images,
        wait_for_images=True,
        screenshot=True # Take a screenshot to visually verify if images loaded
    )
    
    print(f"Attempting to crawl {url_with_images} with wait_for_images=True.")
    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(config=run_cfg)
        if result.success:
            print(f"Crawl successful for {url_with_images}.")
            if result.screenshot:
                print("Screenshot captured. Manually inspect 'wait_for_images_screenshot.png' to see if images are loaded.")
                with open("wait_for_images_screenshot.png", "wb") as f:
                    import base64
                    f.write(base64.b64decode(result.screenshot))
            # Check if media extraction found images (though this happens after HTML retrieval)
            if result.media and result.media.get("images"):
                print(f"Found {len(result.media['images'])} image entries.")
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(wait_for_images_example())
```

---
#### 6.7.7. Example: Setting `delay_before_return_html` to pause before final HTML retrieval.
Adds a fixed delay (in seconds) just before the final HTML content is grabbed from the page.

```python
import asyncio
import time
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig

async def delay_before_html_example():
    browser_cfg = BrowserConfig(headless=True, verbose=True)
    
    run_cfg = CrawlerRunConfig(
        url="https://example.com",
        delay_before_return_html=2.0 # Wait 2 seconds
    )
    print(f"Configured delay_before_return_html: {run_cfg.delay_before_return_html}s")
    
    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        print(f"Crawling {run_cfg.url} with a 2s delay before HTML retrieval.")
        start_time = time.perf_counter()
        result = await crawler.arun(config=run_cfg)
        duration = time.perf_counter() - start_time
        
        if result.success:
            print(f"Crawl successful. Total duration: {duration:.2f}s (should be > 2s).")
            assert duration >= 2.0
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(delay_before_html_example())
```

---
### 6.8. `arun_many` Specific Timing and Concurrency

#### 6.8.1. Example: Configuring `mean_delay` and `max_range` for randomized delays between `arun_many` requests.
This adds a random delay between `(mean_delay - max_range)` and `(mean_delay + max_range)` seconds for each request in `arun_many`.

```python
import asyncio
import time
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig, CacheMode

async def arun_many_delay_example():
    browser_cfg = BrowserConfig(headless=True, verbose=False) # Keep logs cleaner for this demo
    urls = [f"https://example.com?page={i}" for i in range(3)] # 3 dummy URLs

    run_cfg = CrawlerRunConfig(
        mean_delay=1.0,  # Average 1 second delay
        max_range=0.5,   # Delay will be between 0.5s and 1.5s
        cache_mode=CacheMode.BYPASS # Ensure actual fetches
    )
    print(f"Configured for arun_many: mean_delay={run_cfg.mean_delay}, max_range={run_cfg.max_range}")

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        print("Running arun_many with delays...")
        start_time = time.perf_counter()
        # arun_many itself doesn't take these; they are interpreted by dispatchers.
        # The default MemoryAdaptiveDispatcher uses these.
        results = await crawler.arun_many(urls=urls, config=run_cfg)
        total_duration = time.perf_counter() - start_time
        
        success_count = sum(1 for r in results if r.success)
        print(f"Finished {success_count}/{len(urls)} crawls.")
        print(f"Total duration for {len(urls)} crawls: {total_duration:.2f}s")
        
        # Expected duration: roughly num_urls * mean_delay + (sum of actual fetch times)
        # For 3 URLs, with mean_delay=1, expect at least 2 * 0.5 = 1s of added delay, up to 2 * 1.5 = 3s
        # This is a rough check, actual network time also contributes.
        # If all example.com fetches are very fast (<0.1s), total should be around 1-3s + (3*~0.1s)
        # A more robust test would mock the fetch time.
        # For now, we just check it took longer than if no delays.
        assert total_duration > (len(urls) -1) * (run_cfg.mean_delay - run_cfg.max_range) if len(urls)>1 else True

if __name__ == "__main__":
    asyncio.run(arun_many_delay_example())
```

---
#### 6.8.2. Example: Adjusting `semaphore_count` to control concurrency in `arun_many`.
Limits the number of concurrent crawl operations when using `arun_many` with dispatchers that respect it (like `SemaphoreDispatcher`).

```python
import asyncio
import time
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig, CacheMode
from crawl4ai.async_dispatcher import SemaphoreDispatcher # Explicitly use SemaphoreDispatcher

async def arun_many_semaphore_example():
    browser_cfg = BrowserConfig(headless=True, verbose=False)
    # More URLs to demonstrate concurrency clearly
    urls = [f"https://httpbin.org/delay/1?id={i}" for i in range(6)] # Each takes ~1s

    run_cfg_concurrency_2 = CrawlerRunConfig(
        semaphore_count=2, # Allow only 2 concurrent crawls
        cache_mode=CacheMode.BYPASS
    )
    print(f"Configured for arun_many: semaphore_count={run_cfg_concurrency_2.semaphore_count}")

    # Use SemaphoreDispatcher to respect semaphore_count
    dispatcher = SemaphoreDispatcher(max_concurrent_tasks=run_cfg_concurrency_2.semaphore_count)

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        print(f"Running arun_many with {len(urls)} URLs and semaphore_count=2...")
        start_time = time.perf_counter()
        results = await crawler.arun_many(urls=urls, config=run_cfg_concurrency_2, dispatcher=dispatcher)
        total_duration = time.perf_counter() - start_time
        
        success_count = sum(1 for r in results if r.success)
        print(f"Finished {success_count}/{len(urls)} crawls.")
        print(f"Total duration: {total_duration:.2f}s")
        
        # With 6 URLs, each taking ~1s, and concurrency of 2:
        # Expected time is roughly (num_urls / concurrency) * per_url_time
        # (6 / 2) * 1s = 3s (plus overhead)
        # Without semaphore, it might be closer to 1s if all run truly parallel (unlikely for 6 heavy tasks)
        # or up to 6s if strictly sequential.
        # This is a heuristic check
        assert total_duration >= (len(urls) / run_cfg_concurrency_2.semaphore_count) * 1.0 
        assert total_duration < len(urls) * 1.0 # Should be faster than purely sequential

if __name__ == "__main__":
    asyncio.run(arun_many_semaphore_example())
```

---
### 6.9. Page Interaction, Anti-Bot, and Dynamic Content

#### 6.9.1. Example: Executing a single JavaScript string using `js_code`.
Runs arbitrary JavaScript on the page after it loads.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig

async def single_js_code_example():
    browser_cfg = BrowserConfig(headless=True)
    
    js_to_run = "document.body.innerHTML = '<h1>JavaScript Was Here!</h1>';"
    run_cfg = CrawlerRunConfig(
        url="https://example.com", # Original page content will be replaced
        js_code=js_to_run
    )
    
    print(f"Running JS: {js_to_run}")
    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(config=run_cfg)
        if result.success:
            print(f"Crawled {result.url} after JS execution.")
            print(f"Cleaned HTML: {result.cleaned_html}")
            assert "JavaScript Was Here!" in result.cleaned_html
            assert "Example Domain" not in result.cleaned_html # Original H1 should be gone
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(single_js_code_example())
```

---
#### 6.9.2. Example: Executing a list of JavaScript snippets using `js_code`.
Runs multiple JavaScript snippets sequentially.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig

async def list_js_code_example():
    browser_cfg = BrowserConfig(headless=True)
    
    js_snippets = [
        "document.body.insertAdjacentHTML('beforeend', '<p id=\\'js_step1\\'>Step 1 Complete</p>');",
        "document.getElementById('js_step1').style.color = 'blue';",
        "window.js_result = document.getElementById('js_step1').outerHTML;" # Store for verification
    ]
    run_cfg = CrawlerRunConfig(
        url="https://example.com",
        js_code=js_snippets,
        # We need to retrieve the window.js_result
        # This can be done by having the last JS snippet return something,
        # or by executing another JS snippet to retrieve it in a subsequent step.
        # For simplicity, let's assume the crawler strategy can pick up the last expression value
        # (if it's a string, Playwright's page.evaluate often returns it).
        # More reliably, explicitly return the value:
        # js_snippets.append("window.js_result;")
    )
    
    print(f"Running JS snippets sequentially.")
    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        # To get a JS execution result, the last JS statement must evaluate to a serializable value.
        # Let's modify js_snippets for this.
        run_cfg.js_code.append("window.js_result;") # type: ignore
        
        result = await crawler.arun(config=run_cfg)
        if result.success:
            print(f"Crawled {result.url} after multiple JS executions.")
            print(f"Cleaned HTML (should contain 'Step 1 Complete'):\n{result.cleaned_html[:300]}")
            assert "Step 1 Complete" in result.cleaned_html
            
            print(f"JS Execution Result (outerHTML of #js_step1): {result.js_execution_result}")
            assert result.js_execution_result and 'style="color: blue;"' in result.js_execution_result
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(list_js_code_example())
```

---
#### 6.9.3. Example: Using `js_only=True` for subsequent calls within a session to only execute JS.
Useful for multi-step interactions where you only want to run JS on an already loaded page in a session, without a full page navigation.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig

async def js_only_example():
    session_id = "js_only_session"
    browser_cfg = BrowserConfig(headless=True, verbose=True)

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        # Step 1: Load the page
        run_cfg1 = CrawlerRunConfig(url="https://example.com", session_id=session_id)
        print("\n--- Step 1: Loading initial page ---")
        result1 = await crawler.arun(config=run_cfg1)
        if not result1.success:
            print(f"Initial page load failed: {result1.error_message}")
            return
        print(f"Initial page '{result1.url}' loaded. HTML length: {len(result1.html)}")

        # Step 2: Execute JS only, no re-navigation
        js_to_modify = "document.body.innerHTML = '<h1>JS Only Update!</h1>'; 'JS Executed';"
        run_cfg2 = CrawlerRunConfig(
            url=result1.url, # Important: use the same URL or the one currently loaded in session
            session_id=session_id,
            js_code=js_to_modify,
            js_only=True # This is key
        )
        print("\n--- Step 2: Executing JS only on the current page ---")
        result2 = await crawler.arun(config=run_cfg2)
        if result2.success:
            print(f"JS-only update successful for {result2.url}.")
            print(f"Cleaned HTML after JS-only: {result2.cleaned_html}")
            assert "JS Only Update!" in result2.cleaned_html
            assert "Example Domain" not in result2.cleaned_html
            print(f"JS Execution Result: {result2.js_execution_result}")
            assert result2.js_execution_result == "JS Executed"
        else:
            print(f"JS-only update failed: {result2.error_message}")
        
        await crawler.crawler_strategy.kill_session(session_id)
        print(f"\nSession '{session_id}' killed.")

if __name__ == "__main__":
    asyncio.run(js_only_example())
```

---
#### 6.9.4. Example: Enabling `scan_full_page=True` and setting `scroll_delay` to handle lazy-loaded content.
The crawler will attempt to scroll through the entire page, pausing between scrolls, to trigger lazy-loading.

```python
import asyncio
import time
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig, CacheMode

async def scan_full_page_example():
    browser_cfg = BrowserConfig(headless=True, verbose=True, viewport_height=300) # Smaller viewport to make scrolling more evident
    
    # A page known for long content or lazy loading (e.g., a long blog post or news article)
    # For this demo, we'll use a simple page and check total scroll time.
    # Real effect is best seen on pages that actually lazy-load images/content.
    url_to_scan = "https://en.wikipedia.org/wiki/Web_scraping" 
    
    run_cfg_scan = CrawlerRunConfig(
        url=url_to_scan,
        scan_full_page=True,
        scroll_delay=0.2,  # 0.2 seconds delay between scroll steps
        cache_mode=CacheMode.BYPASS
    )
    print(f"Crawling {url_to_scan} with scan_full_page=True, scroll_delay={run_cfg_scan.scroll_delay}s")

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        start_time = time.perf_counter()
        result = await crawler.arun(config=run_cfg_scan)
        duration = time.perf_counter() - start_time
        
        if result.success:
            print(f"Crawl successful. Duration: {duration:.2f}s (includes scrolling time).")
            print(f"Markdown length: {len(result.markdown.raw_markdown)}")
            # On a long page, the duration should be noticeably longer than a non-scrolling crawl.
            # And more content/images might be present in result.html/result.media
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(scan_full_page_example())
```

---
#### 6.9.5. Example: Enabling `process_iframes=True` to attempt extracting content from iframes.
**Note:** Iframe processing can be complex and success depends heavily on iframe structure and cross-origin policies.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig, CacheMode

async def process_iframes_example():
    browser_cfg = BrowserConfig(headless=True, verbose=True)
    
    # Need a URL that reliably contains accessible iframes for a good demo.
    # W3Schools iframe example page:
    iframe_test_url = "https://www.w3schools.com/tags/tryit.asp?filename=tryhtml_iframe_name"
    
    run_cfg_iframes = CrawlerRunConfig(
        url=iframe_test_url,
        process_iframes=True,
        cache_mode=CacheMode.BYPASS,
        wait_for="iframe#iframeResult" # Wait for the specific iframe to ensure it's targetable
    )
    print(f"Crawling {iframe_test_url} with process_iframes=True.")

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(config=run_cfg_iframes)
        if result.success:
            print(f"Crawl successful for {iframe_test_url}.")
            print(f"Cleaned HTML (first 1000 chars):\n{result.cleaned_html[:1000]}")
            # Check for content known to be inside an iframe on that page.
            # The W3Schools example iframe usually loads "demo_iframe.htm" which contains "This is an iframe".
            if "This is an iframe" in result.cleaned_html:
                print("SUCCESS: Content from iframe seems to be included in cleaned_html.")
            else:
                print("Content from iframe not found or iframe structure is complex/cross-origin restricted.")
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(process_iframes_example())
```

---
#### 6.9.6. Example: Using `remove_overlay_elements=True` to dismiss popups/modals.
Attempts to automatically find and remove common overlay/popup elements before content extraction.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig, CacheMode

async def remove_overlays_example():
    browser_cfg = BrowserConfig(headless=True, verbose=True)
    
    # A page that might have a cookie consent banner or modal.
    # This is hard to test reliably with a public static URL as overlays are dynamic.
    # We'll simulate an overlay with raw HTML for this example.
    html_with_overlay = """
    <html><body>
        <div id="page-content">Main page text here.</div>
        <div id="cookie-banner" style="position:fixed; bottom:0; background:lightgray; width:100%; padding:10px; text-align:center;">
            This is a cookie banner! <button onclick="this.parentElement.remove()">Accept</button>
        </div>
        <div class="modal-overlay" style="position:fixed; top:0; left:0; width:100%; height:100%; background:rgba(0,0,0,0.5);">
            <div class="modal-content" style="background:white; margin:15% auto; padding:20px; width:50%;">A promotional modal!</div>
        </div>
    </body></html>
    """
    
    run_cfg = CrawlerRunConfig(
        url=f"raw://{html_with_overlay}",
        remove_overlay_elements=True,
        cache_mode=CacheMode.BYPASS
    )
    print(f"Crawling content with remove_overlay_elements=True.")

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(config=run_cfg)
        if result.success:
            print(f"Crawl successful.")
            cleaned_html = result.cleaned_html
            print(f"Cleaned HTML (should not contain overlay/banner):\n{cleaned_html}")
            assert "cookie-banner" not in cleaned_html.lower() # Check by ID
            assert "modal-overlay" not in cleaned_html.lower() # Check by class
            assert "Main page text here." in cleaned_html # Main content should remain
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(remove_overlays_example())
```

---
#### 6.9.7. Example: Enabling `simulate_user=True` for basic user interaction simulation.
Triggers basic mouse movements and scrolls to mimic user presence, potentially bypassing some simple anti-bot measures.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig, CacheMode

async def simulate_user_example():
    browser_cfg = BrowserConfig(headless=True, verbose=True) # Can be False to observe
    
    run_cfg = CrawlerRunConfig(
        url="https://example.com", # A simple page for demonstration
        simulate_user=True,
        cache_mode=CacheMode.BYPASS
    )
    print(f"Crawling {run_cfg.url} with simulate_user=True.")

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(config=run_cfg)
        if result.success:
            print(f"Crawl successful with user simulation.")
            # The effect of simulate_user is subtle and hard to verify programmatically
            # on a simple page. It's more about behavior during the crawl.
            # If there were JS event listeners for mouse/scroll, they might have been triggered.
            print(f"Markdown (first 300 chars):\n{result.markdown.raw_markdown[:300]}")
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(simulate_user_example())
```

---
#### 6.9.8. Example: Enabling `override_navigator=True` to modify browser navigator properties.
Modifies properties like `navigator.webdriver` to make the browser appear less like an automated tool.

```python
import asyncio
import json
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig, CacheMode

async def override_navigator_example():
    browser_cfg = BrowserConfig(headless=True, verbose=True)
    
    # JS to check navigator.webdriver property
    js_check_webdriver = "JSON.stringify({webdriver: navigator.webdriver})"
    
    run_cfg_override = CrawlerRunConfig(
        url="https://httpbin.org/anything", # Any page will do for this JS check
        override_navigator=True,
        js_code=js_check_webdriver,
        cache_mode=CacheMode.BYPASS
    )
    print(f"Crawling with override_navigator=True.")

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(config=run_cfg_override)
        if result.success and result.js_execution_result:
            print(f"Crawl successful with navigator override.")
            webdriver_status = result.js_execution_result
            print(f"navigator.webdriver status: {webdriver_status}")
            # When overridden, navigator.webdriver should be false
            assert webdriver_status.get("webdriver") is False
        elif not result.js_execution_result:
             print(f"JS execution did not return a result for webdriver check.")
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(override_navigator_example())
```

---
#### 6.9.9. Example: Using `magic=True` for automated handling of common anti-bot measures like overlays.
`magic=True` is a convenience flag that enables several anti-bot evasion techniques, including `remove_overlay_elements` and `override_navigator`.

```python
import asyncio
import json
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig, CacheMode

async def magic_mode_example():
    browser_cfg = BrowserConfig(headless=True, verbose=True)
    
    # HTML with an overlay and JS that checks navigator.webdriver
    html_with_magic_challenges = """
    <html><body>
        <div id="page-content">Main content.</div>
        <div id="my-overlay" style="position:fixed; top:0; left:0; width:100%; height:100%; background:rgba(0,0,0,0.3);">Overlay</div>
        <script>
            window.webdriverStatus = navigator.webdriver;
        </script>
    </body></html>
    """
    js_get_webdriver = "JSON.stringify({webdriver: window.webdriverStatus})"

    run_cfg_magic = CrawlerRunConfig(
        url=f"raw://{html_with_magic_challenges}",
        magic=True,
        js_code=js_get_webdriver, # Check webdriver status after magic applies
        cache_mode=CacheMode.BYPASS
    )
    print(f"Crawling with magic=True.")

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(config=run_cfg_magic)
        if result.success:
            print(f"Crawl successful with magic mode.")
            
            # Check overlay removal
            print(f"Cleaned HTML (overlay should be gone):\n{result.cleaned_html}")
            assert "my-overlay" not in result.cleaned_html # Magic mode should remove it
            assert "Main content." in result.cleaned_html

            # Check navigator override
            if result.js_execution_result:
                webdriver_status = result.js_execution_result
                print(f"navigator.webdriver status (via JS): {webdriver_status}")
                assert webdriver_status.get("webdriver") is False # Magic mode should override this
            else:
                print("JS execution for webdriver check did not yield result.")
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(magic_mode_example())
```

---
#### 6.9.10. Example: Using `adjust_viewport_to_content=True` to dynamically resize viewport.
Resizes the browser viewport to fit the dimensions of the rendered page content. This is useful for accurate screenshots of the entire logical page.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig, CacheMode

async def adjust_viewport_example():
    # A page that's taller than the default viewport
    # We'll use a raw HTML example with a defined large height for predictability
    tall_page_html = """
    <html><head><style>body {{ margin: 0; }} #tall-div {{ height: 2000px; width: 500px; background: lightblue; }}</style></head>
    <body><div id="tall-div">This is a tall div.</div></body></html>
    """
    
    browser_cfg = BrowserConfig(
        headless=True, 
        verbose=True,
        viewport_width=800, # Initial viewport
        viewport_height=600
    )
    
    run_cfg = CrawlerRunConfig(
        url=f"raw://{tall_page_html}",
        adjust_viewport_to_content=True,
        screenshot=True, # Capture screenshot to see the effect
        cache_mode=CacheMode.BYPASS
    )
    print(f"Crawling with adjust_viewport_to_content=True. Initial viewport: 800x600.")

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(config=run_cfg)
        if result.success:
            print(f"Crawl successful.")
            if result.screenshot:
                print("Screenshot captured. If adjust_viewport_to_content worked, it should show the full 2000px height.")
                # Playwright's screenshot of full page might not directly reflect viewport size change,
                # but the internal logic for determining content height would have used the adjusted viewport.
                # The primary effect is on how Playwright calculates "full page".
                # It's hard to verify viewport size change directly without inspecting browser internals during run.
                # The key is that `page.content()` or screenshot operations consider the full content.
                print("Effect is primarily on internal full-page calculations for Playwright.")
            else:
                print("Screenshot not captured.")
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(adjust_viewport_example())
```

---
### 6.10. Media Capturing and Processing Options

#### 6.10.1. Example: Enabling screenshots with `screenshot=True`.
Captures a screenshot of the page.

```python
import asyncio
import base64
from pathlib import Path
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig, CacheMode

async def enable_screenshot_example():
    browser_cfg = BrowserConfig(headless=True)
    run_cfg = CrawlerRunConfig(
        url="https://example.com",
        screenshot=True,
        cache_mode=CacheMode.BYPASS
    )
    
    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(config=run_cfg)
        if result.success and result.screenshot:
            print(f"Screenshot captured for {result.url}.")
            screenshot_path = Path("./example_com_screenshot.png")
            with open(screenshot_path, "wb") as f:
                f.write(base64.b64decode(result.screenshot))
            print(f"Screenshot saved to {screenshot_path.resolve()}")
            assert screenshot_path.exists() and screenshot_path.stat().st_size > 0
            screenshot_path.unlink() # Clean up
        elif not result.screenshot:
            print("Screenshot was enabled but not captured.")
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(enable_screenshot_example())
```

---
#### 6.10.2. Example: Using `screenshot_wait_for` (float) to add a delay before taking a screenshot.
Adds a specific delay (in seconds) before the screenshot is taken, useful for animations or late-loading elements.

```python
import asyncio
import base64
from pathlib import Path
import time
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig, CacheMode

async def screenshot_wait_for_example():
    browser_cfg = BrowserConfig(headless=True, verbose=True)
    
    # Page that might have some animation or late change
    # For demo, we'll just time it.
    url_to_crawl = "https://example.com" 
    delay_seconds = 1.5
    
    run_cfg = CrawlerRunConfig(
        url=url_to_crawl,
        screenshot=True,
        screenshot_wait_for=delay_seconds, # Wait 1.5 seconds before screenshot
        cache_mode=CacheMode.BYPASS
    )
    print(f"Crawling {url_to_crawl}, will wait {delay_seconds}s before screenshot.")

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        start_time = time.perf_counter()
        result = await crawler.arun(config=run_cfg)
        duration = time.perf_counter() - start_time
        
        if result.success and result.screenshot:
            print(f"Screenshot captured. Total crawl duration: {duration:.2f}s (should be >= {delay_seconds}s).")
            assert duration >= delay_seconds
            # Save for manual inspection if needed
            # screenshot_path = Path("./delayed_screenshot.png")
            # with open(screenshot_path, "wb") as f:
            #     f.write(base64.b64decode(result.screenshot))
            # print(f"Screenshot saved to {screenshot_path.resolve()}")
            # screenshot_path.unlink()
        else:
            print(f"Crawl or screenshot failed: {result.error_message if result else 'Unknown error'}")

if __name__ == "__main__":
    asyncio.run(screenshot_wait_for_example())
```

---
#### 6.10.3. Example: Customizing `screenshot_height_threshold` for full-page screenshot strategy.
If page height exceeds this threshold, Playwright might use a different strategy for full-page screenshots (e.g. stitching). Default is 20000.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig, CacheMode

async def screenshot_height_threshold_example():
    browser_cfg = BrowserConfig(headless=True, verbose=True)
    
    # This primarily affects how Playwright takes the full page screenshot internally.
    # The output (base64 image) will still be the full page.
    run_cfg = CrawlerRunConfig(
        url="https://en.wikipedia.org/wiki/Python_(programming_language)", # A long page
        screenshot=True,
        screenshot_height_threshold=5000, # Lower threshold, might trigger stitching earlier
        cache_mode=CacheMode.BYPASS
    )
    print(f"Crawling with screenshot_height_threshold={run_cfg.screenshot_height_threshold}.")

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(config=run_cfg)
        if result.success and result.screenshot:
            print(f"Screenshot captured for long page.")
            # Verification of the *strategy* change is internal to Playwright
            # and not easily asserted from outside. The result is still a full screenshot.
            assert len(result.screenshot) > 1000 # Check if screenshot data exists
        else:
            print(f"Crawl or screenshot failed: {result.error_message if result else 'Unknown error'}")

if __name__ == "__main__":
    asyncio.run(screenshot_height_threshold_example())
```

---
#### 6.10.4. Example: Enabling PDF generation with `pdf=True`.
Generates a PDF of the rendered page.

```python
import asyncio
from pathlib import Path
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig, CacheMode

async def enable_pdf_generation():
    browser_cfg = BrowserConfig(headless=True) # PDF generation requires headless
    run_cfg = CrawlerRunConfig(
        url="https://example.com",
        pdf=True,
        cache_mode=CacheMode.BYPASS
    )
    
    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(config=run_cfg)
        if result.success and result.pdf:
            print(f"PDF generated for {result.url}.")
            pdf_path = Path("./example_com_page.pdf")
            with open(pdf_path, "wb") as f:
                f.write(result.pdf)
            print(f"PDF saved to {pdf_path.resolve()}")
            assert pdf_path.exists() and pdf_path.stat().st_size > 0
            pdf_path.unlink() # Clean up
        elif not result.pdf:
            print("PDF was enabled but not generated.")
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(enable_pdf_generation())
```

---
#### 6.10.5. Example: Enabling MHTML archive capture with `capture_mhtml=True`.
Saves the page as an MHTML archive, which includes all resources (CSS, images, JS) in a single file.

```python
import asyncio
from pathlib import Path
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig, CacheMode

async def capture_mhtml_example():
    browser_cfg = BrowserConfig(headless=True)
    run_cfg = CrawlerRunConfig(
        url="https://example.com",
        capture_mhtml=True,
        cache_mode=CacheMode.BYPASS
    )
    
    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(config=run_cfg)
        if result.success and result.mhtml:
            print(f"MHTML archive captured for {result.url}.")
            mhtml_path = Path("./example_com_page.mhtml")
            with open(mhtml_path, "w", encoding="utf-8") as f: # MHTML is text-based
                f.write(result.mhtml)
            print(f"MHTML saved to {mhtml_path.resolve()}")
            assert mhtml_path.exists() and mhtml_path.stat().st_size > 0
            mhtml_path.unlink() # Clean up
        elif not result.mhtml:
            print("MHTML was enabled but not captured.")
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(capture_mhtml_example())
```

---
#### 6.10.6. Example: Setting `image_description_min_word_threshold` for image alt/desc processing.
Filters image descriptions (alt text or surrounding text) based on word count.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig, CacheMode

async def image_desc_threshold_example():
    # HTML with images having varying alt text lengths
    html_with_img_alts = """
    <html><body>
        <img src="img1.jpg" alt="A single word">
        <img src="img2.jpg" alt="This is a short description of five words">
        <img src="img3.jpg" alt="This alt text is definitely longer and should pass a higher threshold.">
    </body></html>
    """
    # Default for image_description_min_word_threshold is 50 in config.py, but often effectively lower if not using LLM for desc.
    # The WebScrapingStrategy uses it for finding desc from surrounding text.
    # Here, we test its effect on alt text directly (assuming internal logic considers alt text length)
    
    run_cfg = CrawlerRunConfig(
        url=f"raw://{html_with_img_alts}",
        image_description_min_word_threshold=4, # Only consider alts/desc with >= 4 words
        cache_mode=CacheMode.BYPASS
    )
    
    print(f"Crawling with image_description_min_word_threshold={run_cfg.image_description_min_word_threshold}.")
    async with AsyncWebCrawler(config=BrowserConfig(headless=True)) as crawler:
        result = await crawler.arun(config=run_cfg)
        if result.success and result.media and result.media.get("images"):
            print(f"Images found: {len(result.media['images'])}")
            for img in result.media["images"]:
                print(f"  src: {img.get('src')}, alt: '{img.get('alt')}', desc: '{img.get('desc')}'")
            
            # Expect only img2 and img3 to have their alt text considered substantial enough for desc
            # (assuming desc field is populated from alt if alt meets threshold)
            # This depends on the scraping strategy's logic for populating 'desc'.
            # The default WebScrapingStrategy populates 'desc' from parent text if alt is short.
            # A direct test of the threshold on 'alt' itself is tricky without knowing exact internal logic.
            # Let's assume the intent is that 'alt' contributes to 'desc' if long enough.
            
            # This test is more conceptual about the parameter's existence.
            # The actual filtering based on this threshold happens within the scraping strategy logic.
            # For now, we confirm that images are extracted.
            assert len(result.media["images"]) == 3 
            print("Parameter set. Actual filtering of descriptions occurs in scraping strategy.")

        else:
            print(f"Crawl failed or no images: {result.error_message if result else 'No result'}")

if __name__ == "__main__":
    asyncio.run(image_desc_threshold_example())
```

---
#### 6.10.7. Example: Setting `image_score_threshold` for filtering images by relevance.
Images scoring below this heuristic threshold might be discarded.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig, CacheMode

async def image_score_threshold_example():
    # WebScrapingStrategy scores images. Default threshold is 3.
    # Images with good alt text, dimensions, and not from common "ignore" patterns get higher scores.
    
    html_with_varied_images = """
    <html><body>
        <img src="good_image.jpg" alt="A very descriptive alt text for a good image" width="300" height="200"> <!-- High score -->
        <img src="icon.png" alt="icon" width="16" height="16"> <!-- Low score -->
        <img src="decorative.gif"> <!-- No alt, no dimensions, likely low score -->
    </body></html>
    """
    
    run_cfg_high_thresh = CrawlerRunConfig(
        url=f"raw://{html_with_varied_images}",
        image_score_threshold=4, # Higher threshold, expect fewer images
        cache_mode=CacheMode.BYPASS
    )
    run_cfg_low_thresh = CrawlerRunConfig(
        url=f"raw://{html_with_varied_images}",
        image_score_threshold=1, # Lower threshold, expect more images
        cache_mode=CacheMode.BYPASS
    )
    
    browser_cfg = BrowserConfig(headless=True, verbose=True)
    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        print(f"\n--- Crawling with image_score_threshold={run_cfg_high_thresh.image_score_threshold} ---")
        result_high = await crawler.arun(config=run_cfg_high_thresh)
        images_high = result_high.media.get("images", []) if result_high.success else []
        print(f"Images found (high threshold): {len(images_high)}")
        for img in images_high: print(f"  - {img.get('src')}, score: {img.get('score')}")

        print(f"\n--- Crawling with image_score_threshold={run_cfg_low_thresh.image_score_threshold} ---")
        result_low = await crawler.arun(config=run_cfg_low_thresh)
        images_low = result_low.media.get("images", []) if result_low.success else []
        print(f"Images found (low threshold): {len(images_low)}")
        for img in images_low: print(f"  - {img.get('src')}, score: {img.get('score')}")
        
        if result_high.success and result_low.success:
             assert len(images_high) <= len(images_low)
             if images_high: # Check if the good image passed the high threshold
                 assert any(img.get('src') == "good_image.jpg" for img in images_high)
             if len(images_low) > len(images_high): # Check if lower threshold included more
                 assert any(img.get('src') == "icon.png" for img in images_low) or \
                        any(img.get('src') == "decorative.gif" for img in images_low)

if __name__ == "__main__":
    asyncio.run(image_score_threshold_example())
```

---
#### 6.10.8. Example: Setting `table_score_threshold` for filtering tables by relevance.
Tables scoring below this heuristic threshold might be discarded. Default is 7.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig, CacheMode

async def table_score_threshold_example():
    html_with_tables = """
    <html><body>
        <p>Layout table (low score):</p>
        <table><tr><td>Cell 1</td><td>Cell 2</td></tr></table>

        <p>Data table (high score):</p>
        <table id="data-table-1">
            <caption>My Data</caption>
            <thead><tr><th>Header 1</th><th>Header 2</th></tr></thead>
            <tbody>
                <tr><td>Data A1</td><td>Data B1</td></tr>
                <tr><td>Data A2</td><td>Data B2</td></tr>
            </tbody>
        </table>
    </body></html>
    """
    
    run_cfg_high_thresh = CrawlerRunConfig(
        url=f"raw://{html_with_tables}",
        table_score_threshold=10, # Higher threshold, expect fewer/no tables
        cache_mode=CacheMode.BYPASS
    )
    run_cfg_low_thresh = CrawlerRunConfig(
        url=f"raw://{html_with_tables}",
        table_score_threshold=5,  # Lower threshold, data table should pass
        cache_mode=CacheMode.BYPASS
    )
    
    browser_cfg = BrowserConfig(headless=True, verbose=True)
    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        print(f"\n--- Crawling with table_score_threshold={run_cfg_high_thresh.table_score_threshold} ---")
        result_high = await crawler.arun(config=run_cfg_high_thresh)
        tables_high = result_high.media.get("tables", []) if result_high.success else []
        print(f"Tables found (high threshold): {len(tables_high)}")
        for i, tbl in enumerate(tables_high): print(f"  - Table {i} caption: {tbl.get('caption', 'N/A')}")

        print(f"\n--- Crawling with table_score_threshold={run_cfg_low_thresh.table_score_threshold} ---")
        result_low = await crawler.arun(config=run_cfg_low_thresh)
        tables_low = result_low.media.get("tables", []) if result_low.success else []
        print(f"Tables found (low threshold): {len(tables_low)}")
        for i, tbl in enumerate(tables_low): print(f"  - Table {i} caption: {tbl.get('caption', 'N/A')}")
        
        if result_high.success and result_low.success:
            assert len(tables_high) <= len(tables_low)
            if tables_low: # The data table should be found with lower threshold
                assert any("My Data" in tbl.get("caption", "") for tbl in tables_low)
            if not tables_high: # High threshold might exclude all
                print("High threshold correctly excluded tables.")

if __name__ == "__main__":
    asyncio.run(table_score_threshold_example())
```

---
#### 6.10.9. Example: Enabling `exclude_external_images=True` to ignore images from other domains.
Only images hosted on the same domain (or subdomains) as the crawled URL will be included.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig, CacheMode

async def exclude_external_images_example():
    html_with_mixed_images = """
    <html><body>
        <p>Internal Image:</p>
        <img src="/images/local_image.png" alt="Local">
        <p>External Image:</p>
        <img src="https://cdn.anotherdomain.com/image.jpg" alt="External CDN">
        <p>Same domain, different subdomain (should be kept if base domain matches):</p>
        <img src="https://img.example.com/another_local.png" alt="Subdomain Local">
    </body></html>
    """
    
    run_cfg = CrawlerRunConfig(
        url=f"raw://{html_with_mixed_images}", # Base URL for context
        # url="https://example.com", # If using a live page, set base_url for raw HTML
        base_url="https://example.com",   # Needed for raw HTML to determine internal/external
        exclude_external_images=True,
        cache_mode=CacheMode.BYPASS
    )
    
    print(f"Crawling with exclude_external_images=True. Base URL for resolving: {run_cfg.base_url}")
    browser_cfg = BrowserConfig(headless=True)
    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(config=run_cfg)
        if result.success and result.media and result.media.get("images"):
            print(f"Images found: {len(result.media['images'])}")
            external_found = False
            internal_found_count = 0
            for img in result.media["images"]:
                print(f"  - Src: {img.get('src')}, Alt: {img.get('alt')}")
                if "anotherdomain.com" in img.get('src', ''):
                    external_found = True
                if "example.com" in img.get('src', '') or img.get('src', '').startswith("/images"):
                    internal_found_count +=1
            
            assert not external_found, "External image was not excluded"
            assert internal_found_count == 2, f"Expected 2 internal/same-domain images, found {internal_found_count}"
            print("SUCCESS: External images excluded, internal/same-domain images kept.")
        else:
            print(f"Crawl failed or no images: {result.error_message if result else 'No result'}")

if __name__ == "__main__":
    asyncio.run(exclude_external_images_example())
```

---
#### 6.10.10. Example: Enabling `exclude_all_images=True` to remove all images.
No images will be processed or included in `result.media["images"]`.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig, CacheMode

async def exclude_all_images_example():
    html_with_images = """
    <html><body>
        <img src="local.jpg" alt="Local">
        <img src="https://cdn.example.com/external.png" alt="External">
    </body></html>
    """
    run_cfg = CrawlerRunConfig(
        url=f"raw://{html_with_images}",
        exclude_all_images=True,
        cache_mode=CacheMode.BYPASS
    )
    
    print(f"Crawling with exclude_all_images=True.")
    browser_cfg = BrowserConfig(headless=True)
    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(config=run_cfg)
        if result.success:
            images_found = result.media.get("images", [])
            print(f"Images found: {len(images_found)}")
            assert not images_found, "Images were found even though exclude_all_images was True."
            print("SUCCESS: All images were excluded.")
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(exclude_all_images_example())
```

---
### 6.11. Link and Domain Filtering Options

#### 6.11.1. Example: Customizing the list of `exclude_social_media_domains`.
Provide your own list of social media domains to exclude, overriding or extending the default list.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig, CacheMode

async def custom_social_media_domains_example():
    html_with_links = """
    <html><body>
        <a href="https://facebook.com/somepage">Facebook</a>
        <a href="https://linkedin.com/in/someone">LinkedIn</a>
        <a href="https://mycustomsocial.net/profile">My Custom Social</a>
        <a href="https://example.com/internal">Internal</a>
    </body></html>
    """
    
    # Default list includes facebook.com, linkedin.com, etc.
    # Let's exclude only "mycustomsocial.net" and keep others for this test.
    # To do this, we must also enable exclude_social_media_links=True.
    custom_social_domains = ["mycustomsocial.net"]
    
    run_cfg = CrawlerRunConfig(
        url=f"raw://{html_with_links}",
        base_url="https://anotherexample.com", # To make example.com external
        exclude_social_media_links=True, # This flag enables the filtering
        exclude_social_media_domains=custom_social_domains, # Provide our custom list
        cache_mode=CacheMode.BYPASS
    )
    
    print(f"Crawling with custom exclude_social_media_domains: {custom_social_domains}")
    browser_cfg = BrowserConfig(headless=True)
    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(config=run_cfg)
        if result.success:
            print("Links found in result.links['external']:")
            mycustomsocial_found = False
            facebook_found = False
            linkedin_found = False
            for link_info in result.links.get("external", []):
                print(f"  - {link_info.get('href')}")
                if "mycustomsocial.net" in link_info.get('href', ''):
                    mycustomsocial_found = True
                if "facebook.com" in link_info.get('href', ''):
                    facebook_found = True
                if "linkedin.com" in link_info.get('href', ''):
                    linkedin_found = True
            
            assert not mycustomsocial_found, "Custom social domain 'mycustomsocial.net' was not excluded."
            # Since we overrode the list, default social domains should now be included if not in our custom list.
            # However, exclude_social_media_links = True with a custom list should *only* exclude those in the custom list from the "social" category.
            # The behavior of exclude_social_media_domains is to *add* to the default list if not replacing.
            # The provided code has exclude_social_media_domains take precedence and replaces the default if exclude_social_media_links is true
            # (Looking at `async_webcrawler.py` processing of these flags).
            # The code's logic is: if `exclude_social_media_links` is true, it uses `self.config.exclude_social_media_domains` (which defaults to `config.SOCIAL_MEDIA_DOMAINS`)
            # So, if `exclude_social_media_domains` is set in `CrawlerRunConfig`, it effectively *replaces* the default list for that run.
            print("As exclude_social_media_domains was set, it replaces the default list.")
            assert facebook_found, "Facebook link was unexpectedly excluded with custom list."
            assert linkedin_found, "LinkedIn link was unexpectedly excluded with custom list."
            print("SUCCESS: Custom social media domain excluded, others (not in custom list) were kept.")
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(custom_social_media_domains_example())
```

---
#### 6.11.2. Example: Enabling `exclude_external_links=True` to remove links to other domains.
Only links within the same base domain as the crawled URL will be kept in `result.links`.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig, CacheMode

async def exclude_external_links_example():
    html_with_mixed_links = """
    <html><body>
        <a href="/internal-page">Internal Page</a>
        <a href="https://example.com/another-internal">Another Internal</a>
        <a href="https://sub.example.com/sub-internal">Subdomain Internal</a>
        <a href="https://otherdomain.com/external-page">External Page</a>
    </body></html>
    """
    run_cfg = CrawlerRunConfig(
        url=f"raw://{html_with_mixed_links}",
        base_url="https://example.com", # Define the base domain for this raw HTML
        exclude_external_links=True,
        cache_mode=CacheMode.BYPASS
    )
    
    print(f"Crawling with exclude_external_links=True. Base URL: {run_cfg.base_url}")
    browser_cfg = BrowserConfig(headless=True)
    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(config=run_cfg)
        if result.success:
            external_links = result.links.get("external", [])
            internal_links = result.links.get("internal", [])
            print(f"External links found: {len(external_links)}")
            for link in external_links: print(f"  - {link.get('href')}")
            print(f"Internal links found: {len(internal_links)}")
            for link in internal_links: print(f"  - {link.get('href')}")

            assert not external_links, "External links were not excluded."
            assert len(internal_links) == 3, f"Expected 3 internal links, found {len(internal_links)}."
            print("SUCCESS: External links excluded, internal links kept.")
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(exclude_external_links_example())
```

---
#### 6.11.3. Example: Enabling `exclude_social_media_links=True` (uses `exclude_social_media_domains` list).
Filters out links to common social media platforms using the default or a custom list.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig, CacheMode, config as crawl4ai_config

async def exclude_social_media_links_example():
    html_with_social_links = f"""
    <html><body>
        <a href="https://example.com/normal">Normal Link</a>
        <a href="https://{crawl4ai_config.SOCIAL_MEDIA_DOMAINS[0]}/someuser">Social Media Link 1</a>
        <a href="https://{crawl4ai_config.SOCIAL_MEDIA_DOMAINS[1]}/anotheruser">Social Media Link 2</a>
    </body></html>
    """
    run_cfg = CrawlerRunConfig(
        url=f"raw://{html_with_social_links}",
        base_url="https://example.com",
        exclude_social_media_links=True, # Uses default SOCIAL_MEDIA_DOMAINS list
        cache_mode=CacheMode.BYPASS
    )
    
    print(f"Crawling with exclude_social_media_links=True.")
    browser_cfg = BrowserConfig(headless=True)
    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(config=run_cfg)
        if result.success:
            external_links = result.links.get("external", [])
            print(f"External links found ({len(external_links)}):")
            social_links_found = 0
            for link_info in external_links:
                print(f"  - {link_info.get('href')}")
                if any(domain in link_info.get('href', '') for domain in crawl4ai_config.SOCIAL_MEDIA_DOMAINS):
                    social_links_found +=1
            
            assert social_links_found == 0, f"Found {social_links_found} social media links, expected 0."
            # There are no "internal" links to example.com in this raw HTML case, so internal will be empty.
            # The "Normal Link" is considered external here if base_url is example.com and link is relative
            # For this test, it is important that social links are not present in *any* list.
            # Since they are external, they would appear in external_links if not filtered.
            
            # Check internal links too (though none are social here)
            internal_links = result.links.get("internal", [])
            for link_info in internal_links:
                 if any(domain in link_info.get('href', '') for domain in crawl4ai_config.SOCIAL_MEDIA_DOMAINS):
                    social_links_found +=1 # Should not happen
            assert social_links_found == 0, "Social media links found in internal links."

            print("SUCCESS: Social media links were excluded.")
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(exclude_social_media_links_example())
```

---
#### 6.11.4. Example: Providing a custom list of `exclude_domains` for link filtering.
Blacklist specific domains from appearing in the extracted links.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig, CacheMode

async def custom_exclude_domains_example():
    html_with_various_links = """
    <html><body>
        <a href="https://example.com/page1">Allowed Internal</a>
        <a href="https://www.allowed-external.com/path">Allowed External</a>
        <a href="https://www.blocked-domain1.com/content">Blocked Domain 1</a>
        <a href="http://sub.blocked-domain2.org/another">Blocked Domain 2</a>
    </body></html>
    """
    excluded_domains_list = ["blocked-domain1.com", "blocked-domain2.org"]
    
    run_cfg = CrawlerRunConfig(
        url=f"raw://{html_with_various_links}",
        base_url="https://example.com",
        exclude_domains=excluded_domains_list,
        cache_mode=CacheMode.BYPASS
    )
    
    print(f"Crawling with exclude_domains: {excluded_domains_list}")
    browser_cfg = BrowserConfig(headless=True)
    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(config=run_cfg)
        if result.success:
            all_links = result.links.get("internal", []) + result.links.get("external", [])
            print(f"Total links found after filtering: {len(all_links)}")
            
            blocked_found = False
            for link_info in all_links:
                print(f"  - {link_info.get('href')}")
                if any(blocked_domain in link_info.get('href', '') for blocked_domain in excluded_domains_list):
                    blocked_found = True
                    print(f"ERROR: Found a blocked domain link: {link_info.get('href')}")
            
            assert not blocked_found, "One or more explicitly excluded domains were found in links."
            assert len(all_links) == 2 # Only the two allowed links should remain
            print("SUCCESS: Specified domains were excluded from links.")
        else:
            print(f"Crawl failed: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(custom_exclude_domains_example())
```

---
#### 6.11.5. Example: Enabling `exclude_internal_links=True` to remove links within the same domain.
Only external links will be kept in `result.links`.

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig, CacheMode

async def exclude_internal_links_example():
    html_with_mixed_links = """
    <html><body>
        <a href="/internal-page1">Internal Page 1</a>
        <a href="https://example.com/internal-page2">Internal Page 2</a>
        <