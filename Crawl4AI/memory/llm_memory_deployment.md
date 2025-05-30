```markdown
# Detailed Outline for crawl4ai - deployment Component

**Target Document Type:** memory
**Target Output Filename Suggestion:** `llm_memory_deployment.md`
**Library Version Context:** 0.6.0 (as per Dockerfile ARG `C4AI_VER` from provided `Dockerfile` content)
**Outline Generation Date:** 2025-05-24
---

## 1. Introduction to Deployment
    * 1.1. Purpose: This document provides a factual reference for installing the `crawl4ai` library and deploying its server component using Docker. It covers basic and advanced library installation, various Docker deployment methods, server configuration, and an overview of the API for interaction.
    * 1.2. Scope:
        * Installation of the `crawl4ai` Python library.
        * Setup and diagnostic commands for the library.
        * Deployment of the `crawl4ai` server using Docker, including pre-built images, Docker Compose, and manual builds.
        * Explanation of Dockerfile parameters and server configuration via `config.yml`.
        * Details of API interaction, including the Playground UI, Python SDK, and direct REST API calls.
        * Overview of additional server API endpoints and Model Context Protocol (MCP) support.
        * High-level understanding of the server's internal logic relevant to users.
        * The library's version numbering scheme.

## 2. Library Installation

    * 2.1. **Basic Library Installation**
        * 2.1.1. Standard Installation
            * Command: `pip install crawl4ai`
            * Purpose: Installs the core `crawl4ai` library and its essential dependencies for performing web crawling and scraping tasks. This provides the fundamental `AsyncWebCrawler` and related configuration objects.
        * 2.1.2. Post-Installation Setup
            * Command: `crawl4ai-setup`
            * Purpose:
                * Initializes the user's home directory structure for Crawl4ai (e.g., `~/.crawl4ai/cache`).
                * Installs or updates necessary Playwright browsers (Chromium is installed by default) required for browser-based crawling. The `crawl4ai-setup` script internally calls `playwright install --with-deps chromium`.
                * Performs OS-level checks for common missing libraries that Playwright might depend on, providing guidance if issues are found.
                * Creates a default `global.yml` configuration file if one doesn't exist.
        * 2.1.3. Diagnostic Check
            * Command: `crawl4ai-doctor`
            * Purpose:
                * Verifies Python version compatibility.
                * Confirms Playwright installation and browser integrity by attempting a simple crawl of `https://crawl4ai.com`.
                * Inspects essential environment variables and potential library conflicts that might affect Crawl4ai's operation.
                * Provides diagnostic messages indicating success or failure of these checks, with suggestions for resolving common issues.
        * 2.1.4. Verification Process
            * Purpose: To confirm that the basic installation and setup were successful and Crawl4ai can perform a simple crawl.
            * Script Example (as inferred from `crawl4ai-doctor` logic and typical usage):
                ```python
                import asyncio
                from crawl4ai import AsyncWebCrawler, BrowserConfig, CrawlerRunConfig, CacheMode

                async def main():
                    browser_config = BrowserConfig(
                        headless=True,
                        browser_type="chromium",
                        ignore_https_errors=True,
                        light_mode=True,
                        viewport_width=1280,
                        viewport_height=720,
                    )
                    run_config = CrawlerRunConfig(
                        cache_mode=CacheMode.BYPASS,
                        screenshot=True,
                    )
                    async with AsyncWebCrawler(config=browser_config) as crawler:
                        print("Testing crawling capabilities...")
                        result = await crawler.arun(url="https://crawl4ai.com", config=run_config)
                        if result and result.markdown:
                            print("âœ… Crawling test passed!")
                            return True
                        else:
                            print("âŒ Test failed: Failed to get content")
                            return False

                if __name__ == "__main__":
                    asyncio.run(main())
                ```
            * Expected Outcome: The script should print "âœ… Crawling test passed!" and successfully output Markdown content from the crawled page.

    * 2.2. **Advanced Library Installation (Optional Features)**
        * 2.2.1. Installation of Optional Extras
            * Purpose: To install additional dependencies required for specific advanced features of Crawl4ai, such as those involving machine learning models.
            * Options (as defined in `pyproject.toml`):
                * `pip install crawl4ai[pdf]`:
                    * Purpose: Installs `PyPDF2` for PDF processing capabilities.
                * `pip install crawl4ai[torch]`:
                    * Purpose: Installs `torch`, `nltk`, and `scikit-learn`. Enables features relying on PyTorch models, such as some advanced text clustering or semantic analysis within extraction strategies.
                * `pip install crawl4ai[transformer]`:
                    * Purpose: Installs `transformers` and `tokenizers`. Enables the use of Hugging Face Transformers models for tasks like summarization, question answering, or other advanced NLP features within Crawl4ai.
                * `pip install crawl4ai[cosine]`:
                    * Purpose: Installs `torch`, `transformers`, and `nltk`. Specifically for features utilizing cosine similarity with embeddings (implies model usage).
                * `pip install crawl4ai[sync]`:
                    * Purpose: Installs `selenium` for synchronous crawling capabilities (less common, as Crawl4ai primarily focuses on async).
                * `pip install crawl4ai[all]`:
                    * Purpose: Installs all optional dependencies listed above (`PyPDF2`, `torch`, `nltk`, `scikit-learn`, `transformers`, `tokenizers`, `selenium`), providing the complete suite of Crawl4ai capabilities.
        * 2.2.2. Model Pre-fetching
            * Command: `crawl4ai-download-models` (maps to `crawl4ai.model_loader:main`)
            * Purpose: Downloads and caches machine learning models (e.g., specific sentence transformers or classification models from Hugging Face) that are used by certain optional features, particularly those installed via `crawl4ai[transformer]` or `crawl4ai[cosine]`. This avoids runtime downloads and ensures models are available offline.

## 3. Docker Deployment (Server Mode)

    * 3.1. **Prerequisites**
        * 3.1.1. Docker: A working Docker installation. (Link: `https://docs.docker.com/get-docker/`)
        * 3.1.2. Git: Required for cloning the `crawl4ai` repository if building locally or using Docker Compose from the repository. (Link: `https://git-scm.com/book/en/v2/Getting-Started-Installing-Git`)
        * 3.1.3. RAM Requirements:
            * Minimum: 2GB for the basic server without intensive LLM tasks. The `Dockerfile` HEALTCHECK indicates a warning if less than 2GB RAM is available.
            * Recommended for LLM support: 4GB+ (as specified in `docker-compose.yml` limits).
            * Shared Memory (`/dev/shm`): Recommended size is 1GB (`--shm-size=1g`) for optimal Chromium browser performance, as specified in `docker-compose.yml` and run commands.
    * 3.2. **Installation Options**
        * 3.2.1. **Using Pre-built Images from Docker Hub**
            * 3.2.1.1. Image Source: `unclecode/crawl4ai:<tag>`
                * Explanation of `<tag>`:
                    * `latest`: Points to the most recent stable release of Crawl4ai.
                    * Specific version tags (e.g., `0.6.0`, `0.5.1`): Correspond to specific library releases.
                    * Pre-release tags (e.g., `0.6.0-rc1`, `0.7.0-devN`): Development or release candidate versions for testing.
            * 3.2.1.2. Pulling the Image
                * Command: `docker pull unclecode/crawl4ai:<tag>` (e.g., `docker pull unclecode/crawl4ai:latest`)
            * 3.2.1.3. Environment Setup (`.llm.env`)
                * File Name: `.llm.env` (to be created by the user in the directory where `docker run` or `docker-compose` commands are executed).
                * Purpose: To securely provide API keys for various LLM providers used by Crawl4ai for features like LLM-based extraction or Q&A.
                * Example Content (based on `docker-compose.yml`):
                    ```env
                    OPENAI_API_KEY=your_openai_api_key
                    DEEPSEEK_API_KEY=your_deepseek_api_key
                    ANTHROPIC_API_KEY=your_anthropic_api_key
                    GROQ_API_KEY=your_groq_api_key
                    TOGETHER_API_KEY=your_together_api_key
                    MISTRAL_API_KEY=your_mistral_api_key
                    GEMINI_API_TOKEN=your_gemini_api_token
                    ```
                * Creation: Users should create this file and populate it with their API keys. An example (`.llm.env.example`) might be provided in the repository.
            * 3.2.1.4. Running the Container
                * Basic Run (without LLM support):
                    * Command: `docker run -d -p 11235:11235 --shm-size=1g --name crawl4ai-server unclecode/crawl4ai:<tag>`
                    * Port Mapping: `-p 11235:11235` maps port 11235 on the host to port 11235 in the container (default server port).
                    * Shared Memory: `--shm-size=1g` allocates 1GB of shared memory for the browser.
                * Run with LLM Support (mounting `.llm.env`):
                    * Command: `docker run -d -p 11235:11235 --env-file .llm.env --shm-size=1g --name crawl4ai-server unclecode/crawl4ai:<tag>`
            * 3.2.1.5. Stopping the Container
                * Command: `docker stop crawl4ai-server`
                * Command (to remove): `docker rm crawl4ai-server`
            * 3.2.1.6. Docker Hub Versioning:
                * Docker image tags on Docker Hub (e.g., `unclecode/crawl4ai:0.6.0`) directly correspond to `crawl4ai` library releases. The `latest` tag usually points to the most recent stable release. Pre-release tags include suffixes like `-devN`, `-aN`, `-bN`, or `-rcN`.

        * 3.2.2. **Using Docker Compose (`docker-compose.yml`)**
            * 3.2.2.1. Cloning the Repository
                * Command: `git clone https://github.com/unclecode/crawl4ai.git`
                * Command: `cd crawl4ai`
            * 3.2.2.2. Environment Setup (`.llm.env`)
                * File Name: `.llm.env` (should be created in the root of the cloned `crawl4ai` repository).
                * Purpose: Same as above, to provide LLM API keys.
            * 3.2.2.3. Running Pre-built Images
                * Command: `docker-compose up -d`
                * Behavior: Uses the image specified in `docker-compose.yml` (e.g., `${IMAGE:-unclecode/crawl4ai}:${TAG:-latest}`).
                * Overriding image tag: `TAG=0.6.0 docker-compose up -d` or `IMAGE=mycustom/crawl4ai TAG=mytag docker-compose up -d`.
            * 3.2.2.4. Building Locally with Docker Compose
                * Command: `docker-compose up -d --build`
                * Build Arguments (passed from environment variables to `docker-compose.yml` which then passes to `Dockerfile`):
                    * `INSTALL_TYPE`: (e.g., `default`, `torch`, `all`)
                        * Purpose: To include optional Python dependencies during the Docker image build process.
                        * Example: `INSTALL_TYPE=all docker-compose up -d --build`
                    * `ENABLE_GPU`: (e.g., `true`, `false`)
                        * Purpose: To include GPU support (e.g., CUDA toolkits) in the Docker image if the build hardware and target runtime support it.
                        * Example: `ENABLE_GPU=true docker-compose up -d --build`
            * 3.2.2.5. Stopping Docker Compose Services
                * Command: `docker-compose down`

        * 3.2.3. **Manual Local Build & Run**
            * 3.2.3.1. Cloning the Repository: (As above)
            * 3.2.3.2. Environment Setup (`.llm.env`): (As above)
            * 3.2.3.3. Building with `docker buildx`
                * Command Example:
                    ```bash
                    docker buildx build --platform linux/amd64,linux/arm64 \
                      --build-arg C4AI_VER=0.6.0 \
                      --build-arg INSTALL_TYPE=all \
                      --build-arg ENABLE_GPU=false \
                      --build-arg USE_LOCAL=true \
                      -t my-crawl4ai-image:custom .
                    ```
                * Purpose of `docker buildx`: A Docker CLI plugin that extends the `docker build` command with full support for BuildKit builder capabilities, including multi-architecture builds.
                * Explanation of `--platform`: Specifies the target platform(s) for the build (e.g., `linux/amd64`, `linux/arm64`).
                * Explanation of `--build-arg`: Passes build-time variables defined in the `Dockerfile` (see section 3.3).
            * 3.2.3.4. Running the Custom-Built Container
                * Basic Run: `docker run -d -p 11235:11235 --shm-size=1g --name my-crawl4ai-server my-crawl4ai-image:custom`
                * Run with LLM Support: `docker run -d -p 11235:11235 --env-file .llm.env --shm-size=1g --name my-crawl4ai-server my-crawl4ai-image:custom`
            * 3.2.3.5. Stopping the Container: (As above)

    * 3.3. **Dockerfile Parameters (`ARG` values)**
        * 3.3.1. `C4AI_VER`: (Default: `0.6.0`)
            * Role: Specifies the version of the `crawl4ai` library. Used for labeling the image and potentially for version-specific logic.
        * 3.3.2. `APP_HOME`: (Default: `/app`)
            * Role: Defines the working directory inside the Docker container where the application code and related files are stored and executed.
        * 3.3.3. `GITHUB_REPO`: (Default: `https://github.com/unclecode/crawl4ai.git`)
            * Role: The URL of the GitHub repository to clone if `USE_LOCAL` is set to `false`.
        * 3.3.4. `GITHUB_BRANCH`: (Default: `main`)
            * Role: The specific branch of the GitHub repository to clone if `USE_LOCAL` is `false`.
        * 3.3.5. `USE_LOCAL`: (Default: `true`)
            * Role: A boolean flag. If `true`, the `Dockerfile` installs `crawl4ai` from the local source code copied into `/tmp/project/` during the build context. If `false`, it clones the repository specified by `GITHUB_REPO` and `GITHUB_BRANCH`.
        * 3.3.6. `PYTHON_VERSION`: (Default: `3.12`)
            * Role: Specifies the Python version for the base image (e.g., `python:3.12-slim-bookworm`).
        * 3.3.7. `INSTALL_TYPE`: (Default: `default`)
            * Role: Controls which optional dependencies of `crawl4ai` are installed. Possible values: `default` (core), `pdf`, `torch`, `transformer`, `cosine`, `sync`, `all`.
        * 3.3.8. `ENABLE_GPU`: (Default: `false`)
            * Role: A boolean flag. If `true` and `TARGETARCH` is `amd64`, the `Dockerfile` attempts to install the NVIDIA CUDA toolkit for GPU acceleration.
        * 3.3.9. `TARGETARCH`:
            * Role: An automatic build argument provided by Docker, indicating the target architecture of the build (e.g., `amd64`, `arm64`). Used for conditional logic in the `Dockerfile`, such as installing platform-specific optimized libraries or CUDA for `amd64`.

    * 3.4. **Server Configuration (`config.yml`)**
        * 3.4.1. Location: The server loads its configuration from `/app/config.yml` inside the container by default. This path is relative to `APP_HOME`.
        * 3.4.2. Structure Overview (based on `deploy/docker/config.yml`):
            * `app`: General application settings.
                * `title (str)`: API title (e.g., "Crawl4AI API").
                * `version (str)`: API version (e.g., "1.0.0").
                * `host (str)`: Host address for the server to bind to (e.g., "0.0.0.0").
                * `port (int)`: Port for the server to listen on (e.g., 11234, though Docker usually maps to 11235).
                * `reload (bool)`: Enable/disable auto-reload for development (default: `false`).
                * `workers (int)`: Number of worker processes (default: 1).
                * `timeout_keep_alive (int)`: Keep-alive timeout in seconds (default: 300).
            * `llm`: Default LLM configuration.
                * `provider (str)`: Default LLM provider string (e.g., "openai/gpt-4o-mini").
                * `api_key_env (str)`: Environment variable name to read the API key from (e.g., "OPENAI_API_KEY").
                * `api_key (Optional[str])`: Directly pass API key (overrides `api_key_env`).
            * `redis`: Redis connection details.
                * `host (str)`: Redis host (e.g., "localhost").
                * `port (int)`: Redis port (e.g., 6379).
                * `db (int)`: Redis database number (e.g., 0).
                * `password (str)`: Redis password (default: "").
                * `ssl (bool)`: Enable SSL for Redis connection (default: `false`).
                * `ssl_cert_reqs (Optional[str])`: SSL certificate requirements (e.g., "none", "optional", "required").
                * `ssl_ca_certs (Optional[str])`: Path to CA certificate file.
                * `ssl_certfile (Optional[str])`: Path to SSL certificate file.
                * `ssl_keyfile (Optional[str])`: Path to SSL key file.
            * `rate_limiting`: Configuration for API rate limits.
                * `enabled (bool)`: Enable/disable rate limiting (default: `true`).
                * `default_limit (str)`: Default rate limit (e.g., "1000/minute").
                * `trusted_proxies (List[str])`: List of trusted proxy IP addresses.
                * `storage_uri (str)`: Storage URI for rate limit counters (e.g., "memory://", "redis://localhost:6379").
            * `security`: Security-related settings.
                * `enabled (bool)`: Master switch for security features (default: `false`).
                * `jwt_enabled (bool)`: Enable/disable JWT authentication (default: `false`).
                * `https_redirect (bool)`: Enable/disable HTTPS redirection (default: `false`).
                * `trusted_hosts (List[str])`: List of allowed host headers (e.g., `["*"]` or specific domains).
                * `headers (Dict[str, str])`: Default security headers to add to responses (e.g., `X-Content-Type-Options`, `Content-Security-Policy`).
            * `crawler`: Default crawler behavior.
                * `base_config (Dict[str, Any])`: Base parameters for `CrawlerRunConfig`.
                    * `simulate_user (bool)`: (default: `true`).
                * `memory_threshold_percent (float)`: Memory usage threshold for adaptive dispatcher (default: `95.0`).
                * `rate_limiter (Dict[str, Any])`: Configuration for the internal rate limiter for crawling.
                    * `enabled (bool)`: (default: `true`).
                    * `base_delay (List[float, float])`: Min/max delay range (e.g., `[1.0, 2.0]`).
                * `timeouts (Dict[str, float])`: Timeouts for different crawler operations.
                    * `stream_init (float)`: Timeout for stream initialization (default: `30.0`).
                    * `batch_process (float)`: Timeout for batch processing (default: `300.0`).
                * `pool (Dict[str, Any])`: Browser pool settings.
                    * `max_pages (int)`: Max concurrent browser pages (default: `40`).
                    * `idle_ttl_sec (int)`: Time-to-live for idle crawlers in seconds (default: `1800`).
                * `browser (Dict[str, Any])`: Default `BrowserConfig` parameters.
                    * `kwargs (Dict[str, Any])`: Keyword arguments for `BrowserConfig`.
                        * `headless (bool)`: (default: `true`).
                        * `text_mode (bool)`: (default: `true`).
                    * `extra_args (List[str])`: List of additional browser launch arguments (e.g., `"--no-sandbox"`).
            * `logging`: Logging configuration.
                * `level (str)`: Logging level (e.g., "INFO", "DEBUG").
                * `format (str)`: Log message format string.
            * `observability`: Observability settings.
                * `prometheus (Dict[str, Any])`: Prometheus metrics configuration.
                    * `enabled (bool)`: (default: `true`).
                    * `endpoint (str)`: Metrics endpoint path (e.g., "/metrics").
                * `health_check (Dict[str, str])`: Health check endpoint configuration.
                    * `endpoint (str)`: Health check endpoint path (e.g., "/health").
        * 3.4.3. JWT Authentication
            * Enabling: Set `security.enabled: true` and `security.jwt_enabled: true` in `config.yml`.
            * Secret Key: Configured via `security.jwt_secret_key`. This value can be overridden by the environment variable `JWT_SECRET_KEY`.
            * Algorithm: Configured via `security.jwt_algorithm` (default: `HS256`).
            * Token Expiry: Configured via `security.jwt_expire_minutes` (default: `30`).
            * Usage:
                * 1. Client obtains a token by sending a POST request to the `/token` endpoint with an email in the request body (e.g., `{"email": "user@example.com"}`). The email domain might be validated if configured.
                * 2. Client includes the received token in the `Authorization` header of subsequent requests to protected API endpoints: `Authorization: Bearer <your_jwt_token>`.
        * 3.4.4. Customizing `config.yml`
            * 3.4.4.1. Modifying Before Build:
                * Method: Edit the `deploy/docker/config.yml` file within the cloned `crawl4ai` repository before building the Docker image. This new configuration will be baked into the image.
            * 3.4.4.2. Runtime Mount:
                * Method: Mount a custom `config.yml` file from the host machine to `/app/config.yml` (or the path specified by `APP_HOME`) inside the running Docker container.
                * Example Command: `docker run -d -p 11235:11235 -v /path/on/host/my-config.yml:/app/config.yml --name crawl4ai-server unclecode/crawl4ai:latest`
        * 3.4.5. Key Configuration Recommendations
            * Security:
                * Enable JWT (`security.jwt_enabled: true`) if the server is exposed to untrusted networks.
                * Use a strong, unique `jwt_secret_key`.
                * Configure `security.trusted_hosts` to a specific list of allowed hostnames instead of `["*"]` for production.
                * If using a reverse proxy for SSL termination, ensure `https_redirect` is appropriately configured or disabled if the proxy handles it.
            * Resource Management:
                * Adjust `crawler.pool.max_pages` based on server resources to prevent overwhelming the system.
                * Tune `crawler.pool.idle_ttl_sec` to balance resource usage and responsiveness for pooled browser instances.
            * Monitoring:
                * Keep `observability.prometheus.enabled: true` for production monitoring via the `/metrics` endpoint.
                * Ensure the `/health` endpoint is accessible to health checking systems.
            * Performance:
                * Review and customize `crawler.browser.extra_args` for headless browser optimization (e.g., disabling GPU, sandbox if appropriate for your environment).
                * Set reasonable `crawler.timeouts` to prevent long-stalled crawls.

    * 3.5. **API Usage (Interacting with the Dockerized Server)**
        * 3.5.1. **Playground Interface**
            * Access URL: `http://localhost:11235/playground` (assuming default port mapping).
            * Purpose: An interactive web UI (Swagger UI/OpenAPI) allowing users to explore API endpoints, view schemas, construct requests, and test API calls directly from their browser.
        * 3.5.2. **Python SDK (`Crawl4aiDockerClient`)**
            * Class Name: `Crawl4aiDockerClient`
            * Location: (Typically imported as `from crawl4ai.docker_client import Crawl4aiDockerClient`) - Actual import might vary based on final library structure; refer to `docs/examples/docker_example.py` or `docs/examples/docker_python_sdk.py`.
            * Initialization:
                * Signature: `Crawl4aiDockerClient(base_url: str = "http://localhost:11235", api_token: Optional[str] = None, timeout: int = 300)`
                * Parameters:
                    * `base_url (str)`: The base URL of the Crawl4ai server. Default: `"http://localhost:11235"`.
                    * `api_token (Optional[str])`: JWT token for authentication if enabled on the server. Default: `None`.
                    * `timeout (int)`: Default timeout in seconds for HTTP requests to the server. Default: `300`.
            * Authentication (JWT):
                * Method: Pass the `api_token` during client initialization. The token can be obtained from the server's `/token` endpoint or other authentication mechanisms.
            * `crawl()` Method:
                * Signature (Conceptual, based on typical SDK patterns and server capabilities): `async def crawl(self, urls: Union[str, List[str]], browser_config: Optional[Dict] = None, crawler_config: Optional[Dict] = None, stream: bool = False) -> Union[List[Dict], AsyncGenerator[Dict, None]]`
                    *Note: SDK might take `BrowserConfig` and `CrawlerRunConfig` objects directly, which it then serializes.*
                * Key Parameters:
                    * `urls (Union[str, List[str]])`: A single URL string or a list of URL strings to crawl.
                    * `browser_config (Optional[Dict])`: A dictionary representing the `BrowserConfig` object, or a `BrowserConfig` instance itself.
                    * `crawler_config (Optional[Dict])`: A dictionary representing the `CrawlerRunConfig` object, or a `CrawlerRunConfig` instance itself.
                    * `stream (bool)`: If `True`, the method returns an async generator yielding individual `CrawlResult` dictionaries as they are processed by the server. If `False` (default), it returns a list containing all `CrawlResult` dictionaries after all URLs are processed.
                * Return Type: `List[Dict]` (for `stream=False`) or `AsyncGenerator[Dict, None]` (for `stream=True`), where each `Dict` represents a `CrawlResult`.
                * Streaming Behavior:
                    * `stream=True`: Allows processing of results incrementally, suitable for long crawl jobs or real-time data feeds.
                    * `stream=False`: Collects all results before returning, simpler for smaller batches.
            * `get_schema()` Method:
                * Signature: `async def get_schema(self) -> dict`
                * Return Type: `dict`.
                * Purpose: Fetches the JSON schemas for `BrowserConfig` and `CrawlerRunConfig` from the server's `/schema` endpoint. This helps in constructing valid configuration payloads.
        * 3.5.3. **JSON Request Schema for Configurations**
            * Structure: `{"type": "ClassName", "params": {...}}`
            * Purpose: This structure is used by the server (and expected by the Python SDK internally) to deserialize JSON payloads back into Pydantic configuration objects like `BrowserConfig`, `CrawlerRunConfig`, and their nested strategy objects (e.g., `LLMExtractionStrategy`, `PruningContentFilter`). The `type` field specifies the Python class name, and `params` holds the keyword arguments for its constructor.
            * Example (`BrowserConfig`):
                ```json
                {
                    "type": "BrowserConfig",
                    "params": {
                        "headless": true,
                        "browser_type": "chromium",
                        "viewport_width": 1920,
                        "viewport_height": 1080
                    }
                }
                ```
            * Example (`CrawlerRunConfig` with a nested `LLMExtractionStrategy`):
                ```json
                {
                    "type": "CrawlerRunConfig",
                    "params": {
                        "cache_mode": {"type": "CacheMode", "params": "BYPASS"},
                        "screenshot": false,
                        "extraction_strategy": {
                            "type": "LLMExtractionStrategy",
                            "params": {
                                "llm_config": {
                                    "type": "LLMConfig",
                                    "params": {"provider": "openai/gpt-4o-mini"}
                                },
                                "instruction": "Extract the main title and summary."
                            }
                        }
                    }
                }
                ```
        * 3.5.4. **REST API Examples**
            * `/crawl` Endpoint:
                * URL: `http://localhost:11235/crawl`
                * HTTP Method: `POST`
                * Payload Structure (`CrawlRequest` model from `deploy/docker/schemas.py`):
                    ```json
                    {
                        "urls": ["https://example.com"],
                        "browser_config": { // JSON representation of BrowserConfig
                            "type": "BrowserConfig",
                            "params": {"headless": true}
                        },
                        "crawler_config": { // JSON representation of CrawlerRunConfig
                            "type": "CrawlerRunConfig",
                            "params": {"screenshot": true}
                        }
                    }
                    ```
                * Response Structure: A JSON object, typically `{"success": true, "results": [CrawlResult, ...], "server_processing_time_s": float, ...}`.
            * `/crawl/stream` Endpoint:
                * URL: `http://localhost:11235/crawl/stream`
                * HTTP Method: `POST`
                * Payload Structure: Same as `/crawl` (`CrawlRequest` model).
                * Response Structure: Newline Delimited JSON (NDJSON, `application/x-ndjson`). Each line is a JSON string representing a `CrawlResult` object.
                    * Headers: Includes `Content-Type: application/x-ndjson` and `X-Stream-Status: active` while streaming, and a final JSON object `{"status": "completed"}`.

    * 3.6. **Additional API Endpoints (from `server.py`)**
        * 3.6.1. `/html`
            * Endpoint URL: `/html`
            * HTTP Method: `POST`
            * Purpose: Crawls the given URL, preprocesses its raw HTML content specifically for schema extraction purposes (e.g., by sanitizing and simplifying the structure), and returns the processed HTML.
            * Request Body (`HTMLRequest` from `deploy/docker/schemas.py`):
                * `url (str)`: The URL to fetch and process.
            * Response Structure (JSON):
                * `html (str)`: The preprocessed HTML string.
                * `url (str)`: The original URL requested.
                * `success (bool)`: Indicates if the operation was successful.
        * 3.6.2. `/screenshot`
            * Endpoint URL: `/screenshot`
            * HTTP Method: `POST`
            * Purpose: Captures a full-page PNG screenshot of the specified URL. Allows an optional delay before capture and an option to save the file server-side.
            * Request Body (`ScreenshotRequest` from `deploy/docker/schemas.py`):
                * `url (str)`: The URL to take a screenshot of.
                * `screenshot_wait_for (Optional[float])`: Seconds to wait before taking the screenshot. Default: `2.0`.
                * `output_path (Optional[str])`: If provided, the screenshot is saved to this path on the server, and the path is returned. Otherwise, the base64 encoded image is returned. Default: `None`.
            * Response Structure (JSON):
                * `success (bool)`: Indicates if the screenshot was successfully taken.
                * `screenshot (Optional[str])`: Base64 encoded PNG image data, if `output_path` was not provided.
                * `path (Optional[str])`: The absolute server-side path to the saved screenshot, if `output_path` was provided.
        * 3.6.3. `/pdf`
            * Endpoint URL: `/pdf`
            * HTTP Method: `POST`
            * Purpose: Generates a PDF document of the rendered content of the specified URL.
            * Request Body (`PDFRequest` from `deploy/docker/schemas.py`):
                * `url (str)`: The URL to convert to PDF.
                * `output_path (Optional[str])`: If provided, the PDF is saved to this path on the server, and the path is returned. Otherwise, the base64 encoded PDF data is returned. Default: `None`.
            * Response Structure (JSON):
                * `success (bool)`: Indicates if the PDF generation was successful.
                * `pdf (Optional[str])`: Base64 encoded PDF data, if `output_path` was not provided.
                * `path (Optional[str])`: The absolute server-side path to the saved PDF, if `output_path` was provided.
        * 3.6.4. `/execute_js`
            * Endpoint URL: `/execute_js`
            * HTTP Method: `POST`
            * Purpose: Executes a list of JavaScript snippets on the specified URL in the browser context and returns the full `CrawlResult` object, including any modifications or data retrieved by the scripts.
            * Request Body (`JSEndpointRequest` from `deploy/docker/schemas.py`):
                * `url (str)`: The URL on which to execute the JavaScript.
                * `scripts (List[str])`: A list of JavaScript code snippets to execute sequentially. Each script should be an expression that returns a value.
            * Response Structure (JSON): A `CrawlResult` object (serialized to a dictionary) containing the state of the page after JS execution, including `js_execution_result`.
        * 3.6.5. `/ask` (Endpoint defined as `/ask` in `server.py`)
            * Endpoint URL: `/ask`
            * HTTP Method: `GET`
            * Purpose: Retrieves context about the Crawl4ai library itself, either code snippets or documentation sections, filtered by a query. This is designed for AI assistants or RAG systems needing information about Crawl4ai.
            * Parameters (Query):
                * `context_type (str, default="all", enum=["code", "doc", "all"])`: Specifies whether to return "code", "doc", or "all" (both).
                * `query (Optional[str])`: A search query string used to filter relevant chunks using BM25 ranking. If `None`, returns all context of the specified type(s).
                * `score_ratio (float, default=0.5, ge=0.0, le=1.0)`: The minimum score (as a fraction of the maximum possible score for the query) for a chunk to be included in the results.
                * `max_results (int, default=20, ge=1)`: The maximum number of result chunks to return.
            * Response Structure (JSON):
                * If `query` is provided:
                    * `code_results (Optional[List[Dict[str, Union[str, float]]]])`: A list of dictionaries, where each dictionary contains `{"text": "code_chunk...", "score": bm25_score}`. Present if `context_type` is "code" or "all".
                    * `doc_results (Optional[List[Dict[str, Union[str, float]]]])`: A list of dictionaries, where each dictionary contains `{"text": "doc_chunk...", "score": bm25_score}`. Present if `context_type` is "doc" or "all".
                * If `query` is not provided:
                    * `code_context (Optional[str])`: The full concatenated code context as a single string. Present if `context_type` is "code" or "all".
                    * `doc_context (Optional[str])`: The full concatenated documentation context as a single string. Present if `context_type` is "doc" or "all".

    * 3.7. **MCP (Model Context Protocol) Support**
        * 3.7.1. Explanation of MCP:
            * Purpose: The Model Context Protocol (MCP) is a standardized way for AI models (like Anthropic's Claude with Code Interpreter capabilities) to discover and interact with external tools and data sources. Crawl4ai's MCP server exposes its functionalities as tools that an MCP-compatible AI can use.
        * 3.7.2. Connection Endpoints (defined in `mcp_bridge.py` and attached to FastAPI app):
            * `/mcp/sse`: Server-Sent Events (SSE) endpoint for MCP communication.
            * `/mcp/ws`: WebSocket endpoint for MCP communication.
            * `/mcp/messages`: Endpoint for clients to POST messages in the SSE transport.
        * 3.7.3. Usage with Claude Code Example:
            * Command: `claude mcp add -t sse c4ai-sse http://localhost:11235/mcp/sse`
            * Purpose: This command (specific to the Claude Code CLI) registers the Crawl4ai MCP server as a tool provider named `c4ai-sse` using the SSE transport. The AI can then discover and invoke tools from this source.
        * 3.7.4. List of Available MCP Tools (defined by `@mcp_tool` decorators in `server.py`):
            * `md`: Fetches Markdown for a URL.
                * Parameters (derived from `get_markdown` function signature): `url (str)`, `filter_type (FilterType)`, `query (Optional[str])`, `cache (Optional[str])`.
            * `html`: Generates preprocessed HTML for a URL.
                * Parameters (derived from `generate_html` function signature): `url (str)`.
            * `screenshot`: Generates a screenshot of a URL.
                * Parameters (derived from `generate_screenshot` function signature): `url (str)`, `screenshot_wait_for (Optional[float])`, `output_path (Optional[str])`.
            * `pdf`: Generates a PDF of a URL.
                * Parameters (derived from `generate_pdf` function signature): `url (str)`, `output_path (Optional[str])`.
            * `execute_js`: Executes JavaScript on a URL.
                * Parameters (derived from `execute_js` function signature): `url (str)`, `scripts (List[str])`.
            * `crawl`: Performs a full crawl operation.
                * Parameters (derived from `crawl` function signature): `urls (List[str])`, `browser_config (Optional[Dict])`, `crawler_config (Optional[Dict])`.
            * `ask`: Retrieves library context.
                * Parameters (derived from `get_context` function signature): `context_type (str)`, `query (Optional[str])`, `score_ratio (float)`, `max_results (int)`.
        * 3.7.5. Testing MCP Connections:
            * Method: Use an MCP client tool (e.g., `claude mcp call c4ai-sse.md url=https://example.com`) to invoke a tool and verify the response.
        * 3.7.6. Accessing MCP Schemas:
            * Endpoint URL: `/mcp/schema`
            * Purpose: Returns a JSON response detailing all registered MCP tools, including their names, descriptions, and input schemas, enabling clients to understand how to use them.

    * 3.8. **Metrics & Monitoring Endpoints**
        * 3.8.1. `/health`
            * Purpose: Provides a basic health check for the server, indicating if it's running and responsive.
            * Response Structure (JSON from `server.py`): `{"status": "ok", "timestamp": float, "version": str}` (where version is `__version__` from `server.py`).
            * Configuration: Path configurable via `observability.health_check.endpoint` in `config.yml`.
        * 3.8.2. `/metrics`
            * Purpose: Exposes application metrics in a format compatible with Prometheus for monitoring and alerting.
            * Response Format: Prometheus text format.
            * Configuration: Enabled via `observability.prometheus.enabled: true` and endpoint path via `observability.prometheus.endpoint` in `config.yml`.

    * 3.9. **Underlying Server Logic (`server.py` - High-Level Understanding)**
        * 3.9.1. FastAPI Application:
            * Framework: The server is built using the FastAPI Python web framework for creating APIs.
        * 3.9.2. `crawler_pool` (`CrawlerPool` from `deploy.docker.crawler_pool`):
            * Role: Manages a pool of `AsyncWebCrawler` instances to reuse browser resources efficiently.
            * `get_crawler(BrowserConfig)`: Fetches an existing idle crawler compatible with the `BrowserConfig` or creates a new one if none are available or compatible.
            * `close_all()`: Iterates through all pooled crawlers and closes them.
            * `janitor()`: An `asyncio.Task` that runs periodically to close and remove crawler instances that have been idle for longer than `crawler.pool.idle_ttl_sec` (configured in `config.yml`).
        * 3.9.3. Global Page Semaphore (`GLOBAL_SEM`):
            * Type: `asyncio.Semaphore`.
            * Purpose: A global semaphore that limits the total number of concurrently open browser pages across all `AsyncWebCrawler` instances managed by the server. This acts as a hard cap to prevent excessive resource consumption.
            * Configuration: The maximum number of concurrent pages is set by `crawler.pool.max_pages` in `config.yml` (default: `30` in `server.py`, but `40` in `config.yml`). The `AsyncWebCrawler.arun` method acquires this semaphore.
        * 3.9.4. Job Router (`init_job_router` from `deploy.docker.job`):
            * Role: Manages asynchronous, long-running tasks, particularly for the `/crawl` (non-streaming batch) endpoint.
            * Mechanism: Uses Redis (configured in `config.yml`) as a backend for task queuing (storing task metadata like status, creation time, URL, result, error) and status tracking.
            * User Interaction: When a job is submitted to an endpoint using this router (e.g., `/crawl/job`), a `task_id` is returned. The client then polls an endpoint like `/task/{task_id}` to get the status and eventual result or error.
        * 3.9.5. Rate Limiting Middleware:
            * Implementation: Uses the `slowapi` library, integrated with FastAPI.
            * Purpose: To protect the server from abuse by limiting the number of requests an IP address can make within a specified time window.
            * Configuration: Settings like `enabled`, `default_limit`, `storage_uri` (e.g., `memory://` or `redis://...`) are managed in the `rate_limiting` section of `config.yml`.
        * 3.9.6. Security Middleware:
            * Implementations: `HTTPSRedirectMiddleware` and `TrustedHostMiddleware` from FastAPI, plus custom logic for adding security headers.
            * Purpose:
                * `HTTPSRedirectMiddleware`: Redirects HTTP requests to HTTPS if `security.https_redirect` is true.
                * `TrustedHostMiddleware`: Ensures requests are only served if their `Host` header matches an entry in `security.trusted_hosts`.
                * Custom header logic: Adds HTTP security headers like `X-Content-Type-Options`, `X-Frame-Options`, `Content-Security-Policy`, `Strict-Transport-Security` to all responses if `security.enabled` is true. These are defined in `security.headers` in `config.yml`.
        * 3.9.7. API Request Mapping:
            * Request Models: Pydantic models defined in `deploy/docker/schemas.py` (e.g., `CrawlRequest`, `MarkdownRequest`, `HTMLRequest`, `ScreenshotRequest`, `PDFRequest`, `JSEndpointRequest`, `TokenRequest`, `RawCode`) define the expected JSON structure for incoming API request bodies.
            * Endpoint Logic: Functions decorated with `@app.post(...)`, `@app.get(...)`, etc., in `server.py` handle incoming HTTP requests. These functions use FastAPI's dependency injection to parse and validate request bodies against the Pydantic models.
            * `AsyncWebCrawler` Interaction:
                * The parameters from the parsed request models (e.g., `CrawlRequest.urls`, `CrawlRequest.browser_config`, `CrawlRequest.crawler_config`) are used.
                * `BrowserConfig` and `CrawlerRunConfig` objects are created by calling their respective `.load()` class methods with the dictionary payloads received in the request (e.g., `BrowserConfig.load(crawl_request.browser_config)`).
                * These configuration objects are then passed to an `AsyncWebCrawler` instance obtained from the `crawler_pool`, typically to its `arun()` (for single URL or when JS execution context is critical) or `arun_many()` (for batch processing of multiple URLs) methods.
            * Result Serialization: The `CrawlResult` objects (or lists/generators of them) returned by the `AsyncWebCrawler` are usually serialized to JSON using their `.model_dump()` method before being included in the HTTP response. For streaming endpoints, each `CrawlResult` is serialized and sent as a separate NDJSON line.

## 4. Version Numbering Scheme

    * 4.1. **Standard Versioning (`MAJOR.MINOR.PATCH`)**
        * `MAJOR`: Incremented when incompatible API changes are made.
        * `MINOR`: Incremented when functionality is added in a backward-compatible manner.
        * `PATCH`: Incremented for backward-compatible bug fixes.
    * 4.2. **Pre-release Suffixes**
        * `devN`: (e.g., `0.6.0.dev1`) Development release. These are typically unstable and used for internal testing or early feedback on new, unrefined features.
        * `aN`: (e.g., `0.6.0a1`) Alpha release. Indicates an early preview of a new version, potentially unstable, and APIs might still change.
        * `bN`: (e.g., `0.6.0b1`) Beta release. Generally feature-complete for the targeted minor or major version but may still contain bugs. APIs are mostly stable at this point.
        * `rcN`: (e.g., `0.6.0rc1`) Release Candidate. A version that is potentially the final release, undergoing final testing to catch critical bugs before official release.
```