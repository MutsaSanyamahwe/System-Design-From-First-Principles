# Virtual Machines

## The Problem

Imagine a company has one physical server.

```
Physical Server
│
├── CPU
├── RAM
├── Storage
└── Network
```

The company wants to run three applications:

```
Application A
Application B
Application C
```

One option is to install everything directly onto the server:

```
Physical Server
│
├── Operating System
│
├── Application A
├── Application B
└── Application C
```

But problems quickly appear.

**Problem 1: Applications have different requirements**
Application A might require:

```
Python 3.12
```

while Application B requires:

```
Python 3.9
```

They may also require different versions of:

- Node.js
- Java
- PostgreSQL
- System libraries

These dependencies can conflict.

---

**Problem 2: Applications can interfere with each other**

Suppose Application A consumes almost all available RAM.

```
Application A
████████████████████████ 90% RAM

Application B
Application C
```

The other applications can become slow or even crash.

---

**Problem 3: Security and isolation**

If applications share the same operating system environment, a vulnerability in one application could potentially affect other applications.

We therefore want some sort of isolation.

---

**Problem 4: Hardware is being wasted**

Suppose the physical server has:

```
32 CPU cores
128 GB RAM
```

but one application only uses:

```
4 CPU cores
16 GB RAM
```

The remaining resources may sit unused.

The question becomes:

> Can we divide one physical machine into several independent machines?


Yes.

This is the fundamental idea behind virtual machines.

---

## What Is a Virtual Machine?

A Virtual Machine (VM) is a software-based computer that behaves like a physical computer.

It has its own:

- CPU allocation
- Memory
- Storage
- Network interface
- Operating system

Multiple VMs can run on a single physical machine.

```
             Physical Server
        ┌─────────────────────────┐
        │                         │
        │       Hypervisor        │
        │                         │
        ├──────────┬──────────────┤
        │          │              │
        ▼          ▼              ▼
      VM 1       VM 2           VM 3
      Linux      Linux          Windows
        │          │              │
      App A      App B          App C
```

Each VM behaves as if it were its own computer.

---

## The Hypervisor

The component responsible for creating and managing virtual machines is called a hypervisor.

The hypervisor sits between the physical hardware and the virtual machines.

```
Applications
     │
Operating Systems
     │
Virtual Machines
     │
──────────────
 Hypervisor
──────────────
     │
Physical Hardware
```


The hypervisor manages resources such as:

- CPU
- RAM
- Storage
- Networking

and allocates them to different VMs.

---


### Type 1 Hypervisors

A Type 1 hypervisor runs directly on the physical hardware.

```
Physical Hardware
       │
       ▼
Hypervisor
       │
 ┌─────┼─────┐
 ▼     ▼     ▼
VM1   VM2   VM3
```

There isn't a traditional host operating system underneath the hypervisor.


Examples include:

- VMware ESXi
- Microsoft Hyper-V
- Xen

These are commonly used in data centers and cloud infrastructure.


---

### Type 2 Hypervisors

A Type 2 Hypervisor runs as an application on top of an existing operating system.

```
Physical Hardware
       │
       ▼
Host Operating System
       │
       ▼
Hypervisor
       │
 ┌─────┼─────┐
 ▼     ▼     ▼
VM1   VM2   VM3
```

Examples include:

- VirtualBox
- VMware Workstation
- Parallels Desktop

This is common when developers want to run another operating system on their own computer.

For example:

```
Windows
   │
VirtualBox
   │
Ubuntu VM
   │
Linux Application
```

---

## What Makes a VM Different From a Physical Machine?

A VM isn't physically separate hardware.

The hypervisor creates a virtual representation of hardware.

For example, you might create a VM with:

```
4 virtual CPUs
8 GB RAM
50 GB disk
```

The physical machine might actually have:

```
32 CPU cores
128 GB RAM
2 TB storage
```

The hypervisor maps the virtual resources to the physical resources.

```
Virtual CPU
     ↓
Physical CPU

Virtual RAM
     ↓
Physical RAM

Virtual Disk
     ↓
Physical Storage
```

---

**Virtual Machines Provide Isolation**

Suppose we have:

```
VM 1
Ubuntu
Application A

VM 2
Ubuntu
Application B
```

If Application A crashes, VM2 can continue running.

This isolation is one of the major reasons virtualization became so important.

---

## Virtual Disks

A VM needs storage.

Instead of requiring a physical disk exclusively for each VM, the VM can use a virtual disk.

For example:

```Physical Disk
──────────────────────────────
│ VM1.vdi │ VM2.vdi │ VM3.vdi │
──────────────────────────────
```

To the VM, its virtual disk appears like a normal physical disk.

---


## Virtual Networking

VMs also need network connectivity.

The hypervisor can provide each VM with a virtual network interface.

```
VM 1
  │
Virtual NIC
  │
  ▼
Virtual Network
  │
  ▼
Physical Network
```

This allows VMs to communicate:

- With each other
- With the host machine
- With the internet
- With external systems

---

## Virtual Machines and Cloud Computing

Virtualization is one of the foundational technologies behind modern cloud computing.

When you rent a virtual server from a cloud provider, you're generally not receiving an entire physical machine.

Instead, you receive a virtualized computing environment.

Conceptually:

```
Cloud Data Center

Physical Server
       │
       ▼
  Hypervisor
       │
 ┌─────┼─────┬─────┐
 ▼     ▼     ▼     ▼
VM    VM    VM    VM
```

This allows cloud providers to run many customers' workloads on their physical infrastructure.

---

## VM Images

Creating a VM manually every time would be inefficient.

Instead, you can create an image containing a preconfigured operating system and software.

For example:

```
Ubuntu
+
Python
+
Nginx
+
Application
+
Configuration
```

You can turn that into an image.

Then:

```
Image
 │
 ├──→ VM 1
 ├──→ VM 2
 ├──→ VM 3
 └──→ VM 4
```

This makes deployment much faster and more consistent.

Cloud platforms commonly use this concept.

---

## The Problem With Virtual Machines

VMs solved a major problem, but they introduced another.

Each VM typically includes a complete operating system.

Imagine running:

```
VM 1 → Ubuntu
VM 2 → Ubuntu
VM 3 → Ubuntu
VM 4 → Ubuntu
```
You now have four operating systems consuming:
- RAM
- CPU
- Disk space
- Startup time

If you need hundreds or thousands of small services, this can become expensive.

For example:

```
Physical Server
│
├── VM
│   └── Full OS
│
├── VM
│   └── Full OS
│
├── VM
│   └── Full OS
│
└── VM
    └── Full OS
```

This leads to the next problem:

> Can we isolate applications without needing a complete operating system for every application?

Yes.

That is where containers come in.

---

## Virtual Machines vs Containers

**Virtual Machines**

```
Hardware
   │
Hypervisor
   │
┌──┴─────────┐
│ VM         │
│ Guest OS   │
│ App        │
└────────────┘
```

**Containers**

```
Hardware
   │
Host OS
   │
Container Runtime
   │
┌────┬────┬────┐
│ C1 │ C2 │ C3 │
│App │App │App │
└────┴────┴────┘
```

Containers share the host operating system's kernel, while VMs generally run their own guest operating system.

This makes containers much lighter and faster to start.

---

|                 | Virtual Machine   | Container             |
| --------------- | ----------------- | --------------------- |
| Isolation       | Strong            | Process-level         |
| Guest OS        | Yes               | No separate guest OS  |
| Startup         | Slower            | Usually much faster   |
| Resource usage  | Higher            | Lower                 |
| Size            | Larger            | Smaller               |
| Kernel          | Own guest kernel  | Shares host kernel    |
| Best suited for | Full OS isolation | Application packaging |


Neither completely replaces the other.

Modern infrastructure often uses both.

For example:

```
Physical Server
      │
      ▼
Virtual Machine
      │
      ▼
Container Runtime
      │
 ┌────┼────┐
 ▼    ▼    ▼
 C1   C2   C3
```

This is a very common cloud architecture.



