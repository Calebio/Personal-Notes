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
