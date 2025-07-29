# ADR: Use of npm Workspaces for Monorepo

## Status

Accepted

## Context

Managing multiple backend microservices as separate repositories leads to fragmented dependency management, inconsistent tooling, and duplicated configuration. We require a unified structure that supports development, testing, and deployment across services.

## Decision

Use **npm workspaces** to manage the project as a monorepo, allowing each service to have its own dependencies while sharing common tools and configurations at the root.

## Benefits

- Centralized dependency and script management
- Isolated yet coordinated service development
- Simplified CI/CD setup
- Promotes reuse of shared libraries and tools

## Installation

Ensure you're using **npm v7+** and enable workspaces in the root `package.json`.

```json
{
  "name": "backend-monorepo",
  "private": true,
  "workspaces": ["services/*"]
}
```

Install dependencies from the root:

```bash
npm install
```

## Project Structure

```
root/
├── package.json             # Root-level workspaces config
├── node_modules/            # Shared dependencies
├── tsconfig.base.json       # Shared TypeScript config
├── .eslintrc.cjs            # Shared lint rules
├── .env                     # Global environment config
└── services/
    ├── auth-service/
    │   ├── package.json
    │   └── src/
    └── dashboard-service/
        ├── package.json
        └── src/
```

## Consequences

- Encourages consistency across microservices
- Requires learning npm workspace behaviors (e.g., hoisting rules)
- Best suited for projects where services are closely coupled or deployed together

