# Why Cloud Computing Exists

Before cloud computing, companies had to purchase and maintain their own physical infrastructure to run applications.

A typical setup looked like this:

```text
Users
  │
  ▼
Application
  │
  ▼
Physical Server
├── CPU
├── RAM
├── Storage
└── Network
```

If a company had multiple applications, it would often end up buying multiple physical servers.

For example:

```text
Payroll Application
       │
       ▼
Physical Server 1

HR Application
       │
       ▼
Physical Server 2

Inventory Application
       │
       ▼
Physical Server 3
```

At first this sounds reasonable because each application gets its own server and one application does not interfere with another.

But this creates several practical problems.

## Physical servers are expensive even when they are barely used

A company may buy a server with:

```text
32 CPU Cores
128 GB RAM
4 TB Storage
```

but the application may normally use only:

```text
CPU    → 10%
Memory → 15%
```

The rest of the server capacity stays unused.

The company still pays for the full server.

It also continues paying for:

- Electricity
- Cooling
- Datacenter space
- Hardware support
- Maintenance
- Operating system licensing
- Networking equipment
- Backup infrastructure
- Physical security

So the problem is not just that the server is expensive. The bigger problem is that the company paid for capacity that may remain unused most of the time.

This is called **underutilization**.

## Why not just buy a smaller server?

That seems like the obvious solution.

Suppose the application normally receives:

```text
1,000 users per day
```

and a small server handles that traffic comfortably.

Now the company runs a large marketing campaign and suddenly receives:

```text
100,000 users
```

The small server may not have enough CPU or memory.

```text
Normal Day

Users
  │
  ▼
Small Server
CPU → 30%
Everything works

Marketing Campaign

100,000 Users
      │
      ▼
Small Server
CPU → 100%
Memory → Full
Requests → Waiting
Response time → Slow
Errors → Increasing
```

So the company has another decision to make.

It could buy infrastructure for normal demand. That saves money during normal periods but creates problems during traffic spikes.

Or it could buy enough infrastructure for the maximum possible traffic.

```text
Capacity Purchased

████████████████████ 100%

Normal Usage

████                 20%
```

Now the application can handle the peak, but most of the infrastructure sits idle when demand is normal.

This creates a difficult trade-off:

```text
Buy for normal demand
        ↓
Cheaper
        ↓
Risk during traffic spikes

Buy for peak demand
        ↓
Handles spikes
        ↓
Large amount of unused capacity
```

## Physical infrastructure also takes time to increase

Suppose the company realizes today that it needs another server.

With traditional infrastructure, the process may look something like:

```text
Need additional capacity
        │
        ▼
Request approval
        │
        ▼
Purchase server
        │
        ▼
Wait for delivery
        │
        ▼
Rack and cable server
        │
        ▼
Configure networking
        │
        ▼
Install operating system
        │
        ▼
Configure application
        │
        ▼
Server finally available
```

That can take days or weeks.

Application demand can change much faster than physical infrastructure can be purchased and installed.

## Hardware also fails

Physical servers are not permanent.

Components can fail:

```text
Hard disk
Power supply
Memory
CPU
Motherboard
Network interface
Cooling system
```

Suppose the entire application runs on one physical server.

```text
Users
  │
  ▼
Application
  │
  ▼
Physical Server
```

If that server fails:

```text
Physical Server ❌
        │
        ▼
Application ❌
        │
        ▼
Users cannot access service
```

The company has to repair or replace the hardware and restore the application.

So companies also need:

- Spare hardware
- Backups
- Recovery procedures
- Monitoring
- Disaster recovery plans
- Engineers to maintain everything

## Building infrastructure in multiple locations is even harder

Suppose a company wants its application available even if an entire datacenter fails.

It may need infrastructure in another location.

```text
Datacenter A
│
└── Application

Datacenter B
│
└── Recovery Environment
```

Now the company may have to maintain:

- Two datacenters
- Servers in both locations
- Networking between locations
- Data replication
- Backup systems
- Security
- Monitoring
- Power and cooling in both locations

This can become extremely expensive.

# The bigger business problem

All these issues together create the same fundamental problem:

> **Companies need computing resources, but owning enough physical infrastructure for every possible situation is expensive, slow and difficult to manage.**

The company really wants something simpler.

It wants to say:

```text
I need more compute.
```

and receive more compute.

Or:

```text
I no longer need these resources.
```

and stop using them.

Without purchasing another physical server every time.

# This is where cloud computing becomes useful

Instead of every company building its own massive infrastructure, cloud providers such as Microsoft build enormous datacenters containing large amounts of:

```text
Compute
Storage
Networking
Databases
Security services
Monitoring systems
```

Microsoft owns and operates the physical infrastructure.

Customers consume resources from that infrastructure.

Conceptually:

```text
                    MICROSOFT AZURE

        ┌────────────────────────────────┐
        │ Physical Servers               │
        │ Storage Systems                │
        │ Networking                     │
        │ Datacenters                    │
        │ Power / Cooling                │
        │ Physical Security              │
        └───────────────┬────────────────┘
                        │
                        │ Cloud services
                        ▼

                   CUSTOMER
                        │
             ┌──────────┼──────────┐
             ▼          ▼          ▼
             VM       Storage    Database
```

The customer does not have to purchase the physical server underneath every Azure resource.

The customer requests the resource it needs.

For example:

```text
I need:

2 vCPU
8 GB RAM
Ubuntu
West US 2
```

Azure finds appropriate infrastructure and provides the requested resource.

# CAPEX vs OPEX

This also changes the financial model.

With traditional infrastructure, companies may have to purchase expensive hardware upfront.

This is generally associated with **Capital Expenditure (CAPEX)**.

Example:

```text
Buy servers
Buy storage
Buy network equipment
Build datacenter
```

The money is spent before the infrastructure can even be used.

Cloud computing can shift much of this toward an **Operational Expenditure (OPEX)** model.

Instead of buying the physical server, the company consumes cloud resources and pays based on the service/pricing model.

Conceptually:

```text
Traditional

Buy infrastructure first
        ↓
Own it
        ↓
Maintain it

Cloud

Request resource
        ↓
Use resource
        ↓
Pay according to usage/service pricing
```

This does not mean cloud is automatically cheaper in every situation.

Poorly designed cloud environments can also become very expensive.

The advantage is that infrastructure can be consumed more flexibly instead of requiring the company to purchase all physical capacity upfront.

# Cloud does not mean the internet

The internet and cloud computing are related, but they are not the same thing.

The internet is the network that allows systems around the world to communicate.

Cloud computing is a model for delivering computing resources and services.

For example:

```text
My Laptop
    │
    │ Internet
    ▼
Azure
    │
    ▼
Application
```

The internet is helping the request reach Azure.

Azure is providing the actual computing infrastructure and services.

If the internet is working but the Azure application is down, the application will still not work.

# Cloud providers still use physical servers

Cloud resources are not created out of nothing.

Azure still depends on real physical infrastructure.

Underneath Azure there are:

```text
Physical servers
Physical CPUs
Physical RAM
Physical storage
Network switches
Routers
Power systems
Cooling systems
Datacenters
```

The difference is that **Microsoft owns and manages those resources instead of the customer**.

So the cloud is not:

```text
No physical hardware
```

It is closer to:

```text
Physical hardware
        │
        ▼
Managed by cloud provider
        │
        ▼
Abstracted into services
        │
        ▼
Consumed by customers
```

# But another problem appears

Microsoft has a huge number of physical servers and a huge number of customers.

If one customer needs a small server, Microsoft cannot realistically dedicate an entire powerful physical server to that customer and leave most of it unused.

That would recreate the same underutilization problem companies already had.

For example:

```text
Physical Server

64 CPU cores
256 GB RAM

Customer VM Requirement:

2 CPU
8 GB RAM
```

Giving the entire physical server to that one customer would waste most of the hardware.

So Microsoft needs a way to safely divide physical servers into multiple independent computing environments.

```text
Physical Server
        │
        ├── Customer A computer
        ├── Customer B computer
        ├── Customer C computer
        └── Customer D computer
```

But each customer must remain isolated.

Customer A must not see Customer B's memory.

Customer B must not access Customer C's applications.

And every customer should feel as though they have their own computer.

That problem leads directly to the next topic:

# Virtualization
