# Disaster Recovery — Recovering After a Major Failure

High Availability protects an application against many localized failures such as a VM failure, host failure, maintenance event, or zone-level problem when the architecture has enough redundancy.

But eventually we reach a larger failure boundary.

Suppose the application is already distributed across multiple Availability Zones in one Azure region:

```text
                         Azure Region A
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
      Zone 1                Zone 2                Zone 3
        │                     │                     │
       VM1                   VM2                   VM3
```

This can protect against a zone-level failure.

But all three zones still belong to the same Azure region.

If the entire region becomes unavailable because of a major regional incident, every application instance in that region can be affected.

```text
Azure Region A ❌
├── Zone 1 ❌
├── Zone 2 ❌
└── Zone 3 ❌
```

Now the problem is no longer:

> Which VM should receive the traffic?

The problem becomes:

> **Where will the application run if the entire primary environment becomes unavailable?**

That is the problem **Disaster Recovery (DR)** solves.

---

# High Availability and Disaster Recovery solve different failure scopes

High Availability and Disaster Recovery are related, but they are not the same thing.

A simple way to separate them is by failure scope.

```text
Single VM / host / rack / maintenance failure
               ↓
        High Availability

Availability Zone failure
               ↓
    Zone-resilient High Availability

Entire primary region / major disaster
               ↓
        Disaster Recovery
```

High Availability tries to keep the service running through expected component failures with minimal interruption.

Disaster Recovery focuses on restoring or continuing the service when a larger disaster affects the primary environment.

A production system may require both.

---

# The basic Disaster Recovery architecture

A simplified multi-region design can look like this:

```text
                         USERS
                           │
                           ▼
                    Primary Region A
               ┌────────────────────────┐
               │                        │
               │ Application            │
               │ Database               │
               │ Storage                │
               │ Networking             │
               │                        │
               └───────────┬────────────┘
                           │
                           │ Replication / backup / synchronization
                           ▼
                  Secondary Region B
               ┌────────────────────────┐
               │                        │
               │ Recovery Application   │
               │ Recovery Data          │
               │ Recovery Networking    │
               │                        │
               └────────────────────────┘
```

Region A is the **primary environment**.

Region B is the **recovery environment**.

If Region A becomes unavailable, the organization performs a recovery or failover process and directs service to Region B.

The exact method depends on the Azure services used. Databases, storage systems, VMs, application platforms, DNS, networking and identity components can all have different replication and failover capabilities.

So DR must be designed **service by service and dependency by dependency**.

---

# DR is not just “copy everything to another region”

A real application has many components.

For example:

```text
Users
  │
  ▼
DNS / Traffic Entry
  │
  ▼
Application Tier
  │
  ▼
Database
  │
  ├── Storage
  ├── Messaging
  └── External dependencies
```

If only the database is replicated to another region but the application cannot start there, recovery is incomplete.

If the application exists in Region B but the DNS still points only to Region A, users may still not reach it.

If the application and database recover but required secrets, identities, networking rules or storage are missing, the service can still fail.

So Disaster Recovery is an **end-to-end application recovery design**, not just a database feature.

---

# Before designing DR, the business must define tolerance

Not every application needs the same recovery speed.

A public payment platform may have very strict requirements.

A development environment may tolerate hours of downtime.

A monthly reporting system may tolerate even longer recovery.

So architects need two business-defined recovery objectives:

```text
RTO — Recovery Time Objective
RPO — Recovery Point Objective
```

These values describe what the business can tolerate.

They are not automatically assigned by Azure just because a resource is in the cloud.

---

# RTO — Recovery Time Objective

**RTO answers:**

> After a disruption, how long can the service remain unavailable before it must be restored?

Suppose the application fails at:

```text
2:00 PM
```

and the business requirement says:

```text
RTO = 5 minutes
```

The target is approximately:

```text
2:00 PM  Failure
   │
   │  Recovery activities
   ▼
2:05 PM  Service restored
```

The five minutes include the recovery work needed to make the service usable again.

Depending on the architecture, that may include:

```text
Detect failure
Decide whether to fail over
Start or activate recovery resources
Promote a secondary database
Update routing / DNS / traffic management
Start application services
Run health checks
Allow users back in
```

RTO is therefore mainly about **downtime tolerance**.

A lower RTO generally requires more automation and more recovery infrastructure already prepared before the disaster occurs.

---

# RPO — Recovery Point Objective

**RPO answers:**

> How much recent data can the business tolerate losing during recovery?

Suppose:

```text
RPO = 30 seconds
```

That does not mean recovery finishes in 30 seconds.

It means the recovery design should target a recovery point that is no more than approximately 30 seconds behind the primary data at the time of failure.

For example:

```text
1:59:00  Transaction A
1:59:20  Transaction B
1:59:50  Transaction C
2:00:00  Primary region fails
```

If the recovery environment contains data only through:

```text
1:59:30
```

then Transaction C may not exist in the recovered copy.

That represents approximately a 30-second recovery-point gap.

RPO is therefore about **data-loss tolerance**, not service downtime.

---

# RTO and RPO are independent dimensions

It is possible to have:

```text
RTO = 5 minutes
RPO = 30 seconds
```

This means:

- the application should be restored within about 5 minutes;
- the recovered data should ideally be no more than about 30 seconds behind the primary state.

They measure different things.

```text
RTO
│
└── How long can the service be down?

RPO
│
└── How much recent data can be lost?
```

A solution can have a low RPO but a long RTO.

Example:

```text
Data continuously replicated to Region B
Recovery data is only seconds behind
BUT
Application servers must be manually rebuilt

Result:
Low RPO
High RTO
```

The opposite can also happen.

```text
Recovery application is always running
Traffic can switch quickly
BUT
Data is restored from a backup taken one hour ago

Result:
Low RTO
High RPO
```

The architecture must satisfy both objectives independently.

---

# The transaction example — what happens if the last transaction was not replicated?

This was one of the most important questions in our discussion.

Suppose:

```text
1:59:50  Customer completes a transaction
1:59:55  Transaction exists in the primary database
1:59:58  Region A fails
2:00:00  Secondary region takes over
```

If that transaction had **not yet been replicated** to Region B before Region A became unavailable, the secondary system cannot magically recreate it just because the RPO is 30 seconds.

RPO is a **target/tolerance**, not a recovery mechanism.

The transaction can be missing from the recovery environment.

```text
Primary Region A
Transaction exists ✅
       │
       │ replication not completed
       X
       │
Secondary Region B
Transaction absent ❌
```

If Region A later comes back, whether that transaction can be recovered and synchronized depends on the specific data platform, replication technology, failover process and whether the original primary data is still available and consistent.

There is no universal Azure behavior saying:

> The old primary will always return later and automatically send every missing transaction to the new primary.

Some technologies support resynchronization or catch-up. Others may require reconciliation. In some disaster scenarios the original primary may be permanently lost.

This is why applications with very strict data-loss requirements use replication technologies and architectures designed for very low RPO.

---

# Does RPO “save” an unreplicated transaction?

No.

Suppose:

```text
RPO = 30 seconds
```

and a transaction happened 10 seconds before the disaster but was not replicated.

That transaction may still be lost from the recovered environment.

RPO means the business has stated that up to roughly the defined recovery-point window may be acceptable under the designed disaster scenario.

It does not provide a hidden backup of every transaction.

A better way to think about RPO is:

```text
Business says:
“We can tolerate at most X amount of recent data loss.”

Architect says:
“Then we need replication / logging / recovery technology capable of meeting that objective.”
```

---

# Replication

Replication copies data changes from a primary system to another location or system.

A simplified flow is:

```text
Primary Database
      │
      │ Change replication
      ▼
Secondary Database
```

Replication can be:

```text
Synchronous
Asynchronous
Service-specific variations
```

The exact behavior depends on the Azure service.

The key concept is that replication keeps another copy relatively current so recovery does not have to start entirely from an old backup.

---

# Synchronous replication

With synchronous replication, the primary operation generally waits for confirmation from the required replica before the transaction is considered fully committed according to that system's replication design.

Conceptually:

```text
Application
    │
    ▼
Primary
    │
    ├── Write locally
    │
    └── Send to replica
             │
             ▼
         Secondary
             │
             ▼
        Acknowledge
             │
             ▼
       Commit completes
```

This can reduce data-loss risk because the replica is kept very close to the primary state.

But it introduces a latency trade-off because the primary may need to wait for the replica acknowledgment.

Long-distance cross-region synchronous replication can therefore be difficult for latency-sensitive workloads.

---

# Asynchronous replication

With asynchronous replication, the primary can commit a transaction before the secondary has fully received the change.

```text
Application
    │
    ▼
Primary
    │
    ├── Commit transaction ✅
    │
    └──────── later ────────► Secondary
```

This improves performance and works better over long distances, but introduces a **replication lag window**.

If the primary fails before the latest changes reach the secondary:

```text
Primary has latest transaction ✅
Secondary has not received it yet ❌
```

those recent changes may be missing after failover.

That replication lag directly affects achievable RPO.

---

# Replication and backup solve different problems

Replication is not the same as backup.

Imagine someone accidentally runs:

```text
DELETE FROM customers;
```

The primary database applies the delete.

Replication then does exactly what it is supposed to do:

```text
Primary
Customers deleted
      │
      │ replicate change
      ▼
Secondary
Customers deleted too
```

Replication successfully copied the bad change.

So the secondary is not necessarily a clean historical copy.

A **backup** provides recovery points from earlier states.

```text
Database
│
├── Backup 10:00 AM
├── Backup 11:00 AM
├── Backup 12:00 PM
└── Current state
```

If corruption or accidental deletion occurs, a backup may allow recovery to a point before the problem.

So mature recovery designs often use both:

```text
Replication
    ↓
Fast availability / recent recovery copy

Backup
    ↓
Historical recovery / corruption protection / point-in-time recovery
```

---

# Where should backups be stored?

A backup should not blindly depend on the same failure boundary as the production system it protects.

Suppose production and all backups are stored only inside the same location:

```text
Region A
├── Production Database
└── Backup copy
```

If the failure destroys access to both, the backup is not useful during that disaster.

A stronger protection strategy uses service capabilities that provide appropriate redundancy and/or geographic separation based on the recovery requirement.

The exact backup storage model depends on the Azure service. Some Azure services provide built-in local, zone or geo-redundant backup/storage options; others require separate configuration.

So the correct question is not simply:

> Is the backup in another database?

The better questions are:

```text
What service stores the backup?
What failure boundary does it survive?
Is it in the same zone?
Same region?
Geo-redundant?
How quickly can it be restored?
How long is it retained?
```

---

# Service-specific recovery objectives

RTO and RPO are usually defined for a **business service or application**, but every component underneath the application can have its own recovery characteristics.

Suppose an application contains:

```text
Web Tier
API Tier
Database
Storage
Messaging
Identity
```

The business may say:

```text
Application RTO = 5 minutes
Application RPO = 30 seconds
```

But each underlying service must be capable of supporting that overall target.

For example:

```text
Web Tier recovery        = 2 minutes
Database failover        = 3 minutes
Messaging recovery       = 15 minutes
```

Even though the web and database recover quickly, the complete application may not meet a 5-minute RTO if the messaging component requires 15 minutes.

So application RTO is constrained by critical dependencies.

The same applies to RPO.

If the database supports a 30-second RPO but another critical data store only restores from hourly backups, the whole application may not truly meet a 30-second data-loss objective.

---

# Recovery patterns — cold, warm, hot, active-active

Disaster Recovery designs usually trade cost against recovery speed.

## Cold recovery

In a cold design, the secondary environment is minimal or not running until a disaster happens.

```text
Region A
Production running ✅

Region B
Backups / templates / minimal resources
```

During disaster:

```text
Restore data
Create/start infrastructure
Deploy application
Configure networking
Redirect users
```

Advantages:

```text
Lower steady-state cost
```

Trade-off:

```text
Longer RTO
```

---

# Warm standby

A warm environment keeps some recovery infrastructure prepared.

```text
Region A
Full production capacity

Region B
Reduced application capacity
Replicated data
Networking prepared
```

During disaster:

```text
Promote secondary data
Scale recovery application
Redirect traffic
```

This costs more than cold recovery but can significantly reduce RTO.

---

# Hot standby / active-passive

In a hot standby architecture, Region B is already running and ready to take traffic but normally remains passive or receives little traffic.

```text
Region A — Active
Region B — Hot Standby
```

Failover can be much faster because resources are already running.

The trade-off is higher cost because substantial infrastructure exists in both regions.

---

# Active-active multi-region

In an active-active design, both regions can serve production traffic.

```text
                    Global Traffic Layer
                      /             \
                     ▼               ▼
                Region A          Region B
                 Active            Active
```

If Region A fails:

```text
Region A ❌
Region B ✅
   │
   ▼
Region B handles remaining traffic
```

This can provide very low failover time, but it is much more complex.

Challenges include:

```text
Data consistency
Cross-region writes
Conflict handling
Latency
Global traffic routing
Capacity planning
Cost
Application state management
```

Active-active should therefore be used because the business requirement justifies it, not because it sounds more advanced.

---

# Failover

**Failover** is the process of switching service from the primary environment to the recovery environment.

Conceptually:

```text
Before failure

Users
  │
  ▼
Region A — Primary ✅
Region B — Secondary

After failure

Region A ❌
   │
   ▼
Failover
   │
   ▼
Region B — Active ✅
```

Failover may involve:

```text
Promoting secondary data
Starting/scaling application resources
Changing traffic routing
Updating DNS or global routing
Validating dependencies
Health testing
Opening service to users
```

Automated failover can reduce RTO, but automatic failover must be designed carefully to avoid incorrect switching during temporary or partial failures.

---

# Failback

Eventually the original primary region may recover.

But we should not simply switch traffic back immediately.

While Region B was active, users may have created new data there.

```text
Region B active during disaster
      │
      ├── New Transaction 1
      ├── New Transaction 2
      └── New Transaction 3
```

Before returning to Region A, the environments may need to be synchronized.

```text
Region B — current production data
       │
       │ resynchronize
       ▼
Region A — recovered environment
       │
       ▼
Validate
       │
       ▼
Failback
```

The exact process depends on the data service and architecture.

This is called **failback**.

Failover and failback are therefore different operations:

```text
Failover
Primary → Recovery environment

Failback
Recovery environment → Preferred/primary environment after recovery
```

---

# What happens to the old primary after failover?

This depends on the failure and the technology.

Possible outcomes include:

```text
Old primary completely lost
Old primary returns but is stale
Old primary returns and can resynchronize
Old primary must be rebuilt
Old primary becomes the new secondary
```

A safe DR design avoids allowing both sides to independently accept conflicting writes unless the system is specifically designed for multi-primary operation.

Otherwise a **split-brain** situation can occur where both environments believe they are primary.

That can create conflicting data and serious recovery problems.

---

# Why recovery testing matters

A DR plan that exists only in documentation should not automatically be trusted.

Real recovery requires many components to work together:

```text
Backups must be valid
Replication must be healthy
Permissions must work
Secrets must exist
Networking must be configured
DNS/traffic switching must work
Application deployment must succeed
Database promotion must work
Staff must know the procedure
```

So organizations perform DR exercises and recovery tests.

A practical recovery process looks like:

```text
Design recovery
      ↓
Document procedure
      ↓
Test failover
      ↓
Measure actual RTO/RPO
      ↓
Find gaps
      ↓
Improve design
      ↓
Test again
```

The target RTO/RPO and the actual tested recovery performance are not necessarily the same.

---

# A complete timeline example

Consider this business requirement:

```text
RTO = 5 minutes
RPO = 30 seconds
```

Normal operation:

```text
1:59:00  Transaction A committed
1:59:20  Transaction B committed
1:59:50  Transaction C committed
2:00:00  Region A fails
```

Assume the secondary contains data through:

```text
1:59:40
```

Then:

```text
Transaction A ✅ available
Transaction B ✅ available
Transaction C ❌ possibly missing
```

The recovery point is approximately 20 seconds behind the failure time, so this example is still within a 30-second RPO target.

Then the recovery process begins:

```text
2:00:00 Failure detected
2:00:30 Disaster confirmed
2:01:00 Secondary database promoted
2:02:00 Application recovery resources ready
2:03:00 Traffic redirected
2:04:00 Health validation completes
2:04:30 Users can access service
```

Actual recovery time:

```text
4 minutes 30 seconds
```

This is within the 5-minute RTO target.

This example clearly separates:

```text
Data recovery point → RPO
Service restoration duration → RTO
```

---

# A realistic enterprise architecture

A simplified enterprise DR architecture could look like:

```text
                           GLOBAL USERS
                                │
                                ▼
                     Global Traffic / DNS Layer
                                │
                ┌───────────────┴───────────────┐
                │                               │
                ▼                               ▼

        PRIMARY REGION A                RECOVERY REGION B
      ┌──────────────────┐            ┌──────────────────┐
      │ Application Tier │            │ Recovery App Tier│
      │                  │            │                  │
      │ API / Services   │            │ API / Services   │
      └────────┬─────────┘            └────────┬─────────┘
               │                               │
               ▼                               ▼
      ┌──────────────────┐   Replication   ┌──────────────────┐
      │ Primary Database │ ───────────────►│ Secondary Data   │
      └──────────────────┘                 └──────────────────┘
               │
               │ Backup
               ▼
        Backup / Recovery Store
```

During normal operation, users are served from Region A.

During a regional disaster:

```text
Region A unavailable
        ↓
Promote recovery data
        ↓
Activate/scale Region B application
        ↓
Switch global traffic
        ↓
Validate service
        ↓
Users served from Region B
```

This is the general DR flow. The exact Azure services used for each box depend on the application.

---

# Cost and recovery trade-offs

Faster recovery usually costs more.

```text
Backup only
    ↓
Lowest standby cost
Longer recovery

Warm standby
    ↓
Moderate standby cost
Faster recovery

Hot standby
    ↓
High standby cost
Very fast recovery

Active-active
    ↓
Highest complexity and often highest cost
Potentially minimal failover interruption
```

This is why RTO and RPO should be determined **before** choosing a DR design.

If the business can tolerate a 24-hour outage, maintaining a full active-active architecture may be unnecessary.

If the business loses millions of dollars per minute of downtime, a cold recovery process may be unacceptable.

---

# The correct order for DR design

A good Disaster Recovery process begins with business impact, not Azure features.

```text
Understand business impact
        ↓
Identify critical services and dependencies
        ↓
Define RTO
        ↓
Define RPO
        ↓
Choose recovery pattern
        ↓
Design replication + backups
        ↓
Design application/network failover
        ↓
Document runbook
        ↓
Test recovery
        ↓
Measure actual recovery
        ↓
Improve continuously
```

This is the core idea behind Disaster Recovery.

---

# Where this leads next

We now know how to handle failure across increasingly large boundaries:

```text
VM / host failure
      ↓
High Availability

Zone failure
      ↓
Availability Zones

Regional disaster
      ↓
Disaster Recovery
      ↓
RTO / RPO
      ↓
Replication + Backup
      ↓
Failover / Failback
```

The next major cloud problem is different.

Nothing has failed — the application is healthy — but traffic suddenly increases beyond the capacity of the current resources.

That leads to:

# Scalability, Elasticity and Load Balancing
