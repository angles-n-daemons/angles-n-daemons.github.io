---
title: "RBAC x ReBAC, a hybrid authorization model for complex businesses"
date: 2023-02-17T15:12:30-04:00
tags: ["security", "authorization"]
categories: ["security", "software"]
---

## When one access-control model isn't enough

Access controls are an important aspect of ensuring security within complex businesses. Most companies I've worked with have employed an RBAC model

## Terminology

 * *Permission*: 
 * *Relationship*: 
 * *Action*: Any action taken by a user within the system. Can be described as RPC or HTTP endpoints in a technical sense.
 * **Gate**: 
 * **Permission**: 

## Use Case

## A quick refresher of the access models

### Role Based Access Control (RBAC)

In a traditional Role Base Access Control model, there are Users, Roles, Permissions and Operations. Users are mapped to Roles, Roles are mapped to Permissions, and Permissions are mapped to Operations.

For the above use case you could imagine the following records:

*Roles*
 * `AUTO_POLICY_ADVOCATE`
 * `AUTO_POLICY_ADMIN`
 * `HOME_POLICY_ADVOCATE`
 * `HOME_POLICY_ADMIN`

*Permissions*
 * `AUTO_POLICY_READ`
 * `AUTO_POLICY_WRITE`
 * `AUTO_POLICY_OWNER`
 * `HOME_POLICY_READ`
 * `HOME_POLICY_WRITE`
 * `HOME_POLICY_OWNER`

*Operations*
 * `LoadAutoPolicy`
 * `ModifyAutoPolicy`
 * `LoadHomePolicy`
 * `ModifyHomePolicy`

### Relationship Based Access Control (ReBAC)

In a Relationship Based Access Control model, resources are identified in the system and members have access through relationships to those resources. 

*Resources*
 * `AutoPolicy`
 * `HomePolicy`

*Relationships*
 * `READ`
 * `WRITE`
 * `OWNER`


TODO: Brian define why this breaks down as the business becomes more complex

Fine grained control can be achieved by identifying each of the systems that users can interact with by resources, but controlling relationships at this point becomes challenging and error prone. It's easier to define


Defining a relationship model for each of these complex
* IDV Viewing
* Government ID Viewing
* Government ID Writes

You're a technology business who has a number of product lines, and a remarkable sprawl of customers. You get on using RBAC authorization for your internal applications but down the line you're faced with a challenge.

In this access model, if Jen has the permission to refund Carol, then she can also refund Jim and Wendy. If Justin can deactivate Lindsay's account, then certainly he could deactivate Nick and Nancy's as well. Herein lies the difficulty in a RBAC access scheme, controlling which accounts employees have access to becomes difficult.

todo: image describing permission check

Moving entirely over to ReBAC doesn't solve this challenge either. If Terrance is able to refund purchases should he also be allowed to reset passwords? Otherwise, are your relationship levels so complex that establishing relationships becomes indecipherable? 

todo: image describing rebac gap

## A practical example

## Using the good bits of each

After a good bit of exploration, one approach that seems reasonable is layering both approaches carefully.

Let's start with a practical example. We operate an insurance business who's primary 

We want to establish two internal user groups, one for managing auto insurance policies, and one for managing home insurance policies.

## Creating the framework teams will use

```
class AccessGate (
    val permission: Permission,
    val relationship: Relationship,
) {
    fun assert(caller: Identity, account: Account) {
        val hasPermission = callerHasPermission(permission)
        val hasRelationship = callerHasRelationship(caller, account, relationship)

        if (hasPermission && hasRelationship) {
            return
        }
        
        throw(AuthorizationError(
            "missing {permission} or relationship {relationship} with {account}",
        ))
    }
}
```

test test


test test

```
rpc AutoPolicy:
    - LoadAutoPolicy:
        permission: "AUTO_POLICY_READ"
        relationship: "ACCOUNT_READ"

    - ModifyAutoPolicy:
        permission: "AUTO_POLICY_WRITE"
        relationship: "ACCOUNT_WRITE"
```

## Creating narrow overrides for exceptions

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
