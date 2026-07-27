# REST APIs

## The Problem

Imagine you're building an online store.

You have:
- A web application
- A mobile application
- An admin dashboard

All of them need to:
- View products
- Create orders
- Process payments

How should they communicate with the backend?

Without a standard, every application could invent its own way of requesting data.

One endpoint might expect:

```text
getProducts()
```

Another:

```text
fetch_all_products
```

Another:

```
products?id=5&type=list
```

Every developer would need to learn a different way of interacting with every service.

As applications grew larger, communication became messy.

The industry needed a common architectural style for designing APIs.

That style became REST.

---

## What is REST?

REST (Representational State Transfer) is an architectural style for designing web APIs.

Rather than defining strict rules, REST provides a set of principles for building APIs that are
- Simple
- Consistent
- Scalable
- Easy to understand

REST APIs use HTTP as their communication protocol.

## How REST Works

A client sends an HTTP request.

```text
GET /users/42
````

The server processes it.

The server returns a representation of the resource.

```text
{
    "id":42,
    "name": "Alice"
}
```

The client never accesses the database directly.

Instead:

```text
Client
   │
HTTP Request
   │
REST API
   │
Business Logic
   │
Database
```

REST acts as the interface between clients and data.

---

## Resources

Everything in REST is treated as a resource.

Examples:

```text
Users

Products

Orders

Posts

Comments
```

Resources are identified using URLs:

```text
/users

/users/5

/products

/orders/18
```

Notice that these are nouns, not verbs.

Good:

```text
GET /users

POST /orders
```

Bad:

```text
GET /getUsers

POST /createOrder
```

The HTTP method already describes the action.

---

## HTTP Methods

REST relies heavily on HTTP methods

| Method | Purpose               |
| ------ | --------------------- |
| GET    | Retrieve data         |
| POST   | Create new data       |
| PUT    | Replace existing data |
| PATCH  | Partially update data |
| DELETE | Remove data           |

Example:

```text
GET /products
```

Returns every product.

```text
POST /products
```
Creates a product.

```text
DELETE /products/15
```

Deletes product 15.

---

## Statelessness

One of REST's most important principles is statelessness.

Each request contains everything the server needs.

Example:

```text
GET /orders
Authorization: Bearer ey...
```

The server doesn't remember previous requests.

This means:

```text
Request 1

↓

Server responds

↓

Request 2

↓

Server treats it as completely independent
```

Benefits:
- Easier scaling
- Better reliability
- Load balancers can send requests to any server
- Servers don't need shared session memory.

---

## Representations

Clients don't receive the database itself.

They receive a representation of the resource.

Usually JSON.

```text
{
    "id":7,
    "username":"mutsa",
    "followers":180
}
```

Other formats are possible:
- XML
- HTML
- Plain text

JSON is the most common today.

---

## Status Codes

REST APIs communicate results using HTTP status codes

| Code | Meaning               |
| ---- | --------------------- |
| 200  | Success               |
| 201  | Created               |
| 204  | No Content            |
| 400  | Bad Request           |
| 401  | Unauthorized          |
| 403  | Forbidden             |
| 404  | Not Found             |
| 409  | Conflict              |
| 500  | Internal Server Error |


---

## URL Design

Good REST APIs have predictable URLs.

Good

```text
/users

/users/10

/users/10/orders

/products/4/reviews
```
Avoid

```text
/getUser

/fetchAllProducts

/deleteProduct
```

Use nouns instead of actions.

---

## REST Constraints

REST is based on several architectural constraints.

**Client-Server**

Clients and servers are independent.

**Stateless**

Every request is independent.

**Cacheable**

Responses can be cached when appropriate.

**Uniform Interface**
Resources follow consistent naming and behavior.

**Layered System**
Clients don't need to know whether they communicate with one server, a proxy, or a load balancer.


---

## Advantages

- Easy to understand
- Uses standard HTTP
- Supported by almost every programming language
- Highly scalable
- Works well with browsers
- Excellent tooling.

---

## Limitations

REST isn't perfect.

Sometimes clients receive more data than needed.

Example:

```text
GET /users/7
```

returns

```text
{
    "id":7,
    "name":"Alice",
    "email":"...",
    "address":"...",
    "orders":[...],
    "profilePicture":"...",
    ...
}

```

Maybe the client only needed the user's name.

This problem is called over-fetching.

Sometimes clients need data from multiple resources.

```text
/users/7

/orders

/products
```
Three requests instead of one.

These limitations are part of the reason technologies like `GraphQL` were created.


---

## REST in the Real World

Nearly every major web application exposes REST APIs.

Examples:

- GitHub API
- Stripe API
- Spotify API
- Slack API
- Most internal microservices at companies like Netflix, Uber, and Amazon

Even when companies adopt GraphQL or gRPC, REST often remains the default choice for public APIs.

## Where REST Fits in System Design

```text
User
   │
Browser / Mobile App
   │
HTTP Request
   │
API Gateway
   │
REST API
   │
Business Logic
   │
Database

```


  
