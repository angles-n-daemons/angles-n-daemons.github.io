---
title: "RBAC x ReBAC, a hybrid authorization model for big technology companies"
date: 2023-05-16T15:12:30-04:00
tags: ["security", "authorization"]
categories: ["security", "software"]
---

*Note:* This article is an abandoned draft. I still feel some of the ideas and concepts are helpful, so I've left it here in case the thought experiment benefits anyone.

## When one access-control model isn't enough

An application's access-control scheme is an important part of ensuring safety within your business. Most companies I've worked with have employed an RBAC model to secure their internal applications, which has provides a good-enough set of restrictions and controls for employees. This model breaks down as the business gets more complex, as does ReBAC - a popular alternative to RBAC. The wikipedia page for ReBAC describes how it can be layered in conjunction with RBAC [citation needed], but the literature online is sparse on exactly how to go about doing so. In this article, I aim to describe an implementation for doing so using a concrete business use case.

This document is broken down into the following sections:
 * Use Case Primer
 * Intro / Refresher to RBAC and ReBAC
 * Implementation of the hybrid approach
 * Extensions, Additions, Drawbacks

It's my hope that by sharing this approach, the industry understanding on how to layer these two authorization schemes becomes more standardized.

## Terminology

 * *RBAC*: Role Based Access Control
 * *ReBAC*: Relationship Based Access Control
 * *Permission*: 
 * *Relationship*: 
 * *Action*: Any action taken by a user within the system. Can be described as RPC or HTTP endpoints in a technical sense.
 * **Gate**: 
 * **Permission**: 

## Use Case

Since this approach is more complex than most access control models, it's best to first consider whether it would benefit the business. Companies that meet the following criteria are likely to gain from this scheme.

 * Direct to consumer
 * Large Operations division
 * Diverse set of responsibilities within internal groups

The reason for this is that this access scheme is just-complex enough to allow for easy controls and restriction for both account access and actions.

With these things in mind, I propose an insurance business "Quinnsurance" as the foundation for this exercise. This company contains two product lines, home and auto insurance.

The operations department consists of two departments, Fraud and Customer Service. Fraud is a single team, which investigates fraud in both auto and home claim fraud and Customer Service contains two teams, one for each product line.

The engineering department consists of a Product, a Fraud Tools and a Customer Service Tools team. The product team will be the the user facing application which isn't relevant to the system we're describing, because they aren't used by internal employees. The Fraud and Customer Service Tools will be the systems which we'll be using to leverage our authorization scheme.

```
                                ┌────────────────┐
                                │                │
                                │  Quinnsurance  │
                                │                │
                                └───────┬────────┘
                                        │
                                        │
                                        │
                       ┌────────────┐   │   ┌─────────────┐      ┌────────────────┐
                       │            │   │   │             │      │                │
      ┌───┬────┬───────┤Engineering ◄───┴───► Operations  ├──────►─Fraud          │
  ┌───▼───┤    │       │            │       │             │      │ Investigations │
  │       │    │       └──────┬─────┘       └─────┬───────┘      │                │
  │ App   │    │              │                   │              └────────────────┘
  │       │    │              │                   │
  └───────┘    │              │                   │
               │              │                   │
        ┌──────▼────┐  ┌──────▼────┐        ┌─────▼───────┐
        │           │  │           │        │             │
        │ Fraud     │  │ Customer  │        │ Customer    ├────────────┐
        │ Tools     │  │ Service   │        │ Service     │            │
        │           │  │ Tools     │        │             │            │
        └───────────┘  │           │        └────┬────────┘       ┌────▼────┐
                       └───────────┘             │                │         │
                                                 │                │ Home    │
                                            ┌────▼────┐           │         │
                                            │         │           └─────────┘
                                            │ Life    │
                                            │         │
                                            └─────────┘
```
Figure 1. A depiction of the company hierarchy

## A refresher on the access control schemes

### Role Based Access Control (RBAC)

Role Based Access Control is a simple strategy for defining who has access to what. Every Employee is assigned to one or more Roles. Each Role has a set of Permissions, which are the ability to see or do things within internal apps. In this article, Permissions are going to be mapped directly to HTTP endpoints. In practice, there may be one additional layer of mapping from Permissions to Operations, but to keep things simple we've omitted that layer here.

Here are some proposed definitions for Roles and Permissions that would support Quinnsurance's operational division.

*Roles*
 * `AUTO_POLICY_AGENT`
 * `AUTO_POLICY_ADMIN`
 * `HOME_POLICY_ADVOCATE`
 * `HOME_POLICY_ADMIN`

*Permissions*
 * `LoadAutoPolicy`
 * `ModifyAutoPolicy`
 * `RefundAutoPolicy`
 * `LoadHomePolicy`
 * `ModifyHomePolicy`
 * `RefundHomePolicy`

[Image on mapping]

In this access model, if Jen has the permission to refund Carol, then she can also refund Jim and Wendy. If Justin can deactivate Lindsay's account, then can also deactivate Nick and Nancy's as well. Herein lies the difficulty in a RBAC access scheme, you're able to define what actions employees are able to take, but not which accounts they are able to take them on. This is generally more access than the employees need - as they rarely need to affect changes across a broad collection of accounts

Below we'll talk about ReBAC, which addresses this problem - but comes with its own unique challenges.

### Relationship Based Access Control (ReBAC)

In a Relationship Based Access Control model, resources are identified in the system and members have access through relationships to those resources. 

There are a few ways that we could define these for Quinnsurance, each with its own unique drawbacks.

1. Account as the primary Resource; Read, Write and Admin as the Relationships between Employees and these Resources.

*Resources*
 * `Account`

*Relationships*
 * `READ`
 * `WRITE`
 * `OWNER`

This model is the simplest and easiest to manage. It is also the approach with the most glaring safety issues.

For establishing relationships, it should be fairly easy to automate access. When a Customer Service Agent begins a call with a customer, they can be assigned the READ / WRITE relationships with the account that they are servicing.

If a Fraud Agent begins an investigation, they can be granted similar relationships for the accounts they're investigating.

There issue with this approach is that since Account is the only resource and the relationships are so simple, access granted to an account is absolute. For example if I'm granted access to see a person's address, it's technically the same permission required to view their social security number.

This can quickly get out of hand as the actions that operations needs to take become more sensitive like creditint accounts, viewing PII, or wiping account data. We need some way to limit access to certain actions between groups in Operations.

2. Various pieces of the Account as Resources; Read, Write and Admin as the Relationships between Employees and these Resources.

*Resource*
 * `HOME_POLICY`
 * `AUTO_POLICY`

*Relationships*
 * `READ`
 * `WRITE`
 * `OWNER`

This approach is better from an access restriction perspective than both RBAC and the above ReBAC model, but introduces some engineering challenges. Specifically, at what points are these relationships granted? If a Customer Service call starts, how are the correct relationships configured? If transferred between teams, do we add new relationships and remove some old?

Configuring relationships becomes difficult, either it's:

 * Automated, and of great difficulty to the engineering team as the business changes
 * Manual, and requires supervisors to be constantly granting and removing access between accounts.

This approach breaks down as the business internally becomes more complex. The layering of both RBAC and ReBAC is meant to address these issues.

3. Account as the primary Resource; various relationships defined to model functionality of the system.

*Resources*
 * `Account`

*Relationships*
 * `READ_AUTO_POLICY`
 * `WRITE_AUTO_POLICY`
 * `ADMIN_AUTO_POLICY`
 * `READ_HOME_POLICY`
 * `WRITE_HOME_POLICY`
 * `ADMIN_HOME_POLICY`

This approach suffers from the same challenges as the last, as configuring the relationships becomes the primary challenge in managing access to resources.

## Using the good bits of each

After a good bit of exploration, the approach that I've found the easiest to implement has been merging RBAC with the first ReBAC implementation. This comes with the following benefits:

 * Access is limited to only accounts that the employee has a relationship with.
 * Relationship configuration is simple, and therefore can be automated.
 * Which actions the user is able to take is determined by role, and managed by the RBAC portion of our scheme.
 * The approach is clear and easy to configure.

## Implementation

The implementation of this access control model is going to be simple enough, consisting of Access Gates and Relationship Checkpoints.

### Access Gates

The simplest part of this exercise is to define what an Access Gate will do an how it will look in code. Our access gates are little bits of code that run before any action is taken. They generally will live as middleware in RPC / HTTP handlers, and will ensure that users have the correct permissions before the action is executed.

Each Access Gate takes a small bit of configuration:
 * A permission requirement
 * A relationship requirement

In order to pass the gate, the user will first need both the permissions specified as well as the relationship to the account in question.

[Disclaimer? can these be null?]

```
class AccessGate (
    val permission: Permission?,
    val relationship: Relationship?,
) {
    fun assert(caller: Identity, account: Account) {
        val hasPermission = callerHasPermission(permission)
        val hasRelationship = !notNull(permission) && callerHasRelationship(caller, account, relationship)

        if (hasPermission && hasRelationship) {
            return
        }
        
        throw(AuthorizationError(
            "missing {permission} or relationship {relationship} with {account}",
        ))
    }
}
```

[TODO, describe usage]
Once your access gate abstraction is ready to be used, you can start using it in your handlers.

```
class AccountService {
    @GET("/api/account/address")
    @AccessGate(Permissions.Address.READ, Relationship.Account.READ)
    fun getAddress(req: GetAddressRequest): GetAddressResponse {
        // Logic to get address
    }

    @PUT("/api/account/address")
    @AccessGate(Permissions.Address.WRITE, Relationship.Account.WRITE)
    fun updateAddress(req: UpdateAddressRequest): UpdateAddressResponse {
        // Logic to update the address
    }
}
```

## Relationship Checkpoints

The other part of the system that requires definition is the one in which Relationships are added and removed between Accounts and Employees. You see this in products like Facebook where Users establish these relationships (ie: one person "friends" another) or like Google Docs where each resource has an owner and the owner manually configures the relationships.

For internal products, it's often easier to setup checkpoints where relationships start and end using existing points in your system. For example, when a Customer Service Agent is connected with an end user, that block ends up being a good point to establish a relationship between the Agent and the User's Account - so that the Agent can gain access to the functionality they'll need to assist the User.

Likewise, when the support call ends - it's easy enough to remove that relationship so that the Agent is no longer able to access that User's account. Since the only Resource in the system is an Account, automating these relationships stays easy over the lifetime of the system.

```
class CustomerSupportAssignment {
    ...

    fun assignSupportRepToCustomer(accountId: Number, repId: Number) {
        configureRelationship(accountId, repId, Relationships.WRITE)
    }
    ...

    fun closeSupportCase(case: Case) {
        removeRelationship(case.accountId, case.repId, Relationships.WRITE)
    }
}
```



```
rpc AutoPolicy:
    - LoadAutoPolicy:
        permission: "AUTO_POLICY_READ"
        relationship: "ACCOUNT_READ"

    - ModifyAutoPolicy:
        permission: "AUTO_POLICY_WRITE"
        relationship: "ACCOUNT_WRITE"
```

## Creating overrides

```
class AccessGate (
    val permission: Permission,
    val relationship: Relationship,
++  val override: Permission,
) {
    fun assert(caller: Identity, account: Account) {
        val hasPermission = callerHasPermission(caller, permission)
        val hasRelationship = callerHasRelationship(caller, account, relationship)
++      val hasOverride = callerHasOverride(caller, override)

++      if (hasOverride || (hasPermission && hasRelationship)) {
            return
        }
        
        throw(AuthorizationError(
            "missing {permission} or relationship {relationship} with {account}",
        ))
    }
}
```

## Extensions and Drawbacks

### Gate configuration

```
rpc AutoPolicy:
    - LoadAutoPolicy:
        permission: "AUTO_POLICY_READ"
        relationship: "ACCOUNT_READ"
++      override: "AUTO_POLICY_ADMIN"

    - ModifyAutoPolicy:
        permission: "AUTO_POLICY_WRITE"
        relationship: "ACCOUNT_WRITE"
++      override: "AUTO_POLICY_ADMIN"
```

### Drawbacks

Some drawbacks on this layered approach include:

`Complexity`

The first notable drawback of using this approach is that it is more complex than RBAC or a very simple ReBAC model on its own.

`Restrictiveness`

There are certain cases where Employees will need broader access to many accounts and if this becomes the norm rather than the exception within your Operations department you might be better served with a different authorization scheme.

[Note, meditate on this a little further]

### Final Comments

I'll close by saying that I've used this scheme in a business setting with a good degree of success. Using our company's existing Registrar (from which we could configure RBAC controls), we were able to add it to our internal case review system. It worked well for the end users, and got a solid affirmation from the Security team. I'd encourage anyone thinking of a better way to secure their systems to explore whether it could work for your teams.
