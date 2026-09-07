### GraphQL — Short Note

**GraphQL** is a query language and API specification where the **client specifies exactly what data it needs**.

- Usually uses a single endpoint such as `POST /graphql`.
- **Query** → read data.
- **Mutation** → create/update/delete data.
- **Subscription** → receive real-time updates when data changes.
- GraphQL has a **strongly typed schema** defining available data and operations.
- **Resolvers** fetch the requested data from databases, APIs, microservices, etc.
- It helps reduce **over-fetching** and **under-fetching** compared with traditional REST APIs.

### GraphQL Subscription

A **Subscription** means: Keep me updated whenever this event/data changes.

Example:

```graphql
subscription {
  messageAdded {
    id
    text
  }
}
```

Instead of repeatedly polling:

```text
Client → "Any new message?"
Client → "Any new message?"
Client → "Any new message?"
```

the client subscribes once, and the server pushes updates when they occur.

Subscriptions are commonly implemented using **WebSocket** or **SSE**.

```text
Query        → Request/Response
Mutation     → Request/Response
Subscription → Long-lived connection + Server updates
```

[ChatGPT Link](https://chatgpt.com/share/6a9edd53-6314-83ee-92b3-6b1e3a67901e)
