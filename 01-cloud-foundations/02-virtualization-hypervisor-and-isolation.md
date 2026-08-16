# Virtualization — From Physical Servers to Virtual Machines

The cloud computing problem leads directly to virtualization.

A cloud provider can own thousands of powerful physical servers, but simply owning a lot of hardware does not solve the utilization problem. If every customer who asks for a small server receives an entire physical machine, most of the physical capacity could still remain unused.

Suppose one physical server has:

```text
Physical Server
├── 64 CPU cores
├── 256 GB RAM
├── 8 TB storage
└── High-speed network interfaces
```

A customer may need only:

```text
2 CPU
8 GB RAM
Ubuntu
```

Giving that customer the complete physical server would waste most of the hardware.

So a more useful question is:

> Can one powerful physical computer safely behave like several independent computers?

That is the problem virtualization solves.

---

## Before virtualization — one workload tied closely to one physical server

A traditional server environment could look like this:

```text
Payroll Application
        │
        ▼
Operating System
        │
        ▼
Physical Server 1

HR Application
        │
        ▼
Operating System
        │
        ▼
Physical Server 2

Inventory Application
        │
        ▼
Operating System
        │
        ▼
Physical Server 3
```

This provides separation, but it can produce poor utilization.

For example:

```text
Server 1 capacity
CPU: 32 cores
RAM: 128 GB

Payroll usage
CPU: 4 cores worth of work
RAM: 16 GB
```

A large part of the machine may remain unused while another application requires additional capacity elsewhere.

The physical server is also a fixed hardware boundary. Moving the workload normally means preparing another server, installing an operating system, configuring software and transferring the application and data.

Virtualization introduces a layer between the operating systems and the physical hardware so the physical machine can be divided into multiple virtual computing environments.

---

# What virtualization creates

After virtualization, the same physical host can support several Virtual Machines (VMs).

```text
                         PHYSICAL SERVER
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│   │     VM 1     │  │     VM 2     │  │     VM 3     │       │
│   │              │  │              │  │              │       │
│   │ Ubuntu       │  │ Windows      │  │ Linux        │       │
│   │ Application A│  │ Application B│  │ Application C│       │
│   └──────────────┘  └──────────────┘  └──────────────┘       │
│                                                              │
│                 Virtualization Layer                         │
│                                                              │
│ CPU                 RAM                Storage      Network   │
└──────────────────────────────────────────────────────────────┘
```

Each VM behaves like an independent computer even though the underlying physical hardware is shared.

VM1 could run Ubuntu while VM2 runs Windows. Rebooting the guest operating system in one VM does not mean all the other VMs have to reboot.

This gives much better flexibility than dedicating one physical machine to every workload.

---

# A VM needs virtual hardware

An operating system expects a computer to have hardware.

Ubuntu expects CPUs, memory, disks and networking. Windows expects the same kinds of resources.

But a VM does not normally receive physical RAM modules or a dedicated physical CPU socket directly. Instead, virtualization presents **virtual hardware** to the guest operating system.

A VM can therefore see something like:

```text
Virtual Machine
│
├── 2 vCPU
├── 8 GB virtual memory
├── Virtual disk
├── Virtual network interface
└── Firmware / other virtual devices
```

The guest operating system treats these resources as the hardware available to that machine.

This is why, during our Azure VM lab, Ubuntu could report the CPUs allocated to the VM even though we never purchased a physical server ourselves.

Conceptually:

```text
Application
    │
    ▼
Guest Operating System
    │
    ▼
Virtual Hardware
    │
    ▼
Virtualization Layer
    │
    ▼
Physical Hardware
```

The operating system is still important. Virtualization does not replace Ubuntu or Windows. It creates the virtual machine on which the guest operating system runs.

---

# The Hypervisor

Now another problem appears.

If three VMs share one physical machine, something has to coordinate them.

VM1 cannot simply decide to use all physical memory. VM2 cannot be allowed to overwrite VM1's memory. Each VM needs virtual CPUs mapped onto real processing resources, virtual memory backed by physical memory, and controlled access to storage and networking.

The component responsible for managing this virtualization is the **hypervisor**.

A simplified architecture is:

```text
┌────────────────────┐   ┌────────────────────┐   ┌────────────────────┐
│       VM 1         │   │       VM 2         │   │       VM 3         │
│                    │   │                    │   │                    │
│ Application        │   │ Application        │   │ Application        │
│ Ubuntu             │   │ Windows            │   │ Linux              │
│ Virtual Hardware   │   │ Virtual Hardware   │   │ Virtual Hardware   │
└─────────┬──────────┘   └─────────┬──────────┘   └─────────┬──────────┘
          │                        │                        │
          └────────────────────────┼────────────────────────┘
                                   ▼
                         ┌──────────────────┐
                         │    Hypervisor    │
                         │                  │
                         │ VM scheduling    │
                         │ Memory control   │
                         │ Device access    │
                         │ Isolation        │
                         └────────┬─────────┘
                                  ▼
                         ┌──────────────────┐
                         │ Physical Server  │
                         │ CPU / RAM / NIC  │
                         │ Storage access   │
                         └──────────────────┘
```

The hypervisor is therefore not another application running inside our Ubuntu VM. It belongs to the virtualization platform underneath the VM.

---

# Type 1 and Type 2 hypervisors

Hypervisors are commonly discussed in two broad categories.

## Type 1 — bare-metal hypervisor

A Type 1 hypervisor runs directly on the server hardware or as part of the platform directly controlling that hardware.

```text
VMs
 │
 ▼
Hypervisor
 │
 ▼
Physical Hardware
```

This model is commonly associated with datacenters and enterprise virtualization because the virtualization layer directly controls the host resources.

Microsoft's Hyper-V architecture is an example associated with this class of virtualization, and Azure's infrastructure uses Microsoft's virtualization technologies at cloud scale.

## Type 2 — hosted hypervisor

A Type 2 hypervisor runs on top of a normal host operating system.

```text
Virtual Machines
      │
      ▼
Virtualization Application
      │
      ▼
Host Operating System
      │
      ▼
Physical Hardware
```

This is common for desktop/laptop virtualization and development scenarios.

For example, a developer may run a Linux VM on a Windows laptop using desktop virtualization software.

The important difference is the extra host operating-system layer in the hosted model.

---

# What does a vCPU actually mean?

When Azure says a VM has **2 vCPUs**, it does not mean Microsoft removed two physical processors from a server and permanently handed them to that VM.

A vCPU is a virtual processor exposed to the guest VM. The virtualization platform schedules virtual CPU work onto the host's physical CPU execution resources.

Conceptually:

```text
VM 1                     VM 2
2 vCPU                    2 vCPU
  │                         │
  └───────────┬─────────────┘
              ▼
      Hypervisor scheduling
              │
              ▼
      Physical CPU resources
```

Suppose VM1 and VM2 both need CPU time. The virtualization platform schedules their work on available physical CPU resources according to the platform's scheduling and resource-management rules.

From inside VM1, Ubuntu sees the virtual CPUs assigned to it and schedules its processes against those CPUs.

This gives two scheduling layers conceptually:

```text
Application processes
        │
        ▼
Guest OS scheduler
        │
        ▼
VM's vCPUs
        │
        ▼
Hypervisor / platform scheduler
        │
        ▼
Physical CPU execution resources
```

This is why our Ubuntu VM could report **2 CPUs** with `nproc`: those were the CPUs visible to the guest operating system.

---

# What happens with memory?

Memory needs even stronger separation.

Imagine:

```text
Physical Host RAM
256 GB
```

Several VMs may be allocated portions of that host's memory resources.

```text
VM1 → allocated virtual memory
VM2 → allocated virtual memory
VM3 → allocated virtual memory
```

Each guest operating system operates within the memory space provided to it. The virtualization and hardware memory-management mechanisms enforce mappings between guest memory and physical host memory.

The important result is isolation:

```text
VM1 memory
    │
    X  VM2 cannot simply read it
    │
VM2 memory
```

Without this isolation, public cloud multi-tenancy would be impossible because one customer's workload could inspect another customer's data.

---

# VM isolation

Sharing physical infrastructure does **not** mean sharing operating systems or application memory.

Consider two different Azure customers whose workloads happen to use infrastructure within the same cloud environment.

Conceptually:

```text
                    Physical Host
                         │
                  Virtualization
                  /             \
                 /               \
                ▼                 ▼
       Customer A VM        Customer B VM
       Ubuntu               Windows
       App A                App B
       Memory A             Memory B
```

Customer A should not be able to say:

> Show me whatever is currently stored in Customer B's VM memory.

The virtualization boundary isolates the VM environments.

Isolation applies across several dimensions, including virtual CPU execution context, memory mappings, virtual devices and access to the resources presented to each VM.

This isolation is one of the key foundations of **multi-tenancy** in cloud computing.

---

# Resource pooling

Virtualization also makes resource pooling practical.

Instead of thinking of a physical server as belonging permanently to one application, the cloud provider can manage large pools of infrastructure and allocate virtual resources according to customer requirements.

```text
                   AZURE COMPUTE CAPACITY
                           │
              ┌────────────┼────────────┐
              │            │            │
              ▼            ▼            ▼
          Customer A   Customer B   Customer C
             VM           VM           VM
```

Customers request a VM size rather than choosing a particular physical server rack and CPU socket.

For example:

```text
Request
├── Region: West US 2
├── Image: Ubuntu
├── VM size: selected B-series size
├── vCPU requirement
└── Memory requirement
```

Azure's platform determines where the VM can be placed based on capacity, availability requirements, hardware capabilities and other platform constraints.

The customer normally works with the **logical Azure resource**, not the physical host underneath it.

---

# Resource overcommitment — what it means

Virtualization platforms can sometimes allocate virtual resources in ways that take advantage of the fact that workloads do not all use their maximum capacity at exactly the same time. This general idea is often called **overcommitment**.

A simplified example would be several VMs whose configured virtual CPU capacity is greater than the number of physical CPU execution resources available simultaneously, with the hypervisor scheduling the workloads over time.

```text
VM1 vCPU work ─┐
VM2 vCPU work ─┼──► Scheduler ───► Physical CPU
VM3 vCPU work ─┘
```

This does **not** mean every Azure VM size is simply an arbitrary overcommitted slice, and Azure's exact host scheduling/capacity implementation is a platform detail that varies by VM family and service design.

The useful concept is that virtualization separates the **virtual resources visible to a VM** from the details of the underlying physical hardware scheduling.

---

# Why multiple VMs on one host do not automatically give high availability

Virtualization allows this:

```text
Physical Host A
├── VM1
├── VM2
└── VM3
```

But now consider a physical-host failure.

```text
Physical Host A ❌
├── VM1 ❌
├── VM2 ❌
└── VM3 ❌
```

Having three VMs did not help if all three depended on the same physical failure boundary.

This introduces an important architecture lesson:

> **Virtualization provides isolation and flexible resource allocation, but virtualization alone does not guarantee application availability.**

For availability, redundant application instances need to be distributed across appropriate failure boundaries.

That is why concepts such as fault domains, availability sets and availability zones become important later.

---

# VM lifecycle is different from physical-server lifecycle

With a physical server, replacing or moving a workload can require physical work.

With a VM, the machine is represented largely through software-defined configuration and virtualized resources.

This enables operations such as:

```text
Create VM
   ↓
Start VM
   ↓
Stop / Deallocate VM
   ↓
Restart VM
   ↓
Resize VM
   ↓
Delete VM
```

The physical host underneath is an implementation detail managed by Azure.

During our Azure lab, we saw this separation directly.

When the VM was restarted:

```text
VM restart
    ↓
Guest OS rebooted
    ↓
Persistent OS disk remained
    ↓
Ubuntu returned
    ↓
Nginx returned
    ↓
Our HTML file still existed
```

The VM's execution state and persistent storage were separate concerns.

When the VM was later **Stopped (deallocated)**, Azure released the VM's compute allocation while the VM configuration and persistent resources remained available for a later start. The compute charge for the deallocated VM stopped, although attached resources such as managed disks and certain networking resources can still have their own charges.

---

# VM migration and host maintenance

Another benefit of virtualization is that the workload is no longer conceptually welded to one physical machine in the same way as a traditional bare-metal installation.

Cloud platforms can perform infrastructure maintenance and use virtualization capabilities to manage workloads across hosts. The exact behavior depends on the Azure service, maintenance event, VM type and platform capabilities; some events can be handled transparently while others can involve a pause or reboot.

The important architectural idea is:

```text
Application
    ↓
Guest VM
    ↓
Virtualization abstraction
    ↓
Physical host is managed by Azure
```

The customer manages the Azure VM resource while Azure manages the underlying datacenter hosts.

---

# Connecting virtualization to our Azure VM lab

When we created our Ubuntu VM, the portal asked us to choose a **VM size**.

The size described the compute resources/capabilities Azure would expose to the VM.

Inside Ubuntu we ran commands such as:

```bash
nproc
free -h
hostname -I
```

Ubuntu reported the resources it could see:

```text
Ubuntu Guest OS
│
├── 2 CPUs visible
├── Guest memory available
├── Virtual disk/filesystem
└── Virtual network interface with private IP
```

The complete conceptual stack was therefore:

```text
Our Nginx application
        │
        ▼
Ubuntu Guest Operating System
        │
        ▼
Virtual Machine
├── vCPU
├── Virtual memory
├── Virtual disk interface
└── Virtual NIC
        │
        ▼
Azure Virtualization Platform
        │
        ▼
Azure Physical Datacenter Infrastructure
├── Physical compute
├── Physical memory
├── Storage infrastructure
└── Physical networking
```

We interacted with the VM and Azure resources. We did not need to know which exact physical server rack was executing our VM.

That abstraction is one of the major reasons cloud infrastructure can be delivered at large scale.

---

# A complete real-world example

Imagine an organization needs separate Development, UAT and Production application servers.

Without virtualization, it might purchase three physical servers:

```text
Physical Server 1 → DEV
Physical Server 2 → UAT
Physical Server 3 → PROD
```

DEV may sit idle at night. UAT may be used only during testing. Production may need significantly more resources.

With virtualization, infrastructure can be pooled and the environments can be represented as independent VMs:

```text
                    Virtualized Infrastructure
                              │
            ┌─────────────────┼─────────────────┐
            │                 │                 │
            ▼                 ▼                 ▼
          DEV VM            UAT VM           PROD VM
          Linux             Linux             Linux
          Smaller           Medium            Larger
          capacity          capacity          capacity
```

Each VM has its own guest OS and virtual resources, while the cloud provider manages the physical infrastructure underneath.

The production environment should still be architected for its own availability, security and performance requirements. Virtualization gives us the building blocks; it does not automatically make every workload highly available or disaster resistant.

---

# Where virtualization leads next

Virtualization solved several major problems:

```text
One physical server per workload
        ↓
Poor utilization
        ↓
Virtualization
        ↓
Multiple isolated VMs can use shared infrastructure
        ↓
Resource pooling and flexible VM creation
```

But it creates the next architectural question.

Suppose our application runs on one VM and the physical host underneath that VM fails.

Or suppose we create two VMs but both are affected by the same underlying failure boundary.

```text
Users
  │
  ▼
Application VM(s)
  │
  ▼
Underlying infrastructure failure ❌
```

How do we keep the application available when infrastructure components fail?

That problem leads naturally into **High Availability, fault domains, Availability Sets and Availability Zones**.