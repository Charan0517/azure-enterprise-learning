# Azure Virtual Machines (IaaS) — From Physical Infrastructure to Our First Azure Workload

Azure Virtual Machines are our first major Azure Compute service because they connect almost every cloud-foundation concept we learned to a real Azure resource.

We already built and operated an Ubuntu VM in Azure, connected to it with SSH, installed Nginx, hosted a web page, observed CPU metrics, restarted it, and finally stopped/deallocated it. This note documents both the concepts and that practical flow.

---

# 1. The real-world problem

Traditionally, if a team needs a server for an application, someone may need to:

```text
Purchase physical hardware
      ↓
Rack/install server
      ↓
Connect network
      ↓
Install operating system
      ↓
Configure storage
      ↓
Install application/runtime
      ↓
Maintain hardware + OS
```

This can take significant time and creates hardware ownership responsibilities.

With Azure Virtual Machines:

```text
Engineer
   ↓
Azure Portal / CLI / PowerShell / IaC / API
   ↓
Request VM configuration
   ↓
Azure allocates virtualized compute
   ↓
VM becomes available
```

We consume compute capacity without buying the physical host ourselves.

---

# 2. Azure VM is IaaS

Azure Virtual Machines are **Infrastructure as a Service (IaaS)**.

Microsoft manages the physical cloud infrastructure and virtualization platform.

We manage much of the guest environment.

```text
MICROSOFT MANAGES
─────────────────
Datacenter
Physical servers
Physical networking
Hypervisor / virtualization platform
Underlying cloud infrastructure

CUSTOMER MANAGES
────────────────
Guest operating system
OS updates/patching
Installed software
Application/runtime
Application configuration
Data
Guest-level security configuration
```

This explains why, after Azure created our Ubuntu VM, we still had to SSH into it and install Nginx ourselves.

---

# 3. VM architecture

Conceptually:

```text
Physical Azure Host
        │
        ▼
Hypervisor
        │
        ├── VM A
        │   ├── vCPU
        │   ├── virtual memory
        │   ├── virtual disks
        │   ├── virtual NIC
        │   └── guest OS
        │
        └── Other isolated workloads
```

This connects directly to our virtualization foundation:

```text
Physical CPU → vCPU
Physical memory → virtual memory
Physical network → virtual NIC
Physical storage → virtual disks
```

The guest operating system behaves as though it has its own machine, while the hypervisor maps virtual resources to physical infrastructure.

---

# 4. What Azure creates around a VM

A useful mistake to avoid is thinking:

> VM = one single Azure object with everything inside it.

A working Azure VM normally depends on several Azure resources.

Conceptually:

```text
Resource Group
│
├── Virtual Machine
│    ├── vCPU + memory allocation
│    └── OS configuration
│
├── OS Disk
│
├── Network Interface (NIC)
│      └── Private IP
│
├── Virtual Network
│      └── Subnet
│
├── Network Security Group
│
└── Public IP (when configured)
```

Each resource has its own lifecycle, configuration, and potentially its own cost.

---

# 5. Subscription and Resource Group

When creating our VM, the resource first existed inside the Azure resource hierarchy:

```text
Microsoft Entra Tenant
        ↓
Subscription
        ↓
Resource Group
        ↓
VM and supporting resources
```

The **subscription** provides a management/billing boundary.

The **resource group** provides a logical lifecycle and management container for related resources.

This connects the VM lab to our governance notes.

---

# 6. Region

We selected an Azure region when creating the VM.

Our lab used **West US 2**.

Conceptually:

```text
Azure Global Infrastructure
        ↓
West US 2 Region
        ↓
Azure physical infrastructure
        ↓
Our VM
```

Region selection affects factors such as:

```text
Latency
VM/SKU availability
Pricing
Availability Zone support
Data residency
Disaster Recovery design
Capacity/quota
```

A production team should not select a region only because it appears first in the portal.

---

# 7. Availability options

During VM creation we saw availability choices such as:

```text
No infrastructure redundancy required
Availability Zone
Virtual Machine Scale Set
Availability Set
```

These represent different architecture choices.

## No infrastructure redundancy required

Suitable for workloads where a single VM is acceptable, such as our temporary learning environment.

```text
One VM
   ↓
If that VM becomes unavailable
   ↓
Application becomes unavailable
```

## Availability Zone

Allows supported resources to be placed in a physically separate zone within a region.

For HA, multiple instances should be distributed appropriately across zones.

## Availability Set

A classic VM availability mechanism that distributes VMs across fault/update domains within a datacenter infrastructure scope.

## Virtual Machine Scale Set

Designed to manage a group of VM instances and can support scaling and availability patterns.

We will treat VM Scale Sets as their own Compute topic because their operational/scaling model deserves separate discussion.

---

# 8. Security type

During creation we saw options including:

```text
Standard
Trusted launch virtual machines
Confidential virtual machines
```

These options change the security capabilities and VM requirements.

For learning, the important mental model is:

```text
Standard
   ↓
Traditional supported VM security configuration

Trusted Launch
   ↓
Adds protections designed to help defend the VM boot/integrity path using supported security features

Confidential VM
   ↓
Designed for scenarios requiring stronger protection of data while it is being processed, using supported confidential-computing technology
```

Exact support depends on VM generation, image, size, region, and other service requirements.

Security type should be selected from workload requirements rather than simply choosing the most advanced-sounding option.

---

# 9. Image

The VM **image** provides the starting operating-system/software template.

We selected Ubuntu.

Other available image families can include:

```text
Windows Server
Ubuntu
Debian
Red Hat Enterprise Linux
SUSE Linux Enterprise
Oracle Linux
Other marketplace/custom images
```

Conceptually:

```text
VM configuration
      +
Selected image
      ↓
Guest OS environment
```

The image is not the physical VM. It is the template used to create the VM's guest environment.

---

# 10. VM architecture — x64 vs Arm64

The portal can expose processor architecture choices depending on image and VM family.

Common architecture families include:

```text
x64
Arm64
```

The selected OS image and software must be compatible with the processor architecture.

For most learning scenarios, x64 is familiar and widely compatible, but Arm-based VM families can be useful for workloads that support them and where their performance/cost characteristics fit the requirement.

---

# 11. VM size

The **VM size** determines an important part of the compute capacity allocated to the VM.

A size defines characteristics such as:

```text
vCPU count
Memory
Supported disk count/type characteristics
Disk throughput/IOPS limits
Network performance characteristics
Temporary/local storage on applicable sizes
Architecture/features
Price
```

During our lab we reviewed B-series options because we wanted a low-cost learning VM.

The key lesson is:

> Never select a VM only by the lowest displayed hourly/monthly estimate.

It still has to satisfy workload requirements and subscription/region availability.

---

# 12. What is a vCPU?

A vCPU is virtual CPU capacity presented to the VM by the virtualization platform.

Inside Linux, CPU information may show the processors available to the guest.

Conceptually:

```text
Physical CPU cores
      ↓
Hypervisor scheduling
      ↓
vCPUs presented to VM
      ↓
Ubuntu processes use CPU time
```

The VM does not directly own the entire physical CPU socket.

---

# 13. B-series / burstable VMs

B-series VM families are designed for workloads that normally use relatively little CPU but occasionally need to burst higher.

Example workload pattern:

```text
Most of the time
CPU usage = low

Occasionally
CPU demand = high
```

This can fit:

```text
Development/test
Small web servers
Low-traffic applications
Learning environments
Other burstable workloads
```

The exact credit/burst model depends on the B-series generation/SKU, so current Azure documentation should be checked when selecting one for production.

---

# 14. “Free services eligible” does not always mean available

During our lab we saw a size marked as eligible for free-service benefits but also unavailable for our subscription/region.

These are separate concepts.

```text
Free-service eligible
        ≠
Guaranteed deployable in every subscription/region
```

Deployment can be affected by:

```text
Subscription offer
Regional capacity
SKU availability
Quota
Policy
Account restrictions
```

This is an important real-world Azure lesson: portal options can be constrained by both architecture and subscription context.

---

# 15. Azure Spot VMs

The portal also offered an Azure Spot option.

Spot VMs can provide unused Azure compute capacity at lower cost, but Azure can evict the workload when capacity is needed or other Spot conditions are met.

Therefore Spot is generally suited to interruptible workloads such as:

```text
Batch processing
Dev/test
Stateless workers
Fault-tolerant distributed jobs
Other restartable workloads
```

It is generally not the right default for a single critical production server that cannot tolerate interruption.

---

# 16. Administrator authentication

A VM needs an administrator authentication method.

For Linux, SSH public-key authentication is commonly preferred.

The basic model is:

```text
Your computer
├── Private key  ← keep secret
└── Public key
        │
        ▼
Azure VM
Authorized public key
```

When connecting:

```text
SSH client proves possession of private key
        ↓
VM validates against authorized public key
        ↓
Login allowed
```

The private key should never be uploaded publicly or committed to GitHub.

---

# 17. What SSH is

**SSH — Secure Shell** is a secure protocol used to remotely access and administer systems, especially Linux servers.

Conceptually:

```text
Your Laptop
   │
   │ Encrypted SSH connection
   ▼
Linux VM
```

Typical command form:

```bash
ssh username@PUBLIC_IP
```

If key authentication is used, SSH uses the associated private key to authenticate.

SSH commonly uses TCP port 22.

---

# 18. Public IP vs Private IP

Our VM had both a private IP and a public IP.

## Private IP

Used for communication inside the Azure virtual network/private network scope.

Example:

```text
172.16.0.x
```

Conceptually:

```text
VM A 172.16.0.4
      ↕
Azure VNet
      ↕
VM B 172.16.0.5
```

## Public IP

Provides an Internet-routable address when configured and allowed by the network/security design.

```text
Your laptop on Internet
        ↓
Public IP
        ↓
VM NIC/private networking
        ↓
VM
```

A VM does not need a public IP simply because it exists in Azure. Production architectures often avoid direct public exposure when it is unnecessary.

---

# 19. Network Interface (NIC)

The VM connects to Azure networking through a **network interface**.

Conceptually:

```text
VM
 ↓
Virtual NIC
 ↓
Subnet
 ↓
Virtual Network
```

The NIC holds network configuration such as private IP association and participates in NSG/network behavior depending on the architecture.

We will cover networking deeply in the Networking module.

---

# 20. Network Security Group (NSG)

An NSG filters network traffic using security rules.

For our Linux VM, SSH required appropriate inbound access.

For the Nginx page, HTTP access required appropriate inbound access on port 80.

Conceptually:

```text
Internet
   ↓
Public IP
   ↓
NSG rule evaluation
   ↓
Allowed traffic
   ↓
VM
```

Example:

```text
TCP 22 → SSH
TCP 80 → HTTP
TCP 443 → HTTPS
```

Opening a port is a security decision. Production rules should restrict source/destination/port/protocol according to requirements rather than allowing unnecessary Internet access.

---

# 21. Our SSH connection

After deployment completed, the VM status showed **Running** and Azure displayed its public and private IP information.

We connected from our local computer to the Ubuntu VM using SSH.

Conceptually:

```text
Local Terminal
      │
      │ SSH / TCP 22
      ▼
VM Public IP
      │
      ▼
NSG allows connection
      │
      ▼
Ubuntu SSH service
      │
      ▼
Shell session
```

At that point we were executing commands **inside the Azure VM**, not on our laptop.

---

# 22. Inspecting the VM from Linux

Inside Ubuntu, Linux commands let us inspect the resources visible to the guest OS.

Examples include:

```bash
nproc
free -h
ip addr
```

These can show information such as:

```text
CPU count visible to guest
Memory visible to guest
Private network interface/IP information
```

This was useful because it proved that the VM size we selected translated into resources visible inside the guest OS.

---

# 23. Installing Nginx

We then converted the VM from “just a Linux server” into a simple web server.

Conceptually:

```text
Ubuntu VM
   ↓
Install Nginx
   ↓
Nginx listens for HTTP requests
   ↓
Browser requests VM public IP
   ↓
Web page returned
```

Typical Ubuntu commands are:

```bash
sudo apt update
sudo apt install nginx -y
```

And service status can be checked with:

```bash
systemctl status nginx
```

The exact package/service commands depend on the Linux distribution.

---

# 24. Why the browser initially could not reach the page

Installing Nginx inside the VM is only one layer.

For a browser to reach it from the Internet, the complete path must work:

```text
Browser
   ↓
Internet
   ↓
Public IP
   ↓
NSG permits TCP 80
   ↓
VM network interface
   ↓
Ubuntu
   ↓
Nginx running/listening
   ↓
HTML content
```

A failure at any layer can prevent the page from loading.

This is the beginning of real infrastructure troubleshooting: follow the request layer by layer instead of randomly changing settings.

---

# 25. Updating the web page

Nginx served an HTML file from the VM's disk.

When the HTML/title syntax was wrong, the page did not appear as expected.

We edited the file and refreshed the browser.

This demonstrated an important separation:

```text
Azure infrastructure can be healthy
        BUT
Application/content can still be wrong
```

A VM showing **Running** only tells us the Azure compute resource is running. It does not prove the application is correct.

---

# 26. Why the page survived a VM restart

We restarted the VM and observed that:

```text
Nginx still existed
HTML file still existed
Page could be served again
```

Why?

Because the VM's operating system and files were stored on persistent disk storage rather than only in CPU/RAM.

Conceptually:

```text
VM Running
│
├── RAM → volatile runtime state
└── OS/Data disk → persistent files

Restart
│
├── RAM/process state resets
└── Persistent disk remains
```

After boot, services configured to start automatically can run again and access the same persistent files.

---

# 27. Restart vs Stop vs Deallocate

These states are extremely important in Azure.

## Restart

```text
VM reboots
Compute remains allocated
Persistent disks remain
Billing for VM compute continues
```

Comparable conceptually to rebooting a computer.

## Stop inside the operating system / allocated stopped state

A VM can be stopped while Azure compute remains allocated depending on how it is stopped/state shown.

If compute remains allocated, compute charges can continue.

## Stop (deallocate)

When Azure shows the VM as **Stopped (deallocated)**:

```text
VM compute allocation released
Persistent disks remain
Configuration remains
VM can be started again later
```

Compute charges for the VM instance stop while deallocated, but other resources can still incur charges.

---

# 28. Deallocated does NOT mean free

This is one of the most important cost lessons from our lab.

When the VM is deallocated:

```text
VM compute billing → stopped
```

But resources such as these may still cost money:

```text
Managed disks
Some public IP configurations
Snapshots
Backup
Other attached Azure services
```

Therefore:

> Stopped (deallocated) means compute capacity is released; it does not mean the entire resource group costs $0.

---

# 29. Why we deallocated our learning VM

Our VM existed only for learning and did not need to serve production traffic continuously.

Keeping it running during a long break would consume paid compute unnecessarily.

So we used:

```text
Learning session complete
        ↓
Stop VM from Azure
        ↓
Wait for Stopped (deallocated)
        ↓
Compute allocation released
```

This connected our VM lab to both **cost optimization** and **sustainability**.

---

# 30. Monitoring CPU

Azure exposes VM metrics such as CPU percentage.

When we generated/served activity, we observed CPU utilization change.

Conceptually:

```text
Workload activity
      ↓
CPU consumption changes
      ↓
Azure metric collected
      ↓
Metrics chart
```

Monitoring helps answer questions such as:

```text
Is the VM overloaded?
Is it mostly idle?
Did utilization change after deployment?
Do we need a larger/smaller VM?
Should we scale horizontally?
```

Metrics therefore feed capacity planning, troubleshooting, predictability, and rightsizing.

---

# 31. Vertical scaling of a VM

If a VM needs more CPU or memory, one option is to resize it to another supported VM size.

```text
Current VM
2 vCPU / X GB RAM
      ↓
Resize
      ↓
Larger VM
4 vCPU / more RAM
```

This is **vertical scaling — scale up**.

Reducing the VM size is **scale down**.

Resizing can involve restart/deallocation constraints depending on the requested size and available host capacity, so production changes must be planned.

---

# 32. Horizontal scaling

Instead of making one VM larger, we can add VM instances.

```text
One VM
   ↓
VM1 + VM2 + VM3
```

This is **horizontal scaling — scale out**.

For web applications, traffic can then be distributed through a load-balancing layer.

```text
Users
  ↓
Load Balancer
 ├── VM1
 ├── VM2
 └── VM3
```

Removing instances when demand decreases is **scale in**.

VM Scale Sets can automate/manage this pattern and will be studied separately.

---

# 33. VM and High Availability

One VM is a single application instance.

Even though Azure manages reliable physical infrastructure, one VM can still become unavailable due to guest issues, maintenance scenarios, configuration errors, application failures, or infrastructure events.

Production HA may require:

```text
Multiple VM instances
      +
Availability architecture
      +
Load balancing
      +
Resilient data dependencies
```

Example:

```text
                Users
                  ↓
            Load Balancer
             /          \
            ▼            ▼
      VM1 — Zone 1   VM2 — Zone 2
```

---

# 34. VM and Disaster Recovery

Availability Zones protect against many localized/zone failures **inside one region**.

They do not protect against complete regional loss.

Regional DR may require:

```text
Primary Region
      ↓
Replication / backup / recovery design
      ↓
Secondary Region
```

VM recovery architecture can use service-specific Azure recovery capabilities, infrastructure redeployment, replicated data, backups, or combinations depending on RTO/RPO.

The important point is:

```text
One VM in one region ≠ Disaster Recovery
```

---

# 35. VM disks

A VM can use several storage concepts.

## OS disk

Contains the guest operating system and system files.

```text
Ubuntu
Nginx package
System configuration
Our HTML file if stored there
```

## Data disks

Additional managed disks can be attached for application data.

## Temporary/local disk

Some VM sizes expose temporary local storage intended for data that can be lost during certain lifecycle/host events.

It must not be treated like durable persistent storage.

We will cover Azure storage in detail in the Storage module.

---

# 36. VM lifecycle

A useful operational model is:

```text
Plan
 ↓
Choose subscription/resource group
 ↓
Choose region
 ↓
Choose availability design
 ↓
Choose image
 ↓
Choose VM size
 ↓
Configure administrator/authentication
 ↓
Configure networking/security
 ↓
Configure disks
 ↓
Deploy
 ↓
Connect/configure application
 ↓
Monitor
 ↓
Patch + secure
 ↓
Resize/scale if required
 ↓
Backup/recover according to requirements
 ↓
Stop/deallocate when temporarily unused
 ↓
Delete safely when no longer required
```

---

# 37. Production responsibilities people often forget

Creating a VM is easy. Operating one responsibly requires more work.

Teams need to think about:

```text
OS patching
Vulnerability management
SSH/RDP access
Identity and privileges
Firewall/NSG rules
Application patching
Monitoring and alerting
Logging
Backup
Disaster Recovery
Disk capacity
Secrets
Certificates
Malware/endpoint protection where appropriate
Configuration management
Cost optimization
Lifecycle/decommissioning
```

This operational responsibility is one reason organizations may choose PaaS instead of IaaS when they do not require OS-level control.

---

# 38. When Azure VMs are a good fit

VMs are useful when workloads require:

```text
Operating-system control
Custom software installation
Legacy application compatibility
Specific runtime/system dependencies
Lift-and-shift migration
Server-level administration
Software that expects a traditional machine
```

---

# 39. When a VM may NOT be the best first choice

Suppose the requirement is simply:

> Host a modern web API.

Creating and maintaining an entire VM may be unnecessary if a managed Azure service satisfies the requirements.

Alternatives can include managed application platforms, containers, or serverless services depending on the workload.

Architecture should start from requirements rather than automatically selecting a VM because VMs are familiar.

---

# 40. Common misunderstandings

## “Running means my application is healthy.”

No. It means the Azure VM compute resource is running. The application can still be broken.

## “A public IP means anyone can connect to everything.”

No. Network security rules, guest firewall/service listeners, routing, and application configuration still determine connectivity.

## “Private IP can only be used inside the VM itself.”

No. It is used for private network communication within connected network scopes.

## “Restarting deletes my files.”

Persistent disk files remain through a normal restart.

## “Stopping Linux with a command always stops Azure billing.”

Do not assume that. Verify the Azure VM reaches **Stopped (deallocated)** when the goal is to release compute allocation.

## “Deallocated means every Azure resource is free.”

No. Persistent resources can continue incurring charges.

## “A bigger VM always makes the application faster.”

No. The bottleneck could be database, disk I/O, network, application code, locking, or another dependency.

## “One VM in an Availability Zone is highly available.”

No. It is still one instance in one zone.

---

# 41. Our lab architecture

```text
                         INTERNET
                            │
                            │ HTTP / SSH
                            ▼
                       Public IP
                            │
                            ▼
                   Network Security Group
                  TCP 22 / TCP 80 as allowed
                            │
                            ▼
                     Network Interface
                            │
                  Private IP: 172.16.0.x
                            │
                            ▼
                     Azure Virtual Network
                            │
                            ▼
                      Ubuntu Azure VM
                     ┌───────────────┐
                     │ vCPU          │
                     │ Memory        │
                     │ Ubuntu OS     │
                     │ Nginx         │
                     │ HTML page     │
                     └───────┬───────┘
                             │
                             ▼
                       Managed OS Disk
                      Persistent storage

Azure Monitor / Metrics
          ▲
          │ CPU percentage and other telemetry
          │
          └──────────── VM
```

And underneath everything:

```text
Microsoft Azure Datacenter
        ↓
Physical Host
        ↓
Hypervisor
        ↓
Our virtual machine
```

---

# 42. How the foundation topics appear in one VM

Our single VM lab connected many earlier concepts:

```text
Virtualization
→ VM runs on virtualized physical Azure infrastructure

IaaS / Shared Responsibility
→ Microsoft manages hardware; we manage Ubuntu/application

Region
→ VM deployed in West US 2

Availability
→ We intentionally accepted a single-instance lab architecture

Networking
→ NIC + private IP + public IP + NSG

Security
→ SSH key + allowed inbound ports

Storage
→ Persistent OS disk preserved files

Monitoring
→ CPU metric changed with workload activity

Scalability
→ VM can be resized; multiple VMs enable scale-out

Reliability
→ Production would require redundancy and resilient dependencies

Cost
→ Running compute costs money

Sustainability
→ Deallocate unused learning compute

Governance
→ VM lives inside subscription/resource group hierarchy
```

This is why Azure Virtual Machines are such a useful first Compute service: they make the cloud foundations tangible.

---

# 43. Reference links

Use current Microsoft documentation for product-specific behavior, supported VM sizes/features, pricing, and regional availability:

- Azure Virtual Machines overview: https://learn.microsoft.com/azure/virtual-machines/overview
- VM sizes: https://learn.microsoft.com/azure/virtual-machines/sizes/overview
- Linux VM SSH: https://learn.microsoft.com/azure/virtual-machines/linux/mac-create-ssh-keys
- VM states and billing: https://learn.microsoft.com/azure/virtual-machines/states-billing
- Azure Spot Virtual Machines: https://learn.microsoft.com/azure/virtual-machines/spot-vms
- Availability options for Azure VMs: https://learn.microsoft.com/azure/virtual-machines/availability
- Azure Monitor VM monitoring overview: https://learn.microsoft.com/azure/azure-monitor/vm/monitor-virtual-machine
- Network Security Groups: https://learn.microsoft.com/azure/virtual-network/network-security-groups-overview

---

# 44. What comes next

We have now documented the Azure VM concepts and lab we already completed.

The next Compute topics should follow our learning rule:

```text
Already learned → document
New topic       → learn interactively first → lab where useful → notes + Miro architecture
```

Before selecting the next service, we should distinguish the major Azure Compute choices and understand what problem each solves:

```text
Virtual Machines
Virtual Machine Scale Sets
App Service
Containers / Container Instances
AKS
Azure Functions
```

We should not assume they are interchangeable. Each removes or adds different operational responsibilities.