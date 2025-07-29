# ADR: Use of Ts.ED Framework

## Status
Accepted

## Context
We need a backend framework that promotes clean code architecture, supports decorators and dependency injection, integrates well with TypeScript, and allows us to expose REST and GraphQL APIs within a microservices environment.

## Decision
Adopt **Ts.ED**, a TypeScript framework built on top of Express.js. It offers out-of-the-box support for controllers, middlewares, OpenAPI (Swagger), GraphQL, and Dependency Injection using a familiar decorator-based approach.

## Benefits
- Strong TypeScript support with decorators
- Native REST API structure with controllers and routes
- Built-in OpenAPI and Swagger integration
- Supports DI, middlewares, GraphQL, and testing tools
- Integrates easily with TypeORM and other libraries

## Installation
```bash
npm install @tsed/cli -g
npx -p @tsed/cli tsed init .
```
You will be prompted to choose:
- Framework: Express
- Features: TypeORM, TypeGraphQL, Swagger, etc.

## Project Structure
```
services/auth-service/
├── src/
│   ├── controllers/
│   ├── services/
│   ├── middlewares/
│   ├── repositories/
│   └── Server.ts            # Entry point (TsED bootstrap)
├── .env
└── package.json
```

## Consequences
- Slight learning curve due to custom decorators and framework conventions
- Clean separation of concerns via modular files
- Provides structure for scalable microservice development