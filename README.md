# System Design from First Principles

> Understanding modern distributed systems by starting with the problems they were built to solve.

Welcome!

This repository documents my journey of learning **System Design from first principles**.

Instead of memorizing technologies or architectural patterns, the goal is to understand **why they exist**, **what problems they solve**, **when they should be used**, and **how they're applied in real-world systems**.

Every concept begins with a real-world problem, followed by the reasoning that led to the solution. Whenever possible, each topic is accompanied by a hands-on implementation to bridge the gap between theory and practice.

Throughout the repository, I use examples from applications like **Uber, Netflix, WhatsApp, Instagram, and Amazon** to connect concepts to real distributed systems.

---
## What You'll learn

### Foundations
- Client-Server Architecture
- Latency & Throughput
- Scalability
- Vertical vs Horizontal Scaling
- Availability & Reliability

### Networking
- DNS
- HTTP & HTTPS
- TCP & UDP
- CDNs
- Reverse Proxies
- Load Balancers
- API Gateways

### APIs & Communication
- REST
- GraphQL
- gRPC
- API Versioning

### Microservices & Deployment
- Monolith vs Microservices
- Virtual Machines
- Containers
- Docker
- Kubernetes

### Performance
- Caching
- Cache Invalidation
- TTL
- Eviction Policies (LRU, LFU)

### Distributed Systems
- Databases
- Replication
- Sharding
- CAP Theorem
- Consistency Models
- Message Queues
- Event-Driven Architecture

### Observability & Reliability
- Logging
- Monitoring
- Distributed Tracing
- Circuit Breakers
- Rate Limiting

### Case Studies
- Designing Uber
- Designing Netflix
- Designing WhatsApp
- Designing Instagram
- Designing YouTube

---

## Repository Structure

```
System-Design-From-First-Principles/

├── README.md
├── concepts/
├── hands-on/
├── case-studies/
├── diagrams/
└── images/

```

---


## Learning Philosophy

This repository follows one simple principle:

> **Understand the problem before learning the solution.**
Rather than memorizing technologies, focus on the engineering challenges that led to their creation.

For example:

- Why do we need a Load Balancer?
- Why isn't one database enough?
- Why do Docker and Kubernetes both exist?
- Why are message queues necessary?
- Why isn't caching as simple as storing data in memory?

By understanding the problem first, the solution becomes much easier to remember.

---

## Progress

- [x] [Client-Server Architecture](concepts/01-client-server.md)
- [x] [Latency & Throughput](concepts/02-Latency-&-throughput.md)
- [x] [Scalability](concepts/03-Scalability.md)
- [ ] DNS
- [ ] CDN
- [ ] Reverse Proxy
- [ ] Load Balancer
- [ ] API Gateway
- [ ] REST
- [ ] GraphQL
- [ ] gRPC
- [ ] API Versioning
- [ ] Virtual Machines
- [ ] Containers
- [ ] Docker
- [ ] Kubernetes
- [ ] Caching
- [ ] Message Queues
- [ ] Databases
- [ ] Replication
- [ ] Sharding
- [ ] CAP Theorem
- [ ] Event-Driven Architecture
- [ ] Designing Uber
- [ ] Designing Netflix
- [ ] Designing WhatsApp

## Hands-On

- [x] [DNS in Practice](hands-on/DNS-in-practice.md)
- [ ] Reverse Proxy with NGINX
- [ ] Load Balancer with NGINX
- [ ] Redis Caching
- [ ] Dockerizing an Application
- [ ] Kubernetes Deployment
- [ ] Python Rate Limiter
- [ ] PostgreSQL Replication
- [ ] PostgreSQL Sharding

## Case Studies

- [ ] Designing Uber
- [ ] Designing Netflix
- [ ] Designing WhatsApp
- [ ] Designing Instagram
- [ ] Designing YouTube
---

##  Contributions

This repository primarily documents my personal learning journey, but suggestions, corrections, and discussions are always welcome. If you spot an error or have a different perspective, feel free to open an issue or submit a pull request.

---

##  If you find this repository helpful

If these notes help you understand System Design, consider starring the repository. It helps others discover it and motivates me to continue improving it.

Happy learning! 


  
