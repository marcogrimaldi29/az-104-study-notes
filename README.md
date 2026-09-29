# 📘 AZ-104: Microsoft Azure Administrator

### Study Notes Repository

[![GitHub](https://img.shields.io/badge/GitHub%20Pages-Live-brightgreen?logo=github)](https://marcogrimaldi29.com/az-104-study-notes)
[![marcogrimaldi29.com](https://img.shields.io/badge/Blog-marcogrimaldi29.com-blue?logo=rss)](https://marcogrimaldi29.com)

> 🎯 **Goal:** Earn the Microsoft Certified: Azure Administrator Associate badge
> 📅 **Notes Version:** 2026
> 📋 **Based on:** [Official Microsoft AZ-104 Study Guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/az-104)
> - 🔗 **Companion repos:** [📘 AZ-305 Study Notes](https://marcogrimaldi29.com/az-305-study-notes/) · [🥽 AZ-305: Azure Compute Services — Deep Dive](https://marcogrimaldi29.com/az-305-compute/) · [🥽 AZ-305: Azure Messaging Services — Deep Dive](https://marcogrimaldi29.com/az-305-messaging/) · [🥽 AZ-305: Data & Analytics Services — Deep Dive](https://marcogrimaldi29.com/az-305-data-analytics/) · [🥽 AZ-305: Migration, HA & BCDR — Deep Dive](https://marcogrimaldi29.com/az-305-bcdr/)

---

## 📋 Exam At-a-Glance

| Detail | Info |
|--------|------|
| 🏅 Certification | Microsoft Certified: Azure Administrator Associate |
| 📝 Passing Score | **700 / 1000** |
| 💶 Exam Price | **~€126 EUR** *(varies by EU country & Pearson VUE location; VAT may apply)* |
| ⏱️ Duration | **~120 minutes** |
| ❓ Question Types | MCQ, multi-select, drag-and-drop, case studies |
| 🔁 Renewal | **Annual** via free online assessment on Microsoft Learn |
| 🛡️ Prerequisite | None *(AZ-900 recommended but not required)* |

---

## 📊 Official Domain Breakdown

> ⚠️ **Official ranges** from the Microsoft study guide (updated April 2025)

```
pie title Exam Domain Weights of the AZ-104 (official ranges)
    "Manage Azure Identities & Governance (20–25%)" : 25
    "Implement and Manage Storage (15–20%)" : 20
    "Deploy and Manage Azure Compute Resources (20–25%)" : 20
    "Implement and Manage Virtual Networking (15–20%)" : 20
    "Monitor and Maintain Azure Resources (10–15%)" : 15
```

| # | Domain | Official Weight | Key Services |
|---|---------|----------------|--------------|
| 1 | Manage Azure Identities & Governance | **20–25%** | Entra ID, RBAC, Azure Policy, Subscriptions |
| 2 | Implement and Manage Storage | **15–20%** | Storage Accounts, Blob, Azure Files, AzCopy |
| 3 | Deploy and Manage Azure Compute Resources | **20–25%** | VMs, VMSS, App Service, Containers, ARM |
| 4 | Implement and Manage Virtual Networking | **15–20%** | VNets, NSGs, Bastion, DNS, Load Balancer |
| 5 | Monitor and Maintain Azure Resources | **10–15%** | Azure Monitor, Backup, Site Recovery |

---

## 🗺️ Certification Path

```
flowchart LR
    AZ900["☁️ AZ-900\nAzure Fundamentals\n(Recommended)"]
    AZ104["🔧 AZ-104\nAzure Administrator\nAssociate\n(This Exam)"]
    AZ305["🏛️ AZ-305\nDesigning Azure\nInfrastructure Solutions\n(Next Step)"]

    AZ900 -->|Foundation| AZ104
    AZ104 -->|"Prerequisite\n(active cert required)"| AZ305
```

---

## 🗂️ Repository Structure

```
az-104-study-notes/
├── README.md                             ← 📍 You are here
├── 00-azure-prerequisites.md             ← Core Azure architecture fundamentals
├── 01-identity-governance.md             ← Domain 1 (20–25%)
├── 02-storage.md                         ← Domain 2 (15–20%)
├── 03-compute.md                         ← Domain 3 (20–25%)
├── 04-virtual-networking.md              ← Domain 4 (15–20%)
├── 05-monitor-maintain.md                ← Domain 5 (10–15%)
└── 06-quick-reference-cheatsheet.md      ← Last-minute review & exam traps
```

---

## 📚 Official Learning Resources

| Resource | Link |
|----------|------|
| 📚 Microsoft's AZ-104 Certification Learning Paths | [Certification Learning Paths](https://learn.microsoft.com/en-us/credentials/certifications/azure-administrator/) |
| 📄 Official Exam Page | [AZ-104 Exam](https://learn.microsoft.com/en-us/credentials/certifications/exams/az-104/) |
| 📋 Skills Measured / Study Guide | [Official Study Guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/az-104) |
| 🧪 Free Practice Assessment | [Practice Assessment](https://learn.microsoft.com/en-us/credentials/certifications/exams/az-104/practice/assessment?assessment-type=practice&assessmentId=21) |
| 🎬 Exam Readiness Videos | [Exam Readiness Zone](https://learn.microsoft.com/en-us/shows/exam-readiness-zone/?terms=az-104) |
| 🏗️ Azure Documentation | [Azure Docs](https://learn.microsoft.com/en-us/azure/?product=popular) |
| 💶 EU Exam Pricing | [Pearson VUE Microsoft](https://home.pearsonvue.com/microsoft) |

---

## ✅ Key Study Tips

- 🎯 The exam is **hands-on and scenario-based** — you need to know *how* to do things, not just *what* they are
- 🔧 Practice in the **Azure portal, PowerShell, and Azure CLI** — multi-tool questions are common
- 💰 Know **SKU tier feature gates** — what Premium has that Standard doesn't (e.g. P-tier App Service for slots)
- 📐 Understand **RBAC scope hierarchy** — Management Group → Subscription → Resource Group → Resource
- ⚡ Memorise **SLA uptime percentages** and know how Availability Zones improve them
- 📖 For case studies: read **business requirements and constraints first**, then eliminate answers
- 🔄 Know the difference between **IaaS, PaaS, and serverless** patterns and when to use each

---

## ⚡ Quick Navigation

| File | Topics Covered |
|------|----------------|
| [📘 00 — Azure Prerequisites](./00-azure-prerequisites.md) | Regions, AZs, ARM, VNets, storage basics, identity fundamentals, SLAs |
| [🔐 01 — Identity & Governance](./01-identity-governance.md) | Entra ID, RBAC, Azure Policy, Subscriptions, Cost Management |
| [🗄️ 02 — Storage](./02-storage.md) | Storage Accounts, Blob, Azure Files, SAS, encryption, AzCopy |
| [🖥️ 03 — Compute](./03-compute.md) | VMs, VMSS, ARM/Bicep, App Service, Container Instances, Container Apps |
| [🌐 04 — Virtual Networking](./04-virtual-networking.md) | VNets, NSGs, Bastion, DNS, Load Balancer, Private Endpoints |
| [📊 05 — Monitor & Maintain](./05-monitor-maintain.md) | Azure Monitor, Log Analytics, Alerts, Backup, Site Recovery |
| [⚡ 06 — Quick Reference Cheatsheet](./06-quick-reference-cheatsheet.md) | Key numbers, decision tables, CLI/PS commands, exam traps |

---

## 📚 About the Study Notes

These notes are hosted on **GitHub Pages** and published as a searchable website on this URL:

👉 **[📘 AZ-104 Study Notes](https://marcogrimaldi29.com/az-104-study-notes/)**

The site includes full-text search, Mermaid diagram rendering, and mobile-friendly navigation for on-the-go review. 

These notes are designed to be a structured, exam-focused summary of the most important concepts and services baseds on the official [Microsoft AZ-104 Study Guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/az-104){:target="_blank"} and its criteria.

Additional study notes maintained by me are also available for those pursuing Microsoft and Azure certifications at the following Landing Page:

👉 **[🛬 Landing Page: Study Notes](https://marcogrimaldi29.com/study-notes/)**

---

## ✍️ About the Author

Maintained by **[Marco Grimaldi](https://www.linkedin.com/in/marco-grimaldi29/)** — Cloud Consultant, Language Trainer & Lifelong Learner.

Find more certification guides, study tips, and tech content at **[🏠 marcogrimaldi29.com](https://marcogrimaldi29.com)**

The site is continuously updated and based on my personal study notes and experiences. If you have any feedback, suggestions, or corrections, feel free to [reach out](https://marcogrimaldi29.com/contact/)!

---

## 📈 Analytics

This site uses [Umami](https://umami.is/) for privacy-friendly analytics.

---

## 🐞 Spotted a Mistake?

Azure changes fast, so if something here is wrong or out of date, please [report an issue](https://github.com/marcogrimaldi29/az-104-study-notes/issues).

---

## 💚 Support

If these notes helped you prepare for AZ-104, there are a few ways to show it: ⭐ star this repo on GitHub so other candidates can find it, 🤝 connect with me on LinkedIn, or ☕ support the work behind them with a coffee.

<p>
  <a href="https://github.com/marcogrimaldi29/az-104-study-notes"><img src="https://img.shields.io/badge/Star_on_GitHub-181717?style=for-the-badge&logo=github&logoColor=white" alt="Star on GitHub" /></a>
  <a href="https://www.linkedin.com/in/marco-grimaldi29/"><img src="https://img.shields.io/badge/Connect_on_LinkedIn-0A66C2?style=for-the-badge&logo=data:image/svg%2Bxml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNCAyNCI%2BPHBhdGggZmlsbD0id2hpdGUiIGQ9Ik0yMC40NDcgMjAuNDUyaC0zLjU1NHYtNS41NjljMC0xLjMyOC0uMDI3LTMuMDM3LTEuODUyLTMuMDM3LTEuODUzIDAtMi4xMzYgMS40NDUtMi4xMzYgMi45Mzl2NS42NjdIOS4zNTFWOWgzLjQxNHYxLjU2MWguMDQ2Yy40NzctLjkgMS42MzctMS44NSAzLjM3LTEuODUgMy42MDEgMCA0LjI2NyAyLjM3IDQuMjY3IDUuNDU1djYuMjg2ek01LjMzNyA3LjQzM2EyLjA2MiAyLjA2MiAwIDAgMS0yLjA2My0yLjA2NSAyLjA2NCAyLjA2NCAwIDEgMSAyLjA2MyAyLjA2NXptMS43ODIgMTMuMDE5SDMuNTU1VjloMy41NjR2MTEuNDUyek0yMi4yMjUgMEgxLjc3MUMuNzkyIDAgMCAuNzc0IDAgMS43Mjl2MjAuNTQyQzAgMjMuMjI3Ljc5MiAyNCAxLjc3MSAyNGgyMC40NTFDMjMuMiAyNCAyNCAyMy4yMjcgMjQgMjIuMjcxVjEuNzI5QzI0IC43NzQgMjMuMiAwIDIyLjIyNSAweiIvPjwvc3ZnPg%3D%3D" alt="Connect on LinkedIn" /></a>
  <a href="https://buymeacoffee.com/marcogrimaldi29"><img src="https://img.shields.io/badge/Support_with_a_coffee-FFDD00?style=for-the-badge&logo=buymeacoffee&logoColor=black" alt="Support with a coffee" /></a>
</p>

---

## ©️ Credits & Acknowledgements

The **[Just the Docs](https://github.com/just-the-docs/just-the-docs)** theme is used for a clean, documentation-style layout. Licensed under [MIT](https://opensource.org/license/MIT).

Created with the help of AI. Model used: **[Claude Sonnet 4.6](https://www.anthropic.com/news/claude-sonnet-4-6)**. The content has been reviewed and edited by the author for accuracy and clarity, but may contain errors. Always verify against the latest [Microsoft documentation](https://learn.microsoft.com/en-us/azure/).

> *Not affiliated with or endorsed by Microsoft.*

---