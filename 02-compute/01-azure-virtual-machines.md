# Azure Virtual Machines (IaaS) — From Physical Server to Our Ubuntu/Nginx Workload

Azure Virtual Machines are our first major Azure Compute service because a VM makes many cloud concepts visible at once: virtualization, IaaS, regions, networking, storage, security, monitoring, scaling, availability, and cost.

We did not only create a VM in the portal. We followed the VM through its lifecycle: selected a region and size, created an Ubuntu guest, authenticated with SSH, inspected CPU/memory/networking, installed Nginx, opened HTTP connectivity, edited a web page, restarted the VM, verified that files persisted, watched CPU metrics, and finally deallocated the VM to stop unnecessary compute charges.

This note explains what was happening underneath each step and how the same concepts change in a production enterprise environment.

---

## 1. The real-world problem Azure VMs solve

Before virtualization and public cloud, a team needing a new server might have to estimate hardware requirements, request budget, purchase hardware, wait for delivery, rack the server, cable networking, install an operating system, allocate storage, configure firewalls, and then install the application.

That process could take days or weeks even before developers received a usable machine.

Virtualization improved utilization by allowing many isolated virtual machines to share a physical host. Public cloud took the model further: Microsoft operates enormous pools of physical infrastructure and exposes virtual compute as an on-demand service.

When we request an Azure VM, we are essentially saying:

> “Give this workload a virtual machine with this operating-system image, CPU/memory capacity, disks, networking, security configuration, and regional placement.”

Azure then provisions the required logical resources on Microsoft's infrastructure.

---

# Part A — What an Azure VM Actually Is

## 2. Azure VM is Infrastructure as a Service

Azure VM is IaaS because Microsoft manages the physical datacenter, hosts, physical networking, and virtualization platform, while we manage the guest operating system and workload.

That responsibility boundary explains our lab. Azure gave us an Ubuntu VM, but it did not automatically know that we wanted Nginx. We logged in, installed Nginx, edited the HTML page, and decided which ports to expose.

For an IaaS VM, our responsibilities can include:

- guest OS configuration and patching;
- installed packages and runtimes;
- application deployment;
- guest-level security/hardening;
- data and secrets;
- monitoring and logging;
- backup/recovery configuration;
- access management;
- deciding when and how to scale.

VMs provide control, but the reason we have that control is that Azure has intentionally left those layers for us to manage.

---

## 3. From physical host to vCPU and virtual memory

Underneath our VM is physical Azure hardware. A hypervisor/virtualization platform allows physical compute resources to be presented to isolated virtual machines.

If our VM size provides two vCPUs and a certain amount of memory, Ubuntu sees those virtual resources as the machine available to it. The guest OS does not need to know which exact physical CPU socket or memory module is underneath.

This is the abstraction created by virtualization:

- physical CPU capacity becomes virtual CPU capacity;
- physical memory is allocated/presented as VM memory;
- physical networking is exposed through virtual network interfaces;
- storage is exposed through virtual disks.

This is why the Linux commands we ran inside the VM showed CPU and memory values matching the virtual machine configuration rather than the entire physical Azure host.

---

## 4. A VM is not one isolated Azure object

The VM resource is only one part of the working solution. A typical Azure VM depends on several supporting resources.

Our VM needed a network interface. The NIC existed in a subnet inside a virtual network. The NIC had a private IP. We also used a public IP so our laptop could reach the VM over the Internet. Network Security Group rules controlled permitted inbound traffic. The operating system lived on a managed OS disk.

That distinction matters operationally because each supporting resource has its own configuration and lifecycle. Deleting or deallocating the VM does not automatically mean every related resource disappears or stops costing money.

This is why Azure architecture should be thought of as a collection of connected resources rather than “one VM box.”

---

# Part B — Decisions We Made in the Create VM Screen

## 5. Subscription and Resource Group

Every Azure resource belongs to a subscription, and resources are organized into resource groups.

The subscription is an important billing, quota, management, and governance boundary. The resource group is a logical lifecycle/management container for related resources.

For a learning environment, we can place related lab resources together so they are easy to find and delete later. In an enterprise, resource-group design is usually based on lifecycle, ownership, environment, deployment boundaries, and governance rather than simply putting everything into one giant group.

---

## 6. Region — why we selected West US 2

We selected **West US 2** in our lab. That told Azure where to deploy the VM geographically.

For our learning workload, the region did not need to satisfy a production SLA. In a real system, region selection would consider user latency, data residency, service/SKU availability, Availability Zone support, quota/capacity, price, and DR strategy.

A VM size that exists in Azure globally may still be unavailable to our subscription in a particular region. That is why we saw portal messages indicating that certain sizes were not currently available even though the sizes appeared in the selection list.

---

## 7. Availability options — what those portal choices actually mean

During VM creation, Azure displayed availability choices such as no infrastructure redundancy required, Availability Zone, Availability Set, and Virtual Machine Scale Set-related options.

These choices are not simply different names for the same thing.

### No infrastructure redundancy required

This was reasonable for our temporary lab. We accepted that one VM was one failure point because the workload had no business SLA.

For a customer-facing production application, one VM would usually be inadequate if downtime matters.

### Availability Zone

A VM can be placed in a specific zone in a region that supports zones. But one VM in one zone is still one instance. To tolerate a zone failure, the architecture normally needs additional instances or zone-redundant services across other zones.

### Availability Set

Availability Sets are a VM availability mechanism that helps distribute VMs across fault/update domains within the relevant infrastructure scope. They are useful to understand historically and for applicable workloads, but zone-based designs are important for modern regional resiliency where supported.

### Virtual Machine Scale Set

A VM Scale Set manages a group of VM instances and is designed for scenarios where we need multiple similar instances, scaling, and fleet-style management. It is not simply “a bigger VM.” We will learn VMSS separately before documenting it.

---

## 8. Security type — Standard, Trusted Launch, and Confidential VM

The portal offered multiple VM security types. These options relate to different threat models and hardware/platform capabilities.

**Standard** represents the traditional supported VM security configuration.

**Trusted Launch** is designed to improve protection of the VM boot and integrity chain using supported technologies such as Secure Boot and virtual TPM. The goal is to make it harder for low-level boot malware/rootkits to compromise the VM before the operating system is fully running.

**Confidential VMs** are designed for scenarios requiring stronger protection of data while it is being processed in memory, using supported confidential-computing hardware and technologies.

The important architecture lesson is not “always select the most advanced option.” Image compatibility, VM size, regional support, workload requirements, cost, and security requirements must all be considered.

---

## 9. Image — where Ubuntu came from

A VM image is the template used to create the guest operating-system environment. We selected Ubuntu, so Azure provisioned an OS disk based on the selected Ubuntu image.

Other images include Windows Server, Debian, Red Hat, SUSE, Oracle Linux, and many marketplace/custom images.

The image answers questions such as:

- Which operating system starts on first boot?
- Which base packages/configuration are present?
- Which processor architecture is supported?

The image is not the VM size. Image determines the software/OS starting point; size determines the virtual hardware capacity.

---

## 10. x64 vs Arm64 architecture

Processor architecture affects which VM families and software binaries can run. x64 is widely supported across enterprise software. Arm64-based Azure VM families can provide attractive performance/cost characteristics for compatible workloads.

An application compiled only for x64 cannot simply be assumed to run unchanged on Arm64. OS image, dependencies, native libraries, containers, and application binaries must support the chosen architecture.

For our basic Ubuntu learning VM, x64 kept compatibility simple.

---

## 11. VM size — much more than “CPU and RAM”

The VM size determines a package of compute characteristics. Depending on the VM family/SKU, that can include:

- vCPU count;
- memory;
- maximum data-disk count;
- storage throughput/IOPS limits;
- network bandwidth characteristics;
- local/temporary storage characteristics;
- GPU or specialized accelerators on applicable families;
- processor architecture/generation;
- price.

This means two VMs with the same number of vCPUs are not necessarily equivalent. One family may be optimized for memory, another for compute, another for storage, and another for burstable low-average-CPU workloads.

Production sizing should start from workload behavior, not from whichever SKU is cheapest in the portal.

---

## 12. Why B-series made sense for our lab

B-series/burstable VM families are designed for workloads that usually consume a low baseline of CPU but occasionally need higher CPU performance.

Our Nginx learning page spent most of its time idle. It did not need sustained high CPU. That makes a low-cost burstable VM conceptually suitable for a lab or small low-utilization workload.

A continuously CPU-intensive production workload may be a poor fit because burstable families are not intended to provide unlimited sustained burst performance. Exact credit behavior varies by B-series generation, so the current SKU documentation must be checked for real deployments.

---

## 13. “Free services eligible” vs “available to my subscription”

We encountered an important Azure portal lesson: a VM size can be marked as eligible under a free-service offer yet still be unavailable for our subscription/region.

Those statements answer different questions.

**Free-services eligible** describes pricing/offer eligibility under applicable terms.

**Available** depends on regional capacity, subscription type, quota, policy, VM family availability, and other restrictions.

Therefore an architecture cannot assume that because a SKU appears in documentation, every subscription can deploy it immediately in every region.

---

## 14. Azure Spot — cheaper capacity with an interruption trade-off

Azure Spot VMs use spare Azure compute capacity at a discount, but Azure can evict the VM when capacity is needed or when configured price/capacity conditions apply.

Spot therefore works best when the workload can tolerate interruption: batch processing, stateless workers, dev/test, distributed jobs, or tasks that can restart elsewhere.

A single critical production database server that cannot tolerate sudden interruption is generally not a sensible Spot workload.

The lower price exists because the customer accepts reduced capacity guarantees.

---

# Part C — Authentication and Networking

## 15. Why we used SSH keys

SSH provides encrypted remote administration for Linux systems. Instead of relying only on a password, SSH public-key authentication uses a cryptographic key pair.

The **public key** can be placed on the VM. The **private key** stays with the administrator and must be protected.

When we connect, the SSH protocol allows the client to prove possession of the private key without sending that private key to the server.

This is why committing a private SSH key to GitHub would be a serious security mistake.

---

## 16. What happened when we ran SSH

When we used a command like `ssh username@PUBLIC_IP`, several layers had to work:

1. our laptop needed Internet connectivity;
2. the VM needed a reachable public endpoint or another network path;
3. routing had to deliver traffic to Azure;
4. the NSG had to permit TCP port 22 from the allowed source;
5. Ubuntu's SSH service had to be running/listening;
6. authentication had to succeed.

If any one of these failed, SSH would fail.

This is the beginning of layered troubleshooting: determine **which layer broke** instead of randomly changing configuration.

---

## 17. Private IP

The VM's NIC had a private IP in the Azure virtual network. A private IP is used for communication inside private/connected network scopes and is not directly Internet-routable.

In a production architecture, application VMs often communicate with databases and other internal services using private IPs rather than sending internal traffic through public Internet endpoints.

The private IP belongs to the NIC/IP configuration, not directly to “the Ubuntu application.”

---

## 18. Public IP

We used a public IP because our laptop needed a simple Internet path to the learning VM.

A public IP does not automatically make every VM port accessible. It provides an Internet-routable endpoint, but NSG rules, routing, guest firewall configuration, and whether a service is actually listening still determine connectivity.

In production, directly assigning public IPs to application VMs is often avoided where possible. Organizations may instead use bastion/jump access, load balancers, application gateways, private endpoints, VPN/ExpressRoute, or other controlled access patterns depending on requirements.

---

## 19. Network Interface (NIC)

The virtual NIC connects the VM to an Azure subnet. It carries IP configuration and participates in Azure network/security behavior.

This separation matters because the VM's compute and network identity are different resource concepts. Networking can be managed and reasoned about independently from the guest operating system.

We will go much deeper into VNet, subnet, routing, NSG, DNS, and private connectivity in the Networking module.

---

## 20. Network Security Group — opening a port is only one layer

An NSG evaluates network rules based on properties such as source, destination, protocol, port, direction, and priority.

For our lab:

- SSH used TCP 22;
- HTTP used TCP 80.

When we wanted to view the Nginx page from our browser, allowing TCP 80 in Azure was necessary—but it was not sufficient by itself.

Nginx also had to be installed and running, the VM had to be reachable, and the application had to return valid content.

This gives us a useful troubleshooting distinction:

> **Network access means traffic can reach the service. Application health means the service can correctly process the request.**

They are related but not the same.

---

# Part D — What We Did Inside Ubuntu

## 21. Verifying virtual hardware from the guest OS

After SSH login, we used Linux commands to inspect CPU, memory, and network information.

Commands such as `nproc`, `free -h`, and `ip addr` show what the guest operating system sees.

This connected the Azure Portal configuration to the virtualization concept. Azure showed us the selected VM size; Ubuntu showed us the virtual hardware presented to the guest.

If the portal says a VM has a particular CPU/memory configuration but the application performs poorly, those guest-level tools become part of troubleshooting.

---

## 22. Installing Nginx changed the VM's role

Before Nginx, the VM was essentially a general-purpose Ubuntu server. After installing Nginx, it became capable of serving HTTP content.

On Ubuntu, package-management commands downloaded and installed the Nginx software. The Nginx service then listened for web requests, normally on TCP 80 for HTTP unless configured differently.

This demonstrates why IaaS is flexible: Azure did not dictate that the VM must be a web server. We decided what software to install.

The same flexibility also means we are responsible for maintaining that software.

---

## 23. Why the web page initially failed when our HTML was wrong

At one point the page did not look/work as expected because the page content had incorrect syntax. The Azure VM itself was still running.

That illustrates a critical operational concept:

**Infrastructure health and application health are different.**

A portal status of **Running** means the Azure VM compute resource is running. It does not prove that:

- Nginx is running;
- port 80 is reachable;
- HTML is valid;
- application code is correct;
- database connectivity works;
- the business transaction succeeds.

Production monitoring therefore needs both infrastructure metrics and application-level health checks.

---

## 24. End-to-end HTTP request path

When we entered the VM's public IP in the browser, the request crossed several layers:

**Browser → Internet → public IP → Azure networking/NSG → NIC/private networking → Ubuntu → Nginx → HTML file → HTTP response back to browser.**

If the page does not load, we can troubleshoot from outside inward:

1. Is the VM running?
2. Is the public IP correct?
3. Is TCP 80 allowed by the NSG?
4. Is the guest firewall blocking it?
5. Is Nginx running?
6. Is Nginx listening on the expected port?
7. Does the requested file exist?
8. Is the application/content valid?

This layered method is far more reliable than changing random settings.

---

# Part E — Persistence and VM Lifecycle

## 25. Why our files survived restart

CPU and RAM represent active runtime state. Persistent files live on disk.

When the VM restarts, running processes stop and memory state is lost. The OS disk remains attached and preserves its data, so Ubuntu, Nginx installation files, configuration, and our HTML file remain available after boot.

If Nginx is configured to start automatically, the service starts again and reads the same persistent files.

This is why a restart did not recreate our VM from scratch.

---

## 26. Persistent disk vs temporary/local disk

Azure VMs can have persistent managed disks and, depending on VM size, temporary/local storage.

Persistent OS/data disks are designed to retain data through normal VM stop/restart/deallocation lifecycle operations.

Temporary/local storage is tied more closely to the physical host lifecycle and must not be used for data that must survive host movement or other lifecycle events.

A production application must know which data is disposable and which data requires durable storage.

We will cover managed disks and Azure Storage more deeply later.

---

## 27. Restart, Stop, and Deallocate

These terms affect both behavior and billing.

### Restart

The guest reboots, but Azure still has compute allocated to the VM. The OS disk persists and compute billing continues.

### Stopped but still allocated

Depending on how the VM is shut down/state reported, the guest can be stopped while Azure compute remains allocated. If it remains allocated, compute charges can continue.

### Stopped (deallocated)

Azure releases the VM's compute allocation. The VM configuration and persistent disks remain so the VM can be started again later.

Compute charges for the VM instance stop while deallocated, but other resources can continue generating charges.

This is why we explicitly verified **Stopped (deallocated)** before taking a break.

---

## 28. Why deallocated does not mean $0

The VM is an architecture composed of resources. Deallocation releases compute, but managed disks still store data. Public IP resources, backup, snapshots, monitoring retention, or other attached services can have separate charges depending on configuration.

Therefore cost troubleshooting should look at the resource group/subscription cost breakdown, not only the VM power state.

For temporary labs, the safest cleanup after learning is often to delete resources that are no longer needed—not merely leave them deallocated forever.

---

# Part F — Monitoring and Performance

## 29. Why CPU percentage increased

When the VM performs work, its virtual CPUs consume processing time. Azure exposes metrics such as CPU percentage so we can observe utilization.

We saw CPU change as the VM performed activity. That showed the relationship between workload demand and infrastructure telemetry.

But one CPU chart does not tell the whole story. A slow application can be constrained by memory, disk latency/IOPS, network throughput, database performance, locks, external APIs, or inefficient code even when CPU is low.

Monitoring therefore needs to be driven by the suspected bottleneck and application behavior.

---

## 30. Metrics vs logs

Metrics are numerical time-series measurements such as CPU percentage. Logs contain richer event/diagnostic information.

A useful troubleshooting pattern is:

- metrics tell us **something changed**;
- logs often help explain **what happened and why**;
- application traces can show **which request/component was slow or failed**.

We will study Azure Monitor in its own module, but our VM lab gave us the first practical example.

---

# Part G — Scaling and Availability

## 31. Vertical scaling — resize the VM

If the workload needs more CPU or memory, we can change to a larger supported VM size. This is scale up.

If the VM is consistently underutilized, we can potentially move to a smaller size to reduce cost. This is scale down.

Resizing is simple conceptually but can involve restart/deallocation and capacity constraints. A larger VM also does not solve every performance problem. If the bottleneck is an inefficient SQL query, simply adding CPU to the web server may do nothing.

---

## 32. Horizontal scaling — add instances

Instead of making one VM larger, we can run multiple application instances. A load-balancing layer distributes traffic among healthy instances.

This can increase capacity and improve availability because the application no longer depends on one VM.

However, horizontal scaling requires the application to support multiple instances. Local session state, local files, background jobs, database connections, and concurrency behavior all need to be considered.

VM Scale Sets help manage groups of VM instances, but we will learn that topic separately before creating its documentation.

---

## 33. Why one VM is a single point of failure

Even if Azure's physical infrastructure is highly engineered, one application instance can fail because of:

- guest OS crash;
- bad patch;
- application crash;
- configuration mistake;
- VM-level issue;
- underlying infrastructure event;
- accidental administrative action.

If all users depend on that one VM, the application is unavailable when that VM is unavailable.

Production High Availability therefore usually requires redundant application instances and resilient dependencies.

---

## 34. Availability Zones and VMs

For a zone-resilient application, VMs can be placed across different Availability Zones where supported.

But simply putting VM1 in Zone 1 and VM2 in Zone 2 is not enough. Traffic routing/load balancing, database, storage, and other dependencies must also survive the failure.

This is why we documented Azure Global Infrastructure before moving deeper into Compute: VM placement only makes sense when we understand failure boundaries.

---

# Part H — Disaster Recovery

## 35. High Availability is not Disaster Recovery

Multiple VMs across zones help with failures inside one region. If the entire region is unavailable, all those zones may be inaccessible.

Regional DR requires another region and a recovery strategy for application infrastructure and data.

For VMs, this may involve service-specific replication/recovery technologies, backups, Infrastructure as Code, image/configuration management, replicated data, DNS/global traffic failover, and documented recovery procedures.

The exact solution depends on RTO and RPO.

---

## 36. Backup and replication solve different problems

Replication keeps another copy relatively current and can support faster failover, but corruption or accidental deletion may also replicate depending on the technology.

Backups provide historical recovery points and are important for restoring from corruption, deletion, ransomware, or other logical failures.

A mature design often uses both rather than treating one as a complete replacement for the other.

---

# Part I — Security and Enterprise Operations

## 37. Why directly exposing SSH is acceptable for a lab but questionable for production

For our learning exercise, a public IP and controlled SSH rule made it easy to understand the connection path.

In a production enterprise, direct Internet SSH/RDP exposure increases attack surface. Teams may instead use Azure Bastion, private connectivity, VPN/ExpressRoute, jump hosts, just-in-time access, centralized identity, or other controlled administration patterns.

The correct design depends on requirements, but the general principle is to expose only what must be exposed.

---

## 38. OS patching is our responsibility in IaaS

Because Ubuntu is our guest operating system, we must maintain it. Over time, vulnerabilities are discovered in the OS and installed packages.

Production operations need a patch strategy that includes testing, maintenance windows, automation, rollback planning, and availability considerations.

Running a VM for years without patching it is not made safe simply because the physical server belongs to Microsoft.

---

## 39. Secrets should not live casually on the VM

Applications often need database passwords, API keys, certificates, or tokens. Hard-coding these into scripts or committing them to GitHub is dangerous.

Enterprise Azure architectures commonly use managed identities and services such as Azure Key Vault so applications can authenticate securely without distributing long-lived credentials unnecessarily.

We will cover identity/secrets later, but the VM should already be understood as part of that security model.

---

## 40. Backup, monitoring, and lifecycle are part of owning a VM

Creating a VM is the easy part. Operating one responsibly includes:

- patching;
- vulnerability management;
- access control;
- disk capacity monitoring;
- application monitoring;
- log collection;
- backups;
- recovery testing;
- certificate/secret management;
- cost optimization;
- configuration management;
- decommissioning.

This operational burden is a major reason organizations choose PaaS when they do not actually need OS-level control.

---

# Part J — When Should We Use a VM?

## 41. Good VM use cases

Azure VMs are a strong fit when the workload needs server-level control: custom operating-system configuration, legacy applications, specialized agents/packages, software that expects a traditional server, or a lift-and-shift migration where major application changes are not yet possible.

They are also useful for learning because the operating system is visible and we can see how compute, networking, storage, and security connect.

---

## 42. When a VM may be unnecessary

Suppose the requirement is simply to host a modern stateless web API. If the application fits Azure App Service or another managed platform, maintaining an entire VM may add patching and administration work that provides no business value.

The architecture question should therefore be:

> **Does this workload actually require control of the operating system?**

If yes, IaaS may be appropriate. If no, we should evaluate PaaS, containers, or serverless options rather than defaulting to VMs because they are familiar.

---

# Part K — Our Lab as an End-to-End Story

## 43. What happened from beginning to end

We selected our subscription and resource group, chose West US 2, reviewed availability/security/image/architecture options, selected an affordable supported B-series VM, configured Linux administrator access, and deployed Ubuntu.

Azure created the VM and its supporting compute/storage/network resources. Once the VM was running, we observed both private and public IP information. We connected through SSH using the public path and authenticated to Ubuntu.

Inside the guest, we inspected CPU, memory, and network information. We installed Nginx and started using the VM as a web server. To reach Nginx from our browser, the network path had to allow HTTP traffic. We then edited the page content and fixed an HTML syntax problem when the page was not displaying as intended.

We restarted the VM and confirmed that Nginx/files still existed because the OS disk was persistent. We watched CPU metrics change as the VM did work. Finally, when we were taking a break, we stopped the VM through Azure and verified that its state became **Stopped (deallocated)** so the compute allocation was released.

That single lab demonstrated the full relationship among virtualization, IaaS, networking, persistent storage, application configuration, monitoring, and cloud cost management.

---

# Part L — Common Misunderstandings

## “VM status Running means the website is healthy.”

No. It only confirms the Azure VM resource is running. Nginx or the application can still be broken.

## “Public IP means every port is open.”

No. NSGs, routing, guest firewall rules, and listening services still control connectivity.

## “The VM's private IP is only usable inside the VM.”

No. It is used for private communication across permitted connected network scopes.

## “Restarting deletes my installed software.”

No. Software/files on persistent disk remain through normal restart.

## “Deallocated means the entire resource group costs $0.”

No. Disks and other resources can still incur charges.

## “Two vCPUs means two dedicated physical CPU cores belong only to me.”

Not necessarily. vCPU is virtual compute capacity presented by Azure according to the VM SKU/virtualization platform.

## “One VM in an Availability Zone is highly available.”

No. It remains one application instance in one zone.

## “A larger VM always fixes slowness.”

No. The bottleneck may be storage, database, network, application code, or another dependency.

## “Azure patches Ubuntu because Azure owns the physical server.”

Not automatically. Guest OS maintenance is a customer responsibility in the IaaS model, though Azure provides management tooling that can help automate it.

---

# 44. Architecture companion

The Azure learning Miro board contains the visual architecture for our VM lab: local user → Internet → public IP → NSG → NIC/VNet → Ubuntu VM → persistent disk, with Azure Monitor and the underlying Microsoft-managed virtualization layer shown separately.

The Miro diagram should be used as the visual architecture. This note focuses on explaining why every component exists and what happens when requests move through the system.

Miro board: https://miro.com/app/board/uXjVH3UtYkg=/

---

# 45. Official Microsoft references

Product behavior, VM families, prices, and regional capabilities change, so use current Microsoft documentation when implementing a real environment.

- Azure Virtual Machines overview: https://learn.microsoft.com/azure/virtual-machines/overview
- Azure VM sizes overview: https://learn.microsoft.com/azure/virtual-machines/sizes/overview
- Linux SSH keys: https://learn.microsoft.com/azure/virtual-machines/linux/mac-create-ssh-keys
- VM states and billing: https://learn.microsoft.com/azure/virtual-machines/states-billing
- Azure Spot Virtual Machines: https://learn.microsoft.com/azure/virtual-machines/spot-vms
- Availability options for Azure VMs: https://learn.microsoft.com/azure/virtual-machines/availability
- Trusted Launch: https://learn.microsoft.com/azure/virtual-machines/trusted-launch
- Confidential VMs: https://learn.microsoft.com/azure/confidential-computing/confidential-vm-overview
- Network Security Groups: https://learn.microsoft.com/azure/virtual-network/network-security-groups-overview
- Monitor Azure Virtual Machines: https://learn.microsoft.com/azure/azure-monitor/vm/monitor-virtual-machine

---

# 46. Final mental model

An Azure VM is not “a server somewhere on the Internet.” It is an IaaS compute resource placed in an Azure region, running on Microsoft-managed physical infrastructure through virtualization, connected to Azure networking through a NIC, using persistent storage, and operated by us at the guest OS/application layer.

The most important question when considering a VM is not **“Can Azure create one?”** Azure can create one very easily.

The important question is:

> **Does the workload require the control that a VM gives us, and are we prepared to own the operating, security, availability, backup, monitoring, and cost responsibilities that come with that control?**