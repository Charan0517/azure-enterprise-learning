# Cloud Deployment Models, Service Models, and Shared Responsibility

This topic answers three different questions that are easy to mix together:

1. **Deployment model:** Where does the environment run, and who owns the infrastructure?
2. **Service model:** How much of the technology stack do we operate ourselves?
3. **Shared responsibility:** For the service model we choose, which security and operational duties belong to Microsoft and which remain ours?

These are related, but they solve different decisions. A company can use a **hybrid deployment model** while simultaneously using **IaaS, PaaS, and SaaS** for different parts of the same business system.

---

## 1. Why this topic matters in a real enterprise

Imagine a company needs a new customer portal. The portal eventually depends on much more than application code:

- physical datacenter space;
- physical servers;
- storage hardware;
- physical networking;
- virtualization;
- operating systems;
- application runtimes;
- web/application servers;
- application code;
- databases and data;
- identity and access;
- monitoring;
- backup and recovery;
- patching and security operations.

If the company builds the whole environment in its own datacenter, it owns almost every responsibility. If the company deploys the portal to an Azure VM, Microsoft takes over the physical infrastructure and virtualization layer, but the company still manages the guest operating system and application. If the portal runs on a managed platform such as Azure App Service, Microsoft manages more of the operating platform. If the business requirement can be satisfied by a complete SaaS product, the organization may not need to build the underlying application platform at all.

So the architecture question is not simply, **“Should we use cloud?”** A better question is:

> **Which parts of this capability create business value for us, and which parts are undifferentiated infrastructure work that a provider can manage?**

That is the reason deployment models and service models exist.

---

# Part A — Cloud Deployment Models

## 2. Public Cloud

In a public cloud, the cloud provider owns and operates large datacenters and offers computing services to many customers. Microsoft Azure is a public cloud.

“Public” does **not** mean that every customer's VM, database, or files are publicly visible. The word describes the provider model: Azure infrastructure is operated by Microsoft and offered as a service to many organizations. Customer environments are separated using virtualization boundaries, identity controls, network isolation, service-level isolation, encryption, authorization, and other security mechanisms.

### What the customer is really buying

When we create an Azure VM, we do not buy a physical server rack. We request a logical resource with characteristics such as region, VM size, image, disk configuration, and networking. Azure decides how to place that workload on the underlying infrastructure.

That changes the business model dramatically. Instead of waiting for procurement, installation, cabling, and hardware configuration, a team can provision infrastructure through the Azure Portal, CLI, PowerShell, APIs, or Infrastructure as Code.

### Why companies choose public cloud

Public cloud is useful when organizations need rapid provisioning, elastic capacity, global regions, managed services, automation, or consumption-based billing. It can also reduce the amount of hardware the organization must purchase and operate itself.

However, public cloud does not eliminate architecture work. Poorly designed public-cloud systems can still be insecure, unreliable, expensive, or difficult to operate.

### Example

Suppose a development team needs a temporary server for a proof of concept. In an on-premises environment, that request might require a hardware or virtualization ticket, IP assignment, storage allocation, operating-system installation, and firewall changes. In Azure, the team can create a VM in minutes if governance, quota, and permissions allow it.

That speed is one of cloud's major benefits, but it is also why governance matters: if everyone can create anything without standards, cost and security problems can appear just as quickly.

---

## 3. Private Cloud

A private cloud is a cloud-style environment dedicated to one organization. It may run in the organization's own datacenter or in dedicated hosted infrastructure.

A private cloud is **not simply “a company owns some servers.”** A true private-cloud environment normally adopts cloud characteristics such as resource pooling, virtualization, automation, standardized provisioning, self-service, and centralized management.

### Why would an organization still want private cloud?

Some workloads need unusually deep hardware control, specialized appliances, strict location constraints, or integration with existing datacenter systems. Some companies also have large investments in datacenter infrastructure that cannot be retired immediately.

Private cloud can provide more infrastructure control, but that control comes with responsibility. The organization must plan hardware capacity, refresh servers, operate virtualization, manage facilities or hosting contracts, and handle much more of the infrastructure lifecycle.

### Example

A manufacturing company might have a legacy control system tied to specialized equipment inside a factory. Moving that system directly into public cloud may create latency, connectivity, or hardware-integration problems. The organization may keep that system in a private environment while modernizing other applications in Azure.

---

## 4. Hybrid Cloud

Hybrid cloud combines on-premises or private-cloud infrastructure with public-cloud services as part of one overall architecture.

Hybrid is extremely common in enterprises because migrations are rarely all-or-nothing. Large organizations often have decades of applications, databases, identity systems, network dependencies, and regulatory requirements.

### A real hybrid scenario

Imagine a company has a legacy Oracle database on-premises that cannot yet be migrated. The company builds a new web/API layer in Azure. The Azure application connects to the on-premises database using secure network connectivity.

The application is now hybrid because the business service spans both environments.

The design must consider:

- connectivity between Azure and the datacenter;
- DNS resolution across both environments;
- latency between application and database;
- identity integration;
- firewall rules;
- monitoring across both environments;
- what happens when the private connection fails;
- whether DR depends on one side being available.

Hybrid therefore provides flexibility, but it can also be operationally more complex than a system that runs entirely in one environment.

### Common reasons for hybrid cloud

Hybrid is often used for gradual migration, legacy integration, regulatory/data-location constraints, disaster recovery, or because some systems are not technically or economically suitable for immediate cloud migration.

---

## 5. Multi-cloud

Multi-cloud means using services from more than one public cloud provider.

For example, a company may use Azure for most enterprise applications but inherit workloads in another cloud after an acquisition. Another organization may intentionally use a specialized service from a second provider.

Multi-cloud can be valid, but it introduces real complexity. Each provider has different identity models, networking, monitoring, policy systems, billing models, service names, deployment tools, and operational behaviors.

For that reason, multi-cloud should solve a real business or technical requirement. It should not be adopted only because “using more clouds sounds safer.” In some cases, spreading workloads across providers can actually make security and operations harder.

---

## 6. Deployment-model comparison

| Question | Public Cloud | Private Cloud | Hybrid Cloud |
|---|---|---|---|
| Who owns underlying infrastructure? | Cloud provider | Organization/dedicated provider | Both sides are involved |
| Upfront hardware investment | Usually lower | Often higher | Depends on retained on-prem footprint |
| Elastic capacity | Generally strong | Limited by dedicated capacity | Combination |
| Low-level infrastructure control | Lower | Highest | Mixed |
| Integration complexity | Moderate | Internal | Often highest because two environments must work together |
| Typical use | Modern cloud workloads, rapid provisioning | Specialized/control-heavy workloads | Enterprise migration and integration |

The table is a guide, not a rule. A company's real design may include all three patterns across different applications.

---

# Part B — Cloud Service Models

## 7. Deployment model vs service model

A deployment model answers **where/how the environment is hosted**.

A service model answers **how much of the technology stack the provider manages for us**.

The usual service-model progression is:

- On-premises
- IaaS — Infrastructure as a Service
- PaaS — Platform as a Service
- SaaS — Software as a Service

As we move from on-premises toward SaaS, the provider operates more of the stack and the customer performs less infrastructure management. At the same time, the customer normally has less low-level control.

---

## 8. Think in layers, not just product names

A traditional application depends on several layers:

| Layer | Example |
|---|---|
| Physical datacenter | Building, power, cooling |
| Physical networking | Switches, routers, cabling |
| Physical servers/storage | Compute and storage hardware |
| Virtualization | Hypervisor and host virtualization |
| Operating system | Ubuntu, Windows Server |
| Runtime/middleware | Java, .NET, Node.js, web server |
| Application | Business code |
| Data | Customer/business data |
| Identity/configuration | Users, permissions, secrets, application settings |

Someone must manage every layer. The service model determines where the responsibility boundary sits.

---

## 9. On-premises — maximum infrastructure responsibility

In a traditional on-premises environment, the organization manages nearly the whole stack.

If a physical host fails, the organization replaces or repairs it. If the hypervisor needs an upgrade, the organization owns that work. If the guest operating system needs patches, the organization patches it. If the application has a bug, the organization fixes it.

This provides maximum control, but it also means the company must maintain expertise and operational processes across hardware, virtualization, operating systems, networking, security, backup, and applications.

The question is not whether on-premises is “bad.” Some workloads genuinely need it. The important point is to understand the operational cost of owning every layer.

---

## 10. IaaS — Infrastructure as a Service

Azure Virtual Machines are the clearest IaaS example.

With IaaS, Microsoft manages the Azure datacenter, physical servers, physical network, and virtualization platform. We receive a virtual machine and manage the guest operating system and workload.

Our Ubuntu VM lab demonstrated this boundary perfectly. Azure created the VM, but Azure did not log into Ubuntu and install Nginx for us. We connected through SSH, installed Nginx, created the page, configured access, and maintained the guest environment.

### What control do we gain?

IaaS is useful when we need operating-system control, custom packages, specific runtimes, legacy software, or a migration path that closely resembles an existing server.

### What responsibility do we keep?

We must think about OS patching, hardening, guest firewall configuration, installed software, malware protection where appropriate, application deployment, monitoring, backup, secrets, and lifecycle management.

That is the trade-off:

> **IaaS gives us server-level control because it leaves us responsible for the server-level operating environment.**

---

## 11. PaaS — Platform as a Service

PaaS moves the responsibility boundary upward. Instead of receiving a server that we must administer, we receive a managed application or data platform.

Azure App Service is a useful conceptual example for web applications. We deploy application code and configure the application, while Azure manages much more of the operating system, platform, runtime hosting infrastructure, patching of the managed platform, and underlying servers.

### Why does PaaS exist?

Suppose a development team is building a customer API. The company earns money from the API's business functionality, not from manually patching Ubuntu or maintaining a web server. If the application fits a managed platform, PaaS can let the team spend more time on the application and less time on infrastructure operations.

### What do we still own?

PaaS does not mean Azure owns our application. We still own the application code, business logic, data, identity decisions, secrets, configuration, authorization behavior, and secure development practices.

If our API has a programming bug that lets one customer read another customer's account, Azure cannot solve that simply because the application runs on PaaS.

### PaaS trade-off

PaaS reduces infrastructure-management work, but it also imposes platform constraints. We cannot normally treat the underlying managed host as if it were our personal VM and make arbitrary operating-system changes.

So the decision is not “PaaS is always better.” The decision is whether the application's requirements fit the managed platform.

---

## 12. SaaS — Software as a Service

With SaaS, the provider delivers a complete application. The customer consumes and configures the software instead of building and operating the application stack.

Microsoft 365 is a familiar example. A company that needs business email usually does not create dozens of Azure VMs and build its own global mail platform. It subscribes to Microsoft 365 and focuses on users, licensing, access policies, data governance, security configuration, retention, and how employees use the service.

SaaS therefore removes a large amount of infrastructure and application-platform responsibility, but it does **not** remove customer responsibility completely.

The organization still decides:

- who receives accounts;
- who becomes an administrator;
- whether MFA and Conditional Access should be required;
- what data employees may store;
- how external sharing should work;
- how retention and governance should be configured.

---

## 13. Responsibility comparison

| Layer | On-Prem | IaaS | PaaS | SaaS |
|---|---|---|---|---|
| Physical datacenter | Customer | Microsoft | Microsoft | Microsoft/provider |
| Physical servers/network | Customer | Microsoft | Microsoft | Microsoft/provider |
| Virtualization | Customer | Microsoft | Microsoft | Provider |
| Guest OS | Customer | Customer | Provider manages platform OS | Provider |
| Runtime/middleware | Customer | Customer | Mostly provider-managed | Provider |
| Application code | Customer | Customer | Customer | Provider |
| Business data | Customer | Customer | Customer | Customer responsibility remains important |
| Identities/access/configuration | Customer | Customer | Customer | Customer responsibility remains important |

This table is intentionally conceptual. Every Azure service has its own exact responsibility boundary, so production architecture should verify the documentation for the service being used.

---

# Part C — Shared Responsibility Model

## 14. What “shared responsibility” actually means

Cloud security is not transferred completely to Microsoft. Instead, Microsoft and the customer each own different parts of the system.

The easiest way to understand this is to ask:

> **Who is in the best position to control this layer?**

Microsoft controls the Azure datacenter, physical hosts, and Azure platform. The customer controls its users, its data, its application logic, and many aspects of configuration. The exact dividing line moves depending on whether the workload is IaaS, PaaS, or SaaS.

---

## 15. What Microsoft always manages in Azure cloud services

Microsoft is responsible for the physical Azure cloud infrastructure: datacenter facilities, physical hosts, physical networking, and the provider-side platform components associated with the service.

If the physical server hosting our VM experiences a hardware problem, we do not travel to the Azure datacenter and replace the motherboard. That is Microsoft's responsibility.

This is one of the major operational benefits of public cloud.

---

## 16. What customers never completely give away

Even in SaaS, the customer remains responsible for important business decisions around data, identities, access, and configuration.

Azure cannot know automatically that Alice should be allowed to see payroll data while Bob should not. The organization must design and configure those permissions.

Likewise, Microsoft can protect the Azure platform, but it cannot prevent a developer from intentionally writing insecure business logic if the application allows it.

This is why “moving to cloud” does not remove the need for security architecture.

---

## 17. Shared responsibility in our Azure VM lab

Our Ubuntu VM was IaaS, so the boundary was very visible.

**Microsoft managed:**

- Azure datacenter;
- physical servers;
- physical network;
- virtualization/hypervisor platform;
- underlying cloud control infrastructure.

**We managed:**

- Ubuntu guest OS;
- SSH configuration;
- operating-system patches;
- Nginx installation;
- web content;
- application configuration;
- network rules we selected;
- data stored in the VM;
- whether the VM was securely operated.

If Ubuntu had an unpatched vulnerability because we ignored updates, that would not be fixed simply by saying “the VM is hosted in Azure.”

---

## 18. Shared responsibility in PaaS

With a managed web platform, Microsoft takes over more of the operating system and runtime hosting platform. That reduces our patching and server-administration burden.

But our responsibilities become more concentrated around the application itself:

- secure code;
- authorization;
- secrets and connection settings;
- data protection;
- identity configuration;
- network exposure choices;
- logging and monitoring;
- application lifecycle.

PaaS therefore changes **what** we manage; it does not eliminate management.

---

## 19. Shared responsibility in SaaS

With SaaS, the provider runs the complete application platform, but the organization still controls how its users consume the service.

For Microsoft 365, for example, the customer still makes decisions about user accounts, administrative roles, external sharing, data governance, authentication policy, and endpoint access.

A stolen administrator account can still cause serious damage even though the company never manages the underlying SaaS servers.

---

# Part D — How to Choose a Model

## 20. A practical decision process

Start from the business requirement, not from the Azure product name.

### Choose or retain IaaS when:

The workload needs guest-OS control, custom server software, legacy compatibility, low-level configuration, or a lift-and-shift migration path.

### Consider PaaS when:

The application can run within a managed platform and the team would rather spend time on application functionality than operating servers.

### Consider SaaS when:

A mature product already solves the business capability and building the capability ourselves provides little competitive value.

### Keep private/on-premises components when:

The workload has hardware, regulatory, latency, or legacy requirements that make immediate public-cloud migration impractical.

### Use hybrid when:

The complete business service needs both Azure and on-premises/private systems.

---

## 21. Migration scenario: from legacy server to modern cloud

Imagine a company has a Java application running on an old on-premises VM.

**Stage 1 — Rehost:** Move the application to an Azure VM with minimal code changes. This reduces physical-infrastructure responsibility but still leaves the team managing the OS and application server.

**Stage 2 — Modernize:** Refactor parts of the application to run on managed Azure application/database services. This reduces operating-system and platform-management work.

**Stage 3 — Replace where appropriate:** If some capabilities are commodity functions already solved by SaaS, stop maintaining custom software for those capabilities.

This progression shows why cloud migration is not simply “move every server to Azure.” The larger goal is to choose the correct responsibility boundary for each workload.

---

## 22. Cost is not a simple IaaS < PaaS < SaaS formula

It is incorrect to assume that one service model is always cheaper.

An Azure VM may have a lower visible monthly resource price than a managed PaaS service, but the VM also requires engineering time for patching, monitoring, backup, hardening, upgrades, and troubleshooting.

The correct comparison is **total cost of ownership**, including operational effort, licensing, reliability, scaling, security, and support—not only the price displayed next to one Azure resource.

---

## 23. Security is not automatically solved by choosing PaaS or SaaS

Managed services remove some infrastructure responsibilities and can reduce certain operational risks, but insecure configuration can still expose a workload.

Examples include:

- public network access that should have been restricted;
- overly broad identities and permissions;
- secrets stored insecurely;
- vulnerable application code;
- missing logging;
- weak authentication policies.

The service model changes the security boundary. It does not eliminate the need for secure design.

---

# Part E — Common Misunderstandings

## “Public cloud means my data is public.”

No. Public cloud refers to the provider model. Workloads still use isolation, authentication, authorization, and network controls.

## “Private cloud means a few virtual machines in our datacenter.”

Not necessarily. Private cloud normally implies cloud-style automation, pooling, standardized provisioning, and management in addition to dedicated infrastructure.

## “Hybrid cloud means using two Azure regions.”

No. Two Azure regions are still public-cloud architecture. Hybrid means the solution spans public cloud and private/on-premises infrastructure.

## “Multi-cloud and hybrid cloud are the same.”

No. Multi-cloud uses more than one public-cloud provider. Hybrid combines public cloud with private/on-premises infrastructure.

## “Azure VM is PaaS because Microsoft owns the hardware.”

No. Azure VM is IaaS because the customer still manages the guest operating system and workload.

## “PaaS means Microsoft manages our application.”

No. Microsoft manages more of the platform. We still own our code, data, access model, and application configuration.

## “SaaS means we have no security responsibilities.”

No. Identity, data, access, configuration, and governance responsibilities remain with the customer.

---

# 24. Architecture companion

The corresponding architecture diagram is maintained on the Azure learning Miro board. The Miro diagram is the visual source for the stack/responsibility relationships; this Markdown file intentionally focuses on explanation instead of trying to recreate a large architecture diagram with ASCII art.

Miro board: https://miro.com/app/board/uXjVH3UtYkg=/

---

# 25. Official references

Use current Microsoft documentation when making real architecture decisions because service capabilities and responsibility boundaries evolve.

- Microsoft Learn — Describe cloud service types: https://learn.microsoft.com/training/modules/describe-cloud-service-types/
- Microsoft Learn — Shared responsibility in the cloud: https://learn.microsoft.com/azure/security/fundamentals/shared-responsibility
- Microsoft Learn — What is Azure App Service?: https://learn.microsoft.com/azure/app-service/overview
- Microsoft Learn — Azure Virtual Machines overview: https://learn.microsoft.com/azure/virtual-machines/overview

---

# 26. Final mental model

Remember these three questions:

**Deployment model:** Where/how is the environment hosted?

**Service model:** How much of the stack do we manage?

**Shared responsibility:** For the service we chose, who is accountable for each security and operational layer?

A mature enterprise can use public cloud, private infrastructure, hybrid connectivity, IaaS, PaaS, and SaaS at the same time. The correct architecture is the combination that satisfies the business requirement with the right balance of control, operational effort, security, reliability, and cost.