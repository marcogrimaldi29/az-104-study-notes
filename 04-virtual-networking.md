---
layout: default
title: 04 — Virtual Networking
nav_order: 6
description: "Implement and manage virtual networking: VNets, NSGs, peering, Bastion, DNS, service endpoints, private endpoints, and load balancing."
permalink: /04-virtual-networking/
mermaid: true
---

# 🌐 04 — Implement and Manage Virtual Networking
{: .no_toc }

**Domain Weight: 15–20% of exam**
{: .label .label-blue }

Covers VNets, subnets, peering, NSGs, ASGs, route tables, Azure Bastion, DNS, service endpoints, private endpoints, and load balancing.
{: .fs-5 .fw-300 }

> 📁 [← Back to Home](/az-104-study-notes/)

---

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 🔧 Configure and Manage Virtual Networks

### Virtual Networks (VNets)

A VNet is Azure's foundational **private network** — isolated, region-scoped, and supports subnetting.

```bash
# Create a VNet with address space
az network vnet create \
  --resource-group MyRG \
  --name MyVNet \
  --address-prefixes 10.0.0.0/16 \
  --location westeurope

# Add a subnet
az network vnet subnet create \
  --resource-group MyRG \
  --vnet-name MyVNet \
  --name WebSubnet \
  --address-prefixes 10.0.1.0/24

# Add another subnet
az network vnet subnet create \
  --resource-group MyRG \
  --vnet-name MyVNet \
  --name DbSubnet \
  --address-prefixes 10.0.2.0/24
```

**Azure Reserved IP Addresses (first 4 + last 1 per subnet):**

For `10.0.1.0/24`:

| Address | Reserved For |
|---------|-------------|
| 10.0.1.0 | Network address |
| 10.0.1.1 | Default gateway |
| 10.0.1.2–10.0.1.3 | Azure DNS mapping |
| 10.0.1.255 | Broadcast |

> ⚠️ **Exam Caveat:** Azure reserves **5 IP addresses** per subnet. A /29 subnet has 8 IPs total — only **3 usable**. Always account for this in sizing calculations.

### VNet Peering

Peering connects two VNets so traffic flows directly (not over internet), using **Azure backbone network**.

```bash
# Peer VNet-A to VNet-B
az network vnet peering create \
  --resource-group MyRG \
  --name AtoB \
  --vnet-name VNet-A \
  --remote-vnet VNet-B \
  --allow-vnet-access

# Peer VNet-B to VNet-A (peering is NOT bidirectional automatically)
az network vnet peering create \
  --resource-group MyRG \
  --name BtoA \
  --vnet-name VNet-B \
  --remote-vnet VNet-A \
  --allow-vnet-access
```

**Peering key rules:**

| Rule | Detail |
|------|--------|
| **Non-transitive** | A↔B and B↔C does NOT mean A↔C; need separate peering |
| **Address space** | Cannot overlap between peered VNets |
| **Global peering** | Peering across regions is supported |
| **Bidirectional** | Must create peering in both directions |

**Peering options:**

| Option | Effect |
|--------|--------|
| **Allow virtual network access** | Enables traffic flow |
| **Allow forwarded traffic** | Allow traffic that was forwarded (not originated) from remote VNet |
| **Allow gateway transit** | Let remote VNet use this VNet's VPN Gateway |
| **Use remote gateways** | Use remote VNet's gateway (need "Allow gateway transit" on remote) |

> ⚠️ **Exam Caveat:** VNet Peering is **not transitive**. If you need hub-spoke connectivity where all spokes share one gateway, use **Azure Virtual WAN** or **User-Defined Routes (UDRs)** with an NVA.

### Public IP Addresses

```bash
# Create a Standard Public IP (zone-redundant)
az network public-ip create \
  --resource-group MyRG \
  --name MyPublicIP \
  --sku Standard \
  --allocation-method Static \
  --zone 1 2 3  # Zone-redundant
```

| SKU | Allocation | Default Security | AZ Support |
|-----|-----------|-----------------|-----------|
| **Basic** (deprecated) | Static or Dynamic | Open by default | ❌ |
| **Standard** | Static only | Closed by default (requires NSG) | ✅ |

> ⚠️ **Exam Caveat:** Standard Public IPs are **secure by default** — all inbound traffic is blocked unless explicitly allowed by an NSG. Basic IPs are **open by default** and are being deprecated.

### User-Defined Routes (UDRs)

UDRs override Azure's default routing to send traffic through a custom path (e.g. firewall appliance).

```bash
# Create a route table
az network route-table create \
  --resource-group MyRG \
  --name MyRouteTable \
  --location westeurope

# Add a route (send all internet traffic through a firewall NVA)
az network route-table route create \
  --resource-group MyRG \
  --route-table-name MyRouteTable \
  --name ForceToFirewall \
  --next-hop-type VirtualAppliance \
  --next-hop-ip-address 10.0.0.4 \
  --address-prefix 0.0.0.0/0

# Associate route table with a subnet
az network vnet subnet update \
  --resource-group MyRG \
  --vnet-name MyVNet \
  --name WebSubnet \
  --route-table MyRouteTable
```

**Next hop types:**

| Type | Description |
|------|-------------|
| **VirtualNetworkGateway** | On-prem via VPN/ExpressRoute |
| **VnetLocal** | Within the VNet |
| **Internet** | Public internet |
| **VirtualAppliance** | Custom IP (NVA, firewall) |
| **None** | Drop the packet (blackhole) |

### Troubleshooting Network Connectivity

**Azure Network Watcher tools:**

| Tool | Purpose |
|------|---------|
| **IP flow verify** | Test if a packet will be allowed/denied by NSG |
| **Next hop** | Show the next hop for a packet from a VM |
| **Connection troubleshoot** | Test connectivity between two endpoints |
| **Connection Monitor** | Continuous connectivity monitoring |
| **Packet capture** | Capture network traffic on a VM |
| **NSG flow logs** | Log all NSG decisions to a storage account |
| **Topology** | Visualise network topology |

```bash
# IP flow verify
az network watcher test-ip-flow \
  --resource-group MyRG \
  --vm MyVM \
  --direction Inbound \
  --protocol TCP \
  --local 10.0.1.4:80 \
  --remote 203.0.113.0:12345

# Check next hop
az network watcher show-next-hop \
  --resource-group MyRG \
  --vm MyVM \
  --source-ip 10.0.1.4 \
  --dest-ip 8.8.8.8
```

---

## 🔒 Configure Secure Access to Virtual Networks

### Network Security Groups (NSGs)

NSGs are **stateful firewalls** that filter inbound and outbound traffic using security rules.

**Rule properties:**

| Property | Description |
|----------|-------------|
| **Priority** | 100–4096; lower = higher priority |
| **Source/Destination** | IP, CIDR, service tag, or ASG |
| **Protocol** | TCP, UDP, ICMP, ESP, AH, or Any |
| **Port range** | Single port or range (e.g. 80, 443, 8080-8090) |
| **Action** | Allow or Deny |

**Default NSG rules (cannot be deleted):**

| Priority | Name | Direction | Source | Dest | Action |
|----------|------|-----------|--------|------|--------|
| 65000 | AllowVnetInbound | Inbound | VirtualNetwork | VirtualNetwork | Allow |
| 65001 | AllowAzureLoadBalancerInbound | Inbound | AzureLoadBalancer | Any | Allow |
| 65500 | DenyAllInbound | Inbound | Any | Any | Deny |
| 65000 | AllowVnetOutbound | Outbound | VirtualNetwork | VirtualNetwork | Allow |
| 65001 | AllowInternetOutbound | Outbound | Any | Internet | Allow |
| 65500 | DenyAllOutbound | Outbound | Any | Any | Deny |

```bash
# Create an NSG
az network nsg create --resource-group MyRG --name MyNSG

# Add allow rule for HTTP
az network nsg rule create \
  --resource-group MyRG \
  --nsg-name MyNSG \
  --name AllowHTTP \
  --priority 100 \
  --direction Inbound \
  --protocol TCP \
  --source-address-prefixes "*" \
  --destination-port-ranges 80 \
  --access Allow

# Associate NSG with a subnet
az network vnet subnet update \
  --resource-group MyRG \
  --vnet-name MyVNet \
  --name WebSubnet \
  --network-security-group MyNSG

# Associate NSG with a NIC
az network nic update \
  --resource-group MyRG \
  --name MyVM-NIC \
  --network-security-group MyNSG
```

> ⚠️ **Exam Caveat:** NSGs can be applied at both **subnet** and **NIC** level. If both are applied, traffic must pass BOTH NSG evaluations. Subnet NSG is evaluated first for inbound, NIC NSG first for outbound.

### Application Security Groups (ASGs)

ASGs allow you to group VMs logically and use those groups in NSG rules instead of IP addresses.

```bash
# Create ASGs
az network asg create --resource-group MyRG --name WebServers
az network asg create --resource-group MyRG --name DbServers

# Add VM NIC to an ASG
az network nic ip-config update \
  --resource-group MyRG \
  --nic-name MyWebVM-NIC \
  --name ipconfig1 \
  --application-security-groups WebServers

# Use ASG in NSG rule (allow web servers to talk to db servers on 1433)
az network nsg rule create \
  --resource-group MyRG \
  --nsg-name MyNSG \
  --name WebToDb \
  --priority 200 \
  --direction Inbound \
  --protocol TCP \
  --source-asgs WebServers \
  --destination-asgs DbServers \
  --destination-port-ranges 1433 \
  --access Allow
```

### Effective Security Rules

When troubleshooting, view the **effective security rules** — the merged result of all NSGs applied to a NIC:

```bash
az network nic list-effective-nsg \
  --resource-group MyRG \
  --name MyVM-NIC \
  -o table
```

> ⚠️ **Exam Caveat:** If you can't figure out why traffic is blocked, always check effective rules. A deny in the subnet NSG can block even if the NIC NSG allows it.

### Azure Bastion

Azure Bastion provides **browser-based RDP/SSH** access to VMs without public IPs.

```bash
# Create Bastion (requires AzureBastionSubnet with min /26)
az network vnet subnet create \
  --resource-group MyRG \
  --vnet-name MyVNet \
  --name AzureBastionSubnet \
  --address-prefixes 10.0.100.0/26

az network public-ip create \
  --resource-group MyRG \
  --name BastionPublicIP \
  --sku Standard

az network bastion create \
  --resource-group MyRG \
  --name MyBastion \
  --vnet-name MyVNet \
  --public-ip-address BastionPublicIP \
  --location westeurope \
  --sku Basic
```

**Bastion SKUs:**

| SKU | Concurrent Sessions | Native Client | Tunneling |
|-----|--------------------|--------------|---------:|
| **Basic** | Limited | ❌ | ❌ |
| **Standard** | Higher | ✅ | ✅ |
| **Premium** | Highest | ✅ | ✅ + session recording |

> ⚠️ **Exam Caveat:** The Bastion subnet **must** be named exactly `AzureBastionSubnet`. It cannot be any other name. Minimum size is **/26**.

### Service Endpoints

Service endpoints extend your VNet's private address space to Azure PaaS services, routing traffic over the Azure backbone.

```bash
# Enable service endpoint for storage on a subnet
az network vnet subnet update \
  --resource-group MyRG \
  --vnet-name MyVNet \
  --name WebSubnet \
  --service-endpoints Microsoft.Storage

# Restrict storage account to only allow from that subnet
az storage account network-rule add \
  --account-name mystorageacct \
  --resource-group MyRG \
  --vnet-name MyVNet \
  --subnet WebSubnet
```

**Service endpoints vs Private endpoints:**

| Feature | Service Endpoint | Private Endpoint |
|---------|-----------------|-----------------|
| Source IP | Private VNet IP | Private VNet IP |
| Traffic path | Azure backbone; exits VNet | Stays fully within VNet |
| PaaS service exposure | Still has public endpoint | Gets a private IP in VNet |
| DNS change needed | ❌ | ✅ |
| Cost | Free | Per hour + data |
| Cross-VNet access | ❌ | ✅ (via peering or private DNS) |

### Private Endpoints

Private endpoints give PaaS services a **private IP in your VNet** — no public endpoint required.

```bash
# Create private endpoint for a storage account
az network private-endpoint create \
  --resource-group MyRG \
  --name MyStoragePrivateEndpoint \
  --vnet-name MyVNet \
  --subnet DbSubnet \
  --private-connection-resource-id /subscriptions/<sub>/resourceGroups/MyRG/providers/Microsoft.Storage/storageAccounts/mystorageacct \
  --group-id blob \
  --connection-name StoragePEConnection
```

**Private DNS zone for auto-resolution:**

After creating a private endpoint, add a **Private DNS Zone** so the resource FQDN resolves to the private IP:

| Service | DNS Zone |
|---------|---------|
| Azure Blob | `privatelink.blob.core.windows.net` |
| Azure Files | `privatelink.file.core.windows.net` |
| Azure SQL | `privatelink.database.windows.net` |
| Azure Key Vault | `privatelink.vaultcore.azure.net` |

---

## 🌍 Configure Name Resolution and Load Balancing

### Azure DNS

**Public DNS Zone:** Host publicly accessible DNS records.

```bash
# Create a public zone
az network dns zone create \
  --resource-group MyRG \
  --name contoso.com

# Add an A record
az network dns record-set a add-record \
  --resource-group MyRG \
  --zone-name contoso.com \
  --record-set-name www \
  --ipv4-address 203.0.113.10

# Add a CNAME
az network dns record-set cname set-record \
  --resource-group MyRG \
  --zone-name contoso.com \
  --record-set-name blog \
  --cname mywebapp.azurewebsites.net
```

**Private DNS Zone:** Name resolution within VNets only.

```bash
# Create private zone
az network private-dns zone create \
  --resource-group MyRG \
  --name contoso.internal

# Link to a VNet (with auto-registration = auto-create records for VMs)
az network private-dns link vnet create \
  --resource-group MyRG \
  --zone-name contoso.internal \
  --name MyVNetLink \
  --virtual-network MyVNet \
  --registration-enabled true
```

**Common DNS record types:**

| Type | Purpose |
|------|---------|
| **A** | Map hostname → IPv4 address |
| **AAAA** | Map hostname → IPv6 address |
| **CNAME** | Alias → another hostname (no apex support) |
| **MX** | Mail exchange |
| **TXT** | Text; used for domain verification |
| **NS** | Name server delegation |
| **SOA** | Start of authority; required for each zone |
| **Alias** | Azure-native; supports apex, tracks IP changes |

> ⚠️ **Exam Caveat:** **CNAME** records cannot be used at the zone apex (`@` / root domain). Use an **Alias record** pointing to a Traffic Manager, Front Door, or Public IP resource instead.

### Azure Load Balancer

Azure Load Balancer distributes inbound traffic across backend VMs.

**SKUs:**

| SKU | Scope | AZ Support | Backend | Notes |
|-----|-------|-----------|---------|-------|
| **Basic** (deprecated) | Regional | ❌ | Availability Set or VMSS | Free |
| **Standard** | Regional/Global | ✅ | VMs, VMSS in VNet | Secure by default; SLA |
| **Gateway** | Regional | ✅ | NVAs (Network Virtual Appliances) | For NVA chaining |

**Load Balancer components:**

| Component | Purpose |
|-----------|---------|
| **Frontend IP** | Public or private IP that receives traffic |
| **Backend pool** | VMs or VMSS instances that receive traffic |
| **Health probe** | HTTP, HTTPS, or TCP check; determines if backend is healthy |
| **Load balancing rule** | Maps frontend IP:port to backend pool:port |
| **Inbound NAT rule** | Maps frontend IP:port to a specific VM:port (for RDP/SSH per VM) |
| **Outbound rule** | SNAT configuration for outbound from backend VMs |

```bash
# Create a Standard public load balancer
az network lb create \
  --resource-group MyRG \
  --name MyLB \
  --sku Standard \
  --frontend-ip-name FrontendIP \
  --public-ip-address MyPublicIP \
  --backend-pool-name BackendPool

# Create health probe
az network lb probe create \
  --resource-group MyRG \
  --lb-name MyLB \
  --name HttpProbe \
  --protocol Http \
  --port 80 \
  --path /health

# Create load balancing rule
az network lb rule create \
  --resource-group MyRG \
  --lb-name MyLB \
  --name HttpRule \
  --protocol Tcp \
  --frontend-port 80 \
  --backend-port 80 \
  --frontend-ip-name FrontendIP \
  --backend-pool-name BackendPool \
  --probe-name HttpProbe
```

**Session persistence (affinity):**

| Mode | Behaviour |
|------|-----------|
| **None** | Any healthy backend can serve any request |
| **Client IP** | Same client IP always goes to same backend |
| **Client IP and protocol** | Both IP and protocol pinned |

> ⚠️ **Exam Caveat:** Azure Load Balancer operates at **Layer 4** (TCP/UDP). It does not inspect HTTP headers or URLs. For Layer 7 routing (path-based, host-based), use **Application Gateway**.

### Troubleshoot Load Balancing

Common issues and checks:

| Issue | Check |
|-------|-------|
| Backend unhealthy | Health probe failing? Check app is listening on probe port |
| Traffic not reaching backends | NSG blocking probe port or traffic port? |
| Source port exhaustion | Check SNAT allocation; add outbound rules |
| Session dropping | Client affinity configured? Long-lived sessions need affinity |

```bash
# View load balancer health probe status
az network lb probe show \
  --resource-group MyRG \
  --lb-name MyLB \
  --name HttpProbe

# List backend pool members
az network lb address-pool show \
  --resource-group MyRG \
  --lb-name MyLB \
  --name BackendPool
```

---

## 🔄 Domain 4 Scenario Quick Reference

| Scenario | Solution |
|----------|---------|
| Provide RDP/SSH to VMs without exposing public IPs | **Azure Bastion** (requires AzureBastionSubnet with minimum CIDR size /26) |
| Allow traffic between two VNets without internet | **VNet Peering** (both directions required) |
| Route all internet traffic through a firewall | **User-Defined Route** with next-hop VirtualAppliance |
| Allow web servers to talk to DB servers, not vice versa | **NSG rule** using **ASGs** |
| Access Azure Storage without leaving the VNet | **Service Endpoint** (free) or **Private Endpoint** (private IP) |
| Resolve internal hostnames within a VNet | **Azure Private DNS Zone** linked to VNet |
| Distribute HTTP traffic across 3 web VMs | **Azure Load Balancer (Standard)** with health probe |
| Route traffic by URL path (/api vs /web) | **Application Gateway** (Layer 7, not covered by Load Balancer) |
| Host DNS for a custom domain in Azure | **Azure DNS Public Zone** |
| Block all internet inbound except port 443 | NSG inbound rule: Allow 443, ensure DenyAll has higher priority number |

---

[← Domain 3 — Compute](/az-104-study-notes/03-compute/){: .btn } [Domain 5 — Monitor & Maintain →](/az-104-study-notes/05-monitor-maintain/){: .btn .btn-primary }
