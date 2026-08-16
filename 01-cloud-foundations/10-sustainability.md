# Sustainability in Cloud Computing and Azure

By this point we have discussed how cloud systems should be available, scalable, resilient, predictable, manageable, governed, and recoverable.

There is one more cloud foundation quality that connects many of those ideas together:

```text
Sustainability
```

Sustainability in cloud architecture is about designing and operating technology in ways that reduce unnecessary resource consumption, improve efficiency, and lower environmental impact while still meeting business requirements.

It is not simply:

> Use fewer servers.

The real question is:

> **How can we deliver the required business outcome using computing resources efficiently rather than wasting capacity, energy, storage, and network usage?**

---

# 1. The real-world problem

Imagine an organization owns 100 physical servers.

```text
100 Physical Servers
│
├── CPU capacity
├── Memory capacity
├── Storage
├── Cooling demand
└── Electricity consumption
```

But average utilization is only:

```text
CPU = 10–15%
```

Most of the machines still consume power and require cooling even though much of their capacity is unused.

The company purchased infrastructure for occasional peak demand, but that peak may happen only a few times each year.

```text
Peak requirement
████████████████████ 100%

Normal utilization
███                  15%
```

This is not only a cost problem.

It is also an efficiency and sustainability problem because physical infrastructure consumes energy, cooling, space, networking, and hardware materials even when much of its capacity is idle.

---

# 2. How cloud computing helps

Cloud providers operate infrastructure at very large scale and pool resources across many customers.

Virtualization allows physical infrastructure to support multiple virtual workloads instead of dedicating one entire server to one small application.

```text
Before virtualization

Physical Server A → App A → 15% utilization
Physical Server B → App B → 10% utilization
Physical Server C → App C → 20% utilization

After virtualization / pooling

Physical Infrastructure
      │
      ├── VM A
      ├── VM B
      └── VM C
```

This can improve overall infrastructure utilization.

The important sustainability connection is:

```text
Better utilization
      ↓
Less unnecessary infrastructure for the same workload
      ↓
Potentially lower energy and hardware consumption
```

Cloud does not make computing free of environmental impact. Datacenters still consume electricity, water, equipment, networking, and other resources.

The advantage is the opportunity to operate infrastructure more efficiently at scale.

---

# 3. Resource utilization is central to sustainability

Suppose an application runs on:

```text
VM
16 vCPU
64 GB RAM
```

but monitoring shows:

```text
Average CPU = 5%
Memory used = 8 GB
```

The machine may be significantly oversized.

If a smaller VM can safely meet performance and availability requirements, rightsizing can reduce:

```text
Unused compute capacity
Cost
Energy consumption associated with the workload
```

This connects sustainability directly with **cost optimization** and **predictability**.

Often the same change that reduces unnecessary cost also reduces unnecessary resource consumption.

---

# 4. Rightsizing

Rightsizing means selecting resource capacity that matches the actual workload requirements.

Example:

```text
Current VM
16 vCPU
64 GB RAM
CPU average = 8%

Measured requirement
4 vCPU
16 GB RAM

Possible action
Resize to a smaller suitable VM
```

But rightsizing must be based on evidence.

If we size only for the current average and ignore peaks, failures, or scaling time, we can create reliability problems.

A good rightsizing process is:

```text
Collect metrics
      ↓
Understand normal + peak demand
      ↓
Understand availability headroom
      ↓
Choose appropriate resource size
      ↓
Monitor after change
```

Sustainability should not come at the cost of making a critical application unreliable.

---

# 5. Elasticity reduces idle capacity

Elasticity is one of the strongest connections between cloud architecture and sustainability.

Suppose a web application needs:

```text
Normal traffic → 2 instances
Peak traffic   → 10 instances
```

Without elasticity, the company may keep all 10 instances running continuously.

```text
24 hours/day
10 instances running
Even when only 2 are needed
```

With autoscaling:

```text
Normal traffic
VM1 VM2

Traffic increases
VM1 VM2 VM3 VM4 VM5 VM6 VM7 VM8 VM9 VM10

Traffic decreases
VM1 VM2
```

Now extra capacity exists only when required.

This improves:

```text
Cost efficiency
Resource utilization
Operational efficiency
Sustainability
```

---

# 6. Scale In is as important as Scale Out

Teams often focus on scaling out when traffic increases.

But sustainability also depends on scaling **back in** when demand decreases.

Bad configuration:

```text
Traffic spike
2 VMs → 10 VMs

Traffic returns to normal
10 VMs remain running
```

Better configuration:

```text
Traffic spike
2 → 10

Traffic normalizes
10 → 2
```

If scale-in rules are missing or too conservative, cloud environments can accumulate unnecessary running resources.

---

# 7. Turn off resources that are not needed

Our own Azure VM lab demonstrated this principle.

When we were not using the VM, we **Stopped (deallocated)** it.

```text
Running VM
      ↓
Compute allocated
      ↓
Compute resources consumed

Stopped (deallocated)
      ↓
Compute allocation released
```

Persistent resources such as disks can still remain and may still incur charges, but the VM compute is no longer running.

In enterprise environments, this is particularly useful for non-production resources.

Example:

```text
Development environment
Needed: Monday–Friday, 8 AM–7 PM

Instead of:
24 × 7 running

Use:
Scheduled start/stop where appropriate
```

This can reduce unnecessary compute usage significantly.

---

# 8. Development and test environments

Production may need continuous availability.

Development and test environments often do not.

Imagine:

```text
DEV VM   → running 24/7
UAT VM   → running 24/7
TEST DB  → running 24/7
```

but engineers use them only during working hours.

Possible optimization:

```text
Working hours
Resources running

Nights/weekends
Resources stopped, scaled down, or removed where architecture allows
```

This is a practical sustainability improvement because non-production environments can otherwise consume a large amount of idle infrastructure.

---

# 9. Managed services can improve efficiency

Suppose a team needs to host a web application.

One option is:

```text
Azure VM
  ↓
Install OS
Patch OS
Install web server
Manage runtime
Monitor VM
Scale VM infrastructure
```

Another option may be a managed platform service such as Azure App Service, depending on the workload.

```text
Application code
      ↓
Managed platform
      ↓
Azure manages more infrastructure underneath
```

Managed services can allow the cloud provider to optimize infrastructure utilization at a larger scale and reduce the amount of infrastructure the customer must explicitly provision and maintain.

This does not mean every workload should use PaaS.

The service should be selected based on requirements such as:

```text
Control
Compatibility
Performance
Security
Cost
Availability
Operational responsibility
```

But sustainability is another factor worth considering.

---

# 10. Serverless and consumption-based models

For some workloads, running a server continuously is unnecessary.

Imagine a function that runs only when a file arrives.

Traditional approach:

```text
VM runs 24 hours/day
      ↓
Waits for file
      ↓
Processes file for 5 minutes
```

A consumption-based/serverless approach may instead allocate execution resources when events occur.

```text
No event
      ↓
Little/no dedicated application compute running

File arrives
      ↓
Function executes
      ↓
Work completes
```

This can improve utilization for intermittent workloads.

Again, serverless is not automatically the best answer for every application. It has its own service limits, execution model, performance characteristics, and pricing considerations.

The principle is to avoid running continuously provisioned resources when the workload does not require them.

---

# 11. Efficient application code matters

Cloud sustainability is not only an infrastructure problem.

Application design affects resource usage.

Suppose two implementations produce the same business result.

Implementation A:

```text
CPU time = 10 seconds
Memory   = 4 GB
```

Implementation B:

```text
CPU time = 2 seconds
Memory   = 1 GB
```

If both meet the same correctness and reliability requirements, the second implementation can process the same work with fewer compute resources.

Examples of software-efficiency improvements include:

```text
Efficient algorithms
Avoiding unnecessary loops/work
Caching where appropriate
Efficient database queries
Batching operations appropriately
Reducing repeated network calls
Using asynchronous processing where useful
```

Performance optimization and sustainability often reinforce each other.

---

# 12. Database efficiency

Databases can be major resource consumers.

Suppose an application repeatedly executes an inefficient query:

```text
Full table scan
Millions of rows
Every request
```

This can increase:

```text
CPU usage
Storage I/O
Memory usage
Response time
Required database size
```

Improving indexing/query design can reduce the resources needed to serve the same workload.

So sustainability must be considered across the whole application stack, not only the VM tier.

---

# 13. Storage sustainability

Data tends to accumulate.

Example:

```text
Year 1 → 10 TB
Year 2 → 30 TB
Year 3 → 80 TB
```

But not all data needs the same performance or retention period.

Organizations should understand:

```text
What data must be retained?
For how long?
How often is it accessed?
Can old data move to a lower-cost/lower-performance storage tier?
Can obsolete data be deleted according to policy?
```

Data lifecycle management can reduce unnecessary high-performance storage consumption.

This must always respect legal, regulatory, business, and recovery requirements.

---

# 14. Hot vs cool/archive-style storage thinking

Frequently accessed data may need higher-performance storage.

Older rarely accessed data may not.

Conceptually:

```text
Recent operational data
      ↓
Hot storage

Older infrequently accessed data
      ↓
Cooler tier

Long-term retention data
      ↓
Archive-style tier where appropriate
```

The exact Azure storage tier options depend on the storage service.

The sustainability principle is:

> Do not keep every byte forever in the highest-performance storage tier unless the workload requires it.

---

# 15. Remove unused resources

Cloud environments often accumulate abandoned resources.

Examples:

```text
Old disks
Unused public IPs
Test VMs
Old snapshots
Temporary storage
Unused load balancers
Forgotten development databases
```

Even if no one is using them, some of these resources can continue consuming infrastructure and generating cost.

A lifecycle process should identify and remove resources that are no longer required.

```text
Inventory
   ↓
Identify unused resource
   ↓
Confirm owner / retention need
   ↓
Backup if required
   ↓
Delete safely
```

Governance and tagging make this easier because resources have clear ownership.

---

# 16. Region selection and sustainability

Azure regions exist in different geographic locations and operate within different energy grids and infrastructure conditions.

When business, latency, compliance, resilience, and data-residency requirements allow it, region choice can be one factor in sustainability planning.

But region selection must never be based on sustainability alone.

Architecture must also consider:

```text
Latency to users
Data residency
Service availability
Availability Zones
Disaster Recovery
Pricing
Legal requirements
Networking
```

The correct region is the one that satisfies the complete set of business and technical requirements.

---

# 17. Carbon-aware workload scheduling

Some workloads do not need to run immediately.

Example:

```text
Nightly analytics job
Monthly report generation
Large batch transformation
Machine learning training
```

If a workload can be shifted in time or location without violating business requirements, organizations can consider scheduling work when infrastructure or energy conditions are more favorable.

This is often described as **carbon-aware computing**.

The idea is:

```text
Flexible workload
      ↓
Understand when/where it can run
      ↓
Schedule execution efficiently
```

This is more relevant to flexible batch workloads than latency-sensitive user transactions.

---

# 18. Data transfer also consumes resources

Architecture can create unnecessary data movement.

Suppose an application continually transfers large datasets between regions even though only a small subset is required.

```text
Region A
100 GB dataset
    │
    │ transfer repeatedly
    ▼
Region B
```

This consumes network capacity and can increase cost.

Better design may include:

```text
Move only required data
Compress data where appropriate
Process data closer to where it resides
Cache commonly used data
Avoid unnecessary cross-region chatter
```

This must be balanced with DR and availability requirements, which may legitimately require cross-region replication.

---

# 19. Sustainability vs Disaster Recovery

This creates an important trade-off.

For sustainability, we might prefer fewer duplicate resources.

For Disaster Recovery, we may deliberately maintain resources in another region.

```text
Primary Region
      +
Secondary Region
```

That duplication consumes additional resources, but it may be essential to meet RTO/RPO requirements.

The correct architecture is therefore not:

> Minimize resource consumption at all costs.

It is:

> **Use the minimum resources necessary to meet the required reliability, security, performance, and recovery objectives.**

This is one of the most important sustainability principles.

---

# 20. Sustainability vs High Availability

The same trade-off exists with High Availability.

One VM uses fewer resources than three VMs.

But a production application may require redundancy.

```text
One VM
Lower resource use
BUT
Single point of failure
```

```text
Multiple VMs across zones
More resource use
BUT
Higher availability
```

The sustainable design is not necessarily the smallest design.

It is the design that meets the availability requirement without unnecessary overprovisioning.

Example:

```text
Required minimum for HA = 2 instances
Peak capacity = 8 instances

Sustainable approach
Normal → 2
Peak → scale to 8
After peak → return to 2
```

---

# 21. Sustainability vs performance

Oversizing every resource creates waste.

Undersizing every resource causes poor performance.

A balanced architecture looks like:

```text
Measure workload
      ↓
Choose suitable baseline capacity
      ↓
Scale when demand increases
      ↓
Scale back when demand decreases
```

This combines:

```text
Performance
Cost efficiency
Sustainability
```

---

# 22. Sustainability and FinOps

FinOps focuses on understanding and optimizing cloud spending through collaboration between engineering, finance, and business teams.

Many FinOps practices also support sustainability because waste often appears both as unnecessary cost and unnecessary resource consumption.

Examples:

```text
Rightsizing
Removing idle resources
Scaling down non-production environments
Storage lifecycle management
Cost allocation
Utilization monitoring
```

Cost and carbon are not identical measurements, but reducing obvious infrastructure waste often helps both.

---

# 23. Sustainability and monitoring

We cannot optimize what we do not measure.

Useful operational signals include:

```text
CPU utilization
Memory utilization
Storage growth
Network usage
Instance count
Database load
Idle resource time
Autoscale activity
Cost trends
```

The optimization loop becomes:

```text
Measure
   ↓
Identify waste / inefficiency
   ↓
Optimize architecture
   ↓
Measure again
```

Sustainability is therefore a continuous operational process, not a one-time architecture decision.

---

# 24. Example — oversized production environment

Suppose a production application runs:

```text
10 VMs
8 vCPU each
```

Monitoring over one month shows:

```text
Average CPU = 12%
Peak CPU = 40%
```

Possible investigation:

```text
Do we need 10 instances for availability?
Could instance size be reduced?
Could minimum count be lower while maintaining HA?
Could autoscaling handle peaks?
Are all instances actually receiving traffic?
```

After testing, the architecture may become:

```text
Normal
2 smaller instances

Peak
Autoscale to 6
```

This could reduce resource consumption without sacrificing the business requirement.

The key is measurement and testing before changing production capacity.

---

# 25. Example — batch processing

A company processes a large file once every night.

Bad design:

```text
Large VM
Runs 24/7
Job uses it 1 hour/night
```

Possible better design:

```text
Scheduled/consumption-based compute
      ↓
Start when job begins
      ↓
Process file
      ↓
Release compute afterward
```

This matches infrastructure consumption to actual workload duration.

---

# 26. Example — data lifecycle

Application logs grow continuously:

```text
1 TB
5 TB
20 TB
100 TB
```

But operational teams only search the newest 30 days frequently.

A lifecycle strategy could be:

```text
Recent logs → higher-performance/searchable tier
Older logs → lower-cost tier
Expired logs → delete according to retention policy
```

This reduces unnecessary high-performance storage while preserving required retention.

---

# 27. Azure Well-Architected thinking

Sustainability should not be considered separately from the other architecture qualities we have studied.

An enterprise architect must balance:

```text
Reliability
Security
Cost optimization
Operational excellence
Performance efficiency
Sustainability
```

An optimization in one area can affect another.

Example:

```text
Reduce VM count aggressively
      ↓
Better utilization
      ↓
BUT possibly insufficient failure capacity
```

Architecture decisions must therefore be evaluated as trade-offs.

---

# 28. Sustainable architecture decision flow

A useful decision process is:

```text
What business outcome is required?
        ↓
What reliability/security/performance requirements exist?
        ↓
What resources are actually required?
        ↓
Can we use managed/consumption-based services?
        ↓
Can capacity scale with demand?
        ↓
Can non-production resources be stopped when unused?
        ↓
Can data lifecycle reduce unnecessary storage?
        ↓
Can we eliminate idle resources?
        ↓
Measure and continuously optimize
```

---

# 29. Common misunderstandings

## “Cloud is automatically sustainable.”

No.

Cloud can provide more efficient infrastructure and scaling capabilities, but a badly designed cloud environment can still waste enormous resources.

## “Sustainability means choosing the smallest VM.”

No.

The VM must still meet performance and reliability requirements. Sustainability means avoiding unnecessary capacity, not creating underpowered systems.

## “High Availability is unsustainable because it duplicates resources.”

Not necessarily.

Redundancy may be essential. The goal is to meet the required availability with efficient capacity and scaling.

## “Stopping a VM removes all cost.”

No.

Deallocating stops VM compute billing, but persistent disks and some other resources can still incur charges.

## “Deleting old data is always sustainable.”

Data can only be deleted according to business, legal, regulatory, security, and recovery requirements.

## “Sustainability is only Microsoft's responsibility.”

No.

Microsoft can optimize Azure infrastructure, but customers influence workload efficiency through architecture, resource sizing, scaling, software design, storage retention, and operational practices.

---

# 30. Connecting all cloud foundation concepts

Sustainability is not isolated from the concepts we learned earlier.

```text
Virtualization
      ↓
Better physical-resource utilization

Scalability + Elasticity
      ↓
Capacity follows demand

Monitoring + Predictability
      ↓
Identify overprovisioning and waste

Manageability + Automation
      ↓
Start/stop/resize/clean resources consistently

Governance
      ↓
Enforce standards and ownership

Reliability / HA / DR
      ↓
Use enough redundancy to meet requirements — but avoid unnecessary duplication

Sustainability
      ↓
Deliver the required outcome efficiently
```

---

# 31. Final mental model

```text
                         BUSINESS REQUIREMENT
                                 │
                                 ▼
                     Required Reliability / Security
                         / Performance / DR
                                 │
                                 ▼
                       Choose Required Capacity
                                 │
                  ┌──────────────┼──────────────┐
                  │              │              │
                  ▼              ▼              ▼
             Right-size      Autoscale       Managed / Serverless
                  │              │              │
                  └──────────────┼──────────────┘
                                 ▼
                     Monitor Actual Utilization
                                 │
                  ┌──────────────┼──────────────┐
                  │              │              │
                  ▼              ▼              ▼
             Remove Idle    Optimize Data   Efficient Code
             Resources       Lifecycle       / Queries
                  │              │              │
                  └──────────────┼──────────────┘
                                 ▼
                       CONTINUOUS OPTIMIZATION
                                 │
                                 ▼
                         SUSTAINABLE CLOUD USE
```

The simplest way to remember sustainability is:

> **Use only the computing resources necessary to meet the required business outcome, and continuously remove waste without sacrificing reliability, security, performance, or recovery requirements.**

---

# 32. Foundation checkpoint

At this point, our cloud-foundation journey includes:

```text
Why Cloud Computing Exists
        ↓
Virtualization / Hypervisor / Isolation
        ↓
High Availability / Failure Boundaries / Zones
        ↓
Disaster Recovery / RTO / RPO / Replication / Backup
        ↓
Scalability / Elasticity / Load Balancing
        ↓
Reliability / Resiliency / Fault Tolerance
        ↓
Predictability / Manageability
        ↓
Governance / Compliance
        ↓
Sustainability
```

Before starting the next major Azure service/module, we should review this foundation list against everything we discussed and make sure no foundation topic or subtopic is missing.