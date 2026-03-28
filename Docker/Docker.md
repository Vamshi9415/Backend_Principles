# The Definitive Docker Guide: Complete Understanding From First Principles

> *"Docker isn't a set of commands to be memorized, but a system underneath to be understood."*

> **How to use this guide**: Read it top to bottom the first time. Every section builds on the previous one. The mental model constructed in sections 2 and 3 is the lens through which everything else becomes obvious. Do not skip ahead.

---

## Table of Contents

1. [Introduction: Why Most People Learn Docker Wrong](#1-introduction)
2. [The Core Mental Model: What Docker Actually Is](#2-mental-model)
3. [Containers vs Virtual Machines: The Critical Distinction](#3-containers-vs-vms)
   - 3.1 [How Operating Systems Work — The Foundation You Need](#31-how-operating-systems-work)
   - 3.2 [Namespaces: The Isolation Mechanism Explained Deeply](#32-namespaces)
   - 3.3 [Cgroups: The Resource Limiter Explained Deeply](#33-cgroups)
   - 3.4 [Virtual Machines vs Containers: A Complete Comparison](#34-vm-vs-containers)
   - 3.5 [What Happens on Mac and Windows: The Hidden VM](#35-mac-and-windows)
4. [Images and Containers: Blueprint vs Running Instance](#4-images-and-containers)
   - 4.1 [What Is a Docker Image? (Deep Explanation)](#41-docker-image)
   - 4.2 [What Is a Docker Container? (Deep Explanation)](#42-docker-container)
   - 4.3 [The Writable Layer: The Complete Story of Why Data Disappears](#43-writable-layer)
   - 4.4 [Image Immutability: Why This Is a Feature, Not a Bug](#44-image-immutability)
   - 4.5 [The Image vs Container Lifecycle](#45-lifecycle)
5. [Docker Layers: The Caching System That Governs Build Speed](#5-layers)
   - 5.1 [What Layers Are and How They Are Created](#51-what-layers-are)
   - 5.2 [How Layer Caching Works Internally](#52-layer-caching)
   - 5.3 [Cache Invalidation: The Rules That Govern Everything](#53-cache-invalidation)
   - 5.4 [Instruction Order: The Single Biggest Build Speed Lever](#54-instruction-order)
   - 5.5 [Layer Sharing Across Images](#55-layer-sharing)
   - 5.6 [How to Inspect and Debug Layers](#56-inspecting-layers)
6. [The Dockerfile: Every Instruction Explained With Full Depth](#6-dockerfile)
   - 6.1 [FROM — Your Starting Point, Your Foundation](#61-from)
   - 6.2 [WORKDIR — Setting the Stage, Not Just Changing Directory](#62-workdir)
   - 6.3 [COPY and ADD — The Difference and When to Use Each](#63-copy-and-add)
   - 6.4 [RUN — Executing Commands, Creating Layers, The Cleanup Rule](#64-run)
   - 6.5 [ENV — Environment Variables, Their Scope, and Secrets Warning](#65-env)
   - 6.6 [ARG — Build-Time Variables That Don't Persist](#66-arg)
   - 6.7 [EXPOSE — The Most Misunderstood Instruction](#67-expose)
   - 6.8 [CMD vs ENTRYPOINT — The Complete, Unambiguous Explanation](#68-cmd-vs-entrypoint)
   - 6.9 [USER — Running as Non-Root, Why It Matters](#69-user)
   - 6.10 [LABEL — Metadata for Your Images](#610-label)
   - 6.11 [VOLUME — Marking Mount Points](#611-volume)
   - 6.12 [HEALTHCHECK — Teaching Docker When Your App Is Ready](#612-healthcheck)
   - 6.13 [Build Context and .dockerignore — What Docker Can See](#613-build-context)
   - 6.14 [Multi-Stage Builds — The Pattern for Production-Ready Images](#614-multi-stage)
7. [Volumes: The Complete Solution to Persistence](#7-volumes)
   - 7.1 [The Root Cause of Data Disappearance — Deeply Explained](#71-root-cause)
   - 7.2 [Named Volumes — Docker-Managed, Lifecycle-Independent Storage](#72-named-volumes)
   - 7.3 [Bind Mounts — Direct Host Filesystem Access](#73-bind-mounts)
   - 7.4 [Anonymous Volumes — The Invisible Third Type](#74-anonymous-volumes)
   - 7.5 [Named Volumes vs Bind Mounts — The Decision Framework](#75-decision-framework)
   - 7.6 [The Volume Initialization Gotcha — Named vs Bind Behavior](#76-initialization-gotcha)
   - 7.7 [Volume Management and Backup Strategies](#77-volume-management)
8. [Networking: The Complete Picture of Container Communication](#8-networking)
   - 8.1 [Why Container Network Isolation Exists](#81-why-isolation)
   - 8.2 [What a Container's Network Namespace Contains](#82-network-namespace)
   - 8.3 [Port Mapping — Every Detail of -p Explained](#83-port-mapping)
   - 8.4 [The Three Default Docker Networks](#84-default-networks)
   - 8.5 [The Default Bridge Network Problem — Why DNS Doesn't Work](#85-default-bridge-problem)
   - 8.6 [Custom Networks — How Docker DNS Actually Works](#86-custom-networks)
   - 8.7 [Network Isolation as a Security Layer](#87-network-security)
   - 8.8 [The host and none Network Drivers](#88-host-and-none)
9. [Docker Compose: The Complete Guide to Multi-Container Apps](#9-docker-compose)
   - 9.1 [The Problem Compose Exists to Solve](#91-the-problem)
   - 9.2 [What Compose Actually Is Under the Hood](#92-what-compose-is)
   - 9.3 [Complete Anatomy of docker-compose.yml](#93-anatomy)
   - 9.4 [depends_on — Startup Order vs True Readiness](#94-depends-on)
   - 9.5 [Health Checks in Compose — The Full Picture](#95-health-checks)
   - 9.6 [Environment Variables in Compose — All the Ways](#96-environment-variables)
   - 9.7 [Volumes and Networks in Compose](#97-volumes-networks)
   - 9.8 [Overriding Compose Files for Different Environments](#98-overrides)
   - 9.9 [Compose Networking — How Services Find Each Other](#99-compose-networking)
10. [Common Mistakes — What Goes Wrong and Exactly Why](#10-common-mistakes)
11. [Troubleshooting and Debugging — The Complete Playbook](#11-troubleshooting)
12. [Best Practices — The Principles Behind the Rules](#12-best-practices)
13. [Complete Real-World Examples](#13-real-world-examples)
14. [Quick Reference Cheat Sheet](#14-cheat-sheet)

---

## 1. Introduction

### Why Most People Learn Docker Wrong

If you've ever used Docker, your first time probably went like this. You clone a project, someone tells you to "just run `docker compose up`", and either it magically works or it doesn't. And in both cases, you have no idea why. So you start memorizing commands — `docker build`, `docker run`. You copy Dockerfiles from the internet. Things mostly work until they don't, and then you feel completely lost because you don't understand what Docker is actually doing underneath.

This is the **command-first, understanding-last** trap. It's how most Docker tutorials are written, and it's why so many developers feel shaky with Docker even after months of using it. They know the commands. They don't know the system.

The symptoms of this trap look like this:

- You follow a tutorial step by step and it works — but you can't explain why any individual step is needed
- You copy a Dockerfile from StackOverflow and modify it blindly until it works
- You fix a Docker problem by randomly trying different things until something sticks
- You get a "port already in use" or "network not found" error and have no mental model for why
- You make a change inside a container, restart it, and your change is gone — and you don't understand why
- You run `docker compose down` and your database data disappears — and you're shocked

All of these symptoms come from the same cause: missing the underlying mental model. Once you have the model, the commands explain themselves. The errors become obvious. The solutions become logical.

This guide builds that model deliberately, from the ground up. We will not teach you commands first. We will teach you what Docker is, what it does, why it works the way it works — and then all the commands will follow naturally from understanding.

### The Core Questions This Guide Answers

By the end of this guide, you will be able to answer every one of these questions from first principles — not from memory:

- Why can't my container see a file I know exists on my system?
- Why did my database lose all its data when I restarted with `docker compose down` and `docker compose up`?
- Why is my Docker build taking 3 minutes when I only changed one line of code?
- Why does `EXPOSE 3000` in my Dockerfile not actually make port 3000 accessible?
- Why can my web container not reach my database container by name?
- What's the difference between `CMD` and `ENTRYPOINT`?
- Why do I keep seeing "image not found" after I build an image locally?
- Why does `depends_on: db` not actually wait for the database to be ready?

These aren't obscure corner cases. They are things every developer hits in their first weeks with Docker. They all have clean, logical answers once you understand the system underneath.

---

## 2. The Core Mental Model

### What Docker Actually Is

Docker is **the tooling that makes it easy to set up isolation walls around a process** — to define what a process can see, what files it has access to, and what resources it can use.

That is the entire idea. Everything else in Docker — images, containers, volumes, networks, Compose — is a tool for expressing and managing those isolation walls.

Let's be specific about what "Docker" actually is as a piece of software:

- A **daemon** (`dockerd`) that runs on your machine as a background service, managing containers, images, networks, and volumes
- A **CLI** (`docker`) that you use to talk to that daemon
- A **set of APIs** that the CLI uses to communicate with the daemon
- A **container runtime** (via `containerd` and `runc`) that actually creates and runs containers using Linux kernel features

When you type `docker run myapp`, the CLI sends a message to the daemon, which tells the container runtime to create an isolated process using Linux kernel features. The daemon manages the lifecycle of that process, its network, its filesystem, and its resource limits.

### The Single Most Important Mental Shift

> **A container is a process, not a machine.**

This is the mental shift that changes everything. Say it again: a container is a **process**, not a machine. Not a mini-computer. Not a virtual server. A process — the same kind of thing as when you run `node server.js` or `python app.py` from your terminal.

The difference between a regular process and a container is that the container's process runs with **isolation features turned on**. It sees its own filesystem. It has its own network. It has its own process tree. But at the end of the day, it's still just a process running on your machine, managed by your operating system's kernel.

Why does this matter so much? Because once you think of containers as processes, the behavior you'd otherwise find confusing becomes completely obvious:

| Confusing Behavior | Obvious When Seen As Process |
|---|---|
| Container can't see my files | Processes only see what the OS gives them access to |
| Data disappeared when container was removed | Processes don't have permanent storage — their state lives in memory and temp space |
| Container starts in milliseconds | Starting a process is fast — booting an OS is slow |
| Containers share the host's disk | Processes share the same underlying hardware |
| Container uses your machine's resources | Processes run on the host machine's CPU and RAM |

Write "a container is a process" on a sticky note. Keep it next to your monitor while you learn Docker. Every time something confuses you, ask yourself: "does this make sense if a container is just an isolated process?"

---

## 3. Containers vs Virtual Machines

### The Critical Distinction

Docker is not a virtual machine. It is not running a separate operating system for each container. This is the most common and most damaging misconception about Docker. It leads developers to treat containers like they treat VMs — giving them gigabytes of memory, treating them like persistent machines, being surprised they start fast — and all of that leads to confusion.

Let's understand the distinction deeply, from the ground up.

### 3.1 How Operating Systems Work — The Foundation You Need

To understand why containers are fundamentally different from VMs, you need a working model of how an operating system is structured.

Every operating system is divided into two major parts:

**Part 1: The Kernel**

The kernel is the core of the operating system. It runs in a privileged execution mode (called "kernel mode" or "ring 0" on x86 CPUs) and has direct access to hardware. The kernel is responsible for:

- **Memory management**: Allocating, freeing, and protecting regions of RAM for different processes
- **Process scheduling**: Deciding which process runs on which CPU core at which moment
- **Filesystem access**: Reading and writing files on disk, managing filesystem structures
- **Network stack**: Sending and receiving packets, managing sockets, routing
- **Hardware drivers**: Communicating with physical devices (disks, network cards, graphics)
- **System calls**: Providing a controlled interface for user-space programs to request kernel services

The kernel is always running. It is the first thing that boots and the last thing that shuts down.

**Part 2: Userspace**

Userspace is everything that runs on top of the kernel. Every application you use — your text editor, your web browser, your Python interpreter, your web server — runs in userspace. These programs run in "user mode" (ring 3) and do not have direct access to hardware.

When a userspace program needs to do something privileged — open a file, create a socket, allocate memory — it makes a **system call**. A system call is like the program saying "excuse me, kernel, could you please open this file for me?" The kernel checks if the program is allowed to do that, does the operation, and returns the result.

```
┌─────────────────────────────────────────┐
│           User Applications             │
│   node server.js  |  python app.py      │  ← Run in user mode
│   nginx           |  postgres           │    Cannot touch hardware directly
├─────────────────────────────────────────┤
│          System Libraries               │
│   libc, glibc, libssl, etc.             │  ← Help apps make system calls
├─────────────────────────────────────────┤
│         SYSTEM CALL INTERFACE           │  ← The boundary between user/kernel
├─────────────────────────────────────────┤
│              KERNEL                     │
│   Memory | Processes | Network | FS     │  ← Runs in kernel mode
│   Device drivers | Scheduler           │    Has full hardware access
├─────────────────────────────────────────┤
│             Hardware                    │
│   CPU  |  RAM  |  Disk  |  Network      │
└─────────────────────────────────────────┘
```

This architecture is important because it means: **as long as the kernel is the same, multiple different userspace environments can run simultaneously on the same machine.** You can run a "Debian-like" userspace and an "Alpine-like" userspace simultaneously, both making system calls to the same Linux kernel.

This is exactly what containers do.

### 3.2 Namespaces: The Isolation Mechanism Explained Deeply

Linux namespaces are the kernel feature that makes container isolation possible. They were introduced into the Linux kernel starting around 2002 (with `mnt` namespaces) and gradually expanded through the 2000s and 2010s.

**What a namespace does**: A namespace is a kernel-level wrapper that makes a process (and its children) believe they have their own, private version of some system resource — when in reality, the kernel is giving them an isolated view of a shared resource.

The "lie" metaphor is useful here. When a process is in a PID namespace, the kernel lies to it: "You are PID 1. You are the first process. You are the top of the process tree." The process has no way of knowing that from the host's perspective, it might actually be PID 17,382. The kernel maintains the mapping internally.

**The six main namespace types**:

**1. PID Namespace (Process IDs)**

Without PID namespace: every process on the system sees all other processes. `ps aux` shows everything.

With PID namespace: the process inside the container sees only the processes in its own namespace. The process that started the container becomes PID 1 from the container's perspective.

```bash
# On the host machine:
ps aux | grep "node server.js"
# → root   17382  node server.js   ← It's PID 17382 on the host

# Inside the container:
ps aux
# → root   1  node server.js       ← It thinks it's PID 1 inside the container
```

Why does this matter? PID 1 has special significance in Unix: it's the "init" process, the ancestor of all other processes. If PID 1 exits, the whole environment shuts down. By making the container's main process PID 1 in its own namespace, Docker ensures the container has correct lifecycle behavior.

**2. Network Namespace (net)**

Without network namespace: all processes share the same network interfaces, the same routing table, the same port space. Two processes can't both listen on port 80.

With network namespace: each container gets its own virtual network interface, its own IP address (in Docker's internal subnet), its own routing table, and its own completely separate port space. Two containers can both listen on port 3000 inside their own namespaces with no conflict.

```bash
# Host machine has one IP (e.g., 192.168.1.100) on en0

# Container A has its own virtual network interface:
docker exec containerA ip addr show eth0
# → eth0: inet 172.17.0.2

# Container B has its own virtual network interface:
docker exec containerB ip addr show eth0
# → eth0: inet 172.17.0.3

# Both can listen on port 3000 internally — no conflict
# Because port 3000 in container A's namespace ≠ port 3000 in container B's namespace
```

**3. Mount Namespace (mnt)**

Without mount namespace: all processes see the same filesystem tree. Any process can read any file (subject to permissions).

With mount namespace: each container sees its own filesystem root. When a container accesses `/`, it's accessing the root of the image's filesystem — not the host's `/`. The host's actual filesystem is completely invisible to the container.

```bash
# Host has /etc/hosts with host machine's entries
# Container has /etc/hosts with container-specific entries

# Inside the container:
cat /etc/hosts
# → 127.0.0.1 localhost
# → 172.17.0.2 mycontainer     ← Container-specific, not the host's file

# Host has /usr/bin/python3
# Container might NOT have Python if it's an Alpine-based image
# They have completely separate filesystem trees
```

**4. UTS Namespace (hostname)**

Without UTS namespace: all processes share the same hostname.

With UTS namespace: each container has its own hostname. By default, Docker sets the hostname to the container's short ID.

```bash
# Host machine hostname:
hostname
# → MacBook-Pro.local

# Inside a container:
hostname
# → 3f8e5a2b1c9d    ← Container's own hostname (its short ID by default)
# Or whatever you set with docker run --hostname myapp
```

**5. IPC Namespace (Interprocess Communication)**

Without IPC namespace: processes can communicate via shared memory segments, semaphores, and message queues that are visible system-wide.

With IPC namespace: the container's IPC resources are isolated. Shared memory created inside a container isn't visible to other containers or the host.

This prevents containers from accidentally or maliciously accessing another container's shared memory.

**6. User Namespace (UIDs and GIDs)**

Without user namespace: UID 0 (root) inside a container is the same as UID 0 on the host. Being root inside a container means being root on the host.

With user namespace: the container can have its own UID/GID mappings. UID 0 inside the container can map to an unprivileged UID (e.g., UID 100000) on the host. This provides a critical security boundary.

Note: Docker doesn't enable user namespaces by default because they add complexity. Enabling them requires additional configuration but significantly improves security.

**Visualizing all namespaces together**:

```
HOST MACHINE
├── Kernel (shared by everything)
├── Host processes (host's PID namespace, network namespace, etc.)
│
├── CONTAINER A (its own set of namespaces)
│   ├── PID namespace: sees only its own processes, PID 1 = node
│   ├── Net namespace: eth0 = 172.17.0.2, port space is private
│   ├── Mnt namespace: / = alpine filesystem root
│   ├── UTS namespace: hostname = "containerA"
│   └── IPC namespace: isolated IPC resources
│
└── CONTAINER B (its own set of namespaces)
    ├── PID namespace: sees only its own processes, PID 1 = postgres
    ├── Net namespace: eth0 = 172.17.0.3, port space is private
    ├── Mnt namespace: / = debian filesystem root
    ├── UTS namespace: hostname = "containerB"
    └── IPC namespace: isolated IPC resources
```

All of these containers share the same physical hardware and the same Linux kernel. The namespaces create the illusion of separation.

### 3.3 Cgroups: The Resource Limiter Explained Deeply

Namespaces control **what a process can see**. Cgroups (control groups) control **what a process can use** in terms of system resources.

Cgroups are another Linux kernel feature, introduced around 2008 by Google engineers who needed to run thousands of processes on shared infrastructure. The problem: without resource limits, one misbehaving process can hog all the CPU, all the memory, or all the disk I/O, degrading performance for everything else on the machine.

**What cgroups control**:

| Cgroup Subsystem | What It Limits |
|---|---|
| `memory` | Maximum RAM usage; what happens when it's exceeded (OOM kill) |
| `cpu` | CPU shares and quotas (how much CPU time a container gets) |
| `cpuset` | Which specific CPU cores a container can use |
| `blkio` | Block I/O rate (reads/writes to disk) |
| `net_cls` | Network packet classification for QoS |
| `pids` | Maximum number of processes/threads |
| `devices` | Which hardware devices the container can access |

**How Docker uses cgroups**:

When you start a container with resource limits, Docker writes configuration to the cgroup filesystem:

```bash
docker run --memory="512m" --cpus="1.5" myapp

# Docker writes to the kernel's cgroup interface:
# /sys/fs/cgroup/memory/docker/<container-id>/memory.limit_in_bytes = 536870912
# /sys/fs/cgroup/cpu/docker/<container-id>/cpu.quota = 150000
# /sys/fs/cgroup/cpu/docker/<container-id>/cpu.period = 100000
# (150000/100000 = 1.5 CPUs)
```

The kernel then enforces these limits automatically. If the container tries to allocate more than 512MB of RAM, the kernel's OOM (Out of Memory) killer terminates a process inside the container.

**Real-world analogy for cgroups**: Imagine a hotel with one massive shared swimming pool. The hotel manager (cgroup subsystem) assigns each guest (container) a specific lane. Guest A gets lanes 1-2. Guest B gets lane 3. Guest C gets lanes 4-6. No matter how fast Guest A swims, they cannot use Guest B's lane. The hotel manager's rules (cgroup limits) are enforced by hotel security (the kernel) automatically — guests can't negotiate around them.

**Why understanding cgroups matters in practice**:

```bash
# Without cgroup limits, a memory leak in one container
# can crash the entire host machine

# With limits, the OOM killer only affects that container
docker run --memory="512m" --memory-swap="512m" --oom-kill-disable=false myapp
# If the app leaks memory and hits 512MB:
# → The container is killed
# → Host machine is unaffected
# → Other containers keep running normally

# Checking if a container was OOM-killed
docker inspect myapp | grep -i oom
# → "OOMKilled": true
```

**CPU limits explained**:

```bash
# --cpus="1.5" means the container can use at most 1.5 CPU cores' worth of time
# If the host has 8 cores, the container won't monopolize them
# The actual mechanism: cpu.quota / cpu.period
# default period = 100ms, quota = 150ms → 1.5 CPUs

# On a 4-core machine without limits:
# One container could use 400% CPU (all 4 cores), starving everything else

# With --cpus="1.5":
# Container is limited to 150% CPU usage
# Remaining 250% available for other containers and host processes
```

### 3.4 Virtual Machines vs Containers: A Complete Comparison

Now that you understand namespaces and cgroups, let's see how containers fundamentally differ from virtual machines architecturally.

**Virtual Machine Architecture**:

A virtual machine uses a hypervisor to emulate an entire computer. The hypervisor sits between the hardware and the guest OS, intercepting hardware access and translating it.

```
┌────────────────────────────────────────────────────────────┐
│                     HOST MACHINE                           │
│  ┌──────────────────────────────────────────────────────┐  │
│  │                   HOST OS + KERNEL                   │  │
│  ├──────────────────────────────────────────────────────┤  │
│  │               HYPERVISOR (KVM / VMware / Hyper-V)   │  │
│  ├─────────────────────────┬────────────────────────────┤  │
│  │         VM 1            │          VM 2              │  │
│  │  ┌──────────────────┐   │  ┌─────────────────────┐   │  │
│  │  │   Guest OS (Linux│   │  │  Guest OS (Windows) │   │  │
│  │  │   Full Kernel    │   │  │  Full Kernel        │   │  │
│  │  ├──────────────────┤   │  ├─────────────────────┤   │  │
│  │  │   System Libs    │   │  │  System Libs        │   │  │
│  │  ├──────────────────┤   │  ├─────────────────────┤   │  │
│  │  │   Application    │   │  │  Application        │   │  │
│  │  └──────────────────┘   │  └─────────────────────┘   │  │
│  └─────────────────────────┴────────────────────────────┘  │
└────────────────────────────────────────────────────────────┘
```

Each VM runs a **complete, full operating system** — its own kernel, its own system libraries, its own everything. This is why VMs:

- Take minutes to start (they have to boot an entire OS)
- Consume gigabytes of RAM (each guest OS needs its own memory)
- Are large files (a VM image includes a full OS)
- Have higher overhead (the hypervisor adds a translation layer)

**Container Architecture**:

```
┌────────────────────────────────────────────────────────────┐
│                     HOST MACHINE                           │
│  ┌──────────────────────────────────────────────────────┐  │
│  │               HOST KERNEL (shared)                   │  │
│  │    Namespaces + Cgroups enforce isolation             │  │
│  └──────────────────────────────────────────────────────┘  │
│         ↑                  ↑                  ↑            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │ Container A  │  │ Container B  │  │ Container C  │     │
│  │ Alpine Libs  │  │ Debian Libs  │  │ Ubuntu Libs  │     │
│  │ Node.js app  │  │ Python app   │  │ Java app     │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│   No separate kernel — all share the host kernel above     │
└────────────────────────────────────────────────────────────┘
```

Each container shares the host kernel. What makes them "look" different is the **userspace** — the filesystem, the libraries, the tools. Container A can have Alpine Linux's filesystem and tools. Container B can have Debian's. But both use the host's Linux kernel for all system calls.

This is why containers:

- Start in milliseconds (they're just starting a process — no boot)
- Use much less memory (no guest OS overhead per container)
- Are lightweight (images contain only the userspace, not a kernel)
- Have near-zero overhead (process isolation via kernel features, not emulation)

**Detailed comparison table**:

| Feature | Virtual Machine | Docker Container |
|---|---|---|
| **Startup time** | 30 seconds to several minutes | Under 1 second (often milliseconds) |
| **Image size** | Gigabytes (full OS) | Megabytes (just app + userspace libraries) |
| **Memory overhead** | Hundreds of MB per VM (guest OS) | Near zero (shared kernel) |
| **CPU overhead** | Hypervisor translation layer | Near zero (native execution) |
| **OS kernel** | Each VM has its own kernel | Shared host kernel |
| **OS type flexibility** | Can run Windows on Linux host | Must match host kernel type (Linux on Linux) |
| **Isolation strength** | Very strong (separate kernel) | Strong but shared kernel attack surface |
| **Portability** | Heavy, slow to move | Lightweight, fast to distribute |
| **Density** | 10-100 VMs on a host | 100-10,000 containers on a host |
| **Storage efficiency** | Full OS per machine | Shared base layers across containers |
| **Best use case** | Running different OS types; strong isolation requirements | Application deployment, microservices, dev environments |

**When VMs are the right choice over containers**:

1. You need to run Windows applications on a Linux host (or vice versa)
2. You need kernel-level isolation — multi-tenant hosting where tenants are untrusted
3. Compliance requirements mandate full OS-level separation
4. You need to test kernel behavior or kernel modules

For most application development and deployment scenarios, containers are the better choice.

### 3.5 What Happens on Mac and Windows: The Hidden VM

Here's a fact that surprises many developers: Docker containers require Linux kernel features (namespaces and cgroups). These features only exist in the Linux kernel. So how do containers run on a Mac or Windows machine?

**The answer**: Docker Desktop (the tool you install on Mac and Windows) quietly runs a **small, minimal Linux virtual machine** in the background. Your containers actually run inside that VM, using that VM's Linux kernel.

```
┌─────────────────────────────────────────────────────────────┐
│                macOS (Darwin kernel)                        │
│  ┌───────────────────────────────────────────────────────┐  │
│  │     Linux VM (lightweight, managed by Docker Desktop) │  │
│  │     Uses Apple Virtualization Framework / HyperKit    │  │
│  │  ┌─────────────────────────────────────────────────┐  │  │
│  │  │              Linux Kernel                       │  │  │
│  │  ├──────────────┬────────────────────────────────  │  │  │
│  │  │ Container A  │  Container B  │  Container C     │  │  │
│  │  └──────────────┴──────────────┴──────────────     │  │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

**Key implications of this architecture**:

1. **Docker on Mac/Windows is not truly "native"**: There's always a VM layer. This adds some overhead, and this is why Docker Desktop can be resource-heavy.

2. **File I/O with bind mounts is slower on Mac/Windows**: When you mount your local files into a container (bind mount), those files have to cross the VM boundary — from macOS filesystem to the Linux VM filesystem. This is why hot reloading can feel sluggish on Mac during development.

3. **Container IPs are in the VM's network**: On Linux, you can directly access a container's IP (e.g., `172.17.0.2`) from the host. On Mac/Windows, you can't — because that IP is inside the VM's network. You must use published ports (`-p 3000:3000`) to access containers.

4. **On Linux, everything is faster**: No VM needed. Containers talk directly to the host kernel. Bind mounts are native filesystem operations. Container IPs are directly accessible.

```bash
# On Linux: can access container IP directly
docker run -d nginx
docker inspect <id> | grep IPAddress
# → 172.17.0.2
curl 172.17.0.2   # Works!

# On Mac: container IP is inside the VM, not accessible from Mac
curl 172.17.0.2   # Does NOT work — that IP is inside the VM
docker run -p 8080:80 nginx
curl localhost:8080   # This works — port forwarded through VM
```

---

## 4. Images and Containers

### Blueprint vs Running Instance

When you start using Docker, you constantly hear two terms: **image** and **container**. They are related but fundamentally different. Confusing them is extremely common and causes real problems in how developers work with Docker.

The clearest single-sentence explanation: **An image is a blueprint. A container is a running instance created from that blueprint.**

### 4.1 What Is a Docker Image? (Deep Explanation)

A Docker image is a **read-only, immutable, layered filesystem snapshot** that contains everything needed to run an application.

Let's unpack every part of that definition:

**Read-only**: You cannot modify an image while it's in use. If you need a different image, you build a new one.

**Immutable**: Once built, an image never changes. The image you pull today is bit-for-bit identical to the image you pull in six months (assuming the same tag and digest).

**Layered**: An image is not a single monolithic file. It is a stack of filesystem layers, each representing a set of changes. We'll cover this extensively in Section 5.

**Filesystem snapshot**: An image captures a complete view of a filesystem tree — all the files, directories, permissions, and metadata that an application needs.

**Contains everything needed to run**: This is the key value proposition. The image bundles:
- The application code
- The runtime (Python interpreter, Node.js, JVM, etc.)
- System libraries (glibc, libssl, etc.)
- Package dependencies (npm modules, pip packages, Maven JARs)
- System tools (curl, bash, etc. — whatever the app needs)
- Default configuration files
- Environment variable defaults
- The startup command

**What an image does NOT contain**:
- The Linux kernel (containers share the host's kernel)
- Your hardware-specific drivers
- Persistent data (that's what volumes are for)
- Runtime secrets (those should be injected at runtime)

**Real-world analogies**:

1. **Class vs Object**: If you've programmed in any object-oriented language, you understand this immediately. An image is the class definition. A container is an object — an instance of the class. You can instantiate the same class multiple times, and each instance is independent. The class definition itself doesn't "run" — it just describes what an instance looks like and how it behaves.

2. **VM template/snapshot**: A Docker image is like a VM golden master image — a perfect, stable snapshot that you clone when you need to spin up a new instance.

3. **Compiled binary**: A Docker image is like a compiled binary file (a `.exe` or `.app`). It just sits on disk until you execute it. Executing it creates a running process (a container). The binary itself doesn't change when you run it.

4. **A recipe in a cookbook**: The image is a complete recipe with all ingredients listed. A container is a meal prepared from that recipe. You can cook the same recipe a hundred times, each producing an independent meal.

**Looking at what's in an image**:

```bash
# See all images on your machine
docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
node          20-alpine 1234abcd5678   2 weeks ago    180MB
postgres      15        abcd12345678   1 week ago     430MB
myapp         latest    efgh12345678   2 hours ago    245MB

# Inspect the detailed metadata of an image
docker inspect node:20-alpine
# Shows: layers, environment variables, default command,
# exposed ports, architecture, OS, etc.

# See what layers make up an image
docker history node:20-alpine
IMAGE          CREATED BY                          SIZE
1234abcd       CMD ["node"]                         0B
...            ENV NODE_VERSION=20.11.0             0B
...            RUN /bin/sh -c apk add --no-cache... 68.3MB
...            FROM alpine:3.19                     7.38MB
```

**Image naming and tagging**:

```
registry.io / namespace / name : tag
    ↑              ↑         ↑       ↑
  Optional      Optional  Required  Optional
  (default:     (Docker   (the      (default: "latest")
  docker.io)    Hub user) image
                          name)
```

Examples:
```
node:20-alpine              → docker.io/library/node:20-alpine
postgres:15                 → docker.io/library/postgres:15
myusername/myapp:v1.2.3    → docker.io/myusername/myapp:v1.2.3
ghcr.io/myorg/myapp:latest → GitHub Container Registry
registry.mycompany.com/myapp:prod-20240101  → Private registry
```

### 4.2 What Is a Docker Container? (Deep Explanation)

A container is what happens when Docker takes an image, uses Linux kernel features to create an isolated environment, and starts the image's defined process inside that environment.

More precisely, creating a container involves:

1. **Setting up a new set of namespaces**: PID, net, mnt, uts, ipc
2. **Mounting the image's filesystem layers** as the container's root filesystem (in read-only mode)
3. **Adding a new writable layer** on top (called the "container layer" or "copy-on-write layer")
4. **Applying cgroup limits** if specified
5. **Configuring the network** (assigning an IP, connecting to networks)
6. **Starting the process** defined by the image's CMD/ENTRYPOINT instruction

From this moment, the running process is a container.

**What makes a container an instance of an image**:

You can create as many containers from the same image as you want. Each one:
- Starts with the exact same filesystem (from the image layers)
- Has its own independent writable layer
- Has its own process space
- Has its own network identity (IP address)
- Runs completely independently of other containers from the same image

```bash
# Create 3 containers from the same image
docker run -d --name app1 -p 3001:3000 myapp
docker run -d --name app2 -p 3002:3000 myapp
docker run -d --name app3 -p 3003:3000 myapp

# Each has its own IP, its own writable layer, its own processes
docker inspect app1 | grep IPAddress  # → 172.17.0.2
docker inspect app2 | grep IPAddress  # → 172.17.0.3
docker inspect app3 | grep IPAddress  # → 172.17.0.4

# Each is completely independent
# Changes in app1 don't affect app2 or app3
```

**Container states**:

A container can be in several states:

```
[created] → [running] → [paused] → [running]
                ↓
           [stopping]
                ↓
           [stopped/exited]
                ↓
           [removed] ← PERMANENT, cannot undo
```

| State | Description | Writable Layer |
|---|---|---|
| created | Container created but not started | Exists, empty |
| running | Container's process is active | Exists, may have data |
| paused | Process frozen (SIGSTOP) | Exists with data |
| exited/stopped | Process has exited | Exists with data |
| removed (rm'd) | Container deleted | **Permanently gone** |

The critical insight: **stopping a container does not destroy its writable layer**. Data in a stopped container is still there. Only `docker rm` destroys the writable layer permanently.

### 4.3 The Writable Layer: The Complete Story of Why Data Disappears

This section explains one of the most frequently confused aspects of Docker. Let's go deep.

**What the writable layer is**:

When a container starts, Docker uses a technology called **Copy-on-Write (CoW)** to manage the container's filesystem.

The image's layers are mounted as read-only. On top of them, Docker creates one additional layer that is writable — the container layer. This writable layer starts empty. As the container's process reads files, it reads from the image layers below. As it writes or modifies files, those writes go into the writable layer.

```
Container's view of filesystem (what the running app sees):
┌─────────────────────────────────────────┐
│          WRITABLE LAYER                 │
│  Any new files created by the app       │
│  Any modified copies of image files     │  ← Only this layer is writable
│  Temporary files, logs, cache           │
├─────────────────────────────────────────┤
│    [READ-ONLY] App Code Layer           │  ← Image layer, never changes
├─────────────────────────────────────────┤
│    [READ-ONLY] npm modules Layer        │  ← Image layer, never changes
├─────────────────────────────────────────┤
│    [READ-ONLY] Node.js Runtime Layer    │  ← Image layer, never changes
├─────────────────────────────────────────┤
│    [READ-ONLY] Alpine Linux Layer       │  ← Image layer, never changes
└─────────────────────────────────────────┘
```

**The Copy-on-Write mechanism**:

When a container modifies a file that exists in the image layers, here's what happens:

1. The kernel intercepts the write attempt
2. Docker copies the original file from the image layer up to the writable layer
3. The modification is made to the copy in the writable layer
4. Future reads of that file come from the writable layer (the modified version)
5. The original in the image layer remains untouched

This means the image is always pristine. No matter what happens inside a running container, the underlying image is never modified. This is why you can run a hundred containers from the same image simultaneously — they all share the same read-only image layers, each with their own tiny writable layer on top.

**Where the writable layer is stored**:

Docker stores the writable layer on the host machine, in Docker's data directory:

```
/var/lib/docker/overlay2/<container-id>/
                                       diff/    ← The writable layer data
                                       work/    ← Overlay work directory
                                       merged/  ← The merged view (what the container sees)
```

When you do `docker rm`, Docker deletes this entire directory.

**A concrete demonstration of the lifecycle**:

```bash
# Step 1: Run a container and create some data
docker run --name demo ubuntu bash -c "
  echo 'database records' > /data/records.txt &&
  echo 'user uploads' > /uploads/photo.jpg &&
  apt-get install -y curl
"

# What happened: /data/records.txt, /uploads/photo.jpg, and curl
# are all in the container's WRITABLE LAYER.
# The ubuntu image itself is unchanged.

# Step 2: The container has exited. Is the data still there?
docker ps -a  # Shows demo container in 'Exited' state

# Let's start it again (not remove and recreate — just restart)
docker start demo
docker exec demo cat /data/records.txt
# → database records   ← STILL THERE! Stopping ≠ removing.

# Step 3: Now remove the container
docker rm demo

# The writable layer directory is now deleted from the host.
# All that data — records.txt, photo.jpg, the installed curl — is gone.

# Step 4: Create a new container from the same image
docker run --name demo ubuntu bash
# Fresh filesystem. No records.txt. No curl. Nothing from before.
# The ubuntu image was never modified.
```

**Why this is designed this way (the purpose of ephemeral writable layers)**:

1. **Image integrity**: The image is always pristine. Any container created from it starts from a known, clean state.
2. **Efficiency**: All containers sharing an image share the same read-only layers on disk. You don't have 50 copies of node_modules on disk for 50 containers.
3. **Explicit persistence**: If data matters, you have to explicitly say so with a volume. This makes data persistence intentional rather than accidental.
4. **Easy horizontal scaling**: Creating a new container from the same image gives you an exact copy of the running environment. No state to sync or worry about.

**The practical implication**: Never store anything important inside a container's writable layer. If data needs to survive container replacements — database records, user uploads, log files, configuration that changes at runtime — use volumes.

### 4.4 Image Immutability: Why This Is a Feature, Not a Bug

Images never change after they're built. This immutability is what enables Docker's most powerful capabilities.

**What immutability means in practice**:

```bash
# You build image v1.0.0
docker build -t myapp:1.0.0 .

# Three months later, on a different machine:
docker pull myapp:1.0.0

# These two are bit-for-bit identical.
# Every file, every library version, every configuration.
# They will behave identically.
```

**What immutability enables**:

1. **Reproducible environments**: The same image always produces the same environment. "Works on my machine" is eliminated because the image IS the machine.

2. **Reliable rollbacks**: Rolling back means deploying the previous image tag. The old environment is perfectly preserved.
   ```bash
   # Deploy v1.2.0
   docker run myapp:1.2.0
   
   # Problem detected. Roll back in seconds:
   docker run myapp:1.1.0   # Exact previous environment, instantly
   ```

3. **Auditability**: You can examine exactly what's in any deployed image — every file, every package version, every configuration.
   ```bash
   docker history myapp:1.2.0   # Shows every layer that was added
   docker inspect myapp:1.2.0   # Shows all metadata
   ```

4. **Safe sharing**: Pushing an image to a registry and having teammates pull it gives everyone the exact same environment.

5. **Concurrent versions**: You can run v1.0.0 and v2.0.0 simultaneously (on different ports) during a gradual rollout.

**The workflow principle that comes from immutability**:

> If you need to change something permanently, change the Dockerfile, rebuild the image, and deploy the new image. Never rely on changes made inside a running container.

```bash
# WRONG workflow: modifying inside container
docker exec myapp npm install express-rate-limit
# This installs the package in the WRITABLE LAYER
# When this container is replaced, the package is gone

# RIGHT workflow: modify the source, rebuild
# In Dockerfile:
# RUN npm install express-rate-limit
docker build -t myapp:1.2.1 .
docker run myapp:1.2.1
# Now the package is part of the IMAGE, persistent across any container
```

### 4.5 The Image vs Container Lifecycle

Understanding the complete lifecycle helps you know what to do when things go wrong.

```
SOURCE CODE + Dockerfile
        ↓
    docker build
        ↓
      IMAGE (stored locally)
        ↓
    docker push (optional)
        ↓
  IMAGE REGISTRY (Docker Hub, ECR, GCR, etc.)
        ↓
    docker pull (on any machine)
        ↓
      IMAGE (stored locally on target machine)
        ↓
    docker run
        ↓
    CONTAINER (running process)
        ↓
    docker stop
        ↓
    CONTAINER (stopped, writable layer preserved)
        ↓
    docker start  ←─────────────────────────────────┐
        ↓                                            │
    CONTAINER (running again, same writable layer)  │
        ↓                                            │
    docker stop → docker start  (cycle repeats)  ───┘
        ↓
    docker rm
        ↓
    CONTAINER DESTROYED (writable layer gone)
```

---

## 5. Docker Layers

### The Caching System That Governs Build Speed

Understanding layers is what separates developers who fight with slow builds from developers who have fast, predictable builds. It's also essential for understanding image size and distribution efficiency.

### 5.1 What Layers Are and How They Are Created

A Docker image is not a single blob. It is a **stack of filesystem layers**, where each layer captures a set of filesystem changes (files added, modified, or deleted).

Think of layers like git commits. Each git commit records the diff from the previous state. Docker layers work the same way — each layer is a diff from the previous layer's state. When the container runtime wants to present the full filesystem, it merges all the layers together (using a union filesystem like OverlayFS) to give the container a single, coherent view.

**How layers are created when building an image**:

Every Dockerfile instruction that **changes the filesystem** creates a new layer. Instructions that only set metadata (like CMD, ENV, EXPOSE) do not create filesystem layers — they add metadata that gets stored in the image manifest.

```dockerfile
FROM node:20-alpine         # ← Imports all layers from the node:20-alpine image
                            #   (already a stack of 5-8 layers from Alpine + Node install)

WORKDIR /app                # ← Creates /app directory → new layer
                            #   (small layer, just the directory)

COPY package.json           # ← Adds package.json → new layer
package-lock.json ./        #   (tiny layer, just one or two files)

RUN npm install             # ← Runs npm install → new layer
                            #   ALL of node_modules written to this layer
                            #   (often 50-200MB depending on dependencies)

COPY . .                    # ← Adds all your source code → new layer
                            #   (your code, maybe a few hundred KB)

CMD ["node", "server.js"]   # ← No new layer; metadata only
```

**Visualizing the layer stack**:

```
Layer 6: COPY . .                    (your source code, ~500KB)
Layer 5: RUN npm install             (node_modules, ~80MB)
Layer 4: COPY package.json           (~2KB)
Layer 3: WORKDIR /app                (~0KB, just directory entry)
Layer 2: node:20-alpine (Node.js)    (~100MB)
Layer 1: node:20-alpine (Alpine)     (~7MB)
```

Total image = sum of all layer sizes. But on disk, layers are shared between images that use the same base.

**How layers are stored**:

Each layer is stored as a tar archive, identified by the SHA256 hash of its contents. This is what makes caching possible — if the content hasn't changed, the hash hasn't changed.

```bash
# See the layer IDs of an image
docker inspect myapp | grep -A 20 "Layers"
# "sha256:abc123..."  ← Each line is one layer, identified by content hash
# "sha256:def456..."
# "sha256:ghi789..."
```

### 5.2 How Layer Caching Works Internally

Layer caching is the mechanism that makes Docker builds fast after the first build. Understanding it precisely is essential for writing efficient Dockerfiles.

**The caching check Docker performs**:

When rebuilding an image, Docker processes each instruction in the Dockerfile from top to bottom. For each instruction, it checks the **build cache** — a record of previously built layers and what produced them.

The cache check logic:

1. For `FROM`: Check if the base image's layers are already on disk. If yes, use them.
2. For `RUN`: Check if the exact same command text has been run on top of the exact same previous layer (same hash). If yes, reuse the result.
3. For `COPY`/`ADD`: Check if the files being copied have the same content (same checksums) as what was copied before. If yes, reuse the cached layer.
4. **Once any instruction's cache is invalidated, ALL subsequent instructions are also invalidated** — even if their content hasn't changed.

This last rule is the most important. Cache invalidation cascades forward.

```
Dockerfile instructions:      What happens when you change only source code:

FROM node:20-alpine           ✓ Cache HIT (base image unchanged)
WORKDIR /app                  ✓ Cache HIT (same instruction, same parent layer)
COPY package.json .           ✓ Cache HIT (package.json unchanged)
RUN npm install               ✓ Cache HIT (same command, same package.json = same result)
COPY . .                      ✗ Cache MISS (source code changed!)
                                ← Cache invalidation starts HERE
CMD ["node", "server.js"]     ✗ No cache possible (instruction after a miss)
```

When there's a cache miss on `COPY . .`, Docker:
1. Re-runs the COPY instruction with the new files
2. Creates a new layer
3. All subsequent instructions run fresh

But crucially, `npm install` was NOT re-run. That's the entire point of separating the dependency copy from the code copy.

### 5.3 Cache Invalidation: The Rules That Govern Everything

Understanding exactly when cache is invalidated versus preserved is the key to writing fast Dockerfiles.

**Rule 1: For RUN instructions, cache is invalidated when the command text changes**

```dockerfile
RUN npm install              # Cached if this exact string hasn't changed
RUN npm install --verbose    # Different string = cache miss, re-runs
```

Note: Docker doesn't run the command and compare outputs. It only checks the command string. This means:

```dockerfile
RUN date >> /tmp/build-time.txt    # This will always be a cache hit!
# Even though it produces different content each time (different date),
# the command string is the same, so Docker reuses the cached layer
# from the previous build. The date in /tmp/build-time.txt will be stale.
```

This is a source of subtle bugs: if your `RUN` command depends on external state (network resources, timestamps, random values) but its string is unchanged, Docker won't re-run it.

**Rule 2: For COPY/ADD instructions, cache is invalidated when file contents change**

Docker computes checksums of the source files and compares them to what was cached. If any file's content changed, cache is invalidated.

```dockerfile
COPY package.json .        # Cache miss if package.json changed (even one character)
                           # Cache HIT if package.json is identical to last build
```

**Rule 3: Cache invalidation cascades forward through all subsequent layers**

This is the most important rule. Once any layer misses the cache, every subsequent layer will also miss.

```dockerfile
FROM node:20-alpine          # Hit
WORKDIR /app                 # Hit
COPY package.json .          # HIT (unchanged)
RUN npm install              # HIT (package.json unchanged, same command)
COPY . .                     # MISS (code changed)
RUN npm run build            # MISS (forced, even if this would produce same output)
CMD ["node", "dist/server.js"] # MISS (forced)
```

**Rule 4: Base image changes propagate forward**

If you update a base image tag (e.g., from `node:20.10.0` to `node:20.11.0`), every layer in your image is invalidated, even if your Dockerfile didn't change.

```dockerfile
FROM node:20-alpine          # If alpine updated, this is a MISS
                             # → Everything below is also a MISS
WORKDIR /app                 # MISS (forced)
COPY package.json .          # MISS (forced)
RUN npm install              # MISS (forced) ← npm install runs again!
COPY . .                     # MISS (forced)
```

This is one reason why keeping dependencies stable (pinning exact versions) matters for reproducible builds.

### 5.4 Instruction Order: The Single Biggest Build Speed Lever

The single most impactful change you can make to a Dockerfile is **ordering instructions from least-frequently-changing to most-frequently-changing**.

**The principle**: Put things that rarely change near the top. Put things that change often near the bottom. This maximizes cache hits for the expensive operations.

**Frequency of change, roughly ordered**:

```
Changes very rarely:
  - Base image (FROM)
  - System dependency installs (apt-get, apk add)
  - Build tools installation

Changes occasionally:
  - Dependency manifests (package.json, requirements.txt, go.mod)
  - Dependency installation (npm install, pip install, go mod download)

Changes frequently:
  - Application source code
  - Configuration files
  - Tests
```

**The pattern applied to different languages**:

```dockerfile
# ─────────────────────────────────────────────────────────
# Node.js — optimized
# ─────────────────────────────────────────────────────────
FROM node:20-alpine
WORKDIR /app

# Package files first (changes less often than source code)
COPY package.json package-lock.json ./
RUN npm ci

# Source code last (changes most often)
COPY . .
CMD ["node", "server.js"]

# ─────────────────────────────────────────────────────────
# Python — optimized
# ─────────────────────────────────────────────────────────
FROM python:3.11-slim
WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .
CMD ["python", "app.py"]

# ─────────────────────────────────────────────────────────
# Go — optimized
# ─────────────────────────────────────────────────────────
FROM golang:1.21
WORKDIR /app

# Go module files first
COPY go.mod go.sum ./
RUN go mod download

# Source code after
COPY . .
RUN go build -o server .
CMD ["/app/server"]

# ─────────────────────────────────────────────────────────
# Java/Maven — optimized
# ─────────────────────────────────────────────────────────
FROM maven:3.9-openjdk-17
WORKDIR /app

# pom.xml first (dependency definitions)
COPY pom.xml .
RUN mvn dependency:resolve -B

# Source code after
COPY src ./src
RUN mvn package -DskipTests
CMD ["java", "-jar", "target/app.jar"]
```

**Quantifying the difference**:

Suppose you have a Node.js app where `npm install` takes 60 seconds. You make code changes 20 times per day during active development.

| Approach | Cache Behavior | Daily Build Time |
|---|---|---|
| COPY . . before npm install | npm install re-runs every time | 20 × 60s = 20 minutes |
| COPY package.json first | npm install only runs when deps change | ~0 × 60s + 20 × 3s = 1 minute |
| **Savings** | | **19 minutes per day** |

Over a year of active development: ~70 hours of saved waiting time.

### 5.5 Layer Sharing Across Images

Layers aren't just cached per-image. They are **shared across all images on your machine**.

When two images share the same base image (e.g., both are `FROM node:20-alpine`), they literally share the same layer files on disk. Docker identifies layers by their SHA256 hash. If two images have a layer with the same hash, Docker stores it once and references it from both images.

```bash
# You have two services, both based on node:20-alpine
# Service A image layers:
#   sha256:aaa...  (alpine base)
#   sha256:bbb...  (node install)
#   sha256:ccc...  (service A's node_modules)
#   sha256:ddd...  (service A's source code)

# Service B image layers:
#   sha256:aaa...  (alpine base — SAME HASH, stored once on disk!)
#   sha256:bbb...  (node install — SAME HASH, stored once on disk!)
#   sha256:eee...  (service B's node_modules — different)
#   sha256:fff...  (service B's source code — different)

docker system df   # Shows total disk usage
# Images:     4   (~600MB apparent, but only ~200MB unique due to sharing)
```

**Impact on image distribution (push/pull)**:

When you push Service A to a registry, and then push Service B, the registry only receives the layers unique to Service B. The shared base layers are already on the registry (from Service A's push) and are not transferred again.

When a developer pulls Service B on a machine that already has Service A, only the unique layers are downloaded.

**This means**:
- Standardize on the same base images across your team's services
- Keep base images consistent reduces total storage and transfer dramatically
- Update base images (security patches) propagates efficiently

### 5.6 How to Inspect and Debug Layers

```bash
# See all layers of an image with sizes and creation commands
docker history myapp:latest
IMAGE          CREATED        CREATED BY                                   SIZE
a1b2c3d4       2 hours ago    CMD ["node" "server.js"]                      0B
e5f6g7h8       2 hours ago    COPY . .                                      45.2kB
i9j0k1l2       2 hours ago    RUN /bin/sh -c npm ci                         87.3MB
m3n4o5p6       2 hours ago    COPY package.json package-lock.json ./        3.2kB
q7r8s9t0       2 hours ago    WORKDIR /app                                  0B
u1v2w3x4       3 days ago     CMD ["node"]                                  0B
# ... more base layers

# See the same but without truncation
docker history --no-trunc myapp:latest

# Check what build stages produced what
docker build --progress=plain . 2>&1 | grep -E "^#[0-9]+ \[|CACHED|RUN"

# See which layers are shared between images
docker inspect myapp:1.0.0 | grep -A 30 '"RootFS"'
docker inspect myapp:1.1.0 | grep -A 30 '"RootFS"'
# Compare the layer arrays — shared hashes appear in both

# See total disk usage with deduplication
docker system df -v
```

---

## 6. The Dockerfile

### Every Instruction Explained With Full Depth

A Dockerfile is a plain text file. Each line is an instruction to Docker's build system. Docker reads these instructions top-to-bottom and executes them sequentially, producing an image. Each instruction either adds a new layer to the image or adds metadata.

Let's go through every instruction you'll encounter, with complete explanations of what it does, why it exists, and how to use it correctly.

### 6.1 FROM — Your Starting Point, Your Foundation

```dockerfile
FROM <image>[:<tag>][@<digest>]
```

`FROM` is always the first instruction (or one of the first, in multi-stage builds). It specifies the base image — the starting point from which you build your image.

**What FROM actually does**: It imports all the layers from the specified base image. Your image will contain everything in the base image, plus whatever you add on top.

**Why it exists**: Almost no one builds from scratch. The Linux kernel handles the OS kernel stuff. What images provide is a **userspace** — the filesystem layout, the system libraries, the package manager, the runtime (Python, Node, etc.). `FROM` lets you start with that userspace already set up.

**Choosing the right base image**:

```dockerfile
FROM scratch                    # Absolutely empty — for statically compiled binaries
FROM alpine:3.19                # Minimal Linux (5MB). For when you add your own runtime.
FROM ubuntu:22.04               # Full Ubuntu. Large but very compatible.
FROM debian:bookworm-slim       # Smaller Debian, good balance
FROM node:20-alpine             # Alpine + Node.js pre-installed
FROM node:20-slim               # Slim Debian + Node.js
FROM python:3.11-slim           # Slim Debian + Python
FROM openjdk:17-jre-alpine      # Alpine + Java runtime only (not full JDK)
FROM golang:1.21-alpine         # Alpine + Go build tools
```

**The tag matters enormously**:

```dockerfile
FROM node:latest          # ❌ Non-deterministic: "latest" changes. Different build today
                          #    vs tomorrow may produce different images.

FROM node:20              # ✓ Pinned to major version. Smaller breaking changes.

FROM node:20-alpine       # ✓ Pinned major version + specific variant.

FROM node:20.11.0-alpine3.19  # ✓✓ Maximum reproducibility. Pinned to exact patch version.
```

**Using digest for absolute immutability**:

```dockerfile
FROM node:20-alpine@sha256:abc123def456...   # 100% immutable — references exact image content
# Even if the tag "20-alpine" is updated, this always refers to the exact same image
```

**Why alpine vs slim vs full matters**:

| Base | Size | Use case | Tradeoff |
|---|---|---|---|
| `scratch` | 0 | Go/C statically compiled binaries | Must include everything manually |
| `alpine:3.x` | ~5MB | When you control exactly what's added | Uses musl libc (some compatibility issues) |
| `node:20-alpine` | ~180MB | Node.js production apps | Musl libc, some native modules don't work |
| `node:20-slim` | ~250MB | Node.js when alpine has compatibility issues | Good balance |
| `node:20` | ~1.1GB | Development, when size doesn't matter | Large but maximum compatibility |
| `ubuntu:22.04` | ~77MB | When you need Ubuntu-specific tools | No runtime pre-installed |

### 6.2 WORKDIR — Setting the Stage, Not Just Changing Directory

```dockerfile
WORKDIR /path/to/directory
```

`WORKDIR` sets the working directory for all subsequent `RUN`, `COPY`, `ADD`, `CMD`, and `ENTRYPOINT` instructions. If the directory doesn't exist, Docker creates it.

**What WORKDIR does that `RUN cd` does not**:

```dockerfile
# ❌ This does NOT work for subsequent instructions
RUN cd /app

# This is because each RUN instruction starts a new shell session.
# The cd in one RUN has no effect on the next RUN.
# The two instructions below both run from /, not /app:
RUN cd /app
RUN pwd              # → prints "/"  (not /app!)

# ✅ WORKDIR persists for all subsequent instructions
WORKDIR /app
RUN pwd              # → prints "/app"  ✓
COPY . .             # → copies into /app  ✓
CMD ["node", "."]    # → node starts in /app  ✓
```

**Why WORKDIR matters for path resolution**:

```dockerfile
WORKDIR /app

# These are now equivalent:
COPY package.json .         # → /app/package.json
COPY package.json /app/     # → /app/package.json

# And in CMD/ENTRYPOINT, relative paths work:
CMD ["node", "server.js"]   # → node /app/server.js
```

**Best practices**:

```dockerfile
# ✓ Use absolute paths for WORKDIR — never relative
WORKDIR /app            # good
WORKDIR app             # bad — relative to current dir, confusing

# ✓ Set WORKDIR before any COPY or RUN that depends on it
FROM node:20-alpine
WORKDIR /app            # set early
COPY package.json .     # lands in /app/package.json — clear and explicit

# ✓ Consistent convention: /app is the universal standard for web apps
# Other conventions: /opt/app, /srv/app, /home/appuser/app
```

### 6.3 COPY and ADD — The Difference and When to Use Each

Both `COPY` and `ADD` bring files from the build context (your local filesystem) into the image. They look similar but have important differences.

**COPY — what you should use 95% of the time**:

```dockerfile
COPY <src> <dest>

COPY package.json .                      # Single file → current WORKDIR
COPY package.json /app/                  # Single file → explicit path
COPY package.json package-lock.json ./   # Multiple files → current WORKDIR
COPY src/ /app/src/                      # Directory → directory
COPY . .                                 # Everything in context → current WORKDIR
COPY --chown=user:group src dest         # Copy with ownership change
```

**ADD — the same as COPY but with two extra features**:

```dockerfile
ADD <src> <dest>

# Extra feature 1: Fetch from URL
ADD https://example.com/file.tar.gz /tmp/
# (avoid this — use RUN curl or wget instead for better caching)

# Extra feature 2: Auto-extract local tar archives
ADD archive.tar.gz /app/
# (this auto-extracts the tar.gz — convenient but surprising)
```

**Why you should prefer COPY over ADD**:

1. **Predictability**: `COPY` does exactly what it says — copies files. `ADD` has surprising behavior (auto-extraction, URL fetching) that can cause unexpected results.
2. **Cache behavior**: `ADD` with URLs always re-downloads (can't cache), while `RUN curl` gives you more control.
3. **Linting**: Tools like Hadolint (Dockerfile linter) warn you when you use `ADD` where `COPY` would suffice.

**The .dockerignore interaction**:

`COPY . .` copies everything from the build context **that isn't excluded by .dockerignore**. If you have a large `node_modules/` directory and it's not in `.dockerignore`, `COPY . .` will include all of it — potentially hundreds of MB of unnecessary files.

```dockerfile
# With proper .dockerignore:
COPY . .   # Only copies files not excluded by .dockerignore
           # node_modules/, .git/, .env files are excluded
```

**Handling permissions with COPY**:

```dockerfile
# By default, COPY sets files as owned by root:root
COPY . .
# → files owned by root, which matters if you switch USER later

# Use --chown to set correct ownership in one step
RUN adduser -u 1001 -D appuser
COPY --chown=appuser:appuser . .
USER appuser
```

### 6.4 RUN — Executing Commands, Creating Layers, The Cleanup Rule

```dockerfile
RUN <command>          # Shell form (runs in /bin/sh -c)
RUN ["cmd", "arg1"]    # Exec form (runs directly, no shell)
```

`RUN` executes a command during the **build process** and creates a new layer containing any filesystem changes that resulted. It does not run at container start time — it runs when the image is being built.

**Shell form vs Exec form for RUN**:

```dockerfile
# Shell form — runs through /bin/sh -c
RUN npm install
RUN apt-get install -y curl
# Advantage: can use shell features (pipes, &&, etc.)
# Disadvantage: adds a /bin/sh overhead

# Exec form — runs the command directly
RUN ["npm", "install"]
RUN ["apt-get", "install", "-y", "curl"]
# Advantage: more explicit, no shell overhead
# Disadvantage: can't use shell features directly
```

For `RUN`, the shell form is almost always what you want because it lets you chain commands with `&&`.

**The critical cleanup rule — same layer cleanup**:

This is a very important optimization that's often missed.

When a layer is added to an image, its content is permanent. If layer 3 adds 100MB of apt cache, and layer 4 deletes that cache, the image still contains 100MB of apt cache (in layer 3) even though layer 4 deleted it. Layers are additive — deletions in later layers don't actually remove bytes from earlier layers.

```dockerfile
# ❌ Cache is in layer 2, delete attempt is in layer 3
# Net result: cache bytes are permanently in the image
RUN apt-get update                           # Layer A: apt cache downloaded
RUN apt-get install -y curl                  # Layer B: curl installed
RUN rm -rf /var/lib/apt/lists/*              # Layer C: marks cache as deleted
                                             # BUT layer A's bytes are still in image!

# ✅ All in one layer — cache never committed to any layer
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        curl \
        wget \
        ca-certificates \
    && rm -rf /var/lib/apt/lists/*           # Cleanup in same RUN = truly gone
```

**Multi-package install template**:

```dockerfile
# Debian/Ubuntu
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        curl \
        ca-certificates \
        git \
    && rm -rf /var/lib/apt/lists/*
# --no-install-recommends: skip recommended packages (reduces size)
# rm -rf /var/lib/apt/lists/*: remove apt package lists (not needed after install)

# Alpine
RUN apk add --no-cache \
    curl \
    ca-certificates \
    git
# --no-cache: don't create a local cache (saves space automatically)

# CentOS/RHEL
RUN yum install -y \
    curl \
    ca-certificates \
    && yum clean all \
    && rm -rf /var/cache/yum
```

**Using build arguments in RUN**:

```dockerfile
ARG NODE_VERSION=20
RUN curl -fsSL https://deb.nodesource.com/setup_${NODE_VERSION}.x | bash - && \
    apt-get install -y nodejs
```

**Common RUN patterns**:

```dockerfile
# Install Node.js packages (production only)
RUN npm ci --only=production

# Install Python packages
RUN pip install --no-cache-dir -r requirements.txt

# Build a Go binary
RUN CGO_ENABLED=0 GOOS=linux go build -a -o server .

# Create directories
RUN mkdir -p /app/logs /app/uploads /tmp/cache

# Set file permissions
RUN chmod +x /app/entrypoint.sh

# Create a non-root user
RUN addgroup --system --gid 1001 appgroup && \
    adduser --system --uid 1001 --ingroup appgroup appuser
```

### 6.5 ENV — Environment Variables, Their Scope, and the Secrets Warning

```dockerfile
ENV <key>=<value>
ENV KEY1=value1 KEY2=value2    # Multiple on one line
```

`ENV` sets environment variables that are available both:
1. **During subsequent build steps** (other `RUN` instructions can use the variable)
2. **When the container runs** (the variable is in the environment of the container's process)

This is unlike `ARG`, which only exists during build.

**Build-time use of ENV**:

```dockerfile
ENV APP_HOME=/app
WORKDIR $APP_HOME               # Uses the ENV variable
COPY --chown=$APP_HOME . .      # Uses the ENV variable
```

**Runtime use of ENV**:

```dockerfile
ENV NODE_ENV=production
ENV PORT=3000
ENV LOG_LEVEL=info
# These are in the container's environment when it runs:
# process.env.NODE_ENV === 'production'
# process.env.PORT === '3000'
```

**Overriding ENV at runtime**:

```bash
# Values set in ENV are defaults — they can be overridden at runtime
docker run -e NODE_ENV=development myapp
docker run -e PORT=4000 myapp

# Or override in docker-compose.yml
environment:
  NODE_ENV: staging
```

**The non-obvious scope gotcha**:

`ENV` values persist in the image metadata. Every container created from the image will have those environment variables. Be very careful about what you put in `ENV`.

**NEVER put secrets in ENV**:

```dockerfile
# ❌ This is in the image layers permanently
# Anyone who can pull the image can read this
ENV DATABASE_PASSWORD=super_secret_password
ENV AWS_SECRET_KEY=abc123xyz

# ✅ Inject at runtime
# docker run -e DATABASE_PASSWORD=$DATABASE_PASSWORD myapp
# Or use Docker secrets / Kubernetes secrets
```

Even if you try to unset it later:
```dockerfile
ENV SECRET_KEY=mysecret     # Added to layer N
RUN unset SECRET_KEY        # "Removed" in layer N+1
# But layer N is still in the image — docker history shows it
# Anyone can pull the image and inspect layer N to find the secret
```

**Good uses of ENV**:

```dockerfile
# Application defaults (non-sensitive)
ENV NODE_ENV=production
ENV PORT=3000
ENV LOG_LEVEL=info
ENV WORKERS=4

# Version pinning (useful reference)
ENV APP_VERSION=1.2.3

# Path additions
ENV PATH="/app/bin:$PATH"

# Application home directory
ENV APP_HOME=/app
```

### 6.6 ARG — Build-Time Variables That Don't Persist

```dockerfile
ARG <name>[=<default>]
```

`ARG` defines a variable that can be passed to the build process with `--build-arg`. Unlike `ENV`, `ARG` values are NOT available in the running container — they only exist during the build.

```dockerfile
ARG BUILD_VERSION=dev          # Default value
ARG REGISTRY=docker.io        # No default — must be passed at build time

RUN echo "Building version $BUILD_VERSION"
# ^ ARG is available here

CMD ["node", "server.js"]
# At runtime, BUILD_VERSION is NOT available in the container's environment
```

**Passing ARG values at build time**:

```bash
docker build \
  --build-arg BUILD_VERSION=1.2.3 \
  --build-arg REGISTRY=ghcr.io \
  -t myapp .
```

**Using ARG to set ENV (common pattern)**:

```dockerfile
ARG BUILD_ENV=production
ENV NODE_ENV=$BUILD_ENV    # Transfer ARG into ENV so it's available at runtime
# Now NODE_ENV=production is in the container's environment
# But it can be overridden at build time: --build-arg BUILD_ENV=staging
```

**ARG and cache behavior**:

`ARG` affects cache: if the ARG value changes between builds, any `RUN` instructions after the ARG declaration are invalidated. This is by design — if you pass a different version number, the build should re-run with the new value.

```dockerfile
FROM node:20-alpine
ARG APP_VERSION                   # Cache works up to here
RUN echo $APP_VERSION > VERSION   # Cache invalidated if APP_VERSION changes
COPY . .                          # Also invalidated (cascading)
```

**Security note**: `ARG` values are NOT secret. They appear in `docker history` output and in build metadata. Don't use `ARG` for secrets either.

### 6.7 EXPOSE — The Most Misunderstood Instruction

```dockerfile
EXPOSE <port>[/<protocol>]

EXPOSE 3000
EXPOSE 443/tcp
EXPOSE 53/udp
EXPOSE 8080 9090    # Multiple ports
```

`EXPOSE` is the instruction most misunderstood by Docker beginners. Let's be completely clear:

**EXPOSE does NOT open any port. It does NOT make the port accessible from outside the container. It does absolutely nothing at runtime.**

`EXPOSE` is purely **documentation**. It tells humans reading the Dockerfile (and Docker tooling) which port the application inside is designed to listen on.

**What EXPOSE actually does**:

1. Documents which port the app uses — future maintainers see it
2. When you use `docker run -P` (capital P), Docker uses EXPOSE information to decide which ports to publish randomly
3. Some Docker tooling and orchestration systems use EXPOSE as metadata

**What EXPOSE does NOT do**:

```dockerfile
EXPOSE 3000
# This does NOT:
# - Open port 3000 for incoming connections
# - Make localhost:3000 work from the host
# - Bind anything to any network interface
# - Configure iptables rules
```

**The mental model**: Think of `EXPOSE` like a "suggested port" annotation in code comments. It tells you what port to use, but doesn't set it up.

**How to actually make a port accessible**:

```bash
# The -p flag is what actually makes the port accessible
docker run -p 3000:3000 myapp
#              ↑     ↑
#          host   container
#          port    port

# Without -p, even with EXPOSE 3000, the port is NOT accessible from the host
docker run myapp           # Port 3000 accessible only inside container, not from host
```

**EXPOSE plus -p together**:

```bash
# The combination:
# 1. EXPOSE 3000 in Dockerfile: "my app listens on 3000"
# 2. -p 3000:3000 at runtime: "bridge host port 3000 to container port 3000"
# Both are needed for the port to be accessible and self-documenting
```

### 6.8 CMD vs ENTRYPOINT — The Complete, Unambiguous Explanation

This is the instruction pair that confuses the most developers. Let's resolve the confusion once and for all.

**The fundamental purpose of each**:

- `CMD`: The **default command** to run when a container starts. Can be completely replaced by passing a command to `docker run`.
- `ENTRYPOINT`: The **executable** — the program the container always runs. Cannot be easily replaced (requires `--entrypoint` flag).

**Both do NOT run during build**. They are stored as metadata in the image and executed when a container is created.

**CMD — The Flexible Default**:

```dockerfile
CMD ["node", "server.js"]
# This tells Docker: "by default, run 'node server.js' when this container starts"

# It runs when:
docker run myapp            # → runs: node server.js

# It is REPLACED when:
docker run myapp worker.js  # → runs: worker.js   (CMD is gone!)
docker run myapp /bin/sh    # → runs: /bin/sh      (CMD is gone!)
docker run myapp            # → runs: node server.js (CMD used when nothing overrides)
```

CMD can be overridden completely just by adding arguments to `docker run`.

**ENTRYPOINT — The Fixed Executable**:

```dockerfile
ENTRYPOINT ["node"]
# This tells Docker: "this container always runs 'node'. Arguments may change."

docker run myapp             # → runs: node       (no arguments — likely an error)
docker run myapp server.js   # → runs: node server.js  (server.js is arg to node)
docker run myapp --version   # → runs: node --version
docker run myapp -e          # → runs: node -e     (evaluate JS)
```

With ENTRYPOINT alone, anything you pass to `docker run` becomes an argument to node.

**CMD + ENTRYPOINT Together — The Standard Pattern**:

```dockerfile
ENTRYPOINT ["node"]           # Fixed: always run node
CMD ["server.js"]             # Default argument: server.js

docker run myapp              # → runs: node server.js  (ENTRYPOINT + CMD combined)
docker run myapp worker.js    # → runs: node worker.js  (CMD replaced, ENTRYPOINT stays)
docker run myapp --inspect server.js  # → runs: node --inspect server.js
```

When you combine them: ENTRYPOINT is the program, CMD is the default arguments to that program. You can replace the arguments without replacing the program.

**Overriding ENTRYPOINT** (requires explicit flag):

```bash
docker run --entrypoint /bin/sh myapp    # Override the entrypoint
docker run --entrypoint python myapp     # Replace node with python
# This is for debugging/inspection — not for normal use
```

**Shell form vs Exec form — and why it matters for signals**:

```dockerfile
# Shell form (string, no brackets)
CMD npm start                    # Runs as: /bin/sh -c npm start
ENTRYPOINT node server.js        # Runs as: /bin/sh -c node server.js

# Exec form (JSON array, with brackets)
CMD ["npm", "start"]             # Runs npm directly, no shell
ENTRYPOINT ["node", "server.js"] # Runs node directly, no shell
```

**Why exec form is almost always the right choice**:

With shell form, `/bin/sh` becomes PID 1, and your program (node, python, etc.) is a child of the shell. When Docker sends `SIGTERM` (to gracefully stop the container), it goes to PID 1 (`/bin/sh`), which may or may not forward it to your program. After 10 seconds of no response, Docker sends `SIGKILL` — immediate, unclean termination.

With exec form, your program is PID 1. It receives `SIGTERM` directly and can handle it gracefully — finishing in-flight requests, closing database connections, flushing logs.

```
Shell form:
docker stop → SIGTERM → /bin/sh (PID 1) → may not forward to node → 10s → SIGKILL → crash

Exec form:
docker stop → SIGTERM → node (PID 1) → graceful shutdown handler → clean exit
```

**Decision guide**:

| Scenario | Use |
|---|---|
| Simple default command, easily overridable | `CMD ["cmd", "arg"]` |
| Container that always runs one program | `ENTRYPOINT ["program"]` |
| Program with overridable default arguments | Both: `ENTRYPOINT ["program"]` + `CMD ["default-arg"]` |
| Script wrapper with fixed behavior | `ENTRYPOINT ["/entrypoint.sh"]` |

**Real-world examples**:

```dockerfile
# A web server (simple CMD)
FROM node:20-alpine
WORKDIR /app
COPY . .
CMD ["node", "server.js"]   # Can be overridden to run other scripts

# A database utility (ENTRYPOINT + CMD)
FROM postgres:15-alpine
ENTRYPOINT ["postgres"]     # Always run postgres
CMD ["-c", "config_file=/etc/postgresql/postgresql.conf"]  # Default config

# A CLI tool wrapper
FROM python:3.11-slim
COPY tool.py /usr/local/bin/tool
ENTRYPOINT ["python", "/usr/local/bin/tool"]
# docker run mytool analyze --file input.txt
# → python /usr/local/bin/tool analyze --file input.txt

# An init script (entrypoint script)
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]  # Script runs first (can set up env, wait for services)
CMD ["node", "server.js"]       # Script eventually execs this
```

### 6.9 USER — Running as Non-Root, Why It Matters

```dockerfile
USER <user>[:<group>]
USER 1001
USER appuser
USER appuser:appgroup
```

`USER` sets the user (and optionally group) for subsequent `RUN`, `CMD`, and `ENTRYPOINT` instructions, and for the running container.

**Why you should never run containers as root**:

By default, Docker containers run as root (UID 0). This is a significant security risk:

1. If your application has a security vulnerability, an attacker has root access inside the container
2. Root inside the container can access bind-mounted host directories with root permissions
3. In misconfigured environments or with container escape vulnerabilities, root in container = root on host
4. If container writes files to a bind-mounted directory, those files are owned by root on the host

**The pattern for running as non-root**:

```dockerfile
FROM node:20-alpine

# Create a non-root group and user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodeuser -u 1001 -G nodejs

WORKDIR /app

# Copy files with correct ownership in one step (more efficient than chown)
COPY --chown=nodeuser:nodejs package.json package-lock.json ./
RUN npm ci --only=production

COPY --chown=nodeuser:nodejs . .

# Switch to non-root user for all subsequent instructions and runtime
USER nodeuser

EXPOSE 3000
CMD ["node", "server.js"]
```

**Ports below 1024 require root** (or special capabilities). If your app listens on port 80 or 443, you have options:

```dockerfile
# Option 1: Listen on port 8080 internally, map to 80 externally
EXPOSE 8080
# docker run -p 80:8080 myapp

# Option 2: Grant NET_BIND_SERVICE capability without full root
# (in docker-compose.yml or docker run)
# cap_add: [NET_BIND_SERVICE]
```

**The node:20 image already has a user**:

The official Node.js images come with a pre-created `node` user (UID 1000). You can use it directly:

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY --chown=node:node . .
USER node
CMD ["node", "server.js"]
```

### 6.10 LABEL — Metadata for Your Images

```dockerfile
LABEL <key>=<value> [<key>=<value> ...]

LABEL version="1.2.3"
LABEL description="My web application"
LABEL maintainer="team@example.com"
LABEL org.opencontainers.image.source="https://github.com/myorg/myrepo"
```

`LABEL` adds metadata to an image. Labels are key-value pairs that can be read with `docker inspect`.

Labels don't create significant layers (they add to image manifest metadata). They're useful for:

- Tracking image versions and build metadata
- Providing contact information
- CI/CD integration (including commit SHA, build URL, etc.)
- Organizational policies (cost center, team, environment)

**The OCI standard labels** (recommended):

```dockerfile
LABEL \
  org.opencontainers.image.title="My App" \
  org.opencontainers.image.description="My web application" \
  org.opencontainers.image.version="1.2.3" \
  org.opencontainers.image.created="2024-01-15T10:00:00Z" \
  org.opencontainers.image.source="https://github.com/myorg/myapp" \
  org.opencontainers.image.revision="abc123def456" \
  org.opencontainers.image.authors="team@example.com"
```

**Reading labels**:

```bash
docker inspect myapp:latest | grep -A 20 '"Labels"'
docker inspect --format='{{json .Config.Labels}}' myapp:latest | jq
```

### 6.11 VOLUME — Marking Mount Points

```dockerfile
VOLUME ["/data"]
VOLUME /var/log/myapp /var/data/myapp
```

`VOLUME` marks a directory as a mount point and indicates that the directory should be managed externally (not stored in the container's writable layer).

**What VOLUME does**:

1. If you run a container without explicitly mounting anything at that path, Docker creates an **anonymous volume** automatically and mounts it there
2. It signals to users and orchestration tools that this path expects external storage

**What VOLUME does NOT do**:

- It does NOT create a named volume (you specify the name with `-v` at runtime)
- It does NOT make data persist automatically in a usable way
- It does NOT override explicit mounts you specify at runtime

**When VOLUME in Dockerfile is useful**:

```dockerfile
# Database images use VOLUME to prevent database files
# from ending up in the writable layer (slow writes)
VOLUME /var/lib/postgresql/data   # PostgreSQL's data directory

# If you run the container without -v, Docker auto-creates
# an anonymous volume at this path
```

**The truth about VOLUME in Dockerfiles**:

Most applications don't need `VOLUME` in their Dockerfile. It's more of a declaration to users: "this path is where data lives, please mount something here." For application images (web servers, APIs), it's often omitted. It's most useful for databases and other stateful services where the data directory is well-defined.

### 6.12 HEALTHCHECK — Teaching Docker When Your App Is Ready

```dockerfile
HEALTHCHECK [OPTIONS] CMD command
HEALTHCHECK NONE   # Disable any inherited health check

Options:
  --interval=30s     # How often to run (default: 30s)
  --timeout=30s      # Max time for check to complete (default: 30s)
  --start-period=0s  # Initialization grace period before checks count (default: 0s)
  --retries=3        # Consecutive failures before marking unhealthy (default: 3)
```

`HEALTHCHECK` tells Docker how to test whether the container is working properly. This is different from whether the container is running — a container can be "running" but in a broken state (e.g., web server crashed, database not accepting connections).

**What HEALTHCHECK enables**:

1. `depends_on` with `condition: service_healthy` in Compose
2. Container orchestrators (Kubernetes, Swarm) to route traffic only to healthy containers
3. `docker ps` shows `(healthy)` or `(unhealthy)` status
4. Automatic restart policies can trigger on unhealthy state

**Examples for different application types**:

```dockerfile
# Web application — check HTTP endpoint
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

# Postgres — check if server is accepting connections
HEALTHCHECK --interval=10s --timeout=5s --retries=5 \
  CMD pg_isready -U postgres || exit 1

# Redis — ping
HEALTHCHECK --interval=10s --timeout=3s \
  CMD redis-cli ping || exit 1

# Custom health check script
COPY healthcheck.sh /healthcheck.sh
RUN chmod +x /healthcheck.sh
HEALTHCHECK --interval=30s CMD /healthcheck.sh
```

**The health check command exit codes**:
- Exit code 0: healthy
- Exit code 1: unhealthy
- Exit code 2: reserved (do not use)

**Important**: Alpine images don't have `curl` by default. Either install it or use `wget`:

```dockerfile
FROM node:20-alpine
RUN apk add --no-cache curl   # Add curl for health check

HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:3000/health || exit 1
```

Or use `wget` (available on Alpine):

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget -qO- http://localhost:3000/health || exit 1
```

### 6.13 Build Context and .dockerignore — What Docker Can See

When you run `docker build .`, the `.` is the **build context** — the directory whose contents Docker has access to for `COPY` and `ADD` instructions.

**How the build context works**:

Docker's build process works as follows:
1. The Docker client scans the build context directory
2. It sends the files to the Docker daemon (or uses BuildKit to send only what's needed)
3. `COPY` and `ADD` instructions can only reference files within this context

The build context is about what Docker **can see**, not necessarily what ends up in the image.

**Why the build context matters for build speed**:

Even with BuildKit's lazy loading, Docker still needs to scan the directory to know what's there. If your project directory has:

```
myproject/
├── .git/                (50MB of git history)
├── node_modules/        (200MB of dependencies)
├── src/                 (5MB of your code)
├── data/                (2GB of test data)
└── Dockerfile
```

Without .dockerignore, `docker build .` scans all 2.25GB even if you're only copying `src/`.

**The .dockerignore file**:

`.dockerignore` works exactly like `.gitignore` — each line is a pattern, and matching files/directories are excluded from the build context entirely.

```
# .dockerignore — comprehensive example

# Version control
.git
.gitignore
.gitattributes

# Dependencies (rebuild inside the image)
node_modules
vendor/
__pycache__/
*.pyc
*.pyo
*.pyd
.Python

# Build artifacts
dist/
build/
*.egg-info/
target/

# Test files
test/
tests/
spec/
*.test.js
*.spec.ts
coverage/
.nyc_output/

# Development tools
.vscode/
.idea/
*.swp
*.swo
.DS_Store
Thumbs.db

# Docker files (meta-recursive, these don't need to be in the image)
Dockerfile
Dockerfile.*
docker-compose*.yml
.dockerignore

# Environment files — CRITICAL: keep secrets out of images
.env
.env.*
*.env
!.env.example    # Exception: include the example file

# Documentation
docs/
*.md
README*
CHANGELOG*
LICENSE

# Logs
logs/
*.log

# Large data files
data/
fixtures/
*.sql
*.dump
```

**Verifying your .dockerignore works**:

```bash
# See how large your build context is
docker build . 2>&1 | head -1
# → Sending build context to Docker daemon  5.12MB   ← Good (down from 2.25GB)

# Or use --dry-run to list what would be included (Docker Buildx)
docker buildx build --no-cache --progress=plain . 2>&1 | grep "context"
```

### 6.14 Multi-Stage Builds — The Pattern for Production-Ready Images

Multi-stage builds allow you to use multiple `FROM` instructions in a single Dockerfile. Each `FROM` starts a new stage with a fresh filesystem. You can selectively copy artifacts from one stage into another.

**The problem they solve — build dependencies in production images**:

Building software requires tools that are not needed to run it:
- TypeScript compilation needs the TypeScript compiler
- Go compilation needs the Go toolchain (~1GB)
- Java compilation needs the full JDK (Maven, javac)
- Testing needs test frameworks and fixtures
- Bundling needs webpack, rollup, etc.

Without multi-stage builds, your production image includes all these build tools. With multi-stage builds, you build in one stage and copy only the compiled output to a minimal production stage.

**Basic multi-stage example**:

```dockerfile
# ════════════════════════════════════════════
# Stage 1: Build
# ════════════════════════════════════════════
FROM node:20 AS builder

WORKDIR /build

# Install all dependencies (including devDependencies)
COPY package.json package-lock.json ./
RUN npm ci

# Copy source and build
COPY . .
RUN npm run build     # Compiles TypeScript, bundles, etc.
# Result: /build/dist/ contains compiled output

# ════════════════════════════════════════════
# Stage 2: Production
# ════════════════════════════════════════════
FROM node:20-alpine AS production
# Fresh start! Nothing from Stage 1 is here yet.
# We're starting with just the Alpine + Node minimal image.

WORKDIR /app

# Copy only production dependency definitions
COPY package.json package-lock.json ./
# Install only production dependencies (no devDependencies)
RUN npm ci --only=production

# Copy ONLY the compiled output from the builder stage
COPY --from=builder /build/dist ./dist
# This is the magic: --from=builder copies from the builder stage's filesystem

USER node
EXPOSE 3000
CMD ["node", "dist/server.js"]

# Result: production image contains:
# - Alpine Linux (~5MB)
# - Node.js runtime (~175MB)
# - Production node_modules (varies, maybe 50MB)
# - Your compiled app dist (~500KB)
# Total: ~230MB

# WITHOUT multi-stage:
# - Full Debian (~100MB)
# - Node.js + npm (~200MB)
# - ALL node_modules including devDeps (~300MB)
# - TypeScript compiler, webpack, etc. (~200MB)
# - Source code, test files, etc.
# Total: ~800MB+
```

**Multi-stage for different languages**:

```dockerfile
# ════════════ Go — ultimate minimal production image ════════════
FROM golang:1.21-alpine AS builder
WORKDIR /src

# Download dependencies first (cache)
COPY go.mod go.sum ./
RUN go mod download

# Build the binary
COPY . .
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 \
    go build -ldflags="-w -s" -o /bin/server .
# -w: omit DWARF debug info
# -s: omit symbol table
# CGO_ENABLED=0: static binary, no C library dependencies

FROM scratch AS production    # Empty image — literally nothing
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
COPY --from=builder /bin/server /server
EXPOSE 8080
ENTRYPOINT ["/server"]
# Result: ~5-10MB image (just the binary + TLS certs)

# ════════════ Python ════════════
FROM python:3.11 AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir --target=/app/deps -r requirements.txt

FROM python:3.11-slim AS production
WORKDIR /app
COPY --from=builder /app/deps /usr/local/lib/python3.11/site-packages
COPY . .
USER nobody
CMD ["python", "app.py"]

# ════════════ Java/Maven ════════════
FROM maven:3.9-openjdk-17 AS builder
WORKDIR /build
COPY pom.xml .
RUN mvn dependency:resolve -B -q    # Download deps without source
COPY src ./src
RUN mvn package -DskipTests -B -q  # Build jar

FROM eclipse-temurin:17-jre-alpine AS production
# eclipse-temurin JRE: runtime only, no compiler (~200MB vs 600MB for full JDK)
WORKDIR /app
COPY --from=builder /build/target/app.jar app.jar
USER 1001
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```

**Multiple stages for testing**:

```dockerfile
FROM node:20-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

# ─── Test stage (runs during CI, not in production image) ───
FROM deps AS test
COPY . .
RUN npm test     # Fail the build if tests fail
RUN npm run lint

# ─── Build stage (runs only if tests pass) ───
FROM deps AS builder
COPY . .
RUN npm run build

# ─── Production (only the compiled output) ───
FROM node:20-alpine AS production
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY --from=builder /app/dist ./dist
CMD ["node", "dist/server.js"]

# Run only the test stage (for CI validation):
# docker build --target test .

# Run only the production stage:
# docker build --target production .
# docker build .  (production is the last stage, so it's the default)
```

---

## 7. Volumes

### The Complete Solution to Persistence

You've built the mental model: containers are processes with ephemeral writable layers. That's the right model. But real applications need data that outlives individual container instances. Volumes are Docker's answer.

### 7.1 The Root Cause of Data Disappearance — Deeply Explained

We covered this in Section 4.3, but let's restate it with full precision because understanding the root cause is what makes the solution (volumes) make intuitive sense.

A container's writable layer is stored on the host in Docker's managed storage area (typically `/var/lib/docker/overlay2/<container-id>/`). This storage is:

1. **Tied to the container's identity**: When the container is removed, Docker removes this directory.
2. **Not portable**: It can't be easily moved to another container.
3. **Not independently accessible**: You can't easily read/write it from outside Docker.

The key phrase: **when the container is removed**. Many developers confuse "removing" with "stopping":

- `docker stop` → container stops, writable layer on disk, can be restarted
- `docker rm` → container AND writable layer deleted permanently
- `docker compose down` → removes containers (like docker rm for all services)
- `docker compose up` → creates NEW containers with NEW empty writable layers

```bash
# The exact sequence that causes data loss:
docker compose up -d       # Creates containers with empty writable layers
# ... use app, data accumulates in writable layers ...
docker compose down        # Removes containers + their writable layers ← DATA GONE
docker compose up -d       # Creates NEW containers with NEW empty writable layers
# ← Database has no records, uploads are gone, logs are gone
```

**The solution philosophy**: For data that needs to outlive containers, that data should be stored OUTSIDE the container's writable layer — in a volume.

### 7.2 Named Volumes — Docker-Managed, Lifecycle-Independent Storage

A named volume is a piece of storage managed by Docker that exists completely independently of any container. Docker decides where it lives on the host filesystem (you don't manage this), and it persists until you explicitly delete it — regardless of how many containers using it have been created, stopped, or removed.

**Creating and using named volumes**:

```bash
# Create a volume explicitly
docker volume create pgdata

# Or let Docker create it automatically when first referenced
docker run \
  --name mydb \
  -v pgdata:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=secret \
  postgres:15

# The syntax:
# -v <volume-name>:<container-path>
#    named volume    where to mount it inside container
```

**Proving persistence**:

```bash
# Start postgres with named volume
docker run -d --name mydb \
  -v pgdata:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=secret \
  postgres:15

# Wait for postgres to start, then insert data
docker exec mydb psql -U postgres -c "
  CREATE TABLE users (id SERIAL, name TEXT);
  INSERT INTO users VALUES (1, 'Alice');
  INSERT INTO users VALUES (2, 'Bob');
"

# Remove the container completely
docker rm -f mydb

# Create a brand new container using the SAME volume
docker run -d --name mydb-new \
  -v pgdata:/var/lib/postgresql/data \     ← Same volume name
  -e POSTGRES_PASSWORD=secret \
  postgres:15

# Query the new container
docker exec mydb-new psql -U postgres -c "SELECT * FROM users;"
# → id | name
# → ----+-------
# → 1  | Alice     ← DATA IS STILL HERE
# → 2  | Bob       ← DATA IS STILL HERE
# The container was replaced, but the volume preserved everything
```

**Where named volumes are stored on the host**:

```bash
docker volume inspect pgdata
[
    {
        "CreatedAt": "2024-01-15T10:00:00Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/pgdata/_data",
        "Name": "pgdata",
        "Options": {},
        "Scope": "local"
    }
]
# The data is at /var/lib/docker/volumes/pgdata/_data on the host
# Docker manages this location — you don't need to care about it
# But you CAN access it directly (as root) for backup or inspection
```

**Volume lifecycle operations**:

```bash
# List all volumes
docker volume ls
DRIVER    VOLUME NAME
local     pgdata
local     uploads
local     redisdata

# Inspect a volume
docker volume inspect pgdata

# Remove a specific volume (only works if no container is using it)
docker volume rm pgdata

# Remove all unused volumes (volumes not mounted by any container)
docker volume prune
# ⚠️ WARNING: This permanently deletes all unused volumes and their data

# See which containers are using a volume
docker ps -a --filter volume=pgdata
```

**Backup and restore patterns**:

```bash
# Backup a volume to a tar file
docker run --rm \
  -v pgdata:/source:ro \           # Mount volume as read-only
  -v $(pwd):/backup \              # Mount current dir for output
  alpine \
  tar czf /backup/pgdata-backup-$(date +%Y%m%d).tar.gz -C /source .

# Restore a volume from a tar file
docker volume create pgdata-restored   # Create empty volume
docker run --rm \
  -v pgdata-restored:/target \         # Mount empty volume
  -v $(pwd):/backup:ro \               # Mount backup dir as read-only
  alpine \
  tar xzf /backup/pgdata-backup-20240115.tar.gz -C /target

# Copy volume data to another volume (e.g., for migration)
docker run --rm \
  -v pgdata:/source:ro \
  -v pgdata-new:/target \
  alpine \
  sh -c "cp -r /source/. /target/"
```

### 7.3 Bind Mounts — Direct Host Filesystem Access

A bind mount maps a specific path on your host machine directly into the container. It's not managed by Docker — you specify exactly which host path to mount.

**The fundamental difference from named volumes**: With a named volume, Docker controls where on the host the data lives. With a bind mount, YOU control it — it's a specific directory on your host filesystem.

```bash
# Syntax:
docker run -v /absolute/host/path:/container/path image
docker run -v $(pwd):/app image           # Mount current directory
docker run -v $(pwd)/src:/app/src image   # Mount subdirectory

# The --mount equivalent (more explicit):
docker run \
  --mount type=bind,source=$(pwd),target=/app \
  image
```

**Why bind mounts exist and why they're essential for development**:

During development, you want to:
1. Edit code in your IDE on your local machine
2. See changes reflected immediately in the running container
3. NOT rebuild the Docker image on every code change

Bind mounts make this possible. Your local source code directory IS the container's source code directory. They share the same files at the OS level — there's no copying, no syncing, it's the same inode.

```yaml
# Development docker-compose.yml
services:
  app:
    build: .
    volumes:
      - .:/app                   # ← Bind mount: host's current dir = container's /app
    ports:
      - "3000:3000"
    command: npm run dev          # Hot-reload dev server

# Workflow:
# 1. docker compose up -d
# 2. Edit src/server.js in VS Code
# 3. The dev server inside the container sees the change immediately
# 4. Hot reload happens — no rebuild needed
```

**The bind mount experience**:

```
Your IDE       ←→    Host filesystem   ←→    Container
edits file           /home/you/app/           /app/
saves file           server.js (changed)      server.js (same file!)
                     ↑────────────────────────↑
                     These are the SAME file on disk
```

**When bind mounts reflect the host's filesystem truthfully**:

```bash
# If you bind mount a directory with files:
docker run -v $(pwd)/config:/app/config myapp
# → Container sees the config files from your host

# Create a file on the host:
echo "new config" > $(pwd)/config/new.conf
# → Immediately visible inside container at /app/config/new.conf

# Delete a file from the container:
docker exec myapp rm /app/config/old.conf
# → File is deleted from your host too!
# Bind mounts are two-way — changes propagate in both directions
```

**The node_modules problem with bind mounts**:

This is a gotcha that trips up almost everyone the first time.

Suppose your Dockerfile installs packages during build (`RUN npm install`), so `/app/node_modules` exists in the image. Then you bind mount your local directory (which doesn't have `node_modules` — or has a different version):

```yaml
volumes:
  - .:/app    # This overwrites /app completely, including /app/node_modules
              # Now node_modules from the IMAGE is hidden by the empty/different host dir
              # Your app can't find its packages → crash
```

**The solution**: Use an anonymous volume at the node_modules path to "protect" it from being overwritten by the bind mount:

```yaml
volumes:
  - .:/app                  # Bind mount: your code
  - /app/node_modules       # Anonymous volume: protects node_modules installed in image
```

This works because Docker applies volumes in order, and the more specific anonymous volume at `/app/node_modules` takes precedence over the bind mount at `/app` for that specific subdirectory.

### 7.4 Anonymous Volumes — The Invisible Third Type

Anonymous volumes are created when you specify a container path in `-v` without giving the volume a name, or when a Dockerfile has `VOLUME /path` and you run the container without mounting anything there.

```bash
# Creates an anonymous volume (no name before the colon)
docker run -v /app/node_modules myapp   # ← anonymous volume at /app/node_modules
docker run -v /var/lib/mysql mydb       # ← anonymous volume at /var/lib/mysql

# Anonymous volumes get auto-generated names:
docker volume ls
DRIVER    VOLUME NAME
local     a1b2c3d4e5f6g7h8i9j0k1l2m3n4   ← ugly, unrecognizable name
```

**When anonymous volumes are useful**:

1. In development, protecting specific paths from being overwritten by bind mounts (the node_modules pattern above)
2. When you need temporary persistence for a single container run but don't care about naming
3. When `VOLUME` in Dockerfile triggers auto-creation

**The downside of anonymous volumes**:

```bash
# Unmanageable: docker compose down does NOT remove anonymous volumes
# They accumulate over time
docker volume ls   # Dozens of volumes with unrecognizable IDs

# Clean them up:
docker volume prune   # Removes all unused anonymous AND named volumes ← DANGEROUS
```

Best practice: prefer named volumes over anonymous volumes for anything you care about.

### 7.5 Named Volumes vs Bind Mounts — The Decision Framework

| Consideration | Named Volume | Bind Mount |
|---|---|---|
| **Who controls where data lives on host** | Docker | You |
| **File location on host** | Docker-managed, hidden | Wherever you specify |
| **Can I see/edit files from host** | Yes, but at Docker's path | Yes, directly in your normal filesystem |
| **Performance (Linux)** | Excellent | Excellent |
| **Performance (Mac/Windows)** | Excellent (stored in VM) | Slower (crosses VM boundary) |
| **Portability** | Works identically on any host with Docker | Requires same host paths to exist |
| **Use for production data (databases)** | ✓ Ideal | ✗ Path management complexity |
| **Use for development code** | ✗ Inconvenient to edit | ✓ Ideal |
| **Use for config files** | ✓ If config is managed by Docker | ✓ If config lives in your project |
| **Backup complexity** | Requires docker run helper | Direct filesystem access |
| **Seeded with image data on first use** | ✓ Yes (if volume is empty) | ✗ No (obscures image files) |

**The decision in one paragraph**:

Use **named volumes** for anything that needs to persist reliably across container lifecycles in production — database files, persistent caches, user uploads. Docker handles the storage, you handle the data. Use **bind mounts** during development when you want to edit code locally and see it reflected immediately in the container, or when you need to feed specific host files (config, certificates, secrets) into a container.

### 7.6 The Volume Initialization Gotcha — Named vs Bind Behavior

This is subtle but important. Named volumes and bind mounts behave differently when mounted at a path that already has content in the image.

**Bind mount behavior — obscures image content**:

```dockerfile
# Image has files at /app/config:
# /app/config/default.json
# /app/config/settings.json
```

```bash
mkdir -p ./empty-dir   # An empty directory

docker run -v $(pwd)/empty-dir:/app/config myapp
# Inside container:
docker exec myapp ls /app/config
# → (nothing)   ← Image's config files are HIDDEN/OBSCURED
#                 The bind mount completely covers the image content
#                 The image files still exist in the layer, but can't be seen
```

**Named volume behavior — seeded from image on first use**:

```bash
docker volume create myconfig   # Empty named volume

docker run -v myconfig:/app/config myapp
# Inside container:
docker exec myapp ls /app/config
# → default.json    ← Image's files were COPIED INTO the volume!
# → settings.json   ← This happens only on FIRST USE of an empty named volume

# Remove container, run again with same volume
docker rm myapp_container
docker run -v myconfig:/app/config myapp
docker exec myapp ls /app/config
# → default.json    ← Whatever you left in the volume (not from image)
# → settings.json   ← Volume seeding only happens once (on first empty volume use)
```

**Why this matters**:

This is how PostgreSQL and other database images work. Their `VOLUME /var/lib/postgresql/data` combined with the named volume behavior means:
- First `docker run`: empty volume → Docker seeds it with PostgreSQL's initial data setup → database initializes
- Subsequent runs: volume has data → Docker does NOT re-seed → existing database data preserved

If you used a bind mount instead of a named volume, the initialization scripts would be obscured and PostgreSQL wouldn't initialize at all (or would see an apparently empty directory and try to initialize there).

**Practical implications**:

```yaml
# ✓ Named volume — PostgreSQL works correctly
services:
  db:
    image: postgres:15
    volumes:
      - pgdata:/var/lib/postgresql/data   # ← Named volume: gets seeded on first use

# ✗ Bind mount — PostgreSQL initialization may fail or have issues
services:
  db:
    image: postgres:15
    volumes:
      - ./pgdata:/var/lib/postgresql/data  # ← Bind mount: obscures any image content
```

### 7.7 Volume Management and Backup Strategies

**Listing and inspecting volumes**:

```bash
# See all volumes and their sizes
docker system df -v | grep -A 100 "Local Volumes"

# Find containers using a specific volume
docker ps -a --filter volume=pgdata

# See which volumes are unused (safe to remove)
docker volume ls --filter dangling=true

# Full inspection
docker volume inspect pgdata
```

**Production backup strategy**:

```bash
#!/bin/bash
# Backup all critical volumes

BACKUP_DIR="/backup/docker-volumes"
DATE=$(date +%Y%m%d_%H%M%S)
mkdir -p "$BACKUP_DIR"

# Backup PostgreSQL volume
docker run --rm \
  -v pgdata:/source:ro \
  -v "$BACKUP_DIR":/backup \
  alpine \
  tar czf "/backup/pgdata_$DATE.tar.gz" -C /source .

# Keep only last 7 daily backups
find "$BACKUP_DIR" -name "pgdata_*.tar.gz" -mtime +7 -delete

echo "Backup completed: pgdata_$DATE.tar.gz"
```

---

## 8. Networking

### The Complete Picture of Container Communication

Containers need to be isolated — that's the whole point. But isolation creates a challenge: isolated containers need to communicate with the outside world and with each other. Docker's networking layer is how this gets resolved.

### 8.1 Why Container Network Isolation Exists

Because containers have their own network namespace (recall Section 3.2), each container gets a completely isolated network stack. This means:

- Its own virtual network interface (eth0)
- Its own IP address (assigned from Docker's internal subnet)
- Its own routing table
- Its own port space
- Its own firewall rules (iptables)

**The benefit**: Two containers can both listen on port 3000 without conflict. Container A's port 3000 is completely separate from Container B's port 3000 — they're in different network namespaces.

**The challenge**: Since each container has its own isolated network namespace, it can't be reached from the host or from other containers by default. We need mechanisms to enable communication where needed.

Docker solves this with two mechanisms:
1. **Port mapping** (`-p`): host → container
2. **Docker networks**: container → container

### 8.2 What a Container's Network Namespace Contains

When Docker creates a container, it creates a new network namespace for it and sets up:

1. **A virtual network interface (veth pair)**: A virtual ethernet cable connecting the container's network namespace to Docker's internal bridge network. One end is inside the container (`eth0`), the other end is on the host's bridge (`docker0` or a custom bridge).

2. **An IP address**: Assigned from Docker's internal subnet (typically `172.17.0.0/16` for the default bridge, or `172.18.0.0/16`, `172.19.0.0/16`, etc. for custom networks).

3. **A loopback interface**: `127.0.0.1` / `localhost` inside the container.

4. **Routing rules**: Traffic destined for the internet is routed through the host's network.

```bash
# See the container's network setup from inside
docker exec myapp ip addr show
# 1: lo: <LOOPBACK,UP,...> inet 127.0.0.1/8 scope host lo
# 5: eth0@if6: <BROADCAST,MULTICAST,UP,...> inet 172.17.0.2/16 brd 172.17.255.255

docker exec myapp ip route show
# default via 172.17.0.1 dev eth0   ← Default route through Docker's bridge
# 172.17.0.0/16 dev eth0            ← Direct access to other containers on bridge

# See corresponding interfaces on the host
ip addr show docker0
# docker0: inet 172.17.0.1/16       ← Docker's bridge interface
```

### 8.3 Port Mapping — Every Detail of -p Explained

Port mapping creates a NAT (Network Address Translation) rule in the host's iptables that forwards traffic from a host port to a container port.

**Syntax breakdown**:

```
docker run -p [host-ip:]<host-port>:<container-port>[/protocol]

Examples:
-p 3000:3000              # All interfaces, host port 3000 → container port 3000
-p 8080:80                # All interfaces, host port 8080 → container port 80
-p 127.0.0.1:3000:3000   # Localhost only, host port 3000 → container port 3000
-p 3000:3000/tcp          # Explicit TCP (default)
-p 53:53/udp              # UDP protocol
-p 3000-3010:3000-3010    # Range of ports
-P                        # Publish all EXPOSE'd ports to random host ports
```

**What happens under the hood when you use -p**:

Docker modifies the host's iptables rules:

```bash
# This is approximately what happens when you run: docker run -p 8080:80 nginx

# Docker adds an iptables rule like:
iptables -t nat -A DOCKER \
  -p tcp \
  -m tcp --dport 8080 \
  -j DNAT \
  --to-destination 172.17.0.2:80
# Translation: TCP traffic arriving at host port 8080
#             → forward to 172.17.0.2 (container IP) port 80

# You can see these rules:
sudo iptables -t nat -L -n -v | grep 8080
```

**The host IP binding and its security implications**:

```bash
# Default: binds to 0.0.0.0 (all interfaces)
docker run -p 3000:3000 myapp
# → Accessible from:
#   localhost:3000 (your machine)
#   <your-LAN-IP>:3000 (other machines on your network)
#   <your-public-IP>:3000 (internet, if firewall allows)

# Secure: bind to localhost only
docker run -p 127.0.0.1:3000:3000 myapp
# → Only accessible from:
#   localhost:3000 (your machine only)
#   LAN and internet CANNOT reach it

# Good practice for development:
# Bind databases and admin tools to 127.0.0.1
docker run -p 127.0.0.1:5432:5432 postgres   # DB only accessible locally
docker run -p 127.0.0.1:8080:8080 adminer    # DB admin only accessible locally
```

**Running multiple instances on different ports**:

```bash
# Scenario: scale out your web server for testing
docker run -d --name web1 -p 3001:3000 myapp  # Instance 1 on host port 3001
docker run -d --name web2 -p 3002:3000 myapp  # Instance 2 on host port 3002
docker run -d --name web3 -p 3003:3000 myapp  # Instance 3 on host port 3003

# All three containers listen on port 3000 internally (no conflict — separate namespaces)
# They're differentiated by host-side port mapping
```

### 8.4 The Three Default Docker Networks

When you install Docker, it creates three networks automatically:

```bash
docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
abc123456789   bridge    bridge    local
def123456789   host      host      local
ghi123456789   none      null      local
```

**1. bridge — The default container network**

All containers you start without specifying a network go on the `bridge` network. Docker's bridge uses a virtual ethernet bridge interface (`docker0` on the host) to connect containers.

Characteristics:
- Containers get IPs in `172.17.0.0/16`
- Containers can communicate with each other via IP
- **No automatic DNS** — containers cannot reach each other by name on this network
- Containers can reach the internet (via NAT through the host)
- The host can reach containers via `-p` port mapping

**2. host — Bypass network isolation entirely**

The `host` network makes a container share the host's network namespace directly. There's no virtual interface, no IP translation, no port mapping needed — the container uses the host's network directly.

```bash
docker run --network host nginx
# nginx listening on port 80 = host's port 80 is now in use
# No -p needed — it's already on the host's network
```

Use cases: Performance-critical networking, when you need to access host network interfaces directly (monitoring tools, network debugging). Avoid in production — eliminates network isolation.

**3. none — No network access**

Containers on the `none` network have no network access at all — only a loopback interface. Useful for:
- Batch jobs that process local files and don't need network
- Maximum isolation for security-sensitive workloads
- Containers that communicate only via Unix sockets or shared volumes

```bash
docker run --network none myapp
# Inside: no eth0, only lo (loopback)
# Cannot reach the internet, cannot reach other containers
```

### 8.5 The Default Bridge Network Problem — Why DNS Doesn't Work

The default `bridge` network has a significant limitation: **it does not provide automatic DNS resolution between containers**.

On the default bridge network, containers can only communicate via IP address. Container IPs are assigned by Docker's internal DHCP and can change if containers are recreated (new `docker run`). This makes the default bridge network problematic for multi-container applications:

```bash
# Example of the problem:
docker run -d --name db postgres:15 -e POSTGRES_PASSWORD=secret
docker run -d --name api myapp

# What IP did db get?
docker inspect db | grep IPAddress
# → "IPAddress": "172.17.0.2"

# Try to connect api to db by name — FAILS on default bridge
docker exec api nslookup db
# → ;; connection timed out; no servers could be reached
# DNS doesn't know what "db" is on the default bridge network

# Have to use IP address:
docker exec api curl http://172.17.0.2:5432
# Works, but:
# 1. Hardcoded IP in your config — bad practice
# 2. IP changes if db is recreated — your config breaks
```

**Why the default bridge network lacks DNS**:

The default bridge is a legacy network from Docker's early days. Custom networks (introduced later) were designed with proper service discovery. Docker chose not to add DNS to the default bridge to avoid breaking existing configurations.

### 8.6 Custom Networks — How Docker DNS Actually Works

When you create a custom Docker network, Docker's embedded DNS server automatically registers all containers on that network by name. Any container on the network can resolve another container's name to its current IP.

**Creating and using a custom network**:

```bash
# Create a custom network
docker network create myapp-network

# Run containers on the custom network
docker run -d \
  --name db \
  --network myapp-network \
  -e POSTGRES_PASSWORD=secret \
  postgres:15

docker run -d \
  --name api \
  --network myapp-network \
  -e DATABASE_URL="postgresql://postgres:secret@db:5432/myapp" \
  myapp

# Now inside the api container:
docker exec api nslookup db
# Server: 127.0.0.11         ← Docker's embedded DNS server address
# Address: 127.0.0.11#53
#
# Name: db
# Address: 172.18.0.2        ← Resolved to db's current IP

# And the actual connection works:
docker exec api pg_isready -h db
# → db:5432 - accepting connections ← DNS resolved "db" successfully
```

**How Docker's embedded DNS works technically**:

Every container on a custom network has `127.0.0.11` set as its DNS server (via `/etc/resolv.conf`). This address points to Docker's embedded DNS server, which runs inside the Docker daemon.

```bash
# Inside a container on a custom network:
cat /etc/resolv.conf
# nameserver 127.0.0.11    ← Docker's DNS server
# options ndots:0

# When the container does: gethostbyname("db")
# 1. System sends DNS query to 127.0.0.11
# 2. Docker's DNS server checks which containers named "db" are on the same network
# 3. Returns the current IP of the container named "db"
# 4. Container connects to that IP
```

**What happens when the db container restarts and gets a new IP**:

```bash
docker restart db     # db restarts, might get IP 172.18.0.5 instead of 172.18.0.2

# From api container:
nslookup db
# Name: db
# Address: 172.18.0.5   ← Updated automatically! Docker's DNS always reflects current state
```

This is why you should always use service names (like `db`, `cache`, `redis`) in your application's connection strings rather than hardcoded IPs.

**Connecting a container to multiple networks**:

```bash
# A container can be on multiple networks simultaneously
docker network create frontend-net
docker network create backend-net

docker run -d --name api --network frontend-net myapp

# Connect to a second network after creation
docker network connect backend-net api

# Now api can reach both frontend and backend containers
# Containers on backend-net cannot reach containers only on frontend-net
```

### 8.7 Network Isolation as a Security Layer

Docker networks provide container-to-container isolation — you can control exactly which containers can reach which other containers.

**Multi-tier architecture with network isolation**:

```yaml
# docker-compose.yml
services:
  nginx:           # ← Public-facing, reverse proxy
    networks: [frontend]
    ports: ["80:80", "443:443"]

  api:             # ← Internal API
    networks: [frontend, backend]   # ← Connects frontend to backend

  db:              # ← Database
    networks: [backend]             # ← Only on backend

  cache:           # ← Cache
    networks: [backend]             # ← Only on backend

networks:
  frontend:        # nginx + api
  backend:         # api + db + cache
```

**What this network topology achieves**:

```
INTERNET
   ↓
nginx (frontend net only)
   ↓
api (both nets)
   ↓
db (backend net only)
cache (backend net only)

nginx CANNOT reach db directly (not on same network)
nginx CANNOT reach cache directly
api CAN reach db and cache (on backend net)
api CAN receive requests from nginx (on frontend net)
```

If an attacker compromises nginx (through an nginx vulnerability), they can only see the `api` service. They cannot directly query the database or cache, even from inside the nginx container.

**Container-level firewall rules are automatic**:

When containers are on separate networks, Docker automatically sets up iptables rules that block cross-network communication:

```bash
# This will fail — nginx is not on backend network
docker exec nginx psql -h db -U postgres
# → connect: Connection refused (iptables blocks it at the network level)
```

### 8.8 The host and none Network Drivers

**The host network driver**:

The `host` driver makes the container use the host's network namespace directly. No virtual interface. No IP translation. No port mapping needed.

```bash
docker run --network host nginx
# nginx listens on host's port 80 directly
# cat /proc/1/net/dev inside container = same as host's network interfaces
```

When to use host networking:
- Network performance-critical applications (game servers, HPC)
- Monitoring/observability tools that need to see host network traffic
- Situations where NAT overhead matters (very low latency requirements)

When NOT to use host networking:
- Most web applications — loses network isolation
- Multi-container apps where containers should be isolated from each other

**The none network driver**:

Completely disables networking for the container.

```bash
docker run --network none \
  -v $(pwd)/input:/input:ro \
  -v $(pwd)/output:/output \
  dataprocessor process /input/data.csv > /output/result.csv

# This container:
# - Reads from bind-mounted input directory
# - Writes to bind-mounted output directory
# - Has zero network access (maximum isolation)
# Perfect for batch processing or security-sensitive operations
```

---

## 9. Docker Compose

### The Complete Guide to Multi-Container Applications

### 9.1 The Problem Compose Exists to Solve

Modern applications rarely consist of a single service. A typical web application might have:
- A web/API server
- A database (PostgreSQL, MySQL)
- A cache (Redis, Memcached)
- A message queue (RabbitMQ, Kafka)
- A search engine (Elasticsearch)
- A background job worker

Without Compose, running all of this requires remembering and executing a large number of commands every time you or a teammate sets up the development environment:

```bash
# What you'd have to run without Compose:
docker network create myapp-net
docker volume create pgdata
docker volume create redisdata

docker run -d \
  --name db \
  --network myapp-net \
  -v pgdata:/var/lib/postgresql/data \
  -e POSTGRES_USER=myapp \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=myapp \
  --health-cmd="pg_isready -U myapp" \
  --health-interval=10s \
  --health-retries=5 \
  --restart unless-stopped \
  postgres:15

docker run -d \
  --name cache \
  --network myapp-net \
  -v redisdata:/data \
  --health-cmd="redis-cli ping" \
  --health-interval=10s \
  --restart unless-stopped \
  redis:7-alpine

docker run -d \
  --name api \
  --network myapp-net \
  -p 3000:3000 \
  -e DATABASE_URL=postgresql://myapp:secret@db:5432/myapp \
  -e REDIS_URL=redis://cache:6379 \
  -e NODE_ENV=production \
  --restart unless-stopped \
  myapp:latest

# And to tear down:
docker stop api cache db
docker rm api cache db
docker network rm myapp-net
# (Volumes preserved by default — must be explicit to remove them)
```

This is error-prone, hard to maintain, and painful for new team members. The answer is a single declarative file.

### 9.2 What Compose Actually Is Under the Hood

This is important to understand because it shapes how you debug Compose issues.

**Docker Compose is not a separate system**. It is a tool that:
1. Reads your `docker-compose.yml`
2. Translates it into individual Docker commands (docker build, docker run, docker network create, docker volume create)
3. Executes those commands on the Docker daemon

Compose doesn't have its own runtime or its own container management. Everything Compose does, Docker already does — Compose just automates the coordination.

```
docker-compose.yml
       ↓
Compose parses YAML
       ↓
Translates to Docker API calls:
  - docker network create
  - docker volume create
  - docker pull or docker build
  - docker run (with all the flags)
       ↓
Docker daemon executes
       ↓
Containers, networks, volumes running
```

**Why this matters**: When Compose does something unexpected, you can always ask "what docker commands is this translating to?" and reason about it from first principles. The Compose abstraction is never magic — it's always just Docker.

### 9.3 Complete Anatomy of docker-compose.yml

Let's go through every field you'll commonly encounter with full explanations.

```yaml
# ─────────────────────────────────────────────
# TOP-LEVEL STRUCTURE
# ─────────────────────────────────────────────
version: "3.9"    # Compose file format version
                  # 3.x supports all modern features
                  # In newer Docker versions, 'version' is optional

services:         # ← Container definitions
  [service-name]:
    [service configuration]

volumes:          # ← Named volume declarations
  [volume-name]:
    [volume options]

networks:         # ← Network declarations
  [network-name]:
    [network options]

# ─────────────────────────────────────────────
# SERVICE CONFIGURATION — every field explained
# ─────────────────────────────────────────────
services:
  myservice:
    
    # ── Image source (use one of: image, build, or both) ──
    image: postgres:15-alpine
    # ← Use a pre-built image from registry
    # OR:
    build:
      context: ./backend         # ← Directory with Dockerfile (build context)
      dockerfile: Dockerfile     # ← Dockerfile name (default: "Dockerfile")
      target: production         # ← Stop at this multi-stage build target
      args:                      # ← Build-time arguments (--build-arg)
        NODE_ENV: production
        VERSION: 1.2.3
      cache_from:                # ← Pull these images to use as cache
        - myapp:latest
    # You can use BOTH image and build — Docker builds and tags with that name:
    image: myregistry.io/myapp:latest
    build:
      context: .

    # ── Container identity ──
    container_name: myapp_api   # ← Fixed container name (avoid if scaling)
                                #   Default: <project>_<service>_<index>

    # ── Port mapping ──
    ports:
      - "3000:3000"             # ← host:container
      - "9229:9229"             # ← Multiple ports
      - "127.0.0.1:5432:5432"  # ← Bind to specific interface

    # ── Environment variables (multiple ways) ──
    environment:
      NODE_ENV: production      # ← Key: value
      PORT: 3000                # ← Numbers become strings automatically
      DEBUG: "true"             # ← Booleans need quotes to stay string
    # OR as list:
    environment:
      - NODE_ENV=production
      - PORT=3000
    # Override specific vars from env file:
    env_file:
      - .env                    # ← Load all vars from file
      - .env.production         # ← Multiple files, later overrides earlier

    # ── Volume mounts ──
    volumes:
      - pgdata:/var/lib/postgresql/data    # ← Named volume
      - ./logs:/app/logs                   # ← Bind mount (relative path to compose file)
      - /app/node_modules                  # ← Anonymous volume
      - type: bind                         # ← Explicit form
        source: ./config
        target: /app/config
        read_only: true                    # ← Read-only mount

    # ── Networking ──
    networks:
      - backend                # ← Connect to these networks
      - frontend

    # ── Startup order and readiness ──
    depends_on:
      db:
        condition: service_healthy    # ← Wait for health check
      cache:
        condition: service_started    # ← Just wait for container to start (not healthy)

    # ── Health check for this service ──
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s          # ← Check every 30 seconds
      timeout: 10s           # ← Consider failed if takes more than 10s
      retries: 3             # ← Mark unhealthy after 3 consecutive failures
      start_period: 40s      # ← Don't count failures during first 40 seconds

    # ── Restart policy ──
    restart: unless-stopped   # ← no | always | on-failure | unless-stopped
    # no: never restart (default)
    # always: restart always, even on clean exit
    # on-failure: restart only on non-zero exit code
    # unless-stopped: restart unless explicitly stopped

    # ── Command override ──
    command: npm run dev       # ← Override CMD from Dockerfile
    command: ["node", "--inspect=0.0.0.0:9229", "server.js"]

    # ── Entrypoint override ──
    entrypoint: /custom/entrypoint.sh

    # ── Resource limits ──
    deploy:
      resources:
        limits:
          cpus: '1.0'          # ← Max 1 CPU core
          memory: 512M         # ← Max 512MB RAM
        reservations:
          cpus: '0.25'         # ← Reserve 0.25 CPU
          memory: 128M

    # ── Logging ──
    logging:
      driver: "json-file"      # ← Log driver: json-file | syslog | none | splunk etc.
      options:
        max-size: "10m"        # ← Rotate when file reaches 10MB
        max-file: "3"          # ← Keep 3 rotated files (30MB total per container)

    # ── Other useful options ──
    tty: true                  # ← Allocate a pseudo-TTY (for interactive containers)
    stdin_open: true           # ← Keep stdin open (for interactive sessions)
    working_dir: /app          # ← Override WORKDIR
    user: "1001:1001"          # ← Override USER
    read_only: true            # ← Read-only filesystem (extra security)
    tmpfs:
      - /tmp                   # ← Mount tmpfs at /tmp (writable even in read_only)
    extra_hosts:
      - "host.docker.internal:host-gateway"  # ← Add /etc/hosts entries
    dns:
      - 8.8.8.8                # ← Custom DNS servers
    privileged: false          # ← Don't run in privileged mode (default: false)
    cap_add:
      - NET_BIND_SERVICE        # ← Add specific Linux capabilities
    cap_drop:
      - ALL                     # ← Drop all capabilities first, add back what's needed
    sysctls:
      - net.core.somaxconn=1024 # ← Set kernel parameters
    ulimits:
      nofile:
        soft: 65536
        hard: 65536
```

**Volume and network top-level declarations**:

```yaml
volumes:
  pgdata:               # ← Simple declaration, uses default driver (local)
  
  redisdata:
    driver: local       # ← Explicit (same as default)
  
  nfsdata:
    driver: nfs         # ← NFS volume (requires NFS driver plugin)
    driver_opts:
      share: "192.168.1.100:/exports/mydata"

  external_vol:
    external: true      # ← Use a pre-existing volume (not managed by Compose)
    name: my-existing-volume   # ← The actual volume name to use

networks:
  frontend:             # ← Simple declaration
  
  backend:
    driver: bridge      # ← Explicit bridge (default)
    
  overlay_net:
    driver: overlay     # ← For Docker Swarm multi-host networking
    
  external_net:
    external: true      # ← Use pre-existing network
    name: myapp-existing-network
    
  custom_subnet:
    driver: bridge
    ipam:               # ← Custom IP address management
      config:
        - subnet: 192.168.100.0/24
          gateway: 192.168.100.1
```

### 9.4 depends_on — Startup Order vs True Readiness

This is the most commonly misunderstood feature in Docker Compose. The confusion stems from what developers expect vs what `depends_on` actually does.

**What developers expect**: "My API should not start until the database is ready to accept connections."

**What `depends_on` (short form) actually does**: "Start the database container before starting the API container."

**The gap**: Docker starts the containers in the specified order, but it does NOT wait for the service inside the container to be ready. Docker starts the postgres container → immediately starts the api container. The postgres container might take 3-5 seconds to initialize before it accepts connections. During those 3-5 seconds, the api might crash trying to connect.

```yaml
# ❌ Short form — only controls start ORDER
services:
  api:
    depends_on:
      - db      # db CONTAINER starts first, but postgres PROCESS might not be ready

  db:
    image: postgres:15
# Result: api often fails to connect on first start
```

**Solution 1: Health check condition (recommended for Compose)**

```yaml
# ✓ Long form with condition — waits for health check to pass
services:
  api:
    depends_on:
      db:
        condition: service_healthy    # ← Wait for HEALTHY status
      cache:
        condition: service_healthy

  db:
    image: postgres:15
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s    # ← Give postgres 30s grace period to initialize

  cache:
    image: redis:7-alpine
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5
```

Now Compose:
1. Starts `db` container
2. Runs the health check command every 10 seconds
3. After the health check passes `retries` times successfully, marks `db` as `healthy`
4. Only THEN starts `api`

**Solution 2: Application-level retry (best for production resilience)**

The health check approach solves startup ordering in Compose, but your application still needs to handle the case where the database becomes unavailable after startup (network blip, database restart, etc.). Build retry logic into your application:

```javascript
// Node.js with pg (PostgreSQL) — robust connection with exponential backoff
const { Pool } = require('pg');

async function createDbPool(maxRetries = 10) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const pool = new Pool({
        connectionString: process.env.DATABASE_URL,
        max: 20,           // Max pool size
        idleTimeoutMillis: 30000,
        connectionTimeoutMillis: 2000,
      });

      // Test the connection
      const client = await pool.connect();
      await client.query('SELECT 1');
      client.release();

      console.log(`Database connected on attempt ${attempt}`);
      return pool;

    } catch (err) {
      console.error(`Database connection attempt ${attempt}/${maxRetries} failed: ${err.message}`);

      if (attempt === maxRetries) {
        throw new Error(`Could not connect to database after ${maxRetries} attempts`);
      }

      // Exponential backoff: 1s, 2s, 4s, 8s, 16s...
      const delay = Math.min(1000 * Math.pow(2, attempt - 1), 30000);
      console.log(`Retrying in ${delay}ms...`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}

const db = await createDbPool();
```

```python
# Python with psycopg2 — connection with retry
import psycopg2
import time
import os

def connect_db(max_retries=10):
    for attempt in range(1, max_retries + 1):
        try:
            conn = psycopg2.connect(os.environ['DATABASE_URL'])
            print(f"Database connected on attempt {attempt}")
            return conn
        except psycopg2.OperationalError as e:
            print(f"Attempt {attempt}/{max_retries} failed: {e}")
            if attempt == max_retries:
                raise
            delay = min(2 ** (attempt - 1), 30)
            print(f"Retrying in {delay}s...")
            time.sleep(delay)

conn = connect_db()
```

### 9.5 Health Checks in Compose — The Full Picture

Health checks serve two distinct purposes in Docker:

1. **Ordering**: Enable `depends_on: condition: service_healthy` to wait for readiness
2. **Monitoring**: Inform Compose, Swarm, Kubernetes, and load balancers about container health

**Health check states in detail**:

```
starting
  ↓ (after start_period passes, checks begin counting)
  ├── Check passes → stays healthy
  └── Check fails → retries count
        ├── Retry count < retries → still healthy (warning)
        └── Retry count = retries → UNHEALTHY
                                      ↓
                               (depends on restart policy)
                               ├── restart: on-failure → container restarts
                               └── no restart → stays unhealthy
```

**Health check for different service types**:

```yaml
# Web API
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
  interval: 30s
  timeout: 10s
  retries: 3
  start_period: 30s

# Postgres
healthcheck:
  test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER:-postgres} -d ${POSTGRES_DB:-postgres}"]
  interval: 10s
  timeout: 5s
  retries: 5
  start_period: 30s

# MySQL
healthcheck:
  test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "root", "--password=${MYSQL_ROOT_PASSWORD}"]
  interval: 10s
  timeout: 5s
  retries: 5
  start_period: 30s

# Redis
healthcheck:
  test: ["CMD", "redis-cli", "-a", "${REDIS_PASSWORD}", "ping"]
  interval: 10s
  timeout: 3s
  retries: 5

# Elasticsearch
healthcheck:
  test: ["CMD-SHELL", "curl -s http://localhost:9200/_cluster/health | grep -v '\"status\":\"red\"'"]
  interval: 20s
  timeout: 10s
  retries: 5
  start_period: 60s   # ES takes time to start

# RabbitMQ
healthcheck:
  test: ["CMD", "rabbitmq-diagnostics", "-q", "ping"]
  interval: 30s
  timeout: 30s
  retries: 3
  start_period: 60s
```

**Checking health status**:

```bash
# See health status in docker ps output
docker ps
# CONTAINER ID  IMAGE      STATUS                  PORTS
# abc123        myapp      Up 2 minutes (healthy)   0.0.0.0:3000->3000/tcp

# Get detailed health check history
docker inspect myapp --format='{{json .State.Health}}' | jq
# Shows last 5 health check results with timestamps and output

# In Compose
docker compose ps
# NAME          IMAGE     COMMAND     STATUS         PORTS
# myapp-db-1    postgres  ...         Up (healthy)   5432/tcp
# myapp-api-1   myapp     ...         Up (healthy)   0.0.0.0:3000->3000/tcp
```

### 9.6 Environment Variables in Compose — All the Ways

Compose gives you multiple ways to inject environment variables. Understanding each method helps you choose the right one for each situation.

**Method 1: Hardcoded in compose file (for non-sensitive defaults)**

```yaml
services:
  api:
    environment:
      NODE_ENV: production
      LOG_LEVEL: info
      MAX_CONNECTIONS: 100
```

**Method 2: Inherited from shell environment (for developer-specific values)**

```yaml
services:
  api:
    environment:
      - DATABASE_URL       # No value — inherits from shell's DATABASE_URL variable
      - AWS_REGION         # Inherits from shell
```

```bash
export DATABASE_URL="postgresql://localhost/myapp"
docker compose up   # api container gets DATABASE_URL from your shell
```

**Method 3: env_file (for grouped configuration)**

```yaml
services:
  api:
    env_file:
      - .env              # Base configuration
      - .env.production   # Environment-specific overrides (later file wins)
```

```bash
# .env
NODE_ENV=production
PORT=3000
LOG_LEVEL=info

# .env.production
DATABASE_URL=postgresql://prod-host:5432/myapp
REDIS_URL=redis://prod-redis:6379
```

**Method 4: Variable substitution in compose file**

```yaml
services:
  api:
    image: myapp:${APP_VERSION:-latest}    # Uses APP_VERSION, defaults to "latest"
    ports:
      - "${HOST_PORT:-3000}:3000"           # Defaults to 3000 if not set
    environment:
      DATABASE_URL: postgresql://${DB_USER}:${DB_PASS}@db:5432/${DB_NAME}
```

```bash
APP_VERSION=1.2.3 HOST_PORT=8080 docker compose up
# → image: myapp:1.2.3, port: 8080:3000
```

Variables are sourced from:
1. Shell environment
2. `.env` file in the same directory as `docker-compose.yml`
3. Variables set in the `environment:` block of the service

**Security best practices for secrets in Compose**:

```yaml
# ❌ Never hardcode secrets in compose files
services:
  db:
    environment:
      POSTGRES_PASSWORD: mysecret    # Don't do this

# ✓ Use env_file that's in .gitignore
services:
  db:
    env_file:
      - .env.secrets    # Add .env.secrets to .gitignore

# ✓ Reference shell environment variables (set by CI/CD)
services:
  db:
    environment:
      - POSTGRES_PASSWORD   # Must be set in shell/CI environment

# ✓ For Swarm/Kubernetes, use Docker Secrets
services:
  db:
    secrets:
      - db_password
secrets:
  db_password:
    file: ./secrets/db_password.txt  # Or from external secret manager
```

### 9.7 Volumes and Networks in Compose

**Volume declaration in Compose**:

Every named volume used in a service's `volumes:` section must be declared at the top-level `volumes:` section:

```yaml
services:
  db:
    volumes:
      - pgdata:/var/lib/postgresql/data   # ← uses volume named "pgdata"

volumes:
  pgdata:     # ← Must be declared here
  # Without this declaration, Compose errors: "Volume 'pgdata' is declared but not in top-level"
```

**Bind mounts don't need declaration** — they reference host paths directly:

```yaml
services:
  app:
    volumes:
      - ./src:/app/src     # ← Bind mount: no declaration needed in volumes:
```

**Volume scope and naming**:

Compose prefixes volume names with the project name:

```bash
docker compose -p myproject up

docker volume ls
# DRIVER    VOLUME NAME
# local     myproject_pgdata    ← Prefixed with project name "myproject"
# local     myproject_redisdata
```

This prevents volumes from different Compose projects from colliding.

**Using pre-existing external volumes**:

```yaml
volumes:
  pgdata:
    external: true          # ← Don't create — use existing
    name: production-db     # ← The actual volume name (no project prefix)
```

```bash
# Create the volume before running Compose
docker volume create production-db
docker compose up
# Uses existing "production-db" volume — Compose won't prefix it
```

**Network declaration details**:

```yaml
networks:
  frontend:
    # Simple declaration — uses bridge driver, Docker assigns subnet

  backend:
    driver: bridge
    driver_opts:
      com.docker.network.bridge.name: myapp-backend   # Custom bridge name on host

  isolated:
    internal: true   # ← No outbound internet access for containers on this network
                     # Containers can talk to each other but can't reach external IPs
```

### 9.8 Overriding Compose Files for Different Environments

Docker Compose supports multiple compose files that are merged together. This lets you have a base configuration and environment-specific overrides.

```
docker-compose.yml          ← Base configuration
docker-compose.override.yml ← Auto-applied when running docker compose up
docker-compose.dev.yml      ← Development overrides
docker-compose.prod.yml     ← Production overrides
docker-compose.test.yml     ← Test environment
```

**How merging works**:

```bash
# Development (applies base + dev override):
docker compose -f docker-compose.yml -f docker-compose.dev.yml up

# Production (applies base + prod override):
docker compose -f docker-compose.yml -f docker-compose.prod.yml up

# Testing:
docker compose -f docker-compose.yml -f docker-compose.test.yml up
```

**Example setup**:

```yaml
# docker-compose.yml — base (shared across all environments)
services:
  api:
    build: ./api
    environment:
      NODE_ENV: production
    networks:
      - backend

  db:
    image: postgres:15-alpine
    volumes:
      - pgdata:/var/lib/postgresql/data
    networks:
      - backend

volumes:
  pgdata:

networks:
  backend:
```

```yaml
# docker-compose.dev.yml — development overrides
services:
  api:
    build:
      target: development     # Build dev stage instead of production
    volumes:
      - ./api:/app            # Live code reload
      - /app/node_modules     # Protect node_modules
    ports:
      - "3000:3000"           # Expose for local browser
      - "9229:9229"           # Expose Node.js debugger
    environment:
      NODE_ENV: development
      DEBUG: "app:*"
    command: npm run dev      # Dev server with hot reload

  db:
    ports:
      - "5432:5432"           # Expose DB for local DB GUI tools
    environment:
      POSTGRES_PASSWORD: devpassword  # Simple local password
```

```yaml
# docker-compose.prod.yml — production overrides
services:
  api:
    image: registry.example.com/myapp:${VERSION}  # Use pre-built image
    restart: unless-stopped
    deploy:
      resources:
        limits:
          memory: 512M
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"

  db:
    # No ports exposed in prod — DB not accessible from host
    restart: unless-stopped
    deploy:
      resources:
        limits:
          memory: 1G
```

### 9.9 Compose Networking — How Services Find Each Other

This is worth understanding deeply because it's the "magic" that makes Compose convenient.

When you run `docker compose up`, Compose:

1. Creates a custom network named `<project>_default` (by default)
2. Attaches ALL services to this network
3. Each service is reachable by its service name via Docker's DNS

```yaml
services:
  api:
    build: .
    environment:
      DATABASE_URL: postgresql://postgres:secret@db:5432/myapp
      #                                          ↑
      #                            This "db" is the service name
      #                            Docker DNS resolves it automatically

  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
```

Because Compose puts both services on the same custom network (`<project>_default`), the `api` container can resolve `db` to the `db` container's IP via Docker DNS.

**Customizing the default network**:

```yaml
services:
  api:
    networks:
      - default       # ← Explicitly on the auto-created default network
  db:
    networks:
      - default

networks:
  default:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16    # ← Custom subnet for this project
```

---

## 10. Common Mistakes — What Goes Wrong and Exactly Why

### Mistake 1: Running Containers as Root

**What it is**: By default, the process inside a container runs as UID 0 (root).

**Why it's a problem**: If an attacker exploits a vulnerability in your application, they have root access inside the container. In misconfigured setups, or through container escape vulnerabilities, root inside can mean root on the host. Files created by the container in bind-mounted volumes are owned by root on the host — difficult to manage.

**The fix**:

```dockerfile
# For a Node.js app:
FROM node:20-alpine
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodeuser -u 1001 -G nodejs
WORKDIR /app
COPY --chown=nodeuser:nodejs . .
USER nodeuser     # ← All subsequent operations and the container process run as this user
CMD ["node", "server.js"]
```

### Mistake 2: Using `latest` or Floating Tags in Production

**What it is**: Using `FROM node:latest` or `image: postgres:latest` in production Dockerfiles or compose files.

**Why it's a problem**: The `latest` tag is just a pointer that gets updated whenever a new version is published. Today `node:latest` is Node 20. Six months from now it might be Node 22 with breaking API changes. Your build environment silently becomes incompatible.

**The fix**:
```dockerfile
# ❌
FROM node:latest
FROM python:3

# ✅
FROM node:20.11.0-alpine3.19
FROM python:3.11.7-slim-bookworm
```

### Mistake 3: Storing Secrets in Dockerfile ENV or Image Layers

**What it is**: Putting API keys, database passwords, or tokens in `ENV` instructions.

**Why it's a problem**: The `ENV` value is stored in the image layers. Anyone who can pull the image can read it with `docker history` or `docker inspect`. It appears in your git history if your Dockerfile is committed. It propagates to every container and every environment.

**The fix**:
```dockerfile
# ❌ Permanent exposure
ENV API_KEY=abc123secretkey
ENV DB_PASSWORD=supersecret

# ✅ Inject at runtime
docker run -e API_KEY="$API_KEY" -e DB_PASSWORD="$DB_PASSWORD" myapp
# Or via env_file (with .env in .gitignore):
docker run --env-file .env.production myapp
```

### Mistake 4: Not Having or Not Maintaining .dockerignore

**What it is**: Running `docker build .` with `COPY . .` in the Dockerfile and no `.dockerignore` file.

**Why it's a problem**:
- `node_modules/` (200MB) gets copied into the image — both bloating the image and potentially using the wrong platform's native modules
- `.git/` (potentially hundreds of MB) ends up in the image
- `.env` files with secrets end up in the image
- Large test data or fixtures end up in the image

**The fix**:
```
# .dockerignore
node_modules/
.git/
.env
.env.*
data/
test-fixtures/
*.log
dist/
.DS_Store
coverage/
```

### Mistake 5: Incorrect Layer Ordering (Slow Builds)

**What it is**: Copying source code before installing dependencies.

**Why it's a problem**: The `npm install` / `pip install` layer is invalidated on every code change because `COPY . .` comes before it. Every single build re-runs package installation, which can take minutes.

**The fix**: Dependencies first, code second.

```dockerfile
# ❌ npm install invalidated on EVERY code change
COPY . .
RUN npm install

# ✅ npm install only invalidated when package.json changes
COPY package.json package-lock.json ./
RUN npm install
COPY . .
```

### Mistake 6: Short-Form depends_on Without Health Checks

**What it is**: Using `depends_on: [db]` and expecting the API to wait for the database to be ready.

**Why it's a problem**: Short-form `depends_on` only controls start order — it doesn't wait for readiness. The api container starts immediately after the db container starts (not after PostgreSQL inside is ready). First connection attempt fails.

**The fix**: Health checks + long-form depends_on.

```yaml
# ❌ Doesn't wait for postgres to be ready
api:
  depends_on:
    - db

# ✅ Waits for health check to pass
api:
  depends_on:
    db:
      condition: service_healthy
db:
  healthcheck:
    test: ["CMD-SHELL", "pg_isready -U postgres"]
    interval: 10s
    retries: 5
```

### Mistake 7: Expecting `EXPOSE` to Open Ports

**What it is**: Thinking that `EXPOSE 3000` in the Dockerfile makes port 3000 accessible.

**Why it's a problem**: `EXPOSE` is documentation only. Without `-p 3000:3000`, the port is not accessible from the host.

**The fix**:
```bash
# You always need -p to publish a port
docker run -p 3000:3000 myapp
# In Compose:
ports:
  - "3000:3000"
```

### Mistake 8: Shell Form in CMD/ENTRYPOINT (Signal Handling)

**What it is**: Using `CMD npm start` (shell form) instead of `CMD ["npm", "start"]` (exec form).

**Why it's a problem**: Shell form makes `/bin/sh` PID 1, not your process. `docker stop` sends SIGTERM to PID 1 (the shell). The shell may or may not forward it to your process. After 10 seconds, Docker sends SIGKILL — forceful termination. Your application doesn't get a chance to shut down gracefully (finish requests, close DB connections, flush logs).

**The fix**:
```dockerfile
# ❌ Shell form — signal handling broken
CMD npm start
ENTRYPOINT python app.py

# ✅ Exec form — SIGTERM goes directly to your process
CMD ["npm", "start"]
ENTRYPOINT ["python", "app.py"]
```

### Mistake 9: Storing Database Data in Writable Layer

**What it is**: Running a database container without a volume, letting it write to the writable layer.

**Why it's a problem**: `docker compose down` removes the container and its writable layer. All database data is permanently lost.

**The fix**:
```yaml
services:
  db:
    image: postgres:15
    volumes:
      - pgdata:/var/lib/postgresql/data   # ← Named volume required

volumes:
  pgdata:
```

### Mistake 10: Using the Default Bridge Network for Multi-Container Apps

**What it is**: Relying on Docker's default bridge network to connect containers, hoping to use service names.

**Why it's a problem**: The default bridge network doesn't have DNS. Containers can only communicate via IP addresses, which change on restart.

**The fix**: Always use custom networks (or Compose, which creates one automatically).

```bash
# ❌ Default bridge — no DNS
docker run --name db postgres
docker run -e DB_HOST=172.17.0.2 myapp   # Hardcoded IP — will break

# ✅ Custom network — DNS works
docker network create mynet
docker run --network mynet --name db postgres
docker run --network mynet -e DB_HOST=db myapp   # "db" resolves via DNS
```

---

## 11. Troubleshooting and Debugging — The Complete Playbook

### Problem: Container Exits Immediately After Starting

```bash
# Step 1: Check the exit code and last logs
docker ps -a
# CONTAINER ID  IMAGE   STATUS                   
# abc123        myapp   Exited (1) 3 seconds ago  ← Exit code 1 = application error

docker logs abc123
# → Shows the last output before exit — usually the error message

# Step 2: Run interactively to debug
docker run -it --entrypoint /bin/sh myapp
# Now you're inside the container with a shell — poke around

# Step 3: Check if the CMD/ENTRYPOINT script exists and is executable
docker run -it myapp ls -la /app/server.js
docker run -it myapp which node

# Common exit codes:
# 0: Clean exit (e.g., CMD ran to completion — OK for one-shot containers)
# 1: Generic application error (check logs)
# 127: Command not found (your CMD references a binary that doesn't exist in the image)
# 137: SIGKILL — container was killed (OOM, or docker kill)
# 139: Segmentation fault (rare in interpreted languages)
# 143: SIGTERM received — graceful shutdown initiated
```

### Problem: Port Is Not Accessible

```bash
# Step 1: Verify the container is actually running and has the port mapped
docker ps
# CONTAINER ID  IMAGE   PORTS
# abc123        myapp   ← No ports listed! Forgot -p flag!
# abc123        myapp   0.0.0.0:3000->3000/tcp  ← Good

# Step 2: Verify the app is actually listening inside the container
docker exec myapp netstat -tlnp
# tcp    0.0.0.0:3000    LISTEN  ← App is listening on port 3000
# OR — no output → app crashed or isn't listening yet

docker exec myapp ss -tlnp
# State    Recv-Q Send-Q Local Address:Port
# LISTEN   0      128    *:3000              ← Listening on all interfaces

# Step 3: Test the port from INSIDE the container
docker exec myapp curl -s http://localhost:3000
docker exec myapp wget -qO- http://localhost:3000
# If this works but host can't reach it → port mapping issue
# If this fails → app isn't actually responding

# Step 4: Check if a host port conflict exists
lsof -i :3000          # macOS/Linux: see what's using port 3000
ss -tlnp | grep 3000   # Linux
netstat -ano | findstr :3000  # Windows

# Step 5: Check Docker's port forwarding
docker port myapp
# → 3000/tcp -> 0.0.0.0:3000   ← Port mapping is set up

# Step 6: Try explicitly with localhost
curl http://127.0.0.1:3000
curl http://localhost:3000

# Note on Mac/Windows: Container IP (172.17.0.x) is not directly accessible from host
# Must use localhost + published port
```

### Problem: Services Can't Communicate With Each Other

```bash
# Step 1: Verify both containers are on the same network
docker inspect api | grep -A 20 '"Networks"'
docker inspect db | grep -A 20 '"Networks"'
# → Both should show the same network name

# Step 2: Test name resolution from inside the container
docker exec api ping db
docker exec api nslookup db
docker exec api cat /etc/resolv.conf
# Should show: nameserver 127.0.0.11 (Docker's DNS — only on custom networks)

# Step 3: Test actual connection
docker exec api curl http://db:5432
docker exec api pg_isready -h db -p 5432

# Step 4: Check if using default bridge (no DNS) vs custom network
docker network ls
docker network inspect bridge | grep -A 5 Containers
docker network inspect myapp_default | grep -A 5 Containers

# If on default bridge and can't use names:
# Option A: Use container IPs (fragile)
docker inspect db | grep IPAddress
# → 172.17.0.2
docker exec api curl http://172.17.0.2:5432  # Temporary fix

# Option B: Create custom network (proper fix)
docker network create mynet
docker network connect mynet api
docker network connect mynet db
docker exec api curl http://db:5432  # Now DNS works
```

### Problem: Data Is Lost After docker compose down

```bash
# Confirm: docker compose down removes containers but preserves named volumes
docker compose down       # Removes containers, keeps volumes
docker compose down -v    # Removes containers AND volumes ← This is why data is lost!

# Check if you accidentally used -v flag
# Check your compose file for volume declarations
docker volume ls
# If you don't see your volume here, it was never created as a named volume

# Verify the volume is declared and used correctly:
# In docker-compose.yml:
# services:
#   db:
#     volumes:
#       - pgdata:/var/lib/postgresql/data   ← correct named volume syntax
#
# volumes:
#   pgdata:   ← must be declared here

# Check if using bind mount (different behavior)
docker inspect db | grep -A 10 '"Mounts"'
# "Type": "bind"    → bind mount to host path
# "Type": "volume"  → named Docker volume
```

### Problem: Build Is Slow — Every Build Reinstalls Packages

```bash
# Step 1: Verify caching is happening (or not)
docker build --progress=plain . 2>&1 | grep -E "CACHED|RUN|COPY"
# Lines with "CACHED" → layer reused from cache
# Lines with "RUN" or "COPY" without "CACHED" → rebuilt

# Step 2: Check instruction order in Dockerfile
# If you see "CACHED" on COPY package.json but not on RUN npm install,
# something between those two is invalidating the cache

# Step 3: Common cause — COPY . . before dependencies
cat Dockerfile | grep -n "COPY\|RUN npm"
# Wrong order:
# 3: COPY . .            ← Copies code first
# 4: RUN npm install     ← npm install after code = invalidated every time

# Correct order:
# 3: COPY package.json ./     ← Dependencies first
# 4: RUN npm install          ← Only invalidated when package.json changes
# 5: COPY . .                 ← Code last

# Step 4: Verify .dockerignore isn't missing
ls -la | grep dockerignore
# If missing, large directories (node_modules) might be included in context

# Step 5: Check if base image is stale
docker pull node:20-alpine   # Force update of base image
docker build --no-cache .    # Force full rebuild to verify it works
```

### Problem: Out of Disk Space

```bash
# See Docker's total disk usage
docker system df
# TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
# Images          12        4         8.5GB     5.2GB (61%)
# Containers      8         3         2.1GB     1.8GB (85%)
# Local Volumes   5         3         4.2GB     1.1GB (26%)
# Build Cache     0         0         0B        0B

# See detailed breakdown
docker system df -v

# Clean up everything unused (CAREFUL — removes unused images, volumes, etc.)
docker system prune          # removes stopped containers, unused networks, dangling images
docker system prune -a       # also removes unused images (not just dangling)
docker system prune --volumes # also removes unused volumes ← DELETES DATA

# More targeted cleanup:
docker image prune -a        # Remove all unused images
docker container prune       # Remove all stopped containers
docker volume prune          # Remove unused volumes ← DELETES DATA
docker network prune         # Remove unused networks
docker builder prune         # Remove build cache

# See which images are largest
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}" | sort -k3 -h
```

### Problem: Environment Variables Not Available in Container

```bash
# Step 1: Check what env vars the container actually has
docker exec myapp env | sort
docker exec myapp printenv NODE_ENV

# Step 2: Verify the env var is set at startup
docker inspect myapp | grep -A 20 '"Env"'
# Shows all env vars set for the container

# Step 3: Check if ENV in Dockerfile is set correctly
docker history myapp | grep ENV

# Step 4: Check if env var was passed at run time
docker run -e NODE_ENV=production myapp    # Inline
docker run --env-file .env myapp           # From file

# Step 5: In Compose, check environment section
docker compose config   # Shows the resolved Compose config with all substitutions
# ← This shows if variable substitution worked correctly

# Common issue: missing variable in .env file
# docker compose config will show ${VARIABLE:-defaultvalue} if the var is unset
```

### Problem: "Permission Denied" Errors

```bash
# Step 1: Check what user the process runs as
docker exec myapp id
docker exec myapp whoami

# Step 2: Check file ownership inside container
docker exec myapp ls -la /app

# Step 3: For bind mounts, check host file ownership
ls -la ./myapp-code
# drwxr-xr-x  user1  user1  → owned by user1 UID

# Step 4: The mismatch problem
# Container runs as UID 1001 (appuser)
# Host files owned by UID 1000 (your user)
# → Container can't write to bind-mounted directory

# Fix option A: Match UIDs
# Build with the same UID as your host user:
docker build --build-arg UID=$(id -u) --build-arg GID=$(id -g) .
# In Dockerfile: RUN adduser -u $UID -g $GID appuser

# Fix option B: Use named volumes (Docker manages ownership)
volumes:
  appdata:    # Named volume — Docker handles permissions

# Fix option C: chmod the bind-mounted directory
chmod -R 777 ./writable-dir   # Last resort — overly permissive
```

---

## 12. Best Practices — The Principles Behind the Rules

### Dockerfile Best Practices

**Principle 1: One concern per container**

A container should have one primary process. Don't run a web server and a database in the same container. Separation enables independent scaling, updating, and fault isolation.

**Principle 2: Build from a known, pinned base**

```dockerfile
# ❌ Floating — builds differently over time
FROM node:latest
FROM python:3

# ✅ Pinned — reproducible forever
FROM node:20.11.0-alpine3.19
FROM python:3.11.7-slim-bookworm
```

**Principle 3: Layer order = frequency of change**

```dockerfile
FROM node:20-alpine         # Rarely changes
RUN apk add --no-cache curl # Rarely changes
WORKDIR /app
COPY package*.json ./       # Changes when dependencies change
RUN npm ci                  # Expensive, only runs when deps change
COPY . .                    # Changes frequently (your code)
CMD ["node", "server.js"]
```

**Principle 4: Always clean up in the same RUN instruction**

```dockerfile
# The cleanup must be in the SAME RUN to actually reduce image size:
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    rm -rf /var/lib/apt/lists/*     # Same instruction = no orphaned cache bytes
```

**Principle 5: Never run as root**

```dockerfile
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
COPY --chown=appuser:appgroup . .
USER appuser
```

**Principle 6: Use exec form for CMD and ENTRYPOINT**

```dockerfile
# For correct signal handling:
CMD ["node", "server.js"]        # ✅ Exec form
ENTRYPOINT ["python", "app.py"]  # ✅ Exec form
```

**Principle 7: Use multi-stage builds for production images**

Build tools don't belong in production. Use multi-stage to keep production images lean.

**Principle 8: Always include .dockerignore**

Exclude everything that shouldn't be in the image: node_modules, .git, .env files, large test data.

**Principle 9: Include a HEALTHCHECK**

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:3000/health || exit 1
```

### Compose Best Practices

**Principle 1: Pin image versions**

```yaml
# ❌
image: postgres:latest

# ✅
image: postgres:15.4-alpine3.18
```

**Principle 2: Use named volumes for all stateful data**

```yaml
services:
  db:
    volumes:
      - pgdata:/var/lib/postgresql/data   # Named volume — persists across compose down

volumes:
  pgdata:
```

**Principle 3: Use custom networks for DNS**

Compose creates a default network automatically — use it. If you need isolation, declare explicit networks.

**Principle 4: Use health checks with depends_on**

```yaml
api:
  depends_on:
    db:
      condition: service_healthy
db:
  healthcheck:
    test: ["CMD-SHELL", "pg_isready -U postgres"]
```

**Principle 5: Keep secrets out of compose files**

```yaml
# Use env_file (file is in .gitignore):
env_file:
  - .env.secrets

# Or inherit from shell/CI:
environment:
  - DB_PASSWORD  # No value — inherits from environment
```

**Principle 6: Set resource limits**

```yaml
deploy:
  resources:
    limits:
      memory: 512M
      cpus: '1.0'
```

**Principle 7: Configure log rotation**

```yaml
logging:
  driver: json-file
  options:
    max-size: "10m"
    max-file: "3"
```

**Principle 8: Use restart policies**

```yaml
restart: unless-stopped   # Restarts on crash, stops on docker compose stop
```

### Security Best Practices

```bash
# Scan images for known vulnerabilities
docker scout cves myapp:latest
trivy image myapp:latest
grype myapp:latest

# Run with read-only filesystem where possible
docker run --read-only \
  -v /tmp:/tmp \           # writable tmp
  -v logs:/app/logs \      # writable logs volume
  myapp

# Drop all capabilities, add back only what's needed
docker run \
  --cap-drop ALL \
  --cap-add NET_BIND_SERVICE \
  myapp

# Don't mount the Docker socket (gives container control over Docker daemon)
# ❌ docker run -v /var/run/docker.sock:/var/run/docker.sock myapp

# Use secrets management for sensitive values
# Production: AWS Secrets Manager, HashiCorp Vault, Kubernetes secrets
# Development: .env files in .gitignore
```

---

## 13. Complete Real-World Examples

### Example 1: Full-Stack Node.js + PostgreSQL + Redis Application

**Directory structure**:
```
myapp/
├── backend/
│   ├── src/
│   │   ├── server.js
│   │   ├── routes/
│   │   └── models/
│   ├── package.json
│   ├── package-lock.json
│   └── Dockerfile
├── frontend/
│   ├── src/
│   ├── package.json
│   └── Dockerfile
├── nginx/
│   └── nginx.conf
├── .env.example
├── .dockerignore
└── docker-compose.yml
```

**backend/Dockerfile**:
```dockerfile
# ════════════════════════════════════════════
# Stage 1: Dependencies
# ════════════════════════════════════════════
FROM node:20-alpine AS deps
WORKDIR /app

# Install only production dependencies
COPY package.json package-lock.json ./
RUN npm ci --only=production && \
    cp -r node_modules prod_node_modules

# Install all dependencies (including dev) for building
RUN npm ci

# ════════════════════════════════════════════
# Stage 2: Builder
# ════════════════════════════════════════════
FROM node:20-alpine AS builder
WORKDIR /app

COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Run tests and linting (fail build if tests fail)
RUN npm test
RUN npm run lint

# ════════════════════════════════════════════
# Stage 3: Production
# ════════════════════════════════════════════
FROM node:20-alpine AS production

# Security: install dumb-init for proper signal handling
RUN apk add --no-cache \
    dumb-init \
    curl \
    && addgroup -g 1001 -S nodejs \
    && adduser -S nodeuser -u 1001 -G nodejs

WORKDIR /app

# Copy only production node_modules
COPY --from=deps --chown=nodeuser:nodejs /app/prod_node_modules ./node_modules

# Copy application source
COPY --chown=nodeuser:nodejs src ./src
COPY --chown=nodeuser:nodejs package.json ./

USER nodeuser

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s --retries=3 --start-period=10s \
  CMD curl -f http://localhost:3000/health || exit 1

# dumb-init handles PID 1 responsibilities and forwards signals correctly
ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "src/server.js"]
```

**frontend/Dockerfile**:
```dockerfile
FROM node:20-alpine AS builder
WORKDIR /build

COPY package.json package-lock.json ./
RUN npm ci

COPY . .
RUN npm run build   # Output to /build/dist

FROM nginx:1.25-alpine AS production
COPY --from=builder /build/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget -qO- http://localhost/ || exit 1
```

**nginx/nginx.conf**:
```nginx
server {
    listen 80;

    # Serve static frontend files
    location / {
        root /usr/share/nginx/html;
        index index.html;
        try_files $uri $uri/ /index.html;  # SPA routing
    }

    # Proxy API requests to backend service
    location /api/ {
        proxy_pass http://api:3000;         # "api" = service name, Docker DNS resolves it
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

**docker-compose.yml** (production):
```yaml
version: "3.9"

services:
  # ─── Reverse Proxy (public-facing) ───
  nginx:
    build:
      context: ./frontend
      target: production
    ports:
      - "80:80"
    depends_on:
      api:
        condition: service_healthy
    networks:
      - frontend
    restart: unless-stopped
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"

  # ─── API Server ───
  api:
    build:
      context: ./backend
      target: production
    environment:
      NODE_ENV: production
      PORT: 3000
      DATABASE_URL: postgresql://appuser:${DB_PASSWORD}@db:5432/myapp
      REDIS_URL: redis://:${REDIS_PASSWORD}@cache:6379
    depends_on:
      db:
        condition: service_healthy
      cache:
        condition: service_healthy
    networks:
      - frontend
      - backend
    restart: unless-stopped
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: '1.0'
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 15s
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"

  # ─── Database ───
  db:
    image: postgres:15.4-alpine3.18
    environment:
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: myapp
      PGDATA: /var/lib/postgresql/data/pgdata
    volumes:
      - pgdata:/var/lib/postgresql/data
      - ./db/init:/docker-entrypoint-initdb.d:ro   # Schema init scripts
    networks:
      - backend
    restart: unless-stopped
    deploy:
      resources:
        limits:
          memory: 1G
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U appuser -d myapp"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"

  # ─── Cache ───
  cache:
    image: redis:7.2-alpine
    command: redis-server --appendonly yes --requirepass ${REDIS_PASSWORD}
    volumes:
      - redisdata:/data
    networks:
      - backend
    restart: unless-stopped
    deploy:
      resources:
        limits:
          memory: 256M
    healthcheck:
      test: ["CMD", "redis-cli", "-a", "${REDIS_PASSWORD}", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5
    logging:
      driver: json-file
      options:
        max-size: "5m"
        max-file: "3"

volumes:
  pgdata:
  redisdata:

networks:
  frontend:     # nginx + api
  backend:      # api + db + cache
```

**docker-compose.dev.yml** (development overrides):
```yaml
version: "3.9"

services:
  nginx:
    build:
      context: ./frontend
      target: builder     # Build stage with dev server
    command: npm run dev  # Dev server with hot reload
    volumes:
      - ./frontend/src:/build/src       # Live code
    ports:
      - "5173:5173"                     # Vite dev server port (instead of 80)

  api:
    build:
      context: ./backend
      target: deps        # Stop at deps stage (all deps installed)
    command: npm run dev  # Nodemon hot reload
    volumes:
      - ./backend/src:/app/src          # Live code editing
    environment:
      NODE_ENV: development
      DEBUG: "app:*"
    ports:
      - "9229:9229"                     # Node.js debugger

  db:
    ports:
      - "127.0.0.1:5432:5432"          # Expose for local DB GUI tools (localhost only)
    environment:
      POSTGRES_PASSWORD: devpassword    # Simple password for local dev

  cache:
    command: redis-server               # No password in dev
    ports:
      - "127.0.0.1:6379:6379"          # Expose for local Redis inspection
```

**Running the application**:
```bash
# Development
cp .env.example .env
# Edit .env with your values
docker compose -f docker-compose.yml -f docker-compose.dev.yml up

# Production
docker compose up -d

# View logs
docker compose logs -f api

# Scale API instances
docker compose up -d --scale api=3

# Deploy new version
docker compose build api
docker compose up -d --no-deps api   # Restart only api, not db/cache

# Tear down (keep data)
docker compose down

# Tear down (destroy everything including data)
docker compose down -v
```

---

## 14. Quick Reference Cheat Sheet

### Docker Image Commands

```bash
# Build
docker build -t name:tag .                     # Build from Dockerfile in current dir
docker build -t name:tag -f MyDockerfile .     # Use specific Dockerfile
docker build --no-cache -t name:tag .          # Rebuild without cache
docker build --build-arg KEY=value -t name .   # Pass build argument
docker build --target stage-name -t name .     # Build specific multi-stage target
docker build --progress=plain .                # Verbose output showing each step

# View
docker images                                  # List all images
docker images -a                               # Include intermediate images
docker image ls --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
docker history image:tag                       # Show layer history
docker history --no-trunc image:tag            # Full layer commands
docker inspect image:tag                       # Full image metadata

# Registry
docker pull image:tag                          # Download from registry
docker push image:tag                          # Upload to registry
docker tag source:tag target:newtag            # Create new tag
docker login registry.example.com             # Authenticate with registry

# Cleanup
docker rmi image:tag                           # Remove image
docker rmi $(docker images -q)                 # Remove all images
docker image prune                             # Remove dangling images
docker image prune -a                          # Remove all unused images
```

### Docker Container Commands

```bash
# Creating and running
docker run image:tag                           # Run (foreground, exits when cmd exits)
docker run -d image:tag                        # Run in background (detached)
docker run -it image:tag /bin/sh               # Run with interactive terminal
docker run --name myapp image:tag              # Give container a name
docker run --rm image:tag                      # Auto-remove when exits
docker run -p 8080:80 image:tag                # Publish port (host:container)
docker run -p 127.0.0.1:3000:3000 image:tag   # Bind to localhost only
docker run -e KEY=value image:tag              # Set environment variable
docker run --env-file .env image:tag           # Load env from file
docker run -v volume:/path image:tag           # Mount named volume
docker run -v /host/path:/container/path image # Bind mount
docker run --network mynet image:tag           # Connect to network
docker run --memory="512m" --cpus="1.5" image  # Resource limits
docker run --user 1001:1001 image:tag          # Run as specific user
docker run --read-only image:tag               # Read-only filesystem

# Lifecycle
docker ps                                      # List running containers
docker ps -a                                   # List all containers (incl. stopped)
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
docker start container                         # Start stopped container
docker stop container                          # Graceful stop (SIGTERM + wait)
docker kill container                          # Immediate stop (SIGKILL)
docker restart container                       # Stop and start
docker pause container                         # Freeze (SIGSTOP)
docker unpause container                       # Unfreeze
docker rm container                            # Remove stopped container
docker rm -f container                         # Force remove running container
docker rm $(docker ps -aq)                     # Remove all stopped containers

# Interaction
docker exec -it container /bin/sh              # Shell into running container
docker exec -it container /bin/bash            # Bash (if available)
docker exec container command arg              # Run command in container
docker exec -u root container command          # Run as specific user
docker attach container                        # Attach to container's process
docker cp container:/path/file ./local         # Copy from container to host
docker cp ./local container:/path/file         # Copy from host to container

# Inspection and debugging
docker logs container                          # View all logs
docker logs -f container                       # Follow logs (tail -f equivalent)
docker logs --tail 100 container               # Last 100 lines
docker logs --since 10m container              # Logs from last 10 minutes
docker inspect container                       # Full JSON metadata
docker inspect --format='{{.State.Health}}' c  # Specific field
docker stats                                   # Live resource usage (all containers)
docker stats container                         # Live resource usage (one container)
docker stats --no-stream                       # Single snapshot
docker top container                           # Running processes in container
docker port container                          # Show port mappings
docker diff container                          # Show filesystem changes in container
```

### Docker Volume Commands

```bash
docker volume create name                      # Create named volume
docker volume ls                               # List all volumes
docker volume ls -f dangling=true              # List unused volumes
docker volume inspect name                     # Volume details and host path
docker volume rm name                          # Remove volume (must be unused)
docker volume prune                            # Remove all unused volumes ⚠️
```

### Docker Network Commands

```bash
docker network create mynet                    # Create bridge network
docker network create --driver overlay mynet   # Create overlay network (Swarm)
docker network ls                              # List networks
docker network inspect mynet                   # Network details + connected containers
docker network rm mynet                        # Remove network
docker network prune                           # Remove unused networks
docker network connect mynet container         # Connect container to network
docker network disconnect mynet container      # Disconnect container from network
```

### Docker Compose Commands

```bash
# Lifecycle
docker compose up                              # Start all services (foreground)
docker compose up -d                           # Start all services (detached)
docker compose up --build                      # Rebuild images and start
docker compose up --build --force-recreate     # Rebuild + recreate all containers
docker compose up -d --scale service=3         # Run 3 instances of service
docker compose down                            # Stop and remove containers + networks
docker compose down -v                         # Also remove volumes ⚠️ (DESTROYS DATA)
docker compose down --rmi all                  # Also remove images
docker compose start                           # Start existing containers
docker compose stop                            # Stop containers (don't remove)
docker compose restart                         # Restart all services
docker compose restart service                 # Restart specific service

# Building
docker compose build                           # Build all images
docker compose build service                   # Build specific service
docker compose pull                            # Pull latest images

# Inspection
docker compose ps                              # Status of all services
docker compose logs                            # All service logs
docker compose logs -f service                 # Follow specific service
docker compose logs --tail 50                  # Last 50 lines from all services
docker compose top                             # Running processes
docker compose config                          # Resolved Compose config (debug)
docker compose config --services               # List service names

# Interaction
docker compose exec service command            # Run command in service container
docker compose exec -it service /bin/sh        # Shell into service container
docker compose run --rm service command        # Run one-off command in new container

# Cleanup
docker compose down --rmi all -v               # Full cleanup (containers + images + volumes)
```

### Dockerfile Quick Reference

```dockerfile
FROM image:tag                    # Base image (MUST be first)
FROM image:tag AS name            # Multi-stage: named stage

WORKDIR /absolute/path            # Set working directory (creates if missing)

COPY src dest                     # Copy from build context → image (preferred)
COPY --chown=user:group src dest  # Copy with ownership
ADD src dest                      # Like COPY + URL support + auto-tar extract (avoid)

RUN command                       # Run during BUILD, creates layer
RUN cmd1 && cmd2 && cleanup       # Chain in one RUN for single layer
RUN ["executable", "arg1"]        # Exec form (no shell)

ENV KEY=value                     # Env var available at build AND runtime
ENV KEY1=val1 KEY2=val2           # Multiple vars

ARG NAME=default                  # Build-time variable only (not in container)

EXPOSE 8080                       # Documentation only — doesn't open port!
EXPOSE 8080/tcp                   # With protocol
EXPOSE 53/udp

VOLUME /data                      # Mark mount point (creates anonymous vol if not mounted)

USER username                     # Switch user for subsequent instructions + runtime
USER 1001:1001                    # By UID:GID

LABEL key=value                   # Image metadata
LABEL key1=val1 key2=val2         # Multiple labels

HEALTHCHECK --interval=30s CMD command   # Health check command
HEALTHCHECK NONE                         # Disable inherited health check

ENTRYPOINT ["executable"]         # Fixed executable (use exec form)
ENTRYPOINT ["sh", "-c", "command"] # Via shell

CMD ["arg1", "arg2"]              # Default arguments (overridable with docker run)
CMD ["node", "server.js"]         # Default command if ENTRYPOINT not set (use exec form)
```

### Port Mapping Syntax

```bash
-p 3000:3000              # All interfaces, host:container
-p 8080:80                # Different host and container ports
-p 127.0.0.1:3000:3000   # Bind to localhost only (security)
-p 3000-3010:3000-3010   # Range of ports
-P                        # Auto-assign host ports for all EXPOSE'd ports
```

### Volume Mount Syntax

```bash
-v name:/container/path                # Named volume
-v /absolute/host/path:/container/path # Bind mount (absolute path)
-v ./relative/path:/container/path    # Bind mount (relative path, Compose only)
-v /container/path                    # Anonymous volume
-v name:/path:ro                      # Read-only mount
--mount type=volume,source=name,target=/path           # Explicit volume
--mount type=bind,source=/host/path,target=/container  # Explicit bind
--mount type=tmpfs,target=/tmp                         # tmpfs in memory
```

### Common docker run Flags

```bash
-d                         # Detached (background)
-i                         # Interactive (stdin open)
-t                         # Allocate pseudo-TTY
-it                        # Interactive terminal
--name string              # Container name
--rm                       # Auto-remove on exit
-p, --publish host:cont    # Port mapping
-e, --env KEY=val          # Environment variable
--env-file path            # Load env from file
-v, --volume              # Volume mount
--network net              # Connect to network
--user uid:gid             # Run as user
--memory 512m              # Memory limit
--cpus 1.5                 # CPU limit
--restart policy           # Restart policy
--entrypoint cmd           # Override ENTRYPOINT
--read-only                # Read-only filesystem
--cap-add cap              # Add Linux capability
--cap-drop cap             # Remove Linux capability
--privileged               # Full host access (avoid!)
-w, --workdir /path        # Override working directory
--health-cmd cmd           # Custom health check command
--health-interval 30s      # Health check interval
--label key=value          # Container label
```

### Cleanup Reference

```bash
# Safe cleanups (won't affect running things)
docker container prune          # Remove stopped containers
docker image prune              # Remove dangling images
docker network prune            # Remove unused networks
docker system prune             # Remove stopped containers + dangling images + unused networks

# Potentially data-destroying cleanups
docker volume prune             # ⚠️ Removes unused volumes (permanent data loss!)
docker system prune -a          # ⚠️ Also removes unused (not just dangling) images
docker system prune -a --volumes # ⚠️ Full cleanup including unused volumes

# See what would be removed before doing it
docker system df -v             # Show all Docker resource usage with details
```

---

*This guide has covered Docker from the ground up — not as a set of commands to memorize, but as a system to understand. The mental model is everything: containers are isolated processes, images are immutable layered blueprints, volumes solve the persistence problem, networks solve service discovery, and Compose ties it all together into a manageable whole. When you encounter new Docker concepts — multi-stage builds, health checks, Kubernetes orchestration — they all build directly on these foundations. The confusion you felt before was never about Docker being hard. It was about learning the surface before learning the structure. Now you have the structure.*