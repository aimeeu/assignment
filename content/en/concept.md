---
title: Concept
weight: 2
description: This page shows the concept of RBAC in Armory CD-as-a-Service.
---

## Role-based access control (RBAC) in CD-as-a-Service

CD-as-a-Service's RBAC implementation provides the following features:

* System roles for admins and machine-to-machine credentials.
* Custom roles that you create to fit your company's needs.

## RBAC implementation

```mermaid
classDiagram
    class Role {
      +String name
      +Tenant tenant
      +List~Grant~ grants
    }
    class Grant {
      +GrantType type
      +String resource
      +List~Permission~ permissions
    }
    class GrantType {      
      api
    }
    class Permission {      
      full
    }
    class User {
      +List~Role~ roles
    }
    class ClientCredential {
      +List~Role~ roles
    }
    class Tenant {
      +String name
    }

    Role "1" --> "*" Grant
    Grant "1" --> "*" Permission
    Grant "1" --> "1" GrantType
    Role "1" --> "1" Tenant
    Role "1" --> "1" Tenant
    User "1" --> "*" Role
    ClientCredential "1" --> "*" Role
```

Central to CD-as-a-Service's RBAC implementation is a _Role_, which defines what a user or external app can do within the platform. Each Role has at least one _Grant_ object, which defines permissions.

## System roles

CD-as-a-Service provides the following system roles:

| Role Name | Grants | Assign To |
| ---------- | -------- | --------- |
| Organization Admin | Full access to all features | User |
| Deployments Full Access | Trigger and manage deployments | User<br/>Client Credential |
| Remote Network Agent | Access to CD-as-a-Service from your Kubernetes deployment target | Client Credential |

Assign a new user `Organization Admin`, `Deployments Full Access`, or a custom role.

## Custom roles

Define your custom RBAC roles in a YAML file with this structure:

```yaml
roles:
  - name: ROLE_NAME
    tenant: TENANT_NAME
    grants:
      - type: api
        resource: RESOURCE
        permission: full
```

* _`ROLE_NAME`_: (Required) The name for this role.
* _`TENANT_NAME`_: (Optional) The name of an existing tenant. The role is scoped to the tenant. Create an organization-wide role by omitting the `tenant` definition.
* `grants`: (Required) an array of _Grant_ objects. 

  * _`RESOURCE`_: (Required) Defines what area the Grant can access:

    * `tenant`: The tenant specified in the `roles.tenant` field. Use `tenant` when you define a [Tenant Admin role](#tenant-admin-role).
    * `deployment`: Deployments using CLI and the **Deployments** UI. If you omit `roles.tenant`, the role has this Grant across your organization.
    * `organization`: All features across your entire Organization. Use this when you create an Organization Admin role that maps to an SSO group.

## Custom role examples

### Tenant Admin role

This example defines three Tenant Admin roles, one for each tenant. Each role has full authority within the specified tenant.

```yaml
roles:
  - name: Tenant Admin Main
    tenant: main
    grants:
      - type: api
        resource: tenant
        permission: full
  - name: Tenant Admin Finance
    tenant: finance
    grants:
      - type: api
        resource: tenant
        permission: full
  - name: Tenant Admin Commerce
    tenant: commerce
    grants:
      - type: api
        resource: tenant
        permission: full
```

If you want to grant a user permission to manage all of your tenants, assign that user the **Organization Admin** role using the UI.

### Tenant deployment role

This example defines a role that grants permission to use the **Deployments** UI and start deployments using the CLI. The role is bound to the `finance` tenant.

```yaml
roles:
  - name: Deployer Finance
    tenant: finance
    grants:
      - type: api
        resource: deployment
        permission: full
```

### Organization deployment role

This example defines a role that grants permission to use the **Deployments** UI and start deployments using the CLI across your entire organization. `tenant` is not defined, which makes this an organization-wide role.

```yaml
roles:
  - name: Deployer All Tenants
    grants:
      - type: api
        resource: deployment
        permission: full
```

## {{% heading "nextSteps" %}}

* [Add custom roles]({{< ref "iam/manage-rbac-roles" >}}) to your CD-as-a-Service organization.
* [Assign roles to users]({{< ref "iam/manage-users" >}}) using the UI.
