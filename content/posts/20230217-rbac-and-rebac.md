---
title: "RBAC x ReBAC, a hybrid authorization model for large technology companies"
date: 2023-05-16T15:12:30-04:00
tags: ["security", "authorization"]
categories: ["security", "software"]
---

## When one access-control model isn't enough

Access controls are an important aspect of ensuring security within complex businesses. Most companies I've worked with have employed an RBAC model to secure their internal applications, which has provided a good-enough set of restrictions and controls for employees. This model breaks down as the business gets more complex, as does ReBAC - a popular alternative to RBAC. The wikipedia page for ReBAC describes how it can be layered in conjunction with RBAC, but the literature online is sparse on exactly how to go about doing so. In this article, I aim to describe an implementation for doing so using a concrete business use case.

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

Image (company hierarchy)

Company
  - Engineering
    - App
    - Fraud Tooling
    - Customer Service
  - Operations
    - Fraud
    - Customer Service
      - Home
      - Life

System Architecture

App

App Service

Fraud Tools Service

Customer Service Service

Database


## A quick refresher of the access models

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

In this access model, if Jen has the permission to refund Carol, then she can also refund Jim and Wendy. If Justin can deactivate Lindsay's account, then certainly he could deactivate Nick and Nancy's as well. Herein lies the difficulty in a RBAC access scheme, you're able to define what actions employees are able to take, but not which accounts they are able to take them on.

Below we'll talk about ReBAC, which addresses this problem - but comes with its own scaling challenges.

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

When a Fraud Agent begins an investigation, they can be granted similar relationships for the accounts they're investigating.

There's an immediate issue in this approach in that since Account is the only resource and the relationships are so simple, Auto Insurance Agents will be able to take action on the Home Insurance policies of customers that they are assisting.

This can quickly get out of hand as the functionality to take more sensitive actions like credit accounts, view PII, or wipe data becomes the responsibility of the Operations department. We need some way to limit access to certain actions between groups in Operations.

2. Various pieces of the Account as Resources; Read, Write and Admin as the Relationships between Employees and these Resources.

*Resource*
 * `HOME_POLICY`
 * `AUTO_POLICY`

*Relationships*
 * `READ`
 * `WRITE`
 * `OWNER`

This approach is better from an access restriction perspective than both RBAC and the above ReBAC model, but introduces some engineering challenges. Specifically, at what points are these relationships granted? If a Customer Service call starts, what relationships should the system configure? If transferred between teams, do we add new relationships and remove some old?

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

This approach is better from an access restriction perspective than both RBAC and the above ReBAC model, but introduces some engineering challenges. Specifically, at what points are these relationships granted? If a service call begins for an Auto Policy, what system configures the relationship between the account

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
