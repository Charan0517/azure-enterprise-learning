# Virtualization, Hypervisor, and VM Isolation

The cloud problem did not end after Microsoft built large datacenters. A new problem appeared immediately.

Suppose one physical server has:

```text
64 CPU cores
256 GB RAM
Several TB of storage
High-speed network connectivity
```

A customer may need only:

```text
2 vCPU
8 GB RAM
Ubuntu
```

Giving that entire physical server to one small customer workload