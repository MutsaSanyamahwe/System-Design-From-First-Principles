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
 
