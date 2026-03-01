---
layout: default
title: 00 — Azure Prerequisites
nav_order: 2
description: "Core Azure architecture fundamentals: regions, availability zones, etc."
permalink: /00-azure-prerequisites/
mermaid: true
---
---

# 📘 00 — Azure Prerequisites & Fundamentals
{: .no_toc }

Core concepts you must know before tackling any exam domain.
{: .fs-5 .fw-300 }

> 📁 [← Back to Home](/az-104-study-notes/)

---

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 🌍 Global Infrastructure

### Regions and Geographies

A **geography** is a discrete market (e.g. Europe, Asia Pacific) that preserves data-residency boundaries. Each geography contains one or more **regions**.

A **region** is a set of datacenters connected by a low-latency network, usually having less than 2ms latency. When you deploy a resource, you choose a region — resources are hosted in that region unless explicitly replicated.

| Concept | Description | Example |
|---------|-------------|---------|
| **Region Pair** | Two regions in the same geography, ≥ 300 miles apart | East US ↔ West US |
| **Sovereign Region** | Isolated cloud for compliance (gov/national) | Azure Government, Azure China 21Vianet |
| **Special Region** | Single-region geographies with no pair | Brazil South |

> ⚠️ **Exam Caveat:** Services replicate platform updates sequentially across region pairs — only one region updated at a time. This protects against a bad update taking down both regions simultaneously.

### Availability Zones (AZs)

Availability Zones are **physically separate datacenters** within a single region. Each zone has independent power, cooling, and networking. They are designed to protect applications and data from datacenter failures. Not all regions have AZs, but when they do, you can choose to deploy resources in specific zones or use zone-redundant services.

```
Region: West Europe
├── Zone 1 (Datacenter A)
├── Zone 2 (Datacenter B)
└── Zone 3 (Datacenter C)
```

| AZ Resource Type | Behaviour |
|------------------|-----------|
| **Zonal** | Pinned to a specific zone (e.g. a VM in Zone 1) |
| **Zone-redundant** | Automatically spread across zones (e.g. zone-redundant storage) |
| **Regional** | Services that span all AZs by default (e.g. Azure DNS) |

**SLA improvement:**
- Single VM + Premium SSD = **99.9%**
- Two VMs in an Availability Set = **99.95%**
- Two VMs in Availability Zones = **99.99%**

### Datacenters and Edge

- **Datacenters** host the physical hardware — not directly accessible by customers
- **Azure Edge Zones** extend compute to 5G network edges for low-latency scenarios - see also Points of Presence (PoPs) and Microsoft Enterprise Edge (MSEE)
- **Azure Stack** brings Azure services on-premises

---

## 🏗️ Azure Resource Manager (ARM)

ARM is the **management layer** for all Azure resources. Every action — portal, CLI, PowerShell, REST API, Terraform — goes through ARM.

```
User / Tool
    ↓
Authentication (MicrosoftEntra ID)
    ↓
Azure Resource Manager (ARM)
    ↓
Resource Providers (Microsoft.Compute, Microsoft.Storage, ...)
    ↓
Resources (VMs, Storage Accounts, ...)
```

### Resource Hierarchy

```
Azure Tenant (Entra ID)
└── Management Groups (optional, hierarchical)
    └── Subscriptions
        └── Resource Groups
            └── Resources
```

| Level | Purpose |
|-------|---------|
| **Management Group** | Apply policies and RBAC across multiple subscriptions |
| **Subscription** | Billing boundary and primary isolation unit |
| **Resource Group** | Logical container; all resources have a lifecycle |
| **Resource** | Individual service instance (VM, VNet, etc.) |

> ⚠️ **Exam Caveat:** A resource group has a **region**, but resources inside it can be in **different regions**. The resource group region only stores metadata, it's a logical grouping mechanism.

### Resource Providers

Resource providers expose the operations for a service type. You must **register** a provider before using it.

```bash
# List registered providers
az provider list --query "[?registrationState=='Registered']" -o table

# Register a provider
az provider register --namespace Microsoft.Compute
```

### ARM Templates vs Bicep

| Feature | ARM Template (JSON) | Bicep |
|---------|---------------------|-------|
| Format | JSON | Domain-specific language (DSL) |
| Verbosity | High | Low (transpiles to ARM JSON) |
| Modules | Linked templates | Native modules |
| Type safety | Limited | Strong |
| Tooling | VS Code extension | VS Code extension + CLI |
| Exam relevance | Interpret & modify | Interpret & modify |

> ⚠️ **Exam Caveat:** Bicep is a **first-class citizen** on AZ-104 (April 2025 update). You must be able to read and modify both ARM JSON and Bicep files.

---

## 🔑 Identity Fundamentals

### Microsoft Entra ID (formerly Azure AD)

Entra ID is Azure's cloud-based **Identity and Access Management (IAM)** service. It is **not** the same as on-premises Active Directory Domain Services (AD DS).

| Concept | Entra ID | On-prem AD DS |
|---------|----------|---------------|
| Protocol | OAuth 2.0, OpenID Connect, SAML | Kerberos, NTLM, LDAP |
| Structure | Flat (no OUs) | Hierarchical (OUs, GPOs) |
| Join type | Entra ID Join / Hybrid Join | Domain Join |
| Auth scope | Cloud-first | On-premises first |

### Key Identity Concepts

- **Tenant:** A dedicated instance of Entra ID for an organisation
- **User:** A person or service account identity
- **Group:** Security or Microsoft 365 group for bulk access assignment
- **Service Principal:** Identity for an application or automated workload
- **Managed Identity:** Auto-managed service principal for Azure resources (System-assigned or User-assigned)

---

## 💰 Cost and Billing Fundamentals

### Subscription Types

| Type | Use Case |
|------|----------|
| **Free** | 12 months free services + $200 credit |
| **Pay-As-You-Go** | Standard billing by consumption |
| **Enterprise Agreement (EA)** | Org-level discount commitment |
| **CSP** | Purchased through a Microsoft Partner |

### Cost Management Tools

- **Azure Cost Management + Billing:** Analyse spend, set budgets, create cost alerts
- **Azure Pricing Calculator:** Estimate costs before deployment
- **Azure TCO Calculator:** Compare on-prem vs Azure total cost
- **Azure Advisor:** Personalised recommendations including cost saving

---

## 📐 SLA and Composite SLA

### Reading SLA Tables

SLA = Service Level Agreement — the uptime guarantee. Calculated as:

```
Uptime % → Max Downtime Per Month
99%      → ~7.2 hours
99.9%    → ~43.8 minutes
99.95%   → ~21.9 minutes
99.99%   → ~4.4 minutes
99.999%  → ~26 seconds
```

### Composite SLA

When resources depend on each other, multiply their SLAs:

```
Two services in series:
SLA_composite = SLA_A × SLA_B
e.g. 0.999 × 0.999 = 0.998001 → 99.8%

Two services in parallel (redundant):
SLA_composite = 1 − ((1 − SLA_A) × (1 − SLA_B))
e.g. 1 − (0.001 × 0.001) = 0.999999 → 99.9999%
```

> ⚠️ **Exam Caveat:** Adding redundancy always improves the composite SLA. If one path fails, the other takes over.

---

## 🌐 Networking Fundamentals

### Virtual Network (VNet) Basics

- A VNet is a **logical network boundary** in Azure, scoped to a region
- VNets are divided into **subnets** — each subnet must have a non-overlapping address space
- Default VNet-to-VNet traffic is blocked unless using **VNet Peering** or a VPN Gateway
- Azure resources in the same VNet can communicate by default

### IP Addressing

| Type | Description |
|------|-------------|
| **Private IP** | From VNet address space; not routable on internet |
| **Public IP (Basic SKU)** | Deprecated — avoid in new deployments |
| **Public IP (Standard SKU)** | Zone-redundant by default; secure by default (requires explicit inbound rules) |
| **Static IP** | IP does not change across restarts |
| **Dynamic IP** | IP may change when deallocated |

### DNS in Azure

- **Azure-provided DNS:** Automatic; resolves Azure resource names within VNet
- **Custom DNS:** Point to on-prem DNS or your own server; set at VNet level
- **Azure DNS:** Host public or private DNS zones in Azure

---

## 🗄️ Storage Fundamentals

### Storage Account Types

| Type | Supported Services | Use Case |
|------|--------------------|----------|
| **Standard general-purpose v2 (GPv2)** | Blob, Files, Queues, Tables | Default; most scenarios |
| **Premium block blobs** | Block blobs only | High transaction, low latency |
| **Premium file shares** | Azure Files only | High-IOPS SMB/NFS workloads |
| **Premium page blobs** | Page blobs (VM disks) | Unmanaged disks (legacy) |

### Redundancy Overview

| SKU | Copies | Geographic Spread |
|-----|--------|-------------------|
| LRS | 3 | Single datacenter |
| ZRS | 3 | 3 AZs in one region |
| GRS | 6 | Primary + paired region (LRS each) |
| GZRS | 6 | ZRS in primary + LRS in paired region |
| RA-GRS / RA-GZRS | 6 | GRS/GZRS + read access on secondary |

---

## 🔧 Azure CLI & PowerShell Quick Reference

### Azure CLI Patterns

```bash
# Login
az login

# Set subscription
az account set --subscription "<name-or-id>"

# Create resource group
az group create --name MyRG --location eastus

# General resource create pattern
az <service> create --resource-group MyRG --name MyResource [options]

# List resources
az <service> list --resource-group MyRG -o table

# Delete
az <service> delete --name MyResource --resource-group MyRG --yes
```

### PowerShell Patterns

```powershell
# Login
Connect-AzAccount

# Set subscription
Set-AzContext -SubscriptionId "<id>"

# Create resource group
New-AzResourceGroup -Name MyRG -Location eastus

# General resource create pattern
New-Az<Service> -ResourceGroupName MyRG -Name MyResource [parameters]

# List resources
Get-Az<Service> -ResourceGroupName MyRG

# Remove
Remove-Az<Service> -Name MyResource -ResourceGroupName MyRG -Force
```

---

## ✅ Prerequisites Checklist

Before starting domain study, confirm you can answer:

- [ ] What is the difference between a region, a geography, and an availability zone?
- [ ] What is the ARM resource hierarchy from top to bottom?
- [ ] What is a resource provider and how do you register one?
- [ ] What is the difference between Entra ID and on-prem AD DS?
- [ ] What does 99.95% SLA mean in minutes of downtime per month?
- [ ] What is the difference between LRS, ZRS, GRS, and GZRS?
- [ ] What SKU of Public IP is required for Availability Zone support?

---

[← Back to Home](/az-104-study-notes/){: .btn } [Domain 1 — Identity & Governance →](/az-104-study-notes/01-identity-governance/){: .btn .btn-primary }
