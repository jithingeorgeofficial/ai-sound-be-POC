# Architectural Decision Record (ADR): Vitest for Testing

## Status
Accepted

## Context
We needed a fast, modern, and developer-friendly unit testing framework that integrates seamlessly with our TypeScript-based Node.js backend. The key requirements were:

- Compatibility with TypeScript
- Fast execution for unit and integration tests
- Easy mocking and snapshot support
- Good support for ES modules and modern JS syntax
- Compatibility with monorepos and npm workspaces

After evaluating alternatives like **Jest**, **Mocha + Chai**, and **AVA**, we decided to go with **Vitest**.

## Decision

We will use **Vitest** as the default testing framework for all services in the project.

### Justification
- **Performance**: Vitest is built on Vite and offers significantly faster test startup and execution.
- **Developer Experience**: Offers Jest-compatible APIs (e.g., `describe`, `it`, `expect`) for easy migration and readability.
- **Built-in mocking and snapshots**: No need for external libraries.
- **TypeScript-first**: Works out of the box with TypeScript, including support for source maps.
- **Lightweight configuration**: Minimal setup and fast feedback loop.
- **Native ESM support**: Ideal for modern projects using ES modules.

## Implementation Plan

1. Install Vitest and related libraries:
   ```bash
   npm install -D vitest @vitest/ui @vitest/coverage-c8 tsx
   ```

2. Add test script to `package.json`:
   ```json
   "scripts": {
     "test": "vitest run",
     "test:watch": "vitest"
   }
   ```

3. Create `vitest.config.ts`:
   ```ts
   import { defineConfig } from 'vitest/config';

   export default defineConfig({
     test: {
       globals: true,
       environment: 'node',
       include: ['tests/**/*.spec.ts'],
     },
   });
   ```

4. Folder structure:
   ```
   /tests
     └── userService.spec.ts
   /src
     └── services
         └── userService.ts
   ```

5. Add test coverage:
   ```bash
   npm run test -- --coverage
   ```

## Consequences

- ⚠ Some Jest plugins are not compatible; migration may require rewriting certain parts.
- ✅ Improved performance and DX.
- ✅ Consistency with our modern tooling and TS-based stack.
- ✅ Easier setup in monorepo environments (npm workspaces).

## Alternatives Considered

### Jest
- Pros: Mature, full-featured
- Cons: Slower, complex config for monorepos, heavier runtime

### Mocha + Chai
- Pros: Flexible, customizable
- Cons: Requires manual setup for mocks/assertions, slower

