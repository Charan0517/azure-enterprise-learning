# High Availability — Designing for Failures Instead of Assuming They Will Never Happen

Virtualization gave us a way to create many independent Virtual Machines on shared physical infrastructure.

But virtualization by itself does not make an application highly available.

Suppose an application runs on one Azure VM:

```text
Users
  │
  ▼
Application
  │
  ▼
VM1
  │
  ▼
Underlying Azure infrastructure
```

Everything works while VM1 and the infrastructure supporting it are healthy.

But the application now depends on one VM.

If VM1 becomes unavailable:

```text
Users
  │
  ▼
VM1 ❌
  │
  ▼
Application unavailable ❌
```

The problem is not that Azure failed to create the VM correctly.

The architecture itself has only one application instance.

This introduces the next cloud architecture problem:

> **How can an application continue serving users when individual infrastructure components fail?**

That is where **High Availability** begins.

---

# Failure is not one single thing

Before adding redundancy, it is important to understand what can fail.

An application running in Azure depends on several layers:

```text
Application
    │
    ▼
Virtual Machine
    │
    ▼
Physical Host
    │
    ▼
Rack / power / networking infrastructure
    │
    ▼
Datacenter / Availability Zone
    │
    ▼
Azure Region
```

A failure can happen at any of these levels.

For example:

```text
Application process crashes
VM operating system hangs
Physical host fails
Power/network boundary fails
Datacenter/zone becomes unavailable
Entire region experiences a major outage
```

A design that protects against one failure does not automatically protect against all of them.

This is why the first question in availability design should not be:

> Which availability option should I select in the Azure portal?

It should be:

> **What failure boundary does the application need to survive?**

---

# Single Point of Failure

A **Single Point of Failure (SPOF)** is a component whose failure can stop the entire service because there is no equivalent healthy component available to continue the work.

With one VM:

```text
             USERS
               │
               ▼
        ┌──────────────┐
        │     VM1      │
        │ Application  │
        └──────────────┘
```

VM1 is a single point of failure.

The obvious first improvement is to create another application instance:

```text
             USERS
               │
          ┌────┴────┐
          │         │
          ▼         ▼
        VM1        VM2
```

Now there is redundancy.

But another problem appears immediately.

How does a user know whether to connect to VM1 or VM2?

And if VM1 fails, how do new requests stop going to VM1 and start going to VM2?

A traffic-distribution layer is normally needed in front of the instances.

```text
                         USERS
                           │
                           ▼
                  ┌─────────────────┐
                  │ Load Balancing  │
                  │     Layer       │
                  └────────┬────────┘
                           │
                ┌──────────┴──────────┐
                │                     │
                ▼                     ▼
        ┌──────────────┐      ┌──────────────┐
        │     VM1      │      │     VM2      │
        │ Application  │      │ Application  │
        └──────────────┘      └──────────────┘
```

The load-balancing layer can distribute requests to healthy backend instances according to the service configuration.

If VM1 becomes unhealthy:

```text
                         USERS
                           │
                           ▼
                    Load Balancer
                           │
                           │ health information
                           ▼
                  VM1 ❌      VM2 ✅
                    X           ▲
                    │           │
                    └───────────┘
                    Send traffic to healthy backend
```

The important idea is that **redundant instances need a mechanism that can actually use the redundancy**.

Simply creating VM2 and leaving users connected directly to VM1 would not provide the same benefit.

---

# But two VMs are not automatically highly available

Suppose VM1 and VM2 are both running on the same physical host.

```text
                  Physical Host A
              ┌─────────────────────┐
              │                     │
              │       VM1           │
              │       VM2           │
              │                     │
              └─────────────────────┘
```

If VM1 itself fails, VM2 may still work.

But if the physical host fails:

```text
                  Physical Host A ❌
              ┌─────────────────────┐
              │                     │
              │       VM1 ❌        │
              │       VM2 ❌        │
              │                     │
              └─────────────────────┘
```

Both instances can be lost together.

So redundancy is only useful when the redundant resources are distributed across the failure boundaries we are trying to survive.

This leads to the idea of **failure domains**.

---

# Failure Domains

A failure domain is a boundary within which infrastructure can share a common failure.

For example, two machines may depend on the same underlying rack, power source or networking equipment.

A simplified architecture could look like:

```text
                        Application
                             │
                      Load Balancer
                       /           \
                      /             \
                     ▼               ▼

              Fault Domain 1    Fault Domain 2
             ┌──────────────┐  ┌──────────────┐
             │     VM1      │  │     VM2      │
             │              │  │              │
             │ Hardware A   │  │ Hardware B   │
             └──────────────┘  └──────────────┘
```

Now a failure that affects the first hardware boundary is less likely to remove both application instances at the same time.

The exact physical implementation is managed by Azure. The architectural concept we care about is **not placing all redundant instances inside the same failure boundary**.

---

# Availability Sets

For Azure Virtual Machines, an **Availability Set** is a logical grouping that helps Azure distribute VMs across separate **Fault Domains** and **Update Domains** within the datacenter-level infrastructure used by the set.

Suppose we have two application VMs that need to remain available during certain host/infrastructure failures and planned platform maintenance.

We can place both VMs in one Availability Set:

```text
                         Availability Set
                ┌────────────────────────────────┐
                │                                │
                │   Fault Domain 1               │
                │   ┌──────────────┐             │
                │   │     VM1      │             │
                │   └──────────────┘             │
                │                                │
                │   Fault Domain 2               │
                │   ┌──────────────┐             │
                │   │     VM2      │             │
                │   └──────────────┘             │
                │                                │
                └────────────────────────────────┘
```

This answers the question we discussed earlier:

> If the same application VMs are on different physical servers, can they all be in a single Availability Set?

**Yes.** That is exactly the purpose of the set: the VMs belong to the same logical availability grouping while Azure distributes them across the set's underlying fault and maintenance boundaries.

The Availability Set itself is not a server that sits in front of the VMs. It is a **placement/availability construct** that influences how Azure distributes those VM instances.

The traffic still needs an appropriate routing/load-balancing design if users must be sent between multiple VM instances.

---

# Fault Domains inside an Availability Set

Fault domains help reduce the chance that all redundant VMs depend on the same physical hardware failure boundary.

Conceptually:

```text
Availability Set
│
├── Fault Domain 1
│     ├── Host / infrastructure group
│     └── VM1
│
└── Fault Domain 2
      ├── Different host / infrastructure group
      └── VM2
```

If infrastructure associated with Fault Domain 1 fails:

```text
Fault Domain 1 ❌              Fault Domain 2 ✅
       │                              │
      VM1 ❌                         VM2 ✅
                                      │
                                      ▼
                              Application continues
```

This only helps if the application really has multiple instances and can continue working on the remaining instance.

A single VM inside an Availability Set is still only one VM.

---

# Update Domains

Hardware failure is not the only reason a VM can become temporarily unavailable.

Azure also performs maintenance on the platform.

If every redundant VM were updated or restarted at exactly the same time during planned maintenance, redundancy would not help.

**Update Domains** separate VM instances into groups so platform maintenance can be applied in stages rather than affecting every VM in the Availability Set at once.

Conceptually:

```text
Availability Set
│
├── Update Domain 1
│      └── VM1
│
└── Update Domain 2
       └── VM2
```

During a maintenance event requiring VM impact:

```text
Step 1
Update Domain 1 → maintenance
VM1 may be affected
VM2 continues

Step 2
Update Domain 2 → maintenance
VM2 may be affected
VM1 is available again
```

So the two ideas solve different problems:

```text
Fault Domain
    ↓
Separate common hardware failure boundaries

Update Domain
    ↓
Separate groups for planned platform maintenance sequencing
```

They are related because both reduce the chance that all redundant VMs are unavailable simultaneously, but they address different causes.

---

# What Availability Sets do not solve

An Availability Set improves distribution across host-level/fault and maintenance boundaries, but the VMs still operate within a datacenter-level architecture.

Imagine this:

```text
                       Azure Region
                           │
                    Datacenter Area
                           │
                ┌──────────┴──────────┐
                │                     │
          Fault Domain 1        Fault Domain 2
                │                     │
               VM1                   VM2
```

Now suppose the problem is larger than one rack or physical-host group.

Examples include a major datacenter power, cooling or networking incident.

Both fault domains may ultimately depend on the same datacenter location.

So the next question becomes:

> **How do we distribute redundant application instances across physically separate datacenter locations inside the same Azure region?**

This is where **Availability Zones** are introduced.

---

# Availability Zones

An Azure region that supports Availability Zones contains physically separate zone locations with independent infrastructure characteristics such as power, cooling and networking.

Conceptually:

```text
                           AZURE REGION
                              West US 2
                                  │
          ┌───────────────────────┼───────────────────────┐
          │                       │                       │
          ▼                       ▼                       ▼

   Availability Zone 1     Availability Zone 2     Availability Zone 3
   ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
   │ Datacenter      │     │ Datacenter      │     │ Datacenter      │
   │ infrastructure  │     │ infrastructure  │     │ infrastructure  │
   │                 │     │                 │     │                 │
   │ Independent     │     │ Independent     │     │ Independent     │
   │ power/cooling/  │     │ power/cooling/  │     │ power/cooling/  │
   │ networking      │     │ networking      │     │ networking      │
   └─────────────────┘     └─────────────────┘     └─────────────────┘
```

Now the application can be distributed across zones:

```text
                             USERS
                               │
                               ▼
                        Load Balancing Layer
                         /                 \
                        /                   \
                       ▼                     ▼

             Availability Zone 1     Availability Zone 2
             ┌─────────────────┐     ┌─────────────────┐
             │      VM1        │     │      VM2        │
             │  Application    │     │  Application    │
             └─────────────────┘     └─────────────────┘
```

If Zone 1 experiences a significant outage:

```text
             Zone 1 ❌                    Zone 2 ✅
               VM1 ❌                       VM2 ✅
                                               │
                                               ▼
                                    Continue serving traffic
```

The application can continue **if the complete architecture supports operating from the surviving zone**.

That last point is important.

---

# Selecting an Availability Zone for one VM does not create High Availability

Suppose we create only:

```text
Availability Zone 1
        │
        ▼
       VM1
```

We have told Azure where to place VM1.

But there is still only one application instance.

If VM1 fails:

```text
VM1 ❌
  ↓
No second VM
  ↓
Application unavailable
```

If Zone 1 fails:

```text
Zone 1 ❌
   │
   └── VM1 ❌
```

Again there is no second zone instance.

So zone selection and high availability are not the same thing.

A zone-resilient VM architecture usually needs redundant resources across zones and the ability to route traffic to healthy resources.

---

# Availability Set vs Availability Zone

The easiest way to understand the difference is by asking **how large the failure is**.

```text
Physical host / rack / maintenance boundary
            ↓
     Availability Set

Datacenter / zone-level failure
            ↓
     Availability Zones

Entire Azure region failure
            ↓
     Multi-region / Disaster Recovery architecture
```

An Availability Set and Availability Zones are therefore not simply two names for the same feature.

They operate at different availability boundaries.

---

# What if the whole Azure Region fails?

Suppose the application is correctly distributed across two or three Availability Zones in West US 2.

```text
                          WEST US 2

                    ┌──── Zone 1 ──── VM1
                    │
Application ────────┼──── Zone 2 ──── VM2
                    │
                    └──── Zone 3 ──── VM3
```

This helps with zone-level failures inside the region.

But all three zones still belong to **West US 2**.

If the business requirement says the application must continue or recover after an entire regional disaster, another region must be considered.

```text
Region A — Primary
        │
        │ replication / recovery design
        ▼
Region B — Secondary
```

That becomes a **Disaster Recovery** problem rather than simply an Availability Zone problem.

This distinction prevents overestimating what a zone architecture provides.

---

# High Availability vs Disaster Recovery

These concepts are related but solve different scopes of failure.

High Availability typically focuses on keeping the service running through component or localized infrastructure failures with minimal interruption.

Disaster Recovery focuses on recovering the service after a larger disaster, often involving another region, backups, replication, failover and recovery objectives.

For example:

```text
VM1 fails
   ↓
VM2 continues immediately
   ↓
High Availability scenario
```

Compared with:

```text
Entire primary region unavailable
   ↓
Fail over / recover in another region
   ↓
Disaster Recovery scenario
```

We will handle RTO, RPO, replication, backups, regional recovery and failback as their own detailed unit.

---

# Availability

**Availability** answers a practical question:

> Is the application accessible and usable when users need it?

A service that works perfectly for one hour but is unavailable for several hours every day would not be considered highly available.

Availability is therefore about service accessibility over time.

High availability is the architecture used to reduce downtime by avoiding unnecessary single points of failure and using redundancy appropriately.

It does not mean:

```text
Nothing will ever fail
```

It means:

```text
Failures are expected
        ↓
Architecture includes redundancy
        ↓
Failure of one component should not automatically stop the whole service
```

---

# Fault Tolerance

Fault tolerance is a stronger failure-handling objective.

A fault-tolerant system is designed so that the system can continue operating despite component failures, ideally with little or no user-visible interruption for the failures it was designed to tolerate.

For example:

```text
Users
  │
  ▼
Traffic Layer
  │
  ├── VM1 ✅
  ├── VM2 ❌
  └── VM3 ✅

VM2 fails
  ↓
Remaining instances continue
  ↓
Users continue receiving service
```

Fault tolerance usually requires stronger redundancy and can therefore increase infrastructure cost and architectural complexity.

Not every application needs the strongest possible fault tolerance.

The business requirement determines how much interruption is acceptable.

---

# Resiliency

A resilient system is designed to **absorb failures, adapt and recover** while continuing to provide an acceptable service.

Consider an application with three instances:

```text
VM1 ✅
VM2 ✅
VM3 ✅
```

VM2 fails:

```text
VM2 ❌
   ↓
Health monitoring detects the problem
   ↓
Traffic stops going to VM2
   ↓
VM1 and VM3 continue serving users
   ↓
VM2 is repaired or replaced
   ↓
Capacity returns to normal
```

That whole behavior is an example of resiliency.

Resiliency can involve many mechanisms:

```text
Redundancy
Health monitoring
Load balancing
Retries where appropriate
Timeout handling
Scaling
Data replication
Backup and recovery
Failover
```

So resiliency is broader than simply saying, "we have two VMs."

---

# Reliability

Reliability is the ability of a system to perform its intended function consistently over time under the conditions it was designed for.

Availability contributes to reliability, but reliability is broader.

Imagine an application that is technically online 24/7 but gives incorrect results half the time.

It may be available, but it is not reliable.

A reliable architecture considers things such as:

```text
Correct operation
Availability
Failure handling
Capacity
Data integrity
Monitoring
Recovery behavior
Dependency reliability
```

This is why the terms should not be used as exact synonyms.

A useful relationship is:

```text
Availability
   └── Can users access the service?

High Availability
   └── Architecture intended to minimize downtime

Fault Tolerance
   └── Continue operating through defined component failures

Resiliency
   └── Absorb, adapt to and recover from failures

Reliability
   └── Consistently perform the intended function over time
```

---

# Redundancy must include the dependency chain

Suppose we create three application VMs:

```text
             Users
               │
               ▼
          Load Balancer
        /      |       \
       ▼       ▼        ▼
     VM1      VM2      VM3
       \       |       /
        \      |      /
          Database
```

The application tier has redundancy.

But if there is only one database with no suitable availability design:

```text
VM1 ✅
VM2 ✅
VM3 ✅
Database ❌
     ↓
Application cannot complete database operations
```

The system can still become unavailable.

So High Availability must be considered **end-to-end**, including important dependencies such as:

```text
Traffic entry point
Application instances
Databases
Storage
Networking
Identity dependencies
Messaging components
External services
```

Making only the VM tier redundant is not enough if another required component remains a single point of failure.

---

# A real enterprise scenario

Imagine an online payment application with the requirement:

> A single VM or physical host failure should not make the application unavailable.

A possible first architecture is:

```text
                             INTERNET
                                 │
                                 ▼
                         Load Balancing Layer
                                 │
                    ┌────────────┴────────────┐
                    │                         │
                    ▼                         ▼
              Application VM1          Application VM2
              Failure Boundary A       Failure Boundary B
                    │                         │
                    └────────────┬────────────┘
                                 ▼
                       Highly Available Data Tier
```

Now the requirement changes:

> The application must continue even if one datacenter zone is unavailable.

The architecture may need to become:

```text
                             INTERNET
                                 │
                                 ▼
                         Load Balancing Layer
                                 │
                    ┌────────────┴────────────┐
                    │                         │
                    ▼                         ▼
           Availability Zone 1      Availability Zone 2
             Application VM1          Application VM2
                    │                         │
                    └────────────┬────────────┘
                                 ▼
                      Zone-resilient Data Tier
```

Now the requirement changes again:

> The application must recover after the loss of the entire Azure region.

That cannot be solved by simply adding another VM in another zone of the same region.

The architecture must move into multi-region Disaster Recovery planning.

This shows why availability architecture should always begin with the failure scenario rather than with a portal option.

---

# How this connects to the next topic

At this point we know how to protect an application against several localized infrastructure failures:

```text
One VM
   ↓
Single point of failure
   ↓
Multiple instances
   ↓
Traffic distribution
   ↓
Separate failure boundaries
   ↓
Availability Sets / Fault & Update Domains
   ↓
Availability Zones
```

But eventually we reach a failure boundary that all zones inside one region still share:

```text
Azure Region
```

If the whole region is unavailable, we need to answer two new questions:

> How quickly must the application come back?

and

> How much recent data can the business tolerate losing?

Those questions introduce **Disaster Recovery, RTO and RPO**.