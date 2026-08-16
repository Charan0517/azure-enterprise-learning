# Scalability, Elasticity, and Load Balancing

Our previous topics focused mainly on failures.

- High Availability: keep the application available when components fail.
- Disaster Recovery: recover the application after a major disaster.

Now imagine that **nothing has failed**.

The application is healthy, but suddenly thousands of users arrive.

That creates a different problem: **capacity**.

---

# The real-world problem

Suppose we launch an online shopping application on one Azure VM.

```text
Users
  │
  ▼
VM1
2 vCPU
4 GB RAM
```

During normal hours:

```text
500 users
CPU = 25%
Memory = 40%
Application responds quickly
```

Then a major sale begins.

```text
500 users
   ↓
5,000 users
   ↓
20,000 users
```

The VM now reaches:

```text
CPU = 95-100%
Memory = 90%
Requests queue
Response time increases
Some requests fail
```

The VM has not crashed. The region has not failed. The application simply needs more computing capacity.

This introduces **scalability**.

---

# Scalability

Scalability is the ability of a system to increase or decrease its capacity to handle changes in workload.

There are two fundamental ways to scale compute:

```text
Vertical Scaling
      OR
Horizontal Scaling
```

---

# Vertical scaling — Scale Up / Scale Down

Vertical scaling means changing the capacity of an individual machine.

Suppose we have:

```text
VM1
2 vCPU
4 GB RAM
```

Traffic increases, so we resize it:

```text
VM1
8 vCPU
32 GB RAM
```

We did not add another VM.

We made the existing VM more powerful.

This is called:

```text
Scale Up
```

If demand later decreases and we resize it to a smaller VM:

```text
8 vCPU / 32 GB
        ↓
2 vCPU / 4 GB
```

that is:

```text
Scale Down
```

## Vertical scaling example

```text
Before

Users
  │
  ▼
VM1
2 CPU
4 GB RAM

After Scale Up

Users
  │
  ▼
VM1
8 CPU
32 GB RAM
```

## Advantages

Vertical scaling can be simple because the application may continue to run on one server without being redesigned for multiple instances.

It is useful for workloads that benefit from a stronger individual machine.

## Limitations

Every VM size has a maximum capacity.

Eventually we cannot continue adding CPU and RAM forever.

It can also leave us with a single-instance failure risk if only one VM exists.

Depending on the Azure resource and resize operation, changing size may also require a restart or interruption.

So vertical scaling alone does not automatically provide High Availability.

---

# Horizontal scaling — Scale Out / Scale In

Horizontal scaling means changing the **number of instances** rather than making one instance larger.

Start with:

```text
VM1
```

Traffic increases:

```text
VM1
VM2
VM3
VM4
```

This is called:

```text
Scale Out
```

When traffic decreases:

```text
VM1
VM2
VM3
VM4
   ↓
VM1
VM2
```

This is called:

```text
Scale In
```

## Horizontal scaling architecture

```text
                  Users
                    │
                    ▼
              Load Balancer
             /      |      \
            ▼       ▼       ▼
          VM1      VM2      VM3
```

Now workload can be distributed across multiple servers.

Horizontal scaling is extremely important in cloud architectures because cloud platforms make it practical to add and remove instances dynamically.

---

# Scale Up vs Scale Out

```text
Scale Up
Increase power of one machine

2 CPU → 8 CPU
4 GB → 32 GB
```

```text
Scale Out
Increase number of machines

1 VM → 2 VMs → 5 VMs → 10 VMs
```

Similarly:

```text
Scale Down
Reduce size of a machine

Scale In
Reduce number of machines
```

A quick memory trick:

```text
UP/DOWN → size of one resource
OUT/IN  → number of resources
```

---

# Why horizontal scaling needs Load Balancing

Suppose we scale out from one VM to three VMs.

```text
VM1
VM2
VM3
```

Now we have another problem.

When a user sends a request, which VM should receive it?

Without a traffic distribution mechanism, simply creating more VMs does not guarantee that requests will use them.

This introduces a **Load Balancer**.

---

# Load Balancing

Load balancing distributes incoming traffic across multiple healthy backend instances.

```text
                    Internet Users
                          │
                          ▼
                    Load Balancer
                  /       |       \
                 ▼        ▼        ▼
               VM1      VM2      VM3
```

Instead of users needing to know every VM's address, they access one application endpoint.

The load-balancing layer determines which healthy backend should receive each request.

Conceptually:

```text
Request 1 → VM1
Request 2 → VM2
Request 3 → VM3
Request 4 → VM1
...
```

The actual distribution algorithm depends on the service and configuration.

---

# Load balancing is not the same as scaling

This distinction is important.

Scaling changes **capacity**.

```text
2 VMs → 5 VMs
```

Load balancing distributes **traffic** across that capacity.

```text
Users → Load Balancer → VM1/VM2/VM3/VM4/VM5
```

They often work together, but they solve different parts of the problem.

---

# Health probes

A load balancer should not continue sending requests to an unhealthy VM.

For example:

```text
VM1 ✅ Healthy
VM2 ❌ Application stopped
VM3 ✅ Healthy
```

A **health probe** periodically checks whether backend instances are healthy.

Conceptually:

```text
Load Balancer
    │
    ├── probe VM1 → healthy
    ├── probe VM2 → unhealthy
    └── probe VM3 → healthy
```

Traffic can then be directed only to healthy instances:

```text
Users
  │
  ▼
Load Balancer
  ├────────► VM1 ✅
  │
  X────────► VM2 ❌
  │
  └────────► VM3 ✅
```

This is one reason load balancing contributes to High Availability as well as scalability.

---

# Scalability vs High Availability

These concepts overlap but are not identical.

Suppose we have three VMs.

They may help us:

```text
Handle more traffic → Scalability
```

and if one VM fails:

```text
Other VMs continue serving traffic → High Availability
```

So the same architecture can support both goals.

But the intent is different:

```text
Scalability
“Can the system handle increased workload?”

High Availability
“Can the system continue operating when something fails?”
```

---

# What is Elasticity?

Scalability tells us that the system **can change capacity**.

Elasticity means the system can adjust capacity dynamically as demand changes, often automatically.

Imagine an e-commerce application.

Normal traffic:

```text
2 VMs
```

Sale begins:

```text
Traffic rises
    ↓
2 VMs → 4 VMs → 8 VMs
```

Sale ends:

```text
Traffic falls
    ↓
8 VMs → 4 VMs → 2 VMs
```

This ability to expand and contract with workload is **elasticity**.

---

# Scalability vs Elasticity

These terms are often confused.

Think of them this way:

```text
Scalability
Can the system support increased/decreased capacity?

Elasticity
Can capacity adapt dynamically to changing demand?
```

For example, manually resizing a VM from 2 CPUs to 8 CPUs is scaling.

Automatically adding VMs when CPU usage rises and removing them when demand falls demonstrates elasticity.

Elasticity is especially valuable in cloud computing because we can avoid permanently paying for peak capacity when that capacity is only occasionally required.

---

# The traditional datacenter problem

Before cloud-style elasticity, organizations often had to purchase enough hardware for their expected peak demand.

Suppose normal demand needs:

```text
4 servers
```

but Black Friday needs:

```text
20 servers
```

The organization might buy 20 physical servers.

For most of the year:

```text
16 servers are mostly underutilized
```

That means money was spent on capacity sitting idle.

Cloud elasticity changes the model.

```text
Normal day
4 instances

Peak event
20 instances

After event
4 instances
```

This connects elasticity directly with cloud cost efficiency.

---

# Autoscaling

Autoscaling uses rules or metrics to automatically change capacity.

Example policy:

```text
If average CPU > 70% for 10 minutes
    ↓
Add instances

If average CPU < 30% for 20 minutes
    ↓
Remove instances
```

Conceptually:

```text
Azure Monitor / Metrics
        │
        ▼
Scaling Rules
        │
        ├── High demand → Scale Out
        │
        └── Low demand  → Scale In
```

CPU is only one possible signal. Depending on the Azure service, scaling can be based on metrics, schedules or other supported rules.

---

# Why not scale every time CPU briefly spikes?

Suppose CPU behaves like this:

```text
10:00  35%
10:01  90%
10:02  40%
```

If we immediately create a VM every time CPU briefly reaches 90%, we may constantly add and remove resources unnecessarily.

That can cause:

```text
Unnecessary cost
Scaling instability
Frequent instance creation/removal
```

So autoscale rules commonly consider a metric over a period of time and use separate scale-out and scale-in thresholds.

Example:

```text
Scale Out:
CPU > 70% for sustained period

Scale In:
CPU < 30% for sustained period
```

This separation helps prevent rapid back-and-forth scaling.

---

# Minimum, maximum, and default instance counts

An autoscaling design should have boundaries.

Example:

```text
Minimum instances = 2
Default instances = 2
Maximum instances = 10
```

Why minimum 2?

If High Availability is required, keeping at least two instances can prevent one VM from being the entire application tier.

Why maximum 10?

Without a limit, a workload problem or unexpected traffic could potentially cause uncontrolled resource growth and cost.

The maximum also reflects quota, architecture and downstream dependency limits.

---

# Virtual Machine Scale Sets (VMSS)

This connects directly to the Azure VM options we saw during our lab.

A **Virtual Machine Scale Set** is an Azure service for creating and managing a group of load-balanced virtual machines that can scale.

Instead of manually creating:

```text
VM1
VM2
VM3
VM4
```

one by one, we define a VM configuration and manage a group of instances.

Conceptually:

```text
                 Virtual Machine Scale Set
              ┌─────────────────────────────┐
              │ VM1   VM2   VM3   VM4      │
              └─────────────────────────────┘
```

With autoscaling:

```text
Low Traffic
VM1 VM2

      ↓ traffic increases

High Traffic
VM1 VM2 VM3 VM4 VM5 VM6

      ↓ traffic decreases

Low Traffic
VM1 VM2
```

VMSS therefore connects several concepts we have learned:

```text
Virtual Machines
      +
Horizontal Scaling
      +
Autoscaling
      +
Load Distribution
      +
High Availability design
```

---

# A complete scalable web architecture

Consider an application that starts with two VMs.

```text
                         USERS
                           │
                           ▼
                    Public Endpoint
                           │
                           ▼
                    Load Balancer
                    /          \
                   ▼            ▼
                 VM1            VM2
                  │              │
                  └──────┬───────┘
                         ▼
                      Database
```

Now traffic increases.

Monitoring detects sustained high CPU.

```text
Metrics
  │
  ▼
Autoscale Rule
  │
  ▼
VM Scale Set
  │
  ├── VM1
  ├── VM2
  ├── VM3
  └── VM4
```

The load balancer begins distributing traffic across healthy instances.

```text
Users
  │
  ▼
Load Balancer
 ├── VM1
 ├── VM2
 ├── VM3
 └── VM4
```

When demand decreases, autoscaling removes unnecessary instances while respecting the configured minimum.

This is a practical example of **elastic horizontal scaling**.

---

# Stateless applications make horizontal scaling easier

Suppose a user logs into VM1 and the application stores the user's session only in VM1 memory.

```text
Request 1 → VM1
Session exists on VM1

Request 2 → VM2
VM2 does not know that session
```

This creates problems when traffic is distributed across instances.

A common scalable architecture therefore tries to keep application servers stateless where practical and stores shared state in an appropriate shared data/session service.

Conceptually:

```text
             Load Balancer
             /          \
            ▼            ▼
          VM1            VM2
            \            /
             \          /
              ▼        ▼
          Shared State / Data Store
```

Now any healthy application instance can handle the request.

This is an important real-world design principle for horizontal scaling.

---

# Scaling one tier can expose another bottleneck

Suppose we scale the application tier:

```text
2 VMs → 20 VMs
```

but all 20 VMs use one database that cannot handle the increased workload.

```text
20 Application VMs
        │
        ▼
Single overloaded database
```

The application is still slow.

This teaches an important architecture lesson:

> A system is only as scalable as its bottleneck.

We must consider the complete request path:

```text
Network
Load Balancer
Application
Database
Storage
Cache
Messaging
External APIs
```

Scaling only one component does not guarantee that the complete application scales.

---

# Azure Load Balancer vs Application Gateway vs global traffic services

At this foundation stage, the important concept is that Azure has different traffic-management services for different layers and scopes.

**Azure Load Balancer** operates at Layer 4 and distributes TCP/UDP traffic to backend resources.

**Azure Application Gateway** is a Layer 7 web traffic load balancer and can make HTTP/HTTPS-aware routing decisions. It can also provide features such as Web Application Firewall depending on configuration/tier.

For global/multi-region architectures, Azure also has services designed to route users across regions or global endpoints.

We will study these properly in the Networking module rather than treating every traffic-routing service as the same thing.

For now remember:

```text
Load balancing is a concept.
Azure provides multiple services that implement traffic distribution at different layers and scopes.
```

---

# Scalability, Elasticity, Availability and DR together

We can now connect several cloud concepts.

```text
Users
  │
  ▼
Traffic Distribution
  │
  ▼
Multiple Application Instances
  │
  ├── one instance fails → High Availability
  │
  ├── traffic increases → Scale Out
  │
  ├── traffic decreases → Scale In
  │
  └── automatic adjustment → Elasticity
  │
  ▼
Data Layer
  │
  └── regional disaster → Disaster Recovery strategy
```

These are different capabilities working together.

---

# Cost relationship

Elasticity is not simply about performance.

It is also about matching cost to demand.

Without elasticity:

```text
Provision for peak capacity permanently
        ↓
Pay for resources even during low traffic
```

With elasticity:

```text
Low demand → fewer resources
High demand → more resources
Low demand → scale back in
```

However, autoscaling is not a guarantee of low cost. Poor scaling thresholds, excessive minimum capacity or unexpected traffic can still increase spending.

Monitoring, budgets and sensible limits remain important.

---

# Example: online ticket sale

Normal day:

```text
1,000 users/hour
2 VM instances
CPU ~35%
```

Concert tickets open at 10:00 AM.

```text
10:00 → 5,000 users
10:02 → 15,000 users
10:05 → 40,000 users
```

Monitoring observes sustained utilization above the scale-out threshold.

```text
2 VMs
  ↓
4 VMs
  ↓
8 VMs
```

Load balancing distributes requests across the healthy instances.

After ticket demand falls:

```text
8 VMs
  ↓
4 VMs
  ↓
2 VMs
```

This example contains all three concepts:

```text
Scalability
The application can support additional capacity.

Elasticity
Capacity changes as demand changes.

Load Balancing
Traffic is distributed across available instances.
```

---

# Common misunderstandings

## “If I add a Load Balancer, the application automatically scales.”

Not necessarily.

A load balancer distributes traffic. Scaling mechanisms add or remove capacity.

## “If I have multiple VMs, I automatically have High Availability.”

Not necessarily.

The VMs must be deployed across appropriate failure boundaries and traffic must be able to reach healthy instances.

## “Vertical scaling and horizontal scaling are the same.”

No.

```text
Vertical → change machine size
Horizontal → change machine count
```

## “RTO/RPO are part of autoscaling.”

No.

RTO/RPO are recovery objectives associated with failures/disasters. Autoscaling responds to workload/capacity requirements.

## “Elasticity means infinite resources.”

No.

Cloud resources still have quotas, service limits, configured maximums, budget considerations and architectural bottlenecks.

---

# Architecture decision example

Suppose we are designing an enterprise web application.

Requirements:

```text
Must survive one application VM failure
Traffic varies significantly during the day
Normal load needs 2 VMs
Peak load may need 8 VMs
Cost should reduce when demand falls
```

A possible design is:

```text
Minimum VM instances = 2
Maximum VM instances = 8
Health-aware load balancing
Autoscaling based on appropriate metrics
Instances distributed according to availability requirements
```

Now one architecture addresses multiple business needs:

```text
Multiple instances
      ↓
High Availability

Autoscale
      ↓
Elasticity

Additional instances
      ↓
Horizontal Scalability

Load Balancer
      ↓
Traffic Distribution
```

---

# Key takeaway

When traffic grows, ask four separate questions:

```text
1. Do we need a bigger individual resource?
   → Scale Up

2. Do we need more instances?
   → Scale Out

3. Should capacity automatically follow demand?
   → Elasticity / Autoscaling

4. How will requests reach the healthy instances?
   → Load Balancing
```

And remember:

```text
Scale Up / Down = resource SIZE
Scale Out / In  = resource COUNT
Scalability     = ability to change capacity
Elasticity      = capacity adapting to demand
Load Balancing  = distributing traffic
```

---

# Where this leads next

We have now covered how cloud systems respond to both **failure** and **changing demand**.

The next foundation concepts connect this architecture to the broader qualities expected from cloud systems, including **reliability, resiliency, fault tolerance, predictability and manageability**.
