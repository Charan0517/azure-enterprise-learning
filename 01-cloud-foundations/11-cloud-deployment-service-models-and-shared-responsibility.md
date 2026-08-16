# Cloud Deployment Models, Service Models, and Shared Responsibility

Before learning individual Azure services, we need to answer three different architecture questions:

1. **Where does the infrastructure run and who owns it?** → Deployment model
2. **How much of the technology stack does the cloud provider manage for us?** → Service model
3. **Which security and operational responsibilities belong to Microsoft and which belong to us?** → Shared Responsibility Model

These concepts are related, but they are not the same thing.

---

# 1. Real-world problem

Imagine a company wants to launch an employee application.

It needs:

```text
Physical servers
Networking
Storage
Operating system
Runtime
Application code
Database/data
Identity and security
Monitoring
Backups
```

The company has several choices.

It could buy servers and run everything itself. It could rent virtual machines from Azure. It could deploy the application to a managed application platform. Or it could simply subscribe to a complete software product.

The business question becomes:

> How much infrastructure and operational responsibility do we actually want to own?

That question leads to deployment models, service models, and shared responsibility.

---

# 2. Cloud deployment models

A deployment model describes **where cloud infrastructure is operated and how it relates to the organization**.

The main models are:

```text
Public Cloud
Private Cloud
Hybrid Cloud
```

Multi-cloud is also an important architecture strategy, although it describes using multiple cloud providers rather than being one of the classic three deployment models.

---

# 3. Public Cloud

In a public cloud, a cloud provider owns and operates the underlying datacenter infrastructure and provides cloud services to many customers.

Examples of major public cloud providers include Microsoft Azure, AWS, and Google Cloud.

Conceptually:

```text
Microsoft Azure Datacenter
        │
        ├── Customer A workloads
        ├── Customer B workloads
        ├── Customer C workloads
        └── Customer D workloads
```

Customers do **not** all share the same operating system or application environment. Azure uses virtualization, identity controls, network isolation, service boundaries, and other mechanisms to isolate workloads.

This connects directly to our virtualization notes.

## Why companies use public cloud

```text
No need to purchase datacenter hardware upfront
Rapid provisioning
Global regions
Elastic capacity
Managed services
Consumption-based pricing options
Large service catalog
```

## Example

Instead of purchasing a physical server for a new application:

```text
Engineer → Azure Portal/API/IaC → Create VM → VM available in minutes
```

The physical server underneath remains Microsoft's responsibility.

---

# 4. Private Cloud

A private cloud is cloud-style infrastructure dedicated to a single organization.

The infrastructure may exist in the organization's own datacenter or be hosted by another provider, depending on the implementation.

Conceptually:

```text
Company Datacenter / Dedicated Environment
        │
        ├── Virtualization platform
        ├── Internal automation
        ├── Self-service provisioning
        └── Company workloads only
```

A private cloud is more than simply owning several servers.

Cloud characteristics such as automation, pooling, self-service, and standardized provisioning are important.

## Why use private cloud?

Possible reasons include:

```text
Special regulatory requirements
Legacy hardware/software dependencies
Very specific infrastructure control
Data/location requirements
Existing datacenter investments
Specialized workloads
```

But the organization usually retains significantly more infrastructure-management responsibility.

---

# 5. Hybrid Cloud

Hybrid cloud combines private/on-premises infrastructure with public cloud services and connects them as part of an overall architecture.

Example:

```text
Company Datacenter
      │
      │ Private connectivity / VPN
      ▼
Microsoft Azure
```

A company might keep a legacy database on-premises while deploying a new web application in Azure.

```text
Users
  ↓
Azure Web/Application Tier
  ↓
Secure connectivity
  ↓
On-Premises Database
```

Hybrid cloud is extremely common because enterprises rarely move every system to cloud at once.

## Reasons for hybrid architecture

```text
Gradual cloud migration
Legacy dependencies
Regulatory/data-location requirements
Datacenter investments
Disaster recovery
Integration with on-prem systems
```

Hybrid does introduce additional complexity around networking, identity, monitoring, security, latency, and operations.

---

# 6. Multi-cloud

Multi-cloud means intentionally using services from more than one cloud provider.

Example:

```text
Company
│
├── Microsoft Azure
│
└── Another cloud provider
```

Reasons may include acquisitions, business requirements, specialized services, customer requirements, geographic availability, or reducing certain provider dependencies.

However, multi-cloud can increase complexity:

```text
Different IAM models
Different networking
Different monitoring
Different policy systems
Different billing
Different service behavior
Different engineering skills
```

Therefore:

> Multi-cloud should solve a real requirement rather than being adopted simply because using multiple clouds sounds safer.

---

# 7. Deployment-model comparison

| Question | Public Cloud | Private Cloud | Hybrid Cloud |
|---|---|---|---|
| Underlying infrastructure | Cloud provider | Dedicated to organization | Combination |
| Upfront hardware need | Usually low | Often higher | Depends |
| Elasticity | Strong | Limited by owned/dedicated capacity | Combination |
| Infrastructure control | Less physical control | Highest | Mixed |
| Operational complexity | Provider manages more infrastructure | Organization manages more | Usually highest integration complexity |
| Common use | Modern cloud workloads | Specialized/control-heavy workloads | Enterprise transition/integration |

---

# 8. Deployment model is not service model

This distinction is important.

**Deployment model** asks:

> Where/how is the cloud infrastructure deployed?

**Service model** asks:

> Which layers do we manage versus the provider?

For Azure, the main service models are:

```text
IaaS
PaaS
SaaS
```

---

# 9. Start with the complete technology stack

Imagine a traditional application stack:

```text
Data
Application
Runtime
Middleware
Operating System
Virtualization
Servers
Storage
Networking
Physical Datacenter
```

Someone must manage every layer.

The difference between on-premises, IaaS, PaaS, and SaaS is primarily **who manages which layers**.

---

# 10. On-premises

With traditional on-premises infrastructure, the organization manages essentially the entire stack.

```text
YOU MANAGE
──────────────
Data
Application
Runtime
Middleware
Operating System
Virtualization
Servers
Storage
Networking
Datacenter
```

If hardware fails, the organization replaces it.

If the hypervisor needs maintenance, the organization manages it.

If the OS requires patching, the organization patches it.

If the application breaks, the organization fixes it.

This provides control but creates significant operational responsibility.

---

# 11. Infrastructure as a Service — IaaS

IaaS provides infrastructure resources such as virtual machines, networking, and storage while the cloud provider manages the underlying physical infrastructure and virtualization platform.

Azure Virtual Machines are the easiest example.

Our VM lab was IaaS.

When we created Ubuntu in Azure, we received a virtual machine.

Microsoft managed infrastructure such as:

```text
Datacenter
Physical networking
Physical servers
Underlying storage infrastructure
Hypervisor / virtualization platform
```

We managed the guest environment and workload, including responsibilities such as:

```text
Guest operating system configuration
OS patching
Installed software
Nginx
Application files
Application configuration
Many guest-level security settings
Data
```

That is why we could SSH into the VM and install Nginx ourselves.

## Mental model

```text
Microsoft
Physical infrastructure + virtualization
        │
        ▼
Azure VM boundary
        │
        ▼
Customer
Guest OS + software + application + data
```

IaaS gives us significant control, but that control comes with operational responsibility.

---

# 12. Platform as a Service — PaaS

PaaS provides a managed application or data platform where the cloud provider manages more of the underlying technology stack.

A conceptual web application example is Azure App Service.

Instead of:

```text
Create VM
Install OS updates
Install web server
Configure runtime
Maintain server
Deploy application
```

we can focus more directly on:

```text
Application code
Configuration
Data
```

while Azure manages more of the platform underneath.

Conceptually:

```text
YOU MANAGE
────────────
Application
Data
Application configuration

AZURE MANAGES MORE OF
─────────────────────
Runtime/platform components
Operating system
Virtualization
Servers
Storage infrastructure
Networking infrastructure
Datacenter
```

Exact responsibility varies by Azure service, so the model is conceptual rather than a replacement for each service's documentation.

---

# 13. Why PaaS exists

Suppose a development team wants to build a website.

Their business value comes from the application—not from maintaining Ubuntu servers.

IaaS approach:

```text
Develop app
+ manage VM
+ patch OS
+ configure runtime
+ configure web server
+ monitor OS
+ scale servers
```

PaaS approach:

```text
Develop app
+ deploy/configure application
+ manage application/data concerns
```

Azure handles more platform operations.

The trade-off is that the customer has less low-level control.

---

# 14. Software as a Service — SaaS

SaaS provides a complete application that customers consume rather than build and operate as infrastructure.

A common example is Microsoft 365.

Users do not create Exchange servers to use Outlook Online.

They consume the software service.

Conceptually:

```text
User
  ↓
SaaS Application
  ↓
Provider operates application platform and infrastructure
```

The customer still has responsibilities such as managing identities, access, data usage, device/security configuration, and application settings depending on the service.

SaaS does **not** mean the customer has zero responsibility.

---

# 15. IaaS vs PaaS vs SaaS — simple example

Imagine we need email.

## On-premises

```text
Buy servers
Install OS
Install mail software
Patch everything
Operate mail system
```

## IaaS

```text
Create Azure VMs
Install/configure mail software
Manage guest OS and application
Azure manages physical infrastructure
```

## PaaS-style thinking

Use a managed platform where the provider manages the underlying OS/runtime and we configure/deploy our workload.

## SaaS

```text
Subscribe to Microsoft 365
Create users
Configure service
Use email
```

As we move toward SaaS, the provider manages more technology layers.

---

# 16. Pizza analogy — useful but incomplete

A common analogy is pizza:

```text
On-prem → make everything yourself
IaaS    → someone provides the kitchen/infrastructure
PaaS    → more preparation/platform is provided
SaaS    → finished meal/service
```

This is useful for initial understanding, but enterprise architecture requires thinking in actual technology layers and responsibilities rather than relying only on analogies.

---

# 17. Service-model comparison

| Area | On-Prem | IaaS | PaaS | SaaS |
|---|---|---|---|---|
| Physical datacenter | Customer | Provider | Provider | Provider |
| Physical servers | Customer | Provider | Provider | Provider |
| Hypervisor | Customer | Provider | Provider | Provider |
| Guest OS | Customer | Customer | Mostly provider | Provider |
| Runtime/platform | Customer | Customer | Provider | Provider |
| Application | Customer | Customer | Customer | Provider |
| Customer data | Customer | Customer | Customer | Customer responsibility remains significant |
| Customer configuration/access | Customer | Customer | Customer | Customer |

The exact boundary depends on the specific service.

---

# 18. Control vs management responsibility

There is a general trade-off:

```text
More control
    ▲
    │ On-Prem
    │ IaaS
    │ PaaS
    │ SaaS
    ▼
Less infrastructure management
```

IaaS gives greater low-level control.

PaaS reduces infrastructure-management responsibility.

SaaS allows users to consume a complete application.

The correct choice depends on requirements.

---

# 19. When might we choose IaaS?

IaaS may be appropriate when:

```text
Application requires OS-level control
Legacy software expects a server
Custom software must be installed
Specific OS configuration is required
Migration needs minimal application changes
```

Example:

A legacy Java application requires a specific operating-system package and custom server configuration.

Moving it first to an Azure VM may be simpler than redesigning it immediately for PaaS.

This is often called a lift-and-shift style migration.

---

# 20. When might we choose PaaS?

PaaS may be appropriate when:

```text
Team wants to focus on application development
OS management provides little business value
Managed scaling/availability features are useful
Application fits the platform constraints
Faster deployment is important
```

Example:

A new web API can run on a supported managed application platform without requiring custom OS access.

PaaS may reduce operational overhead.

---

# 21. When might we choose SaaS?

SaaS may be appropriate when the business requirement is already solved by a mature product.

Example:

Requirement:

```text
Company needs business email and collaboration
```

Instead of building an email platform on VMs, the company can consume Microsoft 365.

The engineering question should always be:

> Does building and operating this capability ourselves provide business value?

---

# 22. Shared Responsibility Model

Moving to cloud does not mean Microsoft becomes responsible for everything.

Cloud security and operations follow a **shared responsibility model**.

Both Microsoft and the customer have responsibilities.

The boundary changes depending on whether we use IaaS, PaaS, or SaaS.

---

# 23. Responsibilities Microsoft always owns in Azure cloud services

For Azure cloud infrastructure, Microsoft is responsible for the physical cloud infrastructure, including areas such as:

```text
Physical datacenter
Physical hosts
Physical network
```

Customers do not enter an Azure datacenter to replace failed disks in the physical server hosting their VM.

Microsoft operates that infrastructure.

---

# 24. Responsibilities customers always retain

Some responsibilities never disappear simply because we use cloud.

Customers remain responsible for important areas such as:

```text
Data
Identities/accounts
Access decisions
Endpoint/device considerations
How the service is configured and used
```

The exact division varies by service, but a company must still decide:

```text
Who should access the data?
Which users should be administrators?
What information can be stored?
How should identities be protected?
How should application settings be configured?
```

Azure cannot make those business decisions automatically.

---

# 25. Shared responsibility in IaaS

Consider our Ubuntu VM.

Microsoft manages the physical infrastructure and virtualization platform.

But if we never patch Ubuntu and the guest OS becomes vulnerable, that is generally our responsibility.

```text
Microsoft
─────────
Datacenter
Physical servers
Physical networking
Hypervisor

Customer
────────
Guest OS
OS patches
Installed software
Application
Guest firewall/configuration
Data
Identity/access configuration
```

This is why IaaS requires strong operations practices.

---

# 26. Shared responsibility in PaaS

With PaaS, Azure takes responsibility for more layers.

For a managed application platform, Azure manages the underlying OS/platform infrastructure.

But we still manage our application and data.

If our application code has an authorization bug that exposes customer records, Azure cannot automatically fix the business logic.

```text
Azure
────────
Infrastructure
OS/platform
Managed runtime/service components

Customer
────────
Application code
Data
Identity/access choices
Application configuration
Secure development
```

PaaS removes some operational responsibility, not application responsibility.

---

# 27. Shared responsibility in SaaS

With SaaS, the provider operates nearly the entire application stack.

But customers still manage how their organization uses the service.

Example questions:

```text
Who gets an account?
Who is an administrator?
Should MFA be required?
Who can share documents externally?
What data may users upload?
```

These remain customer governance/security responsibilities.

---

# 28. Security does not disappear as we move to SaaS

The type of security work changes.

```text
On-Prem / IaaS
More infrastructure + OS security work

PaaS
More application + identity + configuration focus

SaaS
More identity + access + data governance + configuration focus
```

So the correct statement is not:

> SaaS means no security work.

It is:

> The provider manages more technical layers, while the customer remains responsible for how identities, data, access, and configuration are used.

---

# 29. Example — our Azure VM

Let's connect this directly to what we already built.

We created:

```text
Azure VM
Ubuntu
Nginx
HTML page
```

Microsoft handled:

```text
Azure datacenter
Physical server
Physical networking
Hypervisor
Underlying cloud platform
```

We handled:

```text
VM configuration
SSH access
Ubuntu guest OS
Nginx installation
HTML file
Application configuration
NSG choices
Our data
```

When we restarted the VM and the HTML file remained, the persistent disk preserved the VM's data.

When we deallocated the VM, Azure released compute capacity while persistent resources remained.

That entire lab demonstrates the IaaS boundary.

---

# 30. Example — same website using PaaS

Suppose instead of creating Ubuntu and installing Nginx, we use an Azure managed web application platform.

The workflow becomes conceptually:

```text
Developer
   ↓
Application code
   ↓
Azure managed web platform
   ↓
Azure-managed OS/runtime/infrastructure
```

We no longer care which physical server or hypervisor hosts the application, and we normally do not administer the guest OS as we did with our VM.

This is the shift from infrastructure management toward application management.

---

# 31. Example — SaaS

Suppose the requirement is not to build a website but simply to provide employee email.

Instead of:

```text
Azure VM
   ↓
Install mail server
   ↓
Patch and operate mail application
```

we can consume a SaaS product such as Microsoft 365.

The organization focuses on:

```text
Users
Licensing
Access
Configuration
Data governance
Security settings
```

rather than running mail-server infrastructure.

---

# 32. Cloud model does not automatically determine cost

A common mistake is:

```text
SaaS = cheapest
PaaS = medium
IaaS = expensive
```

That is not universally true.

Cost depends on:

```text
Workload
Scale
Licensing
Traffic
Storage
Operations effort
Architecture
Service pricing
Support requirements
```

A PaaS service may cost more per unit than a small VM but save substantial engineering/operations effort.

Total cost of ownership matters, not only the Azure line-item price.

---

# 33. Cloud model does not automatically determine security

Another mistake is:

```text
PaaS is always secure
IaaS is always insecure
```

Security depends on architecture and configuration.

PaaS removes some infrastructure-management responsibilities, which can reduce certain risks, but insecure application code, weak identities, excessive permissions, exposed data, or bad configuration can still create serious vulnerabilities.

---

# 34. Migration example

Imagine an enterprise has a legacy application:

```text
On-Prem
Physical/virtual server
Windows/Linux
Application server
Legacy application
Database
```

A migration path could be:

```text
Stage 1
On-Prem → Azure VM
Rehost / lift-and-shift

Stage 2
Modernize application components
VM → managed application/database services

Stage 3
Use SaaS for capabilities that no longer need custom development
```

Not every application follows this path, but it demonstrates how service models influence modernization.

---

# 35. Hybrid + service models can exist together

Deployment and service models can be combined.

Example:

```text
On-Premises
Legacy Database
      │
      │ Hybrid connection
      ▼
Azure
PaaS Web Application
      │
      ▼
SaaS Identity/Business Service integration
```

An enterprise architecture can therefore contain multiple deployment and service models simultaneously.

---

# 36. Decision framework

When selecting a service model, ask:

```text
Do we need OS-level control?
        │
        ├── Yes → IaaS may be appropriate
        │
        └── No
             ↓
Does our application fit a managed platform?
        │
        ├── Yes → consider PaaS
        │
        └── No → evaluate IaaS/containers/other architecture

Does a complete SaaS product already solve the business requirement?
        │
        ├── Yes → consider SaaS
        └── No → build using appropriate platform
```

This is not a rigid rule. Architecture decisions require performance, security, cost, compliance, compatibility, reliability, and operational analysis.

---

# 37. Common misunderstandings

## “Public cloud means everyone can see my resources.”

No. Public refers to the provider offering cloud services broadly. Customer workloads still use isolation and access controls.

## “Private cloud means no virtualization.”

No. Private cloud commonly uses virtualization and cloud-style automation but is dedicated to one organization.

## “Hybrid means two Azure regions.”

No. Two Azure regions are still public-cloud architecture. Hybrid combines public cloud with private/on-premises environments.

## “Multi-cloud and hybrid cloud are the same.”

No. Hybrid combines public cloud with private/on-premises environments. Multi-cloud uses multiple public cloud providers.

## “Azure VM is PaaS because Azure manages the hardware.”

No. Azure VM is IaaS because we still manage the guest operating system and workload.

## “PaaS means Azure manages my application code.”

No. Azure manages more of the platform; the customer still owns the application and data.

## “SaaS means we have no security responsibility.”

No. Identity, access, data, configuration, and governance responsibilities remain.

---

# 38. Final mental model

```text
DEPLOYMENT MODEL
Where/how does infrastructure run?

Public      Private      Hybrid
Cloud       Cloud        Cloud
                         │
                         └─ On-prem/private + public cloud


SERVICE MODEL
How much of the stack do we manage?

On-Prem → IaaS → PaaS → SaaS

More customer management ─────────────→ More provider management
More low-level control   ←───────────── Less infrastructure management


SHARED RESPONSIBILITY
Who secures and operates each layer?

Physical Azure infrastructure → Microsoft
Guest OS in IaaS               → Customer
Managed platform in PaaS       → Microsoft manages more
Application/data/configuration → Customer responsibilities remain
Identity/access decisions      → Customer responsibilities remain
```

The simplest way to remember everything is:

> **Deployment model tells us where/how the environment is hosted. Service model tells us how much of the technology stack the provider manages. Shared responsibility tells us who is accountable for securing and operating each layer.**

---

# 39. What comes next

Now that we understand where cloud environments run and who manages each technology layer, the next missing foundation topic is **Azure Global Infrastructure**:

```text
Geographies
      ↓
Regions
      ↓
Datacenters
      ↓
Availability Zones
      ↓
Region pairs / cross-region design concepts
```

That will connect the abstract concepts of High Availability and Disaster Recovery to Azure's actual physical/global architecture.