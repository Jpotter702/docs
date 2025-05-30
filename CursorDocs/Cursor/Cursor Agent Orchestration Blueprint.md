## Cursor Agent Orchestration Blueprint

### 🔧 1. **Define Agent Behavior**

Create a `cursor-agent.config.ts` or YAML/JSON agent file to establish the behavior:

ts

CopyEdit

`// cursor-agent.config.ts  export  default { name: "AutoDocTester", trigger: ["file.create", "file.update"], conditions: [
    { path: "**/pages/**/*.tsx", actions: ["create", "update"] },
    { path: "**/components/**/*.tsx", actions: ["create", "update"] }
  ], tasks: [
    { use: "generateUnitTests" },
    { use: "generateE2ETests" },
    { use: "updateDocumentation" }
  ]
};` 

----------

### ⚙️ 2. **Define Task Templates**

In Cursor’s local or workspace-level task directory, define reusable tasks like:

#### `generateUnitTests.ts`

ts

CopyEdit

``export  async  function  run(filePath, context) { if (!filePath.endsWith(".tsx")) return; const componentName = extractComponentName(filePath); return  `Write a Vitest + RTL unit test for the new component: ${componentName}.
Use MemoryRouter if needed, test all visible elements and interactions.`;
}`` 

#### `generateE2ETests.ts`

ts

CopyEdit

``export  async  function  run(filePath, context) { if (!filePath.includes("/pages/")) return; const route = inferRouteFromPath(filePath); // e.g. "/reddit-trends"  return `Write a Playwright E2E test that:
- Navigates to ${route} - Waits for content to load
- Confirms key text and URL are correct`;
}`` 

#### `updateDocumentation.ts`

ts

CopyEdit

``export  async  function  run(filePath, context) { const name = extractFeatureName(filePath); return `Update the following documentation:
- Create or update \`docs/features/${name}.md\`
- Add route info to \`ROUTING.md\`
- Add a README to the component folder if not present`;
}`` 

----------

### 🧩 3. **Chain Tasks Together**

Cursor’s agent can execute these tasks _sequentially_ for each event. You can use a simple orchestrator like:

ts

CopyEdit

`export  async  function  runAllTasks(filePath, context) { await  run(generateUnitTests, filePath, context); await  run(generateE2ETests, filePath, context); await  run(updateDocumentation, filePath, context);
}` 

----------

### 📋 4. **Make It Declarative (Optional YAML)**

If Cursor supports YAML config:

yaml

CopyEdit

`name:  AutoDocTester  trigger:  -  file.create  -  file.update  conditions:  -  path:  "**/pages/**/*.tsx"  -  path:  "**/components/**/*.tsx"  tasks:  -  template:  generateUnitTests  -  template:  generateE2ETests  -  template:  updateDocumentation` 

----------

## 🧪 Result: Agent Workflow

**On new file or change:**

1.  Detects the route/component
    
2.  Generates the matching test (unit or E2E)
    
3.  Creates/upgrades feature markdown, routing table, README