# 🏗️ ADDoc – Architecture Design Document v1

This document translates the requirement set into logical components and services. These are the architectural building blocks that will satisfy the functional and non-functional requirements defined in the ReqDoc.

----------

## 🔹 Component Map

### 📥 RedditTrendService

-   **Responsibilities**: Fetch trending Reddit content with filtering
    
-   **Satisfies**: FR1.1, FR3.1
    
-   **Interfaces**: Reddit API, Local Cache (Redis), ConfigStore
    

### 🧠 SummarizationEngine

-   **Responsibilities**: Generate key points and summaries
    
-   **Satisfies**: FR1.2, FR3.2, FR3.3, FR5.2
    
-   **Interfaces**: OpenAI/Claude APIs, SummaryTemplateStore
    

### 📝 ScriptGenerator

-   **Responsibilities**: Generate structured scripts from summaries
    
-   **Satisfies**: FR1.3, FR4.1, FR4.2, FR4.3
    
-   **Interfaces**: TemplateManager, OrchestrationEngine
    

### 🔊 VoiceOverService

-   **Responsibilities**: Generate voice narration with optional music
    
-   **Satisfies**: FR1.4, FR5.1, FR5.3
    
-   **Interfaces**: ElevenLabs/Azure/AWS Polly, AudioPostProcessor
    

### 🎞️ VideoRenderer

-   **Responsibilities**: Render final Shorts from scripts + voice
    
-   **Satisfies**: FR1.5
    
-   **Interfaces**: AssetLibrary, SubtitleGenerator
    

### 📺 YouTubeUploader

-   **Responsibilities**: Schedule uploads and manage metadata
    
-   **Satisfies**: FR1.6
    
-   **Interfaces**: YouTube Data API, Scheduler, ThumbnailGen
    

----------

## 🔹 Workflow & Logic

### 🔧 OrchestrationEngine

-   **Responsibilities**: Pipeline logic execution and chaining
    
-   **Satisfies**: FR2.1–FR2.4
    
-   **Interfaces**: All core services, ConfigStore, WorkflowStore
    

### 🧩 WorkflowBuilder (Frontend)

-   **Responsibilities**: Visual editor for assembling pipelines
    
-   **Satisfies**: FR2.1, FR2.2
    

### 🔍 AnalyticsDashboard

-   **Responsibilities**: Visualize KPIs and metrics
    
-   **Satisfies**: FR6.1–FR6.3, NFR7
    
-   **Interfaces**: AnalyticsStore, UserActivityLog, CostEngine
    

### 🔐 APIKeyManager

-   **Responsibilities**: Generate, rotate, monitor API keys
    
-   **Satisfies**: FR7.1–FR7.3, NFR1–2
    
-   **Interfaces**: AuthService, RateLimiter, AuditLog
    

----------

## 🔹 Shared Infrastructure

### 📂 ConfigStore

-   Stores workflow configurations, templates, user prefs
    

### 📈 MetricsService

-   Tracks API usage, job stats, success/failure rates
    

### 🔐 AuthService

-   Manages sessions, JWTs, OAuth, role permissions
    

### 🗃️ JobQueue

-   Background processing using Bull/SQS/RedisQueue
    

### 🧠 CacheLayer

-   Short-term storage of Reddit posts, summaries, audio URLs
    

### 💾 StorageLayer

-   File store for voice, scripts, videos (S3/GCS)