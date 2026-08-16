# Availability, Resiliency, Fault Tolerance, and Reliability

Cloud architecture is not only about creating resources quickly. Applications also need to continue operating when something fails.

These terms are related but describe different ideas.

## Availability

Availability asks:

> Is the application accessible when users need it?

If an application runs on only one server, that server is a single point of failure.

```text
Users → Server A → Application
```

If Server A fails, the application becomes unavailable.

A more available design uses redundant instances.

```text
             ┌→ Server A
Users → LB ──┤
             └→ Server B
```

If one instance fails, another may continue serving requests.

## High availability

High availability is an architectural approach designed to minimize downtime by avoiding unnecessary single points of failure and using redundancy.

It does not mean that failure can never happen.

It means the system is designed so that failures have less impact.

## Reliability

Reliability is the ability of a system to perform its intended function consistently over time.

A reliable architecture considers:

- Redundancy
- Monitoring
- Recovery
- Capacity
- Data protection
- Failure handling

Availability is therefore one important part of overall reliability.

## Resiliency

Resiliency describes a system's ability to respond to failures and recover while continuing to provide an acceptable service.

Example:

```text
VM2 fails
   ↓
Health check detects failure
   ↓
Traffic stops going to VM2
   ↓
VM1 and VM3 continue serving users
   ↓
VM2 is repaired/replaced
```

The architecture experienced a failure but handled it.

## Fault tolerance

Fault tolerance is a stronger concept: the system is designed to continue operating despite component failures, ideally with little or no interruption visible to the user.

Achieving stronger fault tolerance generally requires more redundancy and therefore may cost more.

## Availability Sets

Within Azure VM architecture, an Availability Set helps distribute VMs across different failure and maintenance boundaries within a datacenter.

Two important ideas are:

- **Fault domains** — separate groups of hardware that can fail together, such as power/network-related boundaries.
- **Update domains** — groups used so planned platform maintenance does not update every VM at the same time.

Conceptually:

```text
Availability Set

Fault Domain 1          Fault Domain 2
┌──────────────┐        ┌──────────────┐
│ VM1          │        │ VM2          │
│ Hardware A   │        │ Hardware B   │
└──────────────┘        └──────────────┘
```

The application still needs multiple VMs for this redundancy to be useful.

## Availability Zones

An Azure region may contain multiple physically separate Availability Zones.

Conceptually:

```text
Azure Region

Zone 1          Zone 2          Zone 3
┌────────┐      ┌────────┐      ┌────────┐
│ DC(s)  │      │ DC(s)  │      │ DC(s)  │
│ VM1    │      │ VM2    │      │ VM3    │
└────────┘      └────────┘      └────────┘
```

Zones have independent power, cooling, and networking infrastructure so that a datacenter-level problem in one zone is less likely to affect another zone.

## Availability Set vs Availability Zone

The basic distinction is:

```text
Availability Set
→ Protect against certain hardware/maintenance failures within a datacenter environment

Availability Zones
→ Distribute workloads across physically separate zones within an Azure region
```

Neither automatically protects against the loss of an entire Azure region.

Regional disaster recovery is a different architecture problem.

## Enterprise thinking

When designing a production application, do not begin with:

> Which Azure option should I click?

Begin with:

> What failures must this application survive?

For example:

```text
Server failure?
Rack/hardware boundary failure?
Datacenter/zone failure?
Entire region failure?
```

The required architecture depends on the business requirement and acceptable downtime.

## Key takeaway

High availability, resiliency, reliability, and fault tolerance are not interchangeable words.

They describe different aspects of designing systems that continue functioning when failures occur.