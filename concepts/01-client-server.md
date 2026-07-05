# Client-Server Architecture

---
## 1. The problem
Imagine every computer on the internet trying to communicate directly with every other computer.  

If you wanted to order an Uber, your phone would somehow need to know:

- where every driver is,
- who is available,
- how much they charge,
- their ratings,
- whether another passenger has already booked them.

I believe this would be chaotic. Do you agree? 
Every driver's phone would also need to constantly communicate with every passenger's phone.

Very quickly, this becomes impossible.

Problems include:
- No central source of truth.
- Data becomes inconsistent.
- Security becomes difficult because every device would expose its own data.
- Devices turning off would make information disappear.
- Finding other devices becomes extremely complicated.

As applications grew larger, developers needed a central computer that could store data, process requests, and coordinate communication between users.

This problem led to Client-Server Architecture.

---



## 2. The Solution
Instead of every device talking directly to every other device, introduce a central machine called a server.

This server becomes responsible for:
- storing data
- processing requests
- enforcing business rules
- authenticating users
- communicating with databases
- coordinating all clients

Clients become much simpler.

Their job is mostly to:
- display information
- collect user input
- send requests
- display responses

Every interaction flows through the server.

![Client-Server Architecture](../Images/client-server_model_1253.webp)

---

## 3. How it works
A client is any application used by the user.

Examples include:
- web browsers
- mobile apps
- desktop applications

A server is a computer that continuously listens for incoming requests.

The communication generally follows these steps.

---
### step 1
The client sends an HTTP request.

Example:
```
GET/profile

```
---

### Step 2
The server receives the request.

It determines:
- who sent it?
- Are they authenticated?
- What resource do they want?

---

### Step 3
The server performs work.

This could involve:
-querying the database
- performing calculations
- checking permissions
- calling another service

---

### Step 4
The server sends back a response.

Example:

```

{
    "name": "Mutsa",
    "role": "Software Engineer"
}

```
---
### Step 5
The client displays the information to the user.
The client does not decide whether the user has permissions.
The server always remains the authority.

![Client-Server Architecture](../Images/client-server_model_1253.webp)





