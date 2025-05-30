## Cursor Documentation Prompt Templates

Use these prompts to define how Cursor should generate or update documentation alongside code changes. Each doc-related update should be tied to a feature, page, route, or UI component.

----------

### 🔸 Feature Documentation

**Prompt:**

> Create a `docs/features/<feature-name>.md` file.
> 
> -   Describe what the feature does.
>     
> -   List its route, key UI components, and data dependencies.
>     
> -   Refer to the requirement, feature and use case it satisfies.
>     

**Example:**

```
# Reddit Trends Feature

**Route:** `/reddit-trends`

This feature shows trending Reddit posts based on selected criteria. It pulls data from the Reddit API and visualizes it using cards and charts.

### Components
- `<PostsDiscovery />`
- `<PostCard />`

### Use Cases
- Track daily subreddit performance
- Spot high-engagement threads
```

----------

### 🔸 Routing Table

**Prompt:**

> Update the main `docs/ROUTING.md` file with a new row in the route/component table.

**Example:**

```
| Route              | Page Component           | Feature Component      |
|--------------------|--------------------------|------------------------|
| /reddit-trends     | PostsDiscoveryPage.tsx   | PostsDiscovery         |
```

----------

### 🔸 Developer Notes (Per Component)

**Prompt:**

> Add or update `src/components/<ComponentName>/README.md` with:
> 
> -   Component purpose
>     
> -   Props interface
>     
> -   Integration examples (where/how it's used)
>     

**Example:**

```
# PostsDiscovery

### Purpose
Renders a list of trending Reddit posts based on search criteria.

### Props
```ts
interface Props {
  subreddit?: string;
  sortOrder?: 'top' | 'hot';
}
```

### Used In

-   `PostsDiscoveryPage.tsx`
    
-   `Dashboard.tsx` (legacy)
    

```

---

### 🔸 API Key Setup Instructions

**Prompt:**
> Create or update `docs/setup/api-keys.md` with instructions for generating and adding keys.

**Example:**
```md
# API Keys

To use the content generator, you must provide your OpenAI API key.

1. Go to [platform.openai.com](https://platform.openai.com/account/api-keys)
2. Copy your key
3. Add it to `.env.local` as `OPENAI_API_KEY=your-key-here`
```

----------

### 🔸 Dashboard Overview Doc

**Prompt:**

> Update `docs/pages/dashboard.md` to reflect any added/removed sections or links.

**Example:**

```
# Dashboard Page

The dashboard provides an overview of the app's features and serves as a launchpad for:

- Reddit Trends
- Content Generator
- Stats Overview

Recent changes:
- Removed direct rendering of components in favor of individual route pages.
```

----------

### ✅ Reminder

Each feature task should include a documentation checklist:

-   Feature doc (markdown)
    
-   ROUTING.md update (if applicable)
    
-   Component README
    
-   Any relevant setup instructions
    

These updates help future devs (and Cursor) understand and extend the system clearly.