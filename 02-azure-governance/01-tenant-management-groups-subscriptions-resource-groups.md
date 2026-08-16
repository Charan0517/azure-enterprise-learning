# Azure Tenant, Management Groups, Subscriptions, Resource Groups, and Resources

One of the most important Azure foundations is understanding where resources belong and where identity, governance, and billing boundaries fit.

## Start with Microsoft Entra ID tenant

A Microsoft Entra ID tenant is the identity boundary for an organization.

It contains identity-related objects such as:

- Users
- Groups
- Applications/service principals
- Roles and identity configuration

The tenant answers questions such as:

```text
Who is this user?
Which groups does the user belong to?
How should the user authenticate?
```

This is the **authentication / identity** side of the architecture.

A tenant is not the container where an Azure VM itself is stored.

## Authentication vs authorization

These concepts are different.

### Authentication

Authentication answers:

> Who are you?

Example:

```text
User signs in
   ↓
Microsoft Entra ID verifies identity
```

### Authorization

Authorization answers:

> Now that we know who you are, what are you allowed to do?

Example:

```text
Authenticated user
       ↓
Azure RBAC evaluation
       ↓
Can this user create/read/delete this resource?
```

A user can successfully authenticate but still have no permission to a particular Azure resource.

## Azure hierarchy

A useful enterprise hierarchy is:

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

## Management groups

Management groups provide a governance scope above subscriptions.

Suppose an enterprise has many subscriptions:

```text
Production Subscription
Development Subscription
Security Subscription
Shared Services Subscription
Data Platform Subscription
```

Instead of applying the same governance rule independently to every subscription, management groups allow subscriptions to be organized hierarchically and governance to be applied at a broader scope.

Example:

```text
Tenant Root
│
├── Production MG
│   ├── App Prod Subscription
│   └── Data Prod Subscription
│
└── Non-Production MG
    ├── Dev Subscription
    └── Test Subscription
```

Policies and access assignments can be inherited down the hierarchy depending on configuration.

## Subscriptions

An Azure subscription is a major management, access, quota, and billing boundary for Azure resources.

Resources are deployed into subscriptions.

An organization may separate subscriptions for reasons such as:

- Environment isolation
- Business units
- Billing/accountability
- Security boundaries
- Quota management
- Governance

For example:

```text
Enterprise Tenant
│
├── Production Subscription
├── UAT Subscription
└── Development Subscription
```

A subscription is associated with a tenant for identity and access.

## Resource groups

A resource group is a logical container for related Azure resources inside a subscription.

Example:

```text
Subscription
│
└── rg-orders-prod
    ├── Virtual Machine
    ├── Network Interface
    ├── Virtual Network
    ├── Public IP
    └── Storage resource
```

Resource groups help with lifecycle management, organization, permissions, policy scope, tagging, and cost analysis.

A resource can belong to only one resource group at a time.

## Resources

Resources are the actual Azure service instances you create.

Examples:

- Virtual machine
- Storage account
- Virtual network
- SQL database
- Key vault
- Load balancer

## Governance inheritance concept

Azure supports governance at multiple scopes.

Conceptually:

```text
Management Group
      │ Policy / RBAC
      ▼
Subscription
      │
      ▼
Resource Group
      │
      ▼
Resource
```

Assignments made at higher scopes can affect resources below them through inheritance.

This is why enterprise hierarchy design matters.

## Billing concept

Resources consume Azure services, and their usage contributes to subscription-related cost tracking/billing structures.

Resource groups help organize and analyze costs, but they are not separate subscriptions.

Deploying similar resources in two regions can incur charges for resources operating in both locations. Replication, networking, storage, and other service-specific components may also create costs.

## Example enterprise structure

```text
Microsoft Entra Tenant
│
└── Enterprise Management Group
    │
    ├── Production MG
    │   └── Production Subscription
    │       ├── rg-web-prod
    │       └── rg-data-prod
    │
    └── Non-Production MG
        ├── Development Subscription
        │   └── rg-web-dev
        └── UAT Subscription
            └── rg-web-uat
```

This allows the company to organize environments while still maintaining centralized governance.

## Key takeaway

Do not think of all Azure containers as doing the same thing.

```text
Entra tenant     → Identity boundary
Management group → Organize/govern subscriptions
Subscription     → Resource, billing, quota and management boundary
Resource group   → Logical lifecycle container
Resource         → Actual Azure service instance
```

We will later connect this hierarchy to Azure Policy, RBAC, budgets, tags, and enterprise landing-zone design.