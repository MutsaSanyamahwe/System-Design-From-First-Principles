# gRPC

## The Problem

Imagine you're building Uber.

A passenger requests a ride.

Behind the scenes, multiple services need to communicate.

- User Service verifies the passenger.
- Driver Service finds nearby drivers.
- Pricing Service calculates the fare.
- Payment Service verifies the payment method.
- Notification Service sends updates.

One user request uses JSON over HTTP; each service must:
- Convert objects to JSON
- Send relatively large text payloads.
- Parse JSON back into objects.

As the number of services grows, this overhead becomes significant.

Large distributed systems needed a faster and more efficient way for services to communicate.

That solution became gRPC.

---

## What is gRPC?

gRPC (Google Remote Procedure Call) is an open-source framework developed by Google for building high-performance APIs.

Instead of interacting with resources like REST or requesting fields like GraphQL, clients call methods on remote services almost as if they were local functions.

Example:

```text
paymentService.processPayment(order)
```

The function actually executes on another machine, but the client interacts with it as though it were local.

---

## What Problem Does gRPC Solve?

gRPC focuses on:
- Fast communication
- Efficient serialization
- Low network overhead
- Strong contracts between services

It is commonly used for:
- Microservices
- Internal company APIs
- Real-time systems
- High-performance distributed systems

---

## What Protocol Does gRPC Use?

Unlike REST and GraphQL, gRPC is tightly coupled to HTTP/2.

HTTP/2 provides several features that make gRPC much faster than traditional HTTP/1.1 APIs.

These include:
- Multiplexing multiple requests over one connection.
- Header compression
- Binary framing
- Long-lived connections

Because of HTTP/2, gRPC can process many requests simultaneously without opening a new connection each time.

---

## Data Format

REST usually exchanges JSON.

Example:

```JSON
{
  "id": 42,
  "username": "mutsa",
  "followers": 180
}
```

JSON is easy for humans to read but relatively large.

gRPC uses Protocol Buffers (Protobuf) instead.

Example:

```proto
message User {
  int32 id = 1;
  string username = 2;
  int32 followers = 3;
}
```


Before sending data across the network, Protocol Buffers serialize it into a compact binary format.

Benefits:
- Smaller payloads
- Faster serialization
- Faster parsing
- Less bandwidth usage

The binary format is not human-readable, but it is significantly more efficient.

---

## How gRPC Works

First, you define your service.

```proto
service UserService {
  rpc GetUser(UserRequest) returns (UserResponse);
}
```

Then define the request and response messages.

```proto
message UserRequest {
  int32 id = 1;
}

message UserResponse {
  int32 id = 1;
  string username = 2;
}
```

From this single file, gRPC automatically generates client and server code in many programming languages.

When the client calls:

```text
GetUser(42)
```

gRPC handles:
- Serializing the request
- Sending it over HTTP/2
- Calling the remote service
- Receiving the response
- Deserializing the response

The developer writes very little networking code.

---

### Protocol Buffers

Protocol Buffers act as the contract between services.

Example:

```proto
syntax = "proto3";

service ProductService {
  rpc GetProduct(ProductRequest) returns (ProductResponse);
}

message ProductRequest {
  int32 id = 1;
}

message ProductResponse {
  int32 id = 1;
  string name = 2;
  double price = 3;
}
```

Both client and server generate code from the same `.proto ` file, ensuring they always agree on the request and response format.

---

### Streaming

One of gRPC's biggest strengths is built-in streaming.

Unlike REST, which generally follows a request-response model, gRPC supports four communication patterns.

#### Unary RPC

One request.

One response.

```
Client ─────► Server

Client ◄───── Server
```

---

#### Server Streaming
One request.

Many responses.

```
Client ─────► Server

Client ◄───── Response 1
Client ◄───── Response 2
Client ◄───── Response 3
```

Example:

- Live stock prices
- Ride updates
- News feeds

---

#### Client Streaming

Many requests.

One response.

```
Client ─────►
Client ─────►
Client ─────►

        Server

Client ◄───── Final Response
```

Example:

Uploading a large file in chunks.

---

#### Bidirectional Streaming

Both client and server continuously exchange messages.

```
Client ◄────► Server
```

Example:

- Multiplayer games
- Chat applications
- Live collaboration

---

## Advantages

**High Performance**

Binary Protocol Buffers are significantly smaller than JSON.

**HTTP/2**

Multiplexing reduces latency and improves throughput.

**Strongly Typed**

The `.proto` file defines a clear contract.

**Automatic Code Generation**

Clients and servers can be generated for multiple languages.

**Streaming Support**

Supports real-time communication without additional protocols.


---

## Limitations

**Harder to Debug**

Binary messages cannot be read directly like JSON.

**Browser Support**

Browsers do not natively support standard gRPC. Web applications typically use gRPC-Web or communicate through a gateway.

**Less Human-Friendly**

Testing a REST API is as simple as visiting a URL or using curl.

gRPC requires specialized tooling because it uses Protocol Buffers over HTTP/2.

**Public APIs**

REST is generally easier for third-party developers to consume, so many companies expose REST APIs publicly while using gRPC internally.

---

## REST vs GraphQL vs gRPC

| Feature         | REST                | GraphQL                       | gRPC                       |
| --------------- | ------------------- | ----------------------------- | -------------------------- |
| Primary Goal    | Resource-based APIs | Flexible client queries       | Fast service communication |
| Protocol        | HTTP/HTTPS          | Usually HTTP/HTTPS            | HTTP/2                     |
| Data Format     | JSON                | Query + JSON                  | Protocol Buffers           |
| Human Readable  | Yes                 | Yes                           | No (Binary)                |
| Performance     | Good                | Good                          | Excellent                  |
| Streaming       | Limited             | Subscriptions                 | Native                     |
| Code Generation | No                  | Optional                      | Built-in                   |
| Best For        | Public APIs         | Complex frontend applications | Internal microservices     |


---

## Real-World Usage

Many technology companies use gRPC internally, including:
- Google
- Netflix
- Square
- Dropbox
- Cisco

A common architecture is:

- Public clients communicate using REST or GraphQL.
- Internal microservices communicate using gRPC.

This combines ease of use for external developers with high-performance communication inside the system.


---

## Why gRPC Fits in System Design

```text
Mobile App
        │
REST / GraphQL
        │
API Gateway
        │
────────────────────────────────
│              │              │
User Service  Payment Service  Driver Service
        │         │              │
        └────── gRPC ────────────┘
```
External clients interact with the API using REST or GraphQL, while the backend services communicate efficiently with one another using gRPC.


