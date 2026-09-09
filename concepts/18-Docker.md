# Docker

## The Problem

Containers solve the problem of packaging and isolating applications, but we still need a practical way to create, run, distribute, and manage containers.

Without Docker, you would have to manually deal with things like:

- Creating container environments
- Installing dependencies
- Configuring the filesystem
- Setting environment variables
- Starting processes
- Reproducing the same environment on another machine

This creates the classic problem:

> "It works on my machine."


An application may work locally but fail on another machine because the environment is different.

Different machines may have:

- Different OS versions
- Different library versions
- Different runtime versions
- Different system dependencies
- Different configuration

**The core problem Docker solves**

> How do we package an application together with its dependencies and run it consistently anywhere?

---

## The Key Idea

Docker provides a standardized way to build, distribute, and run containers.

Instead of saying:

> "Install Python 3.12, install these 15 packages, configure these environment variables, then run this command."

We can say:

> "Here is an image containing everything required to run this application."

That image can then be run as a container.

The basic relationship is:

```
Dockerfile
    ↓
 Docker Image
    ↓
 Container
```

---

## Dockerfile

A Dockerfile is a set of instructions describing how to build an image.

For example:

```
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY . .

CMD ["python", "main.py"]
```

Conceptually:

```
Dockerfile
    │
    │ docker build
    ↓
Docker Image
```

The Dockerfile answers:

> "How should this application environment be constructed?"

---

## Docker Image

An image is a packaged, immutable template containing what is needed to create a container.

It can contain:

- Application code
- Runtime
- Libraries
- System dependencies
- Files
- Configuration defaults

Think of an image as a blueprint.


```
            Docker Image
          ┌───────────────┐
          │ Application   │
          │ Dependencies  │
          │ Runtime       │
          │ Filesystem    │
          └───────────────┘
                  │
          ┌───────┴───────┐
          ↓               ↓
     Container 1     Container 2
```

One image can therefore create many containers.

---

## Container

A container is a running instance of an image.

```
Image
  │
  ├──→ Container A
  ├──→ Container B
  └──→ Container C
```

The image is the packaged environment.

The container is the running process created from that environment.

This distinction is extremely important.

> Image = blueprint/package
> Container = running instance

---

Docker vs Containers

Docker is not the same thing as containers.

Containers are an OS-level isolation mechanism.

Docker is a platform/tooling ecosystem that makes working with containers much easier.

Docker provides tools for:

```
Build
  ↓
Package
  ↓
Distribute
  ↓
Run
  ↓
Manage
```

containers.

Historically, Docker made containers accessible to ordinary developers by providing a convenient developer experience around the underlying Linux container mechanisms.

---

## How Docker actually works

When you run:

```Bash
docker run my-app
```
Conceptually, Docker does something like:

```
Docker CLI
    │
    ↓
Docker Engine
    │
    ↓
Find image
    │
    ↓
Create container
    │
    ↓
Configure isolation
    │
    ↓
Start process
```

The Docker Engine is responsible for managing containers.

The Docker CLI is the interface you use to communicate with it.

---

## Docker Architecture

A simplified view:

```
              Docker CLI
                  │
                  ↓
           Docker Engine
          ┌───────┴────────┐
          │                │
          ↓                ↓
       Images          Containers
          │                │
          ↓                ↓
       Registry         Processes
```

The CLI sends commands to the Docker Engine.

The Engine manages:

- Images
- Containers
- Networks
- Volumes
- Container lifecycle

---

## Docker Registry

If Docker solves the problem of packaging applications, we also need somewhere to store and distribute those packages.

That's where a registry comes in.

For example:

Docker Hub is a container registry.

The flow becomes:

```
Developer
    │
    │ docker build
    ↓
Docker Image
    │
    │ docker push
    ↓
Container Registry
    │
    │ docker pull
    ↓
Another Machine
    │
    │ docker run
    ↓
Container
```

This gives us a portable way of distributing applications.

---

## Docker Layers

Docker images are generally built in layers.

For example:

```
FROM python:3.12-slim

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY . .
```
Conceptually:

```
┌─────────────────────────┐
│ Application code        │ ← Layer
├─────────────────────────┤
│ Python dependencies     │ ← Layer
├─────────────────────────┤
│ Python runtime          │ ← Layer
├─────────────────────────┤
│ Base filesystem         │ ← Layer
└─────────────────────────┘
```

Why?

Because layers can be **reused and cached.**

If you change only your application code, Docker doesn't necessarily need to rebuild everything from scratch.

This makes builds faster and saves storage/bandwidth.

---

## Docker and the Linux Kernel

This connects directly to what we learned about containers.

Docker containers do not contain a complete operating system in the way a VM does.

Instead, containers share the host's kernel.

```
Virtual Machines

┌───────────┐ ┌───────────┐
│ App       │ │ App       │
│ Guest OS  │ │ Guest OS  │
│           │ │           │
└─────┬─────┘ └─────┬─────┘
      │             │
      └──────┬──────┘
             ↓
       Hypervisor
             ↓
       Host Hardware
```

Containers:

```
┌───────────┐ ┌───────────┐
│ App       │ │ App       │
│ Libraries │ │ Libraries │
└─────┬─────┘ └─────┬─────┘
      │             │
      └──────┬──────┘
             ↓
        Host Kernel
             ↓
          Hardware
```

Docker therefore relies on the underlying operating system's container primitives.

On Linux, these include mechanisms such as:

- Namespaces -> isolation
- cgroups    -> resource control
- Union/ overlay filesystems -> layered filesystems

---

## Docker Networking

Containers often need to communicate.

For example:

```
Frontend
    │
    ↓
Backend
    │
    ↓
Database
```

Docker provides networking mechanisms that allow containers to communicate with each other.

Instead of manually configuring every network interface, Docker can create a network:

```Bash
docker network create my-network
```

Then containers can be attached to it.

This becomes especially useful when building multi-container applications.

---

## Docker Volumes

Containers are designed to be disposable.

But sometimes data must survive when containers are deleted.

For example:

```
Container
    │
    ↓
Database
```

If the database's data exists only inside the container, deleting the container can destroy that data.

Docker provides volumes for persistent storage.

```
Container
    │
    ↓
Volume
    │
    ↓
Persistent Data
```

So:

> Container lifecycle is not the same as data lifecycle

This distinction becomes very important when you start studying databases and distributed systems.

---

## Docker Compose

Real applications rarely consist of only one container.

You might have:

```
Frontend
Backend
PostgreSQL
Redis
```

Running all of these manually would become annoying.

Docker Compose allows you to define a multi-container application declaratively.

For example:

```YAML
services:
  backend:
    build: .
  
  database:
    image: postgres
```


Then:

```Bash
docker compose up
```

can start the application stack.

Conceptually:

```
             Docker Compose
                   │
       ┌───────────┼───────────┐
       ↓           ↓           ↓
   Frontend     Backend     Database
   Container    Container   Container
```

---

## What Docker Really Gives Us

Docker isn't valuable simply because it runs containers.

Its bigger value is standardization.

Without Docker:

```
Developer A
   ↓
"My environment"

Developer B
   ↓
"Different environment"

Production
   ↓
"Another environment"
```


With Docker:

```
             Docker Image
                  │
        ┌─────────┼─────────┐
        ↓         ↓         ↓
      Dev       Test      Production
```

The same packaged application environment can move through the development pipeline.

---

## Mental model

```
VM
│
├── Solves: hardware isolation
│
↓
Container
│
├── Solves: process/environment isolation
│
↓
Docker
│
├── Solves: packaging + building + distributing + running containers
│
↓
Kubernetes
│
└── Solves: operating many containers at scale
```
