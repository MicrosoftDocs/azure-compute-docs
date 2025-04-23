---
title: Advanced Configuration
description: Creating Allowlist
author: minnielahoti
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 04/08/2025
ms.author: minnielahoti
ms.reviewer: azmetadatadev
---

# Advanced configuration

Metadata Security Protocol (MSP) allows users to optionally define custom Role Based Access Control (RBAC) allowlists for metadata service endpoints on a per-user and/or per-process basis. This allowlist is enabled by a new resource type in the Azure Compute Gallery, the `InVMAccessControlProfile`.

## Motivation

Traditionally, metadata services treat the entire Virtual Machine (VM) as the trust boundary and allow any software running in the guest to access them. This access is overly permissive, especially if your workload isn't strictly implemented with a modern cloud native architecture. Microsoft Security Response Center (MSRC) found that most security attacks could have been prevented by limiting access to only the necessary software.

The `InVMAccessControlProfile` allows you to define more granular control on the applications/users that can have access to the endpoints. It enables you to write rules like:

- Network configuration can be only accessed by the Guest Agent and SQL Server
- Managed Service Identity (MSI) tokens can be only accessed by the Monitoring Extension

The new resource type facilitates managing these configurations at scale. The rules that are appropriate for your VM depend on what software its running. The `InVMAccessControlProfile` allows you to define your configuration once and link it against all applicable VMs.

> [!NOTE]
> We recommend treating these controls as *defense in depth* in your threat modeling. These controls can significantly enhance the security of a workload, but they're best used as extra protection instead of as core isolation mechanisms. In cloud native multitenant workloads, especially if untrusted code is executed, hardware/hypervisor backed isolation like offered by [Azure Container Instances](https://azure.microsoft.com/products/container-instances) offer far greater protection. MSP authorization rules complement hardware backed isolation.

## Properties

| Property | Type | Details |
|--|--|--|
| `mode` |Enum: `"Audit" \| "Enforce" \| "Disabled"` | See MSP [Configuration](./configuration.md#inline-configuration). |
| `defaultAccess` | Enum: `"Allow" \| "Deny"` | Controls if an endpoint is accessible or restricted when no privileges for that resource are specified. See [Privileges](#privileges). |
| `rules` | `object` | The RBAC definition to apply. See [Writing Rules](#writing-rules). |

## MSP Allowlist (RBAC) concepts

The first step of access control is to verify the identity of the client. Normally the identity verification is done by using some type of a credential. MSP was designed to avoid introducing any breaking changes to clients to maximize compatibility and make onboarding easier. Additionally, having clients upgraded with credentials introduces another layer of trust bootstrapping problems to solve.

Instead, MSP supports defining "virtual identities" against existing OS and process metadata. A VM's kernel keeps track of which user account a process belongs to in addition to other metadata about its invocation. The GPA is capable of identifying which process made the HTTP request. As a result, the GPA can access that metadata to determine the identity of a caller and make authorization decisions.

### Roles

Roles allow you to group multiple privileges into a named, reusable unit. They're used to improve organization and readability.

### Role Assignments

Role assignments pair an identity with a role. Meaning requests that come from a process that matches the identity has all the privileges associated with that role.

### Identities

MSP allows the user to define a set of conditions to be evaluated against a property bag of process metadata. If all the conditions match, the process is considered to "belong" to this identity and is granted privileges.

The GPA supports writing *exact match* rules against these metadata:

| Process Metadata | Description |
|--|--|
| `username` | The human readable name of the account the process is running under. |
| `groupName` | The human readable name of the group the account the process is running under must belong to. A user can belong to multiple groups. Conditions here are implemented as a set contains operation. |
| `processName` | The display name of the process. **Note, this is purely defense in depth!** |
| `exePath` | The full path of the running executable. For some programs the runtime, not the actual file being executed, shows. E.g., python. |

The ideal way to configure your workload would be to run each application in a dedicated user account with uniquely scoped groups, then write rules only against those properties. The user ID can't be spoofed and there's no pattern matching to contend with. However, we recognize this isn't practical in all workloads, which is why we offer the other properties.

If multiple properties are specified, then the matching is performed as a logical AND. For example, the identity definition:

```json
{
    "name": "WebServer",
    "exePath": "/usr/sbin/apache2",
    "processName": "apache2",
    "groupName": "www-data"
}
```

Would generate the validation:

```c#
bool isWebServerIdentity = properties["exePath"] == "/usr/sbin/apache2"
    && properties["exePath"] == "apache2"
    && properties["groups"].contains("www-data");
```

### Privileges

Privileges grant access to specific endpoints. Some endpoints are RESTful and can be expressed purely as a path. Others are influenced by query parameters.

Privileges are defined with a `name`, a `path`, and an optional `Set` of `queryParameters`:

```json
"privileges": [
    {
        "name": "GoalState",
        "path": "/machine",
        "queryParameters": {"comp": "goalstate"}
    },
    {
        "name": "config",
        "path": "/machine",
        "queryParameters": {"comp": "config"}
    }
]
```

#### Comparison Rules

- String matching is *case-insensitive*
- If *no* query parameters are specified, the privilege grants access to *all* possible values on that path

#### `defaultAccess` Rules

Default access modes are used to slowly lock down a workload and to simplify rule authoring in the case where only specific endpoints are of interest for restricting access to.

| `defaultAccess` Mode | Behavior |
|--|--|
| `Deny` | If no explicit assignment exists authorizing the guest app to the requested resource, the request is rejected. |
| `Allow` | If no rules exist for a specific privilege, then access defaults to allowed. If any privilege for that resource exists, then the behavior reverts back to default deny for that resource. |

- If query parameters are provided, the question *"is a privilege defined?"* for determining if `defaultAccess` applies becomes: *"Does any rule match this path + every specified query parameter in the rule is present + matches in the request?"*. Extra query parameters are ignored in this assessment.
- The request is rejected if duplicate query parameter keys in a request are invalid

## Writing Rules

Specifying an allowlist via RBAC rules is optional. If one is specified though, *all* fields must be supplied. (For example, if you didn't have any `roles` to define you would still provide the empty array).

The easiest way to generate rules is by first running MSP in `Audit` mode, then [using the audit logs to generate rules](./other-examples/audit-logs-to-rules.md).

Full Instance Metadata Service (IMDS) schema expansion / example of the rules subsection:

```json
{
  "defaultAccess": "allow",
  "mode": "enforce",
  "rules": {
    "privileges": [
      {
        "name": "Msi",
        "path": "/metadata/identity/oauth2/token"
      }
    ],
    "roles": [
      {
        "name": "MyWebApp",
        "privileges": [
          "Msi"
        ]
      }
    ],
    "identities": [
      {
        "name":"MyWebApp1",
        "processName":"WebApp1"
      }
    ],
    "roleAssignments": [
      {
        "role": "MyWebApp",
        "identities": [
          "MyWebApp1"
        ]
      }
    ]
  },
  "id": "D/uTRPye9b9xSUCRIgfRDF41zeY="
}
```

Full Wireserver schema expansion / example of the rules subsection:

```json
{
  "defaultAccess": "allow",
  "mode": "enforce",
  "rules": {
    "privileges": [
      {
        "name": "GoalState",
        "path": "/machine",
        "queryParameters": {
          "comp": "goalstate"
        }
      }
    ],
    "roles": [
      {
        "name": "Provisioning",
        "privileges": [
          "GoalState"
        ]
      }
    ],
    "identities": [
      {
        "name": "WinPA",
        "exePath": "C:\\Windows\\OEM\\Unattend.wsf.exe",
        "processName": "winpa.exe",
        "userName": "SYSTEM"
      }
    ],
    "roleAssignments": [
      {
        "role": "Provisioning",
        "identities": [
          "WinPA"
        ]
      }
    ]
  },
  "id": "D/uTRPye9b9xSUCRIgfRDF41zeY="
}
```

## Implicit + Default Configurations

The following RBAC definition is applied as default behavior for defense-in-depth access rules that existed before MSP:

```json
{
    "imds": {
        "defaultAccess": "Allow",
        "mode": "Disabled",
    },
    "wireServer": {
        "defaultAccess": "Allow",
        "mode": "Enforce",
    }
}
```

Remember though, Wireserver only allows access from Administrator / root processes. The reason `defaultAccess` is still `Allow` here's because this rule isn't user configurable. It's baked into the GPA. The GPA checks if the client is an admin user before evaluating any RBAC rules for a Wireserver request. If the client is not an admin user, the request is immediately rejected.
