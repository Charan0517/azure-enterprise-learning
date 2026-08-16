# Azure Virtual Machines — Hands-On Lab

This lab connects the cloud and virtualization concepts to a real Azure resource.

## Lab objective

Create a Linux virtual machine in Azure, understand the supporting resources Azure creates, connect securely using SSH, expose a simple web service, verify public/private networking, and shut the VM down correctly when it is not needed.

## Architecture we built

Conceptually:

```text
Your Computer
     │
     │ Internet
     │ SSH / HTTP
     ▼
Public IP
     │
     ▼
Network Interface
     │
     ▼
Azure Virtual Network / Subnet
     │
     ▼
Ubuntu Virtual Machine
     │
     └── Web Server
```

The VM also has a private IP address for communication inside its virtual network.

## VM configuration decisions

During the lab we reviewed several VM creation options.

### Region

The VM was created in:

```text
West US 2
```

A region is the geographic Azure location in which the resource is deployed.

Region choice can affect:

- Latency
- Service availability
- Pricing
- Compliance/data residency
- Disaster recovery architecture

### Availability options

The portal presented options such as:

- Availability Zone
- Virtual Machine Scale Set
- Availability Set

For a learning VM, the goal was initially to understand a single VM rather than build a production HA architecture.

### Image

We selected an Ubuntu Linux image.

A VM image provides the starting operating-system environment from which the VM is created.

Other images may include Windows Server, Debian, Red Hat, SUSE, Oracle Linux, and others.

### Security type

We reviewed security-type options including Standard and Trusted Launch.

For the learning VM, Standard was used while focusing first on the core VM architecture.

Trusted Launch is an important security topic that we can cover separately rather than treating a portal option as something to select without understanding it.

### Architecture

The VM used x64 architecture.

### VM size

A B-series VM was selected because this was a small learning workload and cost mattered.

VM size controls resources such as:

- vCPU
- Memory
- Disk capabilities
- Network capabilities

The important lesson is that the VM size is the virtual hardware exposed to the guest operating system.

The physical Azure host underneath can be much larger.

```text
Physical Azure Host
        │
        │ Hypervisor
        ▼
Your VM
├── Assigned vCPU
├── Assigned RAM
├── Virtual disks
└── Virtual NIC
```

## Authentication — SSH

For Linux administration we configured SSH access.

SSH stands for **Secure Shell**.

It provides encrypted remote command-line access to the Linux VM.

Conceptually:

```text
Your Laptop
    │
    │ SSH
    ▼
Public IP of VM
    │
    ▼
Linux SSH Service
    │
    ▼
Ubuntu shell
```

SSH commonly uses TCP port 22.

Access to that port should be restricted appropriately in real environments rather than exposed broadly without a reason.

## Public IP vs private IP

After deployment we observed both addresses.

### Private IP

The private IP belongs to the VM's network interface inside the Azure virtual network.

Example concept:

```text
VNet
└── Subnet
    └── VM → 10.x.x.x private address
```

Private addresses are used for internal communication and are not directly Internet-routable.

### Public IP

A public IP allows Internet-originated communication to reach the Azure resource when networking and security rules allow it.

```text
Internet
   │
   ▼
Public IP
   │
   ▼
NIC / VM
```

Having a public IP alone does not mean every port is automatically allowed. Network security rules and the operating system firewall/service configuration also matter.

## Network Security Group concept

An NSG controls allowed/denied network traffic based on rules such as:

```text
Source
Destination
Port
Protocol
Direction
Priority
```

For example:

```text
SSH  → TCP 22
HTTP → TCP 80
HTTPS → TCP 443
```

Opening a port is not the same as starting an application on that port.

For HTTP to work, something such as a web server must actually be listening on port 80.

## Verifying the VM from Linux

After connecting, Linux commands can show the resources visible to the guest OS.

For example:

```bash
nproc
free -h
ip addr
```

These help verify:

- Number of visible CPUs
- Memory
- Network interfaces/private address

This connects the Azure portal VM size to what the operating system actually sees.

## Running a web server

After the VM was running, we configured a web service and confirmed that its page was accessible.

The request path becomes:

```text
Browser
   │
   │ HTTP request
   ▼
VM Public IP
   │
   ▼
Azure networking / NSG
   │
   ▼
VM network interface
   │
   ▼
Ubuntu OS
   │
   ▼
Web server
   │
   ▼
HTTP response
   │
   ▼
Browser displays page
```

This is an important milestone because the VM is no longer just a portal resource — it is actually serving an application across a network.

## Stopping the VM and cost

A critical Azure cost lesson is the difference between shutting down inside the guest OS and ensuring Azure compute is **deallocated**.

When a VM is deallocated, Azure releases the compute allocation and VM compute billing stops. Other resources can still generate charges, such as disks or certain networking resources.

Therefore:

> Stopping the VM does not necessarily mean the entire solution costs $0.

For learning labs, check the Azure portal status and make sure the VM shows **Stopped (deallocated)** when you are finished with compute.

## Enterprise architecture difference

Our learning VM is intentionally simple:

```text
Internet → Public IP → Single VM
```

A production enterprise architecture would often avoid directly exposing application VMs and might instead use:

```text
Internet
   │
   ▼
Application Gateway / Load Balancing Layer
   │
   ▼
Private Application VMs
   │
   ▼
Private Data Services
```

with monitoring, centralized identity, restricted administration, backup, availability design, and governance.

The lab gives us the building block. Later modules will turn the building block into an enterprise architecture.

## What we proved in this lab

We successfully connected several concepts:

```text
Azure physical infrastructure
        ↓
Virtualization
        ↓
Virtual machine
        ↓
Virtual CPU / memory / NIC
        ↓
VNet + private IP
        ↓
Public connectivity
        ↓
SSH administration
        ↓
Web application traffic
```

This foundation will make Azure networking much easier to understand because we now have a real compute resource to connect to networks, load balancers, DNS, security controls, and monitoring.