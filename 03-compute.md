---
layout: default
title: 03 — Compute
nav_order: 5
description: "Deploy and manage Azure compute resources: ARM templates, VMs, availability options, containers, and App Service."
permalink: /03-compute/
mermaid: true
---

# 🖥️ 03 — Deploy and Manage Azure Compute Resources
{: .no_toc }

**Domain Weight: 20–25% of exam**
{: .label .label-blue }

Covers ARM templates and Bicep, virtual machines and disks, availability sets and zones, VM Scale Sets, containers, and Azure App Service.
{: .fs-5 .fw-300 }

> 📁 [← Back to Home](/az-104-study-notes/)

---

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 📄 ARM Templates and Bicep

### ARM Template Structure

ARM templates deploy resources declaratively using JSON:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vmName": {
      "type": "string",
      "defaultValue": "MyVM",
      "metadata": { "description": "Name of the VM" }
    }
  },
  "variables": {
    "nicName": "[concat(parameters('vmName'), '-nic')]"
  },
  "resources": [
    {
      "type": "Microsoft.Compute/virtualMachines",
      "apiVersion": "2024-03-01",
      "name": "[parameters('vmName')]",
      "location": "[resourceGroup().location]",
      "properties": { }
    }
  ],
  "outputs": {
    "vmId": {
      "type": "string",
      "value": "[resourceId('Microsoft.Compute/virtualMachines', parameters('vmName'))]"
    }
  }
}
```

**Key ARM template sections:**

| Section | Required | Purpose |
|---------|----------|---------|
| `$schema` | ✅ | Identifies template schema version |
| `contentVersion` | ✅ | Tracks your versioning (e.g. 1.0.0.0) |
| `parameters` | ❌ | Input values at deploy time |
| `variables` | ❌ | Reusable expressions and values |
| `resources` | ✅ | Resources to create/update |
| `outputs` | ❌ | Values returned after deployment |

**ARM template functions (common in exam):**

```
concat(...)           → concatenate strings
resourceGroup().location → current RG's region
resourceId(...)       → construct a resource ID
parameters('name')    → reference a parameter
variables('name')     → reference a variable
uniqueString(...)     → deterministic unique hash
```

### Bicep Equivalent

```bicep
param vmName string = 'MyVM'
var nicName = '${vmName}-nic'

resource vm 'Microsoft.Compute/virtualMachines@2024-03-01' = {
  name: vmName
  location: resourceGroup().location
  properties: {}
}

output vmId string = vm.id
```

**Bicep vs ARM JSON:**

| Feature | ARM JSON | Bicep |
|---------|---------|-------|
| Verbosity | High | Low |
| `dependsOn` | Explicit | Inferred from references |
| Modules | Linked templates | `module` keyword |
| Type safety | Weak | Strong |
| Preview | N/A | `az bicep decompile` to convert JSON |

### Deploying Templates

```bash
# Deploy ARM template to resource group
az deployment group create \
  --resource-group MyRG \
  --template-file azuredeploy.json \
  --parameters vmName=ProdVM

# Deploy Bicep file
az deployment group create \
  --resource-group MyRG \
  --template-file main.bicep \
  --parameters vmName=ProdVM

# What-if (preview changes without deploying)
az deployment group what-if \
  --resource-group MyRG \
  --template-file main.bicep

# Export existing resources as ARM template
az group export --name MyRG > exported-template.json
```

**Deployment modes:**

| Mode | Behaviour |
|------|-----------|
| **Incremental** (default) | Only creates/updates resources in template; does not delete extras |
| **Complete** | Deletes resources in RG not in the template; dangerous! |

> ⚠️ **Exam Caveat:** **Complete mode** deletes resources not defined in the template. Use with extreme caution. The exam may offer Complete mode as a wrong answer for "safely deploy".

---

## 🖥️ Create and Configure Virtual Machines

### VM Creation Key Parameters

```bash
az vm create \
  --resource-group MyRG \
  --name MyVM \
  --image Ubuntu2204 \
  --size Standard_D2s_v3 \
  --admin-username azureuser \
  --ssh-key-values ~/.ssh/id_rsa.pub \
  --vnet-name MyVNet \
  --subnet MySubnet \
  --public-ip-address "" \          # No public IP
  --nsg "" \                        # No NSG (use existing)
  --zone 1                          # Deploy to Availability Zone 1
```

### VM Sizes (Families)

| Series | Workload |
|--------|---------|
| **B** | Burstable; low baseline, short spikes |
| **D** | General purpose; balanced CPU/RAM |
| **E** | Memory optimised; high RAM:CPU |
| **F** | Compute optimised; high CPU:RAM |
| **L** | Storage optimised; high disk throughput |
| **M** | Largest memory (up to 4+ TB RAM) |
| **N** | GPU (NVIDIA); ML, graphics |

> ⚠️ **Exam Caveat:** VM size families are **regionally available** — not every size exists in every region. Check availability before deploying.

### VM Disks

| Disk Type | Attachment | Description |
|-----------|-----------|-------------|
| **OS Disk** | Always 1 | Boot disk; managed by Azure |
| **Temporary Disk** | Always 1 (most VMs) | Local SSD; D: or /dev/sdb; data lost on deallocation |
| **Data Disk** | 0 to N | Additional managed disks for data |

**Managed disk SKUs:**

| SKU | IOPS/Disk | Throughput | Use Case |
|-----|-----------|-----------|---------|
| **Standard HDD (S)** | Up to 500 | Up to 60 MB/s | Dev/test, infrequent access |
| **Standard SSD (E)** | Up to 6,000 | Up to 750 MB/s | Web servers, light DB |
| **Premium SSD (P)** | Up to 20,000 | Up to 900 MB/s | Production, latency-sensitive |
| **Ultra Disk** | Up to 160,000 | Up to 4,000 MB/s | Mission-critical, SAP HANA |
| **Premium SSD v2** | Up to 80,000 | Up to 1,200 MB/s | DB workloads without over-provisioning |

> ⚠️ **Exam Caveat:** **Temporary disks** (`D:` on Windows, `/dev/sdb` on Linux) are **lost on stop/deallocation**. Do not store data there. Swap/page files are fine.

### Azure Disk Encryption (ADE)

ADE encrypts OS and data disks using **BitLocker (Windows)** or **DM-Crypt (Linux)** with keys stored in **Azure Key Vault**.

```bash
# Create a Key Vault with disk encryption enabled
az keyvault create \
  --name MyKeyVault \
  --resource-group MyRG \
  --location westeurope \
  --enabled-for-disk-encryption true

# Enable ADE on a VM
az vm encryption enable \
  --resource-group MyRG \
  --name MyVM \
  --disk-encryption-keyvault MyKeyVault
```

> ⚠️ **Exam Caveat:** ADE encrypts at the **OS level**. **Server-Side Encryption (SSE)** encrypts at the **Azure storage level** and is always on. These are different layers — both can be active simultaneously.

### Moving VMs

VMs can be moved between:
- **Resource groups** (same subscription)
- **Subscriptions** (same or different tenant)
- **Regions** (using Azure Resource Mover)

```bash
# Move VM to another resource group
az resource move \
  --destination-group TargetRG \
  --ids /subscriptions/<sub>/resourceGroups/SourceRG/providers/Microsoft.Compute/virtualMachines/MyVM
```

> ⚠️ **Exam Caveat:** Moving a VM also requires moving its **dependent resources** (NIC, OS disk, public IP). The move operation locks the source and destination resource groups temporarily.

### VM Sizes — Resizing

```bash
# List available sizes in a region
az vm list-sizes --location westeurope -o table

# Resize a VM (requires stop/deallocate if new size family)
az vm resize \
  --resource-group MyRG \
  --name MyVM \
  --size Standard_D4s_v3
```

### Availability Options

| Option | SLA | Protects Against |
|--------|-----|-----------------|
| **No redundancy** | 99.9% (Premium disk) | — |
| **Availability Set** | 99.95% | Rack-level hardware failure |
| **Availability Zone** | 99.99% | Datacenter-level failure |
| **VM Scale Sets across zones** | 99.99% | Zone failure + auto-scale |

**Availability Set internals:**
- **Fault domains (FD):** Separate power and network (up to 3 FDs)
- **Update domains (UD):** Only one UD is rebooted at a time during platform updates (up to 20 UDs)

> ⚠️ **Exam Caveat:** Availability Sets must be created **before** VMs — you cannot add an existing VM to an availability set. You'd need to recreate the VM.

### VM Scale Sets (VMSS)

VMSS automatically **creates and scales** identical VM instances.

```bash
az vmss create \
  --resource-group MyRG \
  --name MyScaleSet \
  --image Ubuntu2204 \
  --instance-count 3 \
  --admin-username azureuser \
  --ssh-key-values ~/.ssh/id_rsa.pub \
  --upgrade-policy-mode Automatic \
  --zones 1 2 3
```

**Scaling modes:**

| Mode | Description |
|------|-------------|
| **Manual** | You set the instance count manually |
| **Custom autoscale** | Rules based on metrics (CPU, memory, custom) |
| **Scheduled** | Scale at defined times |

**Upgrade policies:**

| Policy | Behaviour |
|--------|-----------|
| **Automatic** | Azure updates instances immediately |
| **Rolling** | Updates instances in batches |
| **Manual** | You update each instance explicitly |

---

## 📦 Provision and Manage Containers

### Azure Container Registry (ACR)

ACR is a **private Docker registry** hosted in Azure.

```bash
# Create an ACR
az acr create \
  --resource-group MyRG \
  --name myregistry \
  --sku Basic \
  --admin-enabled false

# Build and push an image
az acr build \
  --registry myregistry \
  --image myapp:v1 .

# Pull an image
docker pull myregistry.azurecr.io/myapp:v1
```

**ACR SKUs:**

| SKU | Storage | Features |
|-----|---------|---------|
| **Basic** | 10 GB | Dev/test; no geo-replication |
| **Standard** | 100 GB | Most production workloads |
| **Premium** | 500 GB | Geo-replication, private endpoints, content trust |

> ⚠️ **Exam Caveat:** Authenticating to ACR from other Azure services (ACI, AKS, App Service) should use **Managed Identity** or a **service principal**, not admin credentials.

### Azure Container Instances (ACI)

ACI runs containers **serverlessly** — no VM management required.

```bash
az container create \
  --resource-group MyRG \
  --name mycontainer \
  --image myregistry.azurecr.io/myapp:v1 \
  --cpu 1 \
  --memory 1.5 \
  --registry-login-server myregistry.azurecr.io \
  --registry-username $(az acr credential show -n myregistry --query username -o tsv) \
  --registry-password $(az acr credential show -n myregistry --query passwords[0].value -o tsv) \
  --dns-name-label myapp-unique \
  --ports 80
```

- ACI supports **Linux and Windows** containers
- Billed per second (CPU + memory)
- Supports **container groups** (multiple containers in one host, share network)

### Azure Container Apps (ACA)

ACA is a **managed container platform** built on Kubernetes for running microservices and APIs.

| Feature | ACI | ACA |
|---------|-----|-----|
| Use case | Single containers, batch | Microservices, APIs, background workers |
| Scaling | Manual | Auto-scale to zero (Keda-based) |
| Ingress | DNS label | Built-in HTTPS ingress |
| Revisions | N/A | Rolling updates with traffic splitting |

```bash
# Create a Container Apps environment
az containerapp env create \
  --name MyEnv \
  --resource-group MyRG \
  --location westeurope

# Deploy a container app
az containerapp create \
  --name myapp \
  --resource-group MyRG \
  --environment MyEnv \
  --image myregistry.azurecr.io/myapp:v1 \
  --target-port 80 \
  --ingress external \
  --min-replicas 1 \
  --max-replicas 5
```

**Scaling rules in ACA:** HTTP traffic, CPU/memory, Azure Service Bus queue length, and custom KEDA triggers.

> ⚠️ **Exam Caveat:** ACA can scale to **zero replicas** — cost drops to $0 for idle apps. ACI does not scale automatically. AKS is the full Kubernetes experience (not covered in AZ-104 in depth).

---

## 🌐 Create and Configure Azure App Service

### App Service Plans

The **App Service Plan** defines the region, number of workers, VM size, and pricing tier.

**Pricing tiers:**

| Tier | CPU/RAM | Custom Domain | SSL | Slots | Scale-Out |
|------|---------|--------------|-----|-------|----------|
| **Free (F1)** | Shared | ❌ | ❌ | 0 | ❌ |
| **Basic (B1-B3)** | Dedicated | ✅ | ✅ | 0 | Manual (up to 3) |
| **Standard (S1-S3)** | Dedicated | ✅ | ✅ | 5 | Auto |
| **Premium v3 (P1-P3)** | Dedicated | ✅ | ✅ | 20 | Auto |
| **Isolated v2 (I1-I3)** | Dedicated VNet | ✅ | ✅ | 20 | Auto |

```bash
# Create App Service Plan
az appservice plan create \
  --name MyPlan \
  --resource-group MyRG \
  --sku P1v3 \
  --is-linux
```

### Creating and Configuring an App Service

```bash
# Create web app (Linux, Node.js)
az webapp create \
  --resource-group MyRG \
  --plan MyPlan \
  --name mywebapp-unique \
  --runtime "NODE:20-lts"
```

### Certificates and TLS

- **Free managed certificate:** App Service provides a free TLS cert for custom domains (cannot be exported)
- **App Service Certificate:** Purchased and managed through Azure; can be used across apps
- **Uploaded certificate:** Upload your own .pfx certificate

```bash
# Enforce HTTPS
az webapp update \
  --resource-group MyRG \
  --name mywebapp-unique \
  --https-only true

# Set minimum TLS version
az webapp config set \
  --resource-group MyRG \
  --name mywebapp-unique \
  --min-tls-version 1.2
```

### Custom DNS Mapping

To map `www.contoso.com` to an App Service:

1. Add a CNAME record: `www → mywebapp-unique.azurewebsites.net`
2. Verify domain ownership with a `asuid` TXT record
3. Add the custom domain in the portal: `App Service → Custom Domains → Add custom domain`

```bash
az webapp config hostname add \
  --webapp-name mywebapp-unique \
  --resource-group MyRG \
  --hostname www.contoso.com
```

### Backup for App Service

- Backups include: app content, configuration, and linked database (optional)
- Stored in an **Azure Storage Account** as a ZIP file
- Requires **Standard tier or higher**

```bash
az webapp config backup create \
  --resource-group MyRG \
  --webapp-name mywebapp-unique \
  --container-url "https://mystorageacct.blob.core.windows.net/backups?<SAS>"
```

### Networking Settings for App Service

| Feature | Direction | Description |
|---------|-----------|-------------|
| **VNet Integration** | Outbound | App calls resources in VNet |
| **Private Endpoint** | Inbound | App accessed privately from VNet |
| **Access Restrictions** | Inbound | IP-based allow/deny rules |
| **Service Endpoints** | Outbound | Secure calls to Azure PaaS from VNet |

### Deployment Slots

Deployment slots allow **blue/green deployments** — deploy to a staging slot, test, then swap to production.

- **Standard** plan: 5 slots; **Premium**: 20 slots
- **Slot swap:** App configuration is also swapped (sticky settings excluded)
- **Sticky settings:** App settings/connection strings marked sticky are NOT swapped

```bash
# Create a deployment slot
az webapp deployment slot create \
  --name mywebapp-unique \
  --resource-group MyRG \
  --slot staging

# Deploy to staging slot
az webapp deploy \
  --resource-group MyRG \
  --name mywebapp-unique \
  --slot staging \
  --src-path ./app.zip

# Swap staging to production
az webapp deployment slot swap \
  --resource-group MyRG \
  --name mywebapp-unique \
  --slot staging \
  --target-slot production
```

> ⚠️ **Exam Caveat:** Deployment slots are available from **Standard** tier and above. The **Free** and **Basic** tiers do not support slots. Auto-swap can be configured so a successful deployment to staging automatically swaps to production.

---

## 🔄 Domain 3 Scenario Quick Reference

| Scenario | Solution |
|----------|---------|
| Deploy multiple identical environments from code | ARM template or Bicep with parameters |
| Protect production VMs from a single datacenter failure | Deploy VMs across **Availability Zones** |
| Scale a web tier automatically based on CPU load | Configure **VM Scale Set** with autoscale rules |
| Run a short-lived container task without managing infrastructure | **Azure Container Instances (ACI)** |
| Deploy a microservices API that scales to zero when idle | **Azure Container Apps (ACA)** |
| Blue/green deployment for a web app | Create a **deployment slot**, deploy, then **swap** |
| Encrypt VM disks with company-controlled keys | Enable **Azure Disk Encryption (ADE)** with Key Vault |
| Map a custom domain to an App Service | Add CNAME/A record in DNS + **Add custom domain** in portal |
| Preview ARM template changes without deploying | Use `az deployment group what-if` |
| Allow App Service to access a SQL database in a private VNet | Enable **VNet Integration** on the App Service |

---

[← Domain 2 — Storage](/az-104-study-notes/02-storage/){: .btn } [Domain 4 — Networking →](/az-104-study-notes/04-virtual-networking/){: .btn .btn-primary }
