---
layout: default
title: 01 — Identity & Governance
nav_order: 3
description: "Design identity and governance and solutions."
permalink: /01-identity-governance/
mermaid: true
---

# 🔐 01 — Manage Azure Identities and Governance
{: .no_toc }

**Domain Weight: 20–25% of exam**
{: .label .label-blue }

Covers Microsoft Entra ID, users, groups, RBAC, Azure Policy, resource locks, tags, subscriptions, and cost management.
{: .fs-5 .fw-300 }

> 📁 [← Back to Home](/az-104-study-notes/)

---

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 👤 Manage Microsoft Entra Users and Groups

### Creating and Managing Users

Users in Entra ID can be **cloud-only** (created in Entra ID) or **synchronised** from on-premises AD DS via Entra Connect.

**User types:**

| Type | Description |
|------|-------------|
| **Member** | Standard user within the tenant |
| **Guest** | External user invited via B2B collaboration |
| **Service Account** | User account used for non-human workloads (legacy; prefer Managed Identity) |

**Creating a user — Portal path:** `Microsoft Entra ID → Users → New user → Create user`

```bash
# Azure CLI
az ad user create \
  --display-name "Alice Smith" \
  --user-principal-name alice@contoso.com \
  --password "P@ssword123" \
  --force-change-password-next-sign-in true
```

```powershell
# PowerShell
$passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
$passwordProfile.Password = "P@ssword123"
New-AzureADUser -DisplayName "Alice Smith" `
  -UserPrincipalName "alice@contoso.com" `
  -PasswordProfile $passwordProfile `
  -AccountEnabled $true
```

**Key user properties:**

| Property | Notes |
|----------|-------|
| **UPN (User Principal Name)** | Primary login identity; format `user@domain.com` |
| **Object ID** | Immutable GUID assigned on creation |
| **Usage Location** | Required to assign licences |
| **Account Enabled** | Boolean — disabled accounts can't sign in but retain data |

> ⚠️ **Exam Caveat:** A user must have a **Usage Location** set before you can assign an Entra ID / Microsoft 365 licence. This is a common exam trap.

### Managing Groups

| Group Type | Purpose | Member Assignment |
|------------|---------|-------------------|
| **Security** | Control access to resources | Manual, dynamic |
| **Microsoft 365** | Collaboration (Exchange, Teams, SharePoint) | Manual, dynamic |

**Membership types:**

| Type | Description |
|------|-------------|
| **Assigned** | Admin manually adds members |
| **Dynamic User** | Members auto-added based on user attributes (requires Entra ID P1) |
| **Dynamic Device** | Members auto-added based on device attributes |

**Dynamic group rule example:**

```
(user.department -eq "Engineering") and (user.country -eq "DE")
```

> ⚠️ **Exam Caveat:** Dynamic group membership **requires Entra ID P1 or P2** licence. You cannot add or remove members manually from a dynamic group.

### Managing Licences

- Licences (e.g. Microsoft 365, Entra ID P1) are assigned at the **user** or **group** level
- **Group-based licensing** inherits licences from a group (requires Entra ID P1)
- If a user has no Usage Location, licence assignment will fail
- **Inherited licences** cannot be individually modified — change the group assignment

```bash
# Assign licence via CLI (requires service plan/SKU ID)
az ad user update --id alice@contoso.com --usage-location DE
```

### Managing External Users (B2B)

- External users are invited as **Guests** via email
- Guests authenticate with their own identity provider (Microsoft, Google, etc.)
- Access can be scoped with RBAC just like member users
- **Entra External ID (B2B collaboration):** Invite external partners
- **Bulk invite** supported via CSV through the portal

> ⚠️ **Exam Caveat:** Guest users by default have **limited access** to read directory info. Admins can adjust this via External Collaboration Settings.

### Self-Service Password Reset (SSPR)

SSPR allows users to reset their own password without helpdesk involvement.

| Setting | Options |
|---------|---------|
| **Enabled for** | None / Selected (group) / All |
| **Auth methods required** | 1 or 2 |
| **Available methods** | Email, mobile phone, authenticator app, security questions, office phone |
| **Licence required** | Entra ID P1 (for employees); Free tier for cloud-only admins |

**SSPR flow:** User navigates to `aka.ms/sspr` → Verifies identity → Resets password

> ⚠️ **Exam Caveat:** SSPR **writeback** to on-premises AD requires **Entra ID P1** and **Entra Connect** with password writeback enabled.

---

## 🛡️ Manage Access to Azure Resources (RBAC)

### Role-Based Access Control Overview

RBAC controls **who** can do **what** on **which** resources. Every access decision is: `Who (Security Principal) + What (Role Definition) + Where (Scope)`.

```
Security Principal
├── User
├── Group
├── Service Principal
└── Managed Identity

Role Definition (permissions)
├── Actions (allowed control-plane operations)
├── NotActions (excluded from Actions)
├── DataActions (allowed data-plane operations)
└── NotDataActions

Scope (where the role applies)
├── Management Group
├── Subscription
├── Resource Group
└── Resource
```

### Built-in Roles — Key Roles

| Role | Can Do | Cannot Do |
|------|--------|-----------|
| **Owner** | Everything, including assign roles | — |
| **Contributor** | Create/manage resources | Assign roles, manage blueprint |
| **Reader** | View resources | Any write or action |
| **User Access Administrator** | Manage user access | Manage resources themselves |
| **Storage Blob Data Contributor** | Read/write blob data | Manage storage account settings |
| **Virtual Machine Contributor** | Manage VMs | VNet, storage account, assign roles |
| **Network Contributor** | Manage networking | VMs, storage |
| **Billing Reader** | View billing | Any change |

> ⚠️ **Exam Caveat:** **Contributor** cannot assign roles. **Owner** can. **User Access Administrator** can assign roles but not manage resources. These distinctions are frequently tested.

### Assigning Roles

```bash
# Assign Owner role at subscription scope
az role assignment create \
  --assignee alice@contoso.com \
  --role "Owner" \
  --scope "/subscriptions/<sub-id>"

# Assign Contributor at resource group scope
az role assignment create \
  --assignee alice@contoso.com \
  --role "Contributor" \
  --scope "/subscriptions/<sub-id>/resourceGroups/MyRG"

# List assignments
az role assignment list --assignee alice@contoso.com -o table
```

### Role Assignment Scope Hierarchy

```
Management Group        ← broadest scope
└── Subscription
    └── Resource Group
        └── Resource    ← narrowest scope
```

- Roles are **inherited downward**: Contributor on a subscription applies to all resource groups and resources in it
- **Deny assignments** can block inherited permissions (set by Azure Blueprints or Azure Policy)
- **Effective permissions** = union of all role assignments - deny assignments

> ⚠️ **Exam Caveat:** There is **no deny role** in standard RBAC — to block access you remove role assignments or use **Deny Assignments** (only set by Blueprints, not manually).

### Interpreting Access Assignments

**Effective access = all assigned roles at all scopes, merged**

Example:
- User has `Reader` on Subscription → can read all resources
- User has `Contributor` on Resource Group `ProdRG` → can create/manage in ProdRG
- Effective: `Contributor` in ProdRG, `Reader` everywhere else

```bash
# Check effective permissions for a user on a resource
az role assignment list --assignee alice@contoso.com --all -o table

# Check what a specific user can do on a resource
az role definition list --custom-role-only false --query "[].{Name:roleName, Actions:permissions[0].actions[0]}"
```

### Custom Roles

When built-in roles don't fit, create custom roles:

```json
{
  "Name": "VM Start/Stop Operator",
  "Description": "Can start and stop VMs but not create or delete",
  "Actions": [
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/deallocate/action",
    "Microsoft.Compute/virtualMachines/read"
  ],
  "NotActions": [],
  "AssignableScopes": ["/subscriptions/<sub-id>"]
}
```

```bash
az role definition create --role-definition custom-role.json
```

---

## 🏛️ Manage Azure Subscriptions and Governance

### Azure Policy

Azure Policy evaluates resources for **compliance** against defined rules. It does not immediately prevent non-compliant resources unless using `Deny` effect.

**Policy effects (in order of restrictiveness):**

| Effect | Behaviour |
|--------|-----------|
| **Disabled** | Policy rule not evaluated |
| **Audit** | Log non-compliant resources; no blocking |
| **AuditIfNotExists** | Audit if a related resource doesn't exist |
| **Append** | Add fields to the resource on create/update |
| **Modify** | Add, update, or remove tags/properties |
| **DeployIfNotExists** | Deploy a related resource if it doesn't exist |
| **Deny** | Block the request |

**Policy scope:** Assign at Management Group, Subscription, or Resource Group.

```bash
# List built-in policies
az policy definition list --query "[?policyType=='BuiltIn'].{Name:displayName}" -o table

# Assign a policy
az policy assignment create \
  --name "RequireTagEnvironment" \
  --policy "<policy-definition-id>" \
  --scope "/subscriptions/<sub-id>/resourceGroups/MyRG" \
  --params '{"tagName": {"value": "Environment"}}'
```

**Policy Initiative (Policy Set):** A collection of policies grouped together for a compliance goal (e.g. "CIS Azure Benchmark").

> ⚠️ **Exam Caveat:** A `Deny` policy prevents resource creation but does **not** remove existing non-compliant resources. Use `DeployIfNotExists` or `Modify` for remediation.

### Resource Locks

Locks prevent **accidental deletion or modification** of resources, regardless of RBAC permissions.

| Lock Type | Prevents | Allows |
|-----------|---------|--------|
| **CanNotDelete** | Delete | Read, modify |
| **ReadOnly** | Delete AND modify | Read only |

```bash
# Apply a delete lock to a resource group
az lock create \
  --name "NoDeletionLock" \
  --lock-type CanNotDelete \
  --resource-group MyRG

# Apply a readonly lock
az lock create \
  --name "ReadOnlyLock" \
  --lock-type ReadOnly \
  --resource-group MyRG

# List locks
az lock list --resource-group MyRG

# Delete a lock
az lock delete --name "NoDeletionLock" --resource-group MyRG
```

> ⚠️ **Exam Caveat:** Even **Owner** or **Subscription Admin** cannot delete a locked resource without **first removing the lock**. Locks apply to all users including owners. `ReadOnly` on a resource group also prevents adding new resources to it.

### Tagging

Tags are **key-value metadata pairs** attached to resources for organisation, cost allocation, and governance.

```bash
# Add tags to a resource group
az group update --name MyRG --tags Environment=Production CostCenter=IT123

# Tag a specific resource
az resource tag \
  --resource-group MyRG \
  --name MyVM \
  --resource-type Microsoft.Compute/virtualMachines \
  --tags Owner=AliceSmith

# List resources by tag
az resource list --tag Environment=Production -o table
```

| Limit | Value |
|-------|-------|
| Max tags per resource | **50** |
| Max tag name length | 512 characters |
| Max tag value length | 256 characters |
| Tag inheritance | **Not automatic** — must use Azure Policy (Inherit-tag-from-RG) |

> ⚠️ **Exam Caveat:** Tags are **not inherited** by child resources from parent resource groups. Use an `Append` or `Modify` Azure Policy to enforce tag inheritance automatically.

### Managing Resource Groups

- A resource group is a **logical container** for related resources
- Resources can only be in **one** resource group at a time
- Deleting a resource group deletes **all resources inside it**
- Resources can be **moved** between resource groups (some resource types have limitations)

```bash
# Create resource group
az group create --name MyRG --location westeurope

# Move resources between RGs
az resource move \
  --destination-group TargetRG \
  --ids /subscriptions/<sub-id>/resourceGroups/MyRG/providers/Microsoft.Compute/virtualMachines/MyVM
```

> ⚠️ **Exam Caveat:** Not all resource types support move operations. Resources are briefly **locked during the move**. Verify with `az resource list-locations` before assuming a move is valid.

### Managing Subscriptions

- A subscription is tied to an **Azure AD tenant**
- Subscriptions have **service limits** (quotas); you can request increases
- Subscriptions can be **transferred** between tenants or billing accounts
- Use **management groups** to apply policies across multiple subscriptions

**Subscription management portal:** `Cost Management + Billing → Subscriptions`

### Management Groups

```
Root Management Group (Tenant Root)
└── Engineering MG
    ├── Dev Subscription
    └── Prod Subscription
└── Finance MG
    └── Finance Subscription
```

- Up to **6 levels** deep (not counting root and subscription)
- Policies and RBAC applied at MG level **inherit to all children**
- A subscription can be in **only one** management group

```bash
# Create management group
az account management-group create --name EngineeringMG --display-name "Engineering"

# Add subscription to management group
az account management-group subscription add \
  --name EngineeringMG \
  --subscription <sub-id>
```

### Cost Management — Alerts, Budgets, and Advisor

**Azure Cost Management + Billing** provides:

| Feature | Description |
|---------|-------------|
| **Cost analysis** | Visualise spend by resource, tag, resource group, subscription |
| **Budgets** | Set spending limits with alerts (does NOT stop spending) |
| **Cost alerts** | Notify when actual or forecast spend hits a threshold |
| **Recommendations** | Right-sizing suggestions from Azure Advisor |

**Budget alert types:**

| Type | Trigger |
|------|---------|
| **Actual** | Triggered when actual spend exceeds % of budget |
| **Forecast** | Triggered when forecasted spend will exceed % of budget |

**Azure Advisor Cost recommendations:**

- Shutdown/right-size underutilised VMs
- Purchase Reserved Instances for consistent workloads
- Delete unattached disks and unused resources

```bash
# Create a budget
az consumption budget create \
  --budget-name MonthlyBudget \
  --amount 500 \
  --time-grain Monthly \
  --start-date 2025-01-01 \
  --end-date 2025-12-31 \
  --resource-group MyRG
```

> ⚠️ **Exam Caveat:** Budgets send **alerts only** — they do **not** stop or throttle resource creation when the budget is exceeded. To enforce hard limits you must use Azure Policy or manual intervention.

---

## 🔄 Domain 1 Scenario Quick Reference

| Scenario | Solution |
|----------|---------|
| User can't be assigned a licence | Check if **Usage Location** is set |
| Prevent anyone from deleting a critical resource | Apply **CanNotDelete** resource lock |
| Stop a specific tag from being left empty | Azure Policy with **Deny** effect |
| Auto-inherit resource group tags to new resources | Azure Policy with **Append/Modify** effect |
| Allow a team to manage VMs but not networking | Assign **Virtual Machine Contributor** to the team at RG scope |
| Block new resources unless they have a tag | Azure Policy with **Deny** effect on tag key |
| Alert when monthly Azure spend exceeds €400 | Create a **Budget** with an Actual cost alert at 80% |
| Apply a policy to 10 subscriptions at once | Apply policy at **Management Group** scope |
| A user needs to add new members to a group but not change RBAC | Assign them **Group Owner** role (not User Access Administrator) |
| Dynamic group for all users in "Sales" department | Create Dynamic Security group with rule `user.department -eq "Sales"` |

---

[← Prerequisites](/az-104-study-notes/00-azure-prerequisites/){: .btn } [Domain 2 — Storage →](/az-104-study-notes/02-storage/){: .btn .btn-primary }
