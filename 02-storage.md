---
layout: default
title: 02 — Storage
nav_order: 4
description: "Implement and manage Azure Storage accounts, redundancy, SAS tokens, access policies, encryption, Azure Files, Blob storage, lifecycle management."
permalink: /02-storage/
mermaid: true
---

# 🗄️ 02 — Implement and Manage Storage
{: .no_toc }

**Domain Weight: 15–20% of exam**
{: .label .label-blue }

Covers storage accounts, redundancy, SAS tokens, access policies, encryption, Azure Files, Blob storage, lifecycle management, and data tools.
{: .fs-5 .fw-300 }

> 📁 [← Back to Home](/az-104-study-notes/)

---

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 🔐 Configure Access to Storage

### Storage Account Access Methods

Azure Storage provides multiple access control mechanisms:

| Method | Best For |
|--------|---------|
| **Account key (access key)** | Full admin access; not recommended for apps |
| **Shared Access Signature (SAS)** | Delegated, time-limited, scoped access |
| **Entra ID + RBAC** | Identity-based; recommended for applications |
| **Stored access policy** | Manage and revoke SAS tokens centrally |
| **Anonymous public access** | Read-only public blob access (must be explicitly enabled) |

### Azure Storage Firewalls and Virtual Networks

By default, storage accounts accept traffic from all networks. The **firewall** restricts this.

```bash
# Restrict storage account to specific VNet and subnet
az storage account update \
  --name mystorageacct \
  --resource-group MyRG \
  --default-action Deny

az storage account network-rule add \
  --account-name mystorageacct \
  --resource-group MyRG \
  --vnet-name MyVNet \
  --subnet MySubnet

# Allow specific public IP
az storage account network-rule add \
  --account-name mystorageacct \
  --resource-group MyRG \
  --ip-address 203.0.113.0/24
```

**Trusted Microsoft services bypass:** Some Azure services (e.g. Azure Backup, Azure Monitor) can be allowed even with a restrictive firewall by enabling "Allow trusted Microsoft services".

> ⚠️ **Exam Caveat:** Setting `--default-action Deny` blocks **all** traffic including from the Azure portal unless you add your IP or enable "Allow Azure services".

### Shared Access Signature (SAS) Tokens

SAS tokens delegate access to storage **without sharing account keys**.

**Types of SAS:**

| Type | Signed By | Scope |
|------|-----------|-------|
| **Account SAS** | Account key | One or more storage services |
| **Service SAS** | Account key | Specific resource (blob, container, file share, queue, table) |
| **User Delegation SAS** | Entra ID credential | Blob and Data Lake Storage Gen2 only |

> ⚠️ **Exam Caveat:** A service SAS token or an account SAS token is authorized with Shared Key and will not be permitted on a request to Blob storage when the `AllowSharedKeyAccess` property is set to false.
> A user delegation SAS is authorized with Microsoft Entra ID and will be permitted on a request to Blob storage when the `AllowSharedKeyAccess` property is set to false.

**SAS token components:**

```
https://mystorageacct.blob.core.windows.net/mycontainer/myblob.txt
?sv=2024-11-04         ← storage service version
&ss=b                  ← storage service (b=blob)
&srt=o                 ← resource type (o=object)
&sp=r                  ← permissions (r=read)
&se=2025-12-31T00:00Z  ← expiry time (UTC)
&spr=https             ← allowed protocol
&sig=<signature>       ← HMAC-SHA256 signature
```

**Creating a SAS in CLI:**

```bash
# Service SAS for a blob container (read + list for 1 day)
az storage container generate-sas \
  --account-name mystorageacct \
  --name mycontainer \
  --permissions rl \
  --expiry 2025-12-31T23:59Z \
  --https-only \
  --output tsv
```

> ⚠️ **Exam Caveat:** A SAS token can only be revoked by **rotating the account key** it was signed with (for Account/Service SAS) or by **deleting the stored access policy** it references. Plan accordingly.

### Stored Access Policies

Stored access policies let you **define and control** SAS tokens centrally without regenerating them.

```bash
# Create a stored access policy on a container
az storage container policy create \
  --account-name mystorageacct \
  --container-name mycontainer \
  --name ReadPolicy \
  --permissions r \
  --expiry 2025-12-31T23:59Z

# Create a SAS referencing the stored access policy
az storage container generate-sas \
  --account-name mystorageacct \
  --name mycontainer \
  --policy-name ReadPolicy \
  --output tsv
```

**To revoke:** Delete or update the stored access policy → all SAS tokens referencing it are immediately invalidated.

### Managing Access Keys

Each storage account has **two access keys** (Key1 and Key2) for rotation without downtime.

```bash
# List keys
az storage account keys list --account-name mystorageacct --resource-group MyRG

# Rotate (regenerate) a key
az storage account keys renew \
  --account-name mystorageacct \
  --resource-group MyRG \
  --key key1
```

**Best practice:** Store access keys in **Azure Key Vault**, not in application config files.

> ⚠️ **Exam Caveat:** Rotating a key **immediately invalidates** all connection strings and SAS tokens signed with that key. Always update applications before rotating.

### Identity-Based Access for Azure Files

Azure Files supports **Entra ID Kerberos** authentication for cloud-based file shares, and **on-premises AD DS** authentication for hybrid scenarios.

| Auth Method | Protocol | Scenario |
|-------------|----------|---------|
| **Entra ID Kerberos** | SMB | Cloud-only or hybrid; Entra-joined devices |
| **On-premises AD DS** | SMB | Hybrid; domain-joined on-prem machines |
| **Storage account key** | SMB/NFS | Admin access; no per-user permissions |

**Assign share-level RBAC role for Files:**

- `Storage File Data SMB Share Reader`
- `Storage File Data SMB Share Contributor`
- `Storage File Data SMB Share Elevated Contributor` (for NTFS permissions)

---

## 🔧 Configure and Manage Storage Accounts

### Creating and Configuring Storage Accounts

```bash
az storage account create \
  --name mystorageacct \
  --resource-group MyRG \
  --location westeurope \
  --sku Standard_LRS \
  --kind StorageV2 \
  --access-tier Hot \
  --allow-blob-public-access false \
  --min-tls-version TLS1_2
```

**Key settings at creation:**

| Setting | Options | Notes |
|---------|---------|-------|
| **Performance** | Standard / Premium | Premium = NVMe; cannot change after creation |
| **Redundancy** | LRS, ZRS, GRS, GZRS, RA-GRS, RA-GZRS | Can change after creation (with limitations) |
| **Access tier** | Hot / Cool | Default tier for blobs; can override per blob |
| **Min TLS** | TLS 1.0, 1.1, 1.2 | Set to TLS 1.2 for security |
| **Blob public access** | Enabled / Disabled | Disable unless you need public blobs |

> ⚠️ **Exam Caveat:** Storage account names must be **globally unique**, **3–24 characters**, **lowercase letters and numbers only**, no hyphens or special characters.

### Azure Storage Redundancy — Deep Dive

| SKU | Copies | Regions | Read on Secondary | Use Case |
|-----|--------|---------|-------------------|---------|
| **LRS** | 3 | 1 datacenter | ❌ | Dev/test, non-critical |
| **ZRS** | 3 | 3 AZs, 1 region | ❌ | HA within region |
| **GRS** | 6 | Primary (LRS) + Secondary (LRS) | ❌ | Disaster recovery |
| **GZRS** | 6 | Primary (ZRS) + Secondary (LRS) | ❌ | Best resilience |
| **RA-GRS** | 6 | Primary + Secondary | ✅ (secondary endpoint) | Read-heavy with DR |
| **RA-GZRS** | 6 | Primary ZRS + Secondary | ✅ (secondary endpoint) | Best resilience + read |

**Secondary endpoint format:** `https://<account>-secondary.blob.core.windows.net`

> ⚠️ **Exam Caveat:** Data replication to the secondary region is **asynchronous**. There can be a **RPO (Recovery Point Objective)** gap. In a failover, the secondary becomes the primary — this is a **customer-initiated** action (not automatic for standard GRS).

### Object Replication

Replicates **block blobs** between source and destination storage accounts asynchronously.

- Destination can be in a **different region** (cross-region) or same region
- Requires **blob versioning** and **change feed** enabled on source
- Destination container must exist

```bash
# Enable versioning (required for object replication)
az storage account blob-service-properties update \
  --account-name mystorageacct \
  --resource-group MyRG \
  --enable-versioning true \
  --enable-change-feed true
```

### Storage Account Encryption

| Type | Default | Notes |
|------|---------|-------|
| **Encryption at rest** | ✅ Always enabled | AES-256; cannot be disabled |
| **Microsoft-managed keys (MMK)** | ✅ Default | Keys managed by Microsoft |
| **Customer-managed keys (CMK)** | Optional | Keys in Azure Key Vault; you control rotation |
| **Infrastructure encryption** | Optional | Double encryption at infrastructure layer |
| **Encryption in transit** | Configurable | Enforce HTTPS with `--https-only true` |

> ⚠️ **Exam Caveat:** Encryption at rest is **always on** — you cannot disable it. The exam may try to trick you into choosing "Enable encryption" as an action.

### Azure Storage Explorer and AzCopy

**Azure Storage Explorer:** GUI desktop tool to browse and manage storage accounts, containers, blobs, file shares, queues, and tables.

**AzCopy:** Command-line utility optimised for **large-scale data transfers**.

```bash
# Copy a local folder to Blob
azcopy copy 'C:\data\*' 'https://mystorageacct.blob.core.windows.net/mycontainer/' --recursive

# Copy between storage accounts
azcopy copy \
  'https://source.blob.core.windows.net/container?<SAS>' \
  'https://dest.blob.core.windows.net/container?<SAS>' \
  --recursive

# Sync (source → destination, delete extras in destination)
azcopy sync \
  'C:\data' \
  'https://mystorageacct.blob.core.windows.net/mycontainer' \
  --recursive
```

> ⚠️ **Exam Caveat:** `azcopy sync` is like `robocopy /MIR` — it deletes files in the destination that don't exist in the source. `azcopy copy` does not delete.

---

## 📁 Configure Azure Files and Azure Blob Storage

### Azure Files — File Shares

Azure Files provides **fully managed SMB and NFS file shares** in the cloud.

| Protocol | Supported OS | Share Type |
|----------|-------------|-----------|
| **SMB 3.0** | Windows, Linux, macOS | Standard and Premium |
| **NFS 4.1** | Linux only | Premium only |
| **REST API** | Any | Standard and Premium |

```bash
# Create a file share
az storage share-rm create \
  --resource-group MyRG \
  --storage-account mystorageacct \
  --name myshare \
  --quota 100  # GiB

# Mount on Linux (SMB)
sudo mount -t cifs //mystorageacct.file.core.windows.net/myshare /mnt/myshare \
  -o username=mystorageacct,password=<key>,serverino
```

**Azure File Sync:** Extends Azure Files to on-premises Windows Servers.
- Caches frequently accessed files on-prem
- Sync groups define which servers and Azure file shares are synchronised
- **Cloud tiering:** Infrequently accessed files stored only in Azure (stub on server)

### Azure File Share Snapshots and Soft Delete

- **Snapshots:** Point-in-time, read-only copies of an entire file share; incremental
- **Soft delete for file shares:** Retains deleted shares for a configurable period (1–365 days)

```bash
# Enable soft delete on file shares
az storage account file-service-properties update \
  --account-name mystorageacct \
  --resource-group MyRG \
  --enable-delete-retention true \
  --delete-retention-days 14
```

### Azure Blob Storage — Container Configuration

Blob storage stores **unstructured object data** in containers.

**Blob types:**

| Type | Description | Use Case |
|------|-------------|---------|
| **Block blob** | Data split into blocks; optimised for upload | Files, images, videos, backups |
| **Append blob** | Blocks appended only (no update/delete) | Log files |
| **Page blob** | Random-write optimised; 512-byte pages | VM unmanaged disks (legacy) |

```bash
# Create a container
az storage container create \
  --account-name mystorageacct \
  --name mycontainer \
  --public-access off  # off, blob, or container

# Upload a blob
az storage blob upload \
  --account-name mystorageacct \
  --container-name mycontainer \
  --name myfile.txt \
  --file /local/path/myfile.txt \
  --tier Hot
```

**Public access levels:**

| Level | What's Public |
|-------|--------------|
| **Private** (off) | Nothing — all requests need auth |
| **Blob** | Individual blob URLs are public |
| **Container** | Container + all blob URLs are public |

### Storage Tiers

| Tier | Access / Transaction Cost | Storage Cost | Min Duration | Use Case |
|------|------------|-------------|--------------|---------|
| **Hot** | Low | High | None | Frequently accessed data |
| **Cool** | Medium | Medium | 30 days | Infrequent, at-least monthly access |
| **Cold** | High | Low | 90 days | Infrequent, at-least quarterly access |
| **Archive** | Highest | Lowest | 180 days | Rarely accessed; hours to rehydrate |

**Rehydration from Archive:** Move to Hot or Cool tier first — takes 1–15 hours standard, or up to 1 hour with Priority rehydration.

> ⚠️ **Exam Caveat:** Archive tier blobs **cannot be read directly**. You must first rehydrate archived blobs to Hot or Cool. Treat them as offline files. Early deletion from Cool (<30d), Cold (<90d), or Archive (<180d) incurs early deletion charges.

### Soft Delete for Blobs and Containers

```bash
# Enable blob soft delete (14-day retention)
az storage account blob-service-properties update \
  --account-name mystorageacct \
  --resource-group MyRG \
  --enable-delete-retention true \
  --delete-retention-days 14

# Enable container soft delete
az storage account blob-service-properties update \
  --account-name mystorageacct \
  --resource-group MyRG \
  --enable-container-delete-retention true \
  --container-delete-retention-days 7
```

### Blob Versioning

- Automatically preserves previous versions of blobs on every write/delete
- Enables restoration of previous versions
- Requires enabling on the storage account (not per-container)

```bash
az storage account blob-service-properties update \
  --account-name mystorageacct \
  --resource-group MyRG \
  --enable-versioning true
```

### Blob Lifecycle Management

Automates **tier transitions** and **deletions** based on rules:

```json
{
  "rules": [
    {
      "name": "TierAndDelete",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "baseBlob": {
            "tierToCool": { "daysAfterModificationGreaterThan": 30 },
            "tierToArchive": { "daysAfterModificationGreaterThan": 90 },
            "delete": { "daysAfterModificationGreaterThan": 365 }
          },
          "snapshot": {
            "delete": { "daysAfterCreationGreaterThan": 90 }
          }
        },
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["logs/"]
        }
      }
    }
  ]
}
```

---

## 🔄 Domain 2 Scenario Quick Reference

| Scenario | Solution |
|----------|---------|
| Grant time-limited read access to a blob without sharing account key | Generate a **User Delegation SAS** |
| Revoke SAS tokens immediately | Delete the **stored access policy** they reference or **rotate the account key** (only for Account and Service SAS) |
| Prevent accidental blob deletion | Enable **soft delete** on the storage account |
| Replicate blobs to another region asynchronously | Configure **object replication** (requires blob versioning and change feed) |
| Allow VMs in a VNet to access storage without internet | Add a **service endpoint** or **private endpoint** for the storage account |
| Copy 10 TB of data from on-prem to Azure | Use **AzCopy** or **Azure Data Box** (for very large datasets, offline) |
| Automatically move logs to Archive after 90 days | Configure a **lifecycle management policy** |
| Enforce encryption with company-managed keys | Use **Customer-Managed Keys (CMK)** stored in Azure Key Vault |
| Mount a file share on Windows Server | Use **Azure Files** via SMB with storage account key |
| Reduce storage costs for blobs rarely accessed | Move to **Cool** or **Archive** tier; use lifecycle management |

---

[← Domain 1 — Identity](/az-104-study-notes/01-identity-governance/){: .btn } [Domain 3 — Compute →](/az-104-study-notes/03-compute/){: .btn .btn-primary }
