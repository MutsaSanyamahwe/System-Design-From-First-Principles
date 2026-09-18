# Kubernetes

## The Problem

Containers solve an important problem.

We can package an application together with its dependencies and run it in an isolated environment.

For example, suppose we have our application:

```
Application
     │
     ├── Frontend Container
     ├── Backend Container
     └── Database Container
```

On one machine, this is manageable.

But real applications usually grow.

A company might eventually have:

```
3 containers
     ↓
50 containers
     ↓
500 containers
     ↓
5,000 containers
```

And these containers may be running across many machines:

```
3 containers
     ↓
50 containers
     ↓
500 containers
     ↓
5,000 containers
```

Now we have a new problem.

**Managing containers manually becomes difficult.**

Suppose our backend is running these containers.

> One container crashes.

> Who notices?

> Who starts the replacement?

> What if an entire server fails?

> What if traffic suddenly increases?

> How do we update containers without taking the application offline?

We could manage all of this manually.

But eventually the amount of operational work becomes too large.

We need something that can manage containers for us.

---

## The Question

We already solved:

> How do we isolate and package an application?

Containers.

Now we need to solve:

> How do we manage large numbers of containers across machines?

This is the problem Kubernetes addresses.

---

## What Is Kubernetes?

Kubernetes is a container orchestration platform.

Its job is to manage containerized applications across a group of machines.

Instead of manually telling individual servers what to run, we describe what we want.


For example:

```
I want:

3 instances of my backend
2 instances of my frontend
1 database
```

Kubernetes then works to make the actual system match the description.

This idea is extremely important.

---

## Desired State vs Actual State

Imagine we tell kubernetes:

```
Desired State:

Backend
3 instances
```

Initially:

```
Desired State → 3
Actual State  → 3
```

Everything is fine.

Now one backend container crashes.

The system becomes:

```
Desired State → 3
Actual State  → 2
```

Kubernetes detects the difference.

It can create another instance:

```
Desired State → 3
Actual State  → 3
```
The important idea is that Kubernetes continuously works to make:

```
Actual State = Desired State
```

This is one of the fundamental ideas behind Kubernentes.

---

### Why Is This Useful?

Without Kubernetes, we might have to manually perform operations such as:

```
Check container
     ↓
Detect failure
     ↓
Find available machine
     ↓
Start replacement
     ↓
Connect it to the application
     ↓
Make sure traffic reaches it
```

With Kubernetes, we describe the desired system.

Kubernetes handles much of this operational work automatically.

---

## What Is a Kubernetes Cluster?

A Kubernetes environment is called a cluster.

A cluster consists of multiple machines called nodes.

For example:

```
Kubernetes Cluster

┌─────────────────────────────────┐
│                                 │
│        Control Plane            │
│                                 │
└─────────────────────────────────┘
                 │
       ┌─────────┼─────────┐
       │         │         │
       ▼         ▼         ▼
   Worker 1  Worker 2  Worker 3
```

The machines that run applications are called worker nodes.

The control plane manages the overall cluster.

---

## The Control Plane

The control plane is responsible for managing the Kubernetes cluster.

Conceptually:

```
                 Control Plane
                      │
          ┌───────────┼───────────┐
          │           │           │
          ▼           ▼           ▼
       Desired     Scheduling   Cluster
        State                    State
          │
          ▼
       Worker Nodes
```

The control plane determines things such as:

- what should be running
- where workloads should run
- whether the cluster matches the desired state
- when workloads need to be replaced
- when workloads need to be scaled

We don't need to memorize every Kubernetes control-plane component yet.

The important mental note is:

> The control plane manages the cluster.


---

## Worker Nodes

Worker nodes are the machines where application workloads actually run.

For example:

```
Worker Node 1
│
├── Backend
├── Backend
└── Frontend

Worker Node 2
│
├── Backend
├── Frontend
└── Worker

Worker Node 3
│
└── Database
```

The control plane decides what should happen.

The worker nodes provide the resources to run the workloads.

---


## But What Exactly Does Kubernetes Run?

This brings us to an important concept:

### Pods

A Pod is the smallest deployable unit on Kubernetes.

A Pod normally contains one application container.


For example:

```
Pod
│
└── Backend Container
```

But a Pod can also contain multiple closely related containers:

```
Pod
│
├── Application Container
└── Supporting Container
```

The containers inside a Pod share certain resources, such as networking.

For now, the most useful mental model is:

> A Pod is the Kubernetes unit in which containers run.


---

## Why Not Just Manage Containers Directly?

Kubernetes could have simply said:

```
Run this container.
```

But Kubernetes needs to manage more than individual containers.

It needs to manage applications as workloads.

For example:

```
Backend
│
├── Pod 1
├── Pod 2
└── Pod 3
```

We don't necessarily care about individual containers.

We care that:

> Three instances of the backend should always be running.

This leads to another important Kubernetes concept.

---

### Replicas

Suppose we want our backend to have three instances.

We can think of the desired state as:


```
Backend

3 replicas
```

Kubernetes then maintains:


```
Backend
│
├── Pod 1
├── Pod 2
└── Pod 3
```

If Pod 2 disappears:

```
Backend
│
├── Pod 1
└── Pod 3
```

Kubernetes can create another:

```
Backend
│
├── Pod 1
├── Pod 3
└── Pod 4
```

The specific Pod doesn't matter.

The desired number of replicas does.


---

## Deployments

We need a way to tell Kubernetes:

> Run this application and maintain its number of replicas.


A Deployment is one of the Kubernetes objects used for this.

Conceptually:

```
Deployment
     │
     │
     ▼
3 replicas
     │
     ├── Pod
     ├── Pod
     └── Pod
```

The Deployment manages the desired state of these application instances.

This allows Kubernetes to perform operations such as updating the application.

---


### What Happens When We Deploy a New Version?

Suppose we currently have:

```
Backend v1

Pod 1 → v1
Pod 2 → v1
Pod 3 → v1
```

We build:

```
Backend v2
```

We don't necessarily want to stop every v1 container at the same time.

That could cause downtime.

Instead, Kubernetes can perform a rolling update.

Conceptually:

```
v1 → v2

Pod 1 → v1
Pod 2 → v1
Pod 3 → v1

       ↓

Pod 1 → v2
Pod 2 → v1
Pod 3 → v1

       ↓

Pod 1 → v2
Pod 2 → v2
Pod 3 → v1

       ↓

Pod 1 → v2
Pod 2 → v2
Pod 3 → v2
```

The exact update behavior can be configured, but the fundamental idea is:

> Replace old instances with new instances gradually.


---

### What About Networking?

Suppose we have:

```
Backend

Pod 1
Pod 2
Pod 3
```

The Pods can come and go.

Their individual network identities aren't something we want users to depend on.

We need a stable way to reach the application.

This is where a Service comes in.

---

## Services

Kubernetes Service provides a stable network endpoint for a set of Pods.

Conceptually:

```
                 Service
                    │
          ┌─────────┼─────────┐
          │         │         │
          ▼         ▼         ▼
        Pod 1     Pod 2     Pod 3
```

The Service receives traffic and directs it to the appropriate Pods.

This is important because Pods are replaceable.

For example:

```
Service
   │
   ├── Pod 1
   ├── Pod 2
   └── Pod 3
```

Pod 2 crashes.

Kubernetes creates Pod 4:

```
Service
   │
   ├── Pod 1
   ├── Pod 3
   └── Pod 4
```

The application can continue using the Service rather than needing to know that Pod 2 disappeared.

---

## Scaling

Now imagine our application normally receives:

```
1,000 requests/minute
```

We might run:

```
3 backend Pods
```

But traffic increases:


```
10,000 requests/minute
```

We may need more instances.

Instead of manually starting containers:

```
3 Pods
  ↓
4 Pods
  ↓
5 Pods
  ↓
6 Pods
```
Kubernetes can scale workloads based on configured requirements.

The fundamental idea is:

> More demand can be handled by running more instances of the application.


---

## Self-Healing

One of Kubernetes' important capabilities is maintaining the desired state.

Suppose:

```
Desired:

3 backend Pods
```

But one fails:

```
Actual:

2 backend Pods
```

Kubernetes can detect the difference and create another Pod.

```
3 desired
   ↓
2 running
   ↓
Kubernetes detects the difference
   ↓
New Pod
   ↓
3 running
```

This is often described as self-healing.

It doesn't mean Kubernetes can fix every possible application problem.

It means Kubernetes can automatically react to certain infrastructure and workload failures according to the desired configuration.

---


## Docker vs Kubernetes

Docker and Kubernetes are often mentioned together, but they solve different problems.

Docker is primarily used to:

- build container images
- package applications
- run containers
- manage container environments

Kubernetes is used to:

- manage many workloads
- schedule workloads across machines
- maintain desired state
- scale applications
- provide service discovery/networking
- perform rolling updates
- recover failed workloads

A simplified mental model is:

```
Application
     │
     ▼
Docker
     │
     ▼
Container
     │
     ▼
Kubernetes
     │
     ├── Pod
     ├── Pod
     └── Pod
```

In real Kubernetes environments, the container runtime architecture is more nuanced than simply "Kubernetes runs Docker," and modern Kubernetes commonly uses runtimes such as containerd.

The important conceptual distinction remains:

> Docker helps create and run containers; Kubernetes manages containerized workloads at cluster scale.


---


## A Simple Kubernetes Architecture

Putting the maor concepts together:

```
                 Kubernetes Cluster
                        │
                 ┌──────▼──────┐
                 │ Control Plane│
                 └──────┬──────┘
                        │
          ┌─────────────┼─────────────┐
          │             │             │
          ▼             ▼             ▼
      Worker 1      Worker 2      Worker 3
          │             │             │
       ┌──┴──┐        ┌─┴──┐         └───┐
       ▼     ▼        ▼    ▼             ▼
      Pod   Pod      Pod   Pod           Pod
       │             │                   │
   Container      Container           Container
```

And applications can be exposed through Services:

```
                    Service
                       │
              ┌────────┼────────┐
              ▼        ▼        ▼
            Pod 1    Pod 2    Pod 3
```

---

## Kubernetes in a Real Application

Consider an AI-assisted data analysis system (Checkout my project DataPilot)

We might eventually have:

```
                    Users
                      │
                      ▼
                  Frontend
                      │
                      ▼
                  API Service
                      │
          ┌───────────┼───────────┐
          │           │           │
          ▼           ▼           ▼
       Analysis     Auth       Reports
        Pods         Pods        Pods
          │
          ▼
       Database
```

Instead of running one backend container manually, we could have Kubernetes maintain several backend Pods.

For example:

```

API Deployment
│
├── API Pod
├── API Pod
└── API Pod
```

And a Service provides a stable way to reach them:

```

                 API Service
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
       API Pod      API Pod     API Pod

```

If one Pod fails, Kubernetes can replace it.

If demand increases, we can increase the desired number of replicas.

If we deploy a new version, Kubernetes can perform a rolling update.

The application becomes a system that can be managed declaratively rather than a collection of containers that we operate manually.


---

## What Kubernetes Does Not Solve

Kubernetes is powerful, but it doesn't make infrastructure problems disappear.

It introduces its own complexity.

We now have concepts such as:

```

Clusters
Nodes
Pods
Deployments
Services
ConfigMaps
Secrets
Ingress
Volumes
Namespaces
Networking
Scheduling

```
There is a significant learning curve.

For a small application running on one server, Kubernetes may provide more complexity than is necessary.

For large systems with many workloads and machines, however, orchestration becomes increasingly useful.

The important lesson is:

> Use Kubernetes because the operational problem requires orchestration, not simply because Kubernetes exists.

---

## The Evolution So Far

We can now see the progression clearly.

```
Physical Machines
        │
        │ Problem:
        │ Applications interfere
        ▼
Virtual Machines
        │
        │ Problem:
        │ Every VM needs a full OS
        ▼
Containers
        │
        │ Problem:
        │ Managing many containers manually
        ▼
Kubernetes
```

Each technology addresses a problem created by the previous level.

That is the first-principles way to understand the stack.

---

## Key Takeaways

1. Kubernetes is a container orchestration platform.
2. It exists because manually managing large numbers of containers becomes difficult.
3. A Kubernetes cluster consists of multiple nodes.
4. The control plane manages the cluster.
5. Worker nodes run application workloads.
6. A Pod is the smallest deployable unit in Kubernetes.
7. Replicas allow multiple instances of an application to run.
8. Deployments manage application workloads and their desired number of replicas.
9. Services provide stable networking for Pods.
10. Kubernetes can perform scaling, self-healing, and rolling updates.
11. Kubernetes is based heavily on the idea of desired state.
12. Kubernetes continuously works to make the actual state match the desired state.
13. Kubernetes does not replace containers. It orchestrates containerized workloads.
14. Kubernetes is powerful but introduces additional complexity.
