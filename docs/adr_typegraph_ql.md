# ADR: Use of TypeGraphQL

## Status
Accepted

## Context
We require a flexible, strongly typed approach to expose APIs where clients can fetch only the data they need. REST APIs often result in over-fetching or under-fetching of data. We also want to leverage TypeScript’s features and decorator-based syntax.

## Decision
Adopt **TypeGraphQL** as the framework for building GraphQL APIs using TypeScript decorators and classes. It integrates well with Ts.ED and allows us to define schema and resolvers in a clean, type-safe manner.

## Benefits
- Fully type-safe schema generation
- Clean integration with classes, decorators, and dependency injection
- Eliminates duplication between TypeScript types and GraphQL schema
- Works seamlessly with Ts.ED, TypeORM, and class-validator

## Installation
```bash
npm install type-graphql graphql reflect-metadata class-validator apollo-server-express
```

## Example Usage
```ts
@ObjectType()
class User {
  @Field()
  name: string;

  @Field()
  email: string;
}

@Resolver()
class UserResolver {
  @Query(() => [User])
  async users(): Promise<User[]> {
    return userService.findAll();
  }
}
```

## Project Integration (with Ts.ED)
In your `Server.ts` or module setup:
```ts
import { GraphQLModule } from '@tsed/typegraphql';

@Configuration({
  imports: [
    GraphQLModule,
    // other modules...
  ]
})
export class Server {}
```

## Consequences
- Requires familiarity with GraphQL concepts and resolver patterns
- Can add complexity if not properly organized
- Greatly enhances API flexibility and type safety