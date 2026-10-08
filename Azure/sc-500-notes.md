# SC-500 Notes — Cloud and AI Security Engineer Associate

> **Exam:** SC-500 — *Implementing End-to-End Security Controls for Cloud and AI Workloads* (replaces AZ-500, which retired 31 Aug 2026).
> **Booked:** 24 Oct 2026 · **Pass score:** 700/1000
> Organized by the **Skills Measured** blueprint (as of July 2026). Lessons are condensed to the points most likely to be tested and filed under the matching objective. Each lesson ends with a **🎯 Exam focus** line; trigger keywords go into the cheat sheet at the bottom.

## Exam Domains (weightings)
1. **Manage identity, access, and governance** (20–25%)
2. **Secure storage, databases, and networking** (25–30%)
3. **Secure compute** — incl. Security for AI (20–25%)
4. **Manage and monitor security posture** (20–25%)

*Cross-reference:* a lot of the old AZ-500 material carries over — see `az-500-notes.md`. The new heavyweight areas are **Security for AI (3.1)**, **Entra Private Access**, **Virtual Network Manager**, and **Security Copilot (4.3)**.

---

# Domain 1 — Manage identity, access, and governance (20–25%)

## 1.1 Secure access to resources by using Microsoft Entra ID
*PIM · Conditional Access · authentication methods (MFA, passwordless) · app identities (enterprise apps, app registrations) · OAuth permission grants & consent · managed identities*

## 1.2 Secure secrets and keys by using Azure Key Vault
*Deploy & configure Key Vault · access model + Key Vault firewall · keys, secrets, certificates · secret scanning with Defender CSPM · Defender for Key Vault*

## 1.3 Implement governance to enforce security and regulatory compliance
*Azure Policy (built-in + custom) · regulatory compliance in Defender for Cloud · resource locks · Azure & Entra role assignments (built-in + custom) · remediate overprivileged RBAC · Azure Backup security features · security controls via IaC*

---

# Domain 2 — Secure storage, databases, and networking (25–30%)

## 2.1 Implement security for storage accounts
*Storage account hardening + Storage firewall · Defender for Storage · access to storage (keys, SAS, access policies, Entra RBAC)*

## 2.2 Implement security for databases
*Azure SQL platform-level security · auditing for SQL Database & SQL Managed Instance · Defender for Databases*

## 2.3 Implement security for Azure network services
*NSGs & ASGs · Azure Virtual Network Manager (security admin rules) · Virtual WAN + VPN security · Entra Private Access · private endpoints vs Private Link services · Azure Firewall · Network Watcher effective security rules*

---

# Domain 3 — Secure compute (20–25%)

## 3.1 Implement security for AI
*SharePoint data overexposure (Copilot oversharing) · Purview DSPM for AI · real-time protection for Copilot Studio agents · Conditional Access + access management for Entra Agent ID · blast-radius analysis in Defender XDR · AI Gateway in API Management for Microsoft Foundry · Defender for AI Services + Foundry guardrails · Data and AI security dashboard (Defender for Cloud) · agent management in M365 admin center*

### Conditional Access for Entra Agent ID — authentication flows & CA scope

**The CA assignment scope depends on who the token subject is.** Three agent access patterns:

| Access pattern | Token subject | CA assignment scope | MFA / device controls? |
|---|---|---|---|
| **On-behalf-of (OBO)** — user signs in, agent exchanges the user's token | The **signed-in user** | **Users or groups** | ✅ Yes — user is present |
| **Application-only (autonomous)** — OAuth 2.0 **client credentials** (cert, managed identity, secret) | The **agent identity** (service principal) | **Agent identities** | ❌ No — no user |
| **Agent's user account** — digital worker with its own mailbox/user account | The agent's **user-type identity** | **That user account** (standard user CA) | ❌ No — non-interactive |

- **OBO = most common.** The user's identity and permissions limit what the agent can do, so **normal user CA policies apply** (MFA, device compliance, sign-in risk, location).
- **Wrong scope = no enforcement.** A user-targeted policy misses autonomous agents; an agent-targeted policy misses OBO flows.
- CA is evaluated **when the token is issued**, before the agent or user gets the token.
- **Portal path:** CA policy → *Assignments* → **Users, agents or workload identities** → **Agents** → **All agent identities** / specific agents / an **agent identity blueprint**.

**Targeting agents at scale (no per-agent lists):**
- **Custom security attributes:** key-value pairs on agent identities, e.g. block where `DataSensitivity = Confidential`. **New agents with that attribute are covered automatically, with no policy edit.**
- **Agent identity blueprint:** target the parent blueprint and **every agent created from it** is covered.

**Limits of CA policies that target agent identities (autonomous agents):**
- **Only condition:** **Agent risk** (Preview)
- **Only grant control:** **Block access**
- **Cannot enforce:** MFA, device compliance, user session controls (e.g. app-enforced restrictions)
- These limits **don't apply to OBO**, which gets the full set of user CA controls.

> 🎯 **Exam focus:** Spot the flow first. *User signed in → target users/groups (MFA possible). Autonomous / client credentials → target agent identities, and the only option is **agent risk → block**.* For "new agents covered automatically", the answer is **custom security attributes** or a **blueprint**. "Require MFA for an autonomous agent" is always a wrong answer.

## 3.2 Implement security for servers and virtual machines
*Disk encryption · Azure Bastion · JIT VM access · Azure Arc for hybrid/multicloud · Defender for Servers (vuln scanning, EDR, agentless scanning) · Trusted Launch (secure boot, vTPM, integrity monitoring) · Azure Machine Configuration*

## 3.3 Implement security for application platform services
*Defender for Containers · AKS, ACR, Container Instances, Container Apps · Azure Functions & Logic Apps (auth + network access) · App Service + Azure WAF · API Management back-end protection*

---

# Domain 4 — Manage and monitor security posture (20–25%)

## 4.1 Manage security posture by using Defender for Cloud
*Defender CSPM + compliance frameworks · workload protection plans · AWS/GCP & hybrid connectors · Defender Vulnerability Management settings for VMs · Defender EASM*

## 4.2 Implement activity and event collection in Microsoft Sentinel
*Workspaces + Sentinel roles · content hub · Microsoft data connectors · Syslog & CEF · Windows Security events via DCRs (incl. WEF) · custom log tables · automation rules & playbooks · data retention · querying Purview Audit in Defender XDR*

## 4.3 Implement Microsoft Security Copilot
*Workspaces · permissions & roles · plugins, Microsoft agents, Security Store agents*

---

# Exam Tips & Gotchas

## Keyword → Service cheat sheet
> Spot the keyword in the question stem → pick the service. Grouped by domain; filled in as lessons are added.

### Identity, access & governance
| If the question says… | The answer is… |
|---|---|

### Storage, databases & networking
| If the question says… | The answer is… |
|---|---|

### Compute & AI
| If the question says… | The answer is… |
|---|---|
| Agent acts **on behalf of a signed-in user** / OBO / token exchange | CA targeting **users or groups** (full user controls) |
| **Autonomous** agent / **client credentials** / no user present | CA targeting **agent identities** |
| Agent is a **digital worker** with its own mailbox/user account | Standard **user-targeted CA** on that account |
| CA for agents, **only condition** available | **Agent risk** (Preview) |
| CA for agents, **only control** available | **Block access** |
| Cover **future/new agents automatically**, by business label (e.g. sensitivity) | **Custom security attributes** in the CA filter |
| Cover **all agents from one project/product template** | Target the **agent identity blueprint** |

### Posture & monitoring
| If the question says… | The answer is… |
|---|---|

## Look-alike traps
> Services that get confused with each other, and the one detail that separates them.

- **User-targeted CA vs agent-targeted CA:** decided by the **token subject**. OBO token = user → user policy. Client-credentials token = agent → agent policy.
- **Agent identity CA vs user CA:** agent identity CA has **no MFA, no device compliance, no session controls**. It only has **agent risk → block**.
- **Custom security attributes vs blueprint:** attributes = targeting by **business label across any agents**. Blueprint = targeting by **shared parent template**.
