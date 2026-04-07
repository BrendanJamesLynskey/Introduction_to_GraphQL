# ◈ Introduction to GraphQL

An interactive Reveal.js presentation covering GraphQL — schema definition, queries, mutations, subscriptions, resolvers, Apollo Server and Client, DataLoader, authentication, pagination, and performance.

## ▶ [Open the Presentation](https://brendanjameslynskey.github.io/Introduction_to_GraphQL/)

## 📄 [Markdown Version](presentation.md)

---

## Contents

| # | Topic | Description |
|---|-------|-------------|
| 01 | Title | GraphQL overview |
| 02 | Agenda | Topics at a glance |
| 03 | What Is GraphQL? | Origin at Facebook, core principles, single endpoint |
| 04 | GraphQL vs REST | Over-fetching, under-fetching, N+1, schema-first |
| 05 | Schema Definition Language | Types, fields, scalars, enums, input types, non-null |
| 06 | Queries | Field selection, nesting, arguments, aliases, fragments |
| 07 | Mutations | Create, update, delete, variables, input types |
| 08 | Subscriptions | Real-time, WebSocket transport, pub/sub pattern |
| 09 | Resolvers | Resolver functions, parent/args/context/info, resolver chain |
| 10 | Apollo Server with Express | Setup, typeDefs, resolvers, context, sandbox |
| 11 | Apollo Client | React integration, useQuery, useMutation, cache |
| 12 | DataLoaders & the N+1 Problem | Batching, caching, dataloader library |
| 13 | Authentication & Authorisation | Context-based auth, directives, field-level permissions |
| 14 | Error Handling | GraphQL error format, custom errors, partial responses |
| 15 | Pagination | Cursor-based, Relay connection spec, edges/nodes/pageInfo |
| 16 | Schema Design Patterns | Interfaces, union types, mutation payloads, nullability |
| 17 | Performance | Query complexity, depth limiting, persisted queries, caching |
| 18 | Testing | Mocking resolvers, integration testing, schema validation |
| 19 | Summary & Next Steps | Key takeaways and recommended reading |

---

## Slide Controls

| Action | Key |
|--------|-----|
| Next / Previous | `→` `←` or swipe |
| Overview | `Esc` |
| Fullscreen | `F` |
| Export to PDF | Append `?print-pdf` to URL, then print |

## Technology

[Reveal.js 4.6](https://revealjs.com) · [highlight.js](https://highlightjs.org) · Playfair Display + DM Sans + JetBrains Mono

Single self-contained `index.html` — no build step, no npm, no dependencies to install.

## References

GraphQL Foundation, *GraphQL Specification* — graphql.org · Apollo GraphQL, *Apollo Server & Client Documentation* — apollographql.com · Marc-André Giroux, *Production Ready GraphQL*, 2020 · Eve Porcello & Alex Banks, *Learning GraphQL*, O'Reilly, 2018 · Facebook, *DataLoader* — github.com/graphql/dataloader

## License

Educational use. Code examples provided as-is.
