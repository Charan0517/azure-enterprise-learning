# Azure Enterprise Learning

Hands-on Azure learning notes, labs, and enterprise architecture exercises.

The goal is to understand **why** Azure services exist, how the underlying architecture works, and then prove the concepts through hands-on labs before moving into enterprise design.

## Learning approach

Each major topic follows the same pattern:

```text
Real-world problem
        ↓
Why the problem exists
        ↓
Core cloud/Azure concept
        ↓
Architecture and failure boundaries
        ↓
Hands-on validation where useful
        ↓
Enterprise design considerations
        ↓
GitHub notes + Miro architecture diagram
```

The notes are intentionally detailed. They are meant to explain the reasoning behind Azure rather than provide only certification definitions.

---

# Learning Path

## 01 — Cloud Foundations

This module establishes the concepts required before studying individual Azure services.

1. [Why Cloud Computing Exists](01-cloud-foundations/01-why-cloud-computing-exists.md)
2. [Virtualization, Hypervisor, and VM Isolation](01-cloud-foundations/02-virtualization-hypervisor-and-isolation.md)
3. [Scalability, Elasticity, and Load Balancing](01-cloud-foundations/06-scalability-elasticity-load-balancing.md)
4. [Availability, Resiliency, Fault Tolerance, and Reliability](01-cloud-foundations/04-availability-resiliency-fault-tolerance-and-reliability.md)
5. [Disaster Recovery, RTO, RPO, and Regions](01-cloud-foundations/05-disaster-recovery-rto-rpo-and-regions.md)
6. [Reliability, Resiliency, and Fault Tolerance — Deeper Architecture View](01-cloud-foundations/07-reliability-resiliency-fault-tolerance.md)
7. [Predictability and Manageability](01-cloud-foundations/08-predictability-and-manageability.md)
8. [Governance and Compliance](01-cloud-foundations/09-governance-and-compliance.md)
9. [Sustainability](01-cloud-foundations/10-sustainability.md)
10. [Cloud Deployment Models, Service Models, and Shared Responsibility](01-cloud-foundations/11-cloud-deployment-service-models-and-shared-responsibility.md)
11. [Azure Global Infrastructure](01-cloud-foundations/12-azure-global-infrastructure.md)
12. [How All Cloud Foundation Concepts Connect](01-cloud-foundations/13-how-cloud-concepts-connect.md)

> **Note on filenames:** Some files retain their original numeric prefixes because they were created incrementally during the learning process. The numbered list above is the authoritative learning order. This avoids unnecessary file renames and broken historical links while keeping the learning path clear.

### Foundation reference material

The final concept-map note contains curated Microsoft Learn links for cloud concepts, reliability, Azure global infrastructure, Availability Zones, management groups, Azure Policy, and Azure RBAC. Product behavior should always be verified against current Microsoft documentation when implementing a real environment.

---

## 02 — Azure Governance and Organization

1. [Tenant, Management Groups, Subscriptions, Resource Groups, and Resources](02-azure-governance/01-tenant-management-groups-subscriptions-resource-groups.md)

This module takes the governance concepts from Cloud Foundations and applies them to the actual Azure resource hierarchy.

---

## 03 — Compute

1. [Azure Virtual Machines — Hands-On Lab](03-compute/01-azure-virtual-machines-hands-on.md)

The VM lab validates concepts such as provisioning, VM sizing, CPU/memory, public/private IP addresses, Linux administration, Nginx, monitoring, stopping versus deallocating, and persistence of the OS disk.

---

# Architecture documentation

Architecture diagrams are maintained on the associated Miro Azure learning board. Diagrams are treated as part of the learning unit rather than as decoration: the written note explains the concept and the architecture diagram shows how the components and failure boundaries connect.

---

# Next major module — Azure Networking

The VM lab gives us a real compute resource. The next major module builds the networking foundation around that resource.

Planned topics include:

```text
Virtual Networks (VNet)
        ↓
Address spaces and CIDR
        ↓
Subnets
        ↓
Private vs public IP addressing
        ↓
Network Interfaces
        ↓
Network Security Groups
        ↓
Routing and route tables
        ↓
DNS
        ↓
VNet-to-VNet connectivity / peering
        ↓
Internet, hybrid and private connectivity
        ↓
Azure Load Balancer and application traffic concepts
```

Networking will continue using the same problem → concept → architecture → lab → enterprise-design approach.