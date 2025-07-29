# ADR: Use of MongoDB and MySQL with TypeORM

## Status

Accepted

## Context

Our microservices backend needs to support both relational and non-relational data. While MySQL is suited for structured, transactional data (e.g., users, auth, payments), MongoDB offers flexibility for dynamic or unstructured content (e.g., logs, analytics, profile metadata). To keep development consistent, we use **TypeORM** to interact with both databases.

## Decision

Use **TypeORM** as the unified ORM to connect with both **MongoDB** and **MySQL**, leveraging its multi-database support and consistent API.

## Benefits

- Unified development experience and syntax
- Enables hybrid persistence strategies (SQL + NoSQL)
- Compatible with Hexagonal Architecture via Repository Pattern
- Enables migration, schema synchronization, and data validations

## Installation

```bash
npm install typeorm reflect-metadata mysql mongodb
```

## MongoDB Example Config

```ts
export const MongoDataSource = new DataSource({
  type: 'mongodb',
  url: process.env.MONGO_URL,
  useNewUrlParser: true,
  useUnifiedTopology: true,
  entities: [__dirname + '/entities/mongo/*.ts'],
  synchronize: true
});
```

## MySQL Example Config

```ts
export const MySQLDataSource = new DataSource({
  type: 'mysql',
  host: process.env.MYSQL_HOST,
  port: 3306,
  username: process.env.MYSQL_USER,
  password: process.env.MYSQL_PASSWORD,
  database: process.env.MYSQL_DB,
  entities: [__dirname + '/entities/sql/*.ts'],
  synchronize: false,
  migrations: [__dirname + '/migrations/*.ts'],
});
```

## Consequences

- Developers need awareness of differences in behavior between SQL and NoSQL
- TypeORM features for MongoDB are less mature than for SQL
- Allows us to choose the best storage type per service use case

