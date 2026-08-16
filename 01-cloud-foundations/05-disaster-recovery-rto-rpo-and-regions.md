# Disaster Recovery, RTO, RPO, and Regions

High availability handles many local failures, but enterprises must also consider larger disasters.

Examples include:

- Major datacenter outage
- Regional infrastructure outage
- Natural disaster
- Large network disruption
- Severe operational failure

This leads to **Disaster Recovery (DR)**.

## High availability vs disaster recovery

They solve related but different problems.

```text
High Availability
→ Keep the service running through expected/local failures

Disaster Recovery
→ Recover the service after a major failure affecting the primary environment
```

A system may use both.

## Primary and recovery environments

A simple DR design could look like:

```text
Users
  │
  ▼
Primary Azure Region
  │
  ├── Application
  └── Database
          │
          │ Replication / backup
          ▼
Secondary Azure Region
  │
  ├── Recovery Application
  └── Recovery Data
```

If the primary region becomes unavailable, the organization follows its recovery/failover process to restore service from the secondary environment.

The exact process depends on the Azure services and architecture.

## RTO — Recovery Time Objective

RTO answers:

> After a disruption, how long can the business tolerate the service being unavailable?

Example:

```text
Failure: 2:00 PM
RTO: 5 minutes

Target:
Service restored by approximately 2:05 PM
```

A lower RTO usually requires faster failover mechanisms and more prepared infrastructure.

## RPO — Recovery Point Objective

RPO answers:

> How much recent data can the business tolerate losing?

Example:

```text
RPO = 30 seconds
```

If a disaster happens at 2:00:00 PM, the recovery design aims to avoid losing more than approximately the most recent 30 seconds of data.

RPO is about **data loss tolerance**, not downtime.

## RTO vs RPO

```text
RTO → TIME to restore service
RPO → DATA AGE / amount of recent data loss tolerated
```

These are business requirements before they are technical settings.

The business might say:

```text
RTO = 5 minutes
RPO = 30 seconds
```

Architects then design a solution capable of meeting those objectives.

## Example timeline

Suppose replication last completed at:

```text
1:59:30 PM
```

A new transaction occurs at:

```text
1:59:50 PM
```

The primary environment fails at:

```text
2:00:00 PM
```

If the transaction had not reached the recovery environment before the failure, it may not exist in the recovered copy.

That illustrates why replication frequency and architecture influence achievable RPO.

## Does replication replace backups?

No.

Replication and backup solve different problems.

If bad data or an accidental deletion is replicated to the secondary environment, the secondary copy may also contain that bad state.

Backups can provide historical recovery points.

A mature data-protection strategy may use both replication and backups.

## Regions and paired-region thinking

Azure has geographic regions around the world. Enterprises can place resources in more than one region when their resilience requirements justify it.

Conceptually:

```text
Region A
┌─────────────────┐
│ Production      │
│ App + Data      │
└────────┬────────┘
         │
         │ Replication / backup
         ▼
Region B
┌─────────────────┐
│ DR Environment  │
│ App + Data      │
└─────────────────┘
```

Not every Azure service implements regional recovery in the same way. Some services provide native replication options; others require application-level architecture.

Therefore we should never assume a universal RTO/RPO just because a workload runs in Azure.

## Cost trade-off

Stronger DR normally costs more.

Compare:

```text
Backup only
→ Lower cost
→ Longer recovery

Warm recovery environment
→ More cost
→ Faster recovery

Fully active multi-region architecture
→ Highest complexity/cost
→ Potentially very fast failover
```

The correct design depends on business impact.

## Key takeaway

Do not choose a DR architecture first and then invent the RTO/RPO.

The better order is:

```text
Business impact analysis
        ↓
Define RTO and RPO
        ↓
Choose architecture
        ↓
Implement replication/backups/failover
        ↓
Test recovery regularly
```

A recovery plan that has never been tested should not automatically be assumed to work.