## Slide 01 — Title

# Introduction to GraphQL

**A Query Language for Your APIs**

schema · queries · mutations · subscriptions · resolvers · Apollo

---

## Slide 02 — Agenda

### Foundations
- What is GraphQL & its origin
- GraphQL vs REST
- Schema Definition Language (SDL)
- Queries, Mutations & Subscriptions

### Server-Side
- Resolvers & the resolver chain
- Apollo Server with Express
- DataLoaders & the N+1 problem
- Authentication & authorisation

### Client-Side
- Apollo Client & React integration
- Caching & optimistic updates
- Error handling strategies
- Pagination patterns

### Production
- Schema design patterns
- Performance & security
- Testing GraphQL APIs
- Summary & next steps

---

## Slide 03 — What Is GraphQL?

GraphQL is an **open-source query language for APIs** and a runtime for executing those queries against a type system you define. Created at **Facebook in 2012**, open-sourced in **2015**, and now governed by the GraphQL Foundation under the Linux Foundation.

Unlike REST, GraphQL exposes a **single endpoint**. Clients describe exactly the data they need, and the server returns precisely that shape — nothing more, nothing less.

### Core Principles
- **Declarative data fetching** — client specifies the shape
- **Strongly typed** — schema defines every field and type
- **Single endpoint** — one URL, one POST
- **Introspective** — clients can query the schema itself

```graphql
query {
  user(id: "42") {
    name
    email
    posts(last: 3) {
      title
      createdAt
    }
  }
}
```

```json
{
  "data": {
    "user": {
      "name": "Alice",
      "email": "alice@example.com",
      "posts": [
        { "title": "GraphQL 101", "createdAt": "2025-12-01" },
        { "title": "SDL Deep Dive", "createdAt": "2025-11-15" },
        { "title": "Resolver Tips", "createdAt": "2025-11-02" }
      ]
    }
  }
}
```

---

## Slide 04 — GraphQL vs REST

| Aspect | REST | GraphQL |
|--------|------|---------|
| Endpoints | Multiple (`/users`, `/users/:id/posts`) | Single (`/graphql`) |
| Data fetching | Server decides response shape | Client specifies exact fields |
| Over-fetching | Common — returns entire resource | Eliminated — request only what you need |
| Under-fetching | Requires multiple round-trips | Single request traverses the graph |
| N+1 Requests | Client may chain GET calls | Resolved server-side with DataLoader |
| Versioning | `/api/v1`, `/api/v2` | Schema evolution, deprecation directives |
| Caching | HTTP caching (ETags, 304) | Normalised client cache (Apollo, Relay) |
| Contract | OpenAPI / Swagger (opt-in) | Schema-first — always present |

### When REST Wins
- Simple CRUD with uniform resource shapes
- Heavy reliance on HTTP caching (CDN edge)
- File uploads (though GraphQL multipart spec exists)
- Team unfamiliarity with GraphQL tooling

### When GraphQL Wins
- Multiple clients needing different data shapes (web, mobile, IoT)
- Deeply nested or interconnected data (social graphs)
- Rapid frontend iteration without backend changes
- Microservice aggregation via federation

---

## Slide 05 — Schema Definition Language (SDL)

The schema is the **contract** between client and server. SDL defines every type, field, argument, and relationship in a human-readable syntax.

```graphql
# Scalar types: String, Int, Float, Boolean, ID
# ! = non-null, [] = list

type User {
  id: ID!
  name: String!
  email: String!
  role: Role!
  posts: [Post!]!
  createdAt: DateTime!
}

type Post {
  id: ID!
  title: String!
  body: String!
  author: User!
  tags: [String!]!
  status: PostStatus!
}

enum Role {
  ADMIN
  EDITOR
  VIEWER
}

enum PostStatus {
  DRAFT
  PUBLISHED
  ARCHIVED
}

scalar DateTime
```

```graphql
input CreateUserInput {
  name: String!
  email: String!
  role: Role = VIEWER
}

input PostFilterInput {
  status: PostStatus
  authorId: ID
  tag: String
}

type Query {
  user(id: ID!): User
  users(limit: Int = 10, offset: Int = 0): [User!]!
  posts(filter: PostFilterInput): [Post!]!
}

type Mutation {
  createUser(input: CreateUserInput!): User!
  updateUser(id: ID!, input: CreateUserInput!): User!
  deleteUser(id: ID!): Boolean!
}

type Subscription {
  postPublished: Post!
}
```

### Non-Null Rules
- `String` — nullable field, may return `null`
- `String!` — never null, server guarantees a value
- `[String!]!` — non-null list of non-null strings
- `[String]!` — non-null list, but items may be null

---

## Slide 06 — Queries

### Field Selection & Nesting

```graphql
query {
  user(id: "42") {
    name
    email
    posts {
      title
      tags
    }
  }
}
```

### Arguments

```graphql
query {
  users(limit: 5, offset: 10) {
    name
    posts(status: PUBLISHED) {
      title
    }
  }
}
```

### Aliases

Rename fields to avoid conflicts when querying the same field with different arguments.

```graphql
query {
  admins: users(role: ADMIN) {
    name
  }
  editors: users(role: EDITOR) {
    name
  }
}
```

### Fragments

Reusable field selections — DRY for repeated structures.

```graphql
fragment UserFields on User {
  id
  name
  email
  role
}

query {
  user(id: "42") { ...UserFields }
  me { ...UserFields }
}
```

---

## Slide 07 — Mutations

Mutations are the GraphQL equivalent of POST/PUT/DELETE. They **modify server-side data** and return a result that the client can query fields from.

### Create

```graphql
mutation {
  createUser(input: {
    name: "Alice"
    email: "alice@example.com"
    role: EDITOR
  }) {
    id
    name
    createdAt
  }
}
```

### Update

```graphql
mutation {
  updateUser(id: "42", input: {
    name: "Alice Smith"
    email: "alice.smith@example.com"
  }) {
    id
    name
    email
  }
}
```

### Delete

```graphql
mutation {
  deleteUser(id: "42")
}
```

### Variables & Operation Names

Production clients always use variables — never string interpolation.

```graphql
mutation CreateUser($input: CreateUserInput!) {
  createUser(input: $input) {
    id
    name
  }
}

# Variables (JSON):
# { "input": { "name": "Bob", "email": "bob@co.io" } }
```

### Mutation Design Tips
- Use **input types** for complex arguments
- Return the **mutated object** so the client cache updates
- Consider a **payload type** with `errors` field for domain errors

---

## Slide 08 — Subscriptions

Subscriptions provide **real-time, push-based updates** from server to client. They use a persistent connection (typically **WebSocket**) instead of request-response.

### Schema

```graphql
type Subscription {
  postPublished: Post!
  commentAdded(postId: ID!): Comment!
  userStatusChanged: User!
}
```

### Client Usage

```graphql
subscription OnNewPost {
  postPublished {
    id
    title
    author {
      name
    }
  }
}
```

### Transport Protocols
- **graphql-ws** — modern protocol (recommended)
- **subscriptions-transport-ws** — legacy, deprecated
- **Server-Sent Events** — HTTP-based alternative

### Server Implementation (Apollo)

```javascript
const { PubSub } = require('graphql-subscriptions');
const pubsub = new PubSub();

const resolvers = {
  Mutation: {
    createPost: async (_, { input }, ctx) => {
      const post = await ctx.db.posts.create(input);
      pubsub.publish('POST_PUBLISHED', {
        postPublished: post,
      });
      return post;
    },
  },
  Subscription: {
    postPublished: {
      subscribe: () =>
        pubsub.asyncIterableIterator(['POST_PUBLISHED']),
    },
    commentAdded: {
      subscribe: (_, { postId }) =>
        pubsub.asyncIterableIterator([`COMMENT_${postId}`]),
    },
  },
};
```

---

## Slide 09 — Resolvers

Resolvers are **functions that produce the value for each field** in the schema. Every field has a resolver — if you don't write one, the default resolver reads the property from the parent object.

```javascript
const resolvers = {
  Query: {
    // resolver(parent, args, context, info)
    user: async (_, { id }, ctx) => {
      return ctx.db.users.findById(id);
    },
    users: async (_, { limit, offset }, ctx) => {
      return ctx.db.users.findAll({ limit, offset });
    },
  },

  User: {
    posts: async (parent, _, ctx) => {
      return ctx.db.posts.findByAuthor(parent.id);
    },
    fullName: (parent) => {
      return `${parent.firstName} ${parent.lastName}`;
    },
  },

  Post: {
    author: async (parent, _, ctx) => {
      return ctx.db.users.findById(parent.authorId);
    },
  },
};
```

### The Four Arguments

| Arg | Purpose |
|-----|---------|
| parent | Result from the parent resolver |
| args | Arguments passed to the field |
| context | Shared per-request state (db, auth, loaders) |
| info | AST of the query, field name, schema metadata |

### Resolver Chain
GraphQL executes resolvers **top-down**. A `Query.user` resolver returns a User object, then each requested field on that User triggers its own resolver. The chain continues until all scalar leaves are resolved.

### Default Resolver
If no resolver is defined for a field, GraphQL reads `parent[fieldName]`. This is why returning plain objects with matching property names "just works".

---

## Slide 10 — Apollo Server with Express

```javascript
const express = require('express');
const { ApolloServer } = require('@apollo/server');
const { expressMiddleware } = require('@apollo/server/express4');
const cors = require('cors');

const typeDefs = `#graphql
  type Query {
    hello: String!
    users: [User!]!
  }
  type User {
    id: ID!
    name: String!
    email: String!
  }
`;

const resolvers = {
  Query: {
    hello: () => 'Hello, GraphQL!',
    users: async (_, __, ctx) => ctx.db.users.findAll(),
  },
};

async function startServer() {
  const app = express();
  const server = new ApolloServer({ typeDefs, resolvers });
  await server.start();

  app.use(
    '/graphql',
    cors(),
    express.json(),
    expressMiddleware(server, {
      context: async ({ req }) => ({
        db: require('./db'),
        user: await authenticate(req),
      }),
    })
  );

  app.listen(4000, () =>
    console.log('GraphQL at http://localhost:4000/graphql')
  );
}

startServer();
```

### Key Components
- **typeDefs** — your SDL schema string or file
- **resolvers** — map of type → field → function
- **context** — built per-request, shared across resolvers
- **expressMiddleware** — plugs Apollo into Express

### Context Best Practices
- Create DataLoader instances per request
- Authenticate the user and attach to context
- Pass database connections / pools
- Never store mutable state between requests

### Alternatives
- **Yoga** — by The Guild, spec-compliant
- **Mercurius** — Fastify-native
- **Pothos** — code-first schema builder

---

## Slide 11 — Apollo Client

Apollo Client is the most popular **GraphQL client for React**. It provides hooks for queries and mutations, a normalised in-memory cache, and optimistic UI updates.

### Setup

```javascript
import { ApolloClient, InMemoryCache, ApolloProvider } from '@apollo/client';

const client = new ApolloClient({
  uri: 'http://localhost:4000/graphql',
  cache: new InMemoryCache(),
});

// Wrap your app
<ApolloProvider client={client}>
  <App />
</ApolloProvider>
```

### useQuery

```javascript
import { useQuery, gql } from '@apollo/client';

const GET_USERS = gql`
  query GetUsers {
    users { id name email }
  }
`;

function UserList() {
  const { loading, error, data } = useQuery(GET_USERS);

  if (loading) return <Spinner />;
  if (error) return <Error msg={error.message} />;

  return data.users.map(u =>
    <div key={u.id}>{u.name}</div>
  );
}
```

### useMutation

```javascript
const CREATE_USER = gql`
  mutation CreateUser($input: CreateUserInput!) {
    createUser(input: $input) { id name email }
  }
`;

function CreateUserForm() {
  const [createUser, { loading }] = useMutation(
    CREATE_USER,
    {
      update(cache, { data: { createUser } }) {
        cache.modify({
          fields: {
            users(existing = []) {
              const newRef = cache.writeFragment({
                data: createUser,
                fragment: gql`fragment NewUser on User { id name email }`,
              });
              return [...existing, newRef];
            },
          },
        });
      },
    }
  );
}
```

### Optimistic Updates
Pass `optimisticResponse` to `useMutation` to instantly reflect changes in the UI before the server responds. If the mutation fails, the cache rolls back automatically.

---

## Slide 12 — DataLoaders & the N+1 Problem

### The N+1 Problem

Querying a list of 50 posts, each with an author, fires **1 query for posts + 50 individual queries for authors = 51 queries**. This pattern kills database performance.

```graphql
query {
  posts {          # 1 query: SELECT * FROM posts
    title
    author {       # N queries: SELECT * FROM users WHERE id = ?
      name
    }
  }
}
```

### The Solution: DataLoader

Facebook's `dataloader` library **batches** individual loads into a single query and **caches** results within a request.

```javascript
const DataLoader = require('dataloader');

const userLoader = new DataLoader(async (ids) => {
  const users = await db.query(
    'SELECT * FROM users WHERE id = ANY($1)',
    [ids]
  );
  const userMap = new Map(users.map(u => [u.id, u]));
  return ids.map(id => userMap.get(id) || null);
});

// In resolver
const resolvers = {
  Post: {
    author: (post, _, ctx) =>
      ctx.loaders.user.load(post.authorId),
  },
};
```

### Critical Rules
- **Create loaders per-request** — never share across requests (stale cache, auth leaks)
- Batch function must return results in **same order** as keys
- DataLoader deduplicates — loading the same ID twice hits DB once
- Use `{ cache: false }` to disable per-request caching if needed

---

## Slide 13 — Authentication & Authorisation

### Context-Based Auth

```javascript
app.use('/graphql', expressMiddleware(server, {
  context: async ({ req }) => {
    const token = req.headers.authorization?.split(' ')[1];
    let user = null;
    if (token) {
      try {
        user = jwt.verify(token, process.env.JWT_SECRET);
      } catch (e) { /* Token invalid or expired */ }
    }
    return { user, db, loaders: createLoaders() };
  },
}));
```

### Resolver-Level Checks

```javascript
const resolvers = {
  Query: {
    adminDashboard: (_, __, ctx) => {
      if (!ctx.user) throw new GraphQLError(
        'Not authenticated',
        { extensions: { code: 'UNAUTHENTICATED' } }
      );
      if (ctx.user.role !== 'ADMIN')
        throw new GraphQLError(
          'Not authorised',
          { extensions: { code: 'FORBIDDEN' } }
        );
      return ctx.db.dashboard.getStats();
    },
  },
};
```

### Directive-Based Auth

```graphql
directive @auth(requires: Role!) on FIELD_DEFINITION

type Query {
  publicPosts: [Post!]!
  adminDashboard: Dashboard! @auth(requires: ADMIN)
  myProfile: User! @auth(requires: VIEWER)
}
```

### Field-Level Permissions
Hide sensitive fields (email, salary) based on the caller's role. Return `null` or throw for unauthorised field access — the rest of the query still resolves.

---

## Slide 14 — Error Handling

### GraphQL Error Format

GraphQL always returns HTTP 200. Errors appear in the `errors` array alongside any partial `data`.

```json
{
  "data": {
    "user": {
      "name": "Alice",
      "secretField": null
    }
  },
  "errors": [
    {
      "message": "Not authorised to view secretField",
      "locations": [{ "line": 4, "column": 5 }],
      "path": ["user", "secretField"],
      "extensions": {
        "code": "FORBIDDEN",
        "http": { "status": 403 }
      }
    }
  ]
}
```

### Partial Responses
Unlike REST, a GraphQL response can include **both data and errors**. Nullable fields resolve to `null` on error; non-null errors bubble up to the nearest nullable parent.

### Custom Error Classes

```javascript
const { GraphQLError } = require('graphql');

class NotFoundError extends GraphQLError {
  constructor(resource, id) {
    super(`${resource} ${id} not found`, {
      extensions: {
        code: 'NOT_FOUND',
        http: { status: 404 },
      },
    });
  }
}

class ValidationError extends GraphQLError {
  constructor(fields) {
    super('Validation failed', {
      extensions: {
        code: 'VALIDATION_ERROR',
        fields,
      },
    });
  }
}
```

### Error Codes Convention
- `UNAUTHENTICATED` — no valid credentials
- `FORBIDDEN` — insufficient permissions
- `NOT_FOUND` — resource does not exist
- `VALIDATION_ERROR` — invalid input
- `INTERNAL_SERVER_ERROR` — unexpected failures

---

## Slide 15 — Pagination

### Offset Pagination (simple)

```graphql
type Query {
  posts(limit: Int = 10, offset: Int = 0): [Post!]!
}
# Problem: adding/removing items shifts pages
```

### Cursor-Based Pagination (recommended)

Uses an opaque cursor (encoded ID or timestamp) as a bookmark. Stable under insertions/deletions.

```graphql
type Query {
  posts(first: Int, after: String, last: Int, before: String): PostConnection!
}

type PostConnection {
  edges: [PostEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type PostEdge {
  node: Post!
  cursor: String!
}

type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}
```

### Resolver Implementation

```javascript
const resolvers = {
  Query: {
    posts: async (_, { first = 10, after }, ctx) => {
      const limit = Math.min(first, 100);
      const cursor = after
        ? Buffer.from(after, 'base64').toString()
        : null;

      const rows = await ctx.db.query(
        `SELECT * FROM posts
         ${cursor ? 'WHERE id > $2' : ''}
         ORDER BY id ASC LIMIT $1`,
        cursor ? [limit + 1, cursor] : [limit + 1]
      );

      const hasNextPage = rows.length > limit;
      const edges = rows.slice(0, limit).map(row => ({
        node: row,
        cursor: Buffer.from(row.id.toString()).toString('base64'),
      }));

      return {
        edges,
        pageInfo: {
          hasNextPage,
          hasPreviousPage: !!after,
          startCursor: edges[0]?.cursor,
          endCursor: edges[edges.length - 1]?.cursor,
        },
        totalCount: await ctx.db.posts.count(),
      };
    },
  },
};
```

---

## Slide 16 — Schema Design Patterns

### Interfaces

```graphql
interface Node {
  id: ID!
}

interface Timestamped {
  createdAt: DateTime!
  updatedAt: DateTime!
}

type User implements Node & Timestamped {
  id: ID!
  name: String!
  createdAt: DateTime!
  updatedAt: DateTime!
}
```

### Union Types

```graphql
union SearchResult = User | Post | Comment

type Query {
  search(term: String!): [SearchResult!]!
}

query {
  search(term: "graphql") {
    ... on User { name email }
    ... on Post { title body }
    ... on Comment { text author { name } }
  }
}
```

### Mutation Payload Pattern

```graphql
type CreateUserPayload {
  user: User
  errors: [UserError!]!
}

type UserError {
  field: String!
  message: String!
}

type Mutation {
  createUser(input: CreateUserInput!): CreateUserPayload!
}
```

### Nullable vs Non-Null
- **Non-null by default** for fields you always return
- **Nullable** when data may not exist (optional relations)
- **Caution:** non-null errors bubble up to the nearest nullable parent — too aggressive nullability can wipe entire responses

---

## Slide 17 — Performance

### The Threat: Abusive Queries

```graphql
query Evil {
  user(id: "1") {
    posts {
      author {
        posts {
          author {
            posts { title }
          }
        }
      }
    }
  }
}
```

### Query Complexity Analysis

```javascript
const { createComplexityRule } = require('graphql-query-complexity');

const server = new ApolloServer({
  typeDefs,
  resolvers,
  validationRules: [
    createComplexityRule({
      maximumComplexity: 1000,
      estimators: [
        fieldExtensionsEstimator(),
        simpleEstimator({ defaultComplexity: 1 }),
      ],
    }),
  ],
});
```

### Depth Limiting

```javascript
const depthLimit = require('graphql-depth-limit');

const server = new ApolloServer({
  typeDefs,
  resolvers,
  validationRules: [depthLimit(7)],
});
```

### Persisted Queries
Pre-register allowed queries at build time. The client sends a **hash** instead of the full query string. Blocks arbitrary queries in production.

### Caching Strategies
- **CDN / HTTP cache** — use `@cacheControl` directive
- **Response cache** — Apollo Server response cache plugin
- **DataLoader** — per-request deduplication
- **Redis** — shared cache across instances
- **Apollo Client** — normalised in-memory cache

---

## Slide 18 — Testing

### Mocking Resolvers

```javascript
const { addMocksToSchema } = require('@graphql-tools/mock');
const { makeExecutableSchema } = require('@graphql-tools/schema');

const schema = makeExecutableSchema({ typeDefs });

const mocks = {
  User: () => ({
    id: () => faker.string.uuid(),
    name: () => faker.person.fullName(),
    email: () => faker.internet.email(),
  }),
  DateTime: () => new Date().toISOString(),
};

const mockedSchema = addMocksToSchema({ schema, mocks });
```

### Schema Validation

```javascript
const { validateSchema, buildSchema } = require('graphql');

test('schema is valid', () => {
  const schema = buildSchema(typeDefs);
  const errors = validateSchema(schema);
  expect(errors).toHaveLength(0);
});
```

### Integration Testing

```javascript
const { ApolloServer } = require('@apollo/server');

describe('User queries', () => {
  let server;

  beforeAll(() => {
    server = new ApolloServer({ typeDefs, resolvers });
  });

  test('fetches user by ID', async () => {
    const res = await server.executeOperation({
      query: `query GetUser($id: ID!) {
        user(id: $id) { id name email }
      }`,
      variables: { id: '42' },
    }, {
      contextValue: { db: mockDb, loaders: createLoaders(mockDb) },
    });

    expect(res.body.singleResult.errors).toBeUndefined();
    const user = res.body.singleResult.data.user;
    expect(user.name).toBe('Alice');
  });
});
```

---

## Slide 19 — Summary & Next Steps

### Key Takeaways
- GraphQL = typed schema + declarative data fetching
- Single endpoint eliminates over/under-fetching
- SDL is the contract — schema-first design
- Resolvers map fields to data sources
- DataLoader solves the N+1 problem via batching
- Apollo Server + Client is the dominant ecosystem
- Cursor-based pagination with Relay Connection spec
- Protect with depth limiting, complexity analysis, persisted queries

### Recommended Reading
- **graphql.org** — official specification and docs
- **apollographql.com/docs** — Apollo Server & Client
- **Production Ready GraphQL** — Marc-André Giroux
- **Learning GraphQL** — Eve Porcello & Alex Banks

### Try These
- Build a full-stack app with Apollo Server + React
- Add real-time features with subscriptions
- Implement cursor pagination with a real database
- Set up schema federation across microservices
- Explore code-first schemas with Pothos or Nexus
