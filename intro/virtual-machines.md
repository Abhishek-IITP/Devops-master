# Virtual Machines (VMs)

# What is a Virtual Machine?

A **Virtual Machine (VM)** is a software-based computer that behaves like a physical computer. It has its own operating system, CPU, memory (RAM), storage, and network, even though it runs on top of another physical machine.

In simple words,

> A Virtual Machine allows us to run multiple independent operating systems on a single physical computer.

Each Virtual Machine works independently and is isolated from other Virtual Machines running on the same host machine.

---

# Why Do We Need Virtual Machines?

Before Virtual Machines, organizations followed the **"One Server = One Application"** approach.

Example:

```text
Physical Server 1 → Banking Application

Physical Server 2 → HR Application

Physical Server 3 → Email Server

Physical Server 4 → Database
```

This created several problems:

- High hardware cost
- Low resource utilization
- More electricity consumption
- More physical space required
- Difficult server management
- High maintenance cost

Most servers were using only **10-20%** of their CPU and RAM while the remaining resources stayed unused.

Virtualization solved this problem.

---

# What is Virtualization?

**Virtualization** is the technology that allows one physical machine to create multiple Virtual Machines.

Each VM behaves like an independent computer with its own:

- Operating System
- CPU
- RAM
- Storage
- Network Interface
- Applications

Because of virtualization, one powerful server can host many virtual machines.

---

# Example

Suppose you have a laptop with

- 16 GB RAM
- 8 CPU Cores
- 1 TB SSD

Instead of installing only Windows, you can create:

```text
Physical Laptop

├── Windows VM
├── Ubuntu VM
├── Kali Linux VM
└── CentOS VM
```

All these operating systems run simultaneously on the same physical machine.

Each VM thinks it owns the entire computer.

---

# Real-World Example

Imagine a large apartment building.

The building is the **Physical Server**.

Each apartment is a **Virtual Machine**.

Every apartment has:

- Its own owner
- Furniture
- Electricity
- Kitchen
- Privacy

Although everyone lives in the same building, each apartment is isolated.

Similarly,

- Physical Server = Apartment Building
- Virtual Machine = Individual Apartment

---

# Components of a Virtual Machine

## 1. Host Machine

The physical computer on which Virtual Machines are created.

Example:

- Laptop
- Desktop
- Physical Server

---

## 2. Hypervisor

A software layer that creates and manages Virtual Machines.

It allocates:

- CPU
- RAM
- Disk
- Network

to each VM.

Without a Hypervisor, Virtual Machines cannot exist.

---

## 3. Guest Operating System

The operating system installed inside the VM.

Examples:

- Ubuntu
- Windows Server
- CentOS
- Debian
- Fedora

---

## 4. Virtual Hardware

Every VM gets virtual hardware such as:

- Virtual CPU (vCPU)
- Virtual RAM
- Virtual Hard Disk
- Virtual Network Card

Although these are virtual, they function like real hardware.

---

# Architecture of a Virtual Machine

```text
                 Applications

                      │

             Guest Operating System

                      │

             Virtual Hardware (vCPU,
          RAM, Disk, Network Adapter)

                      │

                Hypervisor Layer

                      │

      Physical Hardware (CPU, RAM, SSD)

```

---

# What is a Hypervisor?

A **Hypervisor** is software that creates, runs, and manages Virtual Machines.

Its responsibilities include:

- Creating Virtual Machines
- Allocating CPU
- Allocating RAM
- Managing Storage
- Managing Networking
- Isolating VMs
- Scheduling CPU time

It acts as a bridge between the hardware and the Virtual Machines.

---

# Types of Hypervisors

## Type 1 Hypervisor (Bare Metal)

Runs directly on physical hardware.

```text
Applications

Operating Systems

Hypervisor

Hardware
```

### Examples

- VMware ESXi
- Microsoft Hyper-V
- Xen
- KVM

### Advantages

- High Performance
- Better Security
- Better Resource Utilization
- Used in Data Centers and Cloud Providers

---

## Type 2 Hypervisor (Hosted)

Runs on top of an existing Operating System.

```text
Applications

Guest OS

Hypervisor

Host Operating System

Hardware
```

### Examples

- Oracle VirtualBox
- VMware Workstation
- VMware Fusion
- Parallels Desktop

### Advantages

- Easy to Install
- Great for Learning
- Used by Developers and Students

---

# Virtual Machine Lifecycle

A VM typically goes through the following stages:

1. Create
2. Install Operating System
3. Configure Resources
4. Install Applications
5. Start
6. Stop
7. Restart
8. Suspend
9. Resume
10. Delete

---

# Advantages of Virtual Machines

## Better Resource Utilization

Run multiple operating systems on one server.

---

## Cost Reduction

Less hardware is required.

---

## Isolation

One VM crashing does not affect other VMs.

---

## Easy Backup

VMs can be backed up as files.

---

## Snapshots

Save the current state of a VM and restore it later.

---

## Scalability

Create new VMs quickly whenever needed.

---

## Disaster Recovery

VMs can easily be migrated to another server.

---

## Testing Environment

Developers can safely test applications without affecting the host system.

---

# Disadvantages of Virtual Machines

- Slight performance overhead
- High RAM consumption
- Larger storage requirements
- Slower boot time compared to containers
- Each VM requires its own operating system

---

# Virtual Machine vs Physical Machine

| Physical Machine | Virtual Machine |
|------------------|-----------------|
| Real hardware | Software-based computer |
| One Operating System | Multiple Operating Systems possible |
| Expensive | Cost-effective |
| Hard to clone | Easy to clone |
| Hardware dependent | Portable |
| Low flexibility | Highly flexible |

---

# Virtual Machine vs Container

| Virtual Machine | Container |
|-----------------|-----------|
| Includes complete Operating System | Shares Host Operating System |
| Large in size (GBs) | Small in size (MBs) |
| Slower startup | Starts within seconds |
| High resource usage | Lightweight |
| Better isolation | Moderate isolation |
| Managed by Hypervisor | Managed by Container Runtime (Docker) |

---

# Popular Virtualization Software

## Desktop

- Oracle VirtualBox
- VMware Workstation
- VMware Fusion
- Parallels Desktop

---

## Enterprise

- VMware ESXi
- Microsoft Hyper-V
- KVM
- Xen

---

# Virtual Machines in Cloud Computing

Cloud providers create Virtual Machines for customers using virtualization.

Examples:

- AWS → EC2 Instance
- Microsoft Azure → Azure Virtual Machine
- Google Cloud → Compute Engine

When you launch an EC2 instance on AWS, you are actually creating a Virtual Machine on one of AWS's physical servers.

---

# Interview Questions

## What is a Virtual Machine?

A Virtual Machine is a software-based computer that runs its own operating system on top of a physical machine using virtualization technology.

---

## What is Virtualization?

Virtualization is the process of creating multiple Virtual Machines on a single physical server using a Hypervisor.

---

## What is a Hypervisor?

A Hypervisor is software that creates, manages, and allocates resources to Virtual Machines.

---

## Name the two types of Hypervisors.

1. Type 1 (Bare Metal)
2. Type 2 (Hosted)

---

## Why are Virtual Machines used?

- Better hardware utilization
- Cost reduction
- Isolation
- Scalability
- Easy backup
- Disaster recovery
- Testing and development

---

# Key Takeaways

- A Virtual Machine is a software-based computer.
- Virtualization allows multiple VMs to run on one physical server.
- A Hypervisor manages and creates Virtual Machines.
- Each VM has its own Operating System.
- VMs provide strong isolation but consume more resources than containers.
- Cloud services like AWS EC2 are based on Virtual Machine technology.