# Reverse Proxy

## 1. The Problem

Imagine you've built a web application using React and Express.

Your Express server is running on:

```text
http://192.168.1.10:3000
```

Users could technically access your application directly.

But as your application grows, several problems begin to appear.

- You don't want users connecting directly to your backend server.
- You may have multiple backend services running on different ports
- You need HTTPS, but you don't want every application managing SSL certificates.
- You want to serve static files efficiently.
- You may eventually need to distribute traffic across multiple servers.

Exposing every backend service directly to the Internet quickly becomes difficult to manage and less secure.

Engineers needed a component that could sit in front of application servers, receive all incoming requests, and decide where each request should go.

That problem led to the **Reverse Proxy**.

---

## 2. What is a Reverse Proxy?

A **Reverse Proxy** is a server that sits between clients and one or more backend servers.

Instead of clients communicating directly with your application, they communicate with the reverse proxy.

The reverse proxy then forwards the request to the appropriate backend server.

```text
Client

↓

Reverse Proxy

↓

Application Server
```

To the client, it appears as though it is communicating with a single server.

The backend servers remain hidden.

---

## 3. How a Reverse Proxy Works

Suppose a user visits:

```text
https://pawpal.com
```

Instead of the request reaching your Express server directly, it first reaches the reverse proxy.

```text
Browser

↓

Reverse Proxy

↓

Express Server
```

The reverse proxy examines the request and forwards it to the correct backend.

The backend processes the request and sends the response back through the reverse proxy.

---

## 4. Routing Requests

A reverse proxy can route requests based on the URL.

Example:

```text
https://pawpal.com/

↓

React Frontend
```

```text
https://pawpal.com/api

↓

Express API
```

```text
https://pawpal.com/images

↓

Image Server
```
To the user, everything appears to come from the same website.

---

## 5.  Why Not Let Clients Connect Directly?

Without a reverse proxy:

```text
Browser

↓

Frontend :5173

Backend :3000

Images :8080
```

Clients would need to know the address of every service.

This creates several problems:

- More complex client configurations
- Reduce security
- Harder to manage certificates
- Difficult to scale services independently.

A reverse proxy can:

- Route requests to backend services
- Hide backend servers from users
- Handle HTTPS (TLS termination)
- Serve static files
- Compress responses
- Cache content
- Add security headers
-  Log incoming requests

---

## 6. Real-World Example

Suppose PawPal has:

- React Frontend
- Express API
- Authentication Service


Without a reverse proxy:

```text
Browser

↓

frontend.pawpal.com

api.pawpal.com

auth.pawpal.com
```

With a reverse proxy:

```text
Browser

↓

pawpal.com

↓

Reverse Proxy

├── /
│   └── React
│
├── /api
│   └── Express
│
└── /auth
    └── Authentication Service
```

The browser only communicates with one public endpoint.

---

## 7. Reverse Proxy vs Forward Proxy

People often confuse these two.

A **Forward Proxy** sits in front of clients.

```text
Client

↓

Forward Proxy

↓

Internet
```

Examples include:

- School networks
- Corporate networks
- VPN services

A **Reverse Proxy** sits in front of servers.

```text
Client

↓

Reverse Proxy

↓

Servers
```

Examples include:

- NGINX
- HAProxy
- Traefik
- Caddy

---

## 8. Why Reverse Proxies Matter in System Design

Almost every production system places a reverse proxy in front of its backend services.

This allows engineers to:

- Secure backend servers
- Centralize HTTPS
- Route traffic
- Improve performance
- Simplify deployments

Many other system design components are built on top of reverse proxies.

For example:

- Load Balancers
- API Gateways
- Kubernetes Ingress Controllers

---

# Key Takeaways

- A Reverse Proxy sits between clients and backend servers.
- Clients communicate with the reverse proxy instead of directly with the application.
- Reverse proxies route requests to the appropriate backend service.
- They commonly handle HTTPS, routing, caching, compression, and security.
- Nearly every modern web application uses a reverse proxy.

---


## Hands-On

Now that you understand what a reverse proxy is, it's time to configure one yourself.

 **Next:** [Reverse Proxy with NGINX](../hands-on/Reverse-Proxy-in-practice.md)

In the hands-on guide you'll learn how to:

- Install NGINX
- Configure it as a reverse proxy
- Route traffic to an Express application
- Enable HTTPS
- Understand how requests flow through the proxy


