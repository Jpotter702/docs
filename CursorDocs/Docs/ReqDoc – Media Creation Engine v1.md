# 📄 ReqDoc – Media Creation Engine v1

This document extracts functional and technical requirements based on the UseCaseDoc. These requirements define the behavior, interfaces, and performance expectations for the Media Creation Engine.

----------

## 🔹 Functional Requirements (FR)

### 🧠 From UC1: One-Click YouTube Short Generation

-   **FR1.1**: The system must fetch top Reddit posts by subreddit and time range.
    
-   **FR1.2**: The system must summarize Reddit post content using AI.
    
-   **FR1.3**: The system must generate a video script using selected templates.
    
-   **FR1.4**: The system must synthesize voice-over with background music.
    
-   **FR1.5**: The system must render video assets into YouTube Shorts format.
    
-   **FR1.6**: The system must upload and schedule videos on YouTube.
    

### 🧠 From UC2: Custom Workflow Builder

-   **FR2.1**: Users must be able to visually compose workflows via UI.
    
-   **FR2.2**: Each workflow step must be configurable with I/O and conditions.
    
-   **FR2.3**: Workflows must support saving, versioning, and reuse.
    
-   **FR2.4**: Workflows must be executable manually or via scheduler.
    

### 🧠 From UC3: Content Summary Archive

-   **FR3.1**: Users must be able to paste or load Reddit content for summarization.
    
-   **FR3.2**: Users must choose from predefined summary styles.
    
-   **FR3.3**: The system must store/export summaries and metadata.
    

### 🧠 From UC4: Script Template Creation

-   **FR4.1**: The system must provide an interface to create/edit script templates.
    
-   **FR4.2**: Templates must support placeholder variables and narration style tags.
    
-   **FR4.3**: Templates must be saved to the user profile and versioned.
    

### 🧠 From UC5: Multi-Language Voice Content

-   **FR5.1**: The system must offer multi-language support for voice synthesis.
    
-   **FR5.2**: The system must auto-translate summaries before narration.
    
-   **FR5.3**: Captions must match voice language.
    

### 🧠 From UC6: Performance Dashboard Usage

-   **FR6.1**: System must record content creation and engagement metrics.
    
-   **FR6.2**: Users must view visualized data filtered by time, workflow, or type.
    
-   **FR6.3**: Dashboards must display cost breakdowns and conversion KPIs.
    

### 🧠 From UC7: Secure API Key Management

-   **FR7.1**: Users must generate, revoke, and rotate API keys.
    
-   **FR7.2**: System must enforce rate limits based on tier.
    
-   **FR7.3**: API usage logs and alerts must be available to users.
    

----------

## 🔸 Non-Functional Requirements (NFR)

### 🛡️ Security

-   **NFR1**: End-to-end encryption of user data and API keys.
    
-   **NFR2**: Role-based access control.
    

### ⚡ Performance

-   **NFR3**: Average job turnaround < 30s.
    
-   **NFR4**: Backend services must support async job processing.
    

### 🏗️ Scalability

-   **NFR5**: System must support multi-tenant architecture.
    
-   **NFR6**: Auto-scale services based on job queue load.
    

### 🔔 Observability

-   **NFR7**: Real-time service health dashboard.
    
-   **NFR8**: Alerting for service errors, latency, or job failures.
    

----------

Let me know when you're ready for the Architecture Design Doc (ADDoc).