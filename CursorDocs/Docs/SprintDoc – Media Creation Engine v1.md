# 🚀 SprintDoc – Media Creation Engine v1

This SprintDoc breaks down the architectural components from the ADDoc into implementation-ready prompt units. Each prompt satisfies one or more requirements and can be executed by a Cursor agent or development contributor.

----------

## 🏁 Sprint 1: Reddit Trend Discovery Module

### ✅ Prompt 1.1: RedditTrendService

> Build a microservice named `RedditTrendService` that fetches and filters trending Reddit posts using the Reddit API. Implement filters for subreddit, time range, media type, and keyword exclusion. Use Redis to cache results. Satisfies: FR1.1, FR3.1

### ✅ Prompt 1.2: API Endpoints

> Create RESTful endpoints for: `/api/reddit/trends`, `/api/reddit/trends/bulk`, `/api/reddit/subreddits/search`. Add OpenAPI docs.

----------

## 🏁 Sprint 2: Summarization Engine

### ✅ Prompt 2.1: SummarizationEngine Service

> Build `SummarizationEngine` that connects to OpenAI/Claude. Accept post text and return summaries in short/medium/long form with sentiment. Store results in a `summary` table. Satisfies: FR1.2, FR3.2, FR3.3

### ✅ Prompt 2.2: Summary Templates

> Create a template system for summary formats. Enable retrieval and saving of templates via `/api/summarize/templates`.

----------

## 🏁 Sprint 3: Script Generator

### ✅ Prompt 3.1: Template-Driven Script Generator

> Implement a `ScriptGenerator` microservice. Use templates with placeholders (title, summary, key_points). Satisfies: FR1.3, FR4.1–4.3

### ✅ Prompt 3.2: API Routes

> Provide endpoints for `/api/scripts/generate`, `/api/scripts/templates`, with support for GET/POST/PUT on templates.

----------

## 🏁 Sprint 4: Voice Synthesis

### ✅ Prompt 4.1: VoiceOverService

> Create a `VoiceOverService` that accepts script text, language, and voice profile. Support ElevenLabs or AWS Polly. Return audio with SSML effects. Satisfies: FR1.4, FR5.1–FR5.3

----------

## 🏁 Sprint 5: Video & Upload Module

### ✅ Prompt 5.1: VideoRenderer

> Build service to combine voice-over + images into a 9:16 YouTube Shorts format. Add captions. Use static assets for prototype.

### ✅ Prompt 5.2: YouTubeUploader

> Implement upload & schedule via YouTube Data API. Endpoint: `/api/youtube/upload`. Attach metadata + thumbnail. Satisfies: FR1.5–FR1.6

----------

## 🏁 Sprint 6: Workflow & Dashboard

### ✅ Prompt 6.1: OrchestrationEngine

> Build a JSON-defined pipeline runner that triggers service actions in sequence. Add conditionals and retries. Satisfies: FR2.1–FR2.4

### ✅ Prompt 6.2: Analytics Dashboard (Frontend + API)

> Render real-time and historical analytics using data from content jobs, API usage, and cost metrics. Satisfies: FR6.1–FR6.3

----------

## 🏁 Sprint 7: API Keys + System Infra

### ✅ Prompt 7.1: APIKeyManager

> Create API key issue/rotate/revoke service with rate limiting and logs. UI panel included. Satisfies: FR7.1–FR7.3, NFR1–2

### ✅ Prompt 7.2: Shared Infrastructure Setup

> Scaffold: JobQueue, CacheLayer (Redis), StorageLayer (S3/GCS), AuthService (JWT). Use Docker Compose for development.

----------

Let me know if you want:

-   Task assignments or estimates added
    
-   A generated `todo.yaml` from this
    
-   Or a visual kanban-style breakdown