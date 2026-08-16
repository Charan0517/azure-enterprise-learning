# Scalability, Elasticity, and Load Balancing

These concepts solve the demand problem discussed in the cloud-computing notes.

Suppose an application normally handles 1,000 users but occasionally receives much more traffic.

## Scalability

Scalability is the ability of a system to increase or decrease its capacity as workload changes.

There are two major approaches.

### Vertical scaling — scale up / scale down

Increase the power of one machine.

```text
Before
VM: 2 vCPU / 8 GB RAM

        ↓ Scale Up

After
VM: 8 vCPU / 32 GB RAM
```

Scale down means reducing its size when that capacity is no longer required.

Vertical scaling is simple, but every machine has a maximum possible size and one machine can still represent a failure boundary.

### Horizontal scaling — scale out / scale in

Instead of making one machine larger, add more machines.

```text
Before

Users → VM1

After scale-out

             ┌→ VM1
Users → LB ──┼→ VM2
             └→ VM3
```

Removing machines when demand decreases is called **scale in**.

Horizontal scaling is particularly useful for web applications and distributed workloads.

## Elasticity

Elasticity is closely related to scalability, but emphasizes dynamically matching capacity to demand.

```text
Traffic rises
    ↓
Add capacity
    ↓
Traffic falls
    ↓
Remove unnecessary capacity
```

A useful distinction is:

> Scalability means the system **can** change capacity. Elasticity means capacity can **adapt to changing demand**, often automatically.

This helps reduce both overload and unnecessary cost.

## Why a load balancer is needed

If we scale from one application server to several, incoming requests need to be distributed between them.

A load balancer sits in front of the servers.

```text
                    ┌── VM1
Internet → Load Balancer ── VM2
                    └── VM3
```

Instead of users deciding which VM to contact, they connect to the load-balancing endpoint.

The load balancer distributes requests to healthy backend instances.

## Health checks

A load balancer should avoid sending traffic to a failed instance.

```text
VM1 → Healthy ✓
VM2 → Failed  ✗
VM3 → Healthy ✓
```

Traffic can continue to VM1 and VM3 while VM2 is unhealthy.

This demonstrates an important relationship:

```text
Horizontal Scaling
        +
Load Balancing
        +
Health Monitoring
        ↓
Better availability and scalability
```

## Real-world example

An online shopping application normally uses two application instances.

During a major sale:

```text
Normal
Users → Load Balancer → 2 instances

Sale begins
Traffic ↑

Scale out
Users → Load Balancer → 6 instances

Sale ends
Traffic ↓

Scale in
Users → Load Balancer → 2 instances
```

The business does not need to permanently operate six instances just because six are needed during a temporary peak.

## Important Azure connection

Later we will connect these ideas to Azure services such as:

- Virtual Machine Scale Sets
- Azure Load Balancer
- Application Gateway
- Azure Monitor and autoscale

For now, understand the architecture rather than memorizing product names.

## Key takeaway

```text
Vertical scaling   → Make a machine bigger/smaller
Horizontal scaling → Add/remove machines
Elasticity         → Match resources to changing demand
Load balancing     → Distribute requests across instances
```

These are different concepts, but they frequently work together in cloud architecture.