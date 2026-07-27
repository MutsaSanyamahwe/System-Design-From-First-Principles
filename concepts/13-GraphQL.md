# GraphQL

## The Problem

Imagine you're building Instagram.

The home page needs:
- User information
- Profile picture
- Recent posts
- Number of followers

Using a traditional REST API, you might need multiple requests.

```text
GET /users/42

GET /users/42/posts

GET /users/42/followers
```

Or perhaps one endpoint returns everything:

```
GET /users/42
```

but the response includes much more than you need.

```text
{
  "id": 42,
  "username": "mutsa",
  "email": "...",
  "phone": "...",
  "address": "...",
  "posts": [...],
  "followers": [...],
  "notifications": [...],
  "messages": [...]
}
```


The mobile app only wanted:

- username
- profile picture
- followers

Everything else was unnecessary.

This is called over-fetching.

Sometimes the opposite happens.

The client needs data from several endpoints.

```text
/users/42

/posts

/comments

/likes
```

Now the application makes multiple network requests.

This is called under-fetching.

As applications became more complex, especially mobile apps with slower networks, developers wanted a way for clients to request exactly the data they needed.

That solution became GraphQL.

---

## What is GraphQL?

GraphQL is a query language for APIs and a runtime for executing those queries.

Instead of the server deciding what data to return, the client specifies exactly what it wants.

GraphQL was developed by Facebook (now Meta) and released as open source in 2015.

Unlike REST, which exposes many endpoints, GraphQL typically exposes a single endpoint.

```text
POST /graphql
```

---

## How GraphQL Works

The client sends a query describing the required data.

```GraphQL
query {
  user(id: 42) {
    username
    followers
    profilePicture
  }
}
```

The server returns exactly those fields.

```text
{
  "data": {
    "user": {
      "username": "mutsa",
      "followers": 180,
      "profilePicture": "profile.jpg"
    }
  }
}
```

Nothing more.

Nothing less.


---

### A Single Endpoint

Most GraphQL APIs expose only one endpoint.

```text
POST /graphql
```

The request itself describes what should happen.

Unlike REST:

```text
GET /users

GET /orders

GET /products
```

GraphQL always sends queries to the same endpoint.

---

### Queries

Queries to retrieve data.

Example:

```text

query {
  products {
    id
    name
    price
  }
}

```

Response:

```text
{
  "data": {
    "products": [
      {
        "id": 1,
        "name": "Laptop",
        "price": 900
      }
    ]
  }
}
```

Think of queries as the GraphQL equivalent of HTTP GET.


---

### Mutations

Mutations change data.

Creating a user:

```text
mutation {
  createUser(name: "Alice") {
    id
    name
  }
}
```

Deleting a product:

```text
mutation {
  deleteProduct(id: 15)
}
```

Think of mutations as REST's:
- POST
- PUT
- PATCH
- DELETE


---

### Schemas

Every GraphQL API defines a schema.

A schema describes:

- Available data
- Relationships
- Supported queries
- Supported mutations

Example:

```GraphQL
type User {
  id: ID!
  username: String!
  followers: Int!
}
```

The schema acts as a contract between the client and server.

---

### Resolvers

Resolvers are functions that fetch the requested data.

Example:


```text
Client

↓

GraphQL Query

↓

Resolver

↓

Database
```

If a client asks for:

```GraphQL
user {
  username
}
```

the resolver retrieves only the username.

Resolvers connect GraphQL queries to your application's business logic.

---

## Advantages

- **Clients request exactly what they need**; No unnecessary data.
- **Strongly typed**; The schema validates requests before execution.
- **Self-documenting**; Developers can explore the API directly through tools like GraphiQL or Apollo Sandbox.

---

## Limitations

GraphQL introduces new challenges.

**More complex server**

Resolvers, schemas, and query execution require more work than a simple REST endpoint.

**Expensive Queries**

Clients can request deeply nested data.

Example:

```GraphQL
users {
  posts {
    comments {
      author {
        posts {
          comments {
            ...
          }
        }
      }
    }
  }
}
```

Without limits, these queries become very expensive.

**Caching is Harder**

REST naturally uses HTTP caching because resources have unique URLs.

```text
GET /users/42
```

GraphQL usually sends everything to:

```text
POST /graphql
```

making traditional HTTP caching more difficult.


---

## REST vs GraphQL

| REST                         | GraphQL                             |
| ---------------------------- | ----------------------------------- |
| Multiple endpoints           | Single endpoint                     |
| Server decides returned data | Client chooses returned data        |
| Can over-fetch               | Avoids over-fetching                |
| Can under-fetch              | Retrieves related data in one query |
| Simpler to implement         | More complex                        |
| Excellent HTTP caching       | More challenging caching            |


---

## Where GraphQL Excels

GraphQL works especially well when:

- Mobile applications need to minimize network usage.
- Multiple frontend teams require different data.
- Applications have deeply related data.
- Clients frequently change their data requirements.

---

## Real-World Usage

Many large technology companies use GraphQL, including:
- Facebook
- GitHub
- Shopify
- Airbnb
- Pinterest

Many organizations use GraphQL internally while still exposing REST API's publicly.

---

## Where GraphQL Fits in System Design

```text
Client
      │
GraphQL Query
      │
GraphQL Server
      │
Resolvers
      │
Business Logic
      │
Databases / Services
```

GraphQL sits between clients and backend services, allowing clients to request exactly the data they need while the server coordinates fetching it from one or more data sources.

---

## Should you always use GraphQL?

No.

GraphQL is not a replacement for REST; it's another tool.

REST is often the better choice when:

- Your API is relatively simple.
- You benefit from straightforward HTTP caching
- You want an API that's easy to build, understand, and maintain.

GraphQL is often the better choice when:
- Clients have diverse data requirements.
- Reducing network requests is important.
- Your data is highly connected.
- You want clients to have flexibility over the data they receive.

A good system designer understands the trade-offs and chooses the approach that best fits the problem rather than assuming one is always superior.
