## Cursor Test Prompt Templates

Use these prompt templates to instruct Cursor how to write and run tests for new or existing code.

----------

### 🔹 Unit/Integration Test – Vitest + RTL

**Prompt:**

> Write a Vitest + React Testing Library test for `<ComponentName>`.
> 
> -   Use `render()` with `MemoryRouter` if the component relies on routing.
>     
> -   Include user-event simulation for interactions.
>     
> -   Use `getByText`, `getByRole`, or `findBy...` queries only.
>     
> -   Assert key UI elements and behavior.
>     

**Example:**

```
import { render, screen } from '@testing-library/react';
import { MemoryRouter } from 'react-router-dom';
import PostsDiscoveryPage from '../PostsDiscoveryPage';

it('renders the PostsDiscoveryPage component', () => {
  render(
    <MemoryRouter initialEntries={["/reddit-trends"]}>
      <PostsDiscoveryPage />
    </MemoryRouter>
  );
  expect(screen.getByText(/Reddit Trends/i)).toBeInTheDocument();
});
```

----------

### 🔹 E2E Test – Playwright

**Prompt:**

> Write a Playwright test that:
> 
> -   Visits the route `/reddit-trends`
>     
> -   Confirms the URL and visible UI content
>     
> -   Uses `data-testid` or text queries to find elements
>     

**Example:**

```
import { test, expect } from '@playwright/test';

test('navigates to Reddit Trends page', async ({ page }) => {
  await page.goto('/reddit-trends');
  await expect(page).toHaveURL('/reddit-trends');
  await expect(page.getByText('Reddit Trends')).toBeVisible();
});
```

----------

### 🔹 Sidebar Navigation Test (Integration)

**Prompt:**

> Write a Vitest + RTL test that simulates clicking the sidebar NavItem for `/stats` and verifies the new page renders.

**Example:**

```
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { MemoryRouter } from 'react-router-dom';
import App from '@/App';

test('navigates to StatsOverview via sidebar', async () => {
  render(
    <MemoryRouter initialEntries={['/']}>
      <App />
    </MemoryRouter>
  );
  await userEvent.click(screen.getByText('Stats'));
  expect(await screen.findByText(/Stats Overview/i)).toBeInTheDocument();
});
```

----------

### 🔹 404 Fallback Test

**Prompt:**

> Write a test that visits a missing route and confirms the NotFound page displays.

**Example:**

```
import { test, expect } from '@playwright/test';

test('shows 404 page for unknown route', async ({ page }) => {
  await page.goto('/non-existent-page');
  await expect(page.getByText(/not found/i)).toBeVisible();
});
```

----------

### ✅ Reminder

Each test block should include:

-   Clear scenario
    
-   Scope: unit, integration, or E2E
    
-   Test location based on type
    
-   Data-testid or semantic queries
    

Use this structure as the baseline for all new features and routes.