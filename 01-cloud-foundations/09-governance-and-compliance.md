# Governance and Compliance in Azure

As cloud environments grow, the problem changes again.

At first, one person may create a few resources manually. Later, an enterprise may have many teams, subscriptions, applications, regions, environments, budgets, and security requirements.

Without governance, different teams can make completely different choices:

```text
Team A → creates resources only in approved regions
Team B → creates resources anywhere
Team C → applies tags
Team D → does not
Team E → gives broad access
Team F → locks production resources
```

The cloud is still working, but the organization is losing control.

That introduces **governance**.

---

# 1. What governance means

Governance is the set of rules, structures, responsibilities, and technical controls used to make sure cloud resources are created and operated according to organizational requirements.

Governance helps answer questions such as:

```text
Who can create resources?
Where can resources be created?
Which resource types are allowed?
How should resources be named and tagged?
How are costs assigned?
How are production resources protected?
How are security requirements enforced?
How do we prove compliance?
```

Governance is not one Azure service.

It is implemented through several capabilities working together.

---

# 2. Why governance becomes necessary

Imagine a company with:

```text
10 application teams
3 environments each
DEV / UAT / PROD
Multiple subscriptions
Multiple regions
Hundreds of resources
```

If every team manages Azure independently, common problems appear:

```text
Inconsistent naming
Unknown resource ownership
Unexpected costs
Resources deployed in unapproved regions
Security settings differ by team
Production resources deleted accidentally
Too many users have administrator rights
Compliance evidence is difficult to collect
```

Governance provides a common control model across the organization.

---

# 3. Azure hierarchy — where governance is applied

Azure governance depends heavily on scope.

A simplified hierarchy is:

```text
Microsoft Entra Tenant
        │
        ▼
Management Groups
        │
        ▼
Subscriptions
        │
        ▼
Resource Groups
        │
        ▼
Resources
```

Each level has a different purpose.

---

# 4. Microsoft Entra tenant

The tenant is the identity boundary that contains identities such as:

```text
Users
Groups
Applications / service principals
Managed identities
```

The tenant answers the identity question:

> Who exists in this organization and can potentially be authenticated?

It is not the same thing as a subscription.

A tenant can contain multiple subscriptions.

```text
Tenant
│
├── Subscription A
├── Subscription B
└── Subscription C
```

This distinction is important because identity and resource ownership are related but separate concerns.

---

# 5. Subscription

A subscription is a major Azure resource-management, billing, quota, and access boundary.

Resources are created inside a subscription.

An enterprise may separate subscriptions by:

```text
Environment
Business unit
Application portfolio
Security boundary
Billing ownership
Regulatory requirement
```

Example:

```text
Tenant
│
├── Production Subscription
├── Non-Production Subscription
└── Shared Services Subscription
```

A subscription is not a folder inside a resource group.

The hierarchy is the opposite:

```text
Subscription
    ↓
Resource Groups
    ↓
Resources
```

---

# 6. Resource Groups

A Resource Group is a logical container for Azure resources that usually share a lifecycle, application, environment, or management context.

Example:

```text
Production Subscription
        │
        ▼
rg-payments-prod
        │
        ├── Virtual Network
        ├── Application resources
        ├── Database
        └── Monitoring resources
```

Resource Groups help with:

```text
Organization
RBAC scope
Policy scope
Cost grouping
Lifecycle operations
```

Deleting a Resource Group deletes the resources contained in it, so resources should be grouped thoughtfully around lifecycle requirements.

---

# 7. Management Groups

Now imagine the organization has 50 subscriptions.

Applying the same governance settings to each subscription one by one becomes difficult.

Management Groups solve that problem.

```text
Tenant
  │
  ▼
Management Group
  │
  ├── Subscription A
  ├── Subscription B
  └── Subscription C
```

Management Groups let organizations organize subscriptions into a hierarchy and apply governance at a higher scope.

For example:

```text
Tenant Root
│
├── Platform
│    ├── Identity Subscription
│    └── Connectivity Subscription
│
├── Production
│    ├── Prod Subscription A
│    └── Prod Subscription B
│
└── Non-Production
     ├── Dev Subscription
     └── Test Subscription
```

This allows different policy and access strategies for different parts of the organization.

---

# 8. Scope and inheritance

A major governance concept is **inheritance**.

If a policy or role assignment is applied at a higher scope, lower scopes can inherit it depending on the control.

Conceptually:

```text
Management Group
      │
      ├── Subscription
      │      │
      │      ├── Resource Group
      │      │       │
      │      │       └── Resource
```

A control applied at the management-group level can affect many subscriptions beneath it.

This is powerful because one governance rule can be applied consistently across a large environment.

It is also dangerous if used carelessly because a bad assignment at a high scope can affect many resources at once.

---

# 9. Azure Policy

Azure Policy helps enforce or evaluate organizational rules against Azure resources.

Suppose the company says:

> Production resources may only be deployed in approved US regions.

Without Policy, this may depend on every engineer remembering the rule.

With Policy:

```text
Resource deployment request
        ↓
Azure Policy evaluates request
        ↓
Is region allowed?
   ├── Yes → deployment can continue
   └── No  → deny / flag depending on policy effect
```

Azure Policy turns organizational rules into technical controls.

---

# 10. Policy definition

A policy definition describes the rule being evaluated.

Examples:

```text
Allowed locations
Allowed VM SKUs
Require specific tag
Audit resources without encryption
Require secure transport
Restrict public access
```

A definition by itself does not control anything until it is assigned to a scope.

---

# 11. Policy assignment

A policy assignment applies the policy definition to a scope.

Example:

```text
Policy Definition
“Allowed regions = East US, West US 2”
        │
        ▼
Assignment Scope
Production Management Group
        │
        ▼
All subscriptions and resources beneath that scope are evaluated
```

The same definition can be assigned to different scopes with different parameters where supported.

---

# 12. Common Azure Policy effects

Policy behavior depends on the effect configured in the definition.

Common concepts include:

```text
Deny
Audit
Modify
Append
DeployIfNotExists
AuditIfNotExists
Disabled
```

## Deny

Stops a non-compliant operation.

```text
User tries to create resource in unapproved region
        ↓
Policy evaluation
        ↓
Deny
        ↓
Deployment blocked
```

## Audit

Allows the resource but marks the condition as non-compliant.

```text
Resource exists without required setting
        ↓
Audit policy
        ↓
Resource reported as non-compliant
```

Audit is useful when an organization wants visibility before enforcement.

## Modify

Can alter supported resource properties during create/update to help bring resources into compliance.

## DeployIfNotExists

Can trigger deployment of a required related configuration/resource when conditions are met, according to the policy design and permissions.

The key idea is that Azure Policy can do more than simply block resources.

---

# 13. Policy initiatives

An enterprise usually has many related policies.

Instead of assigning dozens individually, policies can be grouped into an **initiative**.

```text
Security Baseline Initiative
│
├── Require encryption policy
├── Restrict public access policy
├── Require monitoring policy
├── Require tags policy
└── Allowed locations policy
```

An initiative makes it easier to assign and track a collection of governance requirements together.

---

# 14. Compliance state

Azure Policy can evaluate resources and report compliance state.

Conceptually:

```text
Policies assigned
      ↓
Resources evaluated
      ↓
Compliant / Non-compliant
      ↓
Governance dashboard / remediation work
```

This is useful because governance is not only about preventing bad deployments.

It is also about knowing whether the current environment satisfies required standards.

---

# 15. Remediation

Suppose an audit identifies 500 resources missing a required configuration.

Fixing every resource manually is inefficient.

Some policy scenarios support remediation tasks that help bring existing resources into compliance when the policy effect and resource type support it.

The important concept is:

```text
Detect non-compliance
        ↓
Remediate where supported
        ↓
Re-evaluate
        ↓
Track compliance
```

---

# 16. Azure Policy vs RBAC

This is one of the most important distinctions.

**RBAC asks:**

> Who is allowed to do what at this scope?

**Azure Policy asks:**

> Even if someone is allowed to perform an action, is the resulting resource/configuration permitted by organizational rules?

Example:

```text
Developer
  │
  │ RBAC says: allowed to create VMs
  ▼
Create VM request
  │
  ▼
Azure Policy says:
VM must be in approved region and approved size
```

So:

```text
RBAC = authorization
Policy = governance/compliance rules
```

They work together.

---

# 17. RBAC model

Azure RBAC can be understood as:

```text
WHO
User / Group / Service Principal / Managed Identity

        +

WHAT
Role definition / allowed actions

        +

WHERE
Scope
```

Example:

```text
WHO   = Application Operations Group
WHAT  = Virtual Machine Contributor
WHERE = Production Resource Group
```

That group can perform the actions allowed by the role within that scope, subject to other controls.

---

# 18. Principle of Least Privilege

A core security/governance principle is:

> Give identities only the permissions required to perform their job, and no more.

Bad model:

```text
Every engineer → Owner at subscription scope
```

Better model:

```text
Developers → permissions required for application resources
Operations → operational permissions
Security team → security-related permissions
Platform team → platform administration
```

Broad permissions increase the impact of mistakes or compromised credentials.

---

# 19. Built-in vs custom roles

Azure provides many built-in RBAC roles.

Examples conceptually include:

```text
Reader
Contributor
Owner
Service-specific contributor roles
```

When built-in roles do not match organizational requirements, custom roles can be created with carefully defined permissions.

Custom roles should be used intentionally because excessive custom-role sprawl can make access management difficult.

---

# 20. Owner vs Contributor vs Reader

At a foundation level:

```text
Reader
Can view resources/configuration but cannot make normal changes.

Contributor
Can manage resources but does not automatically have permission to grant Azure RBAC access to others.

Owner
Can manage resources and also manage access through RBAC.
```

Exact permissions should always be checked in the role definition for production decisions.

---

# 21. Resource Locks

RBAC and Policy still do not completely solve accidental deletion.

Suppose an administrator has legitimate permission to modify a production resource but accidentally clicks Delete.

Resource Locks provide another protection layer.

Two common lock types are:

```text
Delete lock
Read-only lock
```

## Delete lock

The resource can generally be modified but cannot be deleted until the lock is removed by someone with appropriate permissions.

## Read-only lock

Prevents changes and deletion through the Azure control plane, making the resource effectively read-only for those operations.

Locks are governance safeguards, not backups.

A lock cannot recover data after corruption.

---

# 22. Lock inheritance

Locks can be applied at scopes such as:

```text
Subscription
Resource Group
Resource
```

A lock applied to a parent scope can affect child resources.

Example:

```text
Resource Group
Delete Lock
   │
   ├── VM
   ├── Database
   └── Storage Account
```

The resources inherit the protection.

This makes locks powerful but also means they can interfere with deployments or operational processes if applied without understanding dependencies.

---

# 23. Tags

Tags attach metadata to Azure resources.

Example:

```text
Environment = Production
Application = Payments
Owner = PaymentsTeam
CostCenter = FIN-102
DataClass = Confidential
```

Tags help answer management questions such as:

```text
Who owns this resource?
Which application does it belong to?
Which environment is it?
Which cost center should be charged?
```

Tags can support cost analysis, inventory, automation, and governance.

---

# 24. Tags are not security boundaries

A tag such as:

```text
Environment = Production
```

does not prevent someone from changing the resource.

Tags are metadata.

Security and governance enforcement come from controls such as:

```text
RBAC
Azure Policy
Locks
Network/security controls
```

Policy can, however, be used to require or enforce tagging rules.

---

# 25. Naming standards

Azure does not automatically know an organization's naming convention.

A company may adopt a standard such as:

```text
<resource-type>-<application>-<environment>-<region>-<number>
```

Example:

```text
vm-payments-prod-eus-01
```

A naming standard helps people understand resources quickly.

But naming alone should not carry all metadata because some resource names cannot be changed easily and naming rules vary by Azure service.

Tags complement naming conventions.

---

# 26. Cost governance

Governance also applies to spending.

Suppose every team can deploy any VM size with no budget ownership.

Costs can become unpredictable.

Cost governance can include:

```text
Budgets
Cost alerts
Tags / cost-center metadata
Subscription separation
Approved SKUs
Policy restrictions
Resource ownership
Chargeback / showback models
Review of idle resources
```

Example:

```text
Development Management Group
       ↓
Policy restricts very expensive VM families
       ↓
Budgets assigned to subscriptions
       ↓
Cost-center tags required
       ↓
Teams receive spending visibility
```

Governance does not mean “always choose the cheapest resource.”

It means spending is intentional, attributable, and controlled.

---

# 27. Quotas and limits as governance considerations

Azure services have quotas and service limits.

An enterprise should understand them before large deployments.

Example:

```text
Application expected to scale to 500 instances
        ↓
Check regional/subscription quota
        ↓
Capacity and quota must support architecture
```

Quotas are not a replacement for budgets, but they can influence capacity planning and operational controls.

---

# 28. Compliance

Governance and compliance are closely related, but not identical.

**Governance** is how the organization establishes and enforces rules.

**Compliance** is whether the organization and its systems meet required internal, legal, regulatory, contractual, or industry requirements.

Examples may include requirements around:

```text
Data location
Encryption
Access control
Logging
Retention
Network exposure
Change tracking
Backup
Security configuration
```

The exact requirements depend on the organization and regulatory environment.

---

# 29. Internal vs external compliance

Compliance can come from different sources.

## Internal requirements

```text
Company security standard
Architecture standard
Tagging standard
Production-access policy
Backup requirement
```

## External requirements

```text
Laws
Regulations
Industry frameworks
Contracts
Customer requirements
```

Azure can provide technical capabilities and compliance information, but using Azure does not automatically make an organization compliant.

The organization is still responsible for configuring and operating its workloads correctly.

---

# 30. Shared responsibility and compliance

Microsoft manages security and compliance responsibilities for the cloud infrastructure according to the Azure service model and contractual commitments.

The customer remains responsible for many workload-specific controls.

For an IaaS VM, the customer may manage responsibilities such as:

```text
Guest operating system
Application
Identity/access configuration
Data
Network rules
Patching strategy
Monitoring
```

This is why compliance is a shared-responsibility problem.

---

# 31. Compliance evidence

An auditor may ask:

```text
Are production resources encrypted?
Who has administrative access?
Are logs retained?
Are resources deployed only in approved regions?
```

Without centralized controls, teams may have to collect evidence manually.

With governance and monitoring:

```text
Policy compliance results
RBAC assignments
Activity logs
Resource inventory
Security/configuration reports
```

can help provide evidence.

Governance therefore improves both enforcement and auditability.

---

# 32. Azure Activity Log

The Azure Activity Log records control-plane events for Azure resources and subscriptions.

Conceptually:

```text
User / Service Principal
       ↓
Azure management operation
       ↓
Create / Update / Delete / Role assignment etc.
       ↓
Activity Log
```

This helps answer questions such as:

```text
Who changed this resource?
When was it changed?
Which operation was attempted?
Did it succeed?
```

Activity logging contributes to governance, operations, security investigation, and auditability.

---

# 33. Governance hierarchy example

A realistic enterprise structure may look like:

```text
Microsoft Entra Tenant
│
└── Tenant Root Group
    │
    ├── Platform Management Group
    │   ├── Identity Subscription
    │   └── Connectivity Subscription
    │
    ├── Production Management Group
    │   ├── Payments Prod Subscription
    │   └── Customer App Prod Subscription
    │
    └── Non-Production Management Group
        ├── Development Subscription
        └── UAT Subscription
```

Different controls can be applied at different scopes.

Example:

```text
Tenant / top management group
→ baseline security policies

Production management group
→ stricter access + approved regions + production standards

Non-production management group
→ more flexible experimentation within limits

Subscription
→ budget + application ownership

Resource group
→ application-specific RBAC
```

---

# 34. Example — preventing an unapproved deployment

A developer has RBAC permission to create VMs in a production subscription.

They select an unapproved region.

```text
Developer
   │
   │ RBAC check
   ▼
Authorized to create VM ✅
   │
   ▼
Azure Policy check
   │
   ├── Region approved? NO
   │
   ▼
Deployment denied ❌
```

This demonstrates why RBAC and Policy are complementary.

---

# 35. Example — protecting production resources

Suppose production database administrators need to change configuration, but the organization wants to reduce accidental deletion risk.

Possible governance layers:

```text
RBAC
Only approved DBA group has database-management permissions

Policy
Database must meet required security configuration

Delete Lock
Prevents accidental control-plane deletion

Monitoring
Alerts and logs capture important changes

Backup
Provides recovery if data is lost/corrupted
```

No single control solves every risk.

---

# 36. Policy vs Lock vs RBAC vs Tag

These are frequently confused.

```text
RBAC
WHO can do WHAT and WHERE?

Policy
WHAT configurations are allowed/required?

Lock
Can this resource be modified/deleted at the control plane despite normal permissions?

Tag
WHAT metadata describes this resource?
```

Example:

```text
User = DevOps engineer
RBAC = can manage VM
Policy = VM must be in approved region
Lock = production resource cannot be deleted casually
Tags = owner, environment, cost center
```

---

# 37. Governance should be layered

Good governance is rarely one giant rule applied everywhere.

A layered approach is easier to manage:

```text
Organization-wide baseline
        ↓
Management-group policies
        ↓
Subscription-level ownership/budgets
        ↓
Resource-group/application controls
        ↓
Resource-specific exceptions when justified
```

The higher the scope, the broader the impact.

This is why high-scope changes require careful review.

---

# 38. Exceptions

Real organizations sometimes need exceptions.

Example:

```text
Policy says only approved regions
        ↓
Special project has regulatory/business need for another region
```

The solution should not be to disable governance globally.

Instead, organizations use controlled exception processes, narrower scopes, exclusions, or approved policy changes depending on the requirement.

The principle is:

> Exceptions should be explicit, justified, reviewable, and limited in scope.

---

# 39. Governance and automation

Governance works best when integrated into deployment workflows.

```text
Developer changes infrastructure code
        ↓
Pull Request
        ↓
Review / automated checks
        ↓
Deployment pipeline
        ↓
Azure RBAC + Policy evaluation
        ↓
Resources created
        ↓
Monitoring + compliance evaluation
```

This creates governance before, during, and after deployment.

---

# 40. Governance does not replace architecture

A policy can require two Availability Zones, encryption, or tags only where the service/policy capabilities support those checks, but policy itself does not design a resilient application.

Architecture still determines:

```text
How traffic flows
How components scale
How data is replicated
How failures are handled
How applications recover
```

Governance makes sure teams follow approved architecture/security requirements consistently.

---

# 41. A complete enterprise governance flow

```text
Microsoft Entra Tenant
        │
        ▼
Management Group Hierarchy
        │
        ├── Policy / initiative assignments
        ├── High-level RBAC
        └── Organizational standards
        │
        ▼
Subscriptions
        │
        ├── Billing / budgets
        ├── Quotas
        └── Workload ownership
        │
        ▼
Resource Groups
        │
        ├── Application lifecycle
        ├── Team RBAC
        └── Resource organization
        │
        ▼
Resources
        │
        ├── Tags
        ├── Locks
        ├── Policy compliance
        ├── Monitoring
        └── Activity logging
```

Governance is the combination of these controls, not a single checkbox.

---

# 42. Common misunderstandings

## “Management Groups contain resources directly.”

No. Management Groups organize subscriptions. Resources live inside subscriptions and resource groups.

## “A subscription is inside a Resource Group.”

No. Resource Groups are inside subscriptions.

## “RBAC and Azure Policy are the same.”

No. RBAC controls authorization; Policy controls/evaluates resource configuration against rules.

## “Tags secure a resource.”

No. Tags are metadata.

## “A delete lock is a backup.”

No. It helps prevent deletion operations; backup is a recovery mechanism.

## “If Azure is compliant, my application is automatically compliant.”

No. Compliance is shared. The customer must configure and operate workloads correctly.

## “Governance means blocking developers from doing everything.”

No. Good governance enables teams to move quickly inside safe, approved boundaries.

---

# 43. Connection to our Azure learning environment

Even our small lab can use governance ideas.

For example:

```text
Subscription
Free / learning subscription

Resource Group
rg-azure-learning-dev

Resources
VM
VNet
NIC
NSG
Disk
Public IP
```

A more mature learning setup could add:

```text
Tags
Environment = Dev
Project = AzureLearning
Owner = Charan

Budget alert
Protect free-credit spending

Policy examples
Restrict learning resources to selected regions/SKUs

Resource lock
Protect something important from accidental deletion when appropriate
```

This is how the same enterprise principles apply even at small scale.

---

# 44. Final mental model

```text
                   GOVERNANCE
                       │
      ┌────────────────┼────────────────┐
      │                │                │
      ▼                ▼                ▼
 ORGANIZE          CONTROL          PROVE / REVIEW
      │                │                │
Management       RBAC              Compliance state
Groups           Policy            Activity logs
Subscriptions    Locks             Audit evidence
Resource Groups  Tags/standards    Reports
      │                │                │
      └────────────────┼────────────────┘
                       ▼
               CONTROLLED AZURE
                 AT ENTERPRISE SCALE
```

Remember:

```text
Management Groups = organize subscriptions for governance
RBAC              = who can do what at which scope
Azure Policy      = what configurations are allowed/required
Locks             = protect against control-plane change/deletion
Tags              = management metadata
Compliance        = meeting required standards and proving it
```

---

# 45. Where this leads next

With cloud infrastructure, availability, recovery, scaling, reliability, predictability, manageability, and governance understood, one important cloud foundation topic remains:

```text
Sustainability
```

That topic connects cloud design decisions to efficient resource utilization, energy use, workload placement, scaling, and eliminating unnecessary infrastructure.
