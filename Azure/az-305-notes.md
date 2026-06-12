# AZ-305 Exam Notes
> Key decision frameworks, buzz words, and exam triggers for the AZ-305 exam.

---

## 1. Compute — Containers & Apps

| Scenario | Service |
|---|---|
| Full OS control, custom software, lift & shift | Azure Virtual Machines |
| Traditional web app, PaaS, minimal code change | Azure App Service |
| Serverless, event-driven, short execution, scale to zero (code) | Azure Functions |
| Containerized, scale to zero, HTTP/event triggered, no K8s overhead | Azure Container Apps |
| Containers, full Kubernetes control, complex orchestration | AKS |
| Single container, short-lived, no orchestration | Azure Container Instances (ACI) |
| Large-scale parallel/HPC, job scheduling across thousands of VMs | Azure Batch |

**Key Triggers:**
- "No infrastructure management" + containers → Container Apps
- "Full Kubernetes control" / Helm / custom ingress → AKS
- "Scale to zero" + code + pay-per-execution → Functions consumption plan
- "Scale to zero" + containers → Container Apps
- "Custom OS", "install software", "registry", "kernel drivers" → VMs
- "HPC", "parallel jobs", "thousands of nodes" → Azure Batch
- "Queue/event triggered" + container scaling → Container Apps with KEDA (not ACI)
- "Short-lived, single container, no orchestration" → ACI
- "Continuous 24/7 GPU workload" → AKS GPU node pools (not ACI)
- "Stateful orchestration", "human approval", "wait days" → Durable Functions (not Logic Apps)

**Easy Confusions:**
- **ACI vs Container Apps** — ACI = single short-lived, no auto-scaling. Container Apps = event-driven via KEDA, scale to zero
- **Functions vs Container Apps (billing)** — Functions = pay per execution only. Container Apps = bills on vCPU/memory seconds while running
- **ACI GPU vs AKS GPU** — ACI GPU = short on-demand tasks. AKS GPU = persistent 24/7 continuous workloads
- **Logic Apps vs Durable Functions** — Durable Functions = stateful, supports human approval & long waits. Logic Apps = low-code, separate service
- **App Service vs VMs** — App Service handles most IIS apps. VMs needed for kernel drivers, registry access, or custom Windows features

---

## 2. Messaging & Events

| Scenario | Service |
|---|---|
| Enterprise messaging, exactly-once, ordering, large messages (up to 100MB) | Azure Service Bus |
| High-throughput event streaming, telemetry, logs, Kafka compatible | Azure Event Hubs |
| Event routing between Azure services, reactive/pub-sub | Azure Event Grid |
| Simple queue, massive volume, cheap, no ordering (max 64KB) | Azure Queue Storage |

**Key Triggers:**
- "Exactly once" or "duplicate detection" → Service Bus
- "Millions of events per second" or "telemetry" → Event Hubs
- "Trigger a function when a blob is created" → Event Grid
- "Topics and subscriptions" → Service Bus
- "Kafka compatible" → Event Hubs
- "Simple, cheap queue, no ordering" → Queue Storage

---

## 3. Databases

| Scenario | Service |
|---|---|
| Relational, traditional SQL workloads, PaaS | Azure SQL Database |
| Lift & shift SQL Server, needs SQL Agent, CLR, cross-db queries | SQL Managed Instance |
| Global distribution, multi-model, single-digit ms latency, multi-write regions | Azure Cosmos DB |
| Petabyte-scale analytics, high-concurrency SQL, data warehouse | Azure Synapse Analytics |
| Data engineering, ML, Spark, notebook workflows, streaming | Azure Databricks |
| Massive reads, scale beyond 4TB, unpredictable scale | Azure SQL Hyperscale |
| Raw file storage for analytics pipelines, hierarchical namespace | Data Lake Storage Gen2 |

**Synapse vs Databricks (most common confusion):**

| | Synapse Analytics | Databricks |
|---|---|---|
| Primary use | SQL analytics, data warehousing | Spark, ML, data engineering |
| Audience | SQL analysts, BI teams | Data engineers, data scientists |
| Exam trigger | "High-concurrency SQL" | "Spark" or "ML" |

**Key Triggers:**
- "Multi-region writes" or "global low latency" → Cosmos DB
- "Petabyte SQL analytics" + "concurrency" → Synapse
- "Spark" or "ML pipelines" → Databricks
- "Lift & shift SQL Server" with minimal changes → SQL MI
- "99.995% SLA" + "zone redundant" → SQL Business Critical

---

## 4. Big Data & Analytics

| Scenario | Service |
|---|---|
| Petabyte-scale SQL analytics, high concurrency | Synapse Analytics |
| Spark-based data engineering, ML, notebook workflows | Databricks |
| Raw file storage, hierarchical namespace | Data Lake Storage Gen2 |
| High-volume streaming data, Kafka compatible | Event Hubs |
| Orchestrate and schedule pipelines, ETL/ELT | Azure Data Factory |
| Real-time stream processing, SQL queries on streams | Azure Stream Analytics |
| Business intelligence, dashboards | Power BI |

**Common Architecture Combos:**
- Event Hubs → Stream Analytics → Synapse *(ingest, process, warehouse)*
- Data Factory → Data Lake Gen2 → Databricks *(ETL, store, transform)*
- Event Hubs → Stream Analytics → Power BI *(real-time dashboard)*

---

## 5. Disaster Recovery & Backup

| Scenario | Service |
|---|---|
| Replicate VMs/VMware/Hyper-V to Azure, meet RTO/RPO | Azure Site Recovery (ASR) |
| Back up VMs, SQL, files, blobs — point-in-time restore | Azure Backup |
| Database-level geo-replication with readable secondary | Active Geo-Replication |
| Automatic DB failover, no connection string change | Auto-Failover Groups |
| Archive/retain backup data long term, cross-region | Recovery Services Vault |

**Key Triggers:**
- "RTO/RPO" + "replication" + on-prem or Azure VMs → ASR
- "Point-in-time restore" or "backup retention" → Azure Backup
- "Readable secondary" + SQL → Active Geo-Replication
- "Automatic failover" + "no connection string change" → Auto-Failover Groups
- "VMware DR to Azure" → ASR

---

## 6. Networking — Load Balancing & Routing

| Scenario | Service |
|---|---|
| Global HTTP/S load balancing, WAF, SSL offload, nearest healthy endpoint | Azure Front Door |
| Regional HTTP/S load balancing, WAF, path-based routing | Azure Application Gateway |
| Global DNS-based routing, non-HTTP, failover/performance routing | Azure Traffic Manager |
| Regional L4 TCP/UDP load balancing, non-HTTP | Azure Load Balancer |
| Centralized hub with managed routing, firewall, VPN/ER at scale | Azure Virtual WAN |
| Centralized shared services (firewall, DNS, ER) with spoke VNets | Hub & Spoke + VNet Peering |

**Key Triggers:**
- "WAF" + "global" + "nearest endpoint" → Front Door
- "WAF" + "regional" + "path-based routing" → App Gateway
- "Non-HTTP" + "global routing" → Traffic Manager
- "TCP/UDP" + "regional" → Load Balancer
- "Microsoft-managed hub" + "large scale" → Virtual WAN

---

## 7. Identity

| Scenario | Service |
|---|---|
| Azure resource authenticates to another Azure resource, no credentials | Managed Identity |
| External/consumer users, social login, custom branding | Azure AD B2C |
| Partner/guest users access your Azure resources | Azure AD B2B |
| Sync on-prem AD without storing password hashes in Azure AD | Pass-through Authentication (PTA) |
| Sync on-prem AD with password hashes stored in Azure AD | Password Hash Sync (PHS) |
| Lift & shift on-prem AD DS, LDAP/Kerberos required | Azure AD Domain Services |

**Key Triggers:**
- "Without storing credentials" + Azure-to-Azure → Managed Identity (not Key Vault)
- "Consumer users" + "social login" + "custom branding" → B2C
- "Partner/guest" + "external org" → B2B
- "No password hashes in Azure AD" → PTA

### Privileged Access Management — JIT & PIM

**Core concepts:**
- **Standing privilege** — permanently assigned admin roles left active after the work is done (e.g., an unused Global Administrator role lingering after a migration). Credentials are *always* discoverable and exploitable — a persistent attack surface that raises breach risk and business impact
- **Blast radius** — the scope of systems and data an attacker controls after a compromise; permanent high-privilege roles maximize it
- **JIT (Just-In-Time) access** — implemented via **Microsoft Entra PIM (Privileged Identity Management)**; makes elevated roles **eligible** (not permanently active), **time-limited**, and **auditable**
- PIM applies the same model to **both Microsoft Entra ID roles and Azure resource roles**

**Exam trigger:** "Reduce standing admin privileges" / "limit blast radius" / "time-bound elevated access" → Microsoft Entra PIM (JIT access)

### Eligible vs Active Assignments

| Assignment Type | What It Means | Example |
|---|---|---|
| **Eligible** | Holds the entitlement but role is NOT active until requested/activated (true JIT) | "Right to request admin access" — must activate to use |
| **Active** | Role is granted immediately, optionally time-limited | Break-glass admin account that auto-expires at a set time |

**Design rule:** make **eligible** the default; reserve **active** assignments for break-glass or tightly scoped emergency needs only — and monitor them closely.

### Activation Controls (the checkpoints before elevation)

| Control | What It Does |
|---|---|
| **MFA verification** | Confirms identity with a second factor (text code, authenticator app) before granting the role |
| **Justification** | Requires the requester to record *why* they need access — creates an auditable rationale |
| **Approval** | Requires a delegated approver to confirm the request before activation proceeds |
| **Activation duration** | Limits the live session to **1–24 hours**, shrinking the exposure window |

- **Match controls to risk:** for extremely sensitive roles, combine **approval + MFA**; for operational speed on lower-risk roles, **MFA + short duration** is often enough

### Critical Roles That Should Always Be JIT-Only

Treat these tenant-scoped roles as **eligible-only**, never permanently active — their blast radius is too large:
- Global Administrator
- Privileged Role Administrator
- Security Administrator
- Exchange Administrator
- Application Administrator
- Authentication Policy Administrator

### Break-Glass Accounts (Emergency Safeguard)
- A **break-glass account** is an emergency admin account kept **offline** (e.g., printed credentials in a safe) for use only when normal access is blocked
- **Before converting Global Administrator to eligible-only**, confirm you have **at least two break-glass accounts** with **permanent active assignments**
- Approval gates can delay activation during an incident — break-glass accounts bypass that delay for true emergencies
- Pair break-glass accounts with clear **incident runbooks** so they're used correctly under pressure

### How Activation Works (the flow)
1. User requests activation of an eligible role
2. User satisfies the configured gates — MFA, justification, and (if required) **approval** from a delegated approver
3. Role becomes **active** for the configured duration (commonly 1–2 hours for critical roles)
4. When the duration elapses, the session **automatically reverts to eligible** — no manual cleanup required

**Design rule:** role settings (MFA, justification, approval, max duration) apply to **every eligible user of that role** — configure them per privilege tier, with the strictest gates (MFA + justification + approval + 1–2 hr max duration) on the most critical roles.

### Microsoft Entra Roles vs Azure Resource Roles

| | Microsoft Entra (directory) Roles | Azure Resource Roles |
|---|---|---|
| Scope | Tenant-wide — identities, directory configuration | Subscription, resource group, or single resource |
| Examples | Global Administrator, Security Administrator | Owner, Contributor, custom RBAC roles |
| PIM coverage | Secured by PIM | Also secured by PIM — but the assignment path differs |

**Design best practice:** PIM secures both planes, but don't assume the workflow is identical — verify the assignment path (directory vs resource scope) when designing JIT for each.

### Role Hygiene Checklist
- Review your admin roster and **flag any permanent high-privilege assignments**
- Convert flagged roles to **Eligible** in Identity Governance
- Enable **required MFA + justification** on critical roles; add **required approval** for the most sensitive (e.g., Global Administrator)
- Enforce **short max durations (1–2 hours)** for critical roles
- Verify **at least two offline break-glass accounts** exist with permanent active assignments
- Pilot it: take one risky role, make it eligible, test an activation cycle end-to-end, and confirm it reverts automatically when time expires

### PIM for Azure Resource Roles (Control Plane)

PIM protects **Azure resource roles** (subscriptions, resource groups, resources) separately from Microsoft Entra directory roles. The goal: limit blast radius if an **Owner** is compromised — a subscription Owner controls every nested resource (e.g., a compromised Key Vault can expose credentials across many systems).

**RBAC scope hierarchy** (broader assignments inherit downward):
```
Management Group → Subscription → Resource Group → Resource
```
- Choose the **narrowest scope** that does the job — don't assign at subscription level when resource-group level suffices

**Risk-tier model — match controls to risk:**

| Tier | Examples | Controls |
|---|---|---|
| Critical | Key Vault, Azure AI services (secrets/model data = broad downstream exposure) | MFA + justification + approval + shortest duration |
| High | Resources with sensitive data or wide blast radius | MFA + justification + approval, moderate duration |
| Standard | Lower-sensitivity resources | MFA + justification, longer duration acceptable |

- Decide tier by: **data sensitivity, blast radius, regulatory exposure, and reversibility of damage**
- The same role (e.g., Contributor) can be configured with **different settings at different scopes** — keep production tight, dev nimble

**Assignment flow (PIM > Azure resources):**
1. Open **PIM > Azure resources**, pick the target subscription or resource group
2. Open **Roles** → **Add assignments**
3. Choose the role, set **Assignment type = Eligible**, select the member, configure duration, **Assign**
4. To use it: go to **My roles → Azure resources tab → Activate**

- **PIM for Groups** — make group membership itself eligible, reducing per-user assignment overhead at scale

### Scaling JIT — PIM for Groups

**The scale problem:** direct eligible assignments (one per user, per role) create many independent settings. When policy changes (e.g., shortening activation duration), you must update every assignment individually — causing **configuration drift** and uneven enforcement.

**How PIM for Groups works:**
- Shifts the *eligible resource* from an individual role to **group membership**
- You assign roles to the **group**; users become eligible to **join** the group
- When a member activates membership, they **inherit every role the group holds**
- When activation expires, **access is automatically revoked** — no manual cleanup
- Example: a 20-engineer incident-response team activates Key Vault Contributor by joining one eligible group instead of managing 20 separate assignments

**Role-assignable groups (use these, not regular groups):**

| | Role-Assignable Group | Regular Group |
|---|---|---|
| Who can change membership | Requires elevated privilege (e.g., Privileged Role Administrator) | Group owners can edit freely |
| PIM workflow integrity | Membership changes are gated and logged | Can bypass PIM approval entirely — weakens JIT |
| Audit trail | Single membership-activation log auditors can trust | Inconsistent / easy to circumvent |

**Why group-level settings reduce drift:**
- Define role settings (MFA, justification, duration) **once at the group level** — every eligible member inherits them instantly
- Example: changing a 4-hour activation window to 1 hour at the group level applies to all members immediately, with no per-person updates

**When to use group-based JIT vs. direct assignment:**
- **Direct eligible assignments** still belong in the toolbox for **single, one-off cases**
- **Favor a PIM-managed group** when **two or more** of these are true: many users share the same access, the access need **recurs**, or the permissions span **multiple roles**
- Classic fit: a security operations group of analysts who all need the same recurring elevated rights

**Design Best Practices — PIM for Groups:**
- **Use role-assignable groups**, never regular groups, for PIM-managed access — this keeps the privilege bar high on membership changes
- **Bundle a recurring access pattern into one group** with tightly scoped owners, rather than managing many individual assignments
- **Define settings once at the group level** to enforce uniform policy and shrink the audit surface dramatically
- **Start small** — migrate one recurring access pattern to group-based JIT, test a policy change, and watch it ripple consistently across the team
- This scales naturally alongside the Critical/High/Standard risk-tier model — apply group-based JIT especially where Critical-tier resources have recurring shared access needs

### JIT for AI Workloads, Agents & Applications — The Two-Track Model

Azure OpenAI and Azure Machine Learning **control planes are Critical-tier risk** — a compromised contributor can enumerate models, redirect endpoints, exfiltrate training data/weights, and the exploitation window is unlimited if access is standing. Treat these control-plane roles like production-level privileges.

**The core constraint:** managed identities and service principals **cannot perform interactive activation** (no MFA, no human approval step) — so PIM literally cannot govern them. This forces a clean split:

| Track | Identity Type | Access Model | Why |
|---|---|---|---|
| **Humans** (engineers who configure/deploy) | User accounts | **PIM-eligible, role-assignable group** — activate via MFA/approval, time-limited | They can complete interactive activation; standing access = unacceptable risk |
| **Workloads** (agents, pipelines, services) | Managed identities / service principals | **Permanent, narrowly-scoped RBAC** | Cannot activate interactively — least privilege enforced by *scope*, not *time* |

**Identity types defined:**
- **Managed identity** — an Entra ID identity bound to an Azure resource, fully managed by the platform (auto-rotates credentials, no human sign-in needed — e.g., a VM requesting tokens)
- **Service principal** — an app identity authenticating with a secret or certificate (e.g., an external tool with a client secret) — requires active credential rotation plans

**Design Best Practices — AI Workload Access:**
- **Put engineers in one PIM-eligible role-assignable group** for AI control-plane tasks (Cognitive Services, Azure ML workspace roles) — one policy then controls duration, approvers, and audit logs for everyone
- **Never give workload identities PIM-eligible roles** — they can't activate them; give them **permanent, least-privilege RBAC scoped to the specific resource or container**
  - Example: an agent writing inference outputs to Blob Storage gets **Storage Blob Data Contributor on that one container** — not a broad or eligible role
- **Prefer managed identities over service principals** where possible — the platform handles credential rotation automatically, removing a whole class of operational risk
- If you must use service principals, maintain an **active rotation plan** for secrets/certificates
- **Audit your assignments by asking**: "is this for a workload or a human?" — workloads get permanent narrow scope, humans get time-limited PIM activation
- Start small: pick one agent or workload identity, narrow its storage/resource scope to exactly what it needs, and use that as your template

**Design Best Practices — PIM for Azure Resources:**
- **Inventory high-risk resources first**, then create eligible assignments at the **narrowest scope** that works
- **Approval-gate critical resources** (Key Vaults, AI services) with MFA, justification, and short durations
- **Tailor approvals by risk** — require them for critical/prod scopes, but avoid over-blocking dev productivity
- **Use PIM for Groups** to simplify management at scale instead of assigning roles individually
- Start small: pick one critical resource, make its Owner role eligible with approval required, and test the activation cycle

### JIT Design Patterns — Putting It All Together

**Core idea: JIT/PIM is an architectural posture, not a toggle.** Every design decision answers four questions: **who** needs access, **what** they need it for, **how long** they need it, and **under what conditions** (MFA, justification, approval).

**Pattern → Tier mapping (the consolidated rule):**

| Tier | Pattern | Typical Resources |
|---|---|---|
| **Standard** | Eligible-by-default — user activates on demand, minimal extra friction | Routine workloads, lower-sensitivity resources |
| **High** | Eligible + **short activation windows** | Resources with meaningful blast radius but not top-tier |
| **Critical** | Eligible + **short windows + required approval** | Key Vaults, AI control planes, Global Administrator-class roles |

- **Trade-off to weigh deliberately:** eligible-by-default reduces standing privilege but adds activation friction for routine tasks; approvals prevent unilateral escalation but add latency and require designated approvers — match the friction to the risk, not the other way around

**Break-glass accounts are the deliberate exception to JIT** (reinforces the earlier guidance):
- Excluded from JIT because **PIM activation can fail during outages** (e.g., when authentication services themselves are degraded)
- Require: **zero normal/casual use**, **maximum monitoring and alerting**, and **documented, rehearsed procedures**
- Any activation of a break-glass account should trigger an **immediate investigation** — seeing one used casually is a red flag that something is wrong with your access design
- Audit break-glass usage on a **fixed schedule**, regardless of whether it was used

**Design Best Practices — Overall JIT Strategy:**
- **Inventory roles and workloads first**, then assign each to a risk tier (Standard / High / Critical) before configuring anything
- **Map controls to risk deliberately** — don't just "turn on" PIM features; combine short windows, approvals, and reviews into a coherent strategy you can explain and defend to auditors
- **Schedule regular access reviews — at minimum quarterly** — to catch stale privilege accumulation before it becomes a gap (accept the reviewer overhead as the cost of staying clean)
- **Recognize when permanent assignment is correct** — specific workload identities (per the two-track model) legitimately need permanent, narrowly-scoped RBAC rather than JIT
- **Logs and alerts are your safety net** — auditing completes the picture after you've chosen the right controls
- **Start small and prove the pattern**: pick one high-risk asset, apply eligible assignment + a short activation window, run it for a month, then scale the same template across similar resources
- Remember the throughline: **deliberate architectural decisions beat default settings every time**

### Audit & Review Features
- **Access reviews** — periodic checks (e.g., quarterly sweeps) where reviewers confirm or remove entitlements and drop stale roles
- **Activation notifications** — PIM emails the user and approver immediately when a role is activated
- **Audit log entries** — every activation records who activated which role, when it started, how long it lasted, and the justification given — this is the evidence auditors want

**Design Best Practices — Privileged Access:**
- **Remove permanent high-privilege assignments** — convert admin roles from "active/permanent" to **eligible** via PIM
- **Make eligible (JIT) the default**; reserve active assignments for tightly scoped, monitored break-glass scenarios only
- **Require MFA and justification** for sensitive role activations; add **approval** for the most sensitive roles
- **Set short activation durations** (within the 1–24 hour window) so elevated access automatically lapses
- **Enable audit logging and schedule regular access reviews** (e.g., review one privileged role within 30 days, then build a recurring cadence)
- Treat standing privilege as something to actively hunt down and eliminate — especially leftover roles from migrations or one-off projects
- Remember: **temporary is safer than permanent** — combine minimal active assignments with frequent reviews to balance resilience and security

### Kerberos & Global Secure Access Client

**Kerberos authentication — how it works:**
- Users authenticate to the **Key Distribution Center (KDC)** → receive a **Ticket Granting Ticket (TGT)**
- TGT is exchanged for a **Service Ticket** granting access to a specific resource (e.g., file share, on-prem app)
- Traditional Kerberos requires **line-of-sight to a domain controller** — a problem for cloud or remote users

**Entra Kerberos (Cloud Kerberos Trust):**
- Allows cloud-only or hybrid users to get Kerberos tickets for **on-premises resources without a domain controller line-of-sight**
- Enables SSO to on-prem Kerberos-protected apps and shares from Entra-joined devices
- **Azure Files** also uses Kerberos for SMB authentication when joined to a domain

**Global Secure Access Client — Microsoft's Security Service Edge (SSE):**

| Component | What It Does |
|---|---|
| **Microsoft Entra Private Access** | Replaces traditional VPN — secures access to private/corporate resources; applies Conditional Access to all private traffic regardless of user location |
| **Microsoft Entra Internet Access** | Secures access to SaaS apps and internet traffic through Microsoft's security fabric |
| **Global Secure Access Client** | Agent installed on user devices (Windows); routes traffic through the SSE fabric; handles **Kerberos ticket acquisition** for on-prem resources without a VPN or DC line-of-sight |

**Where Kerberos and Global Secure Access connect:**
- The Global Secure Access Client acquires Kerberos tickets on behalf of the user through the service, enabling **SSO to Kerberos-protected on-prem resources** (file shares, internal apps) with no traditional VPN

**Key Exam Triggers:**
- "Replace VPN" + "apply Conditional Access to private resources" → **Microsoft Entra Private Access + Global Secure Access Client**
- "SSO to on-prem resources without domain controller line-of-sight" → **Entra Kerberos / Cloud Kerberos Trust**
- "Secure internet/SaaS traffic at the network level" → **Microsoft Entra Internet Access**

---

## 8. Azure Policy Effects

| Effect | When to Use |
|---|---|
| Deny | Block non-compliant resources from being created/modified |
| Audit | Log non-compliant resources without blocking |
| Modify | Add/change tags or properties on existing resources (tag remediation) |
| Append | Add fields during create/update — cannot update existing |
| DeployIfNotExists | Deploy a related resource if one doesn't exist (e.g. deploy diagnostics) |
| AuditIfNotExists | Audit if a related resource doesn't exist |

**Key Triggers:**
- "Auto-remediate tags" or "add missing tag" → Modify
- "Deploy diagnostic settings automatically" → DeployIfNotExists
- "Block non-compliant resources" → Deny
- "Report but don't block" → Audit

---

## 8a. Cost Management & Governance — Guardrails

### Core Tools

| Tool | What It Does |
|---|---|
| **Tags** | Key-value metadata applied to resource groups/resources (e.g., `environment: pilot`, `owner: it-team`) — used for cost reporting, filtering, and chargeback to real teams/projects |
| **Budgets** | Monitor actual spend against a set amount; send alerts at defined thresholds (commonly 80% and 100%) so stakeholders get early warnings before costs escalate |
| **Azure Policy** | Enforces organizational rules (e.g., *Allowed locations*) and can **deny** noncompliant deployments at creation time — prevents configuration drift and accidental cost exposure |

### Design Best Practices — Cost Guardrails
- **Tag consistently** — agree on a small, standard set of tag keys (environment, owner, cost center) so reporting maps cleanly to teams and projects
- **Apply controls at the smallest manageable scope** — resource group for pilots, subscription for broader/organization-wide controls — so alerts and enforcement only affect intended resources
- **Set budgets with layered alerts** (e.g., 80% warning, 100% hard alert) so teams can act before overspend happens, not after
- **Use Azure Policy to block, not just report** — *Allowed locations* and similar deny-effect policies stop noncompliant resources (e.g., wrong region) at creation time rather than after the fact
- **Always validate and clean up:** test both the denied path (attempt a noncompliant deployment) and the allowed path (deploy correctly), confirm compliance in the Policy dashboard, and **remove policy assignments before deleting resource groups** — leftover assignments can block deletions

### Practical Rollout Sequence (start small, repeat)
1. Create a pilot resource group
2. Apply consistent tags (`environment`, `owner`)
3. Set a modest monthly budget with 80%/100% email alerts
4. Assign an *Allowed locations* policy at the resource-group scope
5. Validate by testing both a denied deployment and an allowed deployment
6. Scale the same pattern across the organization once proven

**Worked examples:**
- *FinOps:* A $10/month budget scoped to a pilot resource group with 80%/100% alerts — at 80%, the ops lead reviews tagged resources by owner and rightsizes/pauses a test VM to avoid overshoot
- *Cloud engineering:* Assigning the built-in *Allowed locations* policy to a pilot resource group — a developer trying to create a storage account in a disallowed region gets a policy deny message and must correct the region before provisioning

---

## 8b. Cloud Adoption Framework (CAF)

A structured Microsoft methodology for adopting Azure. AZ-305 tests it primarily around **governance design, landing zones, and infrastructure strategy**.

### The Eight Phases

| Phase | Focus |
|---|---|
| **Strategy** | Define business justification, motivations (migrate vs. innovate), expected outcomes |
| **Plan** | Digital estate assessment, skills readiness plan, cloud adoption plan |
| **Ready** | Prepare the environment — this is where **Landing Zones** are designed and deployed |
| **Adopt** | Two tracks: **Migrate** (lift-and-shift) or **Innovate** (cloud-native new capabilities) |
| **Govern** | Apply governance disciplines across the environment |
| **Manage** | Operations management — business commitments, operations baseline, expand as needed |
| **Secure** | Security integrated across the full adoption lifecycle |
| **Organize** | Align teams and functions to cloud responsibilities |

### Five Governance Disciplines

| Discipline | What It Governs |
|---|---|
| Cost Management | Control and optimize cloud spend |
| Security Baseline | Align environment with security requirements |
| Identity Baseline | Identity and access management standards |
| Resource Consistency | Resource organization, tagging, and naming |
| Deployment Acceleration | Standardize and automate deployments |

### Landing Zones (most tested for AZ-305)
A **Landing Zone** is a pre-configured Azure environment following CAF best practices — established *before* workloads are deployed so governance, networking, and security are ready from day one.

**Built around:**
- **Management Groups hierarchy** → **Subscriptions** → **Azure Policy** → **RBAC** → **Networking**

**Subscription archetypes:**

| Type | Purpose |
|---|---|
| Platform | Shared services — connectivity, identity, management |
| Landing Zones | Workload subscriptions for applications/teams |
| Sandboxes | Isolated dev/test environments with relaxed policy |
| Decommissioned | Retired subscriptions staged for removal |

**Two landing zone approaches:**

| Approach | When to Use |
|---|---|
| Start small and expand | Smaller orgs or pilots — build incrementally |
| Enterprise-scale (Azure Landing Zone) | Large orgs needing full governance and compliance from day one |

### Key Exam Triggers

| Trigger | Answer |
|---|---|
| "Establish governance at scale before workloads deploy" | Landing Zone (CAF Ready phase) |
| "Standardize resource deployment across teams" | Deployment Acceleration discipline |
| "Migrate existing workloads with minimal change" | CAF Migrate track |
| "Build new cloud-native capabilities" | CAF Innovate track |
| "Organize subscriptions and policies at scale" | Management Group hierarchy (CAF Ready phase) |
| "Shared services — networking, identity, management" | Platform subscription archetype |
| "Isolated dev environment with relaxed controls" | Sandbox subscription |

### Design Best Practices — CAF
- **Land before you lift** — design the landing zone (management groups, policy, RBAC, networking) before migrating any workloads
- **Apply governance disciplines in order of risk** — start with Identity Baseline and Security Baseline; add Cost Management and Resource Consistency early
- **Choose Enterprise-scale** when your org needs consistent compliance across many teams and subscriptions from day one; choose Start Small when you're piloting or iterating
- **Management Groups enable policy inheritance at scale** — assign Azure Policy and RBAC at the management group level so every child subscription inherits the controls automatically
- **Treat the landing zone as a living design** — expand it as requirements grow rather than rebuilding from scratch

---

## 9. Key Vault Patterns

| Scenario | Pattern |
|---|---|
| App accesses secret at runtime, auto-updates on rotation, no redeployment | Key Vault Reference in App Config + Managed Identity |
| App retrieves secret via SDK in code | Managed Identity + SDK call (does NOT auto-refresh on rotation) |
| Encrypt storage with customer-controlled keys | Customer-Managed Keys (CMK) in Key Vault |
| Prevent permanent deletion of secrets/keys for N days | Soft Delete + Purge Protection |
| Restrict Key Vault access to specific VNets only | Private Endpoint + disable public access |

---

## 10. ExpressRoute Scenarios

| Scenario | Solution |
|---|---|
| Multiple subscriptions in same org share one ER circuit | VNet Peering to hub where ER gateway lives |
| External org connects their VNet to your ER circuit | ExpressRoute Authorization Keys |
| Connect two on-premises sites through Azure via two ER circuits | ExpressRoute Global Reach |
| ER circuit fails — failover to VPN automatically | ER + VPN coexistence with BGP on VPN gateway |

---

## 11. Storage for Non-Relational Data

### Data Types

| Type | Description | Examples |
|---|---|---|
| Structured | Fixed schema, rows and columns | Customer tables, orders, financial accounts |
| Semi-structured | Tags/serialization add meaning, no rigid columns | JSON, XML, HTML |
| Unstructured | No shared schema, nonrelational | Photos, videos, audio, Office files, product images |

### Storage Object Decision Table

| Data / Use Case | Storage Object |
|---|---|
| Large media files, backups, logs | **Azure Blob Storage** (Binary Large Object) |
| Shared file access, lift-and-shift file servers, team collaboration | **Azure Files** (SMB, NFS, REST) |
| VM OS disks and application volumes | **Managed Disks** (block-level storage) |
| Async message backlogs, work queues | **Queue Storage** |

**Key design rule:** map each dataset to a storage object by format, access pattern, and cost — not just file type.

### Design Best Practices — Storage Objects
- **Blob Storage** for large media and unstructured files; choose the right access tier to avoid overpaying for rarely accessed data
- **Azure Files** when teams need a shared network drive experience (SMB) or when lifting on-prem file servers to the cloud
- **Managed Disks** only for VM-attached block storage — don't use blobs as VM disks
- **Queue Storage** for decoupling producers and consumers in async workflows
- Test access patterns with a sample dataset before committing to a tier — avoid placing infrequently accessed archives on premium storage

### Storage Accounts
- Groups Azure storage services (blobs, files, queues, tables) under one unique global namespace accessible over HTTPS
- Stores configuration: location, replication strategy, subscription, and access settings
- The account itself has no cost — replication, performance tier, and access tier drive pricing

**Account types:**

| Type | Supports | Best For |
|---|---|---|
| Standard GPv2 | Blobs, Files, Queues, Tables | Default for most workloads |
| Premium Block Blobs | Block blobs only | High transaction rates, low latency, small objects |
| Premium File Shares | SMB + NFS file shares | Enterprise file workloads requiring high IOPS |
| Premium Page Blobs | Page blobs only | VM disks, OS and database files (random read/write) |

**When to use multiple storage accounts:**
- Different regions (data residency / latency requirements)
- Different compliance or regulatory requirements
- Separate billing and chargeback per department or project
- Isolate proprietary or regulated data with VNet restrictions

**Design Best Practices — Storage Accounts:**
- **Start small** — begin with a minimal set of accounts; add accounts only for distinct compliance, performance, or billing needs
- **Locate accounts close to users** to reduce latency and satisfy data residency rules
- **Tag accounts** by purpose, region, and cost center for clearer billing and audits
- **Use VNet restrictions** to protect proprietary or regulated data
- **Separate critical from non-critical data** across accounts — apply GRS to critical, LRS to transient/cheap data
- **Document naming conventions** — more accounts = more admin overhead; clarity is essential
- Avoid large-scale account reorganizations; make small, intentional changes

### Replication / Redundancy

| Type | What it protects | Cost |
|---|---|---|
| LRS (Locally Redundant) | 3 copies within one datacenter, rack failures | Lowest |
| ZRS (Zone Redundant) | Copies across availability zones in one region | Medium |
| GRS (Geo Redundant) | Copies to secondary region | Higher |
| RA-GRS | GRS + readable secondary | Highest |

- **Rule of thumb:** Global read-heavy assets (product images) → GRS/RA-GRS. Ephemeral local logs → LRS.
- Use GRS for critical backups; LRS for transient or easily regenerated data to save cost.

### Storage Account Types

| Type | Use Case |
|---|---|
| General Purpose v2 (GPv2) | Default for most workloads — supports all storage services and tiers |
| Premium Block Blobs | High-throughput object storage, low latency blob workloads |
| Premium Page Blobs | Low-latency disk workloads (unmanaged disks) |
| Premium File Shares | High IOPS file shares requiring low latency |

**Design best practice:** Use GPv2 by default. Only choose Premium accounts when your workload has strict latency or IOPS requirements.

### Blob Types

| Blob Type | When to Use |
|---|---|
| Block Blob | General objects — files, images, videos, backups (most common) |
| Page Blob | Random read/write access — VM unmanaged disks |
| Append Blob | Write-only append operations — logging, telemetry, audit trails |

**Exam trigger:** "Optimized for append operations / logging" → Append Blob

### Blob Storage Tiers

| Tier | Min. Retention | Availability | Latency / First Byte | Cost Profile |
|---|---|---|---|---|
| Hot | None (default for new accounts) | ~99.9% | Milliseconds | Highest storage cost, lowest access cost |
| Cool | ≥30 days | High | Milliseconds | Lower storage cost, higher access/transaction cost |
| Cold | ≥90 days | High | Milliseconds | Lower still, higher access cost than Cool |
| Archive | ≥180 days | Offline | Hours to rehydrate | Lowest storage cost, highest access/retrieval cost |

**Design best practices — Tiers:**
- **Hot** → actively processed / frequently read-write data (default tier)
- **Cool / Cold** → infrequently accessed data that still needs millisecond retrieval (backups, older logs)
- **Archive** → data you rarely need and can tolerate hours of retrieval delay (compliance archives, long-term raw data)
- Always weigh **storage cost vs. access/transaction fees vs. recovery time** — Premium/Hot = cheap to access, expensive to store; Cool/Archive = cheap to store, expensive to access
- **Inventory and tag datasets** by access frequency and retention need, then map each to a tier — revisit periodically as access patterns change

### Blob Lifecycle Management Policies
- Automatically move blobs between tiers based on age or last-accessed date
- Example rule: move to Cool after 30 days idle → Archive after 90 days → delete after 365 days
- **Exam trigger:** "Automatically move blobs to a cheaper tier based on age" → Lifecycle Management Policy
- **Design best practice:** Always configure lifecycle policies for cost control; never leave infrequently accessed data on Hot tier permanently

### Blob Soft Delete & Versioning
- **Soft delete** — retains deleted blobs for a configurable number of days (recoverable)
- **Versioning** — automatically saves prior versions of a blob on every overwrite
- Both protect against accidental deletion or overwrite at the object level (separate from RSV soft delete)
- **Design best practice:** Enable both on all production storage accounts

### Immutable Storage (WORM)
- Write Once, Read Many — prevents modification or deletion for a set period; protects against accidental or malicious deletes/overwrites
- Applied at the **container level**; supports audit logs; available on Premium Blob Storage

**Two immutability types:**

| Type | Behavior | Best For |
|---|---|---|
| Time-based retention | Fixed interval during which objects can't be modified/deleted | Predictable compliance windows (e.g., "retain raw sensor data 90 days") |
| Legal hold | Immutable until the hold is explicitly cleared — no fixed end date | Investigations or compliance situations with unpredictable duration |

**Design best practices — Immutability:**
- Use **time-based retention** when you know the required retention window in advance (regulatory/business recovery needs)
- Use **legal hold** for open-ended situations like litigation or investigations
- **WORM compliance / no admin can delete** → Immutability locked policy on Storage or RSV
- **Backup deletion blocked even from compromised admin** → RSV Immutability + MUA (Multi-User Authorization / Resource Guard)
- Soft delete alone is NOT enough — an admin can disable it, then delete backups
- Balance retention duration against realistic restore/recovery windows — don't over-retain and drive up cost

### Azure Files — Deep Dive

**What it is:** fully managed, serverless cloud file shares that can replace on-prem NAS; supports SMB, NFS 4.1, and REST. Common for app/config storage so new VMs connect quickly, and for lift-and-shift of legacy file shares.

**Two access models:**

| Model | How it Works | Best For |
|---|---|---|
| Direct mount (SMB/NFS) | Serverless — mount the share directly, no file server to manage | Cloud-native, cross-platform clients, low ops overhead |
| Azure File Sync | Caches Azure file shares on a Windows Server for fast local reads, centralizes shares | Large legacy on-prem shares, local responsiveness, smoother migrations |

- Both can **coexist** for hybrid scenarios

**Performance tiers:**

| Tier | Media | Best For |
|---|---|---|
| Premium | SSD | IO-intensive workloads — databases, dev environments; supports ZRS in select regions; highest IOPS/bandwidth |
| Transaction Optimized | HDD | Many small operations |
| Hot | HDD | General-purpose team shares |
| Cool | HDD | Cost-efficient online archive storage |

### Azure Files vs Blob Storage vs Azure NetApp Files

| Service | Access Pattern | Approx. Performance | Best For |
|---|---|---|---|
| Azure Blob Storage | Large, read-heavy, sequential | ~20,000 IOPS / ~15 GiB/s | Big sequential workloads, low maintenance, media/logs/backups |
| Azure Files | Random-access, shared files (SMB/NFS/REST) | Up to ~100,000 IOPS / ~10 GiB/s | Lift-and-shift file shares, team collaboration, app config |
| Azure NetApp Files | Enterprise, latency-sensitive | Up to ~460,000 IOPS, very high throughput | SAP HANA, Oracle, HPC — ultra-low latency enterprise workloads |

**Design Best Practices — Azure Files:**
- Choose **direct mounts** for low-ops, cloud-native, cross-platform access
- Choose **Azure File Sync** for large legacy shares, local caching, or smoother phased migrations — and combine both for hybrid needs
- Match **performance tier** to workload: Premium (SSD) for IO-intensive/databases, Transaction Optimized for many small ops, Hot for general team shares, Cool for archives
- Decide between Files, Blob, and NetApp Files based on **protocol needs, performance goals, and migration complexity**
- **Pilot first** — test with a small share using your expected access method, measure IOPS/latency/cost, then scale

### Azure Table Storage
- Semi-structured NoSQL key-value store
- Massive scale, very low cost, no complex queries
- **Exam trigger:** "Semi-structured data, cheap, no complex queries needed" → Table Storage
- vs Cosmos DB: choose Table Storage when you don't need global distribution, low ms latency, or advanced consistency models; choose Cosmos DB when you do

### Azure NetApp Files
- High-performance, fully managed NFS/SMB file shares
- **Exam trigger:** "SAP HANA", "Oracle", "HPC", "ultra-low latency shared file storage" → Azure NetApp Files
- Supports NFS v3, NFS v4.1, SMB; multiple performance tiers (Standard, Premium, Ultra)

### Managed Disk Tiers

| Tier | Media | Best For |
|---|---|---|
| Ultra Disk | SSD | IO-intensive top-tier workloads — SAP HANA, high-end databases |
| Premium SSD | SSD | High throughput/IOPS, low latency, available in all regions — most production VMs |
| Standard SSD | SSD | Web servers, light enterprise apps — consistent, cost-effective |
| Standard HDD | HDD | Backups, infrequently accessed/non-critical data — slower |

**Key terms:**
- **IOPS** (Input/Output Operations Per Second) — measures disk read/write throughput (e.g., 10,000 small reads/sec for a busy database). The metric to watch when sizing disks.
- **VHD** (Virtual Hard Disk) — the VM's disk file (OS or data drive)

**Disk caching:**
- Default OS disk caching = **ReadWrite**; default data disk caching = **ReadOnly**
- Caching is **not supported on disks ≥ 4 TiB**
- Changing the cache setting **detaches and reattaches** the disk (causes brief disruption — plan for it)

**Size & attachment limits:**
- Each data disk supports up to **32,767 GiB**
- Number of disks you can attach depends on **VM size**

**Design Best Practices — Managed Disks:**
- **Match disk type to workload**: start with Standard SSD, measure IOPS/latency, then scale up to Premium or Ultra only if needed
- Use **Ultra Disk** only for the most IO-intensive workloads (SAP HANA, top-tier DBs) — limited region/zone availability
- Use **Standard HDD** only for backups or rarely accessed data
- **Tune caching** to the read/write pattern of the workload — and remember changing it briefly detaches the disk
- Let Azure handle provisioning — managed disks remove the operational burden of managing storage accounts for VM disks
- Plan attachment counts around your chosen **VM size**

### Disk & Storage Encryption Options

| Option | Layer / Scope | When to Use |
|---|---|---|
| Azure Disk Encryption (ADE) | VM-level — encrypts the VHD so only the owning VM can read it | Protect OS/data drives (e.g., database disks) at the VM level |
| Server-Side Encryption (SSE) | Physical disk in the datacenter — protects data at rest | Default at-rest protection; keeps raw disk unreadable if physically removed |
| Encryption at host | VM host — encrypted data flows directly into Azure Storage | Stronger guarantee than SSE alone; avoids relying on storage-side encryption |
| Microsoft-managed keys | Key management | Simplicity, routine workloads |
| Customer-managed keys (CMK) | Key management | Strict compliance, regulated datasets — adds key management overhead |

**Design best practice:** Layer encryption based on compliance need — SSE is the baseline, add ADE for VM-level VHD protection, and use encryption at host or CMK when regulatory requirements demand stronger guarantees or customer key control.

### Storage Security — Layered Model

Azure Storage security works best as **layered, combinable controls** — think network, access, transfer, and keys as stacked steps rather than either/or choices.

| Layer | Control | What It Does |
|---|---|---|
| Network | Firewall rules | Allow specific IPs or VNet subnets; block the public endpoint |
| Network | Private Endpoint | Dedicated NIC that connects the storage account privately inside your VNet — no public traffic |
| Access | SAS (Shared Access Signature) | Delegated, time-bound token granting limited access without sharing full account keys (e.g., "let a partner read a blob for 2 hours") |
| Transfer | Secure transfer required | Enforces HTTPS; rejects non-secure (HTTP) requests |
| Encryption | Microsoft-managed keys | Microsoft handles key lifecycle — simpler operations |
| Encryption | Customer-managed keys (CMK) | Keys stored/controlled in Azure Key Vault — you manage rotation, access, revocation |

**Design Best Practices — Storage Security:**
- **Start with the Azure Security Baseline** — a checklist of recommended controls to prioritize (limited access, firewall rules, etc.)
- **Require secure transfer (HTTPS)** on every storage account
- **Lock down networks** with firewall rules and/or private endpoints to minimize public exposure
- **Use SAS instead of account keys** for delegated access — run an inventory of who needs direct storage access and replace account keys with SAS where possible
- **Choose CMK over Microsoft-managed keys** when regulatory controls or auditability require you to manage key rotation/revocation yourself; otherwise Microsoft-managed keys reduce operational overhead
- **Defaults should be strict, but tested** — strict settings reduce risk but can block needed platform services; enforce strict defaults, then add specific trusted platform services as exceptions
- Encryption is **on by default** (SSE) — layer ADE, encryption at host, or CMK on top based on compliance needs (see encryption table above)

---

## 12. Advanced Concepts

### KEDA (Kubernetes Event-Driven Autoscaling)
- Powers event-driven scaling in **Azure Container Apps**
- Monitors Service Bus queues, Event Hubs, HTTP traffic → scales to zero
- "Queue/event triggered container scaling" → Container Apps with KEDA (not ACI)

### MUA — Multi-User Authorization
- Protects RSV/Backup Vault destructive operations; requires a second authorized user (Resource Guard)
- Even a compromised global admin cannot delete backups alone

### Availability: Sets vs Zones

| | Availability Set | Availability Zone |
|---|---|---|
| Protects against | Rack/hardware failure | Full datacenter failure |
| SLA | 99.95% | 99.99% |
| Exam trigger | Legacy apps, rack-level HA | Datacenter failure, highest SLA |

### SQL Backup Retention

| Service | Method | Max Retention |
|---|---|---|
| Azure SQL Database | Native automated backups + LTR policy | Up to 10 years |
| SQL Managed Instance | Native automated backups + LTR policy | Up to 10 years |
| SQL Server on Azure VM | Azure Backup or IaaS Agent Extension | Flexible |

---

## 13. Quick-Fire Cheat Sheet

| Trigger | Answer |
|---|---|
| Exactly-once processing | Service Bus |
| Millions of events/sec / telemetry | Event Hubs |
| Kafka compatible | Event Hubs |
| React to blob created | Event Grid |
| Global multi-write / single-digit ms latency | Cosmos DB |
| Petabyte SQL / data warehouse | Synapse Analytics |
| Spark / ML pipelines | Databricks |
| RTO/RPO / VM replication / VMware DR | Azure Site Recovery |
| No password hashes in Azure AD | Pass-through Authentication |
| Consumer users / social login / custom branding | Azure AD B2C |
| Without storing credentials (Azure-to-Azure) | Managed Identity |
| Auto-update secret without redeployment | Key Vault Reference in App Config |
| Add missing tags automatically | Azure Policy — Modify effect |
| Deploy diagnostics automatically | Azure Policy — DeployIfNotExists |
| Full OS control / install custom software | Virtual Machines |
| Scale to zero + containers | Azure Container Apps |
| Queue/event triggered + container scaling | Container Apps with KEDA |
| Short-lived single container, no orchestration | Azure Container Instances (ACI) |
| Continuous 24/7 GPU workload | AKS with GPU node pools |
| Stateful orchestration / human approval / wait days | Durable Functions |
| HPC / parallel jobs / thousands of nodes | Azure Batch |
| WAF + global + nearest endpoint | Azure Front Door |
| WAF + regional + path-based routing | Application Gateway |
| External org shares your ER circuit | ExpressRoute Authorization Keys |
| Connect two on-prem via two ER circuits | ExpressRoute Global Reach |
| Backup deletion blocked even from compromised admin | RSV Immutability + MUA |
| WORM compliance, no admin can delete | Immutability locked policy on Storage or RSV |
| SQL backup beyond 35 days | Native automated backups + LTR policy |
| Datacenter failure / 99.99% SLA | Availability Zones |
| Rack/hardware failure protection | Availability Sets |
| ML models + streaming + batch combined | Azure Databricks |
| Infrequent blob access, cost savings | Cool or Cold tier |
| Compliance lock, no modification for X days | WORM (immutable storage) time-based policy |
| Lift & shift file server to Azure | Azure Files + Azure File Sync |
| Managed, Kafka-compatible event ingestion | Azure Event Hubs |
