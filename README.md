# Azure-labs
# Manage Subscriptions and RBAC

## Overview
Configured Azure role-based access control (RBAC) using management groups, built-in roles, and a custom role definition to enforce least privilege access across subscriptions.

## Objective
Grant a help desk team the ability to manage virtual machines and submit support tickets across all subscriptions — without giving them more access than necessary.

---

## What I Built

### Management Group
Created a management group (`az104-mg1`) to serve as a single scope for all role assignments, so permissions apply across all subscriptions underneath it without repeating assignments individually.

### Built-in Role Assignment
Assigned the **Virtual Machine Contributor** role to the HelpDesk group at the management group level. This allows the team to manage VMs across all subscriptions through inheritance.

### Custom Role
Cloned the built-in **Support Request Contributor** role and removed the `Microsoft.Support/register/action` permission. This prevents the help desk from registering new Azure resource providers — an administrative action outside their responsibilities.

The custom role JSON structure:
```json
{
  "actions": ["Microsoft.Support/*"],
  "notActions": ["Microsoft.Support/register/action"],
  "assignableScopes": [
    "/providers/Microsoft.Management/managementGroups/az104-mg1"
  ]
}
```

### Activity Log
Verified all three operations were recorded in the Activity Log — management group creation, role assignment, and custom role definition — confirming Azure's built-in audit trail was capturing changes in real time.

---

Screenshot----
Role assignments confirmed on az104-mg1---- Screenshot Lab1.webp
Log Activity ------ ScreenshotLab2.webp

## Key Concepts

- **Management groups** sit above subscriptions and allow policies and roles to be assigned once and inherited downward
- **RBAC is additive** — users get the combined permissions of all their role assignments
- **NotActions** subtracts permissions from the allowed set within a role — it is not an explicit deny
- **Assignable scopes** control where a custom role can be used
- **Activity Log** retains 90 days of control plane operations by default and can be exported for longer retention

---


---

## Skills Demonstrated
`Azure RBAC` `Management Groups` `Custom Role Definitions` `Least Privilege` `Activity Log` `Microsoft Entra ID`
