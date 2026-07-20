# Load Balancers

## The Problem

Imagine you've built a successful application.

Initially, one server was enough.


```text
Users

↓

Application Server
```

As your application becomes more popular, thousands or even millions of users begin accessing it simultaneously.

Soon, problems appear.

- The server becomes overloaded.
- Response times increase.
- Users experience slow page loads.
- If the server crashes, your entire application becomes unavailable.

One solution might be to buy a large, more powerful server.

Eventually, even the largest server has limits.

Instead of relying on a single machine, engineers distribute traffic across multiple servers.

The challenge then becomes:

> **How do we decide which server should handle each incoming request?**

That problem led to the **Load Balancer**.

---

## What is a Load Balancer?

A **Load Balancer** is a component that distributes incoming requests across multiple backend servers.

Instead of every client connecting to a single server, they connect to the load balancer.

The load balancer decides which backend server should process each request.

```text
              Users
                 │
                 ▼
          Load Balancer
          ┌─────┼─────┐
          ▼     ▼     ▼
      Server1 Server2 Server3
```

This prevents any one server from becoming overwhelmed.

---

## How a Load Balancer Works

Suppose three users send requests at the same time.

Instead of all requests reaching one server:

The load balancer distributes them.

```text

Users

↓

Server 1
```

The load balancer distributes them.

```text
User A

↓

Server 1

User B

↓

Server 2

User C

↓

Server 3
```

Each server handles part of the workload.

---

## Why Not Let Users Choose a Server?

Imagine telling users:

```text
Use Server 1

or

Use Server 2

or

Use Server 3
```

Problems include:

- Users wouldn't know which server is available.
- Some servers would receive more traffic than others.
- Failed servers would still receive requests.
- Managing server addresses would become difficult.

A load balancer solves these problems automatically.

---

## Load Balancing Algorithms

A load balancer uses algorithms to decide where to send requests.

### Round Robin

Requests are distributed one after another.

```text
Request 1 → Server 1

Request 2 → Server 2

Request 3 → Server 3

Request 4 → Server 1
``` 


Simple and commonly used.

### Least Connections

Requests are sent to the server handling the fewest active connections.

Useful when requests take different amounts of time.

---

### Weighted Round Robin

More powerful servers receive more requests.

Example:

```text
Server 1 (Weight 3)

Server 2 (Weight 2)

Server 3 (Weight 1)
```

The stronger server handles more traffic.

---

## Wealth Checks

A load balancer continuously checks whether servers are healthy.

Suppose Server 2 crashes. The load balancer automatically stops sending traffic to Server 2.

Users continue using the application without noticing.

---

## Why Load Balancers Matter

Without a load balancer:

- One server becomes overloaded.
- Downtime affects every user.
- Scaling is difficult.

With a load balancer

- Traffic is distributed evenly.
- Applications become scalable.
- Failed servers are removed automatically.
- High availability improves.

---

## Reverse Proxy vs Load Balancer

These concepts are closely related but solve different problems

| Reverse Proxy | Load Balancer |
|---------------|---------------|
| Routes requests to backend services | Distributes requests across multiple servers |
| Often routes by URL or hostname | Often routes based on balancing algorithms |
| Hides backend infrastructure | Improves scalability and availability |
| Can terminate HTTPS | Can also terminate HTTPS |


> "If a Reverse Proxy can route requests, why do we also need a Load Balancer?"



> Think of it this way:

> A Reverse Proxy decides which service should receive a request (for example, /api → Express, /images → Image Server).
> A Load Balancer decides which instance of that service should receive the request (for example, Express Server 1, 2, or 3).

> In production systems, a single tool like NGINX or HAProxy is often configured to perform both roles at the same time.


Many modern tools, such as **NGINX**, can act as both a reverse proxy and a load balancer.

The difference lies in **how they are configured**, not the software itself.

---

# Real-World Example

Suppose PawPal has three API servers.

```text
                  Users
                     │
                     ▼
             Load Balancer
                     │
      ┌──────────────┼──────────────┐
      ▼              ▼              ▼
 API Server 1   API Server 2   API Server 3
      │              │              │
      └──────────────┴──────────────┘
                     ▼
                 Database
```

If one API server becomes unavailable, the load balancer routes traffic to the remaining healthy servers.

Users continue using the application with little or no interruption.

---

# Why Load Balancers Matter in System Design

As applications grow, one server is rarely enough.

Load balancers make horizontal scaling possible by allowing multiple servers to work together as though they were a single system.

Without load balancers, many of today's large-scale applications wouldn't be able to handle millions of users.

---

# Key Takeaways

- A Load Balancer distributes incoming traffic across multiple backend servers.
- It improves scalability by preventing a single server from becoming overloaded.
- It improves availability by avoiding failed servers.
- Common algorithms include Round Robin, Least Connections, and Weighted Round Robin.
- Load balancers and reverse proxies are related but solve different problems.

---

## Hands-On

Now that you understand why load balancers exist, it's time to build one.

 **Next:** [Load Balancing with NGINX](../hands-on/Load-Balancer-NGINX-in-practice.md)

In the hands-on guide, you'll learn how to:

- Run multiple backend servers
- Configure NGINX as a load balancer
- Implement Round Robin load balancing
- Perform health checks
- Observe how requests are distributed across servers








