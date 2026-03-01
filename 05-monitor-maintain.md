---
layout: default
title: 05 — Monitor & Maintain
nav_order: 7
description: "Monitor and maintain Azure resources: Azure Monitor, Log Analytics, alerts, action groups, VM Insights, Network Watcher, Azure Backup, Recovery Services vaults, and Azure Site Recovery."
permalink: /05-monitor-maintain/
mermaid: true
---

# 📊 05 — Monitor and Maintain Azure Resources
{: .no_toc }

**Domain Weight: 10–15% of exam**
{: .label .label-blue }

Covers Azure Monitor, Log Analytics, alerts, action groups, VM Insights, Network Watcher, Azure Backup, Recovery Services vaults, and Azure Site Recovery.
{: .fs-5 .fw-300 }

---

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 📈 Monitor Resources in Azure

### Azure Monitor Architecture

Azure Monitor is the **central monitoring platform** for all Azure resources.

```
Azure Resources (VMs, Storage, SQL, App Service...)
         ↓
  Azure Monitor (collection layer)
    ├── Metrics (time-series numerical data)
    └── Logs (structured/unstructured records)
         ↓
  Log Analytics Workspace (logs storage + query)
         ↓
  Action Layers
    ├── Alerts → Action Groups → (Email, SMS, Webhook, ITSM, runbook...)
    ├── Dashboards & Workbooks
    └── Insights (VM Insights, Network Insights, Container Insights...)
```

### Metrics in Azure Monitor

**Metrics** are lightweight, near-real-time numerical values (e.g. CPU %, requests/sec).

| Property | Value |
|----------|-------|
| Retention | 93 days (standard) |
| Granularity | 1 minute to 1 hour |
| Storage | Azure Monitor Metrics store (no workspace needed) |
| Query | Metrics Explorer in portal |

**Common metric namespaces:**

| Resource | Key Metrics |
|---------|------------|
| **VM** | Percentage CPU, Network In/Out, Disk Read/Write |
| **Storage Account** | Transactions, Availability, Egress, Ingress |
| **App Service** | HTTP 2xx/4xx/5xx, Response Time, CPU Time |
| **SQL Database** | DTU consumption, deadlocks, connections |

```bash
# List available metrics for a VM
az monitor metrics list-definitions \
  --resource /subscriptions/<sub>/resourceGroups/MyRG/providers/Microsoft.Compute/virtualMachines/MyVM \
  -o table

# Query CPU metric for last hour
az monitor metrics list \
  --resource /subscriptions/<sub>/resourceGroups/MyRG/providers/Microsoft.Compute/virtualMachines/MyVM \
  --metric "Percentage CPU" \
  --interval PT1M \
  --aggregation Average \
  -o table
```

### Log Analytics Workspace

Log Analytics Workspace is the **storage and query engine** for Azure Monitor Logs (also called **Log Analytics logs** or **Azure Monitor Logs**).

```bash
# Create a workspace
az monitor log-analytics workspace create \
  --resource-group MyRG \
  --workspace-name MyWorkspace \
  --location westeurope \
  --sku PerGB2018 \
  --retention-time 90
```

**Connecting resources to send logs:**

| Resource | Method |
|---------|--------|
| VM | Azure Monitor Agent (AMA) + Data Collection Rule |
| Storage Account | Diagnostic settings |
| VNet NSG | NSG flow logs |
| Activity Log (subscription) | Diagnostic settings to workspace |
| App Service | Diagnostic settings |

```bash
# Create diagnostic setting to send VM metrics/logs to workspace
az monitor diagnostic-settings create \
  --name VMDiagSettings \
  --resource /subscriptions/<sub>/resourceGroups/MyRG/providers/Microsoft.Compute/virtualMachines/MyVM \
  --workspace /subscriptions/<sub>/resourceGroups/MyRG/providers/Microsoft.OperationalInsights/workspaces/MyWorkspace \
  --metrics '[{"category": "AllMetrics", "enabled": true}]' \
  --logs '[{"category": "VMInsightsHealth", "enabled": true}]'
```

### Querying Logs with KQL

Azure Monitor Logs uses **Kusto Query Language (KQL)**.

**Common KQL patterns:**

```kql
// Heartbeat — check which agents are reporting in
Heartbeat
| summarize LastHeartbeat = max(TimeGenerated) by Computer
| where LastHeartbeat < ago(1h)

// CPU over 80% in last 24h
Perf
| where CounterName == "% Processor Time"
| where CounterValue > 80
| summarize avg(CounterValue) by Computer, bin(TimeGenerated, 5m)

// Failed logins on Windows VMs
SecurityEvent
| where EventID == 4625
| summarize count() by Account, Computer
| order by count_ desc

// Application exceptions from App Insights
exceptions
| where timestamp > ago(24h)
| summarize count() by type, outerMessage
| order by count_ desc

// Storage account transactions
AzureMetrics
| where ResourceProvider == "MICROSOFT.STORAGE"
| where MetricName == "Transactions"
| summarize sum(Total) by bin(TimeGenerated, 1h)
```

**KQL key operators:**

| Operator | Purpose |
|----------|---------|
| `where` | Filter rows |
| `project` | Select/rename columns |
| `summarize` | Aggregate (count, avg, sum, max) |
| `extend` | Add computed columns |
| `join` | Join two tables |
| `render` | Visualise (timechart, barchart, etc.) |
| `ago(1h)` | Time expression: 1 hour ago |
| `bin(Time, 5m)` | Round time to 5-minute buckets |

### Setting Up Alert Rules

Alerts trigger when a condition is met. Components: **alert rule → condition → action group**.

**Alert rule types:**

| Type | Source | Use Case |
|------|--------|---------|
| **Metric alert** | Azure Monitor Metrics | CPU > 90%, response time > 2s |
| **Log search alert** | Log Analytics KQL query | Error count > 10 in 5 min |
| **Activity log alert** | Azure Activity Log | VM deleted, policy assignment changed |
| **Smart detection alert** | Application Insights | Anomalous failure rates |

```bash
# Create a metric alert for CPU > 90%
az monitor metrics alert create \
  --resource-group MyRG \
  --name HighCPUAlert \
  --resource /subscriptions/<sub>/resourceGroups/MyRG/providers/Microsoft.Compute/virtualMachines/MyVM \
  --condition "avg Percentage CPU > 90" \
  --evaluation-frequency PT1M \
  --window-size PT5M \
  --severity 2 \
  --action /subscriptions/<sub>/resourceGroups/MyRG/providers/microsoft.insights/actionGroups/MyActionGroup \
  --description "VM CPU above 90% for 5 minutes"
```

**Alert severity levels:**

| Severity | Level |
|----------|-------|
| 0 | Critical |
| 1 | Error |
| 2 | Warning |
| 3 | Informational |
| 4 | Verbose |

### Action Groups

Action groups define **what happens** when an alert fires. One alert can trigger multiple actions.

**Action types:**

| Type | Description |
|------|-------------|
| **Email/SMS/Push/Voice** | Notify a person |
| **Azure Function** | Trigger a function |
| **Logic App** | Start a workflow |
| **Webhook** | HTTP POST to a URL |
| **ITSM** | Create ticket in ServiceNow, etc. |
| **Automation Runbook** | Run a PowerShell runbook |
| **Event Hubs** | Stream alert to Event Hubs |
| **Secure Webhook** | Webhook with Entra ID auth |

```bash
# Create action group
az monitor action-group create \
  --resource-group MyRG \
  --name MyActionGroup \
  --short-name MyAG \
  --action email alice alice@contoso.com
```

### Alert Processing Rules

**Alert processing rules** suppress or reroute alerts — useful for maintenance windows.

- **Suppression:** Silence alerts during scheduled maintenance
- **Action group override:** Add/remove action groups from triggered alerts

```bash
az monitor alert-processing-rule create \
  --resource-group MyRG \
  --name MaintenanceSuppress \
  --scopes /subscriptions/<sub>/resourceGroups/MyRG \
  --rule-type Suppression \
  --schedule-start-datetime "2025-12-24 00:00:00" \
  --schedule-end-datetime "2025-12-26 00:00:00"
```

### VM Insights, Storage Insights, and Network Insights

**VM Insights:** Pre-built workbooks for VM performance and dependency mapping.
- Requires Azure Monitor Agent (AMA) + Dependency agent
- Provides: CPU, memory, disk, network utilisation; Map view of connections

**Storage Insights:** Overview of all storage accounts — transactions, errors, latency, capacity.

**Network Insights:** Topology view, health overview, metrics for VNets, LBs, VPN Gateways.

### Azure Network Watcher and Connection Monitor

| Tool | Frequency | Use Case |
|------|-----------|---------|
| **Connection Troubleshoot** | On-demand | One-time connectivity test |
| **Connection Monitor** | Continuous | Ongoing network health monitoring |
| **NSG Flow Logs** | Continuous | Log all traffic decisions for analysis |
| **Packet Capture** | On-demand | Capture packets for deep analysis |

```bash
# Enable Network Watcher (per region)
az network watcher configure \
  --locations westeurope \
  --enabled true \
  --resource-group NetworkWatcherRG

# Enable NSG flow logs
az network watcher flow-log create \
  --resource-group MyRG \
  --nsg MyNSG \
  --storage-account mystorageacct \
  --workspace MyWorkspace \
  --enabled true \
  --name MyNSGFlowLog
```

---

## 💾 Implement Backup and Recovery

### Recovery Services Vault

A Recovery Services vault stores backup data and is used by both **Azure Backup** and **Azure Site Recovery**.

```bash
# Create Recovery Services vault
az backup vault create \
  --resource-group MyRG \
  --name MyRSVault \
  --location westeurope

# Enable soft delete (default: enabled, 14 days)
az backup vault backup-properties set \
  --resource-group MyRG \
  --name MyRSVault \
  --soft-delete-feature-state Enable
```

**Vault redundancy options:**

| Setting | Replication | Use Case |
|---------|------------|---------|
| **Locally Redundant (LRS)** | 3 copies, 1 datacenter | Low cost; acceptable if no DR needed |
| **Geo-Redundant (GRS)** | 6 copies, 2 regions | Default; recommended for DR |
| **Zone-Redundant (ZRS)** | 3 copies, 3 AZs | HA within region |

> ⚠️ **Exam Caveat:** Vault redundancy must be set **before the first backup item is registered**. You cannot change it after items are protected.

### Azure Backup Vault

**Backup Vault** is a newer vault type used for newer backup datasources (not Recovery Services):

| Datasource | Vault Type |
|-----------|------------|
| Azure VMs | Recovery Services Vault |
| SQL in VMs | Recovery Services Vault |
| Azure Files | Recovery Services Vault |
| Azure Blobs | Backup Vault |
| Azure Disks | Backup Vault |
| PostgreSQL | Backup Vault |

### Backup Policies

Backup policies define **schedule** and **retention**.

```bash
# Show default VM backup policy
az backup policy show \
  --resource-group MyRG \
  --vault-name MyRSVault \
  --name DefaultPolicy

# Enable VM backup
az backup protection enable-for-vm \
  --resource-group MyRG \
  --vault-name MyRSVault \
  --vm /subscriptions/<sub>/resourceGroups/MyRG/providers/Microsoft.Compute/virtualMachines/MyVM \
  --policy-name DefaultPolicy
```

**Policy retention examples:**

| Recovery Point | Retention |
|---------------|-----------|
| Daily | 7–9999 days |
| Weekly | 1–5163 weeks |
| Monthly | 1–1188 months |
| Yearly | 1–99 years |

### Perform Backup and Restore

```bash
# Run an on-demand backup immediately
az backup protection backup-now \
  --resource-group MyRG \
  --vault-name MyRSVault \
  --container-name iaasvmcontainer;iaasvmcontainerv2;MyRG;MyVM \
  --item-name MyVM \
  --backup-management-type AzureIaasVM \
  --retain-until 2025-12-31

# List restore points
az backup recoverypoint list \
  --resource-group MyRG \
  --vault-name MyRSVault \
  --container-name iaasvmcontainer;iaasvmcontainerv2;MyRG;MyVM \
  --item-name MyVM \
  --backup-management-type AzureIaasVM \
  -o table

# Restore a VM (to new VM)
az backup restore restore-disks \
  --resource-group MyRG \
  --vault-name MyRSVault \
  --container-name iaasvmcontainer;iaasvmcontainerv2;MyRG;MyVM \
  --item-name MyVM \
  --rp-name <recovery-point-name> \
  --storage-account mystorageacct \
  --restore-to-staging-storage-account true
```

**Restore types:**

| Type | Description |
|------|-------------|
| **Restore VM** | Create new VM from recovery point |
| **Restore disks** | Restore managed disks; attach manually |
| **File recovery** | Mount recovery point as drive; copy individual files |
| **Cross-region restore** | Restore to secondary region (requires GRS vault) |

> ⚠️ **Exam Caveat:** Azure Backup **does not protect temporary disks**. Data on the temp disk (`D:` / `/dev/sdb`) is not included in backups.

### Azure Site Recovery (ASR)

ASR provides **disaster recovery** — replicates VMs to a secondary region so you can failover.

**ASR components:**

| Component | Role |
|-----------|------|
| **Source region** | Where VMs currently run |
| **Target region** | Failover destination |
| **Recovery Services Vault** | Stores replication config (must be in target region) |
| **Replication policy** | RPO/RTO targets; app-consistent snapshot frequency |
| **Recovery plan** | Ordered failover of multiple VMs with custom scripts |

```bash
# Enable replication for a VM (portal is easier; CLI shows concept)
az site-recovery protection-container-mapping create \
  --resource-group MyRG \
  --vault-name MyRSVault \
  --fabric-name westeurope \
  --protection-container-name westeurope-container \
  --name MyReplicationMapping \
  --policy-id <policy-id>
```

**ASR failover types:**

| Failover Type | Description | Use Case |
|--------------|-------------|---------|
| **Test failover** | Non-disruptive; creates VM in isolated network | DR drill |
| **Planned failover** | Zero data loss; shut down source first | Planned migration |
| **Unplanned failover** | Uses latest recovery point; source may be down | Actual disaster |

> ⚠️ **Exam Caveat:** After failover, you must **commit** the failover to complete the process, then set up **replication back** (re-protect) to the original region if you want the ability to fail back.

### Configure Reports and Alerts for Backups

**Backup Center:** Unified view of all backup data across vaults, subscriptions, and regions.

```bash
# List backup jobs
az backup job list \
  --resource-group MyRG \
  --vault-name MyRSVault \
  -o table

# Set up backup alerts (via Azure Monitor)
az monitor metrics alert create \
  --resource-group MyRG \
  --name BackupFailureAlert \
  --resource /subscriptions/<sub>/resourceGroups/MyRG/providers/Microsoft.RecoveryServices/vaults/MyRSVault \
  --condition "count BackupHealthEvent > 0" \
  --window-size PT1H \
  --severity 1
```

**Built-in backup alerts types:**

| Alert | Trigger |
|-------|---------|
| **Backup failure** | Job fails |
| **Delete backup data** | Protected item deleted |
| **Soft delete** | Item moved to soft-delete state |
| **Stop protection** | Backup protection disabled |

---

## 🔄 Domain 5 Scenario Quick Reference

| Scenario | Solution |
|----------|---------|
| Get notified when VM CPU stays above 90% for 5 minutes | Create **metric alert** (Percentage CPU > 90, 5-min window) + action group |
| Silence alerts during a weekend maintenance window | Create **alert processing rule** with suppression schedule |
| Back up all VMs in a resource group daily and keep for 30 days | Create **Recovery Services Vault** + **backup policy** + enable protection |
| Recover a single file from a VM backup | Use **File Recovery** mount point from recovery point |
| Replicate VMs to a secondary region for DR | Enable **Azure Site Recovery** replication |
| Run a DR drill without affecting production | **Test failover** to isolated network in ASR |
| Analyse which VMs haven't sent a heartbeat in 1 hour | KQL: `Heartbeat \| summarize max(TimeGenerated) by Computer \| where ... < ago(1h)` |
| Send backup failure notifications to the ops team | Configure **Backup alerts** + action group with email |
| Monitor network traffic decisions from NSGs | Enable **NSG flow logs** to a storage account and Log Analytics |
| Continuously test that VMs can reach an internal API | Configure **Connection Monitor** in Network Watcher |

---

[← Domain 4 — Networking](./04-virtual-networking/){: .btn } [Quick Reference Cheatsheet →](./06-quick-reference-cheatsheet/){: .btn .btn-primary }
