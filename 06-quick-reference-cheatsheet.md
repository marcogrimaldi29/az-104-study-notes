---
layout: default
title: 06 — Quick Reference Cheatsheet
nav_order: 8
---

# ⚡ 06 — Quick Reference Cheatsheet
{: .no_toc }

Last-minute review: key numbers, SLA tables, decision trees, CLI/PowerShell commands, SKU comparisons, and exam traps by domain.
{: .fs-5 .fw-300 }

---

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 🔢 Key Numbers to Memorise

| Item | Value |
|------|-------|
| Passing score | **700 / 1000** |
| Max tags per resource | **50** |
| Azure reserved IPs per subnet | **5** (first 4 + last 1) |
| Management group hierarchy depth | **6 levels** (below root, above subscription) |
| Subscription child MGs | A subscription can only be in **1** management group |
| AzureBastionSubnet minimum size | **/26** |
| App Service Standard deployment slots | **5** |
| App Service Premium v3 deployment slots | **20** |
| Max RBAC role assignments per subscription | **4,000** |
| Max custom roles per tenant | **5,000** |
| Storage account name max length | **24 characters** |
| Storage account name min length | **3 characters** |
| Blob Archive rehydration time (standard) | **1–15 hours** |
| Blob Archive rehydration time (priority) | **< 1 hour** |
| Blob soft delete max retention | **365 days** |
| Container soft delete max retention | **365 days** |
| File share soft delete max retention | **365 days** |
| Log Analytics metric retention (default) | **93 days** |
| Log Analytics log retention (default) | **90 days (configurable to 730)** |
| Recovery Services vault redundancy lock-in | Must set **before first backup item** |

---

## ⏱️ SLA Uptime Table

| SLA % | Max Downtime / Month |
|-------|---------------------|
| 99% | ~7.2 hours |
| 99.5% | ~3.6 hours |
| 99.9% | ~43.8 minutes |
| 99.95% | ~21.9 minutes |
| 99.99% | ~4.4 minutes |
| 99.999% | ~26 seconds |

**VM SLA by availability option:**

| Configuration | SLA |
|--------------|-----|
| Single VM, Standard HDD | No SLA |
| Single VM, Standard SSD | **99.5%** |
| Single VM, Premium SSD | **99.9%** |
| Two VMs in Availability Set | **99.95%** |
| Two VMs in Availability Zones | **99.99%** |

---

## 💾 Storage Redundancy Cheatsheet

| SKU | Copies | Locations | Readable Secondary | RPO |
|-----|--------|----------|-------------------|-----|
| LRS | 3 | 1 DC | ❌ | 0 |
| ZRS | 3 | 3 AZs | ❌ | 0 |
| GRS | 6 | 2 regions (LRS each) | ❌ | < 15 min (typical) |
| GZRS | 6 | ZRS primary + LRS secondary | ❌ | < 15 min (typical) |
| RA-GRS | 6 | 2 regions | ✅ | < 15 min (typical) |
| RA-GZRS | 6 | ZRS primary + LRS secondary | ✅ | < 15 min (typical) |

---

## 💿 Blob Storage Tier Cheatsheet

| Tier | Storage Cost | Access Cost | Min Duration | Rehydrate From Archive |
|------|-------------|------------|--------------|----------------------|
| Hot | Highest | Lowest | None | — |
| Cool | Medium | Medium | 30 days | — |
| Cold | Low | Higher | 90 days | — |
| Archive | Lowest | Highest | 180 days | 1–15 hrs (standard) |

> Archive tier requires rehydration to Hot or Cool before access.

---

## 🔐 RBAC — Key Roles Reference

| Role | Manage Resources | Assign Roles | Manage Billing |
|------|-----------------|-------------|---------------|
| Owner | ✅ | ✅ | ❌ |
| Contributor | ✅ | ❌ | ❌ |
| Reader | ❌ (read only) | ❌ | ❌ |
| User Access Administrator | ❌ | ✅ | ❌ |
| Billing Reader | ❌ | ❌ | Read only |

**Common data-plane roles:**

| Role | Service | Access |
|------|---------|--------|
| Storage Blob Data Contributor | Blob | Read/Write/Delete blobs |
| Storage Blob Data Reader | Blob | Read blobs only |
| Storage File Data SMB Share Contributor | Files | Read/Write/Delete |
| Key Vault Secrets Officer | Key Vault | Manage secrets |
| Key Vault Secrets User | Key Vault | Read secrets |

---

## 🏗️ VM Availability Options Decision Tree

```
Need HA for VMs?
│
├── Single region, protect from hardware failure only
│   └── Use Availability Set (FD: 3, UD: 20) → 99.95% SLA
│
├── Single region, protect from datacenter failure (AZ)
│   └── Use Availability Zones → 99.99% SLA
│
└── Multi-region DR
    └── Use Azure Site Recovery (ASR) for failover
```

---

## 🌐 Networking — Service vs Private Endpoint

| Feature | Service Endpoint | Private Endpoint |
|---------|-----------------|-----------------|
| Traffic stays in Azure? | ✅ | ✅ |
| PaaS gets a VNet private IP? | ❌ | ✅ |
| Works across peered VNets? | ❌ | ✅ |
| DNS change required? | ❌ | ✅ (Private DNS Zone) |
| Cost | Free | Hourly + data |
| Scenario | Simple VNet → PaaS isolation | Multi-VNet, on-prem, full private |

---

## ⚖️ Load Balancer vs Application Gateway

| Feature | Azure Load Balancer | Application Gateway |
|---------|--------------------|--------------------|
| OSI Layer | Layer 4 (TCP/UDP) | Layer 7 (HTTP/HTTPS) |
| Protocol | TCP, UDP | HTTP, HTTPS, WebSocket |
| Routing | IP + Port | URL path, hostname, headers |
| SSL Termination | ❌ | ✅ |
| WAF (Web App Firewall) | ❌ | ✅ (WAF SKU) |
| Session affinity | IP-based | Cookie-based |
| Backend types | VMs, VMSS | VMs, VMSS, App Service, IPs |
| Exam hint | Generic TCP load balancing | Web apps, API routing |

---

## 📦 Container Options Decision Tree

```
Run a container in Azure?
│
├── Short-lived / batch / sidecar → Azure Container Instances (ACI)
│
├── HTTP API / microservice, scale to zero → Azure Container Apps (ACA)
│
└── Full Kubernetes control needed → AKS (out of scope for AZ-104)
```

---

## 🔒 Entra ID Licence Requirements

| Feature | Free | P1 | P2 |
|---------|------|----|----|
| Create users/groups | ✅ | ✅ | ✅ |
| SSPR (cloud-only admins) | ✅ | ✅ | ✅ |
| SSPR (all users) | ❌ | ✅ | ✅ |
| Dynamic groups | ❌ | ✅ | ✅ |
| Group-based licensing | ❌ | ✅ | ✅ |
| Conditional Access | ❌ | ✅ | ✅ |
| Privileged Identity Management (PIM) | ❌ | ❌ | ✅ |
| Identity Protection | ❌ | ❌ | ✅ |

---

## 🚨 Exam Traps by Domain

### Domain 1 — Identity & Governance

| Trap | Reality |
|------|---------|
| "Owner can delete a locked resource" | ❌ No — locks apply to everyone, including Owner |
| "Tags inherit from resource group automatically" | ❌ No — use Azure Policy (Append/Modify) |
| "Deny effect in Azure Policy removes existing resources" | ❌ No — Deny only blocks future creates |
| "Contributor can assign RBAC roles" | ❌ No — only Owner and User Access Administrator |
| "Budget alerts stop spending when limit is hit" | ❌ No — alerts only; spending continues |
| "Dynamic group membership can be manually overridden" | ❌ No — dynamic groups are fully automated |

### Domain 2 — Storage

| Trap | Reality |
|------|---------|
| "Encryption at rest must be enabled" | ❌ It's always on — cannot be disabled |
| "GRS replication is synchronous" | ❌ No — asynchronous; RPO is non-zero |
| "Rotating a storage key is safe immediately" | ❌ Update apps FIRST, then rotate |
| "Archive tier blobs can be read directly" | ❌ Must rehydrate first (1–15 hours) |
| "azcopy copy deletes extras in destination" | ❌ Use `azcopy sync` for that behaviour |
| "Storage account names allow hyphens" | ❌ Only lowercase letters and digits |

### Domain 3 — Compute

| Trap | Reality |
|------|---------|
| "ARM Complete mode is safe for updates" | ❌ Complete mode deletes resources not in template |
| "Temporary disk data survives VM stop/deallocate" | ❌ Data is lost — temp disk is ephemeral |
| "ADE and SSE are the same" | ❌ ADE = OS-level; SSE = storage-level. Both can be active |
| "You can add a VM to an Availability Set after creation" | ❌ Set must exist first; VM must be recreated |
| "Basic App Service plan supports deployment slots" | ❌ Standard or higher required |

### Domain 4 — Networking

| Trap | Reality |
|------|---------|
| "VNet Peering is transitive" | ❌ A↔B and B↔C does NOT mean A↔C |
| "Standard Public IP is open by default" | ❌ Secure by default — requires NSG to allow inbound |
| "Bastion subnet can be named anything" | ❌ Must be exactly `AzureBastionSubnet` |
| "Service endpoints give PaaS a private IP" | ❌ Private Endpoints give a private IP; Service Endpoints don't |
| "CNAME can be used at the zone apex" | ❌ Use Alias records for apex domains |
| "Azure Load Balancer handles HTTP routing" | ❌ Layer 4 only; use Application Gateway for Layer 7 |

### Domain 5 — Monitor & Maintain

| Trap | Reality |
|------|---------|
| "Temporary disk is backed up by Azure Backup" | ❌ Temp disk is excluded from all backups |
| "Vault redundancy can be changed any time" | ❌ Must be set before first item is protected |
| "Test Failover in ASR impacts production" | ❌ Test failover uses isolated network — no impact |
| "After ASR failover, protection is automatic" | ❌ Must commit + manually re-protect (failback setup) |
| "Metric alerts work on Log Analytics KQL" | ❌ Use Log Search Alert for KQL-based conditions |

---

## 🖥️ CLI Quick Command Reference

### Identity & Governance

```bash
az ad user create --display-name "Name" --user-principal-name u@domain.com --password "P@ss"
az ad group create --display-name "GroupName" --mail-nickname "GroupName"
az role assignment create --assignee user@domain.com --role "Contributor" --scope /subscriptions/<id>
az role assignment list --assignee user@domain.com -o table
az policy assignment create --name MyPolicy --policy <id> --scope /subscriptions/<id>
az lock create --name NoDelete --lock-type CanNotDelete --resource-group MyRG
az group update --name MyRG --tags Env=Prod Owner=Alice
```

### Storage

```bash
az storage account create --name <name> --resource-group MyRG --sku Standard_LRS --kind StorageV2
az storage container create --account-name <name> --name mycontainer
az storage blob upload --account-name <name> --container-name mycontainer --name file.txt --file ./file.txt
az storage account keys list --account-name <name> --resource-group MyRG
az storage container generate-sas --account-name <name> --name mycontainer --permissions rl --expiry 2025-12-31T23:59Z
azcopy copy 'src' 'dst?<SAS>' --recursive
```

### Compute

```bash
az vm create --resource-group MyRG --name MyVM --image Ubuntu2204 --size Standard_D2s_v3 --admin-username azureuser
az vm resize --resource-group MyRG --name MyVM --size Standard_D4s_v3
az vm encryption enable --resource-group MyRG --name MyVM --disk-encryption-keyvault MyKV
az vmss create --resource-group MyRG --name MyVMSS --image Ubuntu2204 --instance-count 3
az webapp create --resource-group MyRG --plan MyPlan --name myapp --runtime "NODE:20-lts"
az webapp deployment slot create --name myapp --resource-group MyRG --slot staging
az webapp deployment slot swap --name myapp --resource-group MyRG --slot staging
az deployment group create --resource-group MyRG --template-file main.bicep
```

### Networking

```bash
az network vnet create --resource-group MyRG --name MyVNet --address-prefixes 10.0.0.0/16
az network vnet subnet create --resource-group MyRG --vnet-name MyVNet --name Subnet1 --address-prefixes 10.0.1.0/24
az network vnet peering create --resource-group MyRG --name AtoB --vnet-name VNet-A --remote-vnet VNet-B
az network nsg create --resource-group MyRG --name MyNSG
az network nsg rule create --resource-group MyRG --nsg-name MyNSG --name AllowHTTP --priority 100 --direction Inbound --protocol TCP --destination-port-ranges 80 --access Allow
az network bastion create --resource-group MyRG --name MyBastion --vnet-name MyVNet --public-ip-address MyPIP
az network lb create --resource-group MyRG --name MyLB --sku Standard --public-ip-address MyPIP
az network dns zone create --resource-group MyRG --name contoso.com
az network dns record-set a add-record --resource-group MyRG --zone-name contoso.com --record-set-name www --ipv4-address 1.2.3.4
```

### Monitor & Backup

```bash
az monitor log-analytics workspace create --resource-group MyRG --workspace-name MyWS
az monitor metrics alert create --resource-group MyRG --name CPUAlert --resource <vm-id> --condition "avg Percentage CPU > 90"
az monitor action-group create --resource-group MyRG --name MyAG --short-name AG --action email admin admin@contoso.com
az backup vault create --resource-group MyRG --name MyVault --location westeurope
az backup protection enable-for-vm --resource-group MyRG --vault-name MyVault --vm <vm-id> --policy-name DefaultPolicy
az backup job list --resource-group MyRG --vault-name MyVault -o table
```

---

## ✅ Pre-Exam Checklist

### Core Concepts
- [ ] ARM hierarchy: Tenant → MG → Subscription → RG → Resource
- [ ] RBAC scope inheritance flows downward; deny assignments override
- [ ] Owner vs Contributor vs User Access Administrator differences
- [ ] Azure Policy effects: Audit < Append/Modify < Deny < DeployIfNotExists
- [ ] Resource locks: CanNotDelete vs ReadOnly; even Owner needs to remove first

### Storage
- [ ] LRS → ZRS → GRS → GZRS → RA-GRS → RA-GZRS progression
- [ ] SAS types: Account vs Service vs User Delegation
- [ ] Blob tiers: Hot → Cool → Cold → Archive (early deletion fees)
- [ ] Archive rehydration: 1–15 hours standard, <1 hour priority
- [ ] Encryption at rest: always on (SSE), cannot disable

### Compute
- [ ] ARM Incremental vs Complete mode
- [ ] VM availability: Availability Set (99.95%) vs Zones (99.99%)
- [ ] Temp disk: lost on stop/deallocate, not backed up
- [ ] ADE (BitLocker/DM-Crypt) vs SSE (Azure Storage layer)
- [ ] Deployment slots: Standard+ only; sticky settings not swapped

### Networking
- [ ] VNet Peering: non-transitive, bidirectional
- [ ] NSG: applied at subnet AND/OR NIC; both must allow for traffic
- [ ] Standard Public IP: secure by default (all inbound blocked)
- [ ] AzureBastionSubnet: exact name required, minimum /26
- [ ] Service Endpoint vs Private Endpoint: no private IP vs private IP
- [ ] Load Balancer (Layer 4) vs Application Gateway (Layer 7)
- [ ] CNAME cannot be at apex; use Alias record

### Monitor & Backup
- [ ] Metrics: 93-day retention; Logs: 90-day default (configurable)
- [ ] KQL basics: where, project, summarize, extend, render
- [ ] Alert types: Metric, Log Search, Activity Log
- [ ] Vault redundancy must be set before first backup
- [ ] ASR: test failover (non-disruptive) vs planned vs unplanned
- [ ] After failover: must commit, then re-protect for failback

---

**Good luck on the exam! 🎯**

[← Monitor & Maintain](./05-monitor-maintain/){: .btn } [Back to Index](./){: .btn .btn-primary }
