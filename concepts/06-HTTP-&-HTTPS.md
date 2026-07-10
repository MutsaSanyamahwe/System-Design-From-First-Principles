# HTTP & HTTPS

## The Problem

Imagine you've typed the following into your browser:

```
https://www.amazon.com
```

DNS has already done its job.

Your browser now knows the IP address of Amazon's server.

But another problem arises.
> How does your browser actually communicate with the server?

Both the browser and the server need to agree on:
- How to request a web page.
- How to send data.
- How to indicate success or failure
- How to transfer images, videos, and files.

Without a common set of rules, every browser and every server would communicate differently, making the web impossible to standardize.

This problem led to HTTP.

---

## What is HTTP?
HTTP (HyperText Transfer Protocol) is the standard protocol used by web browsers and web servers to communicate.

It defines how requests are sent and how responses are returned.

Every time you:
- Visit a website
- Log in to an application
- Submit a form
- Load an image
- Call an API

you're almost certainly using HTTP.

---
### How HTTP Works
Suppose you visit:

```
https://example.com
```

The browser sends an HTTP request.

```
GET /
```

The server processes the request and returns an HTTP response.

```
HTTP/1.1 200 OK

<html>
...
</html>
```

Communication always follows this pattern:

```
Client

↓

HTTP Request

↓

Server

↓

HTTP Response

↓

Client
```

HTTP follows a request-response model.

The client always initiates communication.

---

### The Problem with HTTP

Traditional HTTP sends data as plain text.

Imagine logging into your bank.

Your browser sends:

```
Username: john@example.com
Password: myPassword123
```

If someone intercepts this communication, they can read everything.

Sensitive information such as:
- Passwords
- Credit card numbers
- Banking information
- Personal messages
could all be exposed.

The internet needed a secure way to communicate.

This problem led to HTTPS.

---

## What is HTTPS?

HTTPS (HyperText Transfer Protocol Secure) is HTTP protected by TLS (Transport Layer Security).

Instead of sending data as plain text. HTTPS encrypts communication between the client and the server.

This ensures that attackers cannot easily read or modify the data in transit.

---

### How HTTPS Works
Before any HTTP data is exchanged, the browser and server perform a TLS handshake.

During this process, they:
- Verify the server's identity using a digital certificate.
- Agree on encryption algorithms
- Generate shared encryption keys.

Once the secure connection is established:

```
Browser

↓

Encrypted HTTP Request

↓

Server

↓

Encrypted HTTP Response

↓

Browser


```

Anyone intercepting the traffic only sees encrypted data.

---

## HTTP vs HTTPS

| HTTP                                          | HTTPS                                  |
| --------------------------------------------- | -------------------------------------- |
| Data is sent as plain text                    | Data is encrypted                      |
| No identity verification                      | Verifies the server using certificates |
| Less secure                                   | Highly secure                          |
| Uses Port 80                                  | Uses Port 443                          |
| Suitable only for non-sensitive communication | Required for modern websites and APIs  |


---

## Why HTTPS Matters
Modern applications constantly exchange sensitive information.

Examples include:

- User logins
- Online payments
- Banking transactions
- API authentication tokens
- Personal messages

Without HTTPS, attackers could intercept or modify this information.

For this reason, nearly every modern website uses HTTPS.

---

## Key Takeaways
- HTTP defines how browsers and servers communicate.
- HTTP follows a request-response model.
- HTTP sends data in plain text.
- HTTPS adds encryption using TLS.
- HTTPS protects data confidentiality and integrity, and verifies the server's identity.
- Modern web applications should always use HTTPS.

---

## Hands-On
Understanding how HTTPS works conceptually is only half the story.

Continue to [HTTPS in Practice](../hands-on/HTTPS-in-practice.md) to learn how you transition from HTTP (development stage) to HTTPS (deployment stage). 


