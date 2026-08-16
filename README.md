# Azure Enterprise Learning

This repository is a structured Azure learning journey built around understanding **why a cloud capability exists**, how it works, what problem it solves, how we validate it hands-on, and how the same idea appears in enterprise architecture.

It is **not** intended to be a collection of short certification definitions or duplicated notes.

## Documentation standard

Every topic we complete should be treated as one learning unit consisting of the written note, the applicable hands-on lab, and the matching Miro architecture.

A completed note should explain the topic in this order:

1. **Real-world problem** — what engineers/companies struggled with before this capability existed.
2. **Why the problem matters** — cost, availability, operations, security, scale, time, or another constraint.
3. **How the solution evolved** — connect the new concept to what we already learned instead of presenting it as an isolated definition.
4. **Core concept in depth** — terminology, internal behavior, components, relationships, boundaries, and lifecycle.
5. **Concrete scenario** — follow one realistic workload/request/failure through the architecture step by step.
6. **Azure implementation** — show how Azure represents the concept and what configuration decisions matter.
7. **Hands-on validation** — when useful, document what we actually created, why each setting was selected, commands/actions performed, expected observations, troubleshooting, and cleanup.
8. **Enterprise architecture** — explain how a production design differs from a learning lab.
9. **Failure scenarios and trade-offs** — what breaks, what survives, what the service does not solve, and important limitations.
10. **Cost, security, operations, HA/DR, monitoring, and governance implications** where applicable.
11. **Common misunderstandings** — explicitly correct confusing interpretations we encountered while learning.
12. **Miro architecture** — a real visual architecture/flow maintained on the Azure learning board. Text diagrams in Markdown may be used only as small explanatory aids; they are not a substitute for the Miro architecture.
13. **Official references** — relevant Microsoft Learn/service documentation for behavior that may change.

For a **new topic**, we learn/discuss it first. Only after the concept is clear do we finalize the GitHub note and architecture. For a topic already learned, we can document it directly from our completed discussion and lab.

## Repository structure

There must be **one canonical location for each topic**. We do not create a second folder or second note simply because we later improve the documentation. Improvements replace or expand the existing canonical note.

```text
azure-enterprise-learning/
│
├── 01-cloud-foundations/
│   └── Foundational concepts learned before individual Azure services
│
├── 02-compute/
│   └── Azure compute services and associated labs
│
├── 03-networking/        # when this module begins
├── 04-storage/           # later
├── 05-databases/         # later
├── 06-monitoring/        # later
└── ...
```

Governance is currently part of the cloud-foundation learning rather than duplicated into another folder. If governance later becomes a full Azure implementation module (Azure Policy, RBAC implementation, landing zones, etc.), we will create that module intentionally at that point rather than duplicating foundation material.

---

# Current Learning Path

## 01 — Cloud Foundations

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

The filename numbering reflects the incremental learning history. The list above is the authoritative learning order until we intentionally normalize filenames in one controlled cleanup.

## 02 — Compute

1. [Azure Virtual Machines (IaaS) — Concepts + Our Ubuntu/Nginx Lab](02-compute/01-azure-virtual-machines.md)

The VM unit connects virtualization and IaaS to a real Azure workload: provisioning, region and availability decisions, VM sizing, Ubuntu image, SSH authentication, NIC/private/public addressing, NSG rules, Nginx, persistent disks, monitoring, scaling, restart versus deallocation, cost, and production considerations.

---

# Architecture documentation

The associated **Miro Azure learning board** is the visual architecture source for completed learning units.

The purpose of a diagram is to show relationships and flow that are difficult to understand from prose alone: request paths, resource boundaries, failure boundaries, redundancy, identity/governance inheritance, network paths, replication, and service dependencies.

We will not use large ASCII/text diagrams as a replacement for architecture. Small text flows can still be used inside notes when they clarify a single sequence.

---

# Quality rule before moving forward

Before adding a new topic we should confirm:

- there is no duplicate canonical note;
- the previous topic is sufficiently detailed;
- the applicable Miro architecture exists and matches the note;
- the lab is documented if we performed one;
- important subtopics from our discussion are not missing;
- references are included where useful;
- the repository learning path still reflects the actual order.

This keeps the repository useful as long-term learning documentation instead of allowing it to become a collection of disconnected files.