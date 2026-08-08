# SC-100 Notes — Microsoft Cybersecurity Architect

> Organized by the official **Skills Measured** blueprint (as of July 28, 2026). Paste lesson transcripts/summaries and they get filed under the matching exam objective below.

## Exam Domains (weightings)
1. **Design solutions that align with security best practices and priorities** (20–25%)
2. **Design security operations, identity, and compliance capabilities** (25–30%)
3. **Design security solutions for infrastructure** (25–30%)
4. **Design security solutions for applications and data** (20–25%)

Pass score: 700/1000. *Source: [Microsoft Learn SC-100 Study Guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/sc-100).*

---

# Domain 1 — Design solutions that align with security best practices and priorities (20–25%)

## 1.1 Resiliency strategy for ransomware and other attacks (Microsoft Security Best Practices)

**Business resiliency:** you can't protect everything equally — identify and **prioritize business-critical assets** ("crown jewels") and accept that a breach *will* happen (assume breach). Resiliency = ability to keep operating and recover fast, not just prevent.

**Ransomware defense (Microsoft's 3-pillar priority order):**
1. **Prepare / prevent recovery cost** — secure **backups** so you never need to pay: follow the **3-2-1 rule** (3 copies, 2 media, 1 offsite), keep backups **offline/immutable** and separated from production identity. Azure Backup **soft delete** + **immutable vaults** + **multi-user authorization (MUA)** stop attackers deleting backups.
2. **Limit scope of damage** — protect **privileged access** (attackers seek domain/global admin to mass-deploy ransomware). Tiered/enterprise access model, PIM JIT, MFA.
3. **Make intrusion harder** — patching, attack surface reduction, email/endpoint protection.

**BCDR:** design **secure backup and restore** for hybrid/multicloud. Key services: **Azure Backup** (VMs, files, SQL, immutable/soft-delete), **Azure Site Recovery (ASR)** for DR replication/failover. Test restores regularly; protect the backup plane with its own RBAC + MFA.

**Security updates:** keep systems patched to close known-vuln attack surface. **Azure Update Manager** (unified patch management across Azure, Arc-enabled on-prem/multicloud); **Defender Vulnerability Management** to prioritize by risk.

*Frameworks referenced:* MCRA (reference architectures), Azure Security Benchmark/MCSB, and the **Zero Trust** "assume breach" principle.

## 1.2 Align with MCRA and Microsoft Cloud Security Benchmark (MCSB)
- Best practices for cybersecurity capabilities and controls
- Best practices for protecting against insider, external, and supply chain attacks
- Design AI solutions aligned to MCSB
- Align with the Zero Trust adoption framework

### Securing multi-agent systems with Azure zero-trust architecture
Making every agent, call, and data item **authenticated, authorized, and auditable** — reduces cross-tenant leak risk and makes compliance evidence repeatable. *(Cross-cutting: also touches [D3.4 network](#34-network-security-and-security-service-edge-sse) and [D4.3 data](#43-securing-an-organizations-data).)*

**Per-agent identities + least privilege:** each agent gets its own **managed identity**; assign **narrow RBAC at the smallest scope** (container, storage container, Cosmos DB partition). Unique identities limit blast radius and enable precise audit trails.
- Prefer a **system-assigned managed identity** per agent — **never share identities**.
- After **publishing**, reassign least-privilege RBAC using the **`agentIdentityId`** so each agent keeps only function-specific permissions.
- Use **federated identity with OBO token exchange** so agents act **temporarily in a user's context** for customer-owned resources (time-bound, function-specific).

**Authentication flows & secrets lifecycle** (4 Entra ID patterns — choosing the right flow prevents privilege escalation):
- **Managed identity** for service-to-Azure-service calls — removes credentials from code, automatic token rotation. **Default for Azure resources.**
- **OBO** when an agent must act as a specific user.
- **OAuth2 authorization code + PKCE** for third-party **interactive consent** flows.
- **Key-based** only as a fallback.
- Store all secrets/refresh tokens/API keys in **Azure Key Vault** with **least-privilege RBAC**; rotate with **blue-green** patterns; use **per-service keys**; prefer **certificates/keys over plaintext secrets**; use **CMK** when crypto-shred or stricter control is required.

**JIT & workload identity:** **Azure PIM** makes high-sensitivity roles **eligible** with **human approval** for short, time-bound activations. **AKS workload identity** binds Kubernetes service accounts to Azure managed identities via **federated credentials per agent service account** so **pods never hold static/stored credentials**. In agent-to-agent APIs, add middleware to validate the bearer **JWT (signature, issuer, audience, expiry)** and check the caller's **object ID (oid) against an allowlist** to prevent impersonation/unauthorized lateral calls.

**Network segmentation & cryptographic controls (prevent lateral movement in AKS):** replace implicit-allow with **deny-by-default NetworkPolicy** — apply a **deny-all in the agents namespace**, then add minimal allow rules (e.g., **orchestrator→specialists on port 8000**). Eliminate public endpoints via **Azure Private Link** (services reachable only from your VNet). Deploy a **service mesh (Istio/Linkerd)** to enforce **mTLS** so both client and server authenticate automatically. Forward **flow logs/traces to Microsoft Sentinel** to alert on any communication outside the baseline.

**Multitenant data isolation & customer control:** propagate **tenant context end-to-end** — middleware **verifies the JWT** and sets a **request-scoped `tenant_id`** on every request; enforce **`tenant_id` as the Cosmos DB partition key** so queries are **physically scoped**; validate tenant boundaries at every API; offer **per-tenant CMK** where cryptographic separation or **crypto-shred** is required.
- **Isolation architectures** (trade off security / cost / update complexity): **per-tenant** (strongest isolation, highest cost), **shared with logical isolation** (cheapest, relies on `tenant_id` enforcement), or **hybrid** (e.g., generic analysis on shared agents, sensitive data kept per-tenant). Pick based on threat model and data sensitivity (e.g., PHI).
- **Boundary validation & encryption enforcement:** validate tenant ID at **every API endpoint** (tenant-check decorator), deny mismatches with **HTTP 403**. Encrypt each tenant's data with a **CMK in the tenant's own Key Vault** with **minimal key permissions** for the service identity — so if a tenant **revokes key access**, the provider can't decrypt their records (supports crypto-shred/IR). Log **`TenantBoundaryViolation`** events to **Microsoft Sentinel** and use **KQL** to detect spikes/patterns signaling compromise or bugs.

*Examples:*
- **EU residency/erasure:** region-locked resources, per-tenant storage + Cosmos DB partitioning, encrypt with customer's **CMK in their Key Vault**, policies denying non-EU deployments → cryptographic + policy basis for auditors and crypto-shred on request.
- **Agent API orchestrator:** middleware extracts/stores `tenant_id` from validated JWTs, rejects callers whose managed-identity object ID isn't allow-listed, and relies on service-mesh **mTLS** — so a compromised pod can't impersonate another agent or reach other tenants' data.

*Takeaway:* Three steps deliver the biggest cross-tenant risk reduction + audit trails: (1) **unique managed identity per agent**, (2) **RBAC scoped to minimal resource/container**, (3) **token validation + tenant-context middleware**.

*Zero-trust principle:* traditional perimeter models trust everything inside the network boundary; **zero-trust assumes no component has inherent trust** — every request from any source needs explicit authN + authZ. Apply controls at **every agent interaction boundary**, not just the external API gateway (agent-to-agent calls authenticate with cryptographic proof; network policies block compromised agents from reaching unrelated agents; tenant context propagates through every operation).

**Threat modeling — STRIDE** (Microsoft framework, six categories) mapped to controls:
- **S**poofing (impersonating an agent) → per-agent managed identity, JIT
- **T**ampering (modifying messages in transit / data at rest) → network isolation, encrypted channels
- **R**epudiation (denying an action occurred) → immutable audit schemas
- **I**nformation disclosure (leaking proprietary/tenant data) → tenant isolation, data residency
- **D**enial of service (resource exhaustion) →
- **E**levation of privilege (permissions beyond scope) → per-agent identity, JIT

For **AI-specific threats** (prompt injection, model extraction, adversarial inputs) use the **OWASP LLM Top 10**. This infra layer (identity, network, data isolation, compliance) applies regardless of which AI-specific defenses sit above it.

**SIEM/SOAR integration:** the controls generate managed-identity audit logs, network flow logs, tenant-context records, and compliance evidence. Feed **OpenTelemetry traces + structured logs + immutable audit records** (agent ID, timestamp, action) into **Microsoft Sentinel (SIEM)** for unified cross-tenant threat detection and **SOAR** for automated response — e.g., a compromised agent's anomalous call pattern lets Sentinel correlate across tenants and trigger an automated containment runbook.

## 1.3 Align with the Cloud Adoption Framework (CAF) and Azure Well-Architected Framework (WAF)

**Three key frameworks (know what each is for):**
- **MCRA** (Microsoft Cybersecurity Reference Architectures) — diagrams of Microsoft security capabilities and how they integrate (the "what").
- **MCSB** (Microsoft Cloud Security Benchmark) — prescriptive control baseline mapped to CIS/NIST/PCI (the "controls").
- **CAF** (Cloud Adoption Framework) — end-to-end lifecycle: **Strategy → Plan → Ready → Adopt → Govern → Manage → Secure**. **WAF** (Well-Architected Framework) — workload design across 5 pillars: **Reliability, Security, Cost Optimization, Operational Excellence, Performance Efficiency**.

**Azure landing zones** — pre-architected, governed environment following CAF. **Platform** landing zones (shared: identity, management, connectivity) vs **application** landing zones (workloads). Enforce security centrally via **Management Groups + Azure Policy** (guardrails inherit down the hierarchy), RBAC, and hub-spoke/vWAN networking. Bakes in Zero Trust, logging, and policy from day one.

**Governance = Azure Policy + Management Groups:** Policy defines audit/deny/**deployIfNotExists** rules; **initiatives** bundle policies; assign at management-group scope so subscriptions inherit. Use for regulatory baselines, region locking, required tags, enforced diagnostic settings.

**Secure AI adoption (CAF "Secure AI"):** govern which AI services/models are allowed, protect training/inference data (classification, DLP, CMK), least-privilege workload identities, add **Defender for AI Services** + **Purview DSPM for AI**, threat-model with **OWASP LLM Top 10 / MITRE ATLAS**.

**DevSecOps** — shift security **left** into CI/CD. **DevOps security in Defender for Cloud** connects GitHub/Azure DevOps to surface code, secret, dependency (SCA), and IaC scanning findings in one posture view. Practices: secret scanning, SAST/DAST, dependency scanning, IaC scanning, signed artifacts, least-privilege pipeline identities.

### Decision table — framework / tool selection

| Scenario | Framework / Tool |
|----------|------------------|
| Understand how Microsoft security capabilities fit together | MCRA |
| Prescriptive control baseline mapped to CIS/NIST/PCI | MCSB |
| End-to-end cloud adoption lifecycle guidance | Cloud Adoption Framework (CAF) |
| Design one workload for security/reliability/cost | Well-Architected Framework (WAF) |
| Stand up a governed, pre-secured Azure environment | Azure landing zones |
| Enforce guardrails across many subscriptions | Management Groups + Azure Policy initiatives |
| Embed security scanning into CI/CD pipelines | DevOps security (Defender for Cloud) |
| Unified patching across Azure + on-prem/multicloud | Azure Update Manager |

---

# Domain 2 — Design security operations, identity, and compliance capabilities (25–30%)

## 2.1 Solutions for security operations

### Microsoft Sentinel SIEM — setup & operations
Configuring Sentinel as a **SIEM** starts with provisioning a **Log Analytics workspace**, then configuring Sentinel on top. Module scope / capabilities:
- Create and configure a **Sentinel workspace**
- Deploy **Content Hub solutions and data connectors**
- Configure **Data Collection rules**, an **NRT (near-real-time) analytic rule**, and **Automation**
- Perform a **simulated attack** to validate analytic + automation rules
- Connect Sentinel to **Microsoft Defender XDR**

### Designing SecOps across hybrid & multicloud
A coherent SecOps design reduces analyst overload and speeds detection/response.

- **Unified operations:** combine **SIEM + SOAR + XDR in a single Defender portal** to correlate incidents across **Azure, AWS, GCP, on-prem** and cut tool-switching for SOC analysts.
- **Monitoring vs logging:** **Monitoring** = real-time ("what's happening now") via **Azure Monitor + Arc**; **centralized logging** = "what happened" via **Log Analytics workspaces** and Sentinel's analytics/data-lake tiers for retention/forensics.
- **Workspace & data-tier design:** choose topology (**single, separate, regional, or dedicated**) based on correlation needs, data residency, and cost. **Analytics tier** = near-term detection; **data lake tier** = long-term retention (**up to 12 years**). Route high-volume logs (e.g., firewall) to the data lake to cut analytics cost; keep critical identity logs in analytics for rapid triage.
- **Detection engineering:** integrate **Defender XDR deep signals** (endpoint, email, identity) with **Sentinel SIEM breadth**; prioritize **high-fidelity alerts**; **map detections to MITRE ATT&CK** to find gaps; plan ongoing rule tuning + ML/behavioral detections.
- **SOAR & workflows:** automate repeatable, low-risk tasks with **playbooks + automation rules**; keep **human approval for high-impact actions**; design incident, threat-intel, and hunting workflows with clear **escalation, SLAs, and post-incident feedback loops**.

*Examples:* **Cloud-first enterprise** — Azure as primary control plane, project AWS + on-prem via **Azure Arc**, single Log Analytics workspace for correlated investigations, high-volume firewall logs → data lake. **MSSP pattern** — host core playbooks/analytics in the MSSP tenant, use **Azure Lighthouse** for cross-tenant access; customer data stays in customer tenants for sovereignty.

*Takeaway:* Map critical detection needs to **MITRE ATT&CK**, design a workspace/data-tier strategy (high-value telemetry in analytics tier, verbose long-retention logs in data lake), then automate low-risk response.

### The SecOps function (MCRA)
Primary objective: **detect, respond to, and recover from active attacks**. Mature SecOps does both **reactive** (respond to tool-detected attacks) and **proactive** (hunt for attacks that slipped past detection). The **[MCRA](https://aka.ms/MCRA)** provides Zero Trust end-to-end design guidance incl. SecOps/SOC diagrams.

- **Zero Trust drives SecOps load:** shifting from trust-by-default to **trust-by-exception** requires attesting to each request's trustworthiness → more signals/alerts → an **integrated platform (SIEM + SOAR + XDR + posture + AI)** is essential to manage the influx.

*Three key outcomes:*
- **Incident management** — respond reactively, hunt proactively, coordinate legal/comms/business implications.
- **Incident preparation** — strategic activities building "muscle memory" and context for future attacks.
- **Threat intelligence** — gather/process/disseminate TI to security + business leadership.

*Team tiers:*
- **Triage (Tier 1)** — frontline, high-volume alert processing (mostly automation-generated); resolves common types, escalates complex/new ones.
- **Investigation (Tier 2)** — deeper, multi-source correlation; builds repeatable solutions so Tier 1 can handle recurrences; responds to business-critical alerts.
- **Hunt (Tier 3)** — proactive hunting for sophisticated attacks, matures controls, escalation point for major incidents + forensics.

*Modernization trends:* SOC elevating to **business risk management**; metrics shifting from "time to detect" to **MTTA** (mean time to acknowledge) + **MTTR** (mean time to remediate); tech evolving from static SIEM log analysis to **AI/ML, behavior analytics, integrated TI**; **AI copilots** (summarize incidents, correlate alerts, NL query generation, recommend actions — lowers expertise barrier); **hypothesis-driven threat hunting**; formalized incident management; integration of **internal business context** (risk scores, data sensitivity, isolation boundaries).

*Key relationships (build before an incident):* IT ops, threat intel, security architecture, insider risk, legal/HR, communications, risk org, industry associations/vendors.

### Monitoring for hybrid & multicloud
Real-time visibility across hybrid, multicloud, and edge from a single control plane. **Monitoring** = "what's happening now?" (real-time detection/response); **logging** = "what happened?" (analysis, compliance, forensics) — complementary.

*Design guidance (MCSB + CAF):* establish a **primary cloud platform** as the enterprise control plane; **unified operations** (one tool/process set across providers); extend governance/ops to on-prem/multicloud/edge; design for **data residency/compliance**.

**Azure Arc (foundation):** projects non-Azure resources into **Azure Resource Manager** → unified resource management, consistent **Azure Policy** enforcement across Azure/on-prem/AWS EC2/GCP VMs, centralized **Azure Monitor**, and cloud-based inventory.

**Azure Monitor (observability):** Metrics (numerical perf data), **VM insights** (OS perf + app discovery on hybrid machines), **Container insights** (Arc-enabled Kubernetes across AKS/EKS/GKE), dashboards/workbooks. The **Azure Monitor agent** deploys to both Azure VMs and Arc-enabled servers using the **same DCRs**.

**Data collection rules (DCRs):** consistent telemetry collection across Azure/AWS/GCP/on-prem, real-time routing to Azure Monitor, and **data transformation** (filter to security-relevant telemetry before ingestion).

**Alerting types:** **Metric alerts** (near-real-time vs static/dynamic ML thresholds), **Log search alerts** (KQL across servers), **Activity log alerts** (resource ops + service health). Alerts trigger **action groups** → notify, run runbooks, or invoke Logic Apps.

**Defender for Cloud (multicloud):** real-time threat detection across Azure/AWS/GCP; workload protection via **Defender plans** (Servers, Containers, Databases); **auto-provisioning** of Arc + Azure Monitor agents to AWS EC2/GCP VMs; alert integration with Azure Monitor + Sentinel.

**Defender for AI Services:** extends threat protection to Azure OpenAI + Azure AI Model Inference; alerts flow to the **Defender XDR portal**. Detects **jailbreak/prompt injection** (incl. ASCII smuggling), **credential theft** via model interactions, **sensitive data exposure**, and **wallet attacks** (excessive resource consumption for financial damage). Include AI-specific detection in the broader SecOps architecture.

**Defender portal = single pane of glass:** correlated incidents across Azure/AWS/GCP/on-prem, real-time dashboards, integrated alert management (Defender for Cloud + Sentinel + Defender XDR) → reduces **MTTD**. Convergence point where infrastructure + M365 productivity monitoring combine.

*Architect-level considerations:*
- **Workspace topology** affects monitoring: separate workspaces force slower cross-workspace queries and risk missing alert context; detection rules work best with all relevant data in one workspace.
- **Agent deployment at scale:** Arc agent must be installed first on non-Azure machines. Enforce via **Azure Policy remediation tasks** (large estates) or **Defender auto-provisioning** (AWS/GCP onboarding). Use **Azure Resource Graph** queries to find coverage gaps (Arc-enabled but missing agent/DCR).
- **Coverage & resilience:** alert on **agent heartbeat failures**; Arc agents need outbound **HTTPS 443**; Azure Monitor agent **caches locally** during disconnects and syncs on reconnect.
- **Network:** use VPN/ExpressRoute/private endpoints; **Private Link** for Log Analytics keeps traffic off the public internet; plan latency vs workspace placement.
- **Design factors:** data residency, latency, cost (centralization vs egress/ingestion), RBAC across teams, scalability. **Azure Lighthouse** enables centralized multi-tenant/subsidiary management with data-sovereignty boundaries.

### Centralized logging & auditing
**Logging** = durable record ("what happened?" / "can we prove compliance?", retained for years); **auditing** = tracks user/admin activities for accountability + regulatory needs.

*Two security data domains converging in Sentinel:*

| Domain | Covers | Tools | Characteristics |
|--------|--------|-------|-----------------|
| Infrastructure | VMs, containers, networks, DBs, cloud, AI services | Azure Monitor, Defender for Cloud, Activity logs | Agent-based, resource-focused, high volume |
| Productivity | Email, files, identity, collaboration, Copilot | Purview Audit, Defender XDR, Entra ID logs | Service-based, user-focused, compliance-critical |

Sentinel is the **convergence point** — correlate an identity attack (Entra/M365) with later infrastructure activity for the complete attack story.

*MCSB logging controls:* **LT-3** enable logging for investigation; **LT-4** enable network logging; **LT-5** centralize log management/analysis (assign data owners, access guidance, retention); **LT-6** configure retention per compliance.

**Log Analytics workspaces** = central repository. Both Azure Monitor and Sentinel store here — **Sentinel is a security solution running on top of a Log Analytics workspace**. Decide what to log, where (single = correlation, multiple = residency/access/cost), and retention per table.

**Azure Monitor logging:** Resource logs (via **diagnostic settings** per resource), Activity logs (subscription-level, auto), Platform metrics, Guest OS logs (via **AMA + DCRs**: Windows Events, Syslog, custom text, IIS). Diagnostic-setting destinations: **Log Analytics** (KQL/correlation/Sentinel), **Storage account** (cheap archival), **Event Hub** (stream to external SIEM), partner solutions. Use **Azure Policy "Deploy diagnostic settings"** to auto-configure logging on new resources.

**Sentinel data connectors:** diagnostic-settings-based (Azure Firewall, Key Vault, Activity), AMA-based (Windows Security Events, Syslog/CEF), **API/service-to-service** (Defender XDR, Entra ID, Office 365 — unique to Sentinel), custom/**Logs ingestion API**. Enabling Sentinel on a workspace adds SIEM (detection, incidents, hunting) to **all** data in it regardless of collection method.
> **Sentinel is moving to the Defender portal — Defender-portal-only after March 31, 2027.** Plan workspace design accordingly.

*Sentinel storage tiers:*
- **Analytics tier** — high-perf querying for real-time analytics/alerting/hunting; **30 days default**, Sentinel solution tables extend to **90 days free**, up to **2 years** at cost. Primary security data.
- **Data lake tier** — cost-effective long-term cold storage, up to **12 years**. Secondary data, compliance, trends.

**Sentinel data lake:** cloud-native, **Parquet** open format, **single copy mirrored from analytics tier** (free when retention matches), storage/compute separation, KQL + Jupyter engines, activity auditing. Can ingest high-volume/low-value logs directly to data lake only. Analytics capabilities: **KQL jobs** (async, can promote to analytics), **summary rules** (scheduled aggregations 20 min–24 hr bins), **search jobs** (up to a year, forensics), **Jupyter notebooks** (Python/ML).

**Microsoft Purview Audit** (M365 productivity):
- **Audit (Standard)** — 180 days; included with most M365 licenses.
- **Audit (Premium)** — 1 year (Exchange/SharePoint/Entra ID), extendable to **10 years** with add-on; high-value forensic events (**MailItemsAccessed, SearchQueryInitiated**) for breach investigation.
- **AI/Copilot auditing** (auto-captured in Standard): **Microsoft Copilot interactions** (included), **connected AI apps** (pay-as-you-go), **third-party AI apps** e.g. ChatGPT (pay-as-you-go). AI records capture sensitivity labels, **jailbreak flags**, agent identity metadata, model transparency → feed **DSPM for AI** in Purview. Factor AI audit volume into retention (esp. EU AI Act windows). Integrate Purview logs into Sentinel for correlation + data-lake retention.

*Architect-level workspace topology:*

| Pattern | When | Trade-off |
|---------|------|-----------|
| Single workspace | Max correlation, smaller orgs | Best visibility; may miss residency/access needs |
| Separate security/operational | Isolated security team, different retention | Dedicated Sentinel (90-day free) but less cross-domain correlation |
| Regional | Data sovereignty | Compliance met, more complexity, limited correlation |
| Dedicated cluster | 100+ GB/day, CMK needed | Better perf/security, higher min commitment |

Combining operational + security data in one workspace = better visibility, commitment-tier savings, simpler management; use **table-level RBAC** to protect sensitive tables.

*Multitenant/MSSP:* **Distributed** (workspace per tenant + **Azure Lighthouse** cross-tenant; full isolation), **Centralized** (single provider workspace; simple, cross-customer analytics), **Hybrid** (customer workspaces export aggregates centrally). Note: **diagnostics-based connectors can only send to workspaces in the same tenant** as the resource.

*Access control layers:* workspace-context (SOC analysts, all data), resource-context RBAC (app teams see only their resources' logs), table-level RBAC (restrict e.g. `SecurityEvent`). Recommendation: put the Sentinel workspace in a **dedicated subscription** for permission isolation.

*Data tiering by classification:* primary security data (alerts/incidents/identity) → **analytics tier** (90 days–2 yrs); secondary (firewall/proxy/NetFlow) → **data lake + summary rules** (1–7 yrs); compliance/audit → analytics **mirrored to** data lake (7–10 yrs); high-volume low-value → **data lake only**, query on-demand with KQL jobs.

### Detection & response with XDR + SIEM
**XDR** = unified incident platform using AI/automation for **deep, coordinated** detection/investigation/response across endpoints, identities, cloud apps, email, OT. **SIEM** = collects/normalizes/analyzes org-wide telemetry in near real-time for **broad** visibility. Complementary: XDR = deep across productivity domain; SIEM = correlates **both** productivity + infrastructure domains.

*Design guidance:* integrate XDR + SIEM; prioritize **high-fidelity alerts** (reduce fatigue); enable **automatic attack disruption** (contain at machine speed); unified incident management; TI integration; AI-assisted detection.

**Microsoft Defender XDR product mapping:**
- Identity threats → **Defender for Identity**
- Email/URL/collaboration → **Defender for Office 365**
- Endpoints → **Defender for Endpoint**
- OT/IT → **Defender for IoT**
- Asset/device posture → **Defender Vulnerability Management**
- SaaS app access → **Defender for Cloud Apps**
- Azure/AWS/GCP workloads → **Defender for Cloud**

Auto-correlates alerts into **incidents** with end-to-end attack-chain visibility.

**Microsoft Sentinel as SIEM + platform:** evolved into an AI-ready, data-first platform — turns telemetry into a **security graph**, standardizes access for AI agents, coordinates autonomous actions (humans stay in command). **350+ connectors**; built-in analytics/ML/TI; SOC tools (unified incidents, entity pages, investigation graph, hunting notebooks, Security Copilot).

*Core platform components:*
- **Data lake** — unifies/retains/analyzes at scale (12 yrs); multi-modal (KQL, graph, notebooks) on single open-format copy.
- **Graph** — models relationships across assets/identities/activities/TI → reason about attack paths from compromised entity to critical asset.
- **MCP server** — hosted interface for **natural-language** interaction + building security agents (no schema/coding needed).
- **Content Hub** — pre-built solutions, analytics rules, playbooks, workbooks.

**AI-powered detection:** **UEBA** (behavioral baselines → anomaly/insider detection), **ML anomaly detection** (no predefined rules), **TI correlation** (IOC matching at scale), **attack chain detection** (correlate low-fidelity signals into multi-stage attacks).

**Security Copilot for SOC:** incident summarization (hours→minutes), guided investigation, natural-language queries (vs KQL), script analysis, report generation. Architect note: **Copilot can only analyze data in your workspace** — ensure comprehensive collection + train teams to prompt well.

**Unified Defender portal** combines Defender XDR + Sentinel + Security Exposure Management + Security Copilot (SIEM + SOAR + XDR + posture + cloud security + TI + genAI). Integration benefits: shared data/unified incidents, **attack disruption signals** from XDR available in Sentinel, Copilot AI summarization, XDR depth + SIEM breadth. *(New customers auto-onboarded; Sentinel Defender-portal-only after March 31, 2027.)*

*Detection engineering — analytics rule types:*
- **Scheduled** (custom KQL) — frequency vs cost, lookback vs latency
- **NRT (near-real-time)** — sub-minute, limited data sources, higher resource use
- **Microsoft security rules** — auto incidents from Defender alerts; filter to avoid duplicates
- **ML behavior analytics** — needs baseline data, tune sensitivity
- **Threat intelligence rules** — IOC matching; keep feeds current

*Coverage strategy:* map detections to **MITRE ATT&CK** (Sentinel coverage matrix for gaps), **layer** rule + ML + TI detections, prioritize high-impact techniques for your industry, plan ongoing **tuning/maintenance**.

### SOAR (security orchestration, automation, and response)
Automates recurring, predictable enrichment/response/remediation so overwhelmed SOCs stop ignoring alerts and free time for deep investigation/hunting.

*Design guidance:* start with **high-value, low-risk** use cases (clear procedures, low variation, low false-positive rate); **human approval** for high-impact actions (account disable, network isolation); **binary decision criteria** for predictability; ensure signal accuracy (watchlists, reliable TI); **augment** analysts, don't replace judgment.

**Unified SOAR in Defender portal** (Sentinel + Defender XDR automation):
- **Automation rules** — orchestration layer: tag/assign/close incidents, create tasks, trigger playbooks. Triggers: **incident created** (triage/assign/notify), **incident updated** (escalation/sync), **alert created** (Scheduled/NRT alerts not creating incidents). Incident-trigger rules run on **both** Sentinel + Defender XDR incidents; alert-trigger rules run on **Sentinel alerts only**. Up to **10 min** alert→execution.
- **Playbooks** — multi-step workflows on **Azure Logic Apps**. Use cases: enrichment, bi-directional sync (ServiceNow/Jira), orchestration (Teams/Slack notify), response (isolate/block). Hosting: **Standard** (single-tenant, high-volume, predictable pricing) vs **Consumption** (multitenant, pay-per-execution). Templates via Content Hub / Automation page / GitHub.
- **Action Center** — central view of automated + manual remediation with approval workflows.

**Defender-native automation** (on that product's alerts):
- **AIR (automated investigation & response)** — virtual Tier 1/2 analyst: investigates, sets verdict (malicious/suspicious/none), takes/recommends remediation. Applies to Endpoint (devices), Office 365 (email/content), Identity.
- **Automatic attack disruption** — real-time containment via high-confidence XDR signals: isolate device, disable user, block IP (ransomware, BEC).

*Architect considerations:* **permissions** — Sentinel uses a dedicated service account; grant **Microsoft Sentinel Automation Contributor** on the playbook resource group (more secure than user-based execution). **MSSP patterns:** centralized (playbooks in provider tenant + **Azure Lighthouse**, protects IP), distributed (per-tenant, data sovereignty, more overhead), hybrid (core central + customer-specific customizations, cross-workspace analytics rules).

### MITRE ATT&CK — threat detection coverage
Public knowledge base of attacker **tactics** (goals) and **techniques** (methods) from real-world observation; used as a common language and to find detection gaps.

*Matrices (evaluate per your tech footprint):*
- **Enterprise** — corporate networks (Windows/macOS/Linux, network infra, containers)
- **Cloud** — cloud-native techniques (Azure/AWS/GCP, M365, SaaS/IaaS, IdPs); **integrated within Enterprise** but cloud-focused
- **Mobile** — iOS/Android
- **ICS** — OT, SCADA, industrial processes

*Evaluate coverage:* map existing detections to techniques → identify gaps → prioritize by risk/threat landscape → incremental improvement.

**Sentinel MITRE page:**
- **Active coverage** — how many active Scheduled/NRT rules cover each technique.
- **Simulated coverage** — detections available but not configured (Content Hub analytics templates, convertible hunting queries, solution-specific detections) → shows possible posture. Technique pane shows active-out-of-available.
- **SOC optimization recommendations** — threat-based scoring (High/Med/Low by % rules activated), spider charts, threat-scenario view, AI tagging recommendations (preview), risk-based recommendations (preview).
- MITRE techniques auto-added to incidents (attack-stage context) and selectable in analytics rules + hunting queries.

**MITRE ATLAS** (Adversarial Threat Landscape for AI Systems) — extends threat modeling to **AI-specific** techniques: recon on ML artifacts, adversarial examples, ML supply-chain compromise, backdoor models, model inference API access, **model/training-data extraction**, evade/erode ML model integrity. Evaluate coverage for both ATT&CK and ATLAS where AI workloads exist. For ICS, use **Defender for IoT** integrated with Sentinel.

### 2.1 objective checklist (all covered above)
- ✅ Detection & response with XDR and SIEM
- ✅ Centralized logging and auditing, including Microsoft Purview Audit
- ✅ Monitoring for hybrid and multicloud environments
- ✅ SOAR, including Microsoft Sentinel and Microsoft Defender XDR
- ✅ Security workflows: incident response, threat hunting, incident management
- ✅ Threat detection coverage using MITRE ATT&CK matrices (Enterprise, Cloud, Mobile, ICS)

## 2.2 Solutions for identity and access management

### Agent identities — Microsoft Entra Agent ID + Conditional Access
Applying **Conditional Access (CA)** controls to AI agent identities in **Microsoft Entra Agent ID** to limit what agents can access, when they can authenticate, and how to restrict/remove them safely. Goal: reduce the blast radius from a compromised or over-privileged agent while preserving business continuity during decommissioning.

**Authentication patterns (determine the CA enforcement point).** CA must be scoped to the token subject:

- **On-behalf-of (OBO)** — agent acts for a signed-in human; scope CA to the **user**.
- **Autonomous / application-only (client credentials)** — no human involved; scope CA to the **agent service principal**.
- **Agent user account** — agent has its own user identity; scope CA to that account.

**CA for autonomous agents = non-interactive conditions only.** No human responds to prompts, so you **cannot require MFA or device compliance**. You *can*:

- **Block sign-in**
- Restrict by **named locations** (IP ranges / Azure regions)
- Apply **sign-in frequency**
- Apply **risk-based blocks**

**Policy design & safe rollout:**
- Create **descriptive, agent-scoped** policies.
- Validate in **report-only mode for 24–48 hours**, reviewing sign-in logs before enforcing.
- Flip to **On** to enforce.
- **Mis-scoped block policies can break production services** — report-only testing is essential.

**Scale targeting (avoid enumerating every service principal):**
- Use **custom security attributes** (e.g., `DataSensitivity`) to group agents.
- Or apply policies to **agent identity blueprints** so new agents are covered automatically.

**Lifecycle & governance:** combine CA block policies with product-level controls — **licensing, RBAC, DLP**, ability to **disable individual agents**, **audit-log monitoring** for Add/Update/Delete service principal events, and **periodic access reviews** to prevent sprawl.

*Examples:*
- **Legacy agent decommissioning:** find obsolete Copilot Studio agent by **object ID** → CA policy scoped to that service principal → **report-only** to confirm only intended agent matches → flip to **On** to block auth and stop workflows **without deleting the identity**.
- **Preventing agent sprawl:** remove **Power Platform Administrator** role from general users, configure a **Copilot Studio approval workflow**; approved agents inherit **security attributes** that auto-include them in restrictive CA policies.

*Takeaway:* Map each critical agent to its auth pattern, then deploy a targeted CA policy in **report-only mode** scoped to the correct token subject — prevents outages while giving logs to tune enforcement.

**Mapping authentication flows to CA scope (detail).** CA evaluates policies **at token issuance time** — before the agent/user receives a resource token. The **assignment scope** determines which auth events trigger evaluation.

*Three patterns → token subject → CA scope:*

| Pattern | Token subject | CA assignment scope | Interactive controls (MFA, device) |
|---------|--------------|---------------------|-----------------------------------|
| On-behalf-of (OBO) | The signed-in **user** | Users or groups | **Available** — user present |
| Application-only (autonomous) | The **agent identity** (service principal) | Agent identities | Not applicable |
| Agent's user account | The agent's **user-type identity** | That user account | Not applicable (non-interactive) |

- **OBO (most common):** user signs in to the agent app → agent exchanges the user's token for a resource-scoped token **issued to the user**; user's identity/permissions constrain the agent. Full user CA set applies (MFA, device compliance, sign-in risk, session controls). Target resources = the corporate resources the agent accesses, so every token exchange is evaluated.
- **Autonomous:** authenticates as its service principal via **OAuth 2.0 client credentials** (certificate, managed identity token, or client secret). Efficient for scheduled/event-driven tasks but removes interactive checkpoints.
- **Agent user account:** e.g., a digital worker with a dedicated mailbox in team workflows; authenticates as a user, target that account with standard user-targeted config.

> Wrong scope = missed enforcement: a user-targeted policy on an autonomous agent misses the enforcement point; an agent-targeted policy on an OBO flow won't intercept the auth that matters.

*Configuring autonomous CA scope in the portal:* CA policy → **Assignments > Users, agents or workload identities** → **Agents**. **All agent identities** covers every agent service principal; or target specific agents or their **parent blueprint**.

*Scalable targeting (two approaches):*
- **Attribute-driven:** custom security attributes (key-value, e.g., `DataSensitivity = Confidential`) → write a CA condition targeting that value. Applies automatically to matching agents, **including future ones** — no policy update.
- **Blueprint-level:** apply the policy to a **parent agent identity blueprint** → auto-covers all agents derived from it. One policy enforces consistent controls across a project/product's agents.

*CA enforcement boundaries for non-interactive flows* (apply to **both** autonomous service principals **and** agent user accounts):
- **Can enforce:** block sign-in; named location conditions (IP ranges / Azure regions); sign-in frequency (token refresh intervals); risk-based conditions (block agents flagged high-risk by Entra ID Protection).
- **Cannot enforce:** MFA (no user to prompt); device compliance (no registered device); user-based session controls (need an interactive session).
- **OBO is the exception** — full user control set available because the human is the token subject throughout.

**Configuring CA policies for agents (procedures).**

*Licensing:* CA for agents requires **Entra ID P1** or **M365 E3**. Agent ID is part of **Microsoft Agent 365** — using Agent ID features needs a **Microsoft Agent 365** or **M365 E5** license; specific security features carry their own tiers.

*Five-step policy lifecycle:* **Define scope** (all agents or specific service principals) → **Set conditions** (named locations, sign-in risk, app filters) → **Choose access control** (block / grant with conditions / session) → **Validate** (report-only, review sign-in logs) → **Enforce** (switch to On after validation).

*OBO flow config:* Protection → **Conditional Access > Policies** → New policy → **Assignments > Users, agents or workload identities** → **Users and service principals** → Include users/groups who interact with the agent → **Target resources** = resources the agent accesses (e.g., Microsoft Graph, SharePoint Online) → **Conditions** (named locations, sign-in risk) → **Grant** controls (Require MFA, Require compliant device) → **Report-only**, review 24–48 hrs before enforcing. Recommended OBO baselines: **block legacy auth, require MFA on elevated sign-in risk, require MFA for guest users** accessing agent-connected resources.

*Autonomous (agent identity) config:* New policy with a **descriptive name** (e.g., "Block Copilot Studio agent - Legacy compliance project") → Assignments → **Users, agents or workload identities** → Include **All agent identities** or **Select agent identities** (by name / object ID).
- **Posture choice:** block-all-by-default + selectively allow = secure-by-default but higher overhead; block-specific-agents = flexible but needs ongoing monitoring for new unauthorized agents.

*Agent-supported conditions* (evaluate at token issuance, no interactive prompt):
- **Named locations** — most common; restrict to corporate IP ranges / Azure regions / datacenters (Contoso restricts agents to East US + West Europe).
- **Sign-in risk** — via Entra ID Protection; block high-risk agent sign-ins.
- **Application filters** — target by resource accessed (e.g., block agents from Microsoft Graph but allow Power Platform APIs) to limit blast radius.

*Access controls (Grant):*
- **Block access** — decommission agents or enforce allow-list model.
- **Grant access** — rare for agents; adds no value over no policy.
- **Grant with controls** — most applicable is **Require compliant network** (via named locations).
- **Session:** limited for agents, but **sign-in frequency** enforces token expiration to shrink the stolen-token window.

*Report-only validation:* evaluates the policy against every auth request and **logs what would happen without blocking**. Review **Entra ID > Monitoring > Sign-in logs** filtered by Conditional Access for **24–48 hrs** (full activity cycle). Check for unexpected matches and confirm intended agents are scoped correctly before switching to **On**. Targeting the wrong service principal in a block policy can break production with no warning.

*Contoso decommissioning example:* identify legacy agent by **object ID** → policy scoped to that service principal, **no conditions, Block access**, report-only → 2 days of logs confirm only that agent matches (3 auth attempts/day from East US) → switch to **On** → next attempt blocked, obsolete workflows safely decommissioned without disrupting other agents.

**Controlling agent access & lifecycle.** Three disabling approaches (layered defense):

| Approach | Scope | When to use |
|----------|-------|-------------|
| CA block | Blocks auth for existing + new agents | Primary enforcement — prevents token issuance |
| Block agent creation per product | Prevents new agents in specific products | Reduces sprawl; enforces approval workflows |
| Disable individual agents | Deactivates a specific agent identity | Targeted decommissioning / incident response |

> Key gap: **CA blocks authentication but doesn't prevent agent creation** — unauthorized agents can still appear and consume directory quota even if they can't authenticate. Combine CA with product-level creation restrictions.

*Block creation per product:*
- **Microsoft Agent 365** (GA) — centralized control plane for agent governance across products. M365 admin center → **Agents > Settings**: **Allowed agent types**, **Security templates** (bundle CA policies + custom security attributes as presets on every new agent), **User access controls**, and **Agent management rules** (bulk governance, e.g., auto-reassign ownership when a creator leaves). **Recommended starting point** before per-product restrictions.
- **Copilot Studio** — combine **licensing + RBAC**: block free-trial signups and remove **Power Platform Administrator** role. Alternatively **DLP policies** block *publishing* (agents can be built but not deployed — allows experimentation with production control).
- **Azure AI Foundry** — control at each hierarchy layer: subscription creation (**Billing/Account Administrator**), Foundry projects (**Azure AI Account Owner**), agents (**Azure AI User**). Don't assign broadly.
- **Security Copilot** — remove users from **Owner/Contributor** in workspaces; Microsoft-owned agents need **Security Administrator** or **Identity Governance Administrator** to enable. Deleting all **SCU capacity** disables Security Copilot entirely (blunt — kills the product too).
- Best combined with **approval workflows** (e.g., ticket request before granting creation roles).

*Disable individual agent identities:* Entra admin center (as **Agent ID Administrator**) → **Entra ID > Agents > Agent identities** → select → **Disable** (blocks token issuance for that agent, no CA policy needed). Disable a **parent blueprint** to disable all derived agents. Product-specific interfaces (e.g., Power Platform admin center) can show dependent workflows/connectors first. Ideal for IR: disable → investigate → rotate credentials → re-enable.

*Monitor lifecycle in audit logs:* Entra ID → **Monitoring > Audit logs**, filter **Category = ApplicationManagement**. Agents appear as **service principals**, so verify: check **agentType** on the `targetResources` field — anything other than `notAgentic` = agent. Or query Graph by object ID.
- Key events: **Add service principal** (created), **Update service principal** (config/permissions changed), **Delete service principal** (removed), **Add app role assignment to service principal** (granted resource access).
- **Agent *user accounts* appear as `Create user`, not a service principal event** — filter for both to cover both identity types.
- Automate alerts via **Logic App** or **Sentinel playbook** on new agent service principals.

*Access reviews for agent identities:* Entra ID → **Identity Governance > Access reviews** → new review → scope = **Workload identities**, specify agent service principals → set frequency (monthly/quarterly) + reviewers (business owners/project managers). Reviewers approve (keep) or deny (deactivate). Prevents agent accumulation; provides a governance checkpoint. (Contoso: monthly review of Copilot Studio agents by the compliance-automation lead; unconfirmed agents flagged and disabled.)

### Identity & access management (non-agent)

**Access to SaaS/PaaS/IaaS/hybrid/multicloud** — enforce access through three control layers: **identity** (Entra ID as the control plane, CA, MFA), **network** (Private Link/private endpoints, NSGs, firewall), and **application** (per-app roles, app proxy). Prefer identity-based auth (managed identities, workload identities) over keys/secrets everywhere.

**Entra ID hybrid & multicloud:**
- Hybrid sync options: **Password Hash Sync (PHS)** (simplest, recommended), **Pass-through Auth (PTA)** (on-prem validates passwords), **Federation/AD FS** (legacy, most complex). **Entra Connect / Cloud Sync** does the syncing (Cloud Sync = lightweight agent, multi-forest).
- Multicloud: use Entra ID as the single identity provider for AWS/GCP; **Defender for Cloud** + **Permissions Management (CIEM)** for cross-cloud entitlements.

**External identities:** **B2B** (invite guests, they use their own identity/home tenant), **B2B direct connect** (Teams shared channels), **Entra External ID** (formerly Azure AD B2C) for customer-facing apps, and **decentralized identity / Verified ID** (issuer→holder→verifier model, user holds credentials in a wallet). **Cross-tenant access settings** govern inbound/outbound B2B trust (incl. trusting other tenants' MFA/device claims).

**Modern authN/authZ strategy:**
- **Conditional Access** = the Zero Trust policy engine: signals (user, device, location, app, risk) → decision (block / grant + controls like MFA/compliant device / session).
- **Continuous Access Evaluation (CAE)** — revokes access **near-real-time** on critical events (user disabled, password reset, risk detected) instead of waiting for token expiry.
- **Risk scoring** — **Entra ID Protection** computes **sign-in risk** (this auth) and **user risk** (account compromise); feed into risk-based CA policies.
- **Protected actions** — require step-up CA (e.g., MFA) before high-impact directory operations (e.g., changing CA policies).
- **Authentication strengths** — require phishing-resistant methods (FIDO2, certificate, passkeys) for sensitive resources.

**Harden AD DS (on-prem):** tiered admin / enterprise access model, limit Domain Admins, **LAPS** for local admin passwords, disable legacy protocols (NTLMv1, SMBv1), **Defender for Identity** to detect AD attacks (pass-the-hash, DCSync, Golden Ticket), regular attack-path review. AD DS ≠ Entra ID (Entra is not a domain controller replacement).

**Secrets, keys, certificates:** centralize in **Azure Key Vault** (Standard = software; **Managed HSM** / Premium = FIPS 140-2/3 hardware). Prefer **managed identities** so apps never hold secrets; enable **soft-delete + purge protection**; use **Key Vault-managed rotation** and RBAC (not access policies) for granular control.

### Decision table — identity & access

| Scenario | Service / Feature |
|----------|-------------------|
| Simplest reliable hybrid password auth | Password Hash Sync (Entra Connect) |
| On-prem must validate passwords in real time | Pass-through Authentication |
| Lightweight, multi-forest sync | Entra Cloud Sync |
| Invite partners using their own credentials | Entra B2B collaboration |
| Customer-facing (CIAM) sign-up/sign-in | Entra External ID |
| User-held, verifiable credentials in a wallet | Verified ID (decentralized identity) |
| Revoke access instantly on risk/password change | Continuous Access Evaluation (CAE) |
| Detect compromised accounts / risky sign-ins | Entra ID Protection (risk scoring) |
| Require MFA before editing CA policies | Protected actions |
| Store app secrets/keys/certs centrally | Azure Key Vault (Managed HSM for FIPS) |
| Detect on-prem AD attacks (DCSync, Golden Ticket) | Defender for Identity |

## 2.3 Solutions for securing privileged access

### Privileged Identity Management (PIM) & Just-in-Time (JIT) access
Why standing privilege is dangerous and how PIM/JIT counters it.

**Standing privilege = persistent attack surface.** A privileged role assignment that's **permanently active** whether or not it's being used. Cloud environments create it by default when admins get role assignments **directly** — no expiration, no justification. Because the role is always active, its credentials are always a target.
- The attacker's **opportunity window = the lifetime of the assignment**. An account standing for 18 months = 18 months to break in (phishing, credential stuffing, token theft). Risk persists even when the account is idle.
- Classic example: a **Global Administrator** account created for a migration project, left permanently assigned after the team moves on.

**Blast radius = scope an attacker can reach with a compromised identity.** For a standing Global Admin in Entra ID: user accounts, group memberships, app registrations, federated trust settings, and every linked Azure subscription. Other high-blast-radius roles: **Privileged Role Administrator, Security Administrator, User Access Administrator** — each a permanent target when left standing.
- Extends to the **control plane** (management layer where resources are created/configured/deleted). A standing **Owner** on a subscription = control over every resource: VMs, storage accounts, key vaults, network config.
- **AI/automation force multiplier:** AI agents, model endpoints, and pipelines run under the permissions of the identity that configured them. A compromised standing Global Admin can redirect agent behavior, exfiltrate data the agent processes, or use the agent to escalate to other systems.

**JIT access = the response.** Elevated permissions you *don't* hold persistently — activate for a specific task, expire after a defined period. Reduces risk via three properties:
- **No access by default** — no persistent credentials to discover/steal between sessions.
- **Intentional activation** — a deliberate action to elevate; every use is an explicit, auditable event, not an ambient condition.
- **Automatic expiration** — even a compromised session's window closes when it ends, not when someone notices.

**Microsoft Entra Privileged Identity Management (PIM)** is the service that implements JIT access for both **Entra ID roles** and **Azure resource roles**.

*Takeaway:* Eliminate standing privilege on high-blast-radius roles by making them **eligible** (JIT via PIM) rather than **active** — shrinks both the attack window and the auditability gap.

**PIM core capabilities.** Applies equally to **Entra ID roles** and **Azure resource roles** — mechanics are consistent across both.

*Eligible vs active assignments:*
- **Eligible** — you hold the *entitlement* but not the role until you explicitly **activate** it (request + receive temporary elevated permissions). This is JIT in practice: permissions exist as potential, not active reality. Expiration configurable (assignment expires, or session expires after activation duration). **Default posture for most privileged roles.**
- **Active** — role assigned directly, access is live immediately, **no activation step**. Can be time-bound with a set expiration. Reserve for cases where activation is impractical: **break-glass emergency accounts**, or tightly scoped temporary roles with a defined end date.

*Activation controls (checkpoint between entitlement and live session — configurable per role):*
- **MFA verification** — confirms identity before elevation; stolen credentials alone aren't enough.
- **Justification** — written rationale required before the session; creates an auditable record of *why*.
- **Approval** — a delegated approver must confirm the request; human checkpoint for sensitive roles.
- **Activation duration** — max time window, **1–24 hours**, after which PIM auto-removes the role, limiting the exposure window.

Some roles require all four; others require only MFA. Match controls to role sensitivity to balance friction vs practicality.

*Visibility & accountability after access:*
- **Email notifications** to the activating user and any configured approver — immediate awareness an elevated session is open.
- **Audit log entry** per activation: who activated, which role, session start, duration, and justification. Accumulates into a durable record of privileged activity — **supports regulatory requirements** for privileged access monitoring (evidence of who held elevated access and when).
- **Access reviews** — periodic process where designated reviewers confirm/deny whether each eligible assignment should continue.

**Implementing JIT for Entra ID roles (tenant-directory plane).**

*Licensing:* PIM requires **Entra ID P2** or **Entra ID Governance** license per managed user.

*Roles that should never be permanent (JIT-only tier):*
- **Global Administrator** — broadest blast radius; full control over all directory settings/services
- **Privileged Role Administrator** — can modify any role assignment, including its own
- **Security Administrator** — controls tenant-wide security policy
- **Exchange Administrator** — full access to email/calendar infrastructure
- **Application Administrator** — can register apps and modify application credentials
- **Authentication Policy Administrator** — controls auth methods, tenant-wide MFA settings, password protection policy

> **Break-glass safeguard:** before making Global Admin eligible, verify **at least two emergency access (break-glass) accounts** hold a **permanent active** assignment to the role, with credentials stored securely offline. Otherwise a misconfig or IdP outage could lock every admin out with no recovery path.

*Assign eligible access:* Entra admin center → **ID Governance > Privileged Identity Management** → **Microsoft Entra roles** → select role → **Assignments > Add assignments** → set **Assignment type = Eligible** → select member, set duration (permanent or time-bound) → **Assign**. Converts standing access to a JIT entitlement (appears in eligible assignments, no active privilege until activation).

*Role settings vs assignments (key distinction):*
- **Role settings** = per-role config of conditions any eligible user must satisfy (MFA, justification, approval, duration). Apply uniformly to all eligible users of that role. Configuring them creates **no** assignments.
- **Assignments** = the individual eligible/active grants to specific users.
- Reach via PIM → Microsoft Entra roles → select role → **Role settings > Edit**.

*Recommended activation controls by privilege tier:*

| Tier | Example roles | MFA | Justification | Approval | Max duration |
|------|---------------|-----|---------------|----------|--------------|
| Critical | Global Admin, Privileged Role Admin | Required | Required | Required | 1–2 hours |
| High | Security Admin, Privileged Authentication Admin | Required | Required | Recommended | 4–8 hours |
| Moderate | Exchange Admin, App Admin | Required | Required | Optional | 8 hours |

*Activate (user perspective):* PIM → **My roles > Eligible assignments** → find role → **Activate** → set duration (≤ role-settings max) → enter justification → if approval required, submit and wait → confirm role under **Active assignments** with countdown. When duration elapses, session expires and reverts to **eligible automatically** — no manual cleanup. For emergencies, use a break-glass account rather than an approval-gated activation.

*Scope note:* Entra roles are **tenant-scoped** (directory-level identities/configs); **Azure resource roles** are narrower-scoped (subscription, resource group, or individual resource). PIM secures both planes.

**Implementing JIT for Azure resource roles (control plane).**

*Scope hierarchy:* Azure RBAC nests **management group → subscription → resource group → individual resource**; assignments **inherit downward** (Owner at subscription = ownership of everything beneath). PIM surfaces this hierarchy — make a user eligible at any level. **Assign eligibility at the narrowest scope** that fits the task (least privilege limits blast radius from a compromised activation).

*Navigation & discovery:* PIM → **Azure resources** (managing RBAC on control-plane objects, not directory identities). Updated PIM uses the latest ARM API and **auto-surfaces resources — no manual onboarding**. If subscriptions/resource groups don't appear, use the one-time **Discover resources** step.

*Entra roles vs Azure resource roles:*

| | Entra roles | Azure resource roles |
|---|---|---|
| Navigation | PIM → Microsoft Entra roles | PIM → Azure resources |
| Scope | Tenant (directory-wide) | Subscription / resource group / resource |
| Discovery | No | No (auto-managed) |
| Example roles | Global Admin, Security Admin | Owner, Contributor, Key Vault Administrator |

*Risk-tiering resources* — controls depend on four factors: **data sensitivity, blast radius, regulatory exposure, reversibility of damage.**

| Tier | Resource examples | Controls |
|------|-------------------|----------|
| Critical | Prod subscription, Key Vault, Azure AI services | MFA + justification + **approval required**, 1–2 hr max |
| High | Prod resource group, Azure SQL, AKS | MFA + justification, approval optional, 4–8 hr max |
| Standard | Dev/test resource groups, sandbox subs | MFA + justification, no approval, up to 8 hr |

- **Key Vault = Critical:** a compromised Owner exposes every secret, including credentials to downstream services → cascading, silent propagation across dependent systems before IR detects the breach.
- **Azure AI services = Critical:** model weights + training data are exfiltratable IP that may not surface in logs immediately.

*Assign eligible access:* PIM → Azure resources → navigate to target (via Subscriptions/Resource groups dropdown) → **Manage > Roles > Add assignments** → pick role (e.g., Key Vault Administrator) → **Assignment type = Eligible** → member + duration → **Assign**. Activate via **My roles > Azure resources tab** (same flow as Entra roles).

*Per-scope role settings (key nuance):* Azure resource role settings are configured **independently per role per resource scope**. The same **Contributor** role can have stricter controls (shorter duration, mandatory approval) at subscription level vs looser settings at resource-group level — avoids the trade-off of over-enabling prod or over-restricting dev. Set via PIM → Azure resources → select resource → **Settings** → select role → **Edit**.

**Scaling with PIM for Groups.**

*The scale problem:* Direct per-user JIT works for 1–2 users, but each new member = a separate eligible assignment with its own settings, max duration, and approvers. This causes **configuration drift** — a policy change (e.g., 4 hr → 1 hr max) must be applied to every assignment; miss one and enforcement is uneven. The real risk at scale is **nonuniform settings auditors flag and attackers probe**, not just overhead.

*How it works:* PIM for Groups shifts the eligible resource from a **role** to a **group membership**. Create a group → assign the group the role(s) (e.g., Key Vault Contributor) → make each engineer eligible for **membership**. Activating membership grants **all roles the group holds**; on expiry, membership ends and access is revoked automatically.

*Use a role-assignable group (best practice for sensitive access):* A standard Entra group can be modified by its owners or a **Groups Administrator** with **no PIM approval/audit entry** — a bypass path. A **role-assignable group** raises the bar: membership changes require at least a **Privileged Role Administrator**.

*Member vs Owner eligibility:* **Member** = gets the group's assigned roles (most JIT scenarios). **Owner** = control over the group itself.

*Why it enforces policy better:* Define role settings **once at the group level**; every eligible member inherits them, and a policy change propagates in a single update.

| Dimension | Direct eligible assignment | PIM for Groups |
|-----------|---------------------------|----------------|
| Policy control | Per assignment | Per group (uniform) |
| Audit surface | N individual assignments | One group + membership history |
| Onboarding | New eligible assignment each | Add to eligible group membership |
| Policy drift risk | High at scale | Low (single settings source) |

*When to choose group-based JIT* — when **2+ of these** are true: more than a few users need the same access pattern; access recurs regularly; permissions span multiple roles/resources that belong together. **Direct assignment** stays right for a single user needing one-time access to one resource.

**JIT for AI workloads, agents, and applications — the two-track model.**

*AI control plane = Critical-tier attack surface.* Azure OpenAI and Azure ML control planes carry the same risk tier as prod subscriptions/Key Vault. A compromised **Cognitive Services OpenAI Contributor** can enumerate deployed models, redirect endpoint configs, exfiltrate model weights from connected storage, and inspect fine-tuning datasets — all via standard API calls that may not trip routine alerts. Standing human access is unacceptable (unlimited exploitation window, disproportionate damage).

*Track 1 — humans: PIM + Groups.* Assign the AI control-plane roles (Cognitive Services OpenAI Contributor, Azure ML workspace roles) to a **role-assignable group**; make each engineer eligible for membership. One policy, one duration, one approvers list, one audit log across a small, well-defined set. Adding an engineer = adding eligible membership, no new config surface.

*Track 2 — workload identities: permanent scoped RBAC.* Applications, automation pipelines, and AI agents authenticate **non-interactively** — no sign-in prompt, no session, no MFA, no point for an approval workflow to pause. **This is a mechanism constraint, not a policy gap:** PIM activation is what elevates eligible→active, and a non-interactive identity can't activate. There is no PIM config that makes a workload identity eligible.

*Managed identity vs service principal:*

| | Managed identity | Service principal |
|---|---|---|
| What | Identity bound to an Azure resource, platform-managed | App/service identity registered manually in Entra ID |
| Credentials | None to manage — platform auto-rotates | Must manage a secret or certificate |
| Typical use | Azure-hosted workloads (VMs, functions, containers) | Apps outside Azure, or tools needing custom auth |
| PIM eligible? | **No** (non-interactive) | **No** (same limitation) |
| Access model | Permanent, scoped RBAC | Permanent, scoped RBAC |

For workload identities: **permanent RBAC scoped as narrowly as possible** to the specific resource + operation — least privilege enforced by **scope, not time**.

*Example:* AI pipeline agent needs to write inference outputs to Blob Storage → **not** PIM-eligible. Assign a **permanent Storage Blob Data Contributor** scoped to the specific container. PIM governs the **human engineer** who configures that RBAC assignment, not the managed identity doing the writes.

> **Clean two-track rule:** PIM governs the **humans** who configure/deploy/manage AI services. RBAC governs the **workload identities** that run them.

**JIT design patterns & best practices.**

*Privilege as an architectural decision:* JIT is a **posture, not a feature toggle**. Every assignment = a deliberate decision on who, to what, how long, under what conditions. Goal is **not zero standing access** (sometimes permanent active is correct) but **zero *unjustified* standing access**. Reframes PIM from a compliance checkbox to a design tool.

*Design patterns:*

| Pattern | When to apply | Prevents | Trade-off |
|---------|--------------|----------|-----------|
| Eligible by default, activate when needed | All roles except break-glass & workload identities | Standing privilege, unlimited exploitation window | Activation friction for routine tasks |
| Short activation windows for production | Critical/High-tier (prod subs, Key Vault, AI control planes) | Prolonged access after task completes | Requires reactivation for extended work |
| Approval for high-privileged roles | Critical-tier where a single activation = irreversible damage | Unilateral privilege escalation | Approval latency; needs designated approvers |
| Regular access reviews | All eligible assignments, **quarterly minimum** | Privilege accumulation; stale assignments | Reviewer overhead |
| Emergency access excluded from JIT | Break-glass accounts only | JIT dependency failure during an outage | Must compensate with max monitoring + documented procedure |

Patterns combine per tier: Critical = short window + mandatory approval; High = short window, no approval; Standard = eligible assignment alone.

*The break-glass exception:* Excluded from JIT because PIM's activation path (MFA, approver availability, service health) can **itself fail during the outage** that requires emergency access. Governance shifts to: **zero normal use, maximum alerting, and a documented procedure rehearsed/audited on a fixed schedule.** Never for routine tasks — any activation should trigger immediate investigation.

*Overarching principle:* A privileged access strategy holds together because the **decisions behind the controls can be explained and defended**, not because every control is simply switched on.

### Privileged access (beyond PIM)

**Enterprise access model** (replaces the old tier 0/1/2 AD model) — organize privileged access by **control plane** (identity/management: Entra roles, subscriptions) vs **data/workload plane**, plus **management, user/app access**. Principle: privileged access originates only from **secured, isolated** paths so a compromised workload can't reach admin control.

**Entra ID governance suite:**
- **PIM** — JIT elevation (covered above).
- **Entitlement management** — **access packages** bundle roles/groups/apps with policies + approval + expiry; ideal for onboarding and external-user access at scale.
- **Access reviews** — periodic recertification of group/role/app/guest access.
- **Lifecycle workflows** — automate joiner/mover/leaver.

**Secure admin of cloud tenants (SaaS/multicloud):** dedicated admin accounts (cloud-only, no email/mailbox), MFA + phishing-resistant auth, PIM for all privileged roles, break-glass accounts, restrict admin to **PAWs**.

**CIEM = Microsoft Entra Permissions Management** — visibility and right-sizing of permissions across **Azure, AWS, GCP**. **Permission Creep Index (PCI)** flags identities with far more permissions than they use; remediate to least privilege; also covers workload identities.

**Secure workstations (PAW)** — dedicated, hardened **Privileged Access Workstations** for admin tasks only (no browsing/email), device-compliance enforced via CA. For remote privileged access, use **Azure Bastion** (no public IP / RDP-SSH over TLS in portal) and **JIT VM access** (Defender for Servers opens management ports only on request, time-bound).

### Decision table — privileged / remote access

| Scenario | Service / Feature |
|----------|-------------------|
| JIT elevation of admin roles with approval | Privileged Identity Management (PIM) |
| Bundle access (roles/groups/apps) with approval + expiry | Entitlement management (access packages) |
| Right-size permissions across Azure/AWS/GCP | Entra Permissions Management (CIEM) |
| Recertify who has access periodically | Access reviews |
| Hardened device for admin tasks only | Privileged Access Workstation (PAW) |
| Secure RDP/SSH to VMs without public IPs | Azure Bastion |
| Open VM management ports only when needed | Just-in-time VM access (Defender for Servers) |

## 2.4 Solutions for regulatory compliance

### Compliance controls for regulated agent deployments
Translating regulations (**EU data privacy/GDPR, SOC 2, HIPAA, ISO 27001, EU AI Act**) into concrete agent controls with continuous audit evidence. *(Part of the [zero-trust multi-agent module](#securing-multi-agent-systems-with-azure-zero-trust-architecture).)*

- **Map each requirement → specific agent control** in a **compliance matrix**, so auditors can verify technical enforcement (requirement ↔ agent behavior).
- **Data residency & minimization:** **region-locked deployments**, **Azure Policy** (deny non-compliant regions), and **per-agent data scoping** (minimization).
- **Audit evidence:** capture **structured consent + processing logs**, export to a **long-retention Log Analytics workspace**, automate periodic reports and access reviews. Include **AI red-teaming results** as testing evidence.
- Example: Fabrikam EU region-locked deployments, per-agent minimization, consent logs → provable technical enforcement for auditors, reduced legal risk.

*Takeaway:* Build a **compliance matrix** (requirements → agent behaviors), enforce **region-bound deployments + Azure Policy**, and stream structured processing logs to a compliant workspace to automate monthly evidence reports.

### Regulatory compliance (tooling)

**Translate requirements → controls:** map each regulatory requirement to a technical control + the tool that enforces and evidences it (control matrix). Use **Microsoft Purview Compliance Manager** — provides **assessments** against templates (NIST, ISO 27001, HIPAA, GDPR, etc.), a **Compliance Score**, and **improvement actions** with implementation/testing guidance.

**Microsoft Purview (data compliance)** — **Information Protection** (sensitivity labels, classification), **Data Loss Prevention (DLP)**, **Data Lifecycle / Records Management** (retention/disposal), **Insider Risk Management**, **Communication Compliance**, **eDiscovery/Audit**. Addresses "protect and prove" for regulated data.

**Azure Policy** — enforce compliance in the resource plane: **regulatory compliance initiatives** (built-in for NIST SP 800-53, PCI-DSS, ISO 27001, CIS, etc.) assigned at management-group scope; **deployIfNotExists/deny** effects auto-remediate or block non-compliant resources.

**Defender for Cloud — Regulatory Compliance dashboard** — continuously assesses your Azure/AWS/GCP posture against standards (MCSB is the default; add PCI-DSS, ISO, NIST, SOC TSP, etc.), shows pass/fail per control with remediation. Use it to **validate and evidence** alignment over time.

### Decision table — compliance tooling

| Scenario | Service / Feature |
|----------|-------------------|
| Assess & score against a regulation, get action plan | Purview Compliance Manager |
| Classify and label sensitive data | Purview Information Protection (sensitivity labels) |
| Prevent sensitive data from leaving | Purview DLP |
| Retain/dispose records per regulation | Purview Data Lifecycle / Records Management |
| Enforce/deny non-compliant resource configs | Azure Policy regulatory initiatives |
| Continuously validate cloud posture vs standards | Defender for Cloud Regulatory Compliance dashboard |

---

# Domain 3 — Design security solutions for infrastructure (25–30%)

## 3.1 Security posture management in hybrid & multicloud

**Defender for Cloud = CNAPP** (Cloud-Native Application Protection Platform), split into two halves:
- **CSPM (posture)** — **Foundational CSPM is free** (Secure Score, recommendations, asset inventory, MCSB assessment). **Defender CSPM** (paid) adds **attack path analysis, cloud security graph, agentless scanning, DevOps security, data-aware posture**.
- **CWPP (workload protection)** — per-resource-type **Defender plans** (see table) providing threat detection/alerts.

**Secure Score** — Defender for Cloud's posture metric; each recommendation maps to controls; raise score by remediating. Assessed against **MCSB** by default; add regulatory standards.

**Multicloud & hybrid:** connect **AWS/GCP** via native **connectors** (agentless), and on-prem/other-cloud VMs via **Azure Arc** (projects them into ARM for policy + Defender coverage). Single posture + protection view across all.

**Defender EASM** — **outside-in** discovery of your internet-facing attack surface (domains, IPs, certs, shadow IT) — finds assets you didn't know were exposed.

**Microsoft Security Exposure Management** — unifies posture across Defender + third-party into a **security graph**: **attack paths** (visualize routes to critical assets, now hybrid on-prem+cloud), **attack surface map**, **security initiatives** (track posture by program, e.g., ransomware/Zero Trust), and **critical asset management**. Complements ASR (attack surface reduction rules on endpoints).

### Decision table — posture management

| Scenario | Service / Feature |
|----------|-------------------|
| Free baseline posture + Secure Score | Foundational CSPM (Defender for Cloud) |
| Attack paths, agentless scanning, data-aware posture | Defender CSPM (paid) |
| Threat protection for VMs/containers/DBs/storage | Defender for Cloud workload plans (CWPP) |
| Bring AWS/GCP into one posture view | Defender for Cloud multicloud connectors |
| Bring on-prem servers under Azure governance | Azure Arc |
| Discover unknown internet-facing assets | Defender EASM |
| Visualize attack paths to crown-jewel assets | Security Exposure Management (attack paths) |
| Track posture by security program/initiative | Security Exposure Management (initiatives) |

## 3.2 Securing server and client endpoints

**Servers** — **Defender for Servers** (Plan 1 = core EDR via Defender for Endpoint; **Plan 2** adds vulnerability assessment, **JIT VM access**, file integrity monitoring, agentless scanning, free 500 MB/day log ingestion). Multi-platform (Windows/Linux), on-prem/multicloud via Arc. Apply **security baselines** (Azure security baseline / CIS) and **Azure Update Manager** for patching.

**Client endpoints** — **Defender for Endpoint** (EDR) for protection/detection; **Microsoft Intune** for management, **hardening**, and **compliance policies** feeding CA (compliant-device requirement). Enable **ASR rules**, disk encryption (BitLocker/FileVault), and security baselines via Intune.

**Mobile** — Intune **MDM** (managed devices) + **MAM / app protection policies** (BYOD, protect app data without enrolling device); Defender for Endpoint mobile threat defense.

**IoT / embedded & OT/ICS** — **Microsoft Defender for IoT**: agentless network monitoring for **OT/ICS** (SCADA, PLCs) via passive sensors, asset discovery, and anomaly detection; maps to **MITRE ATT&CK for ICS**; integrates with Sentinel. Distinct from device-builder embedded security (Defender for IoT micro-agent).

**Windows LAPS** — rotates and stores **local administrator passwords** (in Entra ID or AD), unique per device — stops lateral movement via shared local-admin creds. Now built into Windows.

### Decision table — endpoints

| Scenario | Service / Feature |
|----------|-------------------|
| EDR for servers + JIT/FIM/vuln assessment | Defender for Servers Plan 2 |
| EDR for client devices | Defender for Endpoint |
| Manage/harden/verify compliance of devices | Microsoft Intune |
| Protect app data on unmanaged BYOD | Intune app protection policies (MAM) |
| Monitor OT/ICS/SCADA networks | Defender for IoT (OT monitoring) |
| Unique rotating local-admin passwords | Windows LAPS |
| Patch servers across Azure/on-prem/multicloud | Azure Update Manager |

## 3.3 Securing SaaS, PaaS, and IaaS services

**Baselines** — apply the **Azure security baseline** (MCSB-derived, per-service) to each service; enforce with Azure Policy.

**Web workloads** — front with **Azure WAF** (on Application Gateway or Front Door) against OWASP Top 10; **Azure Front Door / DDoS Protection** for edge + volumetric attacks; private endpoints for backends.

**Containers** — **Defender for Containers**: covers **AKS, ARC-enabled/EKS/GKE**. Capabilities: **image vulnerability scanning** (registry + runtime), **runtime threat detection**, Kubernetes-level hardening recommendations, and admission control (**Azure Policy for Kubernetes / Gatekeeper**). Secure the supply chain (scan in CI/CD) and use private registries (ACR with content trust).

**Container orchestration (AKS)** — RBAC + Entra integration, network policies, private clusters, workload identity (no stored secrets), secrets in Key Vault via CSI driver.

**IoT workloads** — Defender for IoT + IoT Hub with per-device identity, X.509 certs, and least-privilege.

**Azure AI services security** — private endpoints, managed-identity/Entra auth (disable local keys), content filtering, **Defender for AI Services** (jailbreak/prompt-injection/wallet-attack detection), and **Purview DSPM for AI** for data governance.

### Decision table — SaaS/PaaS/IaaS

| Scenario | Service / Feature |
|----------|-------------------|
| Protect web app from OWASP attacks | Azure WAF (App Gateway / Front Door) |
| Absorb volumetric/DDoS attacks | Azure DDoS Protection + Front Door |
| Scan container images + runtime threats | Defender for Containers |
| Enforce policy on Kubernetes deployments | Azure Policy for Kubernetes (Gatekeeper) |
| No stored secrets in AKS pods | AKS workload identity + Key Vault CSI |
| Threat protection for Azure OpenAI | Defender for AI Services |
| Govern data used by AI apps | Purview DSPM for AI |

## 3.4 Network security and Security Service Edge (SSE)

**Network design best practices** — **segment** (hub-spoke or **Virtual WAN**), **deny-by-default** with **NSGs** + **Azure Firewall** (or 3rd-party NVA); eliminate public exposure with **Private Link / private endpoints**; encrypt in transit (TLS); **DDoS Protection** on public-facing; centralize inspection in the hub. Zero Trust: never trust based on network location alone.

**Microsoft SSE = Global Secure Access** (GA as of 2026), delivered over Microsoft's global WAN:
- **Entra Internet Access** — **Secure Web Gateway (SWG)**: protects internet + SaaS + M365 traffic; web content filtering, **universal Conditional Access** applied to any app (incl. non-Microsoft), and a **compliant network** check (block access unless traffic flows through SSE). Cross-tenant configs for M365.
- **Entra Private Access** — **ZTNA**, modern **VPN replacement**: per-app (not network) access to private/on-prem apps via connectors, with Conditional Access — no broad network access.

Together they extend **Zero Trust** to web, SaaS, AI, and private-app traffic with identity-centric policy.

### Decision table — network / SSE

| Scenario | Service / Feature |
|----------|-------------------|
| Central east-west/north-south filtering | Azure Firewall (hub) |
| Subnet-level allow/deny rules | Network Security Groups (NSGs) |
| Remove public endpoints from PaaS | Private Link / private endpoints |
| Secure web gateway + content filtering | Entra Internet Access |
| Apply CA to internet/SaaS/M365 traffic | Entra Internet Access (universal CA) |
| Replace VPN with per-app private access (ZTNA) | Entra Private Access |
| Protect public IPs from volumetric attacks | Azure DDoS Protection |
| Global segmented backbone connectivity | Azure Virtual WAN |

---

# Domain 4 — Design security solutions for applications and data (20–25%)

## 4.1 Securing Microsoft 365

**Posture** — **Microsoft Secure Score** (in the Defender portal) measures M365 identity/apps/device posture; improvement actions raise the score.

**Defender XDR for M365 workloads:**
- **Defender for Office 365** — email/collaboration: **Safe Attachments** (detonation), **Safe Links** (time-of-click URL rewriting), anti-phishing/impersonation, **attack simulation training**.
- **Defender for Cloud Apps** — **CASB**: **shadow IT discovery**, SaaS app governance, **session policies** (via CA app control) to monitor/block risky actions, OAuth app risk.
- **Defender for Identity** — on-prem AD signal into XDR.

**Intune** — device compliance + app protection; feeds **Conditional Access** (require compliant device to reach M365 data).

**Purview for M365 data** — sensitivity labels + **DLP** across Exchange/SharePoint/OneDrive/Teams; retention; Insider Risk; eDiscovery/Audit.

**Copilot for M365 data security** — Copilot honors existing permissions + **sensitivity labels** (won't surface data a user can't already access, and inherits/propagates labels). Key risk = **oversharing** → use **Purview DSPM for AI** to run **data-risk assessments** (weekly by default), find oversharing, apply **one-click policies**; audit Copilot prompts/responses (Purview Audit). Restrict with **SharePoint Advanced Management** + label-based access.

### Decision table — Microsoft 365

| Scenario | Service / Feature |
|----------|-------------------|
| Detonate malicious attachments/links in email | Defender for Office 365 (Safe Attachments/Links) |
| Discover shadow IT / govern SaaS usage | Defender for Cloud Apps (CASB) |
| Real-time block/monitor risky SaaS sessions | Defender for Cloud Apps session policies |
| Require compliant device for M365 | Intune + Conditional Access |
| Classify + protect M365 documents/email | Purview sensitivity labels + DLP |
| Prevent Copilot oversharing | Purview DSPM for AI (data risk assessments) |
| Audit Copilot prompts/responses | Purview Audit |

## 4.2 Securing applications

**Portfolio posture & threat modeling** — inventory apps, classify by criticality/data sensitivity; **threat-model** business-critical apps with **STRIDE** (and **OWASP Top 10** / **API Security Top 10** / **ATLAS** for AI apps) to find design-level risks early.

**Full-lifecycle app security (DevSecOps)** — security across design→build→deploy→run: secure coding standards, **SAST/DAST/SCA** + **secret scanning** + **IaC scanning** in CI/CD (surfaced via **DevOps security in Defender for Cloud**), signed artifacts, least-privilege pipeline identities, runtime protection.

**Workload identities** — apps authenticate to Azure with **managed identities** (no secrets; system- or user-assigned) or **service principals** with **federated credentials** (workload identity federation — GitHub Actions/K8s without stored secrets). Govern with **Workload Identities Premium** (risk detection + CA for workload identities) and **access reviews**.

**API security** — **Azure API Management (APIM)** as the gateway: authN/authZ (OAuth2/JWT validation), **rate limiting/throttling/quotas**, IP filtering, subscription keys, and backend hidden behind private endpoints. Add **WAF** in front for OWASP protection.

**Azure WAF** — on **Application Gateway** (regional) or **Front Door** (global): managed OWASP rulesets + custom rules; prevention vs detection mode.

### Decision table — applications

| Scenario | Service / Feature |
|----------|-------------------|
| App authenticates to Azure without secrets | Managed identity |
| CI/CD pipeline auth without stored secrets | Workload identity federation |
| Risk detection + CA for app/service identities | Workload Identities Premium |
| Publish/secure/throttle APIs | Azure API Management |
| OWASP protection for web app (global) | Azure WAF on Front Door |
| OWASP protection for web app (regional) | Azure WAF on Application Gateway |
| Find code/secret/IaC flaws pre-deploy | DevOps security (Defender for Cloud) |
| Design-level threat identification | STRIDE threat modeling |

## 4.3 Securing an organization's data

**Discovery & classification** — **Purview** scans and classifies data (sensitive info types, trainable classifiers) across M365, Azure, on-prem, and multicloud; apply **sensitivity labels** that drive encryption + DLP. Know your data before you protect it.

**Prioritize threats to data** — protect highest-sensitivity/highest-impact data first (crown jewels); combine **classification + DSPM** (Defender CSPM data-aware posture shows sensitive data in attack paths).

**Encryption:**
- **At rest** — Azure encrypts by default (platform-managed keys). For control: **CMK** in Key Vault; **infrastructure/double encryption** for defense-in-depth; **Managed HSM** for FIPS. **crypto-shred** = delete the key to render data unrecoverable.
- **In transit** — TLS everywhere; private endpoints to keep traffic off the internet.

**Data in Azure workloads:**
- **Azure SQL** — **Transparent Data Encryption (TDE)**, **Always Encrypted** (client-side, admins can't see plaintext), **Dynamic Data Masking**, auditing, Entra auth, **Defender for SQL**.
- **Cosmos DB / Synapse** — encryption at rest + CMK, private endpoints, RBAC/Entra auth, partition-level isolation.
- **Azure Storage** — private endpoints, disable public/anonymous access, **Entra auth over keys/SAS**, immutable/versioning; **Defender for Storage** (malware scanning on upload, sensitive-data threat detection).

**Data for AI workloads** — classify/label training + grounding data, CMK, DLP on prompts, **Purview DSPM for AI**, and restrict which data agents/models can reach (least privilege).

**Defender for Databases / Storage** — CWPP plans generating threat alerts for data services (SQL, open-source DBs, Cosmos DB, Storage).

### Decision table — data security

| Scenario | Service / Feature |
|----------|-------------------|
| Discover & classify sensitive data | Purview (classification + sensitivity labels) |
| Encrypt with keys you control | CMK in Azure Key Vault (Managed HSM for FIPS) |
| SQL data hidden even from DBAs | Always Encrypted |
| Default SQL at-rest encryption | Transparent Data Encryption (TDE) |
| Malware scan on blob upload | Defender for Storage |
| Threat alerts for SQL/Cosmos/OSS DBs | Defender for Databases |
| Render data unrecoverable on request | Crypto-shred (delete CMK) |
| Show sensitive data on attack paths | Defender CSPM (data-aware posture) |
