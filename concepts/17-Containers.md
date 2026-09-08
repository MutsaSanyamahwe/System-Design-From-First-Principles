# Containers

## The Problem

In the previous chapter, we saw how virtual machines allow us to divide one physical machine into several independent machines.

```
Physical Server
│
├── VM 1
│ └── Application A
│
├── VM 2
│ └── Application B
│
└── VM 3
└── Application C
```

Each VM has its own operating system.

This provides strong isolation.

However, we identified a problem.

Each VM needs a complete operating system.

Imagine we have a server running ten small applications:


```
Physical Server
│
├── VM 1
│ ├── Operating System
│ └── Application A
│
├── VM 2
│ ├── Operating System
│ └── Application B
│
├── VM 3
│ ├── Operating System
│ └── Application C
│
├── ...
│
└── VM 10
├── Operating System
└── Application J
```

The application might only need a small amount of CPU and memory.

But every VM still needs an entire operating system.

This consumes:

- RAM
- CPU
- Disk space
- Startup time


> Can we isolate applications without running a complete operating system for every application?

Yes.

This is the fundamental idea behind containers.

---

## What is a container?

A container is an isolated environment in which an application and its dependencies can run.

For example:

```Container
│
├── Application
├── Dependencies
├── Libraries
└── Filesystem
```


The important difference is that a container does not normally contain its own complete operating system.

Instead, containers share the host operating system's kernel.

Conceptually:

```
Physical Server
│
├── Host Operating System
│
├── Container
│ └── Application A
│
├── Container
│ └── Application B
│
└── Container
└── Application C
```

This allows multiple isolated applications to run on the same operating system.

---

## Why Is sharing the Kernel Important?

An operating system has two major parts we should think about:

```
Applications
│
▼
Operating System
│
├── User Space
│
└── Kernel
```

The **kernel** is responsible for managing underlying resources of the machine.

It manages things such as:

- CPU
- Memory
- Processes
- Devices
- Networking
- Filesystems

Applications communicate with these resources through the operating system.

With virtual machines, each VM has its own operating system and therefore its own kernel.

```
Physical Server
│
├── VM 1
│ ├── Operating System
│ │ └── Kernel
│ └── Application A
│
├── VM 2
│ ├── Operating System
│ │ └── Kernel
│ └── Application B
│
└── VM 3
├── Operating System
│ └── Kernel
└── Application C
```


With containers, the applications share the host kernel.

```
Physical Server
│
├── Host Operating System
│ └── Kernel
│
├── Container 1
│ └── Application A
│
├── Container 2
│ └── Application B
│
└── Container 3
└── Application C
```


This is why containers can be much lighter than virtual machines.

---

## A container Is Not a Virtual Machine

It is tempting to think of a container as a small virtual machine.

But this is not quite correct.

A VM behaves like a separate computer.

A container is closer to an isolated process environment.

For example, suppose the host is running:

```
Host
│
├── Browser
├── Database
├── Backend
└── Container
└── Application
```

The application inside the container is still ultimately running on the host machine.

The difference is that the operating system provides isolation around the application.

The container can have its own:

- processes
- filesystem
- network environment
- hostname
- resource limits

- while still sharing the host kernel.

---

## How Do Containers Provide Isolation?

If containers are sharing the same operating system, we need a way to stop applications from simply seeing and interfering with everything else.

For example:

```
Container A
│
▼
Can it see Container B's processes?

Can it modify Container B's files?

Can it consume all available memory?

Can it access the host's network?
```

The operating system needs mechanisms to provide these boundaries.

On Linux, containers rely heavily on features such as:

- Namespaces
- Control groups (cgroups)

---

## Namespaces

The first problem is visibility.

Suppose the host is running many processes:

```
Host

PID 1
PID 2
PID 3
PID 4
PID 5
...
PID 500
```


If every application could see every process, we would not have much isolation.

Namespaces allow the operating system to provide different views of system resources to different processes.

For example, a container can have its own process namespace.

Inside the container, the application might see:


```
PID 1
PID 2
PID 3
```


while the host might have hundreds of processes.

The container therefore has its own view of the process environment.

Namespaces can be used to isolate things such as:

- Processes
- Networking
- Mounts/filesystems
- Users
- Hostnames

The important idea is:

> Namespaces control what a process can see.

---

## Control Groups

Now consider a different problem.

Suppose the server has:

```
16 GB RAM
```


Container A starts using almost all of it:


```
Container A
████████████████████ 95% RAM
```


What happens to the other containers?

Without resource controls, one application could consume resources needed by everything else.

We therefore need another mechanism.

Linux provides **control groups**, commonly called **cgroups**.

Cgroups allow us to control and measure how much of certain resources a group of processes can use.

For example:

```
Container A

CPU → limited
Memory → 512 MB
Processes → limited
```


Now one container can be prevented from consuming unlimited resources.

This gives us a useful distinction:

```
Namespaces
↓
What can the process see?

cgroups
↓
How many resources can the process use?
```


---

## Container Filesystems

A container also needs a filesystem.

The application might expect to find:


```
/app
/usr
/etc
/tmp
```


The container can have its own filesystem view.

For example:


```
Container
│
├── /app
├── /usr
├── /etc
└── /tmp
```



This filesystem is isolated from the normal filesystem view of other containers.

However, this does not mean every container has its own physical disk.

The container runtime creates an isolated filesystem environment using the host's storage.

---

## Container Networking

Applications also need to communicate.

Suppose we have:

```
Container A
└── Backend

Container B
└── Database
```

The backend needs to communicate with the database.

Containers can have isolated network environments while still being able to communicate through container networking.

Conceptually:


```
Container A
│
▼
Container Network
│
▼
Container B
```


This becomes especially useful when applications are divided into multiple services.

For example:


```
             Network
                │
      ┌─────────┼─────────┐
      ▼         ▼         ▼
  Frontend   Backend   Database
  Container  Container  Container
```



We will explore container networking in more detail when we get to Docker.

---

## Containers Package the Application Environment

One of the biggest advantages of containers is that we can package an application together with the environment it needs.

Suppose our application requires:

```
Python 3.12
FastAPI
Pandas
NumPy
```


Instead of installing these manually on every machine, we can define the environment that the application expects.

Conceptually:


```
Application
+
Dependencies
+
Configuration
↓
Container
```

This helps reduce differences between development, testing, and production environments.

For example:


```
Developer Machine
│
▼
Container
│
▼
Testing Environment
│
▼
Production
```


The goal is to make the application environment more consistent.

---

## Why Containers Are Useful

Containers provide several advantages.

### 1. Isolation

Applications can run in separate environments.


```
Container A
└── Application A

Container B
└── Application B
```



Problems in one application are less likely to directly affect another.

---

### 2. Portability

The application and its dependencies can be packaged together.

This makes it easier to move the application between compatible environments.

---

### 3. Resource Efficiency

Containers share the host kernel instead of requiring a complete operating system for each application.

This means a single machine can generally run more containerized workloads than equivalent VM-based workloads.

---

### 4. Faster Startup

A VM needs to boot a guest operating system.

A container can start the application process directly within its isolated environment.

Therefore, containers generally start much faster than VMs.

---

### 5. Reproducibility

The application environment can be defined and packaged instead of being manually recreated on every machine.

This helps reduce the classic:

> "It works on my machine."

problem.

---

## Containers vs Virtual Machines

Now the difference should make sense from the problems they solve.

### Virtual Machines


```
Hardware
│
Hypervisor
│
┌──┴──────────┐
│ VM │
│ Guest OS │
│ Application │
└─────────────┘
```



### Containers

```
Hardware
│
Host OS
│
┌──┬──────────┬──┐
│ C1 │ C2 │ C3
│App │ App │ App
└──┴──────────┴──┘
```


| | Virtual Machine | Container |
|---|---|---|
| Main idea | Virtualize a computer | Isolate an application |
| Guest OS | Yes | No separate guest OS |
| Kernel | Own guest kernel | Shares host kernel |
| Isolation | Strong | Process-level |
| Resource usage | Higher | Lower |
| Startup | Generally slower | Generally faster |
| Size | Larger | Smaller |
| Best suited for | Full machine isolation | Application isolation |

Neither completely replaces the other.

They solve different problems.

And they can even be used together.

For example:

```
Physical Server
│
▼
Virtual Machine
│
▼
Host Operating System
│
▼
Container Runtime
│
┌────┼────┐
▼ ▼ ▼
C1 C2 C3
```


This is common in cloud environments.

---

## The Next Problem

Containers solve an important problem.

But now imagine our application grows.

Instead of three containers, we have:


```
3 containers
↓
50 containers
↓
500 containers
↓
5,000 containers
```


Now we have new problems.

What happens if a container crashes?

Who restarts it?

Where should it run?

How do we distribute containers across multiple machines?

How do we scale an application?

How do containers discover each other?

How do we perform deployments without bringing the application down?

Manually managing thousands of containers would quickly become impractical.

So we arrive at another question:

> **How can we automatically manage large numbers of containers across many machines?**

This is the problem that container orchestration systems solve.

One of the most widely used solutions is:

**Kubernetes.**

---

## Mental Model

The progression is:

```
Problem:
Applications need isolated environments
│
▼
Virtual Machines
│
│ Problem:
│ Every VM needs a complete OS
▼
Containers
│
│ Problem:
│ How do we build and manage containers?
▼
Docker
│
│ Problem:
│ How do we manage thousands of containers?
▼
Kubernetes
```


---

## Key Takeaways

- Containers isolate applications rather than virtualizing entire computers.
- Containers share the host operating system's kernel.
- Containers are therefore generally lighter than virtual machines.
- A container is better understood as an isolated process environment than as a small VM.
- Namespaces provide isolation by controlling what processes can see.
- cgroups control and measure resource usage.
- Containers can provide isolated filesystems and network environments.
- Containers help make application environments more portable and reproducible.
- Docker provides practical tooling for building and running containers.
- Kubernetes addresses the problem of managing containers at scale.

---




