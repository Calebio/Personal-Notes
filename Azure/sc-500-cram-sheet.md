# SC-500 Cram Sheet: keyword → service

> **How to use it:** find the keyword in the question stem, then pick the service.
> - **Part A (AI)** is the main focus. It covers every AI objective in the official outline (3.1 + 4.3).
> - **Part B** is a fast recap of the non-AI domains (mostly carried over from AZ-500).
> - Detailed notes are in `sc-500-notes.md`.
> - ⚠️ Rows marked **(gk)** come from general knowledge, not your course units. Double-check any that look unfamiliar.

**Exam weights:**
- D1 Identity/access/governance 20–25%
- D2 Storage/DB/network 25–30%
- D3 Compute **incl. AI** 20–25%
- D4 Posture & monitoring **incl. Security Copilot** 20–25%

---

# PART A: AI (focus here)

## 🧭 Which product? (the #1 trap)
| If it's about… | Product | Portal |
|---|---|---|
| **Agent identity**: sign-in, Conditional Access, disable, access reviews | **Entra Agent ID** | Entra admin center |
| **Agent risk**: discovery, blast radius, attack paths | **Defender XDR → AI agent inventory** | Defender portal |
| **Copilot Studio agent at runtime**: block tool calls, prompt injection | **Defender for Cloud Apps (MDCA) real-time protection** | Defender portal + Power Platform admin center |
| **Who can call a Foundry model**: auth, token limits, routing, logs | **AI Gateway (Azure API Management)** | Foundry portal (ai.azure.com) |
| **What goes in/out of the model**: harmful content, jailbreak, banned words | **Foundry guardrails** (Content Safety) | Foundry → Guardrails + controls |
| **Threat alerts on Azure OpenAI / AI Model Inference** | **Defender for AI Services** (CWP plan) | Defender for Cloud → Environment settings |
| **Azure AI resource misconfiguration** (public access, keys) | **Defender CSPM** | Defender for Cloud |
| **One view of Azure data + AI protection** | **Data & AI security dashboard** | Defender for Cloud |
| **M365 data that Copilot can expose / SharePoint oversharing** | **Purview DSPM for AI** | Purview portal |
| **Approve/block agents for M365 users** | **Microsoft Agent 365** | **M365 admin center → Agents** |
| **AI for the SOC**: prompts, plugins, security agents | **Microsoft Security Copilot** | securitycopilot.microsoft.com |

## 3.1a SharePoint oversharing + Copilot/AI risk → **Purview DSPM for AI**
| Keyword | Answer |
|---|---|
| Which **sensitive data Copilot surfaces** / sites used for **Copilot grounding** | **Purview DSPM for AI** |
| **SharePoint overexposure** view | **Data risk assessments**: weekly, **top 100 sites** by usage (+ custom assessments) |
| Stop Copilot **summarizing labeled files**, **don't change permissions** | **Restrict access by label** (creates a DLP policy) |
| Hide a site **from Copilot only**, humans keep access | **Restrict all items** = **Restricted Content Discovery** |
| Reduce **human AND AI** exposure | **Tighten SharePoint permissions** |
| Unlabeled sensitive files on a site | **Auto-labeling policy** |
| Stale content (**3+ yrs** no access) | **Retention policy** |
| See anyone-with-the-link / external sharing | Site flyout → **Monitor** tab |
| Run a classification scan now | Site flyout → **Identify** tab |
| **Which files** Copilot used; prompt vs response | **Activity explorer** / interaction detail pane |
| Detect sensitive data pasted into **ChatGPT / Gemini** | **Device onboarding + Purview browser extension** |
| …and **block** that paste | **Purview DLP** (DSPM **never blocks**, it's visibility only) |
| File name for a browser paste event | **Not available**: you only get the sensitive info type |
| Quick-start detection policies | **One-click**: Detect risky AI usage · Unethical behavior in AI apps · Detect sensitive info shared with AI via network |
| **No Copilot data** in DSPM | **Purview Audit on** + **M365 Copilot licenses** |
| License | **M365 E5 / Purview Suite** (**E3 doesn't qualify**) |
| Only credit card/SSN findings, no label prioritization | **Deploy sensitivity labels** |
| Security Copilot missing from Apps and agents | **Separate collection policy** needed |
| Highest-priority risk | **Highly Confidential + external AI site** |

## 3.1b Real-time protection for Copilot Studio agents → **MDCA**
| Keyword | Answer |
|---|---|
| **Block a tool invocation before it runs** / prompt injection at runtime | **MDCA real-time protection** (acts as a **proxy**) |
| Turn on agent discovery and protection | Defender: **System > Settings > Security for AI** |
| Power Platform side of setup | PPAC → **Security > Threat Protection → Microsoft Defender – Copilot Studio AI Agents** |
| Setup handoff | **URL: Defender → Power Platform admin**. **App ID: Power Platform admin → Defender** |
| Who **creates the Entra app (App ID)** | **Power Platform Administrator** |
| Status **Pending / Disconnected** | **App ID mismatch** or the Power Platform step isn't done |
| **Blocks happen but no alerts/incidents** | **Microsoft 365 app connector** isn't connected (Global Admin or Security Admin to connect it) |
| Protection on, but a user's current chat isn't protected | **Only new sessions** are covered, so restart |
| Proxy App ID = the agent's identity? | **No.** The proxy App ID = the inspection connection |
| Find a Copilot Studio agent's Entra Agent ID | Copilot Studio → **Settings > Advanced > Metadata** |
| Agents created **before 18 Mar 2026** | **Legacy app registration** (no Agent ID governance) |
| KQL: what the agent **did** | **`CloudAppEvents`** (`Application == "Microsoft Copilot Studio"`) |
| KQL: agent **configuration / tools / IsBlocked** | **`AIAgentsInfo`** |
| Maker checks protection without Defender | Copilot Studio Agents page → **Protection status** (Protected / **Needs review** = violation or weak auth / Unknown) |
| License | **Agent 365**, or **MDCA + Copilot Studio** |

## 3.1c Entra Agent ID: Conditional Access + access management
| Keyword | Answer |
|---|---|
| Agent acts **on behalf of a signed-in user** (OBO) | CA targeting **users/groups**. Full controls (MFA, device) |
| **Autonomous** / **client credentials** / no user | CA targeting **agent identities** |
| CA for agent identities: **only condition** | **Agent risk** (Preview) |
| CA for agent identities: **only control** | **Block access** |
| "Require MFA for an autonomous agent" | ❌ Always wrong |
| Cover **new agents automatically** by business label | **Custom security attributes** |
| Cover **all agents from one template** | **Agent identity blueprint** |
| Test a CA policy safely | **Report-only** → sign-in logs (24–48 h) |
| **Compromised agent, stop it now** | **Disable** the agent identity (**Agent ID Administrator**) |
| Disable a whole family of agents | **Disable the blueprint** |
| Detect **agent created** | Audit log **Add service principal**. Agent if **`agentType` ≠ `notAgentic`** |
| Agent **user account** created | Audit event **Create user** |
| Periodically confirm agents are still needed | **Access reviews**, scope = **Workload identities** |
| Remove access but **keep the audit trail** | **Block/disable**, not delete |
| CA for agents license | **Entra ID P1** (+ Agent 365 per the docs) |

## 3.1d Blast radius → **Defender XDR**
| Keyword | Answer |
|---|---|
| Discover all AI agents | Defender → **Assets > AI Agents** |
| Discover **Bedrock / Vertex / Foundry** agents | **Foundry integration in Defender for Cloud** |
| Agent uses **API key / external IdP** | **Not discovered**, so use a manual inventory |
| 3 blast radius components | **Permissions** (can ACCESS) · **Knowledge sources** (can KNOW) · **Blueprint actions** (can DO) |
| Blast radius for a **Foundry/Bedrock/Vertex** agent | Inventory → **agent detail pane** |
| Blast radius for a **Copilot Studio** agent | **Advanced Hunting → Queries → AI Agents** |
| Visualize the attacker's route | Attack path → **View on map** |
| Prioritize | By **criticality of the target**, not the number of steps |
| Least-privilege swaps | `Sites.FullControl.All` → **`Sites.Selected`**. `User.Read.All` → **`User.ReadBasic.All`** |
| Bigger blast radius | **Application** perms (tenant-wide) beat **delegated** |
| Dormant agent with broad perms | **Still exploitable** |

## 3.1e AI Gateway (APIM) for Foundry
| Keyword | Answer |
|---|---|
| Anyone with the endpoint can call the model / shared API key | **AI Gateway** + a **managed identity per app** (Entra tokens) |
| Runaway app/agent eating capacity | **Token limits per project per deployment** |
| **429** | **TPM** rate limit (temporary) |
| **403** | **Total token quota** exhausted |
| Where to create it | **Foundry portal → Operate > Admin console → AI Gateway** |
| APIM tier: dev/test / prod / **private networking** | **Basic v2** / **Standard v2** / **Premium v2** (public access disabled, VNet injection + PE) |
| Existing project not going through the gateway | **Add project to gateway** (new projects are automatic) |
| Gateway only accepts your tokens | Set **audience + issuer** |
| Internal → premium model, external → standard / A/B test | **Route policies** |
| Compromised Foundry agent | **Revoke its role assignment** |
| Gateway → model **without secrets** | **Gateway managed identity** + **Cognitive Services User** |
| No public exposure / hybrid callers | **Private endpoint + VNet** / **IP allowlist** |
| Are prompts in the logs? | **No**: metadata only |
| Logs: query+alert / archive / stream to SIEM | **Log Analytics** / **Storage** / **Event Hubs** |
| KQL table | **`GatewayLogs`** |
| Roles | New APIM: **RG Contributor/Owner**. Existing: **APIM Service Contributor**. Foundry: **Azure AI Account Owner / Azure AI Owner** |

## 3.1f Foundry guardrails (Content Safety)
| Keyword | Answer |
|---|---|
| Jailbreak / **indirect** prompt injection | **Prompt Shields** (input) |
| Flag but don't block / block + log | **Annotate only** / **Annotate and block** |
| Violence, hate, sexual, self-harm | **Content filters** (severity slider, input + output) |
| Exact internal terms / codenames | **Blocklist: Exact match** |
| Variations / token-like strings | **Blocklist: Regex** |
| Bulk import terms | **CSV with `Term` + `Type`** |
| Copyrighted text/code in output | **Protected material detection** (output) |
| **Hallucination** / not supported by source | **Groundedness detection** (preview, output) |
| Test before production | **Try it out** (built-in models only by default) |
| Filter exists but a model is unprotected | Filter **not connected to the deployment** |
| Roll out without disruption | **Moderate thresholds + annotate first** |
| Wording | Higher "blocking level" = stricter. Lower "severity threshold" = stricter |

## 3.1g Defender for AI Services + Data & AI dashboard (Defender for Cloud)
| Keyword | Answer |
|---|---|
| Jailbreak / **data leakage** / **credential theft** / **data poisoning** alerts on Azure OpenAI | **Defender for AI Services** |
| Enable | Defender for Cloud → **Environment settings → subscription → AI services On** (**per subscription**) |
| Role to enable | **Owner/Contributor** on the subscription |
| Images/audio scanned? | **No**: **text tokens only** |
| Covers Copilot Studio? | **No**, that's **MDCA** |
| Prompt text masked in alerts | Enable **Suspicious prompt evidence** |
| See the triggering prompt | **Prompt Suspicious Segment** (XDR: Alert → Activities → Related activities) |
| Purview compliance on AI prompts | **Data security for AI interactions** (**separate Purview license**) |
| Foundry **agent** interactions in Purview | **Not supported yet** |
| Scan **custom ML registry models** (malware, serialization flaws, secrets) | **AI model security** |
| Inventory of AI models/datasets/services | **AI BOM** (CSPM) |
| Query AI in use | **Cloud Security Explorer → "AI workloads and models in use"** |
| Vulnerable repos provisioning Azure OpenAI | Cloud Security Explorer → **"Generative AI vulnerable code repositories…"** |
| AI resource with public access / local auth / no logs | **Defender CSPM recommendations** |
| Show only AI alerts | Filter **Product Component Name = AI / AIServices** |
| XDR filter for AI incidents | Service source → **Microsoft Defender for AI Services** |
| Dashboard sections | **Overview** (protection status) · **Top issues** · **Data closer look** · **AI closer look** (prompts scanned, AI alerts, exposed grounding) |
| Dashboard data sections empty | Enable **Defender CSPM (+ sensitive data discovery), Storage, Databases** |
| Least privilege to view the dashboard | **Security Reader** |
| AI alerts missing in Sentinel | Check the **Defender XDR data connector** |
| Recurring pen-test alerts | **Suppression rules** |
| Auto-remediate (quarantine, disable creds) | **Logic App playbook** |
| Assessment data not flowing | Register **Microsoft.Security** resource provider |

## 3.1h Manage agents in the M365 admin center → **Microsoft Agent 365**
| Keyword | Answer |
|---|---|
| Approve agents before users can use them | **Agents → All agents → Requests → Publish** (or Reject) |
| **Least-privilege role to approve/block** | **AI Administrator** (Global Admin also works) |
| SOC can view but not approve | **Security Admin / Security Reader / Security Operator / AI Reader / Global Reader** = **view-only** |
| License | **M365 Copilot** (Agent 365 is inherited) |
| Approved, but only one department should use it | **Manage access** (user scope) |
| Connector DLP (SharePoint ↔ Dropbox) | **Power Platform admin center → Data policies** |
| Fixable issue / policy violation | **Return with feedback** / **Block** (stays blocked, reason logged) |
| Investigating, need it off temporarily | **Block** (reversible) |
| Publisher **added new permissions** after approval | **Revoke approval** (back to pending) |
| Scope crept 20 → 200 users | **Narrow user scope** |
| Creator left the company | **Agents without owners** → assign an owner |
| Risk signals from Entra/Defender/Purview in one place | **Agents at risk** card |
| No use for 60+ days | Review for **decommissioning** |
| Copilot Studio agent access ceiling | The **signed-in user's permissions** (OBO) |
| Central agent governance + templates for new agents | **Agent 365 → Settings**: allowed agent types, **security templates** |

## 4.3 Microsoft Security Copilot ⚠️ (gk). Not in your course units yet
| Keyword | Answer |
|---|---|
| Capacity / billing unit | **Security Compute Units (SCUs)**: provisioned + optional **overage** |
| Separate capacity, data and access per team/region | **Workspaces** (each has its own capacity, settings and role assignments) |
| Who manages settings, capacity, plugins | **Security Copilot Owner** |
| Who can use prompts | **Security Copilot Contributor** (restrict it to a security group instead of everyone) |
| Owner by default | **Global Admin + Security Admin** (Entra) |
| What data a plugin can return | Limited by the **user's own permissions** in that service (on-behalf-of) |
| Built-in plugins | **Microsoft plugins** (Defender XDR, Sentinel, Entra, Intune, Purview, EASM, Azure Firewall/WAF…) |
| Bring your own data/API/KQL | **Custom plugins** (API, KQL, GPT). The owner controls **who can add custom plugins** |
| Sentinel plugin needs | A **selected Sentinel workspace** |
| Microsoft-built autonomous agents | e.g. **Phishing Triage Agent** (Defender), **Conditional Access Optimization Agent** (Entra), **Vulnerability Remediation Agent** (Intune), **Alert triage agents for DLP/IRM** (Purview), **Threat Intelligence Briefing Agent** |
| Third-party agents | **Microsoft Security Store** agents |
| Enabling Microsoft agents (e.g. CA Optimization) | Needs **Security Admin / Identity Governance Admin** (see your 3.1 notes) |
| Block Security Copilot entirely | **Remove all SCU capacity** |
| Data sharing / where data is stored | Owner sets this up at onboarding: **data location** + **data-sharing toggles** |

---

# PART B: Recap of non-AI domains (keyword → service)

## D1 Identity, access & governance
| Keyword | Answer |
|---|---|
| Time-bound / approval-based **admin activation** | **PIM** (Entra roles, Azure roles, groups) |
| MFA when risky / location / device | **Conditional Access**. Risk signals come from **Entra ID Protection (P2)** |
| **Phishing-resistant** MFA | **FIDO2 / passkeys, WHfB, CBA** + CA **authentication strength** |
| App **definition** vs **instance in the tenant** | **App registration** vs **Enterprise app (service principal)** |
| Users consenting to risky apps | **User consent settings** + **admin consent workflow** |
| Resource → resource **without secrets** | **Managed identity** (system = tied to the resource. User-assigned = shared, independent) |
| Store secrets/keys/certs | **Key Vault**. Use **Azure RBAC** (preferred) over access policies |
| Prevent permanent deletion of keys | **Soft delete + purge protection** |
| Single-tenant **FIPS 140-3 L3** HSM | **Managed HSM** |
| Unusual Key Vault access | **Defender for Key Vault** |
| Secrets in code/VMs | **Defender CSPM secret scanning** |
| Enforce/deny config at scale | **Azure Policy** (Deny, Audit, Modify, **DeployIfNotExists** = needs a managed identity for remediation) |
| Compliance vs standards | **Defender for Cloud regulatory compliance** (MCSB default) |
| Prevent delete / any change | **CanNotDelete** / **ReadOnly** lock |
| Least-privilege custom role | **Custom role** (Actions / NotActions / DataActions / AssignableScopes) |
| Protect backups from attackers | **Soft delete, immutable vault, multi-user authorization (Resource Guard)** |
| Security in IaC templates | **Defender for Cloud DevOps security** / Policy-as-code |

## D2 Storage, databases & networking
| Keyword | Answer |
|---|---|
| Most secure SAS | **User delegation SAS** (Entra-backed) |
| Revoke many SAS at once | **Stored access policy** |
| Force Entra auth on storage | **Disable shared key access** |
| Storage reachable only from VNet / Azure services | **Storage firewall** + **trusted Microsoft services** exception / **private endpoint** |
| WORM / can't modify blobs | **Immutable storage** |
| Malware upload / anomalous access to blobs | **Defender for Storage** (malware scanning, sensitive data threat detection) |
| Hide data **from DBAs** (client-side) | **Always Encrypted** |
| Encrypt DB at rest | **TDE** (service-managed or **CMK**) |
| Mask data on query output | **Dynamic data masking** |
| Users see only their rows | **Row-level security** |
| SQL auditing destinations | **Storage / Log Analytics / Event Hubs** (server-level applies to all DBs) |
| SQL injection alerts + vuln assessment | **Defender for SQL** (Defender for Databases) |
| L3/L4 allow/deny on subnet/NIC | **NSG** |
| Group VMs by role in rules | **ASG** |
| **Central rules that override NSGs** | **Azure Virtual Network Manager security admin rules** |
| Replace VPN with ZTNA to private apps | **Entra Private Access** (Global Secure Access) |
| PaaS on a **private IP** | **Private endpoint** |
| Expose **your own** service privately (behind Standard LB) | **Private Link service** |
| L3–L7, FQDN filtering, threat intel | **Azure Firewall**. **Premium** = TLS inspection, IDPS, URL/web categories |
| Secure hub + central policy | **Virtual WAN secured hub + Firewall Manager** |
| Which NSG rules actually apply | **Network Watcher → Effective security rules** |
| Is this flow allowed + which rule | **IP flow verify** |
| OWASP protection for web apps | **WAF** on App Gateway / Front Door |

## D3 Servers, VMs & app platforms
| Keyword | Answer |
|---|---|
| Encrypt **inside the OS** (BitLocker/dm-crypt) | **Azure Disk Encryption** |
| Encrypt **temp disk + caches** end-to-end | **Encryption at host** |
| CMK for managed disks | **SSE with CMK** (disk encryption set) |
| RDP/SSH **without public IP** | **Azure Bastion** (native client = Standard+, session recording = Premium) |
| Open mgmt ports **only when requested** | **JIT VM access** (Defender for Servers P2) |
| Bring on-prem/AWS/GCP servers under Azure control | **Azure Arc** |
| EDR on servers | **Defender for Servers** (MDE). **P2** adds agentless scanning, JIT, FIM |
| Secure boot + vTPM + boot integrity | **Trusted launch** (security type) |
| Enforce settings **inside the guest OS** | **Azure Machine Configuration** |
| Container image vulns + K8s runtime threats | **Defender for Containers** |
| AKS: Entra auth, no local accounts, private API | **Entra integration + disable local accounts + private cluster / authorized IPs** |
| AKS pod policy | **Azure Policy for AKS** (Gatekeeper) |
| ACR hardening | **Disable admin user**, private endpoint, managed identity pull |
| Function/App Service sign-in without code | **App Service Authentication (Easy Auth)** |
| Restrict inbound to App Service/Functions | **Access restrictions** / private endpoint |
| Secrets in app settings | **Key Vault references** + managed identity |
| Protect back-end APIs | **APIM policies**: `validate-jwt`, rate-limit/quota, IP filter, client certs, managed identity to backend |

## D4 Posture & monitoring
| Keyword | Answer |
|---|---|
| Attack paths, cloud security explorer, agentless scanning | **Defender CSPM** (paid). Foundational CSPM = free Secure Score + recommendations |
| Connect AWS/GCP | **Defender for Cloud environment connectors** |
| Unknown internet-facing assets | **Defender EASM** |
| VM vulnerability scanner | **Defender Vulnerability Management** |
| Sentinel run/modify vs read vs triage | **Sentinel Contributor** / **Reader** / **Responder** |
| Let automation rules run playbooks | **Microsoft Sentinel Automation Contributor** |
| Packaged connectors + rules + workbooks | **Content hub** solutions |
| Syslog / CEF from Linux appliances | **AMA on a Linux forwarder + DCR** |
| Windows Security events | **AMA + DCR**. From **WEF** collectors → **Windows Forwarded Events** connector |
| Store a custom log source | **Custom table (`_CL`)** via DCR / Logs Ingestion API |
| Auto-assign/tag/close incidents | **Automation rules** |
| Multi-step response (Logic Apps) | **Playbooks** |
| Long-term cheap retention | **Long-term retention (archive) / data lake tier**. Interactive = analytics tier |
| M365 audit activity in XDR hunting | **Purview Audit** query in Defender XDR (gk: **`CloudAppEvents`**) |
