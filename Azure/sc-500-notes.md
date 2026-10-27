# SC-500 Notes — Cloud and AI Security Engineer Associate

> **Exam:** SC-500 — *Implementing End-to-End Security Controls for Cloud and AI Workloads* (replaces AZ-500, which retired 31 Aug 2026).
> **Booked:** 2 Nov 2026 · **Pass score:** 700/1000
> Organized by the **Skills Measured** blueprint (as of July 2026). Lessons are condensed to the points most likely to be tested and filed under the matching objective. Each lesson ends with a **🎯 Exam focus** line; trigger keywords go into the cheat sheet at the bottom.

> ⚡ **Short on time? Read `sc-500-cram-sheet.md` first.** It has keyword → service pairs for every AI objective plus a recap of the other domains.

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

### Configure Conditional Access policies for agents

**5-step lifecycle:** Define scope → Set conditions → Choose access control → **Validate in report-only** → Enforce (switch to **On**).

**Licensing (likely question):**
| Need | License |
|---|---|
| **Conditional Access** for agents | **Entra ID P1** or **Microsoft 365 E3** (this unit). Docs: **Entra ID P1/P2 + Agent 365 license per user**. **Network controls** for agents need **Entra Internet Access** |
| **Entra Agent ID** features (part of **Microsoft Agent 365**) | **Microsoft Agent 365** or **Microsoft 365 E5** |

**OBO agent policy (token = user):**
- *Protection > Conditional Access > Policies > New policy* → Assignments: **Users, agents or workload identities** → *What does this apply to?* **Users and service principals** → include the **users/groups** who use the agent.
- Target resources: what the agent reaches on the user's behalf (Microsoft Graph, SharePoint Online, or All resources).
- Grant: **Require MFA**, **Require compliant device**. The full user control set is available.
- **Baseline OBO policies:** block **legacy authentication**, require MFA for **elevated sign-in risk**, require MFA for **guests** accessing agent-connected resources.

**Autonomous agent policy (token = agent):**
- Assignments → Include: **All agent identities** OR **Select agent identities** (by name or **object ID**).
- **Design choice:**
  - **Block all agents + allow approved ones** = **secure-by-default / allow list**, but more admin overhead.
  - **Block specific agents** = flexible, but you have to keep watching for new unauthorized agents.
- **Conditions in this unit:**
  - **Named locations:** the most common. Restrict to corporate IPs or Azure regions.
  - **Sign-in risk:** comes from **Entra ID Protection**.
  - **Application filters / target resources:** e.g. block Graph but allow Power Platform APIs. **Limits blast radius.**
- **Grant options:**
  - **Block:** use to **decommission** an agent or enforce an allow list.
  - **Grant:** adds nothing, so it's rare.
  - **Grant with controls:** the relevant one is **Require compliant network**.
- **Session controls:** mostly not applicable. The exception is **Sign-in frequency**, which forces the agent to reauthenticate and **limits stolen-token window**.

> ⚠️ **Conflict between lessons — verified against Microsoft docs:** the official *Target agent identities in Conditional Access* page says that when a policy targets **agent identities**, the **only condition is Agent risk (Preview)** and the **only control is Block access** ("no interactive remediation possible"). The named locations, sign-in risk, compliant network and sign-in frequency options in this unit **don't match the docs for agent identities**. Richer conditions plus device compliance and compliant network **do** exist for **agents' user accounts** (Preview), which may be what this unit is mixing in. **On the exam: agent identity → agent risk + block.**

**Report-only mode (high-yield):**
- Evaluates and **logs** what would happen **without blocking**.
- Check results in **Entra ID > Monitoring > Sign-in logs**, filtered by Conditional Access.
- Leave it running for **24–48 hours** to capture a full cycle of agent activity.
- Then switch **Enable policy → On**.
- Why: blocking the wrong service principal **breaks production with no warning**.

**Contoso scenario (decommission a legacy Copilot Studio agent):** scope = that specific agent's service principal (**object ID**) · conditions = **none** · control = **Block access** · state = **report-only** for 2 days, then **On**.

> 🎯 **Exam focus:** *Safely test or validate a CA policy before enforcing it* → **Report-only + Sign-in logs**. *Decommission / kill one agent* → **CA targeting that agent's service principal (object ID) + Block access**. *Shorten stolen-token exposure* → **Sign-in frequency**. Know the licensing split: **P1/E3** for CA, **Agent 365/E5** for Agent ID.

### Control agent access and lifecycle

**Three defense layers (defense in depth):**
| Layer | What it does | Scope | Use when |
|---|---|---|---|
| **1. CA block policy** | Blocks **token issuance** | **Existing + new** agents | Main enforcement; secure-by-default tenant-wide |
| **2. Block agent creation (per product)** | Stops agents being **created at the source** | Per product: Copilot Studio, Foundry, Security Copilot | Reduce **sprawl**, force approval workflows |
| **3. Disable individual agent** | Deactivates **one** agent identity | Entra admin center or the product's admin UI | **Targeted decommission / incident response** |

- **Key gotcha:** CA **does not stop agents being created**. It only blocks authentication. Unauthorized agents can still appear and **use up directory quota**. Combine it with product-level creation blocks.

**Microsoft Agent 365 (GA): the central control plane.** Start here before per-product restrictions.
- Location: **M365 admin center → Agents → Settings**
- **Allowed agent types:** which categories of agents are permitted
- **Security templates:** presets that bundle **CA policies + custom security attributes**, applied to **every new agent**
- **User access controls:** which users or groups can interact with agents
- **Agent management rules:** bulk governance actions, e.g. **reassign ownership when the creator leaves**

**Blocking creation per product:**
| Product | How to block / restrict agent creation |
|---|---|
| **Copilot Studio** | **Licensing + RBAC**: block **free-trial sign-ups**, remove the **Power Platform Administrator** role. **DLP policies** = users can **build but not publish** (allows experimenting, protects production) |
| **Azure AI Foundry** | Role hierarchy: **Billing Admin / Account Admin** creates subscriptions → **Azure AI Account Owner** creates Foundry projects → **Azure AI User** creates agents. Don't hand these roles out widely |
| **Security Copilot** | Remove **Owner/Contributor** from Security Copilot workspaces. Microsoft-owned agents (e.g. CA Optimization Agent) need **Security Admin** or **Identity Governance Admin** to enable. **Nuclear option:** delete all **SCU** capacity, which disables Security Copilot entirely |

**Disabling an individual agent:**
- Needs at least the **Agent ID Administrator** role.
- Path: **Entra ID → Agents → Agent identities** → select the agent → **Disable**. This blocks token issuance **without needing a CA policy**.
- To disable **all agents from one blueprint**: **Entra ID → Agents → Agent blueprints** → disable the parent.
- The product admin UI gives the same result **with extra context**. For example, the Power Platform admin center shows which **dependent workflows/connectors** are affected.
- **Incident response flow:** disable → investigate → **rotate credentials** → re-enable.

**Monitoring the agent lifecycle in audit logs:**
- Path: **Entra ID → Monitoring → Audit logs**, filtered by **Category = ApplicationManagement**.
- Agents appear as **service principals**. To confirm a service principal is an agent, check **`agentType`** in `targetResources`. **Any value other than `notAgentic` = agent.** You can also check by querying **Graph** with the object ID.

| Audit activity | Meaning |
|---|---|
| **Add service principal** | Agent identity **created** |
| **Update service principal** | Agent config or permissions **changed** |
| **Add app role assignment to service principal** | Agent **granted access** to a resource |
| **Delete service principal** | Agent identity **removed** |
| **Create user** (*not* a service principal event) | **Agent user account** created |

- **Alerting pipeline:** Entra ID → Audit logs → `agentType` filter → **Sentinel playbook / Logic App** → alert or email.

**Access reviews for agents:**
- Path: **Entra ID → Identity Governance → Access reviews** → Review scope: **Workload identities** → select the agent service principals.
- Frequency: **monthly/quarterly**. Reviewers are usually the business owners.
- Reviewers **approve** (keep) or **deny** (deactivate). Denied agents are **auto-disabled** or flagged, depending on configuration.

> 🎯 **Exam focus:**
> - *Stop new agents being created* → **product-level blocks** (or Agent 365 allowed types). CA alone doesn't do this.
> - *Compromised agent, stop it now* → **Disable the agent identity** (Agent ID Administrator).
> - *Disable a whole family of agents* → **disable the blueprint**.
> - *Detect new agents* → audit log **Add service principal** + **`agentType` ≠ `notAgentic`** → Sentinel/Logic App.
> - *Periodically confirm agents are still needed* → **Access reviews, scope = Workload identities**.
> - *Let users experiment but not publish* → **Copilot Studio DLP**.

### 📌 Module recap — Secure access for Entra Agent ID
- **The constraint behind every decision:** autonomous agents authenticate **non-interactively** (OAuth 2.0 **client credentials**), so **no MFA and no device compliance**. Enforcement therefore goes through **Conditional Access policies that target agent/workload identities**.
- **Validate before enforcing:** **report-only** first, scoped to the specific agent service principals.
- **Blocking an inactive or over-privileged agent with a CA policy is effective, reversible, and keeps the identity record for audit.** That's why the exam prefers **block/disable over delete**.
- **The governance loop:**
  1. **Creation controlled:** Agent 365 and per-product blocks.
  2. **Activity visible:** audit log **Add service principal** events.
  3. **Continued access re-validated:** **access reviews** by business owners.
- **Next module:** assessing agent **blast radius and attack paths** with **Defender XDR**.

> 🎯 **Exam focus:** *Remove an unused or over-privileged agent's access but keep evidence for audit* → **CA block** (or **disable**), **not delete**.

---

### Module: Assess agent blast radius & attack paths with Defender XDR
**Problem:** dozens of agents created by IT, by business units (Copilot Studio) and by external integrations. Nobody knows them all, or **what an attacker could reach if one were compromised**.

**Solution:** the **Defender XDR asset inventory** now includes AI agents. The **AI agent inventory** shows:
- **Agent identities**
- Their **product sources** (Copilot Studio, Foundry, etc.)
- **Risk signals** showing how much damage a compromised agent could do

**Three skills this module covers:**
1. **Discover** agents: **AI agent inventory** in Defender XDR
2. **Blast radius**: examine **permissions**, **knowledge sources** and **blueprint configuration**
3. **Attack paths**: routes to unauthorized access if the agent identity is compromised

> 🎯 **Exam focus:** *"What could an attacker reach if this agent were compromised?"* / *"Which agents should we fix first?"* → **Defender XDR** (AI agent inventory → blast radius + attack paths). **Not Entra** (Entra = identity controls, Defender XDR = risk assessment).

### Discover AI agents in the Defender portal (AI agent inventory)

**Prerequisites (high-yield):**
- The inventory is in **Preview**. You must **opt in to preview features in BOTH Defender XDR and Defender for Cloud**.
- **Onboard to Microsoft Agent 365** and enable **security for AI agents** in Defender.
- **Licensing (from 1 July 2026):** agent-level discovery, posture and threat detection for **Copilot Studio and Foundry** agents require a **Microsoft Agent 365-eligible license**. Defender for Cloud Apps or Defender for Cloud licenses **alone no longer cover it**.

**Enabling discovery per platform:**
| Platform | Where to enable |
|---|---|
| **Copilot Studio** | Defender portal: **System > Settings > Security for AI** → turn on **Security for AI Agents**. **Plus** the **Power Platform admin** completes it in the **Power Platform admin center → Security > Threat Protection → Microsoft Defender – Copilot Studio AI Agents** |
| **Microsoft Foundry, AWS Bedrock, GCP Vertex AI** | Enable the **Microsoft Foundry integration** in **Defender for Cloud** |

- **Where to view:** Defender portal → **Assets > AI Agents**, with tabs per platform.
- **Coverage:** Copilot Studio, Foundry, Microsoft 365, supported non-Microsoft platforms (Bedrock, Vertex AI), and **local agents discovered on endpoint devices**.

**Inventory properties:**
| Property | Meaning |
|---|---|
| Agent name | Display name |
| Product source | Copilot Studio / Foundry / AWS Bedrock / GCP Vertex AI |
| Status | **Active** (processing requests) · **Inactive** (still has permissions, not being invoked) · **Disabled** (turned off but **keeps identity and permissions** until removed) |
| **Associated identities** | The **service principal or managed identity** the agent uses. **This is what you assess for risk** |
| Last activity | Most recent invocation. Separates production agents from abandoned ones |

- **Gotcha:** a **dormant agent with broad permissions is as exploitable as an active one**. "Inactive" does not mean "safe".

**Filtering and search:**
- **Filter** by product source, status and **risk level**. The risk score is based on **permissions, knowledge sources and configuration**.
- **Search** by agent name or **associated identity**. If an alert names a service principal, **search by its object ID** to find the agent.

**Discovery limitations (likely exam trap):**
- An agent only appears if it uses a **Microsoft Entra-backed identity**, its platform integration is enabled, **and** its identity has activity in monitored resources.
- **API key or external IdP authentication = NOT discovered.** You need **manual inventory and monitoring**.
- Custom agents in Azure (e.g. **ACI with managed identity**) **do appear**, because a managed identity is an Entra identity.

> 🎯 **Exam focus:**
> - *Agents not showing in the inventory?* → check **preview opt-in in both XDR and Defender for Cloud**, Agent 365 onboarding, and per-platform integration, or the agent uses **API keys / an external IdP**.
> - *Discover Bedrock/Vertex agents* → **Foundry integration in Defender for Cloud**.
> - *Copilot Studio discovery* needs **both** the Defender setting **and** the Power Platform admin step.
> - *Find which agent an alert's service principal belongs to* → **search the inventory by object ID**.

### Assess blast radius and attack paths

**Blast radius** = everything a compromised agent identity can reach. AI agents tend to have **big** blast radiuses because they combine **broad data access** (for grounding) with **action capabilities**.

**The three blast radius components (Permissions + Knowledge sources + Blueprint actions):**
| Component | What it measures | 🚩 Risk indicator |
|---|---|---|
| **Permissions** | API scopes on the agent's service principal | Tenant-wide scopes: **Mail.ReadWrite.All**, **Files.ReadWrite.All**, **Sites.FullControl.All** |
| **Knowledge sources** | SharePoint sites, databases and files used for **grounding** | Sites with **sensitivity labels** (Confidential/Highly Confidential), broad org content |
| **Blueprint configuration** | **Actions** the agent can take: send email, create records, execute code | Actions that can **move or modify data externally** (exfiltration) |

> ⭐ **FLAGGED: likely exam question. Know which component each scenario describes:**
> | Scenario in the question | Component |
> |---|---|
> | Graph permissions to read **all mail, files, calendars** across the tenant (vs **delegated** access to **one SharePoint folder**) | **Permissions**: what APIs and resources the **service principal** can access |
> | Agent connected to a SharePoint site of **employee performance reviews** → access to sensitive HR data | **Knowledge sources**: content repositories used for **grounding** (the data it uses to **generate responses**) |
> | Agent can **send emails, create records in external systems, execute code** → can **exfiltrate or modify** data | **Blueprint configuration**: the **actions** it can take, which push blast radius **beyond read-only** |
>
> **Memory hook: "Can ACCESS / Can KNOW / Can DO"** = Permissions / Knowledge sources / Blueprint actions.
> - **Read-only** risk comes from permissions and knowledge sources.
> - **Actions** (blueprint) are what turn read access into **exfiltration or modification**.

**Narrow vs broad permissions:**
- **Broad (large blast radius):** `Mail.ReadWrite.All`, `Files.ReadWrite.All`, `Sites.FullControl.All`, `User.Read.All`
- **Scoped (small blast radius):** `Mail.Send`, `Files.Read.Selected`, `Sites.Selected`, `User.ReadBasic.All`
- **Application permissions** (tenant-level, no user context) are **larger** than **delegated** permissions (on behalf of a user).
- **Multiple broad permissions together** = can reach and exfiltrate almost anything in the tenant.

**Knowledge sources: sensitivity beats quantity.** 5 "Highly Confidential" sites are riskier than 50 public ones. An agent grounded on a **restricted** site effectively **bypasses that site's access controls**.

**Where to assess it in Defender XDR (high-yield split):**
| Agent platform | How to assess blast radius |
|---|---|
| **Foundry / AWS Bedrock / GCP Vertex AI** | **Assets > AI Agents** → select the agent → **detail pane**: posture insights, risk factors, recommendations (permissions, knowledge sources, capabilities) |
| **Copilot Studio** | The inventory does **discovery only, no posture in the UI**. Use **Advanced Hunting**: *Investigation & response > Hunting > Advanced hunting* → **Queries** tab → **AI Agents** (prebuilt queries) |

- The **risk score** is based on **permission breadth + knowledge source sensitivity**. Sort by it and deep-dive the top agents first.

**Attack paths:**
- **Blast radius = static** scope of access. **Attack path = dynamic**, step-by-step exploitation through **chained permissions**.
- Open it from the agent detail pane with **"View on map"**.
- A path runs: agent identity (**entry point**) → pivots through connected resources → **target** (sensitive data, privileged accounts, critical systems).
- **Prioritize by criticality, not complexity.** A 3-step path to customer financial records beats a 7-step path to meeting notes.
- High-priority targets: **sensitive data stores**, **privileged operations** (creating admins, changing security policy), and **exfiltration points** (email, file upload).
- **Contoso HR agent example:** compromised identity → queries SharePoint employee records (**knowledge source**) → reads profiles via `User.Read.All` (**permission**) → sends data out via the **send-email action** (**blueprint**) → personal data breach.

**The four remediation categories:**
| # | Action | Contoso HR example |
|---|---|---|
| 1 | **Remove excess permissions**, swap broad scopes for scoped ones | `User.Read.All` → **`User.ReadBasic.All`**; `Sites.FullControl.All` → **`Sites.Selected`** |
| 2 | **Restrict knowledge sources** + apply **sensitivity labels** | Only the HR records site |
| 3 | **Modify blueprint actions** | Remove **send-email**, replace with a **create-task** action for **human review** |
| 4 | **Apply Conditional Access** (runtime control) | Block agent sign-in from outside the corporate network |

- **Always coordinate** with agent owners and business stakeholders, and **document the business justification** for each permission. Otherwise you risk breaking production.

> ⚠️ **Note on step 4:** this unit says CA for agents can use "network location, device compliance, or risk level". Per the Microsoft docs verified earlier, **CA policies that target agent identities only support Agent risk + Block**. Device compliance doesn't apply to autonomous agents. For "which CA control fits an autonomous agent" questions, stick with **agent risk → block**.

> 🎯 **Exam focus:**
> - *Assess blast radius for a Foundry/Bedrock/Vertex agent* → **inventory detail pane**. *For a Copilot Studio agent* → **Advanced Hunting → Queries → AI Agents**.
> - *Visualize how an attacker moves from the agent to data* → **attack path → View on map**.
> - *Reduce blast radius* → **least-privilege scopes** (`Sites.Selected`, `User.ReadBasic.All`), trim knowledge sources, remove exfiltration actions (**human-in-the-loop**).
> - *Prioritize* → **criticality of the target**, not path length.

---

### Module: Real-time protection for Copilot Studio agents (Defender for Cloud Apps)
**Problem:** Copilot Studio agents touch sensitive financial data through **Power Platform connectors**, and some are **external-facing**. If an agent misbehaves because of **prompt injection**, a **misconfigured connector** or a **compromised user session**, would we know, and do records exist to investigate?

**Product: Microsoft Defender for Cloud Apps (MDCA).** Three layers of agent protection:
| Layer | What it does |
|---|---|
| **1. Discovery & hunting** | Find agents across the environment |
| **2. Audit & alerts** | Capture agent activity and surface **policy violations** |
| **3. Real-time protection** | **Inspect and intercept** agent interactions **as they happen** (runtime) |

- **Setup spans two admin surfaces:** the **Microsoft Defender portal** **and** the **Power Platform admin center**.
- **Where the results show up:** the Defender XDR **AI agent inventory**, **alerts**, and **Advanced Hunting**.

**Prerequisites:**
- **License (either):** **Microsoft Agent 365**, **OR** **Defender for Cloud Apps + Copilot Studio** licenses together
- **Preview features enabled in both Defender for Cloud and Defender XDR**

> ⚠️ **Licensing note:** the earlier discovery unit said that from **1 July 2026**, agent-level discovery, posture and threat detection need an **Agent 365-eligible license**, and MDCA alone isn't enough. This module lists **MDCA + Copilot Studio** as an alternative. If asked for the **simplest or single license**, answer **Agent 365**.

> 🎯 **Exam focus:**
> - *Detect or stop **prompt injection** / risky agent behaviour **at runtime** in Copilot Studio* → **real-time protection in Defender for Cloud Apps**.
> - Setup needs **both** the **Defender portal + Power Platform admin center**.
> - Verify the results in **XDR inventory, alerts and Advanced Hunting**.

### Explore Copilot Studio AI agent protection (the three pillars in detail)
*Preview. Requires preview features enabled in **Defender for Cloud + Defender XDR**.*

| Pillar | Purpose | Output |
|---|---|---|
| **Discovery & hunting** | Auto-identifies all Copilot Studio agents and their risk exposure | **AI agent inventory** with connectors, data sources, risk signals |
| **Audit & alerts** | Monitors agent activity and detects policy violations | Alerts **correlated into Defender XDR incidents** |
| **Real-time runtime protection** | **Inspects and blocks** suspicious interactions **before they complete** | Immediate blocking plus incident data |

**1. Discovery & hunting**
- **No manual registration.** Turn on **Security for AI** in the Defender portal and agents are discovered automatically.
- For each agent you see its **connectors** and **data sources**: SharePoint libraries, **Dataverse tables**, external APIs.
- **Advanced Hunting (KQL)** for patterns, e.g. repeated sensitive-file access, unusual tool calls after hours.

**2. Audit & alerts**
- Source: **Copilot Studio audit logs**, analyzed by MDCA.
- Detects **prompt injection** and **data exfiltration**, e.g. large volumes of sensitive content returned to external users.
- Alerts are **correlated into Defender XDR incidents** alongside identity, endpoint and cloud signals.

**3. Real-time runtime protection (most advanced)**
- MDCA acts as a **proxy** and inspects **tool invocations before the agent executes them**, using threat intelligence and behavioral analytics.
- If an invocation is suspicious, it's **blocked before it executes**, the **user is notified**, and an **alert** goes to the Defender portal.
- It targets **high-confidence** threats:
  - Extracting **system instructions** or internal tool details
  - Leaking **sensitive data** through responses
  - **External users misusing internal-only tools**
  - Routing output to **untrusted external destinations**
  - **Obfuscated or hidden content** used to manipulate the agent (indirect injection)
  - **Credential leakage** through email or external APIs
- Needs **explicit configuration** plus coordination with the **Power Platform admin**. Setup registers the agent and establishes the proxy connection.

**Agent identity vs proxy App ID (likely trap):**
- Every Copilot Studio agent gets an **Entra identity automatically**. You don't create it.
  - Agents created **after 18 March 2026** (if the tenant opted in) get an **Entra Agent ID**: a **service principal with the "Agent" subtype**.
  - **Older agents** use a **legacy app registration**.
- Agent ID unlocks **CA scoped to individual agents**, **central audit logging in Entra**, and **lifecycle management**. Legacy app registrations don't get these.
- To find an agent's Agent ID: **Copilot Studio → Settings > Advanced > Metadata**.
- The **proxy App ID** used in real-time protection setup is exchanged between the Defender portal and the Power Platform admin center. It identifies the **trusted inspection connection**, **NOT the agent's own identity**.

**MDCA AI agent protection vs Defender for Cloud "AI services" plan (HIGH-YIELD):**
| | **MDCA: AI agent protection** | **Defender for Cloud: Defender for AI Services (CWP plan)** |
|---|---|---|
| Protects | **Copilot Studio** agent **sessions & tool invocations** | **Azure AI services workloads**: **Azure OpenAI**, **AI Model Inference** |
| Layer | Session / interaction layer | **Azure subscription** layer |
| Detects | Prompt injection, exfiltration, tool misuse (and **blocks** in real time) | Scans text tokens: **jailbreak**, **data leakage**, **credential theft**, **data poisoning** |
| Enable at | **Defender portal > System > Settings > Security for AI** | **Azure portal > Defender for Cloud > Environment settings > AI services → On** (per subscription) |
- Both send alerts into the **same Defender XDR incident queue**.
- **Enabling one does NOT enable the other.** If you use both Copilot Studio and Azure AI, turn on both.

**Contoso example:** discovery found a customer-service agent with **unneeded access to a sensitive SharePoint deal library**. Audit alerts flagged each time it accessed the library. **Real-time protection now blocks** retrieval from it, so the agent keeps working without exposing the data.

> 🎯 **Exam focus:**
> - **Block** a malicious Copilot Studio **tool invocation before it runs** → **MDCA real-time protection** (proxy).
> - **Jailbreak detection on Azure OpenAI** → **Defender for AI Services plan in Defender for Cloud** (*not* MDCA).
> - The **proxy App ID ≠ the agent's Entra identity**.
> - Agents after **18 Mar 2026** = **Entra Agent ID** (service principal, Agent subtype). Before that = legacy app registration.
> - Find the Agent ID: **Settings > Advanced > Metadata**.

### Enable real-time protection in Microsoft Defender (setup steps)

**Step 0: check the Microsoft 365 app connector is "Connected"** (likely gotcha)
- It's needed for **audit logs and alerts** to flow into the Defender portal.
- **If it's not connected:** real-time protection **still blocks**, but **no alerts or incidents** appear in the XDR queue.
- Enable it via app connectors → **Connect Microsoft 365**. Requires **Global Admin or Security Admin**.

**Step 1: turn on Security for AI Agents**
- Defender portal → **System > Settings > Security for AI** (may show as "Security for AI agents"; both are the same page).
- The toggle accepts the disclaimer and starts **continuous auto-discovery** of Copilot Studio custom agents.
- The inventory starts populating **within ~30 minutes**.

**Step 2: the two-role handoff for real-time protection**
| # | Who | Does what | Where |
|---|---|---|---|
| 1 | **Security Engineer** | Enables real-time protection and **shares the Defender URL** | Defender portal → System > Settings > Security for AI → *Real time protection during agent runtime* |
| 2 | **Power Platform Admin** | Turns on **Enable Microsoft Defender – Copilot Studio AI Agents**, enters the URL, and **creates or selects an Entra ID app** | **Power Platform admin center → Security > Threat Protection → Microsoft Defender – Copilot Studio AI Agents** |
| 3 | **Power Platform Admin** | **Sends the App ID** back to the security engineer | Configuration output |
| 4 | **Security Engineer** | **Enters the App ID** and saves | Defender portal → Security for AI |

- Flow: **URL goes Defender → Power Platform. App ID goes Power Platform → Defender.** The App ID must **match on both sides**.
- If you change the App ID, it takes **up to ~1 minute** to propagate. A validation error means wait and retry.

**Step 3: verify the status**
- **Green "Connected"** confirms three things: the Entra app config matches on both sides, the proxy endpoint is reachable and authenticated, and agent traffic is being inspected.
- **"Disconnected" or "Pending"** means the **Power Platform side is incomplete** or the **App ID doesn't match**. Recheck in the Power Platform admin center.

**Behaviour after it's enabled:**
- Every **tool invocation** (SharePoint retrieval, Dataverse query) is evaluated. Suspicious ones are **blocked before they execute**.
- The **user gets a "blocked for security reasons" message**, and an **alert** appears in **Incidents & alerts**.
- The **proxy adds a little latency**. Tell agent owners and monitor performance.
- **Applies only to NEW sessions.** Users in active conversations need to **restart the session**.

> 🎯 **Exam focus:**
> - *Real-time blocks are happening but **no alerts or incidents** in XDR* → the **Microsoft 365 app connector isn't connected**.
> - *Status shows Pending/Disconnected* → **App ID mismatch** or the Power Platform step isn't done.
> - *Protection enabled but a user's chat isn't protected* → it's an **existing session**, so **restart** it.
> - **Who does what:** Security Engineer = **Defender portal (URL out, App ID in)**. Power Platform Admin = **PPAC Threat Protection (creates the Entra app, returns the App ID)**.

### Review AI agent protection outputs (verify it end to end)

**Three places to verify:**
| # | Where | What confirms it's working |
|---|---|---|
| 1 | **Assets > AI assets** (or "AI Agents") | Agent listed with **Connected** status, plus its connectors and risk signals |
| 2 | **Incidents & alerts > Incidents** | AI agent alerts **correlated into XDR incidents** |
| 3 | **Hunting > Advanced Hunting** | `CloudAppEvents` returns Copilot Studio activity |

**1. Inventory:** for each agent you see the **name and environment**, **connection status**, **connectors** (SharePoint, Dataverse, APIs) and **risk signals** (sensitive data, external exposure, violations).
- Agent shows **Disconnected** → recheck the Power Platform admin steps and that the **App ID matches**.
- Check the connectors make sense. An **external customer agent shouldn't reach internal confidential SharePoint**.

**Protection status in Copilot Studio (maker view, no Defender access needed):** *Agents* page → **Protection status** column
| Status | Meaning |
|---|---|
| **Protected** (green shield) | Threat detection active, no violations |
| **Needs review** | **Policy violated or authentication inadequate** |
| **Unknown** | Status can't be determined |

**2. Incidents:** each one includes the alert title and severity, the **agent**, the **user or entity**, the **blocked action**, and related identity, endpoint and cloud signals.
- **No alerts right after enabling is normal.** Alerts only appear once activity triggers a detection, so wait or **generate a test interaction**.

| AI agent alert category | Example |
|---|---|
| **Prompt injection detected** | External user tries to **override the agent's instructions** |
| **Data exfiltration risk** | Agent pulls **large volumes** (e.g. 100+ files) from a confidential library |
| **Policy violation** | Agent tries a **blocked connector** or accesses restricted data |

**3. Advanced Hunting tables (HIGH-YIELD: know which table answers which question):**
| Table | Contains | Use it to answer |
|---|---|---|
| **`CloudAppEvents`** | **Runtime activity**: Copilot Studio actions, tool invocations, data access (Agent 365 observability data, routed through the **M365 app connector**) | **What did the agent DO?** |
| **`AIAgentsInfo`** | **Configuration and posture**: name, platform, creator, tools, knowledge sources, auth settings, **IsBlocked** | **How is the agent CONFIGURED / is it a posture risk?** |

```kusto
CloudAppEvents
| where Application == "Microsoft Copilot Studio"
| where Timestamp > ago(1h)
| project Timestamp, AccountDisplayName, ActivityType, ActivityObjects

AIAgentsInfo
| summarize arg_max(Timestamp, *) by AIAgentId   // latest record per agent
| project AIAgentName, Platform, CreatorAccountUpn, AgentToolsDetails, IsBlocked
```
- **`CloudAppEvents` returns nothing?** Check that **≥30 minutes** have passed since enabling, that the agent has actually had user interactions, and that the **M365 app connector is connected**.
- **Proactive hunting ideas:** sensitive-data access **after hours**, repeated **failed auth** to agent-connected resources, **unusual tool sequences** (reconnaissance).

**Contoso example:** the inventory showed the agent as **Connected** with the SharePoint risk flagged. A test interaction produced a **low-severity "AI agent policy violation"** incident. Advanced Hunting showed the successful Dataverse lookups **and** the blocked SharePoint attempt.

> 🎯 **Exam focus:**
> - *Query **what an agent did*** → **`CloudAppEvents`**. *Query an agent's **configuration / tools / whether it's blocked*** → **`AIAgentsInfo`**.
> - *Agent maker wants to check protection without Defender access* → the **Protection status column in Copilot Studio**.
> - "**Needs review**" = **policy violation or inadequate authentication**.
> - *No hunting data* → wait 30 min, generate activity, and check the M365 connector.

---

### Module: AI Gateway in Microsoft Foundry (powered by Azure API Management)
**Problem:** several apps share the same Foundry model deployments with **no control layer**. **Anyone who knows the endpoint can call the model.** There's no visibility into **token consumption**, no **audit trail** of model calls, and no way to **detect or prevent misuse**.

**Solution: AI Gateway**, which is **powered by Azure API Management (APIM)**. It sits **between applications and model deployments** and enforces these **before a request reaches the model**:
- **Authentication**
- **Rate limits** (token limits)
- **Routing policies**
- **Logging** (audit)

*Preview. Check that preview features are available in your subscription and Foundry environment.*

**Required roles (likely exam question):**
| Task | Role needed |
|---|---|
| **Create a NEW APIM instance** for the gateway | **Contributor** or **Owner** on the **resource group** |
| Use an **EXISTING APIM instance** | **API Management Service Contributor** on that APIM instance |
| Configure the gateway on the **Foundry resource** | **Azure AI Account Owner** or **Azure AI Owner** |

> 🎯 **Exam focus:**
> - *Control, rate-limit or audit which apps call Foundry models* / *stop anyone with the endpoint calling the model* / *track token usage* → **AI Gateway (APIM) in Foundry**.
> - Know the role split: **RG Contributor/Owner** for a new APIM, **APIM Service Contributor** for an existing one, plus **Azure AI Account Owner / Azure AI Owner** on Foundry.

### Examine AI Gateway architecture

**Components:**
| Component | Role |
|---|---|
| **Callers** | Apps, **agents**, services requesting inference |
| **AI Gateway (APIM instance)** | **Reverse proxy and enforcement engine**: auth, policies, logging, routing |
| **Model deployments** | Only receive requests that **pass gateway validation** |
| **Azure Monitor / Log Analytics** | Destination for APIM **diagnostic logs**: KQL, dashboards, alert rules |

**Three problems without a gateway (shared endpoint + shared API key):**
1. **No per-caller authentication.** You can't tell which app made a call, and **one compromised app = access to every model** using that key.
2. **No rate limiting or quota.** A runaway or malicious app **exhausts capacity for everyone**. Doing it inside each app is inconsistent.
3. **No central audit trail.** Calls bypass logging, so you'd have to stitch together every app's own logs.

**How the gateway fixes them:**
- **Reverse proxy:** every request goes through APIM, which **validates auth → applies rate limits → logs → routes** to the model.
- **Per-caller auth** with **API keys** or **Microsoft Entra tokens**. Best practice: **each app uses its own managed identity** to get a token. This **removes shared credentials** and gives you **per-app visibility**.
- **Token limits per caller or per subscription**, enforced **at the gateway** (not in the app). Requests over the threshold are **rejected**.
- **Diagnostic logs → Azure Monitor** capture timestamp, **caller identity**, **token count**, response code and latency. You get a full audit trail **without changing the apps**.
- **Advanced configuration** (custom policies, **load balancing**, detailed diagnostics) is done in **APIM in the Azure portal**, not only in the Foundry portal.

**Why agents make this critical:** a Foundry agent can call the model **dozens of times per interaction** (reasoning, tools, subagents), creating **unpredictable bursts**. Without **rate limits**, one **runaway agent** can starve every other caller. Without **per-caller logging**, agent activity is **invisible** in the audit trail.

**Contoso example:** a shared API key is replaced with **one managed identity per app**, each with its own token quota (document analysis 50k tokens/hr, research agent 30k/hr, chatbot 20k/hr with burst). All of them use **one inbound endpoint**. A compromised agent **can't use other apps' quotas or impersonate their identities**.

> 🎯 **Exam focus:**
> - *Shared API key across apps; isolate them and attribute usage* → **AI Gateway + a managed identity per app (Entra token auth)**.
> - *One app or agent is consuming all model capacity* → **gateway token limits per caller/subscription**.
> - *Audit who called the model and how many tokens* → **APIM diagnostic logs → Azure Monitor / Log Analytics**.
> - *Custom policies or load balancing* → configure in **APIM (Azure portal)**.

### Create and configure AI Gateway

**Five steps:** Create the gateway instance → Configure route policies → Set token limits → Configure authentication → Review and test.

**1. Create the gateway (portal and placement facts are high-yield)**
- Done in the **Foundry portal (ai.azure.com)**, **not the Azure portal**: **Operate > Admin console → AI Gateway tab → Add AI Gateway**.
- Pick the Foundry resource, then **Create new** or **Use existing** APIM.

| APIM tier | Use |
|---|---|
| **Basic v2** (what "Create new" deploys) | **Dev/test only** |
| **Standard v2 / Premium v2** (via "Use existing") | **Production** |
| **Premium v2** | **Required** when Foundry has **public network access disabled** and you need **VNet injection + private endpoint** |

- Provisioning takes **5–10 minutes**. Then check status = **Enabled**.
- Enabled at the **Foundry RESOURCE level, not the project level**.
  - **New projects** are gateway-enabled **by default**.
  - **Existing projects** must be added **manually**: select the gateway → find the project → **Add project to gateway**.

**2. Route policies**
- Map the gateway's inbound endpoint to a **model deployment**, with optional filters on **caller identity**, path or headers.
- **One route per model deployment.**
- **Advanced routing:**
  - **Different callers to different models.** For example, internal apps go to a premium model and external agents go to a standard one.
  - **A/B testing**: send part of the traffic to a new model version.

**3. Token limits (HIGH-YIELD: response codes)**
- Set at the **project level per model deployment**. Each project gets its own independent limits.
- Path: **Operate > Admin → select gateway → Token management → + Set limit** → choose the project and deployment → **Limit (Token-per-minute)**.

| Limit type | Window | When exceeded |
|---|---|---|
| **TPM rate limit** | Per **minute** | **429 Too Many Requests**: temporary, resets at the next minute |
| **Total token quota** | Hourly / daily / weekly / monthly / yearly | **403 Forbidden**: quota for the period is **exhausted** |

- Projects are independent. If one hits its limit, the others carry on (Contoso: 50k / 30k / 20k TPM).

**4. Authentication**
| Mode | How it works | Notes |
|---|---|---|
| **API key** | Each caller sends its own key in a header | Simpler, but needs **secure storage + rotation** |
| **Microsoft Entra token** ✅ **production** | The app's **managed identity / service principal** gets a token → `Authorization: Bearer <token>` → the gateway validates **signature, expiry, claims** | **No stored credentials**, works with identity governance |

- **Configure:** auth provider = **Microsoft Entra ID**, and set the required **audience** and **issuer** so the gateway only accepts tokens issued for your environment. Contoso sets the audience to the **gateway's resource ID**.
- Give each app a **managed identity** (system- or user-assigned) and **grant it permission on the gateway**.
- **Foundry agents:** they authenticate with the **managed identity on the agent resource**.
  - Each agent's **unique client ID is logged as the caller**, so you get **per-agent visibility**.
  - **Scope each agent's access to only the deployments it needs.**
  - **Compromised agent → revoke its role assignment**, which leaves every other agent untouched.

**5. Review and test**
- Send a sample request: the token is obtained, included in the header and accepted, and the call appears in the **diagnostic logs with the right caller identity**.
- During rollout, watch for spikes in **auth failures** (misconfigured identities) or **rate-limit rejections** (quotas set wrong).

> 🎯 **Exam focus:**
> - **429 = TPM rate limit (temporary). 403 = total quota exhausted.**
> - **Premium v2** = private networking (public access disabled, VNet injection + PE). **Basic v2** = dev/test only.
> - Gateway is enabled at the **resource level**. **Existing projects must be added manually.** New ones are added automatically.
> - Create it in the **Foundry portal (Operate > Admin console)**, not the Azure portal.
> - Production auth = **Entra tokens + managed identity**, with **audience/issuer** set.
> - **Compromised agent → revoke that agent's role assignment.**

### Secure and monitor AI Gateway access

**Five controls:** network restriction · managed identity for the gateway · diagnostic logging · Azure Monitor metrics · anomaly alerts.

**1. Network restriction**
- **Callers inside Azure:** restrict to their **VNet** and put the gateway behind a **private endpoint**, so traffic **never crosses the public internet**.
- **Callers outside Azure or hybrid:** use an **IP allowlist** of known source addresses.
- **Best practice:** **network controls + Entra token auth** = both layers (network and identity).
- Contoso: apps run in **App Service in one VNet**, and the gateway is reachable **only through a private endpoint**.

**2. The gateway's own identity (gateway → backend)**
- Give the **gateway resource** a **managed identity** (system- or user-assigned) so it can reach model deployments and diagnostic storage **without stored credentials**.
- Grant it **Cognitive Services User** (or a custom role) **scoped to the specific deployments** it routes to.
- ⚠️ This is **separate** from the **callers' managed identities** (app → gateway) covered in the previous unit.

**3. Diagnostic logging**
- **Logs metadata only:** timestamp, **caller identity**, **token count**, response code, latency.
- **Prompt and response content are NOT logged by default.** (Likely trap.)
- Log categories: **Request Logs** and **Authentication Logs**.

| Destination | Use for |
|---|---|
| **Log Analytics workspace** ✅ | **KQL queries + Azure Monitor alerts**. Best for **compliance and incident response** |
| **Storage Account** | **Long-term retention / archive** |
| **Event Hubs** | **Streaming to external systems** (third-party SIEM) |

- Contoso: central Log Analytics workspace, **90-day retention** for compliance.

**4. Azure Monitor metrics and dashboards**
- Track **request volume**, **token consumption per caller**, **rate-limit rejections**, **auth failures** and **error rates**.
- KQL table: **`GatewayLogs`** (APIM's Log Analytics table).
```kusto
GatewayLogs
| where TimeGenerated > ago(24h)
| where ResponseCode == 200
| summarize TotalRequests = count() by ApiName, bin(TimeGenerated, 1h)
| order by TotalRequests desc
```

**5. Anomaly alerts (baseline first, then alert on deviations)**
- A caller hits **>80% of its token limit**: the limit is too low, or the app is over-calling.
- **Error rate >5%**: auth problems or a backend model fault.
- **Off-hours requests** from apps that should only run in business hours.
- **Sudden token spike** (e.g. 10k → 100k per day): possible **credential theft**.
- A **Foundry agent making far more calls than its task needs** (5–10 → hundreds per document): a **retry loop or prompt injection**.
- Notifications go out by **email, SMS** or an **incident management** integration.
- Contoso: a **300% spike at 2 AM on a Sunday** turned out to be an **infinite retry loop**, not an attack. It would have used up all capacity before Monday.

> 🎯 **Exam focus:**
> - *No public internet exposure for the gateway* → **private endpoint + VNet restriction**. *Hybrid / non-Azure callers* → **IP allowlist** (plus Entra auth).
> - *Gateway reaches models without stored creds* → **gateway managed identity + Cognitive Services User**.
> - *Do logs contain prompts?* → **No. Metadata only by default.**
> - *Query and alert on logs* → **Log Analytics**. *Archive* → **Storage**. *Stream to an external SIEM* → **Event Hubs**.
> - *KQL table for APIM gateway requests* → **`GatewayLogs`**.
> - *An agent suddenly makes hundreds of calls* → **retry loop or prompt injection**, caught by an **Azure Monitor alert**.

---

### Module: Foundry guardrails (content filters, blocklists, Prompt Shields)
**What guardrails are:** configurable controls in Microsoft Foundry that evaluate **BOTH prompts (input) AND responses (output)**. They set consistent safety boundaries so AI workloads stay secure, compliant and aligned with policy, **without slowing innovation**.

**The three guardrail controls named:**
| Control | Purpose (headline) |
|---|---|
| **Content filters** | Catch harmful content categories in prompts and responses |
| **Blocklists** | Block **specific custom terms or phrases** your org defines |
| **Prompt Shields** | Detect **prompt injection / jailbreak** attempts |

**Module objectives:** how guardrails secure model interactions · the safety controls · configure and validate **custom guardrails per workload type** · **evaluate effectiveness and refine continuously**.

> 🎯 **Exam focus:** *"Foundry agent guardrails"* is an explicit SC-500 objective (3.1). Guardrails apply to **both input and output**. Expect "which control?" questions:
> - harmful content → **content filters**
> - specific banned words → **blocklists**
> - jailbreak / injection → **Prompt Shields**

### Understand guardrails and Microsoft Content Safety

- **Engine:** Foundry guardrails are **powered by Microsoft (Azure AI) Content Safety**, which uses moderation models that **detect and classify** harmful or sensitive content.
  - **Built in.** No separate setup.
  - **Real-time** on both prompts and responses.
- **When a violation is detected:** Foundry can **flag or block** the request, **log** the event, and **surface it in the portal** for analysis.
- **Layer: the application layer**, where users, data and models interact. It focuses on **model behaviour and data exposure**.
  - It **complements** Defender for Cloud (cloud/workload protection) and Purview (data governance). It **doesn't replace them**.

**What guardrails let you do:**
1. **Prevent ingestion or release** of harmful or sensitive content (both input and output)
2. **Enforce** internal data-handling and **acceptable-use policies**
3. **Detect and mitigate prompt injection / jailbreak**
4. Strengthen **compliance and audit readiness**

- **Control verification:** because guardrails sit close to the model, you can **test prompts and responses in real time**, find policy gaps, and **tune them before deployment**.

> 🎯 **Exam focus:**
> - *What powers Foundry guardrails?* → **Microsoft Content Safety**.
> - *Which layer?* → the **application/model-interaction layer**. Defender for Cloud = workload/infrastructure. Purview = data governance.
> - *Actions on a violation:* **flag or block + log + surface in the portal**.

### Understand safety controls in Microsoft Foundry

- **Where:** the **Guardrails + controls** workspace in Foundry. You configure, test and tune everything there.
- **Try it out:** test sample prompts and outputs against the controls **before applying them to production**.
- **Two levels of protection:**
  - **Input protections:** review prompts **before they reach the model** (jailbreaks, indirect attacks, policy violations).
  - **Output protections:** check responses so **unsafe or confidential info isn't released**.
- Controls can be used **individually or combined**, depending on your risk tolerance.

**The controls (HIGH-YIELD):**
| Control | Detects / does | Applies to | Key config |
|---|---|---|---|
| **Prompt Shields** | **Jailbreak** + **indirect prompt injection** (manipulation that overrides system instructions or extracts restricted data) | **Input** | **Annotate only** (flag via metadata, **doesn't block**) **vs Annotate and block** (stops the prompt and **logs** it) |
| **Content filters** | Harm categories: **Violence, Hate, Sexual, Self-harm** | Input **and** output | **Severity slider per category**: a **Medium** blocking level blocks **medium + high** severity. **High** = the strictest |
| **Blocklists** | **Exact terms or patterns** that must never appear: internal IDs, **project codenames**, proprietary terms | Input **and** output | Custom org/regulatory list |
| **Protected material detection** | **Copyrighted / proprietary third-party content** in generated **text or code** | **Output** | **Annotate or block** |
| **Groundedness detection** *(preview)* | Whether a response is **supported by its source data**, i.e. **fabricated or ungrounded (hallucinated)** statements | **Output** | Accuracy may change while in preview |

- Content filters use the **same detection tech as Microsoft Content Safety**, built into Foundry.

> 🎯 **Exam focus:** match the scenario to the control:
> - *"ignore previous instructions" / hidden instructions in a document or email (indirect)* → **Prompt Shields**
> - *flag but don't block (monitor first)* → **Annotate only**
> - *violence, hate, sexual or self-harm content* → **Content filters** (severity slider)
> - *never output codename "Project Falcon"* → **Blocklist**
> - *model reproduces copyrighted code or lyrics* → **Protected material detection**
> - *model makes up facts not in the source docs (hallucination)* → **Groundedness detection**
> - *test before production* → **Try it out**

### Test built-in guardrails with "Try it out"

- **Purpose:** set a **baseline** for how Foundry's **built-in** safety models behave **before** you design custom filters or blocklists, in a safe environment away from live workloads.
- **Where:** **Guardrails + controls** workspace → **Try it out**. Run the **built-in sample tests** or your **own** text, code or images.

**Test areas available:**
| Area | What you test |
|---|---|
| **Moderate text content** | Classification and blocking across the harm categories and severities |
| **Prompt Shields** | **Jailbreak** + **indirect attack** detection |
| **Groundedness detection** | **Ungrounded / fabricated** responses |
| **Protected material detection** | **Reused / non-original text or code** |
| **Image and multimodal filters** | Moderation of **visual and combined** inputs |
| **Custom categories** | **Org-specific detection criteria** you create and test |

**Reading the results:**
- Which categories triggered, and at what severity.
- **Annotated vs blocked** outcomes, which shows how enforcement behaves.
- Gaps where you need stricter thresholds, refined rules or tuned filters.
- ⚠️ **KEY GOTCHA: your custom blocklists and content filters are NOT applied on the Try it out page.** It only previews **Microsoft's built-in** safety models.

**Order of work:** **test built-ins first (baseline) → then design custom blocklists and filters** that extend them.

> 🎯 **Exam focus:**
> - *Evaluate guardrails safely before production* → **Try it out**.
> - *Your custom blocklist isn't triggering in Try it out* → **expected**. Try it out **only tests built-in models**.
> - *Org-specific detection beyond the built-in harm categories* → **Custom categories**.
> - *Moderate images or mixed inputs* → **Image / multimodal filters**.

### Create and manage blocklists in Microsoft Foundry

**What to block:** confidential **project names**, **proprietary code terms**, **internal URLs / data sources**, any phrases that would expose sensitive info.
**Where:** **Guardrails + controls** workspace. Blocklists are applied **to input and/or output filters** in a guardrail configuration.

**The five steps:**
1. **Create the blocklist** with a descriptive, consistent name (e.g. `RestrictedTermsList`).
2. **Add terms:**
   - **Add new term** (one at a time). Each term is either:
     - **Exact match**: the word or phrase exactly as written.
     - **Regex**: a **pattern** that catches variations, partial phrases, or **token-like strings** (e.g. `Admin-Token 9876`).
   - **Bulk add**: upload a **CSV** with **two columns, `Term` and `Type`** (Exact match / Regex). Good for **migrating existing policy lists**.
3. **Review and manage:** edit, remove or add entries over time.
   - **A blocklist can be reused across multiple guardrail configurations**, keeping enforcement consistent.
   - **Review periodically** as project names and regulations change.
4. **Apply to the guardrail** under **Input filter** AND **Output filter**:
   - **Input** = stop users **submitting** restricted terms.
   - **Output** = stop the model **returning** them.
   - Use **both directions** for a complete boundary.
5. **Validate** with **Try it out**:
   - Test input and output cases.
   - Tune Exact vs Regex.
   - Confirm annotations and blocks show up.
   - Watch for **overblocking**.

> ⚠️ **Conflict between units:** the previous unit said **custom blocklists aren't applied on Try it out** (it only previews built-in models). This unit says to **validate your blocklist on Try it out**, and its screenshot shows a regex blocklist blocking `Admin-Token 9876`. For the exam, if asked **"how do you validate a custom blocklist?"**, answer **Try it out**. If asked **what Try it out previews by default**, answer **built-in models**.

> 🎯 **Exam focus:**
> - *Catch variations or token-like strings* (e.g. `Admin-Token ####`) → **Regex**. *An exact codename* → **Exact match**.
> - *Import hundreds of terms* → **Bulk add, CSV with `Term` + `Type`**.
> - *Stop users entering AND the model returning a term* → apply to **both the Input and Output filter**.
> - *Same list for several workloads* → **reuse one blocklist across multiple guardrail configs**.

### Configure and apply guardrails (content filter configuration)

**Six-step wizard in Guardrails + controls:**
| # | Step | What you set |
|---|---|---|
| 1 | **Basic information** | Descriptive name (e.g. `CustomerSupportFilter`, `InternalCopilotFilter`) |
| 2 | **Input filter** (prompts, *first layer of defense*) | **Harm category thresholds** · **Prompt Shields** (**annotate only / annotate and block / off**) · **Blocklists** |
| 3 | **Output filter** (responses) | **Harm category thresholds** · **Protected material detection** (annotate or block) · **Groundedness** (preview) |
| 4 | **Connection** | **Link the filter to model deployments**, existing or new |
| 5 | **Review** | Confirm thresholds, Prompt Shields, blocklists, input and output |
| 6 | **Validate** | **Try it out**: test input and output, check **detections appear in logs** (audit traceability) |

- **Which controls go where:**
  - **Prompt Shields and blocklists** go on **input**.
  - **Protected material and groundedness** go on **output**.
  - **Harm categories** go on **both**.
- **Thresholds can differ between input and output, and between workloads.** Internal workloads can be looser than customer-facing ones.
- **A filter does nothing until it's connected to a deployment.** Review connections periodically so no model runs **unprotected**.
- Each content filter is a **reusable policy component**: duplicate, modify or extend it. **Share configurations across similar workloads.**
- **Input filters matter most** for **shared or external-facing** workloads, where prompts vary widely in quality and intent.

> ⚠️ **Threshold wording:** the course says a **higher blocking level = blocks more aggressively**, with **Medium** blocking **medium + high**. Azure OpenAI docs phrase it as a *severity threshold*, where setting **Low** blocks low + medium + high (the strictest). **Read the wording in the question carefully:** "blocking level" and "severity threshold" run in opposite directions.

> 🎯 **Exam focus:**
> - *A model is running without guardrails even though a filter exists* → the filter **isn't connected to that deployment**.
> - *Stop jailbreaks* → **input filter → Prompt Shields**. *Stop copyrighted output or hallucinations* → **output filter → protected material / groundedness**.
> - *Same policy for several similar apps* → **shared / reusable content filter configuration**.

### Choose and refine the right guardrails

**Match the control to the workload:**
| Workload trait | Best control |
|---|---|
| **Large or public-facing**, broad harm detection | **Content filters** |
| **Precise policy enforcement**: project names, code tokens, **regulated identifiers** | **Blocklists** |
| **Exposed to end users or connected systems** (high injection/jailbreak risk) | **Prompt Shields** |
| Depends on **verified data sources** (analytics, **knowledge retrieval / RAG**) | **Groundedness detection** |
| **Generates or reuses text/code** | **Protected material detection** |
- Tailor controls to each workload's risk profile. Avoid one-size-fits-all.

**Balancing safety and usability (likely "best practice" question):**
- **Start with moderate thresholds**, then tighten based on testing and feedback.
- **Annotate before blocking**, especially early in rollout.
- **Review logged events** to see what's being blocked and why.
- **Work with product and dev teams** so policies reflect real use, not just theory.
- Too strict means **false positives**, friction and blocked experimentation.

**Measuring effectiveness:**
- Use **Foundry observability (logs/insights)**: which rules fire most, which categories are blocked, how often users see annotations.
- Tune away **false positives and negatives**, and cross-check against **incidents and security alerts**.

**Continuous assurance:**
- Run **periodic audits** of configs and logs, review detections for **emerging risk patterns**, and update blocklists and thresholds for **new findings or regulation changes**.
- **Foundry integrates with Defender for Cloud**, which raises **platform-level recommendations** for AI resources:
  - **missing diagnostic logs**
  - **open network access**
  - **local authentication enabled**
- So you need **guardrails for the AI layer + Defender for Cloud for the Azure layer underneath**.

> 🎯 **Exam focus:**
> - *Rolling out guardrails without disrupting users* → **moderate thresholds + annotate first**, then tighten.
> - *RAG / knowledge assistant must not make things up* → **Groundedness**.
> - *Public chatbot connected to external systems* → **Prompt Shields** (plus content filters).
> - *AI resource has local auth enabled or no diagnostic logs* → a **Defender for Cloud recommendation**, not a guardrail.

---

### Module: Protect AI workloads with Defender for Cloud (AI workloads plan)
**Problem:** AI workloads are built from many connected services (data, models, compute). Each layer brings risks such as **misconfigured storage**, **exposed endpoints** and **malicious prompts**. For example, a model deployed **with public access still enabled** could expose training data or allow prompt manipulation. **Traditional cloud protections often miss AI-specific issues.**

**Defender for Cloud's four jobs for AI:** **Discover** AI assets → **Assess** configurations → **Detect** active threats → **Correlate** incidents.

| Capability | Role for AI | Mindset |
|---|---|---|
| **CSPM** (Cloud Security Posture Management) | **Assess and improve configuration / posture** of AI resources (e.g. public access, local auth, missing logs) | **Proactive**: prevent misconfiguration |
| **CWP** (Cloud Workload Protection), i.e. the **AI workloads plan** (Defender for AI Services) | **Detect runtime threats** as they happen (jailbreak, data leakage, credential theft) | **Reactive**: catch active attacks |

**Module path:**
1. **Enable** the AI workloads plan
2. Review the **Data & AI security dashboard**
3. Improve posture with **CSPM**
4. Detect runtime threats with **CWP**
5. Investigate incidents in **Defender XDR**

> 🎯 **Exam focus:**
> - *Find AI misconfigurations (public access, weak auth)* → **CSPM**.
> - *Detect attacks on AI at runtime* → **CWP / AI workloads plan**.
> - *Single view of AI and data risk* → the **Data & AI security dashboard**.
> - *Investigate correlated AI incidents* → **Defender XDR**.

### Enable the AI workloads plan

- **What it does:** **auto-discovers AI assets** (Foundry, **Azure OpenAI**, **Azure Machine Learning**) and builds an **inventory showing how data, models and services connect**. This is the foundation for posture management and threat detection.
- **Covers the AI lifecycle:** development risks → configuration/posture → runtime threats.

**Enable it (per subscription):** Azure portal → **Defender for Cloud → Management → Environment settings** → choose the **subscription** → **Defender plans** → **AI workloads = On**.
- It's enabled **per subscription**. Turn it on for **every subscription that has AI resources**.
- Discovery results show up in the **Data & AI security dashboard** **within minutes**.

**Optional extensions:** *Monitoring coverage → Settings*. **Not required** for the plan to work.
| Extension | What it adds |
|---|---|
| **Suspicious prompt evidence** | Short, **redacted** snippets of the prompts that **triggered alerts**, shown in **Defender XDR** during investigation |
| **Data security for AI interactions** | Integrates with **Microsoft Purview** to **classify and protect prompt/response data** (compliance) |

**Supporting configuration for full coverage** (not required, but improves findings):
- **Defender CSPM:** posture recommendations + **attack path analysis** for AI resources
- **Sensitive data discovery:** where sensitive or regulated data lives
- **Defender for Storage** + **Defender for Databases:** protect AI **data stores**
- **Cross-cloud connectors:** discover AI workloads in **AWS / GCP**
- **`Microsoft.Security` resource provider registered** on every subscription, so policy and assessment data can flow

> 🎯 **Exam focus:**
> - *Enable AI threat protection* → **Defender for Cloud → Environment settings → subscription → AI workloads On** (**per subscription**).
> - *See the actual prompt text behind an AI alert in XDR* → enable **Suspicious prompt evidence**.
> - *Classify and protect prompts/responses for compliance* → **Data security for AI interactions** (Purview).
> - *Attack paths for AI resources* → needs **Defender CSPM**.
> - *Discover AI in AWS/GCP* → **cross-cloud connectors**.
> - *Assessment data not appearing* → check the **`Microsoft.Security` resource provider is registered**.

### Review insights in the Data & AI security dashboard

**Dashboard sections (know which section shows what):**
| Section | Shows |
|---|---|
| **Overview** | All data and AI resources **by cloud** and **protection status** (fully / partially / not protected) |
| **Top issues** | The most critical **recommendations, alerts and attack paths** |
| **Data closer look** | **Sensitive data discovery** results, common **information types**, **internet-exposed data** assets |
| **AI closer look** | Discovered **AI services**, **prompt and response alerts**, internet-exposed resources used for **grounding or fine-tuning** |

- Each section links into investigation. For example, an **attack path** can show how a **misconfigured storage account exposes grounding data** to the internet. From there you trace which AI service depends on it and fix it with the **linked recommendation**.

**First posture checks after discovery:**
- AI services use **private endpoints, not public access**
- Azure AI accounts use **managed identities, not embedded credentials**
- Sensitive data in AI storage and indexes has **encryption at rest**
- **Internet-exposed** data or compute supporting AI → **fix first, in CSPM**

> 🎯 **Exam focus:**
> - *Which resources are fully, partially or not protected* → **Overview**.
> - *Most critical recommendations, alerts or attack paths* → **Top issues**.
> - *Prompt/response alerts, or exposed grounding/fine-tuning resources* → **AI closer look**.
> - *Sensitive information types, exposed data stores* → **Data closer look**.
> - **Top priority to fix:** **internet-exposed** AI data or compute.

### Assess and improve AI posture with CSPM

- **The AI workloads plan's discovery results feed into CSPM.**
- **Defender for Cloud lifecycle:** **DevOps** (code) → **CSPM** (posture) → **CWP** (runtime) = end-to-end coverage.

**AI Bill of Materials (AI BOM):** CSPM **automatically builds** a map of deployed **AI models, datasets and related services**, so you can confirm where AI runs and whether it meets requirements.

**Cloud Security Explorer (query templates for AI):**
- Path: Defender for Cloud → **General → Cloud Security Explorer**
- Template: **"AI workloads and models in use"** → **Open query** → **Search**. Each result shows the model, data source and resource group, plus metadata.
- Template: **"Generative AI vulnerable code repositories that provision Azure OpenAI"**. Finds **code repos with known vulnerabilities** that could expose model config or **API keys** (shift-left).

**Attack path analysis:** links individual misconfigurations into **routes an attacker could use** to reach AI data or services (e.g. **internet exposure → storage with sensitive data → AI workload**). Use it to prioritize what matters.

**CSPM recommendation categories:**
| Risk area | Recommended action |
|---|---|
| **Identity & access** | **Managed identities** instead of static keys |
| **Network exposure** | **Restrict public endpoints**, use **Private Link** |
| **Data protection** | **Encryption at rest** for all storage and datasets |
| **Dependencies** | **Update vulnerable AI frameworks / libraries** |

> 🎯 **Exam focus:**
> - *Inventory of deployed AI models, datasets and services* → **AI BOM (CSPM)**.
> - *Query which AI workloads/models are in use* → **Cloud Security Explorer → "AI workloads and models in use"**.
> - *Find code repos that provision Azure OpenAI and have vulnerabilities or exposed keys* → **Cloud Security Explorer → "Generative AI vulnerable code repositories…"**.
> - *Show how misconfigurations chain into exposure of AI data* → **attack path analysis**.
> - *Static API keys on an AI service* → recommendation: **use managed identities**.

### Detect AI threats at runtime with CWP

- **CWP = protection for active workloads**, covering AI services, **containers** and **compute** (VMs).
- It detects threats such as **prompt injection**, **data exfiltration**, **model hijacking** and **compromised containers**, in **real time**, then **raises an alert** and **integrates with Defender XDR**.

**Runtime signals CWP flags:**
- **Unusual access patterns** to model or data **storage accounts**
- **Code injection / privilege escalation** in **containers hosting inference**
- **Unauthorized outbound connections** from compute running AI models
- **Anomalous API calls** that suggest abuse of an AI service endpoint

**Find AI alerts:** Defender for Cloud → **General → Security alerts** → **Add filter → Product Component Name = `AI` + `AIServices`**.
- Alternative: **Cloud Security → Workload protections** → click the Security alerts chart. It opens the same list.

**Investigating an alert:**
- The **Alert summary** panel shows severity, description, affected resources and **MITRE ATT&CK tactics**. Then **View full details**.
- The **Alert details** page shows:
  - why the behaviour is suspicious
  - **impacted resources** (model endpoints, containers, storage)
  - **evidence** (accounts, resources, IPs)
  - **Supporting evidence events (Preview)**, including the **Prompt Suspicious Segment**: the exact part of the input that triggered detection
  - recommended actions
- The **Take action** menu lets you change the status, **create an automation rule**, **suppress** the alert, or **trigger a Logic App playbook**. Playbooks can **quarantine resources**, **disable compromised credentials**, or notify **Teams / ServiceNow**.

**Supporting plans for runtime coverage (and fewer false positives):**
| Plan | Protects |
|---|---|
| **Defender for Servers** | Compute hosting AI: **EDR + vulnerability scanning** |
| **Defender for Containers** | Containerized **training & inference** |
| **Defender for Storage** | **Malware / suspicious access** in model & dataset repositories |
| **Defender XDR integration** | Correlates AI alerts with other incidents |

> 🎯 **Exam focus:**
> - *Show only AI alerts* → filter **Product Component Name = AI / AIServices**.
> - *See which part of the prompt triggered the alert* → **Prompt Suspicious Segment** (supporting evidence events). This needs the **Suspicious prompt evidence** extension enabled.
> - *Auto-remediate (quarantine, disable creds, notify Teams/ServiceNow)* → **Take action → Logic App playbook / automation rule**.
> - *Malware uploaded to a training dataset store* → **Defender for Storage**. *An inference container is compromised* → **Defender for Containers**.

### Investigate AI alerts with prompt evidence in Defender XDR

- AI alerts from Defender for Cloud **automatically surface in Defender XDR**, **correlated into incidents** with identity, endpoint and data signals. This shows whether an AI alert is **isolated or part of a bigger attack**.

**Find AI incidents:** Defender portal → **Investigation & response → Incidents & alerts → Incidents** → **Filter: Service/detection source = Microsoft Defender for Cloud → Microsoft Defender for AI Services**.

**See the prompt that triggered it (high-yield path):**
1. Open the incident → **Alerts** tab → select the alert (e.g. **jailbreak**)
2. In the alert details → **Activities** → **Related activities**
3. Select an event → read the **Prompt Suspicious Segment** field in the event details pane
- It shows the **captured prompt text or API payload**, so you can confirm a malicious prompt, an unexpected API request or a data-extraction attempt.
- Prompt segments appear in **both Defender for Cloud and Defender XDR**.

**Manage incident pane fields:**
| Field | Values / use |
|---|---|
| **Severity** | Adjust for impact or urgency |
| **Assign to** | Owner |
| **Status** | **Active / In Progress / Resolved** |
| **Classification** | **True positive / Informational, expected activity / False positive** |
| **Incident tags** | Context for filtering and reporting |
- Other tabs: **Evidence and Response**, **Alerts**.

> 🎯 **Exam focus:**
> - *Filter XDR incidents to AI threats* → **Service/detection source = Microsoft Defender for AI Services**.
> - *Where is the prompt text in XDR?* → Alert → **Activities → Related activities → event → Prompt Suspicious Segment**.
> - *A legitimate red-team test triggered the alert* → Classification **Informational, expected activity**.
> - *Is the AI alert part of a wider attack?* → look at the **correlated incident** in XDR (timeline, assets, other alerts).

### 📌 Module recap: Defender for Cloud for AI workloads

**End-to-end flow (remember the order):**
| # | Step | Tool | Answers |
|---|---|---|---|
| 1 | **Discover** | **AI workloads plan** (Environment settings, per subscription) | *What AI resources exist?* |
| 2 | **See** | **Data & AI security dashboard** | *How protected are they? Where's the sensitive data?* |
| 3 | **Harden** | **CSPM**: AI BOM, Cloud Security Explorer, attack paths, recommendations | *What's misconfigured? Fix it before it's attacked* |
| 4 | **Detect** | **CWP**: runtime alerts (prompt injection, exfiltration) | *What's under attack right now?* |
| 5 | **Investigate** | **Defender XDR** incidents + Prompt Suspicious Segment | *Is it part of a bigger attack? What was the prompt?* |

- **Classic exam scenario:** a developer leaves a **model endpoint publicly exposed** → **CSPM** flags it (recommendation / attack path) → **CWP** alerts if it's exploited → you **investigate in XDR**.

**Microsoft Learn references:**
- [AI threat protection overview](https://learn.microsoft.com/en-us/azure/defender-for-cloud/ai-threat-protection)
- [Enable threat protection for AI services](https://learn.microsoft.com/en-us/azure/defender-for-cloud/ai-onboarding)
- [Data and AI security dashboard](https://learn.microsoft.com/en-us/azure/defender-for-cloud/data-aware-security-dashboard-overview)
- [Discover generative AI workloads](https://learn.microsoft.com/en-us/azure/defender-for-cloud/identify-ai-workload-model)

---

### Module: Defender for AI Services plan (enable, investigate, monitor)
**Scenario gap:** Contoso protected **Copilot Studio agents with Defender for Cloud Apps** (session layer). Its **Azure OpenAI / Foundry model inference layer had no threat detection**, so a jailbreak or exfiltration there produced **no alert, no evidence, no visibility**.

**Fix:** the **Defender for AI Services** plan, a **CWP plan in Defender for Cloud**, enabled **per Azure subscription**.
- **Scans text tokens at inference time** and alerts on **jailbreak**, **data leakage** and **credential theft**.
- Alerts flow into **Defender XDR** alongside identity, endpoint and cloud app signals.

**Prerequisite role:** **Owner or Contributor** on the target **subscription**.

**Naming note:** the previous module called it the **"AI workloads"** plan and the earlier MDCA unit called it the **"AI services"** plan. These are the **same CWP plan**: **Defender for AI Services**.

**Module path:** enable and configure the plan → investigate alerts in the Defender portal → monitor coverage in the **Data and AI security dashboard**.

> 🎯 **Exam focus:**
> - *Copilot Studio is protected by MDCA, but Azure OpenAI/Foundry has no detection* → enable **Defender for AI Services** on the subscription. **Enabling one doesn't enable the other.**
> - *Role needed to enable it* → **Owner or Contributor on the subscription**.

### Enable and configure the Defender for AI Services plan

- **Engine:** integrates **Azure AI Content Safety Prompt Shields** + **Microsoft threat intelligence** to scan **text tokens at inference time**.
- **Scope:** **Azure OpenAI** and **Azure AI Model Inference** supported models.
  - **Copilot Studio agents are NOT covered here.** They're protected by **Defender for Cloud Apps**.

**Four threat categories (memorize):**
| Threat | Meaning |
|---|---|
| **Jailbreak** | Prompts crafted to **bypass model safety instructions** |
| **Data leakage** | Sensitive data **exposed in responses** |
| **Credential theft** | Extracting **secrets / keys / credentials** through AI interactions |
| **Data poisoning** | **Manipulated inputs** to corrupt model behaviour |

- ⚠️ **Text tokens only. Image and audio tokens are NOT scanned** (current release).
- Alerts go to **Defender XDR**, so your existing SOC workflows apply without separate tooling.

**Enable:** Defender for Cloud → **Environment settings** → subscription → **Defender plans** → **AI services = On** → **Save**.
- Alerts appear **within minutes** if there's threat activity.
- ⚠️ It's **per subscription**. A subscription **without** this plan has **no AI threat detection**, **even if other Defender plans are on**.

**Three optional components** (*AI services → Settings*):
| Component | Enable if… | Notes |
|---|---|---|
| **Suspicious prompt evidence** (toggle *"Enable user prompt evidence"*) | The SOC needs alert context **and** data policy **allows access to prompt content** | **Default = prompt/response text MASKED.** When on, alerts include **redacted snippets** of the prompt **and** response. **Off = detection still works**, you just can't see the text |
| **Data security for AI interactions** | You have a **Microsoft Purview license** and compliance needs **audit / classification / eDiscovery** | **Separate Purview license required** (not included). Enables **SIT classification**, **DSPM for AI** analytics, **Insider Risk Mgmt**, **Communication Compliance**, **Purview Audit**, **data lifecycle mgmt**, **eDiscovery** |
| **AI model security** | **Custom models in Azure Machine Learning registries** | Scans registered models for **serialization vulnerabilities, embedded malware, exposed secrets**. Findings appear as **recommendations**. No overhead, so it's fine to turn on as a precaution |

**Purview integration scope limits (likely trap):**
- Covered: **Foundry direct model API calls** using **Entra ID auth with a user-context token**.
- **API calls without user context** only appear in **Purview Audit** and **DSPM for AI Activity Explorer**.
- **Foundry agent interactions are NOT captured at all** (not supported yet).

**Contoso example:** two subscriptions with Foundry, so the plan goes on **both**, plus prompt evidence (for SOC triage) and AI model security (they have custom ML models). The **Purview integration is deferred to the compliance team**, who own the license and policy.

> 🎯 **Exam focus:**
> - **4 threats:** jailbreak, data leakage, credential theft, **data poisoning**.
> - **Text only**: images and audio aren't scanned.
> - *SOC can't see the prompt in alerts* → turn on **Suspicious prompt evidence** (it's masked by default).
> - *Compliance needs eDiscovery / Insider Risk on AI prompts* → **Data security for AI interactions** + a **Purview license**.
> - *Scan custom models for malware or serialization flaws* → **AI model security** (Azure ML registries).
> - *Foundry **agent** interactions missing in Purview* → **not supported yet**.
> - *AI threats not detected on one subscription* → the plan isn't enabled **on that subscription**.

### Monitor AI security with the Data and AI dashboard (new points only)
*(Dashboard sections are already covered above under "Review insights in the Data & AI security dashboard".)*

**What fills each part of the dashboard (likely trap):**
| Dashboard area | Populated by |
|---|---|
| **Data sections** | **Defender CSPM** (+ **sensitive data discovery** extension), **Defender for Storage**, **Defender for Databases** |
| **AI sections** | **Defender for AI Services** plan |
- With **only** the AI plan on, the dashboard opens but shows **partial data**.
- **Minimum role to view the dashboard: Security Reader.**
- **Path:** Defender for Cloud → **Data and AI security** (left menu).
- **AI closer look** shows **prompts scanned** and **alerts by severity**.

**Ongoing checks:**
- Confirm **every subscription with AI services** shows **AI threat protection active**. A gap → enable the plan in that subscription's Environment settings.
- **Review weekly** so **newly provisioned AI subscriptions** don't go unprotected.

**Confirm alert delivery:**
- **Defender portal → Incidents & alerts → Alerts** → filter **Service source: Microsoft Defender for Cloud**.
- **Sentinel:** alerts reach Sentinel through the **Microsoft Defender XDR data connector**. **AI alerts missing in Sentinel → check that connector is active.**
- Each alert should have: title and severity, affected resource, **prompt evidence**, recommended actions.
  - **Prompt evidence field empty → enable the prompt evidence toggle** in Environment settings.

**False positives (pen test / red team):**
- Set Classification to **False positive** + add a comment.
- **Suppression rules** in Defender for Cloud **auto-suppress recurring false positives** from specific resources.

> ⚠️ **Conflict:** the earlier XDR unit listed **"Informational, expected activity"** as the classification for benign/expected events. This unit says to mark **authorized red-team / pen-test alerts as False positive**. For the exam, **match the wording**: if it says "the detection was wrong", answer **False positive**. If it says "real activity but authorized or expected", answer **Informational, expected activity**. If the question is phrased exactly like this unit (red team → classify), lean **False positive**.

**Notifications:**
- **Environment settings → subscription → Email notifications.** Set the recipients and the **severity threshold**. For AI, set it to **High at minimum** (jailbreak, data leakage).
- **Defender XDR alert rules** route AI incidents to dedicated queues or **Sentinel playbooks** for automated triage.
- **Role split:** the security engineer confirms alerts flow, severity is right and notifications are set up. **The SOC does the investigation.**

> 🎯 **Exam focus:**
> - *Dashboard shows partial data* → enable **Defender CSPM (+ sensitive data discovery), Storage, Databases**.
> - *Least privilege to view the dashboard* → **Security Reader**.
> - *AI alerts not in Sentinel* → check the **Defender XDR data connector**.
> - *Recurring pen-test alerts creating noise* → **suppression rules**.
> - *Email the team on jailbreaks* → **Email notifications, severity threshold High**.

### 📌 Module recap: Defender for AI Services plan
- **The gap:** **MDCA protects Copilot Studio** (session layer), but **Azure OpenAI / Foundry inference had no detection**. The fix is **Defender for AI Services**, enabled on **every subscription** hosting AI.
- **Configure:** turn on **Suspicious prompt evidence** (SOC triage). **Purview integration** = the compliance team's call (separate license). **AI model security** = when there are custom ML-registry models.
- **Monitor:** the **Data and AI security dashboard** for coverage, then confirm alerts reach **XDR / Sentinel**, set up **High-severity email notifications**, and suppress known false positives.
- Learn more: [AI threat protection overview](https://learn.microsoft.com/en-us/azure/defender-for-cloud/ai-threat-protection) · [Enable threat protection for AI services](https://learn.microsoft.com/en-us/azure/defender-for-cloud/ai-onboarding)

---

### Module: Manage agents with Microsoft Agent 365 (M365 admin center)
*Covers the SC-500 objective "Manage agents in the Microsoft 365 admin center".*

- **Microsoft Agent 365 became GA on 1 May 2026.**
- **Problem:** department heads are requesting agents, but there's **no formal approval process**. Agents get deployed **without review**, and nobody has a central view of **which agents are active, who approved them, or what they access**.
- **Agent 365 = the governance layer for agents, inside the Microsoft 365 admin center:**
  1. **Enable and navigate** the management interface
  2. **Register agents** through an **approval workflow**
  3. Apply **usage policies** and **scope restrictions** (access controls)
  4. **Monitor agent activity** to detect violations and enforce **accountability**
- **Prereqs:** M365 admin experience **with Microsoft 365 Copilot licensing**, plus an understanding of your org's AI governance policy.

> 🎯 **Exam focus:**
> - *No approval process for agents* / *who approved this agent?* / *central view of active agents in M365* → **Microsoft Agent 365 in the Microsoft 365 admin center**.
> - Know the flow: **register/approve → access controls → monitor**.

### Enable and navigate Microsoft Agent 365

- **Licensing:** requires **Microsoft 365 Copilot**. Copilot orgs **inherit Agent 365 automatically**, with **no separate install**.
- **Without Agent 365:** Copilot Studio and user-submitted agents show up **directly in the M365 Copilot experience with no admin oversight**. That's a big risk for regulated industries.
- **Path:** **M365 admin center → Agents**
  - **Overview** = governance dashboard: activity, pending requests and governance gaps over the **last 30 days**
  - **All agents** = **Agent Registry**

**Roles (HIGH-YIELD):**
| Can **manage** (approve, block, scope, assign ownership) | **View-only** (registry + activity, no governance) |
|---|---|
| **AI Administrator**, **Global Administrator** | **AI Reader**, **Global Reader**, **Security Administrator**, **Security Reader**, **Security Operator** |
- ⚠️ **Security Administrator CANNOT approve agents** here. It's view-only. The SOC can investigate but **can't approve or assign ownership**.
- **"Agents" menu missing?** → check for an **M365 Copilot license** + an **AI Admin or Global Admin** role.

**Management areas:**
| Area | Path | Purpose / actions |
|---|---|---|
| **Registry** | Agents → All agents → **Registry** | All agents in the catalog (**Copilot Studio**, **approved external vendors**, **Microsoft-built**). **Filter by Status** |
| **Requests** | Agents → All agents → **Requests** | Submissions **awaiting approval** (requestor, date, requested capabilities). Actions: **Publish** (approve) or **Reject** |
| **Status: Available** | Registry | Approved: **review usage, modify user scope, block, delete** |
| **Status: Blocked** | Registry | **Hidden from all users**, but **stays in the registry**. **Reversible (unblock)**. Good for **temporary enforcement during investigations** |
- Registry columns: name, status, owner/publisher, category, last activity.
- **Lifecycle:** Copilot Studio submission → **Requests** → (**Publish**) → Registry **Available** → **Blocked** (reversible) → **Deleted**.

> 🎯 **Exam focus:**
> - *Least-privilege role to approve or publish agents* → **AI Administrator** (not Security Admin, not Global Admin).
> - *SOC needs to see agents but must not approve them* → **Security Reader / Security Operator / AI Reader**.
> - *Approve a submitted agent* → **Requests → Publish**.
> - *Temporarily stop an agent during an investigation* → **Block** (reversible), not delete.
> - *Agents menu not visible* → **M365 Copilot license** + an admin role.

### Register agents and apply access controls (Agent 365)

**Approval workflow:**
- An agent is built in **Copilot Studio** and **published to the tenant**. That creates a submission in **Requests**.
- **Until it's approved, nobody can use it, not even its creator**, in production M365 Copilot.
- The detail view shows the name, description, publisher/creator, category, and the **permissions and data access requested** (mailbox, SharePoint, Graph, external APIs through Power Platform connectors).
- ⚠️ **Approval ≠ access for everyone.** Approving makes the agent *available*. **User scope** controls **who** can use it. Approve once, then **roll out in stages to pilot groups**.

**The four approval criteria:**
| Criterion | What to check |
|---|---|
| **Least privilege** | Only the permissions its function needs (a survey agent needs **read on one SharePoint list**, not write on all sites). Too broad → **return with feedback** |
| **Trusted publisher** | Internal = creator's email. External = vendor identity. It must be on the **approved vendor list** |
| **Clear business purpose** | A documented process, the data accessed, the user base. "Helps with productivity" isn't enough. Becomes part of the **governance / audit record** |
| **AI policy compliance** | Personal data, human oversight, automated decisions. **Non-compliant → BLOCK, don't leave it unapproved** |

**Access controls:**
| Control | Purpose | Where |
|---|---|---|
| **User scope** | Limit which **users or M365 groups** can use the agent | Agent settings → **Manage access** |
| **Connector permissions** | Connectors must **comply with DLP** policies | **Power Platform admin center → Data policies** |
| **External data access** | **Data residency / sovereignty** compliance | Agent detail view → **Permissions & data access** |

- **Default scope is usually org-wide**, which is the **highest risk**. Narrow it to the **smallest audience** (specific users or groups = lowest risk). That limits blast radius and follows least privilege.
- **Copilot Studio agents use OBO auth by default:** the agent acts **as the signed-in user**.
  - The **user's own permissions are the effective ceiling** on what the agent can reach.
  - Audit logs record the action **as the user**, with agent context.
  - So **least privilege for users matters as much as for agents.**
- **DLP example:** SharePoint = *business* connector, Dropbox = *non-business*. An agent moving files between them **violates DLP → block it** until it's redesigned.
- **External API in a non-compliant region** (data residency breach) → **block**.

**Return with feedback vs Block:**
| Action | When |
|---|---|
| **Return with feedback** | Valid purpose but needs **minor fixes** (reduce scope, clarify description or retention) |
| **Block** | **Violates policy** or **unacceptable risk**. The agent stays blocked **even if it's modified and resubmitted**. The **reason and admin are recorded** (audit trail) |

**Contoso (14 agents):**
- **10 approved**, scoped to the requesting department.
- **3 blocked:**
  - read on all mailboxes without justification
  - an external API not on the vendor list
  - a duplicate of an existing agent
- **1 returned**: needed clarification on customer **data retention**.

> 🎯 **Exam focus:**
> - *Agent approved but only HR should use it* → **Manage access → user scope = HR group**.
> - *Agent's connectors breach DLP* → check **Power Platform admin center → Data policies**, then **block**.
> - *Agent sends data to an API in a disallowed region* → **block** (residency).
> - *Valid agent asks for too many permissions* → **return with feedback**. *Agent violates AI policy* → **block**.
> - *What can a Copilot Studio agent access?* → by default (**OBO**), **no more than the signed-in user can**.

### Monitor agent activity and enforce governance (Agent 365)

**What to watch for:**
| Focus | Indicator | Action |
|---|---|---|
| **Anomalous usage** | **500% over baseline** | Investigate: automation or misuse? |
| **Inactive agents** | **Zero interactions for 60+ days** | Review for **decommissioning** |
| **Policy violations** | **DLP alerts** / unauthorized access attempts | **Block pending investigation** |

**Where to look:**
- **Agent → Usage / Activity tab** shows **total interactions** since approval, **unique users**, and a **daily chart for the last 30 days**.
- The **Agent overview** has two cards:
  - **"Agents at risk"** aggregates risk signals from **Entra, Defender and Purview** in **one place**.
  - **"Agents without owners"**: when the creator **leaves the org**, the agent becomes **ownerless**, which is an accountability gap. **Assign a new owner.** (Needs **AI Admin or Global Admin**.)

**Reading the patterns:**
- Steady growth = healthy adoption.
- **Spike then silence** = automation, misuse, or the process is no longer needed.
- **Activity between midnight and 6 AM** on an interactive agent = probably **batch automation**. Check it against the approved purpose.
- **High interactions, few unique users** (e.g. 10,000 / 5) = a **small group driving the volume**.
  - If it's within policy, **update the documentation**.
  - If not, **block** and fix the workflow.
- **Inactive agents are still a risk:** they use licenses, clutter the catalog, **expand the attack surface**, and their permissions go unmonitored. Check with the submitter (it may be a **seasonal** need).

**Enforcement actions:**
| Action | Effect | Use when |
|---|---|---|
| **Block / Unblock** | Immediately removes access for **everyone**. **Reversible** | **During an investigation** (e.g. DLP alerts), then unblock |
| **Narrow user scope** | Shrinks the audience | Scope **crept** beyond intent (pilot of 20 → 200 via a distribution list) |
| **Revoke approval** | Sends the agent back to **pending**, so it **must be re-approved** | The publisher **added new permissions or data connections** after approval, or the agent **suddenly accesses new SharePoint sites** |

**Continuous governance cycle:** **Approve → Monitor → Review → Remediate** (repeat)
- **Monthly:** usage anomalies
- **Quarterly:** all active agents against current policy (also filter for inactive ones)
- **Annually:** **re-approve high-access agents** (broad permissions or sensitive data)

**Contoso:** a Sunday-night usage spike turned out to be **undocumented weekend batch processing**. They **blocked → reviewed against data retention → unblocked with updated approval docs**.

> 🎯 **Exam focus:**
> - *Agent's creator left the company* → **Agents without owners → assign an owner**.
> - *A single view of risk signals from Entra, Defender and Purview for agents* → **"Agents at risk"** card.
> - *Publisher added new permissions after approval* → **Revoke approval** (back to pending).
> - *DLP alerts on an agent, need time to investigate* → **Block** (reversible), then unblock.
> - *Pilot scope grew too far* → **narrow user scope**.
> - *No use for 60+ days* → **review for decommissioning**.
> - *Formal yearly checkpoint for high-access agents* → **annual re-approval**.

### 📌 Module recap: Microsoft Agent 365
| Phase | Key facts |
|---|---|
| **Enable & navigate** | **M365 admin center → Agents**. Needs an **M365 Copilot license**. **AI Admin / Global Admin** to govern, everyone else **view-only**. Registry, **Requests** (Publish/Reject), **Available / Blocked** |
| **Register & control** | 4 criteria: **least privilege, trusted publisher, business purpose, AI policy**. **User scope** via Manage access. **Connector DLP** in the Power Platform admin center. **Block with a documented reason** |
| **Monitor & enforce** | Usage/Activity tab, **Agents at risk**, **Agents without owners**. **Block / Narrow scope / Revoke approval** |
| **Cycle** | **Approve → Monitor → Review → Remediate** |
- **Contoso outcome:** 14 pending agents → **10 approved / 3 blocked / 1 returned**. A Sunday-night usage anomaly was caught, investigated and documented.
- Further reading: [Microsoft Agent 365 overview](https://learn.microsoft.com/en-us/microsoft-agent-365/overview)

---

### Module: Purview DSPM for AI (SharePoint oversharing + Copilot data risk)
*Covers the SC-500 objectives "Identify overexposure of data in SharePoint" and "Identify risks to Copilot/AI apps by using Purview DSPM".*

- **Problem:** M365 Copilot is in use and productivity is up, **but** the security team can't see:
  - which **sensitive data Copilot surfaces**
  - which **SharePoint sites are used as AI grounding sources**
  - whether interactions touch **restricted client financial records**
  - *The question isn't whether AI is used, but whether it's used **safely**.*
- **Microsoft Purview DSPM for AI** answers this **without log exports or custom queries**. It:
  1. **Discovers how AI tools interact with org data**
  2. **Identifies SharePoint content overexposed to AI grounding** (oversharing)
  3. **Surfaces interaction risks** from **Copilot and other AI apps**
- **Module path:** configure DSPM for AI → assess SharePoint overexposure → find sensitive-data risks in AI interactions → read the dashboards and prioritize fixes.

> 🎯 **Exam focus:**
> - *Which sensitive data is Copilot surfacing?* / *Which SharePoint sites are overexposed to Copilot grounding?* → **Purview DSPM for AI** (in the **Purview portal**, not Defender).
> - Don't confuse it with **Defender for Cloud's Data & AI dashboard**, which covers Azure resource posture and threats.

### Configure DSPM for AI

- **Two versions exist:**
  - **DSPM for AI (classic)**: *Purview portal → Solutions → DSPM for AI (classic)*. Covered here, still fully functional.
  - The **newer DSPM**: broader data sources, guided workflows, unified UI. **New features land only in the new version.**
- **Focus:** unlike DLP, which focuses on *preventing exfiltration*, DSPM for AI is about **understanding which sensitive content AI can access and reference**.

**The three risk areas it monitors:**
1. **Copilot referencing sensitivity-labeled content** in interactions
2. **SharePoint sites used for Copilot grounding** that hold **broadly accessible sensitive data**
3. Users **pasting or typing sensitive data into external AI sites** in a browser
- **Discovery runs continuously:** which Copilot features are used, which SharePoint sites are used for grounding, which external AI apps connect.

**Prerequisites (HIGH-YIELD):**
| Need | Detail |
|---|---|
| Role | **Compliance Administrator** (or equivalent with info-protection rights) at purview.microsoft.com |
| **Purview Audit enabled** | On by default for new tenants, but **verify**. No audit = no data |
| **M365 Copilot licenses** | Assigned to the users you want to monitor |
| **M365 E5** or **Microsoft Purview Suite** (formerly E5 Compliance) | ⚠️ **E3 does NOT qualify** |
| *External AI sites (ChatGPT, Gemini)* | **Devices onboarded to Purview** + **Purview browser extension** installed |
| *Sensitivity labels* | **Recommended, NOT required**. Without labels you get **limited insights** (SIT detection only). Deploying labels is the **highest-value step** |

**Get started: three one-click policies** (each auto-creates the underlying **DLP or Communication Compliance** rule):
| Policy | Detects |
|---|---|
| **Detect risky AI usage** | **Risky prompts/responses** across Copilot and AI apps |
| **Unethical behavior in AI apps** | **Inappropriate content** in AI interactions |
| **Detect sensitive info shared with AI via network** | Sensitive data **sent to external AI sites** |

**Dashboard sections:**
| Section | Answers |
|---|---|
| **Reports** | Which AI tools are active and the **volume of sensitive interactions**. Three categories: **Copilot experiences & agents** (M365 Copilot, Copilot Studio) · **Enterprise AI apps** (ChatGPT Enterprise) · **Other AI apps** (ChatGPT consumer, Gemini) |
| **Data risk assessments** | **SharePoint overexposure**: auto **weekly** scan of the **top 100 SharePoint sites by usage**. **Custom assessments** go beyond that |
| **Activity explorer** | **Event-level** detail: activity type, user, app category, **SITs detected**, **files referenced** |
| **Apps and agents** | Every discovered AI tool, plus **per-agent** detail (what sensitive data each agent touched, whether it's policy-protected) |
- ⚠️ **Security Copilot** only appears in Apps and agents **if a separate collection policy is configured**. It's **not automatic**.
- **Quick wins:** start with **Data risk assessments**.

> 🎯 **Exam focus:**
> - *No Copilot data in DSPM for AI* → check that **Purview Audit is enabled** + **Copilot licenses are assigned**.
> - *Org has M365 E3* → **doesn't qualify**. You need **E5 / Purview Suite**.
> - *Detect sensitive data pasted into ChatGPT/Gemini* → **device onboarding + Purview browser extension**.
> - *Fastest way to start detecting risky prompts* → the **one-click policies** in **Get started**.
> - *SharePoint oversharing view* → **Data risk assessments** (weekly, top 100 sites).
> - *Event-level detail of which files Copilot referenced* → **Activity explorer**.

### Assess SharePoint overexposure

- **Why it matters:** Copilot grounds answers in **everything the user can access**. If a site's **permissions are broader than its content's sensitivity**, Copilot will **surface confidential content to users who shouldn't see it**.
- **How DSPM detects it:** it correlates **(1) which sites Copilot queries + (2) the sensitivity labels on their files + (3) who has access**. A **mismatch** = overexposure.

**The four tabs on a site's flyout (Data risk assessments):**
| Tab | Shows / does |
|---|---|
| **Overview** | Items scanned, sensitive data found, **broad sharing links** |
| **Identify** | How much content was scanned for SITs, plus an **on-demand classification scan** |
| **Protect** | **Remediation actions** (below) |
| **Monitor** | How items are shared: **specific people / organization / external users / anyone-with-the-link** |

**Protect tab remediation actions (HIGH-YIELD):**
| Action | What it does | Human access? |
|---|---|---|
| **Restrict access by label** | Creates a **DLP policy** that stops **Copilot and agents summarizing** content with specific labels. **No permission change** | ✅ Unchanged |
| **Restrict all items** | **SharePoint Restricted Content Discovery** **removes the site from Copilot grounding entirely** | ✅ **Preserved** (AI access only is removed) |
| **Create auto-labeling policy** | Auto-applies labels to **unlabeled sensitive files** (SIT detected) | n/a |
| **Create retention policies** | **Deletes stale content** (not accessed in **3+ years**), shrinking the oversharing surface | n/a |
- **The most direct fix:** **tighten SharePoint permissions** (company-wide → specific security groups). That cuts **both human and AI** exposure.
- **Relabel** wrongly classified files (e.g. "Internal" → "Confidential – Finance Only").
- **Excluding a site from Copilot grounding** removes the risk, but **reduces Copilot's usefulness** for authorized users too.

**Priority:**
| Risk | Labels + access | Action |
|---|---|---|
| **High** | **Highly Confidential / Confidential** + large groups | **Immediate, within 24 h**: restrict site access |
| **Medium** | Confidential / Internal, beyond the owning team | **Within 1 week**: tighter access, relabel, CA |
| **Low** | Internal / Public, or label accuracy issues | **Monitor**: labeling hygiene, access reviews |
- **Prioritize by sensitivity × access breadth (× Copilot activity).**

**Defense in depth:** use **labels + permissions** together. Even if permissions are misconfigured, **labeled files still trigger Purview DLP**.

**Contoso:** the "Earnings Analysis" site (Confidential, open to the whole finance group) was showing prerelease data to junior analysts. They **restricted it to senior analysts** and **labeled the reports "Highly Confidential – Earnings Data"**, so Copilot still works for authorized users.

> 🎯 **Exam focus:**
> - *Stop Copilot summarizing labeled files without changing SharePoint permissions* → **Restrict access by label** (DLP).
> - *Remove a site from Copilot grounding but keep human access* → **Restrict all items / Restricted Content Discovery**.
> - *Sensitive files are unlabeled* → **auto-labeling policy**.
> - *Old, unused content widening exposure* → **retention policy** (3+ yrs no access).
> - *Reduce exposure for both humans and AI* → **tighten site permissions**.
> - *See anyone-with-the-link sharing* → the **Monitor** tab.
> - *Run a classification scan now* → the **Identify** tab.

### Identify risks in Copilot and AI app interactions

**The four interaction risk types:**
| Risk | Meaning |
|---|---|
| **Sensitive data in prompt** | The user puts labeled content **into** a Copilot prompt |
| **Sensitive data in response** | Copilot's answer **references labeled files** |
| **Sensitive data pasted to external AI** | Labeled content pasted or typed into an **external AI site** in a browser |
| **Cross-boundary interaction** | Sensitive data **moves between security zones** through AI |
- Interaction risks happen **during AI generation itself**, not when an email is sent or a file is shared (that's classic DLP).
- **A flagged risk ≠ a policy violation.** It means sensitive data was **involved** and is **worth reviewing**.

**Activity explorer activity types:**
| Activity type | Meaning |
|---|---|
| **AI interaction** | A Copilot/agent interaction with **prompts and responses captured** |
| **AI website visit** | The user browsed to an **external AI site** |
| **Sensitive info types** | Sensitive info **detected** during an AI interaction |
| **DLP rule match** | A **DLP policy matched** during an AI interaction |
- Each event shows the activity type, label/SIT, user, **AI application** (M365 Copilot / Copilot Studio agent / ChatGPT Enterprise / external consumer AI) and timestamp.
- **External AI** needs **device onboarding + the Purview browser extension**. It logs an **AI website visit** + a **Sensitive info types** event.
  - ⚠️ For browser interactions you get **SIT-level detail only: NO file name or size**.

**Interaction detail pane:**
- Shows the **files involved**, the user, the full label (including **sublabels**), and whether the sensitive data was in the **prompt, the response, or both**.
- **Each back-and-forth turn with labeled content = a separate event**, so you can tell one-off use from extended sessions.

> ⚠️ **DSPM for AI is VISIBILITY ONLY. It does NOT block.** To **prevent** uploads to external AI, configure **Purview DLP policies** (block or warn).

**Priority:**
| Priority | Scenario |
|---|---|
| **Highest** | **Highly Confidential** + **external AI site via browser** (data leaving the perimeter) |
| **High** | Copilot used **Highly Confidential** content and the user **shared the output externally** |
| **Medium** | Copilot used **Confidential** content and the output **stayed internal** |
| **Lower** | **Internal** content / the user had explicit permission to every file referenced |

**From risk to remediation (linked from the detail pane):**
- **Sensitivity label policy** for the content
- **SharePoint site permissions** where the files live. **Root cause = broad permissions → go to Data risk assessments**
- **DLP rules**. **External AI risk → create or update DLP to block or warn** before pasting
- **Review weekly.** The same users and the same external app again and again → **targeted training** or **app controls**.

**Contoso:** an analyst emailed a Copilot summary of a "Confidential – Client Data" report to an external consultant. Fix: **training** (labels carry over to AI-generated content) + a **DLP policy that warns** before sending Copilot summaries of confidential data.

> 🎯 **Exam focus:**
> - *DSPM shows users pasting sensitive data into ChatGPT, so stop it* → **Purview DLP** (DSPM itself **doesn't block**).
> - *Which files did Copilot use for this answer?* → **interaction detail pane / Activity explorer**.
> - *Need the file name for a ChatGPT paste event* → **not available** (SIT-level only).
> - *Top priority* → **Highly Confidential + external AI site**.
> - *The interaction risk comes from overshared SharePoint* → fix the root cause in **Data risk assessments**.

### 📌 Module recap: Purview DSPM for AI
| Step | Key facts |
|---|---|
| **Configure** | **Purview Audit on** + **M365 Copilot licenses** = the **two primary prerequisites**. (Also E5 / Purview Suite.) |
| **Labels** | Not required, but the **highest-value next step**. **Without labels** you only get **pattern-based detection** (credit cards, SSNs). **With labels** you get **full risk prioritization**, **label-scoped remediation**, and **restricting Copilot access by classification** |
| **SharePoint overexposure** | Data risk assessments: sites with sensitive files **ranked by label severity**, then restrict access or add protection |
| **Interaction risks** | Copilot/AI referencing labeled content. Each risk **links to the label policy, the SharePoint location and the DLP config** |
- **Flow of the AI-security learning path:** **Purview DSPM for AI (data risk)** → **Entra Agent ID Conditional Access (agent identity)** → Defender XDR blast radius → MDCA runtime protection → AI Gateway → guardrails → Defender for AI Services → Agent 365.

> 🎯 **Exam focus:**
> - *DSPM only shows credit card / SSN findings, not rich prioritization* → **deploy sensitivity labels**.
> - *The two prerequisites for seeing Copilot interactions* → **Purview Audit** + **M365 Copilot licenses**.

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
| DSPM for AI only shows **pattern-based** detections (credit cards, SSNs), no label-scoped remediation | **Deploy sensitivity labels** |
| DSPM for AI shows sensitive data pasted into external AI, **prevent it** | **Purview DLP policy** (block/warn); DSPM for AI **doesn't block** |
| Which **files** Copilot referenced, and whether sensitive data was in **prompt / response / both** | DSPM for AI **interaction detail pane** (Activity explorer) |
| **File name/size** for a browser paste into ChatGPT | **Not available**: SIT-level only for browser events |
| Activity explorer: user browsed to external AI site | Activity type **AI website visit** |
| Activity explorer: DLP policy matched during AI interaction | Activity type **DLP rule match** |
| **Highest priority** AI interaction risk | **Highly Confidential** content + **external AI site** via browser |
| **No Copilot data** in DSPM for AI | Verify **Purview Audit enabled** + **M365 Copilot licenses** assigned |
| License for **DSPM for AI** | **M365 E5** or **Microsoft Purview Suite** (E3 doesn't qualify) |
| Detect sensitive data pasted into **ChatGPT / Gemini** (external AI sites) | **Device onboarding to Purview** + **Purview browser extension** |
| Quick-start detection of **risky AI prompts/responses** | DSPM for AI **one-click policy: Detect risky AI usage** |
| SharePoint **oversharing** view in DSPM for AI | **Data risk assessments** (weekly, **top 100 sites** by usage) |
| Which **files Copilot referenced** in an interaction (event-level) | DSPM for AI **Activity explorer** |
| **Security Copilot** not showing in DSPM Apps and agents | Configure a **separate collection policy** |
| Stop **Copilot summarizing** labeled content **without changing permissions** | Protect tab → **Restrict access by label** (DLP policy) |
| Exclude a SharePoint site from **Copilot grounding** but keep **human** access | **Restricted Content Discovery** (Restrict all items) |
| Sensitive files on overshared site are **unlabeled** | **Auto-labeling policy** |
| **Stale content** (3+ years no access) expanding oversharing | **Retention policy** |
| Reduce **both human and AI** exposure on overshared site | **Tighten SharePoint site permissions** (security groups) |
| See **anyone-with-the-link / external** sharing on a site | Data risk assessment → **Monitor** tab |
| Which **sensitive data Copilot surfaces** / **SharePoint overexposed** to AI grounding | **Microsoft Purview DSPM for AI** |
| Agent's **creator left the organization** / no accountable person | Agent overview → **Agents without owners** → assign owner |
| Single view of **agent risk signals from Entra, Defender, Purview** | Agent 365 overview → **Agents at risk** card |
| Publisher **added new permissions/connections** after approval | **Revoke approval** (back to pending, re-approve) |
| Agent triggers **DLP alerts**, need time to investigate | **Block** (reversible), then **Unblock** when resolved |
| Pilot agent scope **crept** to far more users than approved | **Narrow user scope** |
| Agent with **zero interactions for 60+ days** | Review for **decommissioning** |
| Interactive agent busy **midnight–6 AM** / huge volume from **few users** | Likely **batch automation**; verify vs approved purpose |
| Agent **usage metrics** (interactions, unique users, 30-day trend) | Agent → **Usage / Activity** tab |
| Approved agent should only be used by **one department/group** | Agent 365 → agent → **Manage access** (user scope) |
| Agent's **Power Platform connectors** must comply with **DLP** | **Power Platform admin center → Data policies** |
| Agent sends data to **external API in non-compliant region** | **Block** (data residency/sovereignty); check **Permissions & data access** |
| Agent has valid purpose but **over-broad permissions / unclear description** | **Return with feedback** |
| Agent **violates org AI policy** / unknown vendor / unjustified mailbox access | **Block** (recorded with reason + admin) |
| Copilot Studio agent's **effective access ceiling** (default auth) | **Signed-in user's permissions** (**OBO**) |
| Agent moves files **SharePoint (business) ↔ Dropbox (non-business)** | **DLP violation** → block until redesigned |
| **Least-privilege** role to **approve/publish/block** agents in Agent 365 | **AI Administrator** |
| View agent registry/activity **without** governance rights | **AI Reader** / Global Reader / **Security Reader** / Security Operator / Security Administrator |
| Approve a **submitted** agent in Agent 365 | **Agents → All agents → Requests → Publish** |
| **Temporarily** stop an agent org-wide during investigation (reversible) | **Block** in Agent 365 Registry |
| **Agents** section missing in M365 admin center | Check **M365 Copilot license** + **AI Admin / Global Admin** role |
| License needed for **Agent 365** management | **Microsoft 365 Copilot** (inherited automatically) |
| **Approve/review agents before use**, central view of active agents & approvers (M365) | **Microsoft Agent 365** in the **Microsoft 365 admin center** |
| Least-privilege role to **view Data and AI security dashboard** | **Security Reader** |
| Data & AI dashboard **data sections empty/partial** | Enable **Defender CSPM + sensitive data discovery**, **Defender for Storage**, **Defender for Databases** |
| Defender for Cloud **AI alerts missing in Microsoft Sentinel** | Check **Microsoft Defender XDR data connector** in Sentinel |
| **Prompt evidence field empty** on AI alert | Enable **prompt evidence** toggle (Environment settings → AI services → Settings) |
| **Recurring false positives** from a specific resource (e.g. pen test) | **Suppression rules** in Defender for Cloud |
| Notify team by email on **jailbreak / data leakage** | Environment settings → subscription → **Email notifications**, threshold **High** |
| Defender for AI Services: **four threat categories** | **Jailbreak, data leakage, credential theft, data poisoning** |
| Defender for AI Services scanning **images/audio**? | **No**: **text tokens only** |
| AI alert shows **masked** prompt text | Default. Enable **Suspicious prompt evidence** ("Enable user prompt evidence") |
| Org policy forbids access to prompt content | Leave **prompt evidence Off**; detection still works |
| **eDiscovery / Insider Risk / Communication Compliance / SIT classification** on Foundry AI interactions | **Data security for AI interactions** + **separate Purview license** |
| Scan **custom models** in **Azure ML registries** for **malware / serialization vulns / secrets** | **AI model security** component |
| Foundry **agent** interactions not showing in Purview | **Not supported yet**: only direct model API calls with user-context Entra token |
| Foundry API calls **without user context**, where visible in Purview? | Only **Purview Audit** + **DSPM for AI Activity Explorer** |
| Engine behind Defender for AI Services detection | **Azure AI Content Safety Prompt Shields** + Microsoft threat intelligence |
| Role required to enable **Defender for AI Services** plan | **Owner or Contributor** on the **subscription** |
| Threat detection at the **model inference layer** (Azure OpenAI / Foundry): jailbreak, data leakage, credential theft | **Defender for AI Services** (scans **text tokens at inference time**) |
| Filter **Defender XDR incidents** to AI threats | Service/detection source → **Microsoft Defender for Cloud → Microsoft Defender for AI Services** |
| XDR path to **prompt text** behind a jailbreak alert | Alert → **Activities → Related activities** → event → **Prompt Suspicious Segment** |
| Alert caused by **authorized test / expected behaviour** | Classification: **Informational, expected activity** |
| Is an AI alert **isolated or part of broader attack**? | **Defender XDR correlated incident** (alerts, assets, timeline) |
| Filter Defender for Cloud alerts to **AI only** | Security alerts → **Product Component Name = AI, AIServices** |
| See the **exact part of the prompt** that triggered an AI alert | **Prompt Suspicious Segment** (Supporting evidence events, preview) |
| **Auto-remediate** AI alert: quarantine, disable creds, notify Teams/ServiceNow | **Take action → Logic App playbook** / automation rule |
| Malware / suspicious access in **model or dataset storage** | **Defender for Storage** |
| Compromised **inference/training container** | **Defender for Containers** |
| **EDR + vuln scanning** on VMs hosting AI | **Defender for Servers** |
| Unauthorized **outbound connections** from AI compute / **anomalous API calls** to AI endpoint | **CWP runtime alert** (Defender for Cloud) |
| Inventory/map of deployed **AI models, datasets, services** | **AI Bill of Materials (AI BOM)**, auto-built by CSPM |
| **Query** which AI workloads & models are in use | **Cloud Security Explorer** → template **AI workloads and models in use** |
| Find **code repos** provisioning Azure OpenAI with **vulnerabilities / exposed API keys** | Cloud Security Explorer → **Generative AI vulnerable code repositories that provision Azure OpenAI** |
| Chain of misconfigs: **internet → storage w/ sensitive data → AI workload** | **Attack path analysis** (Defender CSPM) |
| AI framework/library with known **vulnerabilities** | CSPM recommendation: **update dependencies** |
| Data & AI dashboard: resources by **protection status** (full/partial/none) | **Overview** section |
| Data & AI dashboard: most critical **recommendations / alerts / attack paths** | **Top issues** section |
| Data & AI dashboard: **prompt/response alerts**, exposed **grounding / fine-tuning** resources | **AI closer look** section |
| Data & AI dashboard: **sensitive info types**, internet-exposed **data assets** | **Data closer look** section |
| How a **misconfigured storage account exposes grounding data** | **Attack path** (Top issues → contextual details) |
| Enable Defender **AI workloads plan** | Defender for Cloud → **Environment settings** → subscription → **AI workloads On** (per subscription) |
| See **prompt snippets** that triggered an AI alert in Defender XDR | Enable **Suspicious prompt evidence** (redacted) |
| Classify/protect **AI prompt & response data** with Purview | Enable **Data security for AI interactions** |
| **Attack path analysis** for AI resources | Requires **Defender CSPM** |
| Discover AI workloads in **AWS / GCP** | **Cross-cloud connectors** (multicloud) |
| Defender policy/assessment data not flowing for a subscription | Register the **Microsoft.Security resource provider** |
| AI resource **misconfiguration** (public access, local auth) / **AI posture** | **Defender CSPM** (Defender for Cloud) |
| **Runtime threat detection** on AI workloads | **CWP: AI workloads plan** (Defender for AI Services) |
| Single view of **AI + data security** insights in Defender for Cloud | **Data & AI security dashboard** |
| Guardrail/content filter exists but a **model deployment is unprotected** | Filter **not connected** to that deployment (Connection step) |
| Content filter wizard: where do **Prompt Shields + blocklists** go? | **Input filter** |
| Content filter wizard: where do **protected material + groundedness** go? | **Output filter** |
| Roll out guardrails **without disrupting** users / reduce **false positives** | **Moderate thresholds + annotate first**, review logs, then tighten |
| Precise blocking of **regulated identifiers / code tokens** | **Blocklists** |
| **RAG / knowledge retrieval** workload must stay factual | **Groundedness detection** |
| AI resource **local auth enabled / no diagnostic logs / open network** | **Defender for Cloud recommendations** (platform layer) |
| Blocklist must catch **variations / token-like strings** (e.g. `Admin-Token 9876`) | Blocklist entry type **Regex** |
| Blocklist must catch **one specific phrase exactly** | Blocklist entry type **Exact match** |
| Import **many** restricted terms into a Foundry blocklist | **Bulk add**: **CSV** with **Term** + **Type** columns |
| Block term in **both** user prompts **and** model responses | Apply blocklist to **Input filter AND Output filter** |
| Apply the **same restricted terms** to several workloads | **Reuse** one blocklist across **multiple guardrail configurations** |
| Custom blocklist/filter **not triggering on Try it out page** | **Expected**: Try it out previews **built-in** models only |
| **Org-specific** detection criteria beyond built-in harm categories | **Custom categories** |
| Moderate **images / combined visual + text** inputs | **Image and multimodal filters** |
| Establish a **baseline** of built-in guardrail behavior | **Try it out** (Guardrails + controls workspace) |
| **Indirect prompt injection** (malicious instructions hidden in documents/emails the model reads) | **Prompt Shields** |
| Prompt Shields: **flag but don't block** | **Annotate only** |
| Prompt Shields: **stop the prompt + log it** | **Annotate and block** |
| Content filter **harm categories** | **Violence, Hate, Sexual, Self-harm** (severity slider each) |
| Model outputs **copyrighted text/code** or third-party proprietary content | **Protected material detection** |
| Model **fabricates / hallucinates** facts not supported by source data | **Groundedness detection** (preview) |
| **Test guardrail behavior** with sample prompts before production | **Try it out** in **Guardrails + controls** workspace |
| Service that **powers Foundry guardrails** / moderation models classifying harmful content | **Microsoft (Azure AI) Content Safety** |
| Protect **model behavior** at the **application layer** (prompts in, responses out) | **Foundry guardrails**, not Defender for Cloud or Purview |
| Foundry: filter **harmful content categories** in prompts/responses | **Content filters** (guardrail) |
| Foundry: block **specific custom words/phrases** (e.g. competitor names, internal codenames) | **Blocklists** (guardrail) |
| Foundry: detect **jailbreak / prompt injection** attempts | **Prompt Shields** (guardrail) |
| AI Gateway **never exposed to public internet** (callers in Azure) | **Private endpoint** + restrict to the callers' **VNet** |
| AI Gateway callers **outside Azure / hybrid** | **IP allowlisting** (+ Entra token auth) |
| Gateway authenticates to **backend model deployments** without secrets | **Managed identity on the gateway** + **Cognitive Services User** (scoped) |
| Are **prompts/responses** in APIM diagnostic logs? | **No**: **metadata only** by default |
| Gateway logs for **KQL + alerting** / compliance & IR | **Log Analytics workspace** |
| Gateway logs **long-term retention** | **Storage Account** |
| Gateway logs **streamed to external SIEM/system** | **Event Hubs** |
| KQL table for **APIM gateway requests** | **`GatewayLogs`** |
| Alert on **token spikes / off-hours calls / error rate >5% / near rate limit** | **Azure Monitor alert rules** on gateway logs/metrics |
| Agent makes **hundreds of model calls** for a simple task | **Retry loop** or **prompt injection**, so investigate |
| Caller gets **429 Too Many Requests** from AI Gateway | **TPM rate limit** exceeded (temporary, resets next minute) |
| Caller gets **403 Forbidden** from AI Gateway (was working before) | **Total token quota** for the period **exhausted** |
| AI Gateway with Foundry **public access disabled** / **VNet injection + private endpoint** | **APIM Premium v2** |
| AI Gateway for **dev/test** | **Basic v2** (what "Create new" deploys) |
| AI Gateway for **production** (no private networking needed) | **Standard v2** (or Premium v2) via **Use existing** |
| Where to **create** an AI Gateway | **Foundry portal (ai.azure.com) → Operate > Admin console → AI Gateway → Add AI Gateway** |
| **Existing** Foundry project not going through gateway | Manually **Add project to gateway** (new projects are auto-enabled) |
| Set **TPM limit per project/deployment** | **Operate > Admin → gateway → Token management → + Set limit** |
| Ensure gateway only accepts tokens for **your environment** | Set required **audience** and **issuer** (Entra ID auth provider) |
| Route **internal apps to premium model, external agents to standard** / **A/B test** a new model version | AI Gateway **route policies** (advanced routing) |
| **Compromised Foundry agent** calling models | **Revoke that agent's managed identity role assignment** on the gateway |
| Apps **share one model API key**; need per-app identity & no shared secrets | AI Gateway + **managed identity per app** (**Entra token** auth) |
| **Runaway agent / app** exhausting model capacity | **Token limits per caller/subscription** at the AI Gateway |
| Audit trail: **caller identity, token count, latency** of model calls | APIM **diagnostic logs → Azure Monitor / Log Analytics** |
| **Custom policies / load balancing** for AI Gateway | Configure in **APIM in the Azure portal** (not only Foundry portal) |
| **Control layer** between apps and Foundry models / anyone with endpoint can call model | **AI Gateway** (powered by **Azure API Management**) |
| Track **token consumption** / **rate-limit** / **audit** model calls | **AI Gateway (APIM)** policies + logging |
| Role to **create new APIM** for AI Gateway | **Contributor/Owner** on the resource group |
| Role to use an **existing APIM** for AI Gateway | **API Management Service Contributor** |
| Role on the **Foundry resource** to configure AI Gateway | **Azure AI Account Owner** or **Azure AI Owner** |
| KQL: what a Copilot Studio agent **did at runtime** (tool invocations, data access) | **`CloudAppEvents`** (`Application == "Microsoft Copilot Studio"`) |
| KQL: agent **configuration / tools / knowledge sources / IsBlocked / posture** | **`AIAgentsInfo`** |
| Latest record per agent in KQL | `summarize arg_max(Timestamp, *) by AIAgentId` |
| **Agent maker** checks protection without Defender portal access | Copilot Studio **Agents page → Protection status** column |
| Copilot Studio protection status **"Needs review"** | **Policy violated** or **inadequate authentication** |
| No AI agent alerts right after enabling | **Expected**. Wait for activity or **generate a test interaction** |
| External user tries to **override agent instructions** | Alert: **Prompt injection detected** |
| Agent pulls **large volume** of confidential files | Alert: **Data exfiltration risk** |
| Real-time protection **blocks** but **no alerts/incidents** in XDR | **Microsoft 365 app connector** not connected |
| Permission to connect the **Microsoft 365 app connector** | **Global Admin** or **Security Admin** |
| Real-time protection status **Pending / Disconnected** | **App ID mismatch** or Power Platform config incomplete |
| Power Platform side of Copilot Studio agent protection | **PPAC → Security > Threat Protection → Microsoft Defender – Copilot Studio AI Agents** |
| Who **creates the Entra app / App ID** for the proxy | **Power Platform Administrator** |
| Protection on but **current user conversation** not protected | Only **new sessions** are protected, so **restart the session** |
| How long until the AI agent inventory starts populating | **~30 minutes** after enabling Security for AI Agents |
| **Block** suspicious Copilot Studio **tool invocation before execution** | **MDCA real-time runtime protection** (acts as a **proxy**) |
| **Jailbreak / data poisoning / credential theft** detection for **Azure OpenAI** or AI Model Inference | **Defender for AI Services** plan in **Defender for Cloud** (Environment settings → AI services On) |
| Turn on Copilot Studio agent protection | Defender portal → **System > Settings > Security for AI** |
| Find a Copilot Studio agent's **Entra Agent ID** | Copilot Studio → **Settings > Advanced > Metadata** |
| App ID exchanged between Defender portal & Power Platform admin center | **Proxy App ID**: the trusted inspection connection, **not** the agent's identity |
| Copilot Studio agent created **before 18 Mar 2026** | **Legacy app registration** (no Agent ID governance) |
| External user abusing an **internal-only tool** via agent / extracting **system instructions** | **MDCA real-time protection** blocks it |
| Hunt agent activity with **KQL** | **Advanced Hunting** in Defender XDR |
| **Runtime / real-time** inspection & **interception** of Copilot Studio agent interactions (e.g. **prompt injection**) | **Defender for Cloud Apps: real-time protection for AI agents** |
| Copilot Studio agent protection setup spans which consoles? | **Defender portal + Power Platform admin center** |
| Where to **verify** agent protection outputs | Defender XDR **AI agent inventory, alerts, Advanced Hunting** |
| License for Copilot Studio agent runtime protection | **Agent 365**, OR **MDCA + Copilot Studio** |
| Three components of agent **blast radius** | **Permissions + Knowledge sources + Blueprint actions** |
| Assess blast radius for **Foundry / Bedrock / Vertex** agent | **AI agent inventory → agent detail pane** |
| Assess blast radius for **Copilot Studio** agent | **Advanced Hunting → Queries tab → AI Agents** (prebuilt queries) |
| **Visualize** attacker's route from agent to sensitive data | **Attack path → "View on map"** |
| Limit agent to **specific SharePoint sites** only | **`Sites.Selected`** (instead of `Sites.FullControl.All`) |
| Limit agent's **user profile** access | **`User.ReadBasic.All`** (instead of `User.Read.All`) |
| Agent can **exfiltrate via email**, keep function but add oversight | Replace send-email with a **create-task / human-review** action |
| Agent grounded on a **restricted site** = … | Effectively **bypasses the site's access controls** |
| AI agent inventory location | Defender portal → **Assets > AI Agents** |
| Enable **Copilot Studio** agent discovery | Defender: **System > Settings > Security for AI** + Power Platform admin center: **Security > Threat Protection** |
| Discover **Foundry / AWS Bedrock / GCP Vertex AI** agents | **Foundry integration in Defender for Cloud** |
| License for agent discovery/posture/threat detection (Copilot Studio & Foundry) | **Microsoft Agent 365-eligible license** (MDCA/Defender for Cloud alone isn't enough) |
| Agent uses **API key / external IdP**, won't show in inventory | **Manual inventory/monitoring** (no Entra identity = no auto-discovery) |
| Alert references a **service principal**, find the agent | Search the AI agent inventory by **object ID** |
| Identity to assess for an agent's risk | **Associated identities** (service principal / managed identity) |
| **Discover all AI agents** in the environment (incl. unknown/business-built) | **Defender XDR → AI agent inventory** |
| **Blast radius** of a compromised agent / **prioritize** riskiest agents | **Defender XDR** (permissions, knowledge sources, blueprint) |
| **Attack paths** from an agent identity | **Defender XDR** attack path analysis |
| Remove agent access but **keep the identity record for audit** / **reversible** | **CA block** or **disable** the agent, **not delete** |
| **Prevent agent creation** / reduce **agent sprawl** | **Per-product creation blocks** (or **Agent 365 → Allowed agent types**), not CA |
| **Central** agent governance across all products / **security templates** applied to every new agent | **Microsoft Agent 365** (M365 admin center → Agents → Settings) |
| Reassign agents when the **creator leaves** the org | **Agent 365 agent management rules** |
| Copilot Studio: users can **build but not publish/deploy** agents | **DLP policies** (Power Platform) |
| Copilot Studio: stop users **creating** agents | Block **free trials** + remove **Power Platform Administrator** role |
| Foundry: who can **create agents** in a project | **Azure AI User** role (projects: **Azure AI Account Owner**) |
| Security Copilot: block agents completely | Remove workspace **Owner/Contributor**; extreme option: delete **SCU capacity** |
| **Compromised agent**, stop it **immediately** | **Disable** the agent identity (Entra → Agents → Agent identities) |
| Disable **all agents derived from one template** | Disable the **agent identity blueprint** |
| Role needed to disable agent identities | **Agent ID Administrator** |
| **Detect new agent created** / audit agent lifecycle | Audit logs: **ApplicationManagement → Add service principal**, check **`agentType`** |
| Is this service principal an agent? | **`agentType` ≠ `notAgentic`** |
| Agent **user account** created, which audit event? | **Create user** |
| **Automated alert** when an agent is created | **Sentinel playbook / Logic App** |
| **Periodically validate** agents are still needed | **Access reviews**, scope = **Workload identities** |
| **Test / validate** a CA policy **without impacting** agents/users | **Report-only mode** → review **Sign-in logs** |
| **Decommission / stop** a single unused agent | CA targeting the **specific agent service principal (object ID)** + **Block access** |
| **Secure-by-default** / only approved agents may authenticate | **All agent identities → Block**, then exclude/allow approved agents |
| Limit how long a **stolen agent token** is usable | Session control: **Sign-in frequency** |
| License needed for **CA on agents** | **Entra ID P1** or **M365 E3** |
| License needed for **Entra Agent ID** features | **Microsoft Agent 365** or **M365 E5** |
| Agent **risk/suspicious sign-in** signal comes from… | **Microsoft Entra ID Protection** |
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
- **Report-only vs On:** report-only **logs only** and never blocks. Always use it first for agent block policies.
- **Grant access vs Grant with controls:** plain *Grant* adds no security. For agents, the meaningful control is **Require compliant network**.
- **CA licensing vs Agent ID licensing:** CA = **P1/E3**. Agent ID = **Agent 365/E5**. Don't mix them up.
- **CA block vs disabling an agent vs blocking creation:** CA = blocks **auth** for existing and new agents, but agents still get **created**. Disable = **one agent**, no CA needed. Creation block = stops agents existing **at all**.
- **DLP vs RBAC/licensing in Copilot Studio:** DLP = **can build, can't publish**. RBAC/licensing = **can't create at all**.
- **Agent identity creation vs agent user account creation in audit logs:** **Add service principal** vs **Create user**.
- **Disabling via the Entra admin center vs the product admin UI:** same result. The product UI adds **dependency warnings**.
- **Block/disable vs delete an agent:** block/disable = **reversible + audit trail kept**. Delete = gone. The exam favours block/disable.
- **Entra Agent ID vs Defender XDR for agents:** Entra = **control** access (CA, disable, access reviews). Defender XDR = **discover and assess risk** (inventory, blast radius, attack paths).
- **Inactive vs disabled vs removed agent:** inactive = not invoked **but still has permissions**. Disabled = off **but keeps identity and permissions**. Only removal clears them. A dormant agent with broad rights is still **risky**.
- **Discovered vs not discovered:** **managed identity or Entra service principal = discovered** (even custom code in ACI). **API key or external IdP = invisible**.
- **Enabling Copilot Studio discovery vs Foundry/Bedrock/Vertex discovery:** Copilot Studio = **Defender XDR setting + Power Platform admin**. Foundry/Bedrock/Vertex = **Defender for Cloud** integration.
- **Blast radius vs attack path:** blast radius = **static** scope of what's reachable. Attack path = **dynamic**, step-by-step chain to a target.
- **Application vs delegated permissions:** application = **tenant-wide, no user context**, so a **bigger** blast radius. Delegated = limited by the user's own access.
- **Foundry agent posture vs Copilot Studio agent posture:** Foundry = **detail pane in the inventory**. Copilot Studio = inventory is **discovery only**, use **Advanced Hunting**.
- **Prioritizing attack paths:** by **criticality of the target**, not the **number of steps**.
- **Audit & alerts vs real-time protection (MDCA):** audit & alerts = **after the fact**, you see violations. Real-time protection = **inspects and intercepts during the interaction**.
- **MDCA AI agent protection vs Defender for AI Services:** MDCA = **Copilot Studio** sessions/tools (Defender portal, Security for AI). Defender for AI Services = **Azure OpenAI / AI Model Inference** (Defender for Cloud plan, per subscription). **Enabling one doesn't enable the other.**
- **Proxy App ID vs Entra Agent ID:** proxy App ID = the **inspection connection** between Defender and Power Platform. Agent ID = the **agent's own identity** (service principal, Agent subtype).
- **Entra Agent ID vs legacy app registration:** Agent ID gets **per-agent CA, Entra audit and lifecycle governance**. Legacy app registrations (agents before 18 Mar 2026) don't.
- **Defender URL vs App ID (direction of handoff):** **URL goes Defender → Power Platform admin. App ID goes Power Platform admin → Defender.**
- **M365 connector missing vs real-time protection off:** connector missing = **blocking still works, but you get no alerts**. Protection off = **no blocking**.
- **`CloudAppEvents` vs `AIAgentsInfo`:** CloudAppEvents = **runtime activity** (what it DID). AIAgentsInfo = **configuration/posture** (how it's BUILT).
- **Protection status in Copilot Studio vs Connected status in Defender:** Copilot Studio column = **maker-facing health** (Protected / Needs review / Unknown). Defender inventory = **proxy connection status** (Connected / Disconnected).
- **AI Gateway (APIM) vs Defender for AI Services:** AI Gateway = **preventive control in front of the model** (auth, rate/token limits, routing, logging). Defender for AI Services = **threat detection** on traffic (jailbreak, leakage alerts).
- **Shared API key vs Entra token per managed identity (AI Gateway):** shared key = **no attribution and one compromise exposes everything**. Entra token per managed identity = **per-caller auth, quotas and audit**.
- **429 vs 403 at the AI Gateway:** 429 = **per-minute TPM rate limit** (retry shortly). 403 = **total quota for the period used up** (won't recover until the period resets).
- **APIM Basic v2 vs Standard v2 vs Premium v2:** Basic v2 = **dev/test** (default for "Create new"). Standard v2 = **production**. Premium v2 = **production + VNet injection / private endpoint** (Foundry public access disabled).
- **Gateway scope vs project scope:** the gateway is enabled at the **Foundry resource** level. **Token limits** are set **per project per deployment**.
- **API key vs Entra token auth (gateway):** API key = simple, needs **storage + rotation**. Entra token = **no stored secrets**, validated on **audience/issuer/signature/expiry**, and the **production choice**.
- **Caller identity vs gateway identity:** caller managed identity = **app → gateway** (Entra token). Gateway managed identity = **gateway → model backend** (Cognitive Services User).
- **Log Analytics vs Storage vs Event Hubs (diagnostic destinations):** **query/alert** vs **archive** vs **stream out**.
- **Private endpoint vs IP allowlist:** private endpoint = callers **inside Azure VNets**. IP allowlist = callers **outside Azure/hybrid** with known IPs.
- **`GatewayLogs` vs `CloudAppEvents`:** GatewayLogs = **APIM/AI Gateway** traffic in **Log Analytics**. CloudAppEvents = **Copilot Studio** agent activity in **Defender XDR Advanced Hunting**.
- **Foundry guardrails vs AI Gateway vs Defender for AI Services:** guardrails = **content safety on prompts and responses** (filters, blocklists, Prompt Shields). AI Gateway = **access control** (auth, token limits, routing, logging). Defender for AI Services = **threat detection and alerts** on AI traffic.
- **Guardrails vs Defender for Cloud vs Purview (AI):** guardrails = **model interaction layer** (prompt/response safety). Defender for Cloud = **workload/infrastructure threats**. Purview = **data classification and governance**. They complement each other.
- **Blocklist vs protected material detection:** blocklist = **your own exact terms** (codenames, IDs). Protected material = **third-party copyrighted or proprietary content** (code, text).
- **Prompt Shields vs content filters:** Prompt Shields = **manipulation/attack** intent (jailbreak, injection). Content filters = **harmful subject matter** (violence, hate, sexual, self-harm).
- **Annotate only vs Annotate and block:** annotate only = **flag via metadata, allowed through**. Annotate and block = **stopped + logged**.
- **Groundedness vs protected material:** groundedness = **is it true to the source?** Protected material = **is it someone else's IP?**
- **Try it out vs a deployed guardrail:** Try it out = **built-in models only**, a sandbox **baseline**. Deployed guardrail = **built-in + your custom filters and blocklists** enforced on live traffic.
- **Exact match vs Regex (blocklist):** exact = that **literal phrase only**. Regex = **patterns, variations, token-like strings**.
- **Input filter vs Output filter:** input = **stops the user submitting** it. Output = **stops the model returning** it. Use both for full coverage.
- **Guardrails vs Defender for Cloud recommendations for AI:** guardrails = **AI layer** (prompt/response content). Defender for Cloud = **Azure platform layer** (diagnostic logs, network exposure, local auth).
- **"Blocking level" vs "severity threshold":** a higher *blocking level* = stricter. A lower *severity threshold* = stricter. **Read the question's wording.**
- **CSPM vs CWP (for AI):** CSPM = **posture/configuration**, proactive (recommendations, attack paths). CWP = **runtime threat detection**, reactive (alerts on jailbreak, leakage, credential theft).
- **Suspicious prompt evidence vs Data security for AI interactions:** prompt evidence = **redacted prompt snippets in XDR alerts** (investigation). Data security for AI interactions = **Purview classification/protection** of prompts and responses (compliance).
- **Data closer look vs AI closer look:** Data = **sensitive data discovery, info types, exposed data stores**. AI = **AI services, prompt/response alerts, exposed grounding/fine-tuning resources**.
- **Cloud Security Explorer vs attack path analysis:** Explorer = **you run queries/templates** to find resources (e.g. AI in use). Attack paths = **Defender auto-chains misconfigurations** into exploitable routes.
- **AI BOM vs AI agent inventory:** AI BOM = **Defender for Cloud CSPM** map of models, datasets and services. AI agent inventory = **Defender XDR** (Assets > AI Agents) list of **agents**.
- **Suspicious prompt evidence (setting) vs Prompt Suspicious Segment (alert field):** the setting is the **extension you enable** in the AI plan. The segment is **what you see** in the alert's supporting evidence.
- **Automation rule vs Logic App playbook (Defender for Cloud):** automation rule = **triggers/routes** alerts automatically. Playbook = **runs remediation actions** (quarantine, disable creds, notify).
- **Defender for Cloud alert vs Defender XDR incident:** Defender for Cloud = **single AI workload alert**. XDR incident = **correlated** with identity, endpoint and data signals (the broader attack story).
- **Incident classification:** True positive = **real threat**. Informational, expected activity = **real but benign** (e.g. a test). False positive = **detection was wrong**.
- **Defender for AI Services components:** prompt evidence = **see the prompt in alerts** (SOC). Data security for AI interactions = **Purview compliance** (license needed). AI model security = **scan custom ML-registry models**.
- **Foundry guardrail Prompt Shields vs Defender for AI Services:** guardrail = **blocks/annotates** in the model pipeline. Defender for AI Services = **uses Prompt Shields signals to raise SOC alerts** (detection, not blocking).
- **Suppression rule vs classifying False positive:** classification = a **one-off label** on that alert. Suppression rule = **auto-suppresses future recurring** alerts matching the criteria.
- **Agent 365 (M365 admin center) vs Entra Agent ID vs Defender XDR AI agent inventory:** Agent 365 = **governance** (approval, usage policies, accountability). Entra Agent ID = **identity controls** (CA, disable, access reviews). Defender XDR inventory = **security risk** (discovery, blast radius, attack paths).
- **AI Administrator vs Security Administrator (Agent 365):** AI Admin = **approve, block, scope, ownership**. Security Admin = **view only**.
- **Block vs Delete vs Reject (Agent 365):** Reject = **decline a pending request**. Block = **hide an approved agent from all users, reversible**. Delete = **remove permanently**.
- **Approve (Publish) vs user scope:** Publish = the agent is **available** in the tenant. User scope = **who** can actually use it. Approval alone ≠ access for all.
- **Return with feedback vs Block (Agent 365):** return = **fixable**, resubmit later. Block = **policy violation**, stays blocked **even if resubmitted**, with the reason logged.
- **Block vs Revoke approval vs Narrow scope (Agent 365, post-approval):** Block = **everyone loses access now** (reversible, for investigations). Revoke = **back to pending**, needs fresh approval (agent **changed**). Narrow scope = **fewer users**, the agent stays approved.
- **Agents at risk vs Agents without owners:** at risk = **cross-platform risk signals** (Entra, Defender, Purview). Without owners = **accountability gap** (the creator left).
- **Purview DSPM for AI vs Defender for Cloud Data & AI dashboard:** DSPM for AI (**Purview**) = **M365 data exposure** through Copilot/AI apps and **SharePoint oversharing**. Data & AI dashboard (**Defender for Cloud**) = **Azure AI resource** posture, protection coverage and threat alerts.
- **Restrict access by label vs Restricted Content Discovery vs tightening permissions:** by label = **DLP blocks Copilot summarizing labeled items** (permissions untouched). RCD = **the whole site is hidden from Copilot grounding** (humans keep access). Permissions = **cuts human AND AI** access.
- **DSPM for AI vs DLP:** DSPM = **understand what AI can access and reference** (posture/visibility). DLP = **prevent exfiltration** (enforcement). DSPM's one-click policies *create* DLP/CC rules.
- **Data risk assessments vs Activity explorer:** assessments = **SharePoint site-level overexposure** (weekly). Activity explorer = **individual AI interaction events** (who, what SIT, which files).
- **DSPM for AI (visibility) vs DLP (enforcement) for AI:** DSPM **shows** risky interactions but **never blocks**. DLP **blocks or warns** (e.g. before pasting into ChatGPT or emailing a Copilot summary externally).
- **Prompt risk vs response risk vs browser risk:** prompt = the user **put** sensitive data in. Response = Copilot **pulled** labeled files in. Browser = pasted into an **external** AI site.
