# API Versioning

## The Problem

Imagine you build an API for an e-commerce application

Your original API returns:

```http
GET /users/42
```

```JSON
{
  "id": 42,
  "name": "Mutsa",
  "email": "mutsa@example.com"
}
```

Thousands of mobile apps and websites are already using this API.

Now you want to change the response.

Perhaps you want to replace:

```JSON
{
  "name": "Mutsa"
}
```

with:

```JSON
{
  "firstName": "Mutsa",
  "lastName": "Sanyamahwe"
}
```

The new design may be better, but existing applications still expect;

```
name
```

If you simply change the API, older applications can break.

You therefore have a problem:

> How can an API evolve without breaking the clients that already depend on it?

This is the problem **API versioning** solves.

---

## What is API Versioning?

API versioning is the practice of maintaining different versions of an API so that it can evolve while remaining compatible with existing clients.

For example:

```
/api/v1/users
/api/v2/users
```

The `v1` and `v2` represent different versions of the API contract.

The important idea is:

```
Old clients → Old API version

New clients → New API version
```

This allows the API to evolve without immediately forcing every client to update.

---

## Why do APIs Need Versions?

APIs are contracts.

Suppose a mobile application expects:

```JSON
{
  "name": "Alice"
}
```

If the server suddenly changes this to:

```JSON
{
  "firstName": "Alice",
  "lastName": "Smith"
}
```

the mobile application no longer works correctly.

This is a breaking change.

Other examples of breaking changes include:
- Removing an endpoint
- Renaming a field
- Changing the meaning of a field
- Changing required parameters
- Changing authentication requirements
- Changing the structure of a response

Versioning provides a way to introduce these changes safely.


---

## Common API Versioning Strategies

There are several ways to version an API.

The most common approaches are:
1. URL versioning
2. Query parameter versioning
3. Header versioning
4. Content negotiation

---

### 1. URL Versioning

The version appears directly in the URL.

```http
GET /api/v1/users
```

Later:

```http
GET /api/v2/users
```
Example:

```
/api/v1/products/10
/api/v2/products/10
```

**Advantages**

- Very easy to understand.
- You can immediately see which version is being used.
- It's also easy to test in a browser or with curl.

**Disadvantages**

- The URL now contains implementation/version information.
- You may also end up maintaining multiple URL structures.
- This is probably the easiest strategy to understand and implement, which is why it is very common.


---

### 2. Query Parameter Versioning

The version is included as a query parameter.


```http
GET /api/users?version=1
```
or:

```http
GET /api/users?version=2
```

**Advantages**

- The resource URL remains the same.

```
/api/users
```

**Disadvantages**

- The API version is less obvious.
- Caching can also become more complicated because the query string becomes part of the request.


---


### Header Versioning

The client specifies the version using an HTTP header.

For example:

```http
GET /api/users
Accept-Version: 2
```

The URL remains:

```
/api/users
```

but the header tells the server which API version the client wants.

**Advantages**

- Keeps URLs clean.
- The version becomes part of the request metadata rather than the resource URL.

**Disadvantages**

- It's less obvious when testing the API manually.
- A developer looking at:

```
/api/users
```
cannot immediately tell which version is being requested.

---

### 4. Content Negotiation

The client uses the HTTP Accept header to specify the representation it wants.

For example:


```http
GET /api/users
Accept: application/vnd.example.v2+json
```

The server responds with the appropriate representation.

This approach makes use of HTTP's content-negotiation mechanisms.

However, it is more complicated than simple URL versioning and is less intuitive for beginners.

---

**Comparing the Approaches**

| Strategy            | Example                             | Main Advantage                       |
| ------------------- | ----------------------------------- | ------------------------------------ |
| URL                 | `/api/v1/users`                     | Simple and obvious                   |
| Query parameter     | `/api/users?version=1`              | Keeps resource URL                   |
| Header              | `Accept-Version: 1`                 | Keeps URL clean                      |
| Content negotiation | `Accept: application/vnd...v2+json` | Uses HTTP representation negotiation |


There isn't one universally correct strategy.

The important thing is to choose a strategy and apply it consistently.

---

## Breaking vs Non-Breaking Changes

This is one of the most important concepts in API versioning.

Not every API change requires a new version.

### Non-breaking change

Suppose the API originally returns:

```JSON
{
  "id": 1,
  "name": "Alice"
}
```

You add another optional field:

```JSON
{
  "id": 1,
  "name": "Alice",
  "country": "South Africa"
}

```

Existing clients can continue using `id` and `name`.

This may not require a new version.

---

### Breaking Change

Suppose you change:

 ```JSON
{
  "name": "Alice"
}
```

to:

```JSON
{
  "firstName": "Alice",
  "lastName": "Smith"
}
```

An existing client expecting `name` could break.

That is a good candidate for a new API version.

---

##  Backward Compatibility

This leads to an important concept:

Backward compatibility means that newer versions of a system continue to work with older clients or data.

For example:

```
                     ┌── Mobile App v1
                     │
API Server ──────────┼── Web App v1
                     │
                     └── New Mobile App v2
```

The backend may temporarily support:

```
v1
v2
```

This gives clients time to migrate.

---

## Deprecation

You generally don't want to maintain old API versions forever.

Eventually, you may decide:

Version 1 will be discontinued.

You can mark it as deprecated.

For example:

```
v1 → Deprecated
v2 → Current
```

Clients are given time to migrate.

Eventually:

```
v1 → Removed
v2 → Current
```

A well-designed API should communicate:

- Which version is deprecated
- When it will be removed
- Which version clients should migrate to
- How clients can migrate

---

## API Versioning and Microservices

Consider:

```
                API Gateway
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
     Users        Orders       Payments
       v2           v1            v1
```

Different services can evolve at different speeds.

This is one of the reasons distributed systems are difficult:

> You can't assume that every component changes at the same time.

Versioning helps manage this independence.

---

## What about GraphQL

GraphQL approaches versioning somewhat differently.

Instead of creating:

```
/graphql/v1
/graphql/v2
```

GraphQL APIs often try to evolve the schema without introducing traditional versions.

For example, rather than removing:

```GraphQL
name
```

you might introduce:

```GraphQL
firstName
lastName
```

and deprecate the old field:

```GraphQL
type User {
    name: String @deprecated(reason: "Use firstName and lastName")
    firstName: String
    lastName: String
}
```

Clients can migrate gradually.

This is one of the interesting differences between REST and GraphQL.

---

## What About gRPC?

gRPC uses Protocol Buffers, which were designed with schema evolution in mind.

For example:
```proto
message User {
    int32 id = 1;
    string name = 2;
}
```

You can add a new field:

```proto
message User {
    int32 id = 1;
    string name = 2;
    string email = 3;
}
```

Older clients can generally continue working because they don't need to understand the new field.

However, breaking changes can still happen, and teams still need strategies for compatibility and service evolution.

---


## Best Practices

> Don't create a new version for every tiny change
- Versioning has a maintenance cost.
- Prefer backward-compatible changes when possible.

> Clearly document deprecated versions
- Give clients enough time to migrate.

> Have a migration strategy
- Don't just announce:
`"v1 is going away."`

Provide:

```
v1 → v2 migration guide
```

> Avoid supporting too many versions

Supporting:

```
v1
v2
v3
v4
v5
```
- can become extremely expensive.

> Think about compatibility from the beginning

- When designing an API, ask:
"How might this API need to evolve?"

---

## REST vs GraphQL vs gRPC Versioning

|                   | REST                            | GraphQL                   | gRPC                                    |
| ----------------- | ------------------------------- | ------------------------- | --------------------------------------- |
| Common strategy   | `/v1`, `/v2`                    | Schema evolution          | Protobuf evolution                      |
| Multiple versions | Common                          | Less common               | Sometimes                               |
| Breaking changes  | Usually require care/versioning | Avoid through deprecation | Avoid through compatible schema changes |
| Main concern      | Client compatibility            | Schema evolution          | Service compatibility                   |


---

## Where API Versioning Fits in System Design

You can think of versioning as part of API evolution:

```
                API Design
                    │
                    ▼
             API Contract
                    │
                    ▼
             API Evolution
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
   Non-breaking          Breaking change
       change                  │
                               ▼
                         New API Version
                               │
                               ▼
                          Migration
                               │
                               ▼
                          Old Version
                          Deprecated
                               │
                               ▼
                            Removed
```

The key lesson is:

> An API isn't just something you build. It's a contract that other systems depend on, and changing that contract requires careful evolution.
