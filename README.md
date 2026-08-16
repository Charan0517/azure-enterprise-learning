# Azure Enterprise Learning

Hands-on Azure learning notes, labs, and enterprise architecture exercises.

The goal is to understand **why** Azure services exist, how the underlying architecture works, and then prove the concepts through hands-on labs before moving into enterprise design.

## Learning Path

### 01 - Cloud Foundations

1. [Why Cloud Computing Exists](01-cloud-foundations/01-why-cloud-computing-exists.md)
2. [Virtualization, Hypervisor, and VM Isolation](01-cloud-foundations/02-virtualization-hypervisor-and-isolation.md)
3. [Scalability, Elasticity, and Load Balancing](01-cloud-foundations/03-scalability-elasticity-and-load-balancing.md)
4. [Availability, Resiliency, Fault Tolerance, and Reliability](01-cloud-foundations/04-availability-resiliency-fault-tolerance-and-reliability.md)
5. [Disaster Recovery, RTO, RPO, and Regions](01-cloud-foundations/05-disaster-recovery-rto-rpo-and-regions.md)

### 02 - Azure Governance and Organization

1. [Tenant, Management Groups, Subscriptions, Resource Groups, and Resources](02-azure-governance/01-tenant-management-groups-subscriptions-resource-groups.md)

### 03 - Compute

1. [Azure Virtual Machines — Hands-On Lab](03-compute/01-azure-virtual-machines-hands-on.md)

## Architecture Learning Approach

For each major topic we will follow the same pattern:

```text
Business / technical problem
        ↓
Core concept
        ↓
Azure service or architecture
        ↓
Hands-on lab
        ↓
Enterprise design considerations
        ↓
GitHub documentation + Miro architecture diagram
```

## Coming Next

The VM lab gives us a real compute resource. The next major module will build the networking foundation around it:

- Virtual Networks (VNet)
- Subnets
- IP addressing
- Network Security Groups
- Routing
- Public vs private connectivity
- DNS
- Load balancing

We will then continue through storage, databases, monitoring, security, governance, and eventually full enterprise architecture projects.