## 🧪 Cursor-Friendly Testing Rules

This document defines how we test features in this project using **unit, integration, and end-to-end** strategies. Cursor should follow these rules when writing or updating tests.

----------

### ✅ Test Stack Overview

-   **Unit & Integration Tests:**
    
    -   **Framework:** Vitest
        
    -   **Utils:** React Testing Library (RTL), user-event, jsdom
        
    -   **File Convention:** Place under `src/__tests__` or alongside components as `*.test.tsx`
        
-   **E2E Tests:**
    
    -   **Framework:** Playwright
        
    -   **Location:** `tests/e2e/`
        
    -   **File Convention:** `*.spec.ts`
        

----------

### 📁 Directory Structure (Example)

```
src/
  pages/
    PostsDiscoveryPage.tsx
    __tests__/
      PostsDiscoveryPage.test.tsx
  test/
    setup.ts

tests/
  e2e/
    routing.spec.ts

```

----------

### 🔧 Testing Rules

#### 🧬 Unit & Integration (Vitest + RTL)

-   Use `describe()` blocks per component or route.
    
-   Use `render(<MemoryRouter initialEntries={["/target"]}>...</MemoryRouter>)` for routing tests.
    
-   Query using `getByText`, `getByRole`, or `findBy...` (not `querySelector`).
    
-   Simulate navigation via `@testing-library/user-event`.
    
-   Keep tests focused and readable. Max 1-2 assertions per `it()` block.
    

#### 🌐 E2E (Playwright)

-   All tests start by navigating to the route or using sidebar UI.
    
-   Prefer selectors via `data-testid`.
    
-   Include assertions like:
    
    -   `await expect(page).toHaveURL("/reddit-trends")`
        
    -   `await expect(page.getByText("Reddit Trends")).toBeVisible()`
        
-   Avoid brittle selectors like `nth-child`, `div > div > span`.
    

#### 🧪 General Practices

-   Snapshot testing is discouraged.
    
-   Prefer real data over mocks unless API or state management is too complex.
    
-   Use setup and teardown hooks (`beforeEach`, `afterAll`) responsibly.
    
-   Tag slow tests with `.only` during development, but never commit `.only` or `.skip`.
    

----------

### 🛠 Setup

#### Install Test Libraries

```bash
npm install --save-dev vitest @testing-library/react @testing-library/user-event jsdom
npm install --save-dev playwright
npx playwright install

```

#### Configure Vite

```ts
// vite.config.ts
export default defineConfig({
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: './src/test/setup.ts',
  },
});

```

#### Setup Test Globals

```ts
// src/test/setup.ts
import '@testing-library/jest-dom';

```

----------

### ✅ Enforcement

Cursor should:

-   Automatically generate tests after any route or component is created.
    
-   Use the correct test type based on scope.
    
-   Include test and documentation update checklist for every feature task.