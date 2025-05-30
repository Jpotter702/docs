Okay, I will now generate the "Foundational Memory" document for the `extraction` component of `crawl4ai`, based on the outline you provided and the information I've processed from the codebase and existing documentation.

```markdown
# Detailed Outline for crawl4ai - extraction Component

**Target Document Type:** memory
**Target Output Filename Suggestion:** `llm_memory_extraction.md`
**Library Version Context:** 0.6.3
**Outline Generation Date:** 2024-05-24
---

## 1. Overview of Data Extraction in Crawl4ai

*   1.1. Purpose of the Extraction Component: The extraction component in Crawl4ai is responsible for parsing structured data from web content (HTML, text, Markdown) or PDF documents. It allows users to define how data should be identified and extracted, using various strategies ranging from rule-based (CSS, XPath, Regex) to LLM-powered approaches. Its goal is to transform raw crawled content into usable, structured information.
*   1.2. Core Concepts:
    *   1.2.1. `ExtractionStrategy`: This is an abstract base class (interface) that defines the contract for all specific extraction methods. Each strategy implements how data is extracted from the provided content.
    *   1.2.2. `ChunkingStrategy`: This is an abstract base class (interface) for strategies that preprocess content by splitting it into smaller, manageable chunks. This is particularly relevant for LLM-based extraction strategies that have token limits for their input.
    *   1.2.3. Schemas: Schemas define the structure of the data to be extracted. For non-LLM strategies like `JsonCssExtractionStrategy` or `JsonXPathExtractionStrategy`, schemas are typically dictionary-based, specifying selectors and field types. For `LLMExtractionStrategy`, schemas can be Pydantic models or JSON schema dictionaries that guide the LLM in structuring its output.
    *   1.2.4. `CrawlerRunConfig`: The `CrawlerRunConfig` object allows users to specify which `extraction_strategy` and `chunking_strategy` (if applicable) should be used for a particular crawl operation via its `arun()` method.

## 2. `ExtractionStrategy` Interface

*   2.1. Purpose: The `ExtractionStrategy` class, found in `crawl4ai.extraction_strategy`, serves as an abstract base class (ABC) defining the standard interface for all data extraction strategies within the Crawl4ai library. Implementations of this class provide specific methods for extracting structured data from content.
*   2.2. Key Abstract Methods:
    *   `extract(self, url: str, content: str, *q, **kwargs) -> List[Dict[str, Any]]`:
        *   Description: Abstract method intended to extract meaningful blocks or chunks from the given content. Subclasses must implement this.
        *   Parameters:
            *   `url (str)`: The URL of the webpage.
            *   `content (str)`: The HTML, Markdown, or text content of the webpage.
            *   `*q`: Variable positional arguments.
            *   `**kwargs`: Variable keyword arguments.
        *   Returns: `List[Dict[str, Any]]` - A list of extracted blocks or chunks, typically as dictionaries.
    *   `run(self, url: str, sections: List[str], *q, **kwargs) -> List[Dict[str, Any]]`:
        *   Description: Abstract method to process sections of text, often in parallel by default implementations in subclasses. Subclasses must implement this.
        *   Parameters:
            *   `url (str)`: The URL of the webpage.
            *   `sections (List[str])`: List of sections (strings) to process.
            *   `*q`: Variable positional arguments.
            *   `**kwargs`: Variable keyword arguments.
        *   Returns: `List[Dict[str, Any]]` - A list of processed JSON blocks.
*   2.3. Input Format Property:
    *   `input_format (str)`: [Read-only] - An attribute indicating the expected input format for the content to be processed by the strategy (e.g., "markdown", "html", "fit_html", "text"). Default is "markdown".

## 3. Non-LLM Based Extraction Strategies

*   ### 3.1. Class `NoExtractionStrategy`
    *   3.1.1. Purpose: A baseline `ExtractionStrategy` that performs no actual data extraction. It returns the input content as is, typically useful for scenarios where only raw or cleaned HTML/Markdown is needed without further structuring.
    *   3.1.2. Inheritance: `ExtractionStrategy`
    *   3.1.3. Initialization (`__init__`):
        *   3.1.3.1. Signature: `NoExtractionStrategy(**kwargs)`
        *   3.1.3.2. Parameters:
            *   `**kwargs`: Passed to the base `ExtractionStrategy` initializer.
    *   3.1.4. Key Public Methods:
        *   `extract(self, url: str, html: str, *q, **kwargs) -> List[Dict[str, Any]]`:
            *   Description: Returns the provided `html` content wrapped in a list containing a single dictionary: `[{"index": 0, "content": html}]`.
        *   `run(self, url: str, sections: List[str], *q, **kwargs) -> List[Dict[str, Any]]`:
            *   Description: Returns a list where each input section is wrapped in a dictionary: `[{"index": i, "tags": [], "content": section} for i, section in enumerate(sections)]`.

*   ### 3.2. Class `JsonCssExtractionStrategy`
    *   3.2.1. Purpose: Extracts structured data from HTML content using a JSON schema that defines CSS selectors to locate and extract data for specified fields. It uses BeautifulSoup4 for parsing and selection.
    *   3.2.2. Inheritance: `JsonElementExtractionStrategy` (which inherits from `ExtractionStrategy`)
    *   3.2.3. Initialization (`__init__`):
        *   3.2.3.1. Signature: `JsonCssExtractionStrategy(schema: Dict[str, Any], **kwargs)`
        *   3.2.3.2. Parameters:
            *   `schema (Dict[str, Any])`: The JSON schema defining extraction rules.
            *   `**kwargs`: Passed to the base class initializer. Includes `input_format` (default: "html").
    *   3.2.4. Schema Definition for `JsonCssExtractionStrategy`:
        *   3.2.4.1. `name (str)`: A descriptive name for the schema (e.g., "ProductDetails").
        *   3.2.4.2. `baseSelector (str)`: The primary CSS selector that identifies each root element representing an item to be extracted (e.g., "div.product-item").
        *   3.2.4.3. `fields (List[Dict[str, Any]])`: A list of dictionaries, each defining a field to be extracted from within each `baseSelector` element.
            *   Each field dictionary:
                *   `name (str)`: The key for this field in the output JSON object.
                *   `selector (str)`: The CSS selector for this field, relative to its parent element (either the `baseSelector` or a parent "nested" field).
                *   `type (str)`: Specifies how to extract the data. Common values:
                    *   `"text"`: Extracts the text content of the selected element.
                    *   `"attribute"`: Extracts the value of a specified HTML attribute.
                    *   `"html"`: Extracts the raw inner HTML of the selected element.
                    *   `"list"`: Extracts a list of items. The `fields` sub-key then defines the structure of each item in the list (if objects) or the `selector` directly targets list elements for primitive values.
                    *   `"nested"`: Extracts a nested JSON object. The `fields` sub-key defines the structure of this nested object.
                *   `attribute (str, Optional)`: Required if `type` is "attribute". Specifies the name of the HTML attribute to extract (e.g., "href", "src").
                *   `fields (List[Dict[str, Any]], Optional)`: Required if `type` is "list" (for a list of objects) or "nested". Defines the structure of the nested object or list items.
                *   `transform (str, Optional)`: A string indicating a transformation to apply to the extracted value (e.g., "lowercase", "uppercase", "strip").
                *   `default (Any, Optional)`: A default value to use if the selector does not find an element or the attribute is missing.
    *   3.2.5. Key Public Methods:
        *   `extract(self, url: str, html_content: str, *q, **kwargs) -> List[Dict[str, Any]]`:
            *   Description: Parses the `html_content` and applies the defined schema to extract structured data using CSS selectors.
    *   3.2.6. Features:
        *   3.2.6.1. Nested Extraction: Supports extracting complex, nested JSON objects by defining "nested" type fields within the schema.
        *   3.2.6.2. List Handling: Supports extracting lists of primitive values (e.g., list of strings from multiple `<li>` tags) or lists of structured objects (e.g., a list of product details, each with its own fields).

*   ### 3.3. Class `JsonXPathExtractionStrategy`
    *   3.3.1. Purpose: Extracts structured data from HTML/XML content using a JSON schema that defines XPath expressions to locate and extract data. It uses `lxml` for parsing and XPath evaluation.
    *   3.3.2. Inheritance: `JsonElementExtractionStrategy` (which inherits from `ExtractionStrategy`)
    *   3.3.3. Initialization (`__init__`):
        *   3.3.3.1. Signature: `JsonXPathExtractionStrategy(schema: Dict[str, Any], **kwargs)`
        *   3.3.3.2. Parameters:
            *   `schema (Dict[str, Any])`: The JSON schema defining extraction rules, where selectors are XPath expressions.
            *   `**kwargs`: Passed to the base class initializer. Includes `input_format` (default: "html").
    *   3.3.4. Schema Definition: The schema structure is identical to `JsonCssExtractionStrategy` (see 3.2.4), but the `baseSelector` and field `selector` values must be valid XPath expressions.
    *   3.3.5. Key Public Methods:
        *   `extract(self, url: str, html_content: str, *q, **kwargs) -> List[Dict[str, Any]]`:
            *   Description: Parses the `html_content` using `lxml` and applies the defined schema to extract structured data using XPath expressions.

*   ### 3.4. Class `JsonLxmlExtractionStrategy`
    *   3.4.1. Purpose: Provides an alternative CSS selector-based extraction strategy leveraging the `lxml` library for parsing and selection, which can offer performance benefits over BeautifulSoup4 in some cases.
    *   3.4.2. Inheritance: `JsonCssExtractionStrategy` (and thus `JsonElementExtractionStrategy`, `ExtractionStrategy`)
    *   3.4.3. Initialization (`__init__`):
        *   3.4.3.1. Signature: `JsonLxmlExtractionStrategy(schema: Dict[str, Any], **kwargs)`
        *   3.4.3.2. Parameters:
            *   `schema (Dict[str, Any])`: The JSON schema defining extraction rules, using CSS selectors.
            *   `**kwargs`: Passed to the base class initializer. Includes `input_format` (default: "html").
    *   3.4.4. Schema Definition: Identical to `JsonCssExtractionStrategy` (see 3.2.4).
    *   3.4.5. Key Public Methods:
        *   `extract(self, url: str, html_content: str, *q, **kwargs) -> List[Dict[str, Any]]`:
            *   Description: Parses the `html_content` using `lxml` and applies the defined schema to extract structured data using lxml's CSS selector capabilities (which often translates CSS to XPath internally).

*   ### 3.5. Class `RegexExtractionStrategy`
    *   3.5.1. Purpose: Extracts data from text content (HTML, Markdown, or plain text) using a collection of regular expression patterns. Each match is returned as a structured dictionary.
    *   3.5.2. Inheritance: `ExtractionStrategy`
    *   3.5.3. Initialization (`__init__`):
        *   3.5.3.1. Signature: `RegexExtractionStrategy(patterns: Union[Dict[str, str], List[Tuple[str, str]], "RegexExtractionStrategy._B"] = _B.NOTHING, input_format: str = "fit_html", **kwargs)`
        *   3.5.3.2. Parameters:
            *   `patterns (Union[Dict[str, str], List[Tuple[str, str]], "_B"], default: _B.NOTHING)`:
                *   Description: Defines the regex patterns to use.
                *   Can be a dictionary mapping labels to regex strings (e.g., `{"email": r"..."}`).
                *   Can be a list of (label, regex_string) tuples.
                *   Can be a bitwise OR combination of `RegexExtractionStrategy._B` enum members for using built-in patterns (e.g., `RegexExtractionStrategy.Email | RegexExtractionStrategy.Url`).
            *   `input_format (str, default: "fit_html")`: Specifies the input format for the content. Options: "html" (raw HTML), "markdown" (Markdown from HTML), "text" (plain text from HTML), "fit_html" (content filtered for relevance before regex application).
            *   `**kwargs`: Passed to the base `ExtractionStrategy`.
    *   3.5.4. Built-in Patterns (`RegexExtractionStrategy._B` Enum - an `IntFlag`):
        *   `EMAIL (auto())`: Matches email addresses. Example pattern: `r"[\\w.+-]+@[\\w-]+\\.[\\w.-]+"`
        *   `PHONE_INTL (auto())`: Matches international phone numbers. Example pattern: `r"\\+?\\d[\\d .()-]{7,}\\d"`
        *   `PHONE_US (auto())`: Matches US phone numbers. Example pattern: `r"\\(?\\d{3}\\)?[-. ]?\\d{3}[-. ]?\\d{4}"`
        *   `URL (auto())`: Matches URLs. Example pattern: `r"https?://[^\\s\\'\"<>]+"`
        *   `IPV4 (auto())`: Matches IPv4 addresses. Example pattern: `r"(?:\\d{1,3}\\.){3}\\d{1,3}"`
        *   `IPV6 (auto())`: Matches IPv6 addresses. Example pattern: `r"[A-F0-9]{1,4}(?::[A-F0-9]{1,4}){7}"`
        *   `UUID (auto())`: Matches UUIDs. Example pattern: `r"[0-9a-f]{8}-[0-9a-f]{4}-[1-5][0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}"`
        *   `CURRENCY (auto())`: Matches currency amounts. Example pattern: `r"(?:USD|EUR|RM|\\$|â‚¬|Â¥|Â£)\\s?\\d+(?:[.,]\\d{2})?"`
        *   `PERCENTAGE (auto())`: Matches percentages. Example pattern: `r"\\d+(?:\\.\\d+)?%"`
        *   `NUMBER (auto())`: Matches numbers (integers, decimals). Example pattern: `r"\\b\\d{1,3}(?:[,.]?\\d{3})*(?:\\.\\d+)?\\b"`
        *   `DATE_ISO (auto())`: Matches ISO 8601 dates (YYYY-MM-DD). Example pattern: `r"\\d{4}-\\d{2}-\\d{2}"`
        *   `DATE_US (auto())`: Matches US-style dates (MM/DD/YYYY or MM/DD/YY). Example pattern: `r"\\d{1,2}/\\d{1,2}/\\d{2,4}"`
        *   `TIME_24H (auto())`: Matches 24-hour time formats (HH:MM or HH:MM:SS). Example pattern: `r"\\b(?:[01]?\\d|2[0-3]):[0-5]\\d(?:[:.][0-5]\\d)?\\b"`
        *   `POSTAL_US (auto())`: Matches US postal codes (ZIP codes). Example pattern: `r"\\b\\d{5}(?:-\\d{4})?\\b"`
        *   `POSTAL_UK (auto())`: Matches UK postal codes. Example pattern: `r"\\b[A-Z]{1,2}\\d[A-Z\\d]? ?\\d[A-Z]{2}\\b"`
        *   `HTML_COLOR_HEX (auto())`: Matches HTML hex color codes. Example pattern: `r"#[0-9A-Fa-f]{6}\\b"`
        *   `TWITTER_HANDLE (auto())`: Matches Twitter handles. Example pattern: `r"@[\\w]{1,15}"`
        *   `HASHTAG (auto())`: Matches hashtags. Example pattern: `r"#[\\w-]+"`
        *   `MAC_ADDR (auto())`: Matches MAC addresses. Example pattern: `r"(?:[0-9A-Fa-f]{2}[:-]){5}[0-9A-Fa-f]{2}"`
        *   `IBAN (auto())`: Matches IBANs. Example pattern: `r"[A-Z]{2}\\d{2}[A-Z0-9]{11,30}"`
        *   `CREDIT_CARD (auto())`: Matches common credit card numbers. Example pattern: `r"\\b(?:4\\d{12}(?:\\d{3})?|5[1-5]\\d{14}|3[47]\\d{13}|6(?:011|5\\d{2})\\d{12})\\b"`
        *   `ALL (_B(-1).value & ~_B.NOTHING.value)`: Includes all built-in patterns except `NOTHING`.
        *   `NOTHING (_B(0).value)`: Includes no built-in patterns.
    *   3.5.5. Key Public Methods:
        *   `extract(self, url: str, content: str, **kwargs) -> List[Dict[str, Any]]`:
            *   Description: Applies all configured regex patterns (built-in and custom) to the input `content`.
            *   Returns: `List[Dict[str, Any]]` - A list of dictionaries, where each dictionary represents a match and contains:
                *   `"url" (str)`: The source URL.
                *   `"label" (str)`: The label of the matching regex pattern.
                *   `"value" (str)`: The actual matched string.
                *   `"span" (Tuple[int, int])`: The start and end indices of the match within the content.
    *   3.5.6. Static Method: `generate_pattern`
        *   3.5.6.1. Signature: `staticmethod generate_pattern(label: str, html: str, query: Optional[str] = None, examples: Optional[List[str]] = None, llm_config: Optional[LLMConfig] = None, **kwargs) -> Dict[str, str]`
        *   3.5.6.2. Purpose: Uses an LLM to automatically generate a Python-compatible regular expression pattern for a given label, based on sample HTML content, an optional natural language query describing the target, and/or examples of desired matches.
        *   3.5.6.3. Parameters:
            *   `label (str)`: A descriptive label for the pattern to be generated (e.g., "product_price", "article_date").
            *   `html (str)`: The HTML content from which the pattern should be inferred.
            *   `query (Optional[str], default: None)`: A natural language description of what kind of data the regex should capture (e.g., "Extract the publication date", "Find all ISBN numbers").
            *   `examples (Optional[List[str]], default: None)`: A list of example strings that the generated regex should successfully match from the provided HTML.
            *   `llm_config (Optional[LLMConfig], default: None)`: Configuration for the LLM to be used. If `None`, uses default `LLMConfig`.
            *   `**kwargs`: Additional arguments passed to the LLM completion request (e.g., `temperature`, `max_tokens`).
        *   3.5.6.4. Returns: `Dict[str, str]` - A dictionary containing the generated pattern, in the format `{label: "regex_pattern_string"}`.

## 4. LLM-Based Extraction Strategies

*   ### 4.1. Class `LLMExtractionStrategy`
    *   4.1.1. Purpose: Employs Large Language Models (LLMs) to extract either structured data according to a schema or relevant blocks of text based on natural language instructions from various content formats (HTML, Markdown, text).
    *   4.1.2. Inheritance: `ExtractionStrategy`
    *   4.1.3. Initialization (`__init__`):
        *   4.1.3.1. Signature: `LLMExtractionStrategy(llm_config: Optional[LLMConfig] = None, instruction: Optional[str] = None, schema: Optional[Union[Dict[str, Any], "BaseModel"]] = None, extraction_type: str = "block", chunk_token_threshold: int = CHUNK_TOKEN_THRESHOLD, overlap_rate: float = OVERLAP_RATE, word_token_rate: float = WORD_TOKEN_RATE, apply_chunking: bool = True, force_json_response: bool = False, **kwargs)`
        *   4.1.3.2. Parameters:
            *   `llm_config (Optional[LLMConfig], default: None)`: Configuration for the LLM. If `None`, a default `LLMConfig` is created.
            *   `instruction (Optional[str], default: None)`: Natural language instructions to guide the LLM's extraction process (e.g., "Extract the main article content", "Summarize the key points").
            *   `schema (Optional[Union[Dict[str, Any], "BaseModel"]], default: None)`: A Pydantic model class or a dictionary representing a JSON schema. Used when `extraction_type` is "schema" to define the desired output structure.
            *   `extraction_type (str, default: "block")`: Determines the extraction mode.
                *   `"block"`: LLM identifies and extracts relevant blocks/chunks of text based on the `instruction`.
                *   `"schema"`: LLM attempts to populate the fields defined in `schema` from the content.
            *   `chunk_token_threshold (int, default: CHUNK_TOKEN_THRESHOLD)`: The target maximum number of tokens for each chunk of content sent to the LLM. `CHUNK_TOKEN_THRESHOLD` is defined in `crawl4ai.config` (default value: 10000).
            *   `overlap_rate (float, default: OVERLAP_RATE)`: The percentage of overlap between consecutive chunks to ensure context continuity. `OVERLAP_RATE` is defined in `crawl4ai.config` (default value: 0.1, i.e., 10%).
            *   `word_token_rate (float, default: WORD_TOKEN_RATE)`: An estimated ratio of words to tokens (e.g., 0.75 words per token). Used for approximating chunk boundaries. `WORD_TOKEN_RATE` is defined in `crawl4ai.config` (default value: 0.75).
            *   `apply_chunking (bool, default: True)`: If `True`, the input content is chunked before being sent to the LLM. If `False`, the entire content is sent (which might exceed token limits for large inputs).
            *   `force_json_response (bool, default: False)`: If `True` and `extraction_type` is "schema", instructs the LLM to strictly adhere to JSON output format.
            *   `**kwargs`: Passed to `ExtractionStrategy` and potentially to the underlying LLM API calls (e.g., `temperature`, `max_tokens` if not set in `llm_config`).
    *   4.1.4. Key Public Methods:
        *   `extract(self, url: str, content: str, *q, **kwargs) -> List[Dict[str, Any]]`:
            *   Description: Processes the input `content`. If `apply_chunking` is `True`, it first chunks the content using the specified `chunking_strategy` (or a default one if `LLMExtractionStrategy` manages it internally). Then, for each chunk (or the whole content if not chunked), it constructs a prompt based on `instruction` and/or `schema` and sends it to the configured LLM.
            *   Returns: `List[Dict[str, Any]]` - A list of dictionaries.
                *   If `extraction_type` is "block", each dictionary typically contains `{"index": int, "content": str, "tags": List[str]}`.
                *   If `extraction_type` is "schema", each dictionary is an instance of the extracted structured data, ideally conforming to the provided `schema`. If the LLM returns multiple JSON objects in a list, they are parsed and returned.
        *   `run(self, url: str, sections: List[str], *q, **kwargs) -> List[Dict[str, Any]]`:
            *   Description: Processes a list of content `sections` in parallel (using `ThreadPoolExecutor`). Each section is passed to the `extract` method logic.
            *   Returns: `List[Dict[str, Any]]` - Aggregated list of results from processing all sections.
    *   4.1.5. `TokenUsage` Tracking:
        *   `total_usage (TokenUsage)`: [Read-only Public Attribute] - An instance of `TokenUsage` that accumulates the token counts (prompt, completion, total) from all LLM API calls made by this `LLMExtractionStrategy` instance.
        *   `usages (List[TokenUsage])`: [Read-only Public Attribute] - A list containing individual `TokenUsage` objects for each separate LLM API call made during the extraction process. This allows for detailed tracking of token consumption per call.

## 5. `ChunkingStrategy` Interface and Implementations

*   ### 5.1. Interface `ChunkingStrategy`
    *   5.1.1. Purpose: The `ChunkingStrategy` class, found in `crawl4ai.chunking_strategy`, is an abstract base class (ABC) that defines the interface for different content chunking algorithms. Chunking is used to break down large pieces of text or HTML into smaller, manageable segments, often before feeding them to an LLM or other processing steps.
    *   5.1.2. Key Abstract Methods:
        *   `chunk(self, content: str) -> List[str]`:
            *   Description: Abstract method that must be implemented by subclasses to split the input `content` string into a list of string chunks.
            *   Parameters:
                *   `content (str)`: The content to be chunked.
            *   Returns: `List[str]` - A list of content chunks.

*   ### 5.2. Class `RegexChunking`
    *   5.2.1. Purpose: Implements `ChunkingStrategy` by splitting content based on a list of regular expression patterns. It can also attempt to merge smaller chunks to meet a target `chunk_size`.
    *   5.2.2. Inheritance: `ChunkingStrategy`
    *   5.2.3. Initialization (`__init__`):
        *   5.2.3.1. Signature: `RegexChunking(patterns: Optional[List[str]] = None, chunk_size: Optional[int] = None, overlap: Optional[int] = None, word_token_ratio: Optional[float] = WORD_TOKEN_RATE, **kwargs)`
        *   5.2.3.2. Parameters:
            *   `patterns (Optional[List[str]], default: None)`: A list of regex patterns used to split the text. If `None`, defaults to paragraph-based splitting (`["\\n\\n+"]`).
            *   `chunk_size (Optional[int], default: None)`: The target token size for each chunk. If specified, the strategy will try to merge smaller chunks created by regex splitting to approximate this size.
            *   `overlap (Optional[int], default: None)`: The target token overlap between consecutive chunks when `chunk_size` is active.
            *   `word_token_ratio (Optional[float], default: WORD_TOKEN_RATE)`: The estimated ratio of words to tokens, used if `chunk_size` or `overlap` are specified. `WORD_TOKEN_RATE` is defined in `crawl4ai.config` (default value: 0.75).
            *   `**kwargs`: Additional keyword arguments.
    *   5.2.4. Key Public Methods:
        *   `chunk(self, content: str) -> List[str]`:
            *   Description: Splits the input `content` using the configured regex patterns. If `chunk_size` is set, it then merges these initial chunks to meet the target size with the specified overlap.

*   ### 5.3. Class `IdentityChunking`
    *   5.3.1. Purpose: A `ChunkingStrategy` that does not perform any actual chunking. It returns the input content as a single chunk in a list.
    *   5.3.2. Inheritance: `ChunkingStrategy`
    *   5.3.3. Initialization (`__init__`):
        *   5.3.3.1. Signature: `IdentityChunking(**kwargs)`
        *   5.3.3.2. Parameters:
            *   `**kwargs`: Additional keyword arguments.
    *   5.3.4. Key Public Methods:
        *   `chunk(self, content: str) -> List[str]`:
            *   Description: Returns the input `content` as a single-element list: `[content]`.

## 6. Defining Schemas for Extraction

*   6.1. Purpose: Schemas provide a structured way to define what data needs to be extracted from content and how it should be organized. This allows for consistent and predictable output from the extraction process.
*   6.2. Schemas for CSS/XPath/LXML-based Extraction (`JsonCssExtractionStrategy`, etc.):
    *   6.2.1. Format: These strategies use a dictionary-based JSON-like schema.
    *   6.2.2. Key elements: As detailed in section 3.2.4 for `JsonCssExtractionStrategy`:
        *   `name (str)`: Name of the schema.
        *   `baseSelector (str)`: CSS selector (for CSS strategies) or XPath expression (for XPath strategy) identifying the repeating parent elements.
        *   `fields (List[Dict[str, Any]])`: A list defining each field to extract. Each field definition includes:
            *   `name (str)`: Output key for the field.
            *   `selector (str)`: CSS/XPath selector relative to the `baseSelector` or parent "nested" element.
            *   `type (str)`: "text", "attribute", "html", "list", "nested".
            *   `attribute (str, Optional)`: Name of HTML attribute (if type is "attribute").
            *   `fields (List[Dict], Optional)`: For "list" (of objects) or "nested" types.
            *   `transform (str, Optional)`: e.g., "lowercase".
            *   `default (Any, Optional)`: Default value if not found.
*   6.3. Schemas for LLM-based Extraction (`LLMExtractionStrategy`):
    *   6.3.1. Format: `LLMExtractionStrategy` accepts schemas in two main formats when `extraction_type="schema"`:
        *   Pydantic models: The Pydantic model class itself.
        *   Dictionary: A Python dictionary representing a valid JSON schema.
    *   6.3.2. Pydantic Models:
        *   Definition: Users can define a Pydantic `BaseModel` where each field represents a piece of data to be extracted. Field types and descriptions are automatically inferred.
        *   Conversion: `LLMExtractionStrategy` internally converts the Pydantic model to its JSON schema representation (`model_json_schema()`) to guide the LLM.
    *   6.3.3. Dictionary-based JSON Schema:
        *   Structure: Users can provide a dictionary that conforms to the JSON Schema specification. This typically includes a `type: "object"` at the root and a `properties` dictionary defining each field, its type (e.g., "string", "number", "array", "object"), and optionally a `description`.
        *   Usage: This schema is passed to the LLM to instruct it on the desired output format.

## 7. Configuration with `CrawlerRunConfig`

*   7.1. Purpose: The `CrawlerRunConfig` class (from `crawl4ai.async_configs`) is used to configure the behavior of a specific `arun()` or `arun_many()` call on an `AsyncWebCrawler` instance. It allows specifying various runtime parameters, including the extraction and chunking strategies.
*   7.2. Key Attributes:
    *   `extraction_strategy (Optional[ExtractionStrategy], default: None)`:
        *   Purpose: Specifies the `ExtractionStrategy` instance to be used for processing the content obtained during the crawl. If `None`, no structured extraction beyond basic Markdown generation occurs (unless a default is applied by the crawler).
        *   Type: An instance of a class inheriting from `ExtractionStrategy`.
    *   `chunking_strategy (Optional[ChunkingStrategy], default: RegexChunking())`:
        *   Purpose: Specifies the `ChunkingStrategy` instance to be used for breaking down content into smaller pieces before it's passed to an `ExtractionStrategy` (particularly `LLMExtractionStrategy`).
        *   Type: An instance of a class inheriting from `ChunkingStrategy`.
        *   Default: An instance of `RegexChunking()` with its default parameters (paragraph-based splitting).

## 8. LLM-Specific Configuration and Models

*   ### 8.1. Class `LLMConfig`
    *   8.1.1. Purpose: The `LLMConfig` class (from `crawl4ai.async_configs`) centralizes configuration parameters for interacting with Large Language Models (LLMs) through various providers.
    *   8.1.2. Initialization (`__init__`):
        *   8.1.2.1. Signature:
            ```python
            class LLMConfig:
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
                ): ...
            ```
        *   8.1.2.2. Parameters:
            *   `provider (str, default: DEFAULT_PROVIDER)`: Specifies the LLM provider and model, e.g., "openai/gpt-4o-mini", "ollama/llama3.3". `DEFAULT_PROVIDER` is "openai/gpt-4o-mini".
            *   `api_token (Optional[str], default: None)`: API token for the LLM provider. If `None`, the system attempts to read it from environment variables (e.g., `OPENAI_API_KEY`, `GEMINI_API_KEY`, `GROQ_API_KEY` based on provider). Can also be prefixed with "env:" (e.g., "env:MY_CUSTOM_LLM_KEY").
            *   `base_url (Optional[str], default: None)`: Custom base URL for the LLM API endpoint, for self-hosted or alternative provider endpoints.
            *   `temperature (Optional[float], default: None)`: Controls randomness in LLM generation. Higher values (e.g., 0.8) make output more random, lower (e.g., 0.2) more deterministic.
            *   `max_tokens (Optional[int], default: None)`: Maximum number of tokens the LLM should generate in its response.
            *   `top_p (Optional[float], default: None)`: Nucleus sampling parameter. An alternative to temperature; controls the cumulative probability mass of tokens considered for generation.
            *   `frequency_penalty (Optional[float], default: None)`: Penalizes new tokens based on their existing frequency in the text so far, decreasing repetition.
            *   `presence_penalty (Optional[float], default: None)`: Penalizes new tokens based on whether they have appeared in the text so far, encouraging new topics.
            *   `stop (Optional[List[str]], default: None)`: A list of sequences where the API will stop generating further tokens.
            *   `n (Optional[int], default: None)`: Number of completions to generate for each prompt.
    *   8.1.3. Helper Methods:
        *   `from_kwargs(kwargs: dict) -> LLMConfig`:
            *   Description: [Static method] Creates an `LLMConfig` instance from a dictionary of keyword arguments.
        *   `to_dict() -> dict`:
            *   Description: Converts the `LLMConfig` instance into a dictionary representation.
        *   `clone(**kwargs) -> LLMConfig`:
            *   Description: Creates a new `LLMConfig` instance as a copy of the current one, allowing specific attributes to be overridden with `kwargs`.

*   ### 8.2. Dataclass `TokenUsage`
    *   8.2.1. Purpose: The `TokenUsage` dataclass (from `crawl4ai.models`) is used to store information about the number of tokens consumed during an LLM API call.
    *   8.2.2. Fields:
        *   `completion_tokens (int, default: 0)`: The number of tokens generated by the LLM in the completion.
        *   `prompt_tokens (int, default: 0)`: The number of tokens in the prompt sent to the LLM.
        *   `total_tokens (int, default: 0)`: The sum of `completion_tokens` and `prompt_tokens`.
        *   `completion_tokens_details (Optional[dict], default: None)`: Provider-specific detailed breakdown of completion tokens, if available.
        *   `prompt_tokens_details (Optional[dict], default: None)`: Provider-specific detailed breakdown of prompt tokens, if available.

## 9. PDF Processing and Extraction

*   ### 9.1. Overview of PDF Processing
    *   9.1.1. Purpose: Crawl4ai provides specialized strategies to handle PDF documents, enabling the fetching of PDF content and subsequent extraction of text, images, and metadata. This allows PDFs to be treated as a primary content source similar to HTML web pages.
    *   9.1.2. Key Components:
        *   `PDFCrawlerStrategy`: For fetching/identifying PDF content.
        *   `PDFContentScrapingStrategy`: For processing PDF content using an underlying PDF processor.
        *   `NaivePDFProcessorStrategy`: The default logic for parsing PDF files.

*   ### 9.2. Class `PDFCrawlerStrategy`
    *   9.2.1. Purpose: An implementation of `AsyncCrawlerStrategy` specifically for handling PDF documents. It doesn't perform typical browser interactions but focuses on fetching PDF content and setting the appropriate response headers to indicate a PDF document, which then allows `PDFContentScrapingStrategy` to process it.
    *   9.2.2. Inheritance: `AsyncCrawlerStrategy` (from `crawl4ai.async_crawler_strategy`)
    *   9.2.3. Initialization (`__init__`):
        *   9.2.3.1. Signature: `PDFCrawlerStrategy(logger: Optional[AsyncLogger] = None)`
        *   9.2.3.2. Parameters:
            *   `logger (Optional[AsyncLogger], default: None)`: An optional logger instance for logging messages.
    *   9.2.4. Key Public Methods:
        *   `crawl(self, url: str, **kwargs) -> AsyncCrawlResponse`:
            *   Description: Fetches the content from the given `url`. If the content is identified as a PDF (either by URL extension or `Content-Type` header for remote URLs), it sets `response_headers={"Content-Type": "application/pdf"}` in the returned `AsyncCrawlResponse`. The `html` field of the response will contain a placeholder message as the actual PDF processing happens in the scraping strategy.
        *   `close(self) -> None`:
            *   Description: Placeholder for cleanup, typically does nothing in this strategy.
        *   `__aenter__(self) -> "PDFCrawlerStrategy"`:
            *   Description: Async context manager entry point.
        *   `__aexit__(self, exc_type, exc_val, exc_tb) -> None`:
            *   Description: Async context manager exit point, calls `close()`.

*   ### 9.3. Class `PDFContentScrapingStrategy`
    *   9.3.1. Purpose: An implementation of `ContentScrapingStrategy` designed to process PDF documents. It uses an underlying `PDFProcessorStrategy` (by default, `NaivePDFProcessorStrategy`) to extract text, images, and metadata from the PDF, then formats this information into a `ScrapingResult`.
    *   9.3.2. Inheritance: `ContentScrapingStrategy` (from `crawl4ai.content_scraping_strategy`)
    *   9.3.3. Initialization (`__init__`):
        *   9.3.3.1. Signature: `PDFContentScrapingStrategy(save_images_locally: bool = False, extract_images: bool = False, image_save_dir: Optional[str] = None, batch_size: int = 4, logger: Optional[AsyncLogger] = None)`
        *   9.3.3.2. Parameters:
            *   `save_images_locally (bool, default: False)`: If `True`, extracted images will be saved to the local filesystem.
            *   `extract_images (bool, default: False)`: If `True`, the strategy will attempt to extract images from the PDF.
            *   `image_save_dir (Optional[str], default: None)`: The directory where extracted images will be saved if `save_images_locally` is `True`. If `None`, a default or temporary directory might be used.
            *   `batch_size (int, default: 4)`: The number of PDF pages to process in parallel by the underlying `NaivePDFProcessorStrategy`.
            *   `logger (Optional[AsyncLogger], default: None)`: An optional logger instance.
    *   9.3.4. Key Attributes:
        *   `pdf_processor (NaivePDFProcessorStrategy)`: An instance of `NaivePDFProcessorStrategy` configured with the provided image and batch settings, used to do the actual PDF parsing.
    *   9.3.5. Key Public Methods:
        *   `scrape(self, url: str, html: str, **params) -> ScrapingResult`:
            *   Description: Takes a `url` (which can be a local file path or a remote HTTP/HTTPS URL pointing to a PDF) and processes it. The `html` parameter is typically a placeholder like "Scraper will handle the real work" as the content comes from the PDF file itself. It downloads remote PDFs to a temporary local file before processing.
            *   Returns: `ScrapingResult` containing the extracted PDF data, including `cleaned_html` (concatenated HTML of pages), `media` (extracted images), `links`, and `metadata`.
        *   `ascrape(self, url: str, html: str, **kwargs) -> ScrapingResult`:
            *   Description: Asynchronous version of `scrape`. Internally calls `scrape` using `asyncio.to_thread`.
    *   9.3.6. Internal Methods (Conceptual):
        *   `_get_pdf_path(self, url: str) -> str`:
            *   Description: If `url` is an HTTP/HTTPS URL, downloads the PDF to a temporary file and returns its path. If `url` starts with "file://", it strips the prefix and returns the local path. Otherwise, assumes `url` is already a local path. Handles download timeouts and errors.

*   ### 9.4. Class `NaivePDFProcessorStrategy`
    *   9.4.1. Purpose: The default implementation of `PDFProcessorStrategy` in Crawl4ai. It uses the PyPDF2 library (and Pillow for image processing) to parse PDF files, extract text content page by page, attempt to extract embedded images, and gather document metadata.
    *   9.4.2. Inheritance: `PDFProcessorStrategy` (from `crawl4ai.processors.pdf.processor`)
    *   9.4.3. Dependencies: Requires `PyPDF2` and `Pillow`. These are installed with the `crawl4ai[pdf]` extra.
    *   9.4.4. Initialization (`__init__`):
        *   9.4.4.1. Signature: `NaivePDFProcessorStrategy(image_dpi: int = 144, image_quality: int = 85, extract_images: bool = True, save_images_locally: bool = False, image_save_dir: Optional[Path] = None, batch_size: int = 4)`
        *   9.4.4.2. Parameters:
            *   `image_dpi (int, default: 144)`: DPI used when rendering PDF pages to images (if direct image extraction is not possible or disabled).
            *   `image_quality (int, default: 85)`: Quality setting (1-100) for images saved in lossy formats like JPEG.
            *   `extract_images (bool, default: True)`: If `True`, attempts to extract embedded images directly from the PDF's XObjects.
            *   `save_images_locally (bool, default: False)`: If `True`, extracted images are saved to disk. Otherwise, they are base64 encoded and returned in the `PDFPage.images` data.
            *   `image_save_dir (Optional[Path], default: None)`: If `save_images_locally` is True, this specifies the directory to save images. If `None`, a temporary directory (prefixed `pdf_images_`) is created and used.
            *   `batch_size (int, default: 4)`: The number of pages to process in parallel when using the `process_batch` method.
    *   9.4.5. Key Public Methods:
        *   `process(self, pdf_path: Path) -> PDFProcessResult`:
            *   Description: Processes the PDF specified by `pdf_path` page by page sequentially.
            *   Returns: `PDFProcessResult` containing metadata and a list of `PDFPage` objects.
        *   `process_batch(self, pdf_path: Path) -> PDFProcessResult`:
            *   Description: Processes the PDF specified by `pdf_path` by handling pages in parallel batches using a `ThreadPoolExecutor` with `max_workers` set to `batch_size`.
            *   Returns: `PDFProcessResult` containing metadata and a list of `PDFPage` objects, assembled in the correct page order.
    *   9.4.6. Internal Methods (Conceptual High-Level):
        *   `_process_page(self, page: PyPDF2PageObject, image_dir: Optional[Path]) -> PDFPage`: Extracts text, images (if `extract_images` is True), and links from a single PyPDF2 page object.
        *   `_extract_images(self, page: PyPDF2PageObject, image_dir: Optional[Path]) -> List[Dict]`: Iterates through XObjects on a page, identifies images, decodes them (handling FlateDecode, DCTDecode, CCITTFaxDecode, JPXDecode), and either saves them locally or base64 encodes them.
        *   `_extract_links(self, page: PyPDF2PageObject) -> List[str]`: Extracts URI actions from page annotations to get hyperlinks.
        *   `_extract_metadata(self, pdf_path: Path, reader: PyPDF2PdfReader) -> PDFMetadata`: Reads metadata from the PDF document information dictionary (e.g., /Title, /Author, /CreationDate).

*   ### 9.5. Data Models for PDF Processing
    *   9.5.1. Dataclass `PDFMetadata` (from `crawl4ai.processors.pdf.processor`)
        *   Fields:
            *   `title (Optional[str], default: None)`
            *   `author (Optional[str], default: None)`
            *   `producer (Optional[str], default: None)`
            *   `created (Optional[datetime], default: None)`
            *   `modified (Optional[datetime], default: None)`
            *   `pages (int, default: 0)`
            *   `encrypted (bool, default: False)`
            *   `file_size (Optional[int], default: None)`
    *   9.5.2. Dataclass `PDFPage` (from `crawl4ai.processors.pdf.processor`)
        *   Fields:
            *   `page_number (int)`
            *   `raw_text (str, default: "")`
            *   `markdown (str, default: "")`: Markdown representation of the page's text content, processed by `clean_pdf_text`.
            *   `html (str, default: "")`: HTML representation of the page's text content, processed by `clean_pdf_text_to_html`.
            *   `images (List[Dict], default_factory: list)`: List of image dictionaries. Each dictionary contains:
                *   `format (str)`: e.g., "png", "jpeg", "tiff", "jp2", "bin".
                *   `width (int)`
                *   `height (int)`
                *   `color_space (str)`: e.g., "/DeviceRGB", "/DeviceGray".
                *   `bits_per_component (int)`
                *   `path (str, Optional)`: If `save_images_locally` was True, path to the saved image file.
                *   `data (str, Optional)`: If `save_images_locally` was False, base64 encoded image data.
                *   `page (int)`: The page number this image was extracted from.
            *   `links (List[str], default_factory: list)`: List of hyperlink URLs found on the page.
            *   `layout (List[Dict], default_factory: list)`: List of dictionaries representing text layout elements, primarily: `{"type": "text", "text": str, "x": float, "y": float}`.
    *   9.5.3. Dataclass `PDFProcessResult` (from `crawl4ai.processors.pdf.processor`)
        *   Fields:
            *   `metadata (PDFMetadata)`
            *   `pages (List[PDFPage])`
            *   `processing_time (float, default: 0.0)`: Time in seconds taken to process the PDF.
            *   `version (str, default: "1.1")`: Version of the PDF processor strategy (e.g., "1.1" for current `NaivePDFProcessorStrategy`).

*   ### 9.6. Using PDF Strategies with `AsyncWebCrawler`
    *   9.6.1. Workflow:
        1.  Instantiate `AsyncWebCrawler`. The `crawler_strategy` parameter of `AsyncWebCrawler` should be set to an instance of `PDFCrawlerStrategy` if you intend to primarily crawl PDF URLs or local PDF files directly. If crawling mixed content where PDFs are discovered via links on HTML pages, the default `AsyncPlaywrightCrawlerStrategy` might be used initially, and then a PDF-specific scraping strategy would be applied when a PDF content type is detected.
        2.  In `CrawlerRunConfig`, set the `scraping_strategy` attribute to an instance of `PDFContentScrapingStrategy`. Configure this strategy with desired options like `extract_images`, `save_images_locally`, etc.
        3.  When `crawler.arun(url="path/to/document.pdf", config=run_config)` is called for a PDF URL or local file path:
            *   `PDFCrawlerStrategy` (if used) or the default crawler strategy fetches the file.
            *   `PDFContentScrapingStrategy.scrape()` is invoked. It uses its internal `NaivePDFProcessorStrategy` instance to parse the PDF.
            *   The extracted text, image data, and metadata are populated into the `CrawlResult` object (e.g., `result.markdown`, `result.media["images"]`, `result.metadata`).
    *   9.6.2. Example Snippet:
        ```python
        import asyncio
        from pathlib import Path
        from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, PDFCrawlerStrategy
        from crawl4ai.content_scraping_strategy import PDFContentScrapingStrategy
        from crawl4ai.processors.pdf import PDFContentScrapingStrategy # Corrected import path

        async def main():
            # Setup for PDF processing
            pdf_crawler_strategy = PDFCrawlerStrategy() # Use if directly targeting PDF URLs
            pdf_scraping_strategy = PDFContentScrapingStrategy(
                extract_images=True,
                save_images_locally=True,
                image_save_dir="./pdf_images_output" # Ensure this directory exists
            )
            Path("./pdf_images_output").mkdir(parents=True, exist_ok=True)

            # If crawling a website that links to PDFs, you might use the default crawler strategy
            # and rely on content-type detection to switch to PDFContentScrapingStrategy if needed.
            # For direct PDF URL:
            async with AsyncWebCrawler(crawler_strategy=pdf_crawler_strategy) as crawler:
                run_config = CrawlerRunConfig(scraping_strategy=pdf_scraping_strategy)
                # Example PDF URL (replace with a real one for testing)
                pdf_url = "https://www.w3.org/WAI/ER/tests/xhtml/testfiles/resources/pdf/dummy.pdf"
                result = await crawler.arun(url=pdf_url, config=run_config)

                if result.success:
                    print(f"Successfully processed PDF: {result.url}")
                    if result.markdown:
                         print(f"Markdown content (first 500 chars): {result.markdown.raw_markdown[:500]}")
                    if result.media and result.media.images:
                        print(f"Extracted {len(result.media.images)} images.")
                        for img in result.media.images:
                            print(f"  - Image source/path: {img.src or img.path}, Page: {img.page}")
                    if result.metadata:
                        print(f"PDF Metadata: {result.metadata}")
                else:
                    print(f"Failed to process PDF: {result.url}, Error: {result.error_message}")

        # if __name__ == "__main__":
        #     asyncio.run(main())
        ```
```