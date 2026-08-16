# How All Cloud Foundation Concepts Connect

This is the reference map for the Cloud Foundations module.

The individual notes explain each topic deeply. This page answers a different question:

> When we design a real Azure workload, how do all these concepts work together as one architecture?

---

## 1. Start with the original problem

Traditional infrastructure required organizations to purchase and operate physical capacity themselves.

```text
Business needs an application
        ↓
Need servers, storage and networking
        ↓
Purchase hardware for expected peak capacity
        ↓
Provisioning takes time
        ↓
Capacity may be underused normally
        ↓
Hardware failures and datacenter failures must be handled
        ↓
Organization also owns patching, monitoring, recovery and expansion
```

Cloud computing changes how that infrastructure is consumed.

```text
Physical Azure infrastructure
        ↓
Virtualization + managed platform services
        ↓
Resources provisioned through software/API
        ↓
Capacity can be created, changed and removed on demand
        ↓
Architecture can use multiple failure boundaries
        ↓
Usage can be measured and governed
```

---

# 2. Virtualization creates the cloud resource abstraction

Physical servers still exist.

Virtualization allows Azure to divide physical compute capacity into isolated virtual machines.

```text
Physical Server
      ↓
Hypervisor
      ↓
VM1   VM2   VM3
```

This gives us:

- hardware abstraction
- workload isolation
- flexible provisioning
- better utilization of physical infrastructure
- the foundation for infrastructure automation

But virtualization alone does not solve every cloud problem.

A VM can still become overloaded or fail.

That leads to scalability and availability.

---

# 3. Scalability answers: “Can the system handle more demand?”

If traffic grows, capacity must grow.

Two fundamental approaches are:

```text
Vertical scaling
VM: 2 CPU → 4 CPU → 8 CPU

Horizontal scaling
1 VM → 2 VMs → 5 VMs → 20 VMs
```

Scalability is the **ability** to increase or decrease capacity.

Elasticity adds automatic/dynamic adjustment according to demand.

```text
Traffic increases
      ↓
Scale out
      ↓
More application instances

Traffic decreases
      ↓
Scale in
      ↓
Remove unnecessary instances
```

---

# 4. Load balancing makes horizontal scaling useful

Creating several servers is not enough.

Traffic needs to reach them.

```text
Users
  ↓
Load Balancer
 ├── VM1
 ├── VM2
 └── VM3
```

The load-balancing layer can distribute requests across healthy backends.

This connects scalability with availability:

```text
More instances
      ↓
Capacity increases
      +
If one instance fails, traffic can use healthy instances
```

The exact behavior depends on the Azure load-balancing service and architecture.

---

# 5. Availability answers: “Can users still access the system?”

A service is available when users can use it when required.

One VM creates a single point of failure.

```text
Users
  ↓
VM
  ↓
VM failure = application unavailable
```

Redundancy reduces that dependency.

```text
Users
  ↓
Traffic distribution
 ├── VM1
 └── VM2
```

But where those instances are placed determines what type of failure the architecture can survive.

---

# 6. Azure global infrastructure defines failure boundaries

The physical hierarchy gives meaning to availability architecture.

```text
Geography
   ↓
Region
   ↓
Availability Zones where supported
   ↓
Datacenter infrastructure
   ↓
Physical hosts
```

Therefore we can design against different failure scopes.

```text
Host/local failure
      ↓
Local infrastructure redundancy

Zone/datacenter-group failure
      ↓
Multi-zone architecture

Entire region failure
      ↓
Multi-region Disaster Recovery
```

---

# 7. Reliability, resiliency and fault tolerance are related but different

A useful mental model:

```text
Reliability
Can the system consistently perform its intended function?

Resiliency
Can the system recover or continue when failures occur?

Fault tolerance
Can the system continue operating despite a component failure, ideally with little/no interruption?
```

These are architecture outcomes, not single Azure products.

They come from decisions involving:

- redundancy
- replication
- health detection
- failover
- retry/recovery logic
- monitoring
- backup
- capacity
- operational procedures

---

# 8. High Availability is not the same as Disaster Recovery

High Availability usually addresses failures within the active operating environment.

Example:

```text
Region A
├── Zone 1 → App VM1
└── Zone 2 → App VM2
```

If Zone 1 fails, Zone 2 may continue.

But both zones remain inside Region A.

If Region A is unavailable, another recovery location is required.

```text
Region A                  Region B
Primary                   Recovery
   │                          ▲
   └──── replication/backup ──┘
```

Therefore:

```text
HA → keep service running through localized failures
DR → restore/continue service after a major disaster
```

---

# 9. RTO and RPO turn DR into measurable business requirements

Disaster Recovery should not begin with “Which Azure service should we use?”

It begins with business impact.

```text
RTO = Recovery Time Objective
How long can the service be unavailable?

RPO = Recovery Point Objective
How much recent data can the business afford to lose?
```

Example:

```text
RTO = 5 minutes
RPO = 30 seconds
```

This means the architecture should target recovery of service within five minutes and a recovery point no more than approximately 30 seconds behind, subject to the guarantees and behavior of the selected services.

RTO/RPO influence:

- replication architecture
- backup frequency
- failover automation
- secondary-region readiness
- database design
- cost
- operational testing

---

# 10. Predictability answers: “Can we understand expected behavior and cost?”

A useful cloud platform should not only scale—it should also be understandable.

We need predictability in areas such as:

```text
Performance
Cost
Capacity
Deployment behavior
Operational behavior
```

Examples include:

- known VM sizes
- pricing models
- budgets
- monitoring metrics
- autoscale thresholds
- infrastructure templates

Predictability helps architecture teams plan rather than react blindly.

---

# 11. Manageability answers: “How do we operate all of this?”

Once an environment contains many resources, manual administration does not scale.

Manageability includes:

```text
Azure portal
CLI / PowerShell
APIs
Infrastructure as Code
Monitoring
Automation
Policy
Centralized operations
```

The goal is repeatable, observable and controlled operations.

---

# 12. Governance answers: “What are teams allowed to do?”

Cloud makes resource creation easy.

Without governance:

```text
Teams create resources anywhere
        ↓
Inconsistent naming
Unapproved regions
Unexpected cost
Security drift
Compliance problems
```

Governance introduces controlled boundaries.

Examples:

```text
Management Groups
Subscriptions
Resource Groups
Azure RBAC
Azure Policy
Tags
Budgets
Management locks
```

Governance is not about preventing cloud adoption. It is about allowing cloud usage within agreed organizational rules.

---

# 13. Compliance answers: “Are we meeting required obligations?”

Governance is what the organization controls and enforces.

Compliance asks whether systems satisfy required standards, laws, contracts and internal requirements.

Examples can include requirements around:

- data location
- encryption
- auditability
- access control
- retention
- industry standards

This directly affects architecture.

For example, the technically nearest Azure region may not be permitted for particular data.

---

# 14. Sustainability asks us to avoid unnecessary resource consumption

Cloud resources still consume physical energy and hardware.

Efficient architecture should avoid waste.

Examples:

```text
Right-size resources
Shut down/deallocate unused development compute
Scale down when demand falls
Use efficient managed services where appropriate
Remove abandoned resources
Optimize storage lifecycle
```

Sustainability often aligns with cost optimization because unused capacity consumes both money and infrastructure resources.

---

# 15. Deployment models describe where cloud capability lives

The common deployment models are:

```text
Public cloud
Cloud infrastructure operated by a provider and consumed by many customers with logical isolation

Private cloud
Cloud-style infrastructure dedicated to one organization

Hybrid cloud
Integration of private/on-premises environments with public cloud
```

The deployment model describes the environment relationship—not whether an individual application is IaaS, PaaS or SaaS.

---

# 16. Service models describe who manages what

The three foundational models are:

```text
IaaS → Infrastructure as a Service
PaaS → Platform as a Service
SaaS → Software as a Service
```

The key idea is the changing responsibility boundary.

```text
More customer control                      More provider management

On-premises → IaaS → PaaS → SaaS
```

For example:

```text
Azure VM
You manage the guest OS and application.

Managed application platform/database service
Azure manages more of the underlying platform, while you remain responsible for your data, configuration and application responsibilities.

SaaS application
Provider manages most of the application stack; the customer still owns responsibilities such as users, access decisions and data governance.
```

No cloud service model removes all customer responsibility.

---

# 17. Shared responsibility is the security/operations boundary

Responsibility changes depending on the service model.

But some responsibilities remain with the customer across cloud models, particularly around areas such as:

```text
Data
Identities
Accounts/access
Endpoint/client responsibilities
Configuration choices
```

The exact boundary must be understood for every service.

A useful question before deploying anything is:

> What does Azure manage here, and what are we still responsible for?

---

# 18. One complete enterprise scenario

Suppose a company launches an internet-facing application.

### Requirement

```text
Thousands of normal users
Traffic spikes during promotions
Application must survive a zone failure
Regional disaster recovery required
RTO = 5 minutes
RPO = 30 seconds
Only approved regions may be used
Development resources should not run unnecessarily
```

### Architecture reasoning

```text
1. Cloud computing
   → avoids purchasing fixed physical infrastructure

2. Virtualization / managed services
   → provide software-defined compute/platform resources

3. Scalability
   → capacity can increase as traffic grows

4. Elasticity
   → capacity can adjust automatically when appropriate

5. Load balancing
   → traffic is distributed across healthy application instances

6. Availability Zones
   → application instances are separated across local failure boundaries

7. Reliability / resiliency
   → architecture detects failures and continues/recovers

8. Disaster Recovery
   → second region protects against regional disaster

9. RTO / RPO
   → determine failover speed and data replication requirements

10. Predictability
    → monitoring, sizing and budgets make behavior/cost measurable

11. Manageability
    → automation and centralized operations manage the environment

12. Governance
    → policy/RBAC/subscription structure controls deployment

13. Compliance
    → only permitted locations and controls are used

14. Sustainability
    → unused capacity is removed or scaled down

15. Service model
    → determines how much infrastructure the company manages

16. Shared responsibility
    → defines what Azure handles and what the company must secure/operate
```

This is why these concepts should not be memorized as unrelated definitions.

They are different architecture decisions around the same workload.

---

# 19. The complete mental model

```text
WHY CLOUD?
Business needs faster, flexible infrastructure
        ↓
VIRTUALIZATION / MANAGED CLOUD PLATFORM
Abstract physical infrastructure
        ↓
SCALABILITY
Can capacity grow?
        ↓
ELASTICITY
Can capacity adjust with demand?
        ↓
LOAD BALANCING
How is traffic distributed?
        ↓
AVAILABILITY
Can users still reach the service?
        ↓
RELIABILITY / RESILIENCY / FAULT TOLERANCE
How does the system behave when components fail?
        ↓
AZURE GLOBAL INFRASTRUCTURE
Which physical failure boundaries are we using?
        ↓
DISASTER RECOVERY
What happens if a major location fails?
        ↓
RTO / RPO
How fast must we recover and how much data loss is acceptable?
        ↓
PREDICTABILITY / MANAGEABILITY
Can we understand and operate the environment consistently?
        ↓
GOVERNANCE / COMPLIANCE
Are resources controlled and compliant?
        ↓
SUSTAINABILITY
Are we using capacity responsibly?
        ↓
DEPLOYMENT + SERVICE MODELS
Where does the cloud run and who manages each layer?
        ↓
SHARED RESPONSIBILITY
Exactly what is Azure responsible for and what are we responsible for?
```

---

# 20. Quick decision table

| Question | Concept to think about |
|---|---|
| Why use cloud instead of buying servers? | Cloud computing benefits |
| How can many isolated workloads use physical hardware? | Virtualization |
| Can the application handle more demand? | Scalability |
| Can capacity adjust with changing demand? | Elasticity |
| How do requests reach multiple servers? | Load balancing |
| Can users continue using the service during failures? | Availability |
| Can the system recover from disruption? | Resiliency |
| Can it continue through a component failure? | Fault tolerance |
| Can it consistently perform its function? | Reliability |
| What if a whole region fails? | Disaster Recovery |
| How quickly must service return? | RTO |
| How much recent data loss is acceptable? | RPO |
| Where physically/logically should resources run? | Geography, region, Availability Zone |
| Can cost/performance be anticipated? | Predictability |
| How do we operate resources consistently? | Manageability |
| How do we control what teams can deploy? | Governance |
| Are regulatory/organizational obligations satisfied? | Compliance |
| Are we avoiding unnecessary resource usage? | Sustainability |
| Public, private or combined environment? | Deployment model |
| How much of the technology stack do we manage? | IaaS/PaaS/SaaS |
| Who secures/manages each layer? | Shared responsibility |

---

# 21. Microsoft reference links

These notes are written as learning explanations rather than copies of product documentation. When a concept depends on Azure's current product behavior, verify it against Microsoft documentation because services, regions and capabilities evolve.

## Azure fundamentals and cloud concepts

- [Microsoft Learn — Describe cloud computing](https://learn.microsoft.com/training/modules/describe-cloud-compute/)
- [Microsoft Learn — Describe the benefits of using cloud services](https://learn.microsoft.com/training/modules/describe-benefits-use-cloud-services/)
- [Microsoft Learn — Describe cloud service types](https://learn.microsoft.com/training/modules/describe-cloud-service-types/)

## Azure architecture and reliability

- [Azure Well-Architected Framework](https://learn.microsoft.com/azure/well-architected/)
- [Reliability in the Azure Well-Architected Framework](https://learn.microsoft.com/azure/well-architected/reliability/)

## Azure global infrastructure

- [Azure geographies](https://azure.microsoft.com/explore/global-infrastructure/geographies/)
- [Azure regions](https://azure.microsoft.com/explore/global-infrastructure/geographies/#geographies)
- [What are Azure Availability Zones?](https://learn.microsoft.com/azure/reliability/availability-zones-overview)
- [Azure regions with Availability Zone support](https://learn.microsoft.com/azure/reliability/regions-list)

## Governance

- [Azure management groups](https://learn.microsoft.com/azure/governance/management-groups/overview)
- [Azure Policy overview](https://learn.microsoft.com/azure/governance/policy/overview)
- [Azure RBAC overview](https://learn.microsoft.com/azure/role-based-access-control/overview)

---

# 22. What comes after foundations?

The foundations explain **why architectures are designed the way they are**.

The next modules apply these ideas to actual Azure services.

```text
Cloud Foundations
      ↓
Azure account / governance hierarchy
      ↓
Compute
      ↓
Networking
      ↓
Storage
      ↓
Databases
      ↓
Identity and Security
      ↓
Monitoring and Operations
      ↓
Cost Management
      ↓
Enterprise architecture projects
```

From this point forward, whenever we learn an Azure service, we should keep asking:

```text
What problem does it solve?
Where does it sit in the architecture?
What failure boundary does it use?
How does it scale?
Who manages it?
How is it secured?
How is it monitored?
How much does it cost?
How is it governed?
How would we recover it?
```

That is the bridge from learning Azure services to designing real Azure systems.