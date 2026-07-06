# SC-200 Notes

## Defender XDR Overview

- Microsoft Defender XDR = unified **pre- and post-breach** solution, working across the Defender suite (endpoints, identities, email/collaboration, cloud apps).
- **Defender for Endpoint**: protects endpoints, detects malicious programs/activity, includes built-in **vulnerability management** (software = attack surface).
- **Defender for Office 365**: protects email and collaboration tools (phishing, malicious attachments).
- **Defender for Identity**: protects identities — the "new perimeter" and a common entry point (e.g., Active Directory attacks).
- **Defender for Cloud Apps**: discovers and protects cloud applications (shadow IT, data exfiltration risk).
- Related but separate components: **Azure AD Identity Protection**, **Insider Risk Management** (not covered in this course).
- **Microsoft Sentinel** ties all of these together into one holistic, cross-solution view (SIEM layer).
- Defender XDR shares threat info across products in real time and can take **automatic action** — e.g., if a malicious file is found on an endpoint via Defender for Endpoint, it instructs Defender for Office 365 to scan and remove that file from all mailboxes.

### Typical Attack Chain (mapped to Defender products)
1. **Phishing email** (targeted or broad) → mitigated by **Defender for Office 365**
2. **Malicious attachment clicked** → bridges email/collaboration to the endpoint
3. **Malicious binary dropped on endpoint** → gives threat actor remote access → mitigated by **Defender for Endpoint**
4. **Lateral movement via identities** (pivoting into Active Directory) → mitigated by **Defender for Identity**
5. **Access to business-critical files**, further lateral movement
6. **Compromise of high-privilege server** (e.g., domain controller, SQL server)
7. **Use of cloud services for exfiltration/C2 communication** → mitigated by **Defender for Cloud Apps**
8. **Insider threats** (e.g., user uploading sensitive files to personal cloud storage) → addressed by Cloud Apps / Insider Risk Management

### Key Takeaway
Each stage of the attack chain maps to a specific Defender product, and Sentinel provides the unified, cross-product visibility and correlation layer.

*Verified against: [What is Microsoft Defender XDR?](https://learn.microsoft.com/en-us/defender-xdr/microsoft-365-defender) (Microsoft Learn).*

## Components of a Modern SOC

Layered model (bottom to top) of a modern Security Operations Center:

1. **Devices / raw data layer** (bottom): the enterprise's full device and telemetry surface — network devices, wireless devices, endpoints, mobile phones, infrastructure, applications, cloud services, OT/IoT devices. These generate security logs, activity logs, and other telemetry, all fed into a central repository for threat hunters.
2. **Extended Detection and Response (XDR) layer**: sits above raw data. Microsoft's implementation is **Microsoft Defender XDR**, which correlates events across datasets using deep insight from Microsoft's Threat Intelligence Center and analytical roles — producing high-quality, per-asset detections plus investigation/remediation capabilities.
3. **SIEM layer**: **Microsoft Sentinel** sits above XDR, adding:
   - Integrations across multiple datasets and multiple clouds
   - Ingestion of external threat intelligence
   - Correlation of raw data + XDR insights (increasing complexity but broader coverage)
   - **Automation and response** — the layer where analysts/hunters primarily operate

### Two Crucial Components for SOC Analysts / Threat Hunters
- **Deep insight layer** (XDR — asset-level correlation and detection)
- **Broad enterprise view** (SIEM/Sentinel — cross-dataset, cross-cloud visibility)

Together these enable: incident response and recovery assistance, managed detection and response (MDR), and continuous improvement of detections aligned to the company's security mission. Defender XDR provides cross-domain signal correlation, a unified incident queue, automated response, and self-healing; Microsoft Sentinel is a cloud-native SIEM unifying AI, SOAR, UEBA, and threat intelligence.

## Understanding an SOC Model

SOC operations model broken down by function/tier:

### Automation
- Goal: **real-time resolution** — automation handles the bulk of work so analysts focus only on relevant attacks.
- Relies on **well-defined attacks**, as identified by XDR solutions and cloud telemetry.

### Tier 1 — Triage
- Focus: **rapid remediation of high-volume, well-known alerts**.
- Requires quick human judgment, but analysts should spend minimal time per alert.
- Example: reviewing/approving automated remediation actions in **Defender for Endpoint's Action Center**.
- Escalates unresolved/complex alerts to Tier 2.

### Tier 2 — Investigation and Incident Management
- Escalation point for Tier 1.
- Monitors alerts indicating a more **sophisticated attacker**: behavioral alerts, business-critical asset alerts, ongoing attack campaigns.
- Performs **deeper investigation** than Tier 1 (low-volume, complex attacks).
- Takes on **non-technical incident management** duties: coordination with communications, legal, and business stakeholders.

### Tier 3 — Hunt and Incident Management
- Multi-disciplinary team (e.g., reverse engineering).
- Handles incidents with **major business impact**, especially those that slipped past reactive detections.
- **Hunts for undetected threats**; assists with escalations and advanced forensics.
- Helps **refine/simplify the automation layer**.
- Operates on a **hypothesis-driven model** (vs. Tier 1/2's reactive alert-driven model).
- Coordinates with **red teams / purple teams**.

### Threat Intelligence (optional team)
- Provides business-relevant context/insight from threat intel platforms or in-house curated intelligence.
- **Technical research**: identifies attacker groups, attack trends, high-profile attacks, emerging techniques.
- **Strategic research**: translates technical research into business insights, informing security priorities and processes.

This tier breakdown matches standard industry SOC models and aligns with the role descriptions used across Microsoft's security operations guidance.

## Defender for Endpoint — Advanced Features

Location: Defender portal → **Settings** → **Endpoints** → **Advanced features**.

- **Restrict correlation to within scope device groups**: limits alert correlation to a single device group (device groups must be configured first). Without it, an alert spanning multiple devices is treated as one incident by default; with it on, alerts crossing device groups are no longer merged into a single incident. Only affects future correlations; device groups are supported in Plan 1 and Plan 2.
- **Enable EDR in block mode**: blocks malicious artifacts/behaviors detected post-breach — notably works **even when Microsoft Defender Antivirus is running in passive mode**.
- **Automatically resolve alerts**: auto-resolves alerts with no real threat found or that were remediated — reduces alert fatigue. Won't overwrite an alert an analyst manually set to "In progress"/"Resolved."
- **Allow or block file**: requires Microsoft Defender Antivirus as the **active** antimalware solution **and** cloud-based protection enabled. Blocks a file from being read, written, or executed on devices.
- **Hide potential duplicate device records**: identifies duplicates by hostname + last-seen time; hidden from Device Inventory/Vulnerability Management/APIs (but still visible in search, hunting, alerts, incidents). On by default.
- **Configure custom network indicators**: create indicator lists (IPs, domains, URLs) to allow/block. Requires devices on **Windows 10 1709+ or Windows 11**.
- **Tamper protection**: locks Microsoft Defender Antivirus/security settings so malware can't disable them.
- **Enable user details display**: pulls Entra ID (Azure AD) info — picture, name, title, department — into the Alert queue and Device details page.
- **Configure Skype for Business integration**: lets analysts contact the user via Skype for Business, email, or phone (e.g., during device isolation).
- **Configure Microsoft Defender for Cloud Apps integration**: forwards Defender for Endpoint signals to Cloud Apps for deeper visibility into cloud app usage.
- **Configure web content filtering**: blocks site categories (e.g., adult, gambling) via a web content filtering policy; requires network protection in block mode.
- **Enable the unified audit log**: routes audit records to the Microsoft Purview audit log (formerly Compliance Center) for user/admin activity investigation.
- **Enable device discovery**: finds unmanaged devices on the network using onboarded devices as sensors — no extra appliances needed.
- **Download quarantined files**: lets you download quarantined files for analysis. On by default.
- **Live response** — three separate toggles: **Enable live response** (start sessions on devices), **Enable live response for servers**, and **Allow unsigned script execution in live response** (permits running unsigned PowerShell/scripts in a session).
- **Share endpoint alerts with Microsoft Compliance Center** (now Microsoft Purview portal): forwards alerts + triage status to enrich Insider Risk Management.
- **Microsoft Intune connection**: shares device info with Intune to enable device risk-based Conditional Access; requires enabling the integration on **both** sides plus specific licensing (EMS E3 + Windows E5, or M365 E5).
- **Enable authenticated telemetry**: prevents spoofed telemetry from appearing in the dashboard.
- **Enable preview features**: opt in to upcoming features.
- **Automatic attack disruption**: automatically contains compromised assets (devices/users) to limit lateral movement mid-attack, while leaving the SOC in control of investigation/remediation/recovery. A heavily tested SC-200 topic.
- **Default to streamlined connectivity when onboarding devices** (sets streamlined connectivity as the default onboarding package) and **Configure Endpoint Attack Notifications** (Microsoft actively hunts critical threats on your endpoint data and prioritizes them by urgency/impact) are also current advanced-feature toggles.

**Important**: must click **Save preferences** after configuring, or changes are lost.

*Verified against: [Configure advanced features in Microsoft Defender for Endpoint](https://learn.microsoft.com/en-us/defender-endpoint/advanced-features) (Microsoft Learn, updated Jun 2026).*

## Setting Up Notification Rules

Location: **Settings** → **Endpoints** → **General** → **Email notifications**. Two rule types: **Alerts** and **Vulnerabilities**.

**Permissions**: requires **Manage security settings** permission, or the **Security Administrator**/**Global Administrator** role. Microsoft recommends the least-privileged role that fits rather than defaulting to Global Administrator.

### Alert Notification Rule
- Button: **"Add item"** starts the wizard.
- Fields: **Rule name**, **Include organization name**, **Include tenant-specific portal link**, **Include device information** (adds device name to the email body).
- **Devices** scope: notify for **all devices** (Global Administrator role only) or **selected device groups**. Not applicable if using Defender for Business.
- **Alert severity**: choose which severities trigger the notification.
- Add recipient email address(es) — multiple allowed.
- **Send test email** to verify delivery.
- Final button: **"Save notification rule."**
- Can create **multiple alert rules**; a rule can be edited or deleted, but there's no on/off toggle.
- RBAC note: if role-based access control is used, recipients only get notifications scoped to the device groups configured in that rule; only a **Global Administrator** can manage rules that cover all device groups.

### Vulnerability Notification Rule
- Data source: **Microsoft Defender Vulnerability Management**.
- Add notification rule → name it (+ description) → check **"Activate notification rule"** → Next.
- Settings include:
  - Device groups to notify for
  - **New vulnerability found** (includes severity threshold, and covers **newly detected zero-day vulnerabilities** and patches released for existing zero-days)
  - **Exploit was verified**
  - **New public exploit**
  - **Exploit added to an exploit kit**
  - Option to include organization name
- Add recipient email(s) — no test email option for this rule type.
- Review → final button is **"Create rule."**
- Rules **can** be individually activated/deactivated via the "Activate notification rule" checkbox.

### Key distinction
- Alert rules: multiple allowed, no individual on/off toggle (edit/delete only).
- Vulnerability rules: multiple allowed, **can** be activated/deactivated individually.

*Verified against: [Configure alert notifications](https://learn.microsoft.com/en-us/defender-xdr/configure-email-notifications) and [Configure vulnerability email notifications](https://learn.microsoft.com/en-us/defender-endpoint/configure-vulnerability-email-notifications) (Microsoft Learn).*

## Creating Custom Indicators (IoCs)

Location: Defender portal → **Settings** → **Endpoints** → **Indicators** (listed under "Rules"). Four indicator types: **file hashes**, **IP addresses**, **URLs/domains**, **certificates**.

Tenant-wide limit is **15,000 indicators**, and this limit **cannot be increased**.

General flow for all types: define the **Indicator** (entity + expiration date — never or custom date) → define the **Action** → define **Scope** (all devices, or a specific device group if configured) → **Save**.

### Available actions differ by indicator type
| Type | Available actions |
|---|---|
| **File hashes** | Allow, Audit, Warn, Block execution, Block and remediate |
| **IP addresses** | Allow, Audit, Warn, Block execution *(no "block and remediate")* |
| **URLs/domains** | Allow, Audit, Warn, Block execution *(no "block and remediate")* |
| **Certificates** | **Allow, Block and remediate only** — no Audit/Warn/Block execution options |

Certificates have the most limited action set of the four indicator types.

### File hash indicators
- Only supports **portable executable (PE) files** — `.exe` and `.dll` only (on Windows; macOS/Linux support different file types).
- Requires Behavior Monitoring + cloud-based protection enabled, Microsoft Defender Antivirus in **active** mode.
- Because each version/build of an app has a different hash, blocking hashes is **not recommended** as a way to block an entire application — Microsoft recommends **WDAC or AppLocker** for that.
- Typical enforcement time: blocks usually take effect in 15–30 minutes, up to 2 hours.
- For **Block and Warn** actions, Defender currently generates only a **generic, informational-severity alert** — custom title/severity/category you configure **may not** be reflected in practice. Custom severity is respected for **Audit** actions. **Allow** never generates an alert.

### IP address / URL / domain indicators
- Only **external IPs** can have indicators created — indicators can't be created for internal/private IPs.
- Only **single IP addresses** are supported (no CIDR ranges), and wildcards aren't supported for IPs.
- Domain indicators block the domain **and all its subdomains**.
- Enforcement can take **up to 48 hours**, though most blocks apply in under 2 hours.
- **Warn mode**: configurable bypass duration (e.g., always warn, or intervals) and can redirect to a custom URL.
- Requires the **"Custom network indicators"** advanced feature to be turned on (Settings → Endpoints → General → Advanced features).
- Conflict handling: if the same entity has conflicting indicator types, precedence is **Allow > Warn > Block**.

### Certificate indicators
- File formats: **.CER or .PEM only**.
- Only **leaf certificates** are supported (not parent/child certs in the chain).
- **Microsoft-signed certificates can't be blocked.**
- Creating or removing a certificate IoC **can take up to 3 hours** to apply.
- Certificate-based indicators have **higher precedence** than file-hash indicators — a cert set to Block will override an Allow set on one of its signed files.
- Common use case: blocking a known-abused or leaked signing certificate across the org.

*Verified against: [Overview of indicators](https://learn.microsoft.com/en-us/defender-endpoint/indicators-overview), [Create indicators for files](https://learn.microsoft.com/en-us/defender-endpoint/indicator-file), [Create indicators for IPs and URLs/domains](https://learn.microsoft.com/en-us/defender-endpoint/indicator-ip-domain), [Create indicators based on certificates](https://learn.microsoft.com/en-us/defender-endpoint/indicator-certificates) (Microsoft Learn).*

## Defining Web Content Filtering

Purpose: enforces browsing policy **on the endpoint itself** via Defender for Endpoint, so it still applies off-network (e.g., a laptop taken home using home internet) — not just on-corporate-network firewalls/proxies.

Before creating policies, the **"Web content filtering"** toggle under **Settings → Endpoints → Advanced features** must be **On**.

Location for policies: **Settings → Endpoints → Rules section → Web content filtering**.

Blocking mechanism: **Windows Defender SmartScreen** in Microsoft Edge; **Network Protection** for Chrome, Firefox, Brave, and Opera.

### Categories
Categories are organized as **5 parent categories**, each with child categories:

| Parent category | Example child categories |
|---|---|
| **Adult content** | Cults, Gambling, Nudity, Pornography/Sexually explicit, Sex education, Tasteless, Violence |
| **High bandwidth** | Download sites, Image sharing, Peer-to-peer, Streaming media & downloads |
| **Legal liability** | Child abuse images, Criminal activity, Hacking, Hate & intolerance, Illegal drug, Illegal software, School cheating, Self-harm, Weapons |
| **Leisure** | Chat, Games, Instant messaging, Professional network, Social networking, Web-based email |
| **Uncategorized** | Newly registered domains, Parked domains |

"Uncategorized" contains **only** these two child categories, not a catch-all for every unclassified site — blocking "Uncategorized" can cause unexpected results.

### Policy creation flow
Add policy → name (General) → Next → **Blocked categories** → Next → **Scope** (device groups; default is **Select all**) → Next → **Summary** → **Submit**.

### Additional details
- You can create a policy with **no categories selected** — this produces an **audit-only policy**, useful for understanding user behavior before enforcing blocks.
- If multiple policies apply to the same device, the **more restrictive** setting wins per category.
- There can be **up to 2 hours of latency** between creating a policy and it being enforced on the device.
- To let one specific site bypass a blocked category, create a custom **Allow indicator** for that URL/domain — allow indicators override web content filtering.
- Miscategorized domains can be disputed via **Reports → Web protection → Web content filtering categories details → Domains → Dispute Category**.

*Verified against: [Web content filtering](https://learn.microsoft.com/en-us/defender-endpoint/web-content-filtering) (Microsoft Learn).*

## Automatic Attack Disruption

Purpose: contain an **in-progress** attack automatically, limit blast radius, and buy the SOC time to fully remediate — while leaving the team **in complete control** of investigation/remediation/recovery.

### Three stages
1. **Correlation** — Defender XDR correlates signals from endpoints, identities, email/collaboration, and SaaS apps into a single high-confidence incident.
2. **Identification** — identifies assets the attacker controls and is using to spread the attack.
3. **Automated response** — takes containment/disabling actions across relevant Defender products in real time.

### Example actions
- **Device containment** *(Defender for Endpoint)*: automatically contains a suspicious device by blocking incoming/outgoing communication to/from it, enforced via a policy pushed to all onboarded devices.
- **Disable user** *(Defender for Identity)*: automatically suspends a compromised account to prevent lateral movement, malicious mailbox use, or malware execution. Behavior varies by where the account lives (on-prem AD only, AD synced to Entra ID, or cloud-native Entra ID account) — in the AD+synced case, it's disabled in **both** AD and Entra ID.
- **Contain user** *(Defender for Endpoint)*: temporarily contains a suspicious identity — blocks that user's communication across onboarded devices to reduce lateral movement and remote-encryption (ransomware) risk.

### Broader action set
Current attack-disruption / predictive-shielding actions also include:
- **Contain IP** (Defender for Endpoint) — contains an IP tied to undiscovered/unonboarded devices.
- **Isolate device** (Defender for Endpoint) — full isolation of a device identified as an active foothold.
- **Revoke user session** and **Suspend user in Entra** (Microsoft Entra ID).
- **OAuth app compromise** action (Defender for Cloud Apps).
- **Safeboot hardening** and **GPO hardening** (Predictive shielding, Defender for Endpoint).
- **Proactive user containment** (Predictive shielding) — contains high-risk users pre-emptively.
- Preview support for **AWS IAM** (via Sentinel) and **Okta** (via Sentinel) disruption actions.

### Prerequisites
- **Subscription**: qualifying options include Microsoft 365 E5/A5, Microsoft 365 E3 + Defender Suite or EMS E5 add-on, Windows/Office/EMS E5 or A5 plans, or the individual Defender products (Defender for Endpoint P2, Defender for Identity, Defender for Cloud Apps, Defender for Office 365 P2, Defender for Business).
- **Deployment**: the broader your Defender product deployment (Endpoint, Identity, Cloud Apps, Office 365), the greater the detection/response coverage; each action requires its corresponding product to be deployed.
- **Permissions**: qualifying roles are **Global Administrator, Security Administrator, User Administrator, Authentication Administrator, Privileged Authentication Administrator, Directory Writers, Helpdesk Administrator, or Security Operator**.
- Device group **remediation/automation level** matters — "Full – remediate threats automatically" or "Semi-automation" allows attack disruption to trigger without manual approval; setting a group to "No automated response" excludes it (not generally recommended).
- You can configure **exclusions** for critical users/devices/IPs that should never be auto-contained.
- Microsoft states containment actions run at **~99%+ confidence** (based on signal-to-noise ratio validation), and **all automatic actions can be undone** by the security team — the SOC retains full control.

### Incident UI
Incidents affected by attack disruption show an **"Attack Disruption" tag** (incident queue and incident page), plus a yellow banner highlighting the automatic action taken, and updated asset status (e.g., "account disabled," "device contained") in the incident graph.

*Verified against: [Automatic attack disruption in Microsoft Defender](https://learn.microsoft.com/en-us/defender-xdr/automatic-attack-disruption) and [Configure automatic attack disruption](https://learn.microsoft.com/en-us/defender-xdr/configure-attack-disruption) (Microsoft Learn).*

## Automated Investigation and Response (AIR)

🚨 **Time-sensitive (today is July 4, 2026):** Microsoft announced (Message Center post MC1411577, July 1, 2026) that as of **September 1, 2026**, AIR will **no longer run as a standalone investigation experience and can no longer be manually triggered**. Its detection/remediation logic is being folded permanently into Microsoft Defender's always-on antivirus protection stack, running automatically without a separate investigation workflow. Any scripts/playbooks that start AIR manually will stop working after that date; full antivirus scans replace manual AIR investigations going forward.

### Core mechanics
- An automated investigation starts **when an alert is triggered**, or can be **started manually** by a security operator selecting a high-risk device and choosing "Initiate Automated Investigation" — manual triggering is the part being removed September 1, 2026.
- While running, the investigation **expands scope**: other alerts from the same device are folded in, and if the same threat/entity is seen on other devices, those devices are added too. If **10 or more devices** get pulled in this way, that scope expansion itself requires approval.
- Each piece of evidence gets a verdict: **Malicious, Suspicious,** or **No threats found** — driving what remediation (if any) follows.
- All remediation actions (pending or completed) are tracked in the **Action Center**, where pending ones are approved/rejected and completed ones can be undone.

### Automation levels
| Level | Description |
|---|---|
| **Full automation** (recommended) | Remediation actions taken automatically on artifacts determined malicious. |
| **Semi — require approval for all folders** | Approval required for remediation on **all** files. |
| **Semi — require approval for core folders remediation** | Approval required only for files/executables in **core (OS) folders** like `\windows\*`; non-core folder items are remediated automatically. |
| **Semi — require approval for non-temp folders remediation** | Approval required for anything **not** in a temp folder (e.g., `\appdata\local\temp\*`, `\windows\temp\*`, `\downloads\*`); temp-folder items are remediated automatically. |
| **No automated response** | Automated investigation doesn't run at all; no remediation actions taken or pending. **Not recommended** — reduces security posture. |

- Pending approval actions **time out after 7 days**, and a timeout is treated the same as a rejection.
- Full automation is the default specifically for **tenants created on or after August 16, 2020** with no device groups yet defined. Tenants created before that date historically defaulted to "Semi – require approval for all folders" unless changed.
- Microsoft recommends full automation — cited data shows tenants using full automation had **40% more high-confidence malware samples removed** than those on lower automation levels.

*Verified against: [Overview of automated investigations](https://learn.microsoft.com/en-us/defender-endpoint/automated-investigations) and [Automation levels in AIR](https://learn.microsoft.com/en-us/defender-endpoint/automation-levels) (Microsoft Learn), plus Microsoft 365 Message Center post MC1411577 (July 2026) on AIR's retirement as a standalone experience.*

## Setting Up Device Groups

Purpose: limit alert/data access to specific Entra user groups (RBAC), set different auto-remediation levels per device set, and scope notification rules/indicators.

Location: **Settings → Endpoints → Permissions section → Device groups**. Tenant-wide cap of **up to 2,000 device groups**.

### Wizard flow
General (name + remediation level + optional description) → Next → **Devices** (matching conditions) → Next → **Preview devices** → Next → **User access** → Submit.

### Remediation levels
No automated response / Semi – approval required for all folders / Semi – approval required for non-temporary folders / Semi – approval required for **system folders** / Full remediation.

### Matching conditions
Device name, domain, device tag, OS platform. Each condition type (tag, device name, domain) supports **up to 10 values**, combined with an **OR** operator within that property. The device preview only shows **up to 10 matching devices**, even if more actually match.

### Device group ranking
If a device matches the conditions of **more than one** device group, it's only added to the **highest-ranked** group — not both. After creation, you can **promote/demote** a group's rank (rank 1 = highest priority) to control this. This directly affects which remediation level and RBAC scope actually applies when group definitions overlap.

### User access / RBAC prerequisite
You can only grant device group access to Microsoft Entra user groups that have already been assigned an RBAC role in Defender — the group has to exist in the RBAC configuration first. If you don't assign any Entra group, the device group defaults to **accessible by all users**.

### Ungrouped devices
Devices matching no group's conditions fall into the default **Ungrouped devices** group. You **can't** rename, re-rank, or delete this default group, but you **can** change its remediation level and which Entra groups can access it.

### Additional details
- Applying device group configuration changes can take **up to several minutes** to take effect.
- **Deleting** a device group that's referenced in an email notification rule removes it from that rule — and if it was the *only* group configured on that rule, the whole notification rule is deleted along with the group.
- Device groups can be used to scope both **notification rules** and **indicator (IoC) organizational scope**.

*Verified against: [Create and manage device groups](https://learn.microsoft.com/en-us/defender-endpoint/machine-groups) (Microsoft Learn).*

## Controlling Permissions within Defender (RBAC)

🚨 As of **February 16, 2025**, new Microsoft Defender for Endpoint customers are placed on **Unified Role-Based Access Control (URBAC)** by default — a newer, cross-workload RBAC model managed centrally under Defender XDR settings rather than the classic "Endpoints → Roles" screen described below. **Existing tenants keep the classic model.** Worth confirming which model your own/lab tenant is on.

### Location and flow (classic model)
- Location: **Settings → Endpoints → Roles** (listed under **Permissions**).
- Flow: Add role → name, description, permissions → Next → assign to a Microsoft Entra **security group** → final button is **"Save and close."**
- This workflow assumes you've **already created the Microsoft Entra user group(s)** beforehand.

### Permission categories available when building a role
- **View data** (Security Operations data; Defender Vulnerability Management data)
- **Active remediation actions** (response actions, approve/dismiss pending remediations, manage allow/block lists; plus Vulnerability Management exception/remediation/application handling)
- **Security baselines** (create/manage baseline assessment profiles)
- **Alerts investigation** (manage alerts, initiate automated investigations, run scans, manage device tags, download PE files)
- **Manage portal system settings** (storage, SIEM/threat intel API settings, roles, device groups — only available in the default Global Administrator role)
- **Manage security settings in Security Center** (alert suppression, automation folder exclusions, onboarding/offboarding, email notifications, indicator allow/block lists)
- **Live response capabilities** — split into **Basic** (start session, read-only commands, download file) and **Advanced** (upload/execute scripts, download PE/non-PE files)

### Risks and prerequisites of turning RBAC on
- Before enabling RBAC, you need a role like **Security Administrator** in Entra ID, and your Entra groups should already be set up — otherwise you risk **locking yourself out** of the Defender portal.
- Initially, **only Global Administrator or Security Administrator** (Entra ID roles) can create/assign Defender roles.
- **Turning on RBAC causes any user with read-only access (e.g., Entra "Security Reader" role) to immediately lose portal access** until they're explicitly assigned a Defender role. A common real-world/exam gotcha.
- **Once you opt in to RBAC, you cannot revert** to the original (pre-RBAC) access model.
- Admin users are auto-assigned the built-in **"Defender for Endpoint Global Administrator"** role with full permissions when RBAC is turned on.

### The "Tier 1/2/3" reference
This refers to Microsoft's **geographic/scope-based tiering for large SOC teams** — Tier 1 = local security/IT team (triage within their geolocation, escalate to Tier 2), Tier 2 = regional SOC (regional device visibility + remediation), Tier 3 = global SOC (full visibility and actions everywhere). This is a **different tiering framework** from the escalation-based Tier 1 (triage) / Tier 2 (investigation) / Tier 3 (hunt) SOC model covered earlier in these notes — same "Tier 1/2/3" labels, different meaning (organizational scope vs. escalation path), so don't conflate the two on the exam.

*Verified against: [Create and manage roles for RBAC](https://learn.microsoft.com/en-us/defender-endpoint/user-roles) and [Use RBAC to grant fine-grained access](https://learn.microsoft.com/en-us/defender-endpoint/rbac) (Microsoft Learn).*

## Working with Vulnerability Management

📝 **Heads-up on navigation**: **Endpoints → Vulnerability management** is correct for the current/"existing customer" experience. Microsoft is in the process of migrating this section under a new **Exposure management → Vulnerability management** umbrella as part of integrating Defender Vulnerability Management into **Microsoft Security Exposure Management** — but as of now this restructuring is only live for **preview customers** (the Defender XDR + Defender for Identity preview program), so most tenants will still see it exactly where classic navigation shows it.

### Dashboard
Cards: **Exposure score**, **Microsoft Secure Score for Devices**, top security recommendations, exposure/device distribution, top vulnerable software, top remediation activities, top exposed devices, and top impactful events (feeds the Event timeline). Exposure score factors in: vulnerabilities found, breach likelihood, device value, and related alerts.

### Recommendations
- The **threat insights** icon appears when a vulnerability has an associated exploit — hovering shows whether it's part of a known **exploit kit** or tied to a specific APT/activity group, sometimes linking to a Threat Analytics report.
- The **breach insights** icon is highlighted when the vulnerability has actually been **found/observed in your organization**.
- Vulnerability details flyout: **exposed devices**, **installed devices**, and **associated CVEs**.

### Exceptions and remediation requests
- **Create an exception** is the documented alternative path when a recommendation isn't relevant at the moment.
- **Request remediation**: opens a form to specify what's being remediated, an optional **"Open a ticket in Intune"** checkbox (scoped to **Microsoft Entra ID–joined devices**), a due date, and priority.
- Submitting creates a trackable **security task** visible on the **Remediation** page, which can be marked completed.

### Inventories
Microsoft's "consolidated inventories" cover: **software applications**, **digital certificates**, **hardware and firmware**, and **browser extensions** (including extension permissions/risk level).

### Weaknesses and Event timeline
- **Weaknesses**: lists CVEs affecting the org, with an **exposed devices** filter.
- **Event timeline**: plots vulnerability-related events over time, including new vulnerabilities and zero-days.

### Baseline assessment
You create a **security baseline profile** against an industry benchmark — supported benchmarks are **CIS (Center for Internet Security)** for Windows 10/11/Server, and **STIG (Security Technical Implementation Guides)** for Windows 10 — and Defender continuously monitors compliance drift in real time rather than relying on periodic scans.

**Setup steps — create a baseline profile:**
1. **Vulnerability management → Baselines assessment → Profiles tab → Create profile.**
2. Enter a name and description → Next.
3. On the **Baseline profile scope** page, set the benchmark (CIS or STIG), applicable software, and compliance level → Next.
4. Select the specific configurations to include in the profile → Next.
5. Choose the device groups/tags to include → Next.
6. Review → **Submit** to create the profile.
7. Select **View profile page** to see live assessment results.

*Verified against: [Defender Vulnerability Management overview page](https://learn.microsoft.com/en-us/defender-vulnerability-management/tvm-dashboard-insights), [Security recommendations](https://learn.microsoft.com/en-us/defender-vulnerability-management/tvm-security-recommendation), [Remediate vulnerabilities](https://learn.microsoft.com/en-us/defender-vulnerability-management/tvm-remediation), and [Security baselines assessment](https://learn.microsoft.com/en-us/defender-vulnerability-management/tvm-security-baselines) (Microsoft Learn).*

## Setting up Device Discovery

🚨 **The "Windows authenticated scan" was deprecated on December 18, 2025** and is no longer supported or available in the portal at all — Microsoft has stated there's no direct replacement. The **network device authenticated scan** (for switches, routers, firewalls, etc.) is unaffected and still current.

### Discovery modes
- Location: **Settings → Device discovery**.
- Current terminology: **"Basic scan"** and **"Standard scan."**
- **Basic scan**: passive only — collects network events/traffic seen by onboarded devices via the `SenseNDR.exe` binary, generates **no additional network traffic**, and gives limited visibility. Good for sensitive/legacy networks.
- **Standard scan** (recommended/default): actively probes using common discovery protocols and multicast queries; re-scans a device roughly **no more than once every three weeks**; each active probing attempt generates **up to 50KB of traffic**. Runs via Microsoft-signed PowerShell scripts from `C:\ProgramData\Microsoft\Windows Defender Advanced Threat Protection\Downloads\*.ps1`.
- Standard scanning **only targets unmanaged/unknown devices** — devices already onboarded to Defender are never actively probed, which is why there's generally no security concern about the probing looking "malicious" to monitoring tools.
- You can restrict which onboarded devices perform standard discovery (e.g., via device tag).

### Exclusions
You can exclude specific IP addresses/subnets (with a description) from **standard (active)** scanning — excluded devices fall back to passive-only (basic-like) discovery, they aren't skipped entirely.

### Monitored networks
- Defender distinguishes corporate vs. non-corporate networks and by default only discovers/lists devices on corporate networks (e.g., a user's home network is normally **ignored**, not monitored, by default).
- Defender considers a network "corporate" if most devices in the org report the **same network name, default gateway, and DHCP server address**.

### Enterprise IoT
Enabling this integrates with **Defender for IoT** to extend discovery/vulnerability assessment to enterprise IoT/OT devices (printers, cameras, medical devices, industrial control systems, etc.). Full vulnerability assessment on discovered IoT devices additionally requires the **Enterprise IoT Security license** (via Microsoft 365 E5 or a standalone add-on).

### Authenticated scans
- "Windows authenticated scan" no longer exists (deprecated Dec 18, 2025).
- **Network device authenticated scan** (still current): agentless discovery/assessment of network infrastructure (switches, routers, WLAN controllers, firewalls, VPN gateways) — e.g., Cisco IOS, Juniper, Palo Alto.
- Network device scans use **SNMP (read-only)**, supporting both **SNMPv2 and SNMPv3**.
- You designate specific onboarded Windows devices as "scanners" that periodically scan the target network devices/segment. The Scanner software is supported on **Windows 10 version 1903+ / Windows Server 1903+**, and you can install **up to 40 scanners per tenant**.
- Configuring scan jobs requires the **"Manage security settings"** permission in Defender.
- Discovered network devices feed into the standard vulnerability management workflow (recommendations, vulnerabilities) just like endpoints.

**Setup steps — install the scanner:**
1. **Settings → Device discovery → Authenticated scans.**
2. Download the scanner and install it on the designated onboarded Defender for Endpoint device.
3. Register it: copy/follow the URL shown on the command line, enter the installation code, and sign in with an account holding **Manage security settings** permission (registration can be completed from any device, not just the scanner itself, as long as the sign-in is in the same tenant).

**Setup steps — configure a new network device authenticated scan:**
1. **Settings → Device discovery → Authenticated scans → Add new scan → Network device authenticated scan → Next.**
2. Choose whether to **Activate scan**, and give it a **Scan name**.
3. Select the **Scanning device** (the onboarded device that will run the scan).
4. Enter the **Target (range)** — IP address ranges/hostnames, typed in or imported via CSV (import overrides manual entries).
5. Set the **Scan interval** — every 4 hours by default, adjustable, or **Don't repeat** for a one-time scan.
6. Choose the **Authentication method** (Community String, noAuthNoPriv, authNoPriv, authPriv) — credentials can be entered directly or pulled from **Azure Key Vault**.
7. **Next** to run or skip a one-time test scan (verifies connectivity + credentials; supports up to 1,024 addresses, each scanning device up to 1,500 successful IPs).
8. **Next** to review, then **Submit** to create the scan job.

Newly discovered devices show up under **Device inventory → Network devices**, typically within about 2 hours.

*Verified against: [Device discovery overview](https://learn.microsoft.com/en-us/defender-endpoint/device-discovery), [Set up authenticated network scans](https://learn.microsoft.com/en-us/defender-endpoint/network-devices), and [Defender Vulnerability Management FAQ — Windows authenticated scan deprecation](https://learn.microsoft.com/en-us/defender-vulnerability-management/defender-vulnerability-management-faq#windows-authenticated-scan-deprecation-faqs) (Microsoft Learn).*

## Manage Resources with Azure Arc

🚨 The classic "Getting started → Add non-Azure servers → Configure → download agent, enter workspace ID + primary/secondary key" flow is the **legacy Log Analytics agent (MMA)** onboarding method. That agent was **retired on August 31, 2024**, and the onboarding UI itself (the "Add non-Azure servers" workspace/key experience under Getting started/Inventory) has since been **removed from the Defender for Cloud portal entirely** — you will not find this screen in a current tenant. After **March 2, 2026**, data upload from any remaining Log Analytics agents can stop at any time without further notice. Note also that this legacy workflow isn't actually "Azure Arc" — true Azure Arc onboarding (installing the Arc `azcmagent` to register a machine as an Arc-enabled server) is a separate, still-current path.

### Current guidance for connecting non-Azure/on-prem/multicloud machines
1. **Azure Arc-enabled servers** — required for full **Defender for Servers Plan 2** feature parity.
2. **Direct onboarding via Defender for Endpoint** — a simpler, agentless-from-Defender's-perspective method (no separate Log Analytics agent needed).

### Direct onboarding
- Location: **Defender for Cloud → Environment settings → Direct onboarding**.
- Flip the toggle **On** — you must also **select a subscription** (used for licensing, billing, alerts, and security insights for these servers) as part of turning it on.
- Non-Azure servers **may take up to 24 hours** to appear after first enabling this.
- Required permissions: **Subscription Owner** on the selected subscription, plus **Microsoft Entra Security Administrator** (or higher) on the tenant.
- Direct onboarding applies automatically to **both existing and new** Defender for Endpoint–onboarded servers across the same Entra tenant once turned on.
- Direct onboarding gives you full **Defender for Servers Plan 1** features. If you're on **Plan 2**, some Plan 2 features (and full Azure management capabilities like Azure Policy/Guest Configuration) **still require Azure Arc** — Direct onboarding alone doesn't give complete Plan 2 parity.
- Direct onboarding also supports **AWS and GCP VMs** via the Defender for Endpoint agent directly (not just on-prem machines).
- This capability is **Generally Available (GA)**.

*Verified against: [Onboard non-Azure servers with Defender for Endpoint](https://learn.microsoft.com/en-us/azure/defender-for-cloud/onboard-machines-with-defender-for-endpoint) and [Prepare for retirement of the Log Analytics agent](https://learn.microsoft.com/en-us/azure/defender-for-cloud/prepare-deprecation-log-analytics-mma-agent) (Microsoft Learn).*

## Integrating AWS into Defender for Cloud

### Prerequisites
- Azure subscription with Defender for Cloud enabled + **Contributor** permission on the Azure subscription.
- On the AWS side, what matters is having sufficient AWS rights to **deploy the CloudFormation (or Terraform) stack** that creates the connector's IAM roles during setup.
- If you want **CIEM** (Cloud Infrastructure Entitlement Management) as part of Defender CSPM, you additionally need the **Security Admin** role plus **Application.ReadWrite.All** permission on the Entra tenant.
- The AWS connector **isn't available on national government clouds** (Azure Government, Azure operated by 21Vianet).
- If an AWS account is **already connected to Microsoft Sentinel**, you can't connect it to Defender for Cloud the normal way — a separate specific procedure is required.
- Native connector **plan requirements are extensive and plan-specific** — e.g., Defender for Containers specifically needs an Amazon EKS cluster plus the ability to create an SQS queue, Kinesis Data Firehose stream, and S3 bucket in the same region as the cluster.

### Onboarding choice: Management account vs. Single account
When adding the AWS environment, you choose between:
- **Management account**: connects an AWS Organizations management account; auto-provisioning then creates connectors automatically for discovered member accounts and any newly created accounts under it. Only the actual AWS management account is supported — delegated administrator accounts are not supported.
- **Single account**: connects just one AWS account.

### Additional wizard details
- You choose which **AWS regions** to protect (all selected by default).
- You choose a **scan interval** — every 4, 6, 12, or 24 hours (some data collectors run on fixed intervals regardless).
- For management-account onboarding, you can specify AWS account IDs to **exclude**.
- Deployment method is either **AWS CloudFormation** or **Terraform** — Terraform onboarding is supported even though its tab isn't shown in the UI when onboarding a management account.
- Authentication uses **federated trust and short-lived credentials** — no long-lived secrets are stored by Defender for Cloud.
- After setup, security recommendations typically **appear within a few hours**.
- You can verify connector health afterward via **Environment settings → [account] → Connectivity status**.

*Verified against: [Connect your AWS account to Microsoft Defender for Cloud](https://learn.microsoft.com/en-us/azure/defender-for-cloud/quickstart-onboard-aws) (Microsoft Learn).*

## Integrating GCP into Defender for Cloud

### Prerequisites
- Azure subscription with Defender for Cloud enabled + **Contributor** permission on the Azure subscription.
- On the GCP side, whoever runs the onboarding `gcloud` script needs enough GCP rights to create service accounts, workload identity pools, and IAM policy bindings (project/organization Owner-level access is a practical baseline).
- Enabling **CIEM** as part of Defender CSPM additionally requires the **Security Admin** role and **Application.ReadWrite.All** permission on the Entra tenant.

### Onboarding choice: Organization vs. Single project
- **Organization**: onboards at the GCP organization level (enter the organization ID; optionally exclude specific project numbers or folder IDs).
- **Single project**: onboards one project (project number + project ID).

### Access configuration
GCP onboarding only uses a generated `gcloud` Cloud Shell script — there's no separate deployment-method selection step like AWS has. You choose the permission type (**Default access** or **Least privilege access**) and then run the one generated script.

### Additional details
- You also select a **scan interval** (every 4, 6, 12, or 24 hours); some data collectors (e.g., ComputeInstance-related ones) run on a **fixed 1-hour** interval regardless of your setting.
- The onboarding script requires these GCP APIs to be enabled on the project: `iam.googleapis.com`, `sts.googleapis.com`, `cloudresourcemanager.googleapis.com`, `iamcredentials.googleapis.com`, `compute.googleapis.com` (enabled on the management project for org-level onboarding).
- The script provisions: a **workload identity pool**, a **workload identity provider** (per plan), **service accounts**, and **project-level policy bindings**.
- Authentication uses **workload identity federation + service account impersonation** — no long-lived credentials stored, same federated-trust model as AWS.
- GCP projects connect to Defender for Cloud at the **project level**; you can connect multiple GCP projects to a single Azure subscription, or spread across multiple subscriptions.
- After onboarding, recommendations typically appear within a few hours; if **autoprovisioning** is enabled, **Azure Arc** and any enabled extensions install automatically on newly detected resources — current GCP-connected server protection flows through Defender for Endpoint integration or agentless scanning rather than the retired Log Analytics agent.

*Verified against: [Connect your GCP project to Microsoft Defender for Cloud](https://learn.microsoft.com/en-us/azure/defender-for-cloud/quickstart-onboard-gcp) (Microsoft Learn).*

## Working with Defender for Cloud Security Alerts

Alerts across Azure and connected multi-cloud resources (AWS, GCP, Azure Arc–onboarded Kubernetes, etc.) all surface in one place under **Security alerts**, with "View full details" and a "Take action" tab.

### "Take action" tab — five specific, named actions
- **Inspect resource context** — links to the resource's activity logs supporting the alert.
- **Mitigate the threat** — manual, step-by-step remediation instructions for this specific alert.
- **Prevent future attacks** — related security recommendations to harden the resource and reduce attack surface going forward.
- **Trigger automated response** — option to invoke a **Logic App** in response to the alert.
- **Suppress similar alerts** — option to suppress future alerts with similar characteristics, if not relevant to your org.

### Sample alerts
Defender for Cloud has a built-in **Sample alerts** generator, accessed from the toolbar on the Security alerts page. Requires the **Subscription Contributor** role; you pick a subscription and the specific Defender plan(s) you want sample alerts for. Alerts typically appear within a few minutes, and they also flow through to anywhere else you've configured to receive alerts (connected SIEMs like Sentinel, email notifications, workflow automation) — a legitimate way to validate your full alerting pipeline, not just a cosmetic demo feature.

*Verified against: [Manage and respond to security alerts](https://learn.microsoft.com/en-us/azure/defender-for-cloud/managing-and-responding-alerts) and [Validate alerts in Microsoft Defender for Cloud](https://learn.microsoft.com/en-us/azure/defender-for-cloud/alert-validation) (Microsoft Learn).*

---

# Microsoft Sentinel

🚨 **Platform-wide heads-up for this whole section**: Microsoft has announced that **after March 31, 2027, Microsoft Sentinel will no longer be supported in the Azure portal** — it will only be available in the **Microsoft Defender portal** going forward (as part of the unified security operations experience). The underlying concepts (workspaces, tenants, analytics rules, etc.) don't change, but screenshots/menus showing the Azure-portal Sentinel experience will increasingly not match what you see if your tenant has already moved to the Defender portal.

## Planning a Sentinel Workspace

Sentinel is a cloud-native SIEM (in the same category as tools like Splunk or LogRhythm, but delivered as a cloud service) built around four pillars: collect data at cloud scale across users, devices, apps, and infrastructure — on-premises and across multiple clouds (Azure, AWS, GCP); detect previously undetected threats and minimize false positives using Microsoft's analytics; investigate threats with AI assistance; and respond to incidents fast with built-in orchestration and automation (SOAR). Under the hood, Sentinel is deployed onto a **Log Analytics workspace**, so most of its implementation planning is really Log Analytics workspace planning — and the single most important setting when creating that workspace is the **region**, since that's where your log data physically resides.

### Model 1 — Single tenant, single workspace
A single workspace gives you the full Sentinel experience with the simplest queries and a true single pane of glass: consolidated logs, easier cross-resource querying, and access control that combines standard Log Analytics RBAC with Sentinel-specific roles. A workspace is tied to a specific Azure region, so bringing in data from other regions raises both cross-region bandwidth cost and data sovereignty/regulatory compliance concerns.

### Model 2 — Single tenant, regional workspaces
Legitimate drivers: sovereignty/regulatory compliance, split billing (by placing workspaces in separate subscriptions), and granular access control. Historically this model meant no central pane of glass and duplicated analytics/workbook deployment, but Microsoft now offers tooling that meaningfully softens this: **Workspace Manager** (centrally deploy and manage content like analytics rules and workbooks across multiple workspaces from one place) and **cross-workspace queries/analytics rules** (using the `union` operator and `workspace()` expression) let you search and correlate across workspaces without a single merged repository. Table-level retention settings also now let you set different retention periods per data type **within a single workspace**, removing one of the historical reasons orgs used to split into multiple workspaces just for retention differences.

### Model 3 — Multi-tenant workspaces via Azure Lighthouse
**Azure Lighthouse** is Microsoft's mechanism for a central/global SOC (or an MSSP) to manage multiple Sentinel instances living in different Microsoft Entra tenants — you still choose single-region vs. multi-region workspace design *within* each tenant, with the same tradeoffs as models 1 and 2. For organizations with **multiple Microsoft Entra tenants, separate workspaces per tenant aren't just a design preference — they're a hard technical requirement** in many cases: Sentinel's built-in service-to-service data connectors (e.g., Azure Firewall, Azure Storage, Azure Activity, Microsoft Entra ID — anything based on diagnostic settings) can only send data to a workspace **within the same tenant** as the resource. This is why Microsoft's own MSSP guidance recommends **at least one workspace per Entra tenant** as a best practice, not merely an option.

*Verified against: [What is Microsoft Sentinel?](https://learn.microsoft.com/en-us/azure/sentinel/overview), [Prepare for multiple workspaces and tenants in Microsoft Sentinel](https://learn.microsoft.com/en-us/azure/sentinel/prepare-multiple-workspaces), and [Sample Microsoft Sentinel workspace designs](https://learn.microsoft.com/en-us/azure/sentinel/sample-workspace-designs) (Microsoft Learn).*

## Creating a Workspace and Working with Retention Settings

### Workspace creation and linking Sentinel
- Flow: search **Microsoft Sentinel** → **Create** → **Create a new workspace** → resource group, name, region → **Review + Create** → **Create**. Then back in Sentinel, select the new workspace → **Add**.
- 🚨 If you (or a new Sentinel customer) have **Owner** or **User Access Administrator** permissions on the subscription, your workspace is now **automatically onboarded straight into the Defender portal** — you may never see the classic Azure-portal Sentinel creation flow at all.
- You can run Sentinel on more than one workspace, but data itself is isolated per workspace.
- Default workspaces auto-created by Microsoft Defender for Cloud don't appear in the workspace-selection list — you can't install Sentinel on them.
- Once Sentinel is deployed on a workspace, that workspace can no longer be moved to a different resource group or subscription — a one-way-door constraint.
- Specific permissions required: **Contributor** on the subscription to enable Sentinel; **Microsoft Sentinel Contributor** or **Microsoft Sentinel Reader** on the resource group to use it; **Microsoft Sentinel Contributor** specifically to install/manage Content Hub solutions.

### Retention
Microsoft's own onboarding documentation recommends raising retention to **90 days** to make sure you can use all Microsoft Sentinel functionality and features — 30 days (the workspace default) isn't a recommended steady-state choice for a working Sentinel deployment.

Retention has **two components**: **Analytics retention** (data queryable in near-real-time; extendable up to **730 days / 2 years** for Analytics-plan tables) and **long-term/archive retention** (cheaper, accessed via search jobs; extendable up to **12 years**). The workspace-level slider sets the *default analytics retention* for tables that haven't been individually customized.

- A few tables get **90 days of retention for free by default already**, regardless of the workspace default — specifically **`Usage`** and **`AzureActivity`**, plus Application Insights tables (`AppRequests`, `AppExceptions`, etc.).
- Changing the table-level retention for one table (via **Tables → [table] → Manage table**) only affects that table — tables still inheriting the workspace default are unaffected, and previously-customized tables aren't reset when you later change the workspace-wide default.
- When you **shorten** a table's total retention, Azure Monitor waits **30 days before actually deleting** the data — giving you a window to revert a misconfiguration before data loss.
- Configuring retention requires the **Log Analytics Contributor** role (to set it) or **Log Analytics Reader** (to just view current settings).

*Verified against: [Onboard to Microsoft Sentinel](https://learn.microsoft.com/en-us/azure/sentinel/quickstart-onboard) and [Manage data retention in a Log Analytics workspace](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/data-retention-configure) (Microsoft Learn).*

## Configure Microsoft Sentinel Roles

### Built-in role descriptions
- **Microsoft Sentinel Reader**: view data, incidents, workbooks, recommendations, and other resources.
- **Microsoft Sentinel Responder**: all Reader permissions, plus manage incidents (assign, dismiss).
- **Microsoft Sentinel Contributor**: all Responder permissions, plus install/update Content Hub solutions and create/edit resources (workbooks, analytics rules, etc.).
- **Microsoft Sentinel Playbook Operator**: does *not* incorporate the other three roles — list, view, and manually run playbooks only.
- **Microsoft Sentinel Automation Contributor**: allows Sentinel to add playbooks to automation rules; explicitly **not intended for user accounts**.

### Recommended role assignments by user type
- **Security analysts**: Sentinel Responder + Sentinel Playbook Operator.
- **Security engineers**: Sentinel Contributor + **Logic App Contributor** (an Azure-specific, non-Sentinel role).
- **Service principals**: Sentinel Contributor, for automated configuration/management tasks.

**Setup steps — assign a Sentinel role (Azure portal):**
1. Navigate to the **resource group** containing the Sentinel workspace (recommended assignment scope) → **Access control (IAM)**.
2. **Add → Add role assignment.**
3. Select the role (e.g., Microsoft Sentinel Responder, Contributor, Reader, Playbook Operator, Automation Contributor).
4. On the **Members** tab, assign access to a user, group, or service principal.
5. **Review + assign** to save.

### Additional details
- **Role assignments are cumulative** — a user assigned both Reader and Contributor effectively gets Contributor-level access; it's easy to unintentionally over-grant permissions this way.
- Microsoft recommends assigning these roles at the **resource group level** containing the Sentinel workspace, specifically because it also covers related resources like **Logic Apps/playbooks** automatically. If you instead assign roles directly at the **workspace** level, you must separately assign the same roles to the **SecurityInsights** solution resource within that workspace (and potentially other resources) — meaningfully more overhead than the resource-group approach.
- The ability to **create/edit workbooks** specifically also requires the separate **Workbook Contributor** role alongside Sentinel Contributor (or a lesser Sentinel role) — it's not purely bundled into Sentinel Contributor.
- For **guest users** to be able to assign incidents, they need **Directory Reader** (a Microsoft Entra ID role — regular non-guest users get this by default) **in addition to** Sentinel Responder.
- To let Sentinel actually **run playbooks via automation rules**, its special service account needs **explicit permissions granted on the playbook's resource group** — and *your own* account needs **Owner** permissions to grant that. This is the real mechanism behind what the Automation Contributor role enables.
- 🚨 If your workspace has been onboarded to the **Defender portal** and uses the newer **Microsoft Sentinel data lake** capability, a parallel and different RBAC model applies: read/write access there is governed by **Microsoft Entra ID roles** (Global Reader, Security Reader, Security Operator, Security Administrator, Global Administrator) rather than the classic Azure RBAC Sentinel roles described above. Given the ongoing move of Sentinel into the Defender portal, this parallel model is increasingly relevant.

*Verified against: [Roles and permissions in the Microsoft Sentinel platform](https://learn.microsoft.com/en-us/azure/sentinel/roles) (Microsoft Learn).*

## Manage Multiple Workspaces with Workspace Manager

### Prerequisites
- At least **two** Sentinel workspaces: one to manage from (central), at least one to be managed (member).
- **Microsoft Sentinel Contributor** role required on **both** the central workspace and each member workspace being managed.
- **Azure Lighthouse** required if managing workspaces across multiple Microsoft Entra tenants.

### Architectures
- **Direct-link**: least complex — one central workspace controls all member workspaces.
- **Co-management**: supports more than one central workspace managing the same member workspace — typical for a workspace jointly overseen by an in-house SOC **and** an MSSP.
- **N-Tier**: most complex — a central workspace controls another central workspace (e.g., a conglomerate managing subsidiaries, where each subsidiary also manages its own set of workspaces).

### Notes
- 🚨 **Workspace Manager is currently in Preview** and is explicitly scoped to **"Microsoft Sentinel in the Azure portal."** If your workspace is onboarded to the **Defender portal** instead, the equivalent multi-tenant/multi-workspace management capability is a different, separately named feature: **Microsoft Defender multitenant management**.
- Workspace Manager only centrally publishes specific content types: **analytics rules, automation rules (excluding Playbooks themselves), parsers/saved searches/functions, hunting queries, and workbooks.** **Playbooks attached to analytics/automation rules aren't currently supported** for central publishing.
- It only manages content that was **published from the central workspace** — it has no visibility or control over content created locally within a member workspace, and it currently **can't centrally delete** content already published to member workspaces.
- There's a scaling limit: maximum **2,000 "published operations"** per group, where published operations = (number of member workspaces) × (number of content items).

*Verified against: [Manage multiple Microsoft Sentinel workspaces with workspace manager](https://learn.microsoft.com/en-us/azure/sentinel/workspace-manager) (Microsoft Learn).*

## Setting up Workspace Manager

Configuration flow: enable the toggle in Settings on the central workspace, add member workspaces, create a group (name + workspaces + content via "All content" or custom selection), and publish the group — status shows In progress/Succeeded/Failed.

- Before creating a group, you need **at least one active content item already deployed in the central workspace** — otherwise there's nothing to select when defining the group's shared content.
- Choosing **"All content"** when building a group is a **point-in-time snapshot** of whatever's currently active in the central workspace (not templates) — it does **not** dynamically stay in sync. If you add new analytics rules/workbooks/etc. to the central workspace later, they won't automatically flow to the group; you'd need to update the group's content selection and republish.
- Publish status is **all-or-nothing per group** — if even **one** content item in the group fails to publish, the entire group's "Last publish status" shows **Failed**, not partially succeeded.
- When a publish does fail, there's a **"Failed" link** that opens per-item, per-target-workspace failure details. Common causes: the content item was deleted after being added to the group, permissions changed (e.g., the user is no longer Sentinel Contributor on the member workspace), or the member workspace itself was deleted.
- Publishing can also fail outright if it exceeds the **2,000 published-operations** limit per group (member workspaces × content items).

*Verified against: [Manage multiple Microsoft Sentinel workspaces with workspace manager](https://learn.microsoft.com/en-us/azure/sentinel/workspace-manager) (Microsoft Learn).*

## Identify Data Sources to Be Ingested

Data connectors live in the **Content Hub**, are filterable by content type, show related content (analytics rules, hunting queries, playbooks, etc.) that install alongside them, and are configured via **Data connectors (under Configuration) → select connector → Open connector page**, following documented prerequisites/steps until status shows **Connected**.

- Installing "a data connector" in Content Hub actually installs an entire **packaged solution** — the data connector plus its bundled analytics rules, hunting queries, playbooks, and workbooks together. You can't install just the standalone connector out of a solution; the whole solution installs together.
- Some solutions have **dependencies** (e.g., domain solutions, or ones using the unified Azure Monitor Agent for CEF/Syslog/custom logs) — the install wizard offers an **"Install with dependencies"** option to pull in required companion connectors at the same time.
- The install wizard is a multi-tab wizard (Basics: subscription/resource group/workspace → additional tabs per content type, sometimes prompting for **non-Microsoft service credentials**, e.g., for playbooks that need to authenticate to an external system → Review + create).
- Content Hub also supports **bulk install/update** — switch to list view, checkbox multiple solutions, and use one **Install/Update** button, rather than updating connectors one at a time.
- Data connectors are labeled with a **support type** — Microsoft-supported, Partner-supported, or Community-supported — found on each connector's detail page. Worth checking before relying on a connector for a business-critical source.

🚨 Microsoft has announced that **after September 14, 2026**, the **legacy HTTP Data Collector API** will no longer be supported for custom data ingestion into Sentinel/Log Analytics. Any custom connectors or integrations built on that legacy API (as opposed to packaged Content Hub connectors) need to migrate to the **Logs Ingestion API** or the **Codeless Connector Framework (CCF)** before that date to avoid ingestion interruptions.

*Verified against: [Discover and manage Microsoft Sentinel out-of-the-box content](https://learn.microsoft.com/en-us/azure/sentinel/sentinel-solutions-deploy) and [Microsoft Sentinel data connectors](https://learn.microsoft.com/en-us/azure/sentinel/connect-data-sources) (Microsoft Learn).*

## Ingesting CEF and Syslog Data

🚨 **Recurring theme across this course**: Microsoft **retired the legacy Log Analytics agent (MMA) for Sentinel log forwarding on August 31, 2024**. The **Azure Monitor Agent (AMA) is now the only supported method** for all log forwarding — this is the same Log Analytics agent retirement that also affected Azure Arc onboarding, covered earlier in these notes. Anything requiring "the legacy agent" or "Log Analytics agent (MMA)" is deprecated.

### Syslog vs. CEF definitions
- **Syslog**: standard protocol originally developed for Unix systems; predefined message structure (priority, timestamp, hostname, application name, process ID, message text); can run over UDP, TCP, or TLS. AMA supports both **RFC 3164 (BSD Syslog)** and **RFC 5424 (IETF Syslog)**.
- **CEF (Common Event Format)**: vendor-neutral format, described by Microsoft as **"an extension of Syslog," developed especially for SIEM solutions**. Used by firewalls, routers, detection/response tools, IDS, and other network/security appliances.

### Azure Monitor Agent (AMA) + Data Collection Rules
- AMA-based ingestion is built around **Data Collection Rules (DCRs)** — sets of instructions specifying which systems to monitor, which log types/facilities to collect, and filters to apply before ingestion.
- Flow: install the Syslog/CEF solution from Content Hub → create a DCR (name it, select target machine(s) — Azure VMs or **Azure Arc-enabled** non-Azure machines both selectable) → configure facilities + **minimum log level** → Review + create → then run the installation command on the log forwarder to actually deploy AMA and configure the local Syslog daemon.
- In the **portal**, you can only set a **minimum log level** per facility. If you need more granular filtering (e.g., specific log levels rather than just a minimum), that requires going through the **Logs Ingestion API** directly instead of the portal UI.
- If you use the **same facility for both Syslog and CEF** messages, you risk **duplicate ingestion** into both the `Syslog` and `CommonSecurityLog` tables. Fixes: either reconfigure the source device to stop sending CEF-tagged messages via the same facility as plain Syslog, or add a **KQL ingestion-time transformation** to the DCR to filter out CEF messages from the Syslog stream.
- Once ingested, Syslog messages land in the **`Syslog`** table and CEF messages land in the **`CommonSecurityLog`** table.

*Verified against: [Syslog and CEF AMA connectors for Microsoft Sentinel](https://learn.microsoft.com/en-us/azure/sentinel/cef-syslog-ama-overview) (Microsoft Learn).*

## Connecting Microsoft Defender XDR

🚨 The Defender XDR connector is **automatically enabled** when Sentinel is onboarded to the Defender portal — the manual configuration below only applies to **Sentinel in the Azure portal**.

### Flow
- Content Hub install → **Data connectors** → select **Microsoft Defender XDR** → **Open connector page**.
- The connector brings in incidents/alerts/events spanning **Endpoint, Identity, Office 365, Cloud Apps**, and more.
- Final step: **Apply Changes** to save and activate the selected event connections.
- Defender for Endpoint has exactly **10** connectable advanced-hunting tables: DeviceInfo, DeviceNetworkInfo, DeviceProcessEvents, DeviceNetworkEvents, DeviceFileEvents, DeviceRegistryEvents, DeviceLogonEvents, DeviceImageLoadEvents, DeviceEvents, DeviceFileCertificateInfo.
- Separate tabs also exist for Defender for Office 365, Defender for Identity, Defender for Cloud Apps, and "Defender alerts."

### The Configuration area has three parts
1. **Connect incidents and alerts** — before clicking "Connect incidents & alerts," check **"Turn off all Microsoft incident creation rules for these products"** (recommended, to avoid duplicating incidents). This checkbox only appears before the connector is connected.
2. **Connect entities** — syncs **on-premises Active Directory user identities** into Sentinel via **Microsoft Defender for Identity**, configured through the **UEBA (User and Entity Behavior Analytics) configuration page** — toggle UEBA on, check "Active Directory (Preview)," and apply.
3. **Connect events** — the per-product event checkboxes described above.

### Additional details
- Prerequisites: a valid **Defender XDR license**, the **Security Administrator** role (or equivalent) on the target tenant, **read/write** permissions on the Sentinel workspace, and your account must be in the **same Entra tenant** as the Sentinel workspace to make changes.
- Enabling the XDR connector **automatically disconnects any previously-connected standalone Defender component connectors in the background** — they may still *appear* connected in the UI, but no data actually flows through them anymore.
- Switching from standalone connectors to the XDR connector **changes the alert schema**, which can break existing saved queries built against the old schema.
- To verify ingestion, run `SecurityIncident | where ProviderName == "Microsoft XDR"` in Sentinel Logs. Under normal conditions, new Defender XDR incidents typically show up in Sentinel within about **5 minutes**.

*Verified against: [Stream data from Microsoft Defender XDR to Microsoft Sentinel](https://learn.microsoft.com/en-us/azure/sentinel/connect-microsoft-365-defender) (Microsoft Learn).*

## Enabling Defender for Cloud Bi-directional Synchronization

Regular (one-directional) **alert synchronization** is automatic once the Defender for Cloud connector is enabled: when an alert is closed in Defender for Cloud, it displays as closed in Sentinel too. **Bi-directional sync** is the *reverse* direction: closing a **Sentinel incident** that contains a Defender for Cloud alert automatically closes the **original alert back in Defender for Cloud**.

### Prerequisites
- Read/write permissions on the Sentinel workspace.
- **Contributor or Owner** role on the subscription being connected.
- At least one active Defender for Cloud plan enabled per subscription (enabling Defender plans themselves requires the **Security Admin** role).
- Defender for Cloud connector installed from Content Hub.
- A resource provider registered per subscription (the `SecurityInsights` resource provider — appears in the portal as "Microsoft Security Insights" under Resource providers).
- **Bi-directional sync specifically** requires **Contributor or Security Admin** on the subscription.

### Flow
Register resource provider → install connector from Content Hub → open connector page → enable for the desired subscription → enable bi-directional sync → wait a few minutes for sync to apply.

### Additional notes
- 🚨 There are now **two connector variants** — **Subscription-based Microsoft Defender for Cloud (Legacy)**, enabled per subscription, and **Tenant-based Microsoft Defender for Cloud (Preview)**, which covers an entire tenant without enabling each subscription individually.
- 🚨 This connector applies to **Microsoft Sentinel in the Azure portal**. If Sentinel is onboarded to the Defender portal, Defender for Cloud alerts already flow into Defender XDR directly and the Tenant-based connector won't even be listed (the Defender for Cloud *solution* is still worth installing for its built-in content, though).
- A **"Create incidents"** toggle also exists in the connector configuration, controlling whether a default analytics rule auto-generates Sentinel incidents from Defender for Cloud alerts (editable later under Analytics).
- Verification query: `SecurityAlert | where ProductName == "Azure Security Center"` in Sentinel Logs to confirm alerts are flowing.
- Sync (in both directions) can take a few minutes to apply.

*Verified against: [Ingest Microsoft Defender for Cloud alerts to Microsoft Sentinel](https://learn.microsoft.com/en-us/azure/sentinel/connect-defender-for-cloud) (Microsoft Learn).*

## Domain Summary

This course covers: notification rules, Defender for Endpoint advanced features, custom indicators, web content filtering, automatic attack disruption and AIR, device groups/permissions/automation levels, Azure Arc, Defender for Cloud multi-cloud connections, Sentinel workspace planning, Sentinel roles, Workspace Manager, data connectors/ingestion planning, and XDR/Defender for Cloud synchronization.

🚨 **Exam structure changed April 16, 2026.** As of today (July 4, 2026), the current SC-200 skills-measured outline is:

- **Manage a security operations environment: 40–45%**
- **Respond to security incidents: 35–40%**
- **Perform threat hunting: 20–25%**

The exam also dropped from four domains to three — "Configure protections and detections" as a separate fourth domain was eliminated, with its content redistributed into the other three, and "Manage assets and environments" was folded into "Manage a security operations environment" alongside a new "Configure detections" sub-area. This course's material maps most closely to "Manage a security operations environment," now weighted **40–45%** of the exam — nearly the entire first half by itself. Worth reviewing the current skills-measured list directly before your exam date, since domain names and boundaries have shifted.

*Verified against: [Study guide for Exam SC-200: Microsoft Security Operations Analyst](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/sc-200) (Microsoft Learn).*

---

# Mitigate Threats to the Productivity Environment

Module roadmap: data-loss prevention (sensitive information types, custom sensitive information types), sensitivity labels (publishing default labels, creating custom labels), deploying DLP policies, insider risk (common scenarios, insider risk policies), and detecting/investigating/responding to/remediating threats across the productivity environment using Microsoft 365 Defender.

These topics live in **Microsoft Purview** (DLP, sensitivity labels, insider risk management) and **Microsoft Defender for Office 365**, and map to the current SC-200 skills-measured outline under **Respond to security incidents** — specifically "Investigate and remediate threats or compromised entities identified by Microsoft Purview," "Investigate threats by using Audit from Microsoft Purview," and "Investigate threats by using Content Search in Microsoft Purview" — plus "Investigate and remediate threats by using Microsoft Defender for Office 365" in that same domain.

*Verified against: [Study guide for Exam SC-200: Microsoft Security Operations Analyst](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/sc-200) (Microsoft Learn).*

## Understanding DLP

Data loss prevention (DLP) protects sensitive information — financial data, personal information, health records, credit card numbers, and similar — from inappropriate sharing or leakage, in support of both industry regulations and internal business standards. DLP analyzes content using deep content analysis rather than simple text scanning: keyword matches, regular expression evaluation, internal function validation, and secondary (supporting) matches found in proximity to a primary match, plus machine learning for some classifiers.

### Where DLP applies
DLP policies are built around two broad location categories:
- **Enterprise applications & devices**: Exchange, SharePoint, OneDrive accounts, Teams chat and channel messages, Office desktop apps (Word, Excel, PowerPoint), Windows 10/11 and macOS (three latest versions) devices, non-Microsoft cloud apps, on-premises file shares and on-premises SharePoint, Microsoft Fabric and Power BI workspaces, Microsoft 365 Copilot/Copilot chat, and Microsoft Defender for Cloud Apps–connected instances.
- **Inline web traffic** (preview): monitors and protects against oversharing to *unmanaged* cloud apps and AI tools — OpenAI ChatGPT, Google Gemini, DeepSeek, and Microsoft Copilot via Edge for Business or network-level inspection, plus 34,000+ cataloged cloud apps via the network option.

Locations relevant to this course's Exchange/SharePoint/OneDrive/Teams framing are exactly the classic "Enterprise applications & devices" set, but the full location surface today is considerably broader than those four.

Enabling protection on the desktop versions of Word, Excel, and PowerPoint is a real, current capability — DLP policies sync out from the central policy store to Exchange (and from there Outlook/Outlook on the web), OneDrive, SharePoint sites, Office desktop programs, and Teams. This lets users get real-time policy tips and be prompted to classify a document as they work, reinforcing the "prevention through education" idea.

### Core components

**Sensitive information types (SITs)**: pattern-based classifiers used to detect things like national ID numbers or credit card numbers. A SIT's pattern has four parts: a **primary element** (a regular expression — with or without checksum validation — a keyword list, a keyword dictionary, or a function), a **supporting element** acting as corroborative evidence (also a regex, keyword list, or keyword dictionary) that raises confidence when found near the primary element, a **confidence level** (low/medium/high, corresponding to accuracy values of 65/75/85), and a **proximity** value specifying how many characters apart the primary and supporting elements can be and still count as a match. Microsoft ships a large library of built-in SITs, and you can create fully custom ones or copy a built-in one as a starting template.

**Sensitivity labels**: classify content so that protective functionality (like encryption) can be applied based on the label. Microsoft's own default label set (from the Microsoft Information Protection starter kit, not auto-created in every tenant) is **Personal, Public, General, Confidential, and Highly Confidential** — a slightly different naming convention than "private" or "classified." Organizations can also build fully custom labels. Labels can be applied manually by users or automatically (auto-labeling policies).

**DLP policy**: defines the **locations** to protect, plus one or more **rules**, each made of **conditions** (what content must match — e.g., a specific SIT, a sensitivity label, or a sharing pattern) and **actions** (what happens on a match — e.g., block access, encrypt, show a policy tip, or for on-premises files, quarantine).

### Example SIT: Argentina National Identity (DNI) Number
Format: eight digits separated by periods (two digits, period, three digits, period, three digits), no checksum. Microsoft's definition gives it **medium confidence (75)** when the regex pattern matches **and** a supporting keyword (e.g., "DNI," "Documento Nacional de Identidad") is found within **250 characters** — matching the `recommendedConfidence="75"` and `patternsProximity="250"` values in the actual entity XML, along with the `id`, `confidenceLevel`, and `Match` (regex + keyword) structure.

*Verified against: [Learn about data loss prevention](https://learn.microsoft.com/en-us/purview/dlp-learn-about-dlp), [Learn about sensitive information types](https://learn.microsoft.com/en-us/purview/sit-sensitive-information-type-learn-about), and [Learn about the default sensitivity labels and policies](https://learn.microsoft.com/en-us/purview/default-sensitivity-labels-policies) (Microsoft Learn).*

## Demo: Implementing DLP

🚨 The portal has been rebranded: current Microsoft documentation directs admins to the **Microsoft Purview portal** at **purview.microsoft.com** rather than compliance.microsoft.com, with a restructured navigation under **Solutions**. compliance.microsoft.com still resolves today, but new guidance and screenshots are written against the Purview portal's nav, so expect menu paths to look different from this demo depending on when your tenant last saw an update.

### Sensitive information types
Current navigation: **Solutions → Information Protection → Classifiers → Sensitive info types** (an "Overview"/activity-style landing view also still exists). Microsoft ships a very large built-in library — in the same range as the "over 300" mentioned — covering common patterns like national ID numbers, credit card numbers, and bank account numbers across many countries/regions.

**Setup steps — create a custom sensitive information type:**
1. **Information Protection → Classifiers → Sensitive info types → Create sensitive info type.**
2. Enter **Name** and **Description** → Next.
3. **Create pattern** → set the pattern's default **confidence level** (Low/Medium/High).
4. Define the **primary element**: a regular expression (with optional validator), a keyword list, a keyword dictionary, or one of the preconfigured functions.
5. Set the **character proximity**, and optionally add **supporting elements** (same four element types) as corroborative evidence.
6. Review and **Save**.
7. Recommended (not shown in the demo): run a **test/simulation** against a sample file before using the new SIT in a live policy.

### Sensitivity labels
Current navigation: **Solutions → Information Protection → Sensitivity labels**. Default out-of-box labels (where auto-created) are **Personal, Public, General, Confidential, Highly Confidential**, and fully custom labels can be added alongside them.

**Setup steps — create a sensitivity label:**
1. **Sensitivity labels → + Create a label** → name and description.
2. **Define the scope**: Files & other data assets, Emails, Groups & sites, Meetings. (Meetings can't be selected unless Files & other data assets and Emails are also selected.)
3. Configure settings per scope — e.g., **protection settings** (encryption, content marking such as headers/footers/watermarks), **auto-labeling** (including trainable classifiers — "Source code" is a real, ready-to-use pre-built classifier, so using it to auto-detect source code in a dev-environment label is accurate), and group/site-specific protection settings.
4. Save the label; repeat for additional labels, and reorder them if label priority matters.

🚨 **Worth knowing**: Microsoft has been migrating tenants (new tenants created on/after **October 1, 2025**, or ones manually migrated) to a **"modern label scheme."** Under this newer scheme, the older standalone **"Schematized data assets"** scope option (for Data Map–registered sources like SQL, Azure SQL, Synapse, Cosmos DB, AWS RDS) has been **folded into "Files & other data assets"** rather than appearing as its own separate checkbox. Since today is well past that cutover date, don't be surprised if a current tenant's label-scope screen looks slightly different from a "classic scheme" walkthrough like this one — the demo's mention of "schematized data assets" as its own toggle reflects the classic scheme.

**Setup steps — publish labels (label/publishing policy):**
1. Current navigation: **Solutions → Information Protection → Publishing policies** (the underlying concept is still what's commonly called a "label policy").
2. **Publish label** → choose the labels to publish → Add.
3. Choose scope: **Full directory** (all users) or restrict via **administrative units**/specific groups.
4. Configure policy settings relevant to the scope of the labels selected — e.g., require a justification when a user removes a label or lowers its classification, and set a default label for documents, email, and Power BI where applicable.
5. Name the policy and complete the configuration — publishing happens automatically as part of finishing the wizard (there's no separate "publish" step after that).

### Data loss prevention policies
Current navigation: **Solutions → Data Loss Prevention → Policies → + Create policy**.

**Setup steps — create a DLP policy:**
1. Choose a category/template (e.g., Financial data, Medical and health data, Privacy data for a given country/region) or select **Custom** for a fully custom policy.
2. Name and describe the policy.
3. Choose **locations**. The current location list is extensive: Exchange email, SharePoint sites, OneDrive accounts, Teams chat and channel messages, Windows/macOS devices, Microsoft Defender for Cloud Apps instances, on-premises repositories, Fabric and Power BI workspaces, and Microsoft 365 Copilot (preview) — matching the demo's description of an extensive location list, with on-premises and Defender for Cloud Apps among the options to include or exclude.
4. Resolve any validation errors shown on screen before continuing.
5. Define **policy settings**: use the template's default rules, or select **Create or customize advanced DLP rules** to build rules from scratch — each rule has **conditions** (e.g., "Content contains" a specific sensitive info type) and can include detecting when content is shared with people inside or outside the organization.
6. Define **actions** to take on a match (e.g., restrict access/encrypt, or restrict third-party app access if Defender for Cloud Apps is configured), plus any user override settings.
7. Choose the policy's initial mode: **Test the policy first** (simulation, optionally showing policy tips to users), **Turn it on right away**, or **Keep it off** until you're ready.
8. Review and save. Existing policies can be reordered, and typically also copied, exported, or deleted from the policy list.

*Verified against: [Create a custom sensitive information type](https://learn.microsoft.com/en-us/purview/sit-create-a-custom-sensitive-information-type), [Create and publish sensitivity labels](https://learn.microsoft.com/en-us/purview/create-sensitivity-labels), [Learn about sensitivity labels in Microsoft Purview Data Map](https://learn.microsoft.com/en-us/purview/data-map-sensitivity-labels), [Trainable classifier definitions](https://learn.microsoft.com/en-us/purview/trainable-classifiers-definitions), and [Create and deploy a data loss prevention policy](https://learn.microsoft.com/en-us/purview/dlp-create-deploy-policy) (Microsoft Learn).*

## Identifying Insider Risk

Insider Risk Management, part of Microsoft Purview, correlates signals from **Microsoft 365 logs and Microsoft Graph** (file activity, communications, abnormal user behavior) with optional **HR data** (resignation/termination dates, job-level changes) to identify potential malicious or inadvertent insider risk — IP theft, data leakage, and security violations — while pseudonymizing users by default (privacy by design) and enforcing role-based access and audit logging.

The HR signal isn't a native, named integration with a specific system like SAP — the **Microsoft 365 HR connector** imports HR events (resignation date, last day of work, job-level changes) via a **CSV file** that your organization exports from whatever HR system it uses (SAP, Workday, or otherwise). So "connecting to SAP" in practice means exporting the relevant HR fields from SAP into the CSV format the connector expects, not a dedicated SAP-to-Purview integration.

### Range of insider risks
Confirmed categories: data spillage, confidentiality violations, IP theft, fraud, insider trading, and regulatory/policy compliance violations — this list is explicitly non-exhaustive and varies by industry and risk driver (external, like credential-targeting social engineering campaigns, or internal).

### Common scenarios (the three highlighted here map to real, current policy templates)
- **Data theft by departing users**: requires the Microsoft 365 HR connector to be configured; detects activity like downloading files from SharePoint Online, printing files, or copying data to personal cloud/messaging services around a user's resignation or termination date.
- **Data leaks** (accidental or intentional sharing of sensitive/confidential information) — covered by the **Data leaks**, **Data leaks by priority users**, and **Data leaks by risky users** templates.
- **Security policy violations** (including workplace-communication issues like offensive language, threats, and harassment) — covered by the **Security policy violations** family of templates, which can also draw on Microsoft Defender for Endpoint alerts and integrate with a dedicated **Communication Compliance** policy.

The current template library is broader than just these three scenarios — it also includes templates for **patient data misuse** (healthcare, preview), **risky AI usage**, **risky browser usage** (preview), and **risky agents** (preview, for agentic AI risk).

### Workflow
1. **Policies** — built from predefined templates, defining in-scope users, risk indicators, prioritized services, and detection time period.
2. **Alerts** — automatically generated when a risk indicator matches policy conditions; visible on the Alerts dashboard (or the newer **Triage Agent** dashboard — an AI-assisted triage capability not covered in this transcript). New alerts start with a "Needs review" status.
3. **Triage** — reviewers examine alert details, associated user activity, severity, and user profile information, then resolve each alert by opening a new case, assigning it to an existing case, or dismissing it.
4. **Investigate** — within a case, reviewers use **User activity** (an interactive timeline of risk activity), **Content explorer** (files/emails tied to the alert), and **Case notes** to build a full picture. A separate **User activity reports** tool also lets investigators examine a user's activity without formally assigning them to a policy first.
5. **Action** — reviewers can send the user a customizable notice (e.g., a reminder or link to training), resolve the case directly, or **escalate the case** — which specifically transfers case data and management to **Microsoft Purview eDiscovery (Premium)** for formal legal/investigative workflows (preserve, collect, review, analyze, export), rather than just generically "further investigation."

*Verified against: [Learn about Insider Risk Management](https://learn.microsoft.com/en-us/purview/insider-risk-management) and [Set up a connector to import HR data](https://learn.microsoft.com/en-us/purview/import-hr-data) (Microsoft Learn).*

## Demo: Creating an Insider Risk Policy

Insider Risk Management is configured in the **Microsoft Purview portal**, under **Insider Risk Management**. The landing page's setup checklist (Microsoft's current name for it is the **"Recommended actions"** experience) marks each setup item as required or optional and deep-links straight to the relevant settings page — matching the demo's description of an overview-style onboarding checklist.

### Settings
Global settings apply across all policies regardless of template, and include:
- Anonymizing usernames for anyone flagged by a policy (a global toggle, referenced from both the Users dashboard and DLP policy views).
- **Policy indicators** — an extensive, current list spanning content downloads, device-based indicators, browsing indicators, and more.
- **Policy timeframes** — an **activation window** (1–30 days after a triggering event) and **past activity detection** (0–90 days before a triggering event).
- **Intelligent detections**, **Microsoft Defender for Endpoint integration** (for security-violation signal), **priority user groups**, and **alert export** (via Office 365 Management APIs to a SIEM) are all real, current settings pages.
- Not mentioned in the demo but worth knowing: **Adaptive Protection**, a related current capability that dynamically adjusts other Purview controls (like DLP policy strictness or Conditional Access) based on a user's calculated insider risk level.

**Setup steps — create a policy (e.g., Risky browser usage template):**
1. **Insider Risk Management → Policies → Create policy → choose a policy category and template.**
2. Review the template's **prerequisites, triggering events, and detected activities** to confirm it fits — some templates won't generate alerts if their prerequisites aren't met.
3. **Name and description** — the name can't be changed after the policy is created.
4. **Users and groups** — scope the policy to all users or specific users/groups.
5. **Prioritize content** (optional) — focus the policy on specific Teams, SharePoint sites, sensitive information types, or data labels.
6. **Triggering events / indicators** — define what risk activity brings a user into scope for scoring (e.g., risky browser activity indicators for that template).
7. **Thresholds** — choose **default thresholds** (from global settings) or **custom thresholds** per indicator, including anomaly-based thresholds (relative to a user's daily norm) for some indicators.
8. **Indicators** and, where relevant, **device onboarding** — Windows devices need to meet minimum requirements (Windows 10 x64 build 1809+ with the Feb 2020 update, signed in with an active Microsoft Entra account) to support device-based indicators.
9. **Review → Submit** to create and activate the policy.

After a policy is created, expect roughly **24 hours** before it's fully active and starts generating alerts from activity indicators — matching the demo's on-screen message about a delay before enablement.

*Verified against: [Create and manage Insider Risk Management policies](https://learn.microsoft.com/en-us/purview/insider-risk-management-policies), [Learn about Insider Risk Management settings](https://learn.microsoft.com/en-us/purview/insider-risk-management-settings), [Set policy timeframes in Insider Risk Management](https://learn.microsoft.com/en-us/purview/insider-risk-management-settings-policy-timeframes), and [Get started with Insider Risk Management](https://learn.microsoft.com/en-us/purview/insider-risk-management-configure) (Microsoft Learn).*

## Defining Email Protection

Microsoft Defender for Office 365 protects email through **Safe Attachments**, **Safe Links**, and **anti-phishing** policies.

### Safe Attachments
Provides zero-day/unknown-malware protection by detonating attachments in a virtual sandbox environment before delivery; scanning typically completes within about 15 minutes, though it can take longer. There's no default Safe Attachments policy — the **Built-in protection** preset security policy automatically covers everyone not otherwise scoped by a Standard/Strict preset or a custom policy, so organizations aren't unprotected by default even without custom configuration.

The current, real action set is:
- **Off** — not recommended except for recipients who only receive mail from trusted senders (turning it off also means Zero-hour auto purge won't quarantine messages later if a threat signal wasn't received at delivery time).
- **Monitor** — delivers the message and tracks what happens to detected threats; can optionally redirect messages with detected attachments to a specified address for analysis.
- **Block** — the default and recommended action; quarantines messages with detected attachments (admin-only release by default) and auto-blocks future instances of the same attachment.
- **Dynamic Delivery** — delivers the message body immediately with a placeholder for each attachment (most PDFs/Office docs can be safely previewed while scanning completes); works only for Exchange Online mailboxes.

🚨 **"Replace" is no longer a current action** — current Microsoft documentation doesn't list it among Safe Attachments options at all; it has been deprecated/removed. If your lab or exam material still shows a "Replace" option, treat it as outdated.

A new or updated custom Safe Attachments policy typically takes about **30 minutes** to take effect. Multiple policies are evaluated by **priority**, with the first (highest-priority) matching policy applied and no further policies evaluated after that.

### Safe Links
Provides both mail-flow-time URL rewriting/scanning for inbound email and **time-of-click verification** for links, spanning Microsoft 365 apps for enterprise (Windows and Mac), Office on the web, Office mobile apps (Android/iOS), and Microsoft Teams (chats, group chats, channels, and tabs).

Teams protection works differently from email: Safe Links checks a clicked Teams link against a list of known malicious links at click-time, but **URLs in Teams aren't rewritten** the way email URLs are — a distinction the demo didn't call out. New or updated Safe Links policies can take up to about **30 minutes** to apply.

Core configurable options include: the action for unknown/potentially malicious URLs, extending that protection to URLs found inside file attachments, applying the same real-time checks to links sent internally (not just external mail), and preventing users from bypassing a warning to reach a flagged destination URL.

### Anti-phishing policies
Checks incoming messages for phishing indicators, centered on two related but distinct protections:
- **Impersonation protection** (user, domain, and sender impersonation) — 🚨 **not enabled by default**, even in the default anti-phishing policy; it has to be explicitly configured (add protected users/domains, sensitivity thresholds, mailbox intelligence, etc.) before it does anything.
- **Spoof intelligence** — evaluates sender legitimacy based on mail-flow patterns and sending infrastructure; this one **is** enabled by default and Microsoft recommends leaving it on.

Related settings not mentioned in the demo: configurable **safety tips** shown to end users (first-contact safety tip, user impersonation safety tip, domain impersonation safety tip) and per-detection **actions** (e.g., quarantine a message detected as user impersonation, with an associated quarantine policy).

*Verified against: [Safe Attachments in Microsoft Defender for Office 365](https://learn.microsoft.com/en-us/defender-office-365/safe-attachments-about), [Set up Safe Attachments policies](https://learn.microsoft.com/en-us/defender-office-365/safe-attachments-policies-configure), [Complete Safe Links overview](https://learn.microsoft.com/en-us/defender-office-365/safe-links-about), [Configure anti-phishing policies](https://learn.microsoft.com/en-us/defender-office-365/anti-phishing-policies-mdo-configure), and [Spoof intelligence insight](https://learn.microsoft.com/en-us/defender-office-365/anti-spoofing-spoof-intelligence) (Microsoft Learn).*

## Investigating DLP Alerts

Current navigation (Microsoft Purview portal): **Data Loss Prevention → Overview → Activities**, which links into **Activity explorer**. Activity explorer gathers DLP policy-match events across Exchange, SharePoint, OneDrive, Teams chat and channels, and — not mentioned in the demo — **on-premises SharePoint folders, libraries, and file shares** as well.

Per-location alert detail is accurate: SharePoint/OneDrive alerts show the offending user, file, sensitive info type, rule actions, and the exact site/folder/file path; Exchange alerts show the policy, email subject, and recipient(s), including whether the message stayed internal; Teams alerts show sender, receiver, and the sensitive info type matched. Alert details generally also include the specific content that matched the rule and the name of whoever last modified the item — useful context beyond what the demo called out.

From the **Alerts** view, opening **View details** on an alert moves into the deeper investigation view, from which you can take action and review a **Management log** of everything done on that alert so far. Managing an alert lets you set its status (**Active, Investigating, Dismissed,** or **Resolved**), add comments, and assign ownership — this is the mechanism behind the "management log" the demo references.

Two things worth adding beyond what the demo shows:
- DLP alerts and incidents can also be investigated from the **Microsoft Defender portal**, under **Incidents & alerts → Incidents**, filtering by **Service Source: Data Loss Prevention** — a second, increasingly relevant investigation surface alongside the Purview DLP Alerts dashboard/Activity explorer shown here.
- A newer **Triage Agent** (Copilot in Purview) capability now exists for DLP alerts too, offering AI-assisted first-pass triage similar to the one already noted for Insider Risk Management — not covered in this demo, but a current option worth knowing about.

*Verified against: [Get started with the DLP Alerts dashboard](https://learn.microsoft.com/en-us/purview/dlp-alerts-dashboard-get-started), [Get started with Activity explorer](https://learn.microsoft.com/en-us/purview/data-classification-activity-explorer), and [Investigate data loss alerts with Microsoft Defender XDR](https://learn.microsoft.com/en-us/defender-xdr/dlp-investigate-alerts-defender) (Microsoft Learn).*

## Investigating Insider Risk

The **Alerts** view lists policy matches with anonymized usernames, alert type, status, severity, time detected, and any linked case plus that case's status — matching the demo closely.

The **Cases** dashboard gives an all-up view of active cases and cases opened over the past 30 days, with case name and (anonymized) users involved, filterable by status, date opened, and date last updated. One small terminology note: case status is **Active** or **Closed** — not "open," as said in passing during the demo.

Opening a case surfaces four tabs, one more than the demo covers:
- **Case overview** — case ID, status, creation date/time, and the user's current **risk score** (the demo attributes the risk score to the User activity tab, but it's actually surfaced on Case overview).
- **User activity** — a visual timeline of all potentially risky activity tied to the case's alerts, used to judge whether the activity could escalate to a security incident. Matches the demo's description well.
- **Content explorer** — copies of the actual files and email messages associated with the case's alerts (e.g., if an alert fired because a user downloaded hundreds of SharePoint files, those files are captured into the case), filterable to narrow down to specific evidence — matches the demo closely.
- **Case notes** — not shown in the demo: a dedicated tab for reviewers to share comments, feedback, and insights on the case, consolidated in one place for collaboration.

*Verified against: [Take action on Insider Risk Management cases](https://learn.microsoft.com/en-us/purview/insider-risk-management-cases) and [Investigate Microsoft Purview Insider Risk Management activities](https://learn.microsoft.com/en-us/purview/insider-risk-management-activities) (Microsoft Learn).*

---

# Mitigate Endpoint Threats Using Defender for Endpoint

Module roadmap: initial Defender for Endpoint configuration (alert notifications, advanced features), managing alerts and incidents plus custom detections, threat and vulnerability management, endpoint investigation and response actions (device isolation, investigation packages, etc.), attack surface reduction rules, and threat analytics.

Data retention defaults are confirmed accurate: Defender for Endpoint data is retained for **180 days** by default, while the **advanced hunting** query window looks back **30 days** by default — that window can be extended beyond 30 days by streaming the relevant Defender tables into Microsoft Sentinel and setting a longer retention period there.

*Verified against: [Microsoft Defender for Endpoint data storage and privacy](https://learn.microsoft.com/en-us/defender-endpoint/data-storage-privacy) and [Advanced hunting overview in Microsoft Defender XDR](https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-overview) (Microsoft Learn).*

## Configuration Overview

Defender for Endpoint configuration centers on two areas: **onboarding devices** and **capabilities**.

Onboarding methods currently include: a **local script** (recommended only for small-scale/manual onboarding — 10 devices or fewer; not intended for production-scale rollouts), **Group Policy** (download a GP configuration package from the Defender portal), **Microsoft Intune** (Intune automatically receives an onboarding configuration package once the service connection to Defender for Endpoint is established), and **Microsoft Configuration Manager** (deploys a client onboarding configuration file and can monitor onboarding status/agent health). A newer, lightweight **Defender deployment tool** (self-updating, for both Windows and Linux) also exists as a current onboarding option beyond the classic four methods.

Capabilities toggles are where features like **live response**, **automated investigation and response (AIR)**, integration across the Defender ecosystem, and **preview features** get turned on — this maps directly to the **Advanced features** settings page covered earlier in these notes. **Custom threat indicators** (IoCs) — file hash, IP address, URL/domain, and certificate indicators, as detailed earlier in this document — are configured as part of this same capabilities area.

*Verified against: [Onboard Windows Servers using a local script](https://learn.microsoft.com/en-us/defender-endpoint/configure-endpoints-script) and [Onboarding using Microsoft Intune](https://learn.microsoft.com/en-us/defender-endpoint/onboarding-endpoint-manager) (Microsoft Learn).*

## Demo: Working with the Configuration, Advanced Features, and Custom Indicators

Navigation: **Microsoft Defender portal → Settings → Endpoints**, which houses Advanced features, Licenses, Email notifications, Auto remediation, Permissions and roles, Rules, Configuration management, Device management, and Network assessment.

- **Advanced features**: most toggles are off by default (matches the demo), covering things like Automated Investigation, Live Response (including a separate toggle for servers), and integrations across the Defender ecosystem — Defender for Identity, Office 365 (Threat Intelligence signals), and Defender for Cloud Apps — plus preview features. All of this maps to the full Advanced features list already detailed earlier in these notes.
- **Auto remediation**: a read-only rollup of device groups' automation levels — it's blank until you create at least one device group and assign it an automation level (Full remediation, one of the Semi-automation tiers, or No automated response), at which point it appears here.
- **Permissions and roles**: the classic RBAC model covered earlier in these notes (Roles, Device groups).
- **Rules**: **Alert suppression**; **Indicators** (file hash, IP/URL/domain, and certificate indicators, all covered in depth earlier — plus **process memory indicators**, which are populated automatically as investigations surface them rather than being manually created); **Web content filtering** (policies enforced through Defender for Endpoint, scoped to device groups); and two separate settings pages the demo blends together — **Automation uploads** (a content-analysis toggle controlling which suspicious file types get auto-uploaded to the cloud for deeper Automated Investigation analysis; Microsoft retains submitted files for 6 months) and **Automation folder exclusions** (a distinct page specifying folders/subfolders, extensions, or filenames that Automated Investigation and Remediation should skip — these exclusions are still subject to regular antivirus scanning).
- **Configuration management**: lets you enforce security settings via Microsoft Intune/Endpoint Manager, tailored to Windows client, server, or both.
- **Device management (onboarding/offboarding)**: covers a wide range of Windows and non-Windows operating systems, each with its own deployment steps, plus a post-onboarding detection test to confirm the device is reporting correctly. Offboarding follows the same structure with a modified script/package.

🚨 **Worth updating from the demo's Windows Server 2008 example**: the demo's walkthrough for onboarding **Windows Server 2008 R2** via the **Microsoft Monitoring Agent (MMA)** reflects the legacy path. Microsoft released **new agents that remove the MMA dependency** — for Windows Server 2012 R2 and Windows Server 2016 back in April 2022, and, more recently, for **Windows 7 SP1 and Windows Server 2008 R2 SP1 in May 2026** (about two months before today). The current recommended onboarding method for these older operating systems is the **Defender deployment tool**, not MMA — worth checking which path your own lab/exam material assumes, since this changed very recently.

**Network assessment** in this settings area is the same **network device authenticated scan** capability already detailed under "Setting up Device Discovery" earlier in these notes (SNMP-based scanning of switches, routers, firewalls, etc., via a designated onboarded scanning device) — not a separate feature.

*Verified against: [Onboard previous versions of Windows on Microsoft Defender for Endpoint](https://learn.microsoft.com/en-us/defender-endpoint/onboard-downlevel), [Deploy Microsoft Defender endpoint security to Windows devices using the Defender deployment tool](https://learn.microsoft.com/en-us/defender-endpoint/defender-deployment-tool-windows), [Manage automation file uploads](https://learn.microsoft.com/en-us/defender-endpoint/manage-automation-file-uploads), and [Manage automation folder exclusions](https://learn.microsoft.com/en-us/defender-endpoint/manage-automation-folder-exclusions) (Microsoft Learn).*

## Overview of Alerts and Incidents

### Alert severity
The four severity levels and their definitions match Microsoft's documentation closely:
- **High (red)**: behavior commonly associated with advanced persistent threats (APT) — credential theft tool activity, ransomware activity, tampering with security sensors, or any activity indicative of a human adversary.
- **Medium (orange)**: post-breach EDR behaviors that could be part of an APT — anomalous registry changes, execution of suspicious files, and similar attack-stage behaviors; these need investigation even though some may turn out to be internal security testing.
- **Low (yellow)**: alerts tied to prevalent malware — **hack-tools** (the transcript's "ACT tools" is almost certainly this term) and non-malware hacking utilities like reconnaissance commands or clearing logs — activity that typically doesn't indicate an advanced, organization-targeted threat.
- **Informational (grey)**: activity that isn't necessarily harmful to the network but is useful for driving organizational security awareness — e.g., a threat that Defender Antivirus prevented before any actual damage occurred.

Severity reflects the risk to the **organization**, not just absolute malware severity on one device — the same detected file might be "Informational" if fully prevented with no damage, or "Low"/"Medium"/"High" depending on potential organizational impact even if it was ultimately blocked.

Alert **categories** have been redefined to align with the **MITRE ATT&CK enterprise tactics** matrix (existing older alerts keep their legacy category names) — worth knowing this mapping exists beyond just generic labels like "malware" or "ransomware."

### Incidents
An incident stitches together the alerts, entities, and evidence that make up a single attack story, giving the full picture rather than isolated alerts. Managing an incident includes:
- **Classification**: defaults to "Not set," with options like "True positive" (with a specific threat type) or "False positive" (with a determination) to help the team track patterns over time.
- **Comments and history**: comments are added to a running history log showing prior changes to the incident.
- **Response actions and evidence**: related alerts, AIR investigations, and pending actions are automatically pulled into the incident, with evidence and response actions aggregated on a dedicated **Evidence and response** tab.
- **Alert linking**: alerts can be relinked — moved from one incident to another, or split out into a new incident — and resolving an incident resolves all of its linked, still-active alerts at once.

*Verified against: [View and organize the Microsoft Defender for Endpoint Alerts queue](https://learn.microsoft.com/en-us/defender-endpoint/alerts-queue) and [Manage incidents in Microsoft Defender](https://learn.microsoft.com/en-us/defender-xdr/manage-incidents) (Microsoft Learn).*

## Demo: Working with Alerts and Incidents

### Alerts queue
The queue can be exported, filtered by time range (1 Day, 3 Days, 1 Week, 30 Days, 6 Months), and customized by column; core columns include alert name, severity, investigation state (tied to Automated Investigation), status, category, detection source, impacted assets, and first/last activity timestamps — all matches confirmed.

Opening an alert shows the impacted asset and user (each pivotable to its own asset/user page), the **alert story** (the chain of events leading into and out of the alert — e.g., a process chain ending in a suspicious PowerShell script execution), and per-event **alert details** including available actions: open the alert, take management actions, view a timeline, **create a suppression rule** (useful for known-benign/buggy activity), **link the alert to another incident**, and request expert input (Microsoft's **Defender Experts** capability). Alert details also surface the relevant **MITRE ATT&CK tactic**, detection technology (e.g., machine learning), and generation/first-and-last-activity timestamps, plus a **Recommendations** view (including the option to add indicators to a block list) — all consistent with current documentation. From the impacted asset, response actions like running an antivirus scan, collecting an investigation package, and initiating live response are available directly.

### Incidents
An incident aggregates multiple related alerts into one attack story, which is why the incident list has far fewer entries than the alert list. The incident page's **attack story** includes an **incident graph** showing the full scope and spread of the attack over time — connecting suspicious entities (processes, files, users) to the assets they touched — with layout and node grouping/ungrouping controls, matching the demo's description.

Incident tabs are accurate: **Summary**, **Alerts**, **Devices**, **Mailboxes**, **Investigations** (shows automated investigations triggered by Defender for Identity and Defender for Endpoint, including any pending actions you can act on directly), and **Evidence and response** (all supported events/suspicious entities tied to the incident's alerts, filterable by type like Files or Processes, with per-entity actions such as hunting for related activity, adding a file to a block/allow rule, quarantining a file, adding a custom indicator, or downloading the file for further analysis — auto-remediated items are visible here too). One thing worth knowing: the **Mailboxes** tab only appears if your subscription includes **Defender for Office 365 Plan 2** — it's not a tab every tenant sees by default.

The **Summary** tab wraps up the incident's alerts, MITRE ATT&CK tactics involved, scope (who/what was affected), and evidence entity count — matching the demo's closing description.

*Verified against: [Investigate incidents in the Microsoft Defender portal](https://learn.microsoft.com/en-us/defender-xdr/investigate-incidents) and [Incidents and alerts in the Microsoft Defender portal](https://learn.microsoft.com/en-us/defender-xdr/incidents-overview) (Microsoft Learn).*

## Demo: Performing Endpoint Investigations

### Device response actions
Confirmed accurate: **Isolate device** disconnects the device from the network while specifically **retaining its connection to the Defender for Endpoint cloud service**, so the device stays isolated from peers but remains manageable/investigable — exactly as described. There's also a **selective isolation** mode (not mentioned in the demo) that lets you configure exclusions for specific processes, IPs, or services so certain tools/communications keep working while the device is otherwise isolated. One practical gotcha worth adding: a device fully behind a VPN tunnel may lose its connection to the Defender for Endpoint cloud service once isolated — Microsoft recommends split-tunneling VPN configurations for Defender-related traffic to avoid this.

**Restrict app execution** matches: it allows only files signed with a Microsoft-issued certificate to run, blocking everything else.

**Run antivirus scan** matches: choose **Quick scan** or **Full scan**, add a comment, then confirm. (Quick scan checks common system/kernel malware-startup locations like registry keys and startup folders; full scan runs a quick scan first, then a sequential scan of all fixed/removable drives, which can take hours to days depending on content volume.)

**Collect investigation package** matches: pulls a ZIP of point-in-time device data (running processes, autoruns, and similar artifacts) for download and offline analysis.

The rest of the action list matches current documentation: **Initiate Automated Investigation**, **Ask Defender Experts** (routes a request to Microsoft), adjusting **device value** (increase/decrease), **Exclude device**, and **Troubleshooting mode** — all real, current device-page actions (note: your subscription needs Defender for Endpoint **Plan 2** to see the full set of response actions).

### Live Response
Live response opens a remote shell-style console to the device, logs every command run (viewable via **Command index**/history), and supports commands like `getfile` (download a file from the device), `finddriver`, `persistence` (runs built-in scripts to surface persistence mechanisms — results require analyst judgment, since not everything surfaced is malicious), `processes` (lists running processes), `jobs`, and registry inspection, among others — all matching the demo's walkthrough. One detail worth adding: most live response commands have a **10-minute execution time limit**, but `getfile`, `findfile`, and `run` get an extended **30-minute** limit, since those often take longer to complete.

*Verified against: [Take response actions on a device in Microsoft Defender for Endpoint](https://learn.microsoft.com/en-us/defender-endpoint/respond-machine-alerts), [Isolation exclusions in Microsoft Defender for Endpoint](https://learn.microsoft.com/en-us/defender-endpoint/isolation-exclusions), [Run and customize on-demand scans in Microsoft Defender Antivirus](https://learn.microsoft.com/en-us/defender-endpoint/run-scan-microsoft-defender-antivirus), and [Investigate entities on devices using live response](https://learn.microsoft.com/en-us/defender-endpoint/live-response) (Microsoft Learn).*

## Threat Analytics

Threat analytics is Defender XDR's threat-intelligence solution, built and maintained by Microsoft security researchers. Reports track active threat actors and campaigns, explain popular and emerging attack techniques, assess critical vulnerabilities, and — matching the transcript's framing — tell you whether a given threat is currently active in your own environment, what the potential impact/exposure is, and what mitigations to prioritize.

The dashboard is organized into a few key views not called out in this brief intro: **Latest threats** (most recently published/updated reports, with active and resolved alert counts), **High-impact threats** (ranked by highest number of active/resolved alerts), and **Highest exposure threats** (ranked by your organization's exposure level). Each report typically maps the attack chain to **MITRE ATT&CK** tactics and techniques, lists an **Impacted assets** count (distinct assets with at least one active alert tied to that threat), and provides detailed recommendations plus threat-hunting guidance. Reports also include an analyst-written summary, a link to original sourcing, and automatically extracted, continuously updated **indicators of compromise (IOCs)** that feed both remediation and proactive hunting.

*Verified against: [Threat analytics in Microsoft Defender](https://learn.microsoft.com/en-us/defender-xdr/threat-analytics) and [Understand the analyst report section in threat analytics](https://learn.microsoft.com/en-us/defender-xdr/threat-analytics-analyst-reports) (Microsoft Learn).*

## Demo: Exploring Threat Analytics

Navigation: **Microsoft Defender portal → Threat analytics** (also reachable via a dashboard card showing top threats). The landing dashboard's legend, Latest/high-impact/highest-exposure groupings, and filtering (e.g., by affected assets) all match current documentation, as does the option to set up **email notifications** for report updates (**Settings → Microsoft Defender XDR → Email notifications → Threat analytics tab → Create a notification rule**, where you name the rule and choose which report types/tags to be notified about).

A report's tabs and sections match the demo closely:
- **Overview** — analyst-written summary, with a link into the full **Analyst report** tab (detailed attack-chain write-up, MITRE ATT&CK mapping, recommendations, and hunting guidance).
- **Related incidents** — incidents in your own tenant tied to this tracked threat; alerts within them get tagged to the specific threat/actor, and you can jump straight into investigating as covered in earlier demos.
- **Impacted assets** — shows affected devices (and mailboxes/email attempts where relevant) over time.
- **Endpoints exposure** — your organization's calculated exposure level (Low/Medium/High, based on the severity of exploited vulnerabilities/misconfigurations and how many devices have them), plus the deployment status of relevant security updates.
- **Recommended actions** (the demo refers to this as "Mitigation details" under an "Exposure & mitigations" view) — actionable, trackable mitigations, each linking through to the corresponding **Security recommendation** page (general information, remediation options, and the specific list of exposed devices needing that mitigation) — matching the demo's drill-down closely.

*Verified against: [Threat analytics in Microsoft Defender](https://learn.microsoft.com/en-us/defender-xdr/threat-analytics), [Understand the analyst report section in threat analytics](https://learn.microsoft.com/en-us/defender-xdr/threat-analytics-analyst-reports), and [Get email notifications for Threat analytics updates](https://learn.microsoft.com/en-us/defender-xdr/m365d-threat-analytics-notifications) (Microsoft Learn).*

## Demo: Threat and Vulnerability Management

This demo covers the same dashboard/Recommendations/Inventories/Weaknesses/Event timeline/Baseline assessment structure already fact-checked earlier under "Working with Vulnerability Management," and it holds up well here too. A few additions and clarifications from this walkthrough specifically:

- **Request remediation → "Open a ticket in Intune"** is a real capability, but it's an **optional checkbox scoped to Microsoft Entra ID–joined devices** rather than something that automatically happens for every remediation request — worth knowing since the demo describes it as if filing the ticket is the default outcome. Once a remediation request is submitted, a trackable security task appears on the Remediation page, and if the Intune checkbox was used, IT administrators still need to **review and approve (or reject) the ticket in Intune** before any patch/package deployment actually starts — it's not automatically actioned.
- **Inventories → Certificates**: flags expired certificates and weak signature algorithms — matches the demo.
- **Inventories → Hardware and firmware**: covers BIOS, processor, and system model info (including laptop/desktop/server model-level vulnerabilities) — matches the demo.
- **Inventories → Browser extensions**: includes risk level based on requested permissions, and Defender for Endpoint can identify extensions across multiple browser types, not just one — matches the demo.
- **Weaknesses**: the CVE list can be filtered to only those affecting your organization, or expanded to the full (much larger) global CVE catalog — matches the demo.
- **Security baseline assessment** profile creation flow matches what's already documented in these notes (Profiles → Create profile → name → benchmark/software selection → specific configuration settings → device group scope → Submit), including that benchmark options are OS/version-specific (e.g., separate benchmarks for Windows 10 vs. Windows Server). A benchmark like CIS for Windows 10 legitimately includes a group of BitLocker-related controls, consistent with the demo selecting "BitLocker" as part of building a profile.
- Not covered in earlier notes: the **Settings** view under baseline assessment lets you inspect every individual configuration setting that makes up a benchmark — what it technically checks (e.g., a specific registry key), remediation guidance for that setting, and which profiles/devices are currently in scope for that specific configuration ID.

*Verified against: [Remediate vulnerabilities with Microsoft Defender Vulnerability Management](https://learn.microsoft.com/en-us/defender-vulnerability-management/tvm-remediation) and [Use Intune to remediate vulnerabilities found by Microsoft Defender for Endpoint](https://learn.microsoft.com/en-us/intune/device-security/microsoft-defender/remediate-vulnerabilities) (Microsoft Learn).*

## Attack Surface Reduction Rules

Deployment methodology matches Microsoft's guidance closely: **plan** your deployment, **test** (enable rules in **Audit mode** first so you can see what would have been blocked, based on real business activity, without actually affecting users), then **enable** (switch to Block or Warn mode, typically rolled out **ring by ring** for large organizations) and **operationalize**. Standard-protection rules can generally go straight to Block/Warn with minimal testing; other rules should go through Audit mode first.

⚠️ One correction: the rule the transcript describes as blocking "abuse of exploited vulnerable signed **devices**" is actually **"Block abuse of exploited vulnerable signed drivers"** — it stops apps from writing a vulnerable signed *driver* to disk (it doesn't block already-present drivers from loading). Likely just a slip in the transcript, but worth using the correct term ("drivers," not "devices") for the exam.

The rest of the named rules are accurate, real ASR rules:
- **Polymorphic-threat-style protections**: "Block executable files from running unless they meet a prevalence, age, or trusted list criterion" (requires cloud-delivered protection, since the trusted list is cloud-maintained), blocking untrusted/unsigned processes run from USB, and advanced ransomware protection.
- **Productivity apps**: blocking Office apps (Word, Excel, PowerPoint, OneNote, Access) from creating executable content, from creating child processes at all, and from injecting code into other processes.
- **Signed driver abuse** (corrected above).
- **Lateral movement/credential theft**: blocking process creation from PSExec and WMI commands, and blocking credential theft from the Windows LSASS subsystem.
- **Email**: blocking execution of executable content from email clients and webmail, and blocking Office **communication** apps (Outlook, Skype, Teams) from creating child processes.
- **Scripts**: blocking obfuscated JavaScript/VBScript/PowerShell/macro code, and blocking JavaScript or VBScript from launching downloaded executable content.

*Verified against: [Attack surface reduction (ASR) rules reference](https://learn.microsoft.com/en-us/defender-endpoint/attack-surface-reduction-rules-reference) and [Plan your attack surface reduction (ASR) rules deployment](https://learn.microsoft.com/en-us/defender-endpoint/attack-surface-reduction-rules-deployment-plan) (Microsoft Learn).*

## Demo: Implementing ASR Rules

Deployment options (Group Policy, PowerShell, local policy/MDM, Microsoft Intune/Endpoint Manager) are all real, current methods — matches the demo.

**Setup steps — create an ASR policy in Intune:**
1. **Intune → Endpoint security → Attack surface reduction → Create Policy.**
2. Platform: **Windows 10 and later** (ASR rules apply to this platform).
3. Profile type: **Attack surface reduction rules** — the sibling profile types in this same category are **App and browser isolation**, **Device control**, **Exploit protection**, and **Application control** (the demo mentions Exploit Protection and Device Control as examples, which is accurate, though it doesn't name the other two).
4. Name the policy, then configure each rule under **Configuration settings**.
5. Modes per rule: **Not configured** (default — Windows default of Off), **Block** (enforced), **Audit** (Windows events logged, nothing actually blocked), and **Warn** (logs the event and shows the end user a message they can choose to bypass) — matches the demo's description exactly. Per-rule **exclusions** can also be configured here.
6. **Controlled folder access** is configured within this same policy: define a mode and specify which folders to protect (add individually or import via CSV) — untrusted apps are blocked from modifying files in those folders, which is a real, effective mitigation against ransomware, matching the demo's Documents-folder example.
7. **Next** → **Assignment**: scope to device groups (defaults to all devices), optionally filter by **scope tags**, and/or scope to specific user groups (with the ability to exclude specific groups too) — matches the demo's flow.
8. **Next** → **Create** to finish; the policy then syncs out to in-scope managed devices.

One addition worth knowing: **Warn mode isn't supported for three specific rules** — "Block JavaScript or VBScript from launching downloaded executable content," "Block persistence through WMI event subscription," and "Use advanced protection against ransomware" — those three only support Not configured/Block/Audit when configured through Intune.

*Verified against: [Manage attack surface reduction settings with Microsoft Intune](https://learn.microsoft.com/en-us/intune/intune-service/protect/endpoint-security-asr-policy), [Configure ASR rules and exclusions](https://learn.microsoft.com/en-us/defender-endpoint/attack-surface-reduction-rules-configure), and [Configure controlled folder access](https://learn.microsoft.com/en-us/defender-endpoint/enable-controlled-folders) (Microsoft Learn).*

## Conditional Access

⚠️ **Terminology update**: what the transcript calls "Azure Active Directory Identity Protection" is now **Microsoft Entra ID Protection** — Azure AD itself was renamed **Microsoft Entra ID** back in 2023. Expect Microsoft Learn, the exam, and the current portal to use "Entra ID" terminology throughout, not "Azure AD."

The signals model is confirmed accurate: **user and group membership**, **IP location information** (trusted/named locations, or entire countries/regions to allow or block), **device platform**, and **application** all serve as real Conditional Access conditions, and they can be combined — e.g., requiring MFA for a specific cloud app, from a specific device platform, from outside a trusted location, scoped to a specific user group.

**Real-time risk detection** is confirmed accurate: Microsoft Entra ID Protection analyzes signals during sign-in and calculates both a **sign-in risk** level and a **user risk** level, which then feed into Conditional Access as additional conditions a policy can key off of.

**Decisions (Grant controls)** are confirmed accurate: **Grant access** (optionally with conditions like requiring an approved client app or requiring multi-factor authentication) or **Block access** — block takes precedence if a matching policy is configured that way.

One addition not covered in the transcript: beyond grant controls, Conditional Access also supports **session controls** (applied only after all grant controls are satisfied) — such as app-enforced restrictions, routing through Microsoft Defender for Cloud Apps for additional monitoring, sign-in frequency, and persistent browser session settings — a distinct layer from the block/allow decision itself.

*Verified against: [Microsoft Entra Conditional Access: Zero Trust Policy Engine](https://learn.microsoft.com/en-us/entra/identity/conditional-access/overview) and [Microsoft Entra ID Protection risk-based access policies](https://learn.microsoft.com/en-us/entra/id-protection/concept-identity-protection-policies) (Microsoft Learn).*

## Azure Identity Protection

⚠️ Same terminology update as the previous section: this is now **Microsoft Entra ID Protection**, not "Azure Active Directory Identity Protection" (Azure AD was renamed Microsoft Entra ID in 2023).

### User risks (confirmed accurate)
- **Leaked credentials**: flagged when Microsoft's credential-scanning pipeline finds a confirmed match between a user's password and credentials found on the dark web/breach dumps/paste sites — this is described as a verified match, not a heuristic guess.
- **"Unusual behavior"**: the current, official name for this detection is **Microsoft Entra threat intelligence** — user activity that's unusual for that specific user or consistent with known attack patterns, based on Microsoft's internal/external threat intelligence research (including MSTIC data).

### Sign-in risks (confirmed accurate)
- **Unfamiliar sign-in properties**: triggered when a sign-in doesn't match properties (IP, ASN, location, device, browser, tenant IP subnet) the system has learned as normal for that user.
- **Atypical travel**: flagged when sign-ins occur from two locations where the travel time between them would be physically unrealistic.
- **Malware-linked IP address**: sign-in attempts from IP addresses/devices known to be infected or associated with malicious infrastructure (e.g., botnet servers).
- **Anonymous IP address**: sign-ins via anonymizing services like Tor or anonymous VPNs.

Not mentioned in the transcript: a newer sign-in risk detection, **Verified threat actor IP**, flags sign-in activity matching IP addresses specifically tied to known nation-state actors or cybercrime groups, based on Microsoft Threat Intelligence Center (MSTIC) data — worth knowing as a current addition to the risk detection list.

### Remediation workflows (confirmed accurate)
- **Self-remediation**: with risk-based Conditional Access policies configured, users can resolve their own risk automatically — e.g., completing an MFA prompt or a secure password change — without contacting the help desk.
- **Admin remediation**: used when self-remediation isn't configured, the risk level doesn't qualify for it, or urgency requires it — an administrator investigates and takes action directly (e.g., generating a temporary password, requiring a password change, or dismissing the risk).

Risk-based Conditional Access grant controls tie directly into this: **Require risk remediation** (Entra ID Protection manages the right remediation flow automatically, covering password-based and passwordless methods alike), **Require password change**, or **Block access**.

*Verified against: [What are risk detections?](https://learn.microsoft.com/en-us/entra/id-protection/concept-identity-protection-risks), [Risk detection types and levels](https://learn.microsoft.com/en-us/entra/id-protection/concept-risk-detection-types), and [Remediate risks and unblock users](https://learn.microsoft.com/en-us/entra/id-protection/howto-identity-protection-remediate-unblock) (Microsoft Learn).*

## Demo: Implementing Identity Protection Policies

🚨 **Major, urgent update — this directly affects what the demo teaches.** Microsoft has announced that the **legacy User Risk Policy and Sign-in Risk Policy configured directly in Identity Protection are retiring on October 1, 2026** — about **3 months from today**. These are exactly the two policy types this demo walks through configuring. Going forward, the same functionality (and more) should be configured as **risk-based Conditional Access policies** instead — using the "Sign-in risk" and "User risk" conditions in a normal Conditional Access policy, with grant controls, rather than the standalone Identity Protection policy blades shown here. Microsoft's stated benefits of migrating: managing all access policies in one place, report-only mode support, Graph API support, combining risk with other conditions (like location), sign-in frequency enforcement, and better sign-in log diagnostics. If you're studying this close to your exam date, expect the current exam content and any lab environment to already reflect the Conditional Access–based approach rather than this legacy flow.

The **MFA registration policy** is a separate, standalone Identity Protection policy and is **not** part of this October 2026 retirement — it remains current as described in the demo.

⚠️ Also worth a terminology/navigation update: this is configured in the **Microsoft Entra admin center**, under **Protection → Identity Protection** (the "Azure Active Directory" branding in the Azure portal is outdated — it's Microsoft Entra ID now, consistent with the correction noted in the previous section).

### What's still accurate about the demo's mechanics (as background/legacy concepts)
- **User risk policy**: scope via Assignments (all users, or specific users/groups, with exclusions), a user risk level threshold (e.g., Low and above), and Controls — **Block access**, or **Allow access** with a required secure password change. The modern equivalent Conditional Access grant controls are **Block access**, **Require password change**, and the newer **Require risk remediation** (which automatically picks the right remediation flow — password change or reauthentication — based on the user's authentication method and the specific risk detected).
- **Sign-in risk policy**: same assignment structure, a sign-in risk level threshold, and Controls — Block access, or Allow access with **required multifactor authentication**. The modern Conditional Access equivalent also supports **Require reauthentication (sign-in frequency)** as an additional control. Note: users must already have a registered MFA method capable of satisfying the policy before a sign-in risk policy can prompt them for it.
- **MFA registration policy**: scoped the same way (all/specific users, with exclusions); ensures new users register for MFA at first sign-in, since MFA is one of the key self-remediation methods for risk events.

Both risk-based Conditional Access policies require **Microsoft Entra ID P2** licensing.

*Verified against: [Microsoft Entra ID Protection risk-based access policies](https://learn.microsoft.com/en-us/entra/id-protection/concept-identity-protection-policies) (Microsoft Learn, updated April 2026).*

## Demo: Investigating Risky Users

The report set under Identity Protection is confirmed accurate: **Risky users** (flagged when a user has risky sign-ins and/or risk detections, showing risk state/level with actions to dismiss or confirm compromised), **Risky sign-ins** (lists sign-ins that are at risk, confirmed compromised, confirmed safe, dismissed, or remediated, with risk levels and triggered detection types), **Risk detections** (each individual detected risk event), and **Risky workload identities** — real reports for investigating application/service-principal identity risk rather than user risk, though this report (and the workload identity detections tab under Risk detections) specifically **requires Workload Identities Premium licensing**, which isn't mentioned in the demo.

Drilling into a user's recent risky sign-ins to spot **atypical travel** (two sign-ins from different locations within an implausibly short time window) matches the documented detection logic exactly. Investigating a sign-in's device info, risk info, MFA info, applied Conditional Access policies, and report-only Conditional Access impact are all real, current investigation details.

⚠️ One mechanism worth clarifying precisely: **confirming a user as "compromised" doesn't block their access by itself.** It sets the user's risk level to **High** and generates a new "Admin confirmed user compromised" detection — the actual block only happens because a **risk-based policy** (the User Risk Policy configured in the previous demo, or its Conditional Access equivalent) is already set to enforce Block access at that risk level. Without such a policy in place, an admin would need to take a separate manual action (like blocking sign-in or forcing a password reset) themselves. This is exactly why the demo's test (confirming compromised, then seeing the login attempt blocked) worked — it's the previously configured policy doing the blocking, not the confirmation action itself.

*Verified against: [Investigate risk with Microsoft Entra ID Protection](https://learn.microsoft.com/en-us/entra/id-protection/howto-identity-protection-investigate-risk), [ID Protection Risk Reports](https://learn.microsoft.com/en-us/entra/id-protection/concept-risk-reports), and [Remediate risks and unblock users](https://learn.microsoft.com/en-us/entra/id-protection/howto-identity-protection-remediate-unblock) (Microsoft Learn).*

## Identity Secure Score

The three-part value proposition is confirmed accurate: objectively measuring identity security posture, guiding planned improvements, and letting you track the success of those improvements over time.

⚠️ Current navigation: the **Microsoft Entra admin center** (entra.microsoft.com) → **Entra ID → Identity Secure Score** (or **Entra ID → Overview → Recommendations**, filtered to "Security") — not the classic Azure portal "Azure Active Directory" blade referenced in the demo.

⚠️ **Correction**: the score recalculates **every 24 hours**, not every 48 hours as the demo states — Microsoft evaluates your security configuration against best practices daily and issues a new score based on that evaluation.

Confirmed accurate:
- The score is shown as a percentage, with historical trending over time.
- Each improvement action lists **Score impact**, **User impact**, and **Implementation cost**, and opening one shows a description, what would change if implemented, expected user effect, and a **Get started** link into the relevant configuration area.
- Scoring can be **binary** (full credit for having a setting configured) or **partial/proportional** (e.g., MFA coverage scored as a percentage of protected users out of total users).
- Status options beyond "Planned" also include **To address**, **Risk accepted** (no points awarded, removes it from the active list), and **Resolved through third party**/**Resolved through alternate mitigation** (credited even though it wasn't done via a native Microsoft control).

⚠️ On required roles: current documentation states the **minimum** roles as **Service Support Administrator** to just view an improvement action, and **SharePoint Administrator** to update its status — narrower/different from the demo's broader "read-write vs. read-only" role groupings (Global Admin/Security Admin/Exchange/SharePoint Admin for write; Help Desk Admin/User Admin/Security Reader/Security Operator/Global Reader for read). A fuller table of least-privileged roles per task exists separately; the two minimums above are the ones Microsoft calls out directly on the main Identity Secure Score page.

Identity Secure Score is one of five categories (**Identity, Data, Devices, Infrastructure, Apps**) that roll up into the broader **Microsoft Secure Score** shown in the Defender portal — worth knowing that this identity-specific score is a slice of that larger picture, not a fully separate scoring system.

*Verified against: [What is the Identity Secure Score?](https://learn.microsoft.com/en-us/entra/identity/monitoring-health/concept-identity-secure-score) (Microsoft Learn).*

## Demo: Working with Defender for Cloud Apps

Confirmed accurate: Defender for Cloud Apps (formerly Microsoft Cloud App Security/MCAS) is now managed from the unified **Microsoft Defender portal** (security.microsoft.com) under **Cloud apps** — a legacy/classic Defender for Cloud Apps portal does still exist for some features and backward compatibility, but current guidance centers on the Defender portal, matching the demo's framing exactly.

### Cloud discovery
Confirmed accurate: the dashboard surfaces discovered apps, users, IP addresses, and devices, traffic volume, risk levels (high/medium/low), top app categories, top discovered apps, and where app data/headquarters are located. **Discovered apps** shows a risk score per app and lets you tag apps **Sanctioned** or **Unsanctioned**. **Discovered resources** applies specifically to log collector appliances uploading traffic data (empty if none are configured, as in the demo). **IP addresses**, **Users**, and **Devices** each let you pivot into a specific entity's history and app usage.

⚠️ Worth adding detail on the unsanctioned-app blocking mechanism the demo demonstrates: it requires the **Defender for Endpoint integration** to be enabled first (not automatic by default). Once enabled, marking an app **Unsanctioned** automatically creates a **Block** network indicator in Defender for Endpoint for that app's domains (apps in **Monitor** mode instead get a bypassable **Warn** indicator) — this is exactly the mechanism the demo describes with the GoDaddy example. One thing to flag: Microsoft's documented latency for this to take effect is **up to about 3 hours** (up to 1 hour to sync the sanction status to Defender for Endpoint, plus up to 2 hours to push the block policy to devices) — so in a real environment, don't expect the block to be instantaneous the way it appeared in the demo.

### Policies
Confirmed accurate: **Policy management** groups policies by category — **Threat detection**, **Information protection**, **Conditional access**, and (not explicitly named in the demo) **Compliance** — plus **Shadow IT** discovery-related policies. Creating a policy (e.g., an **Activity policy**) can start from a **template** (like the ransomware template shown, which does warn that it will overwrite any settings you've already configured on that policy type) or be built from scratch, and includes defining alerting and **governance actions**. **Policy templates** is a separate browsable list, filterable by category, with a quick "+"-style shortcut straight into policy creation using that template — all matching the demo.

*Verified against: [Microsoft Defender for Cloud Apps in the Microsoft Defender portal](https://learn.microsoft.com/en-us/defender-cloud-apps/microsoft-365-security-center-defender-cloud-apps), [Govern discovered apps using Microsoft Defender for Endpoint](https://learn.microsoft.com/en-us/defender-cloud-apps/mde-govern), and [Control cloud apps with policies](https://learn.microsoft.com/en-us/defender-cloud-apps/control-cloud-apps-with-policies) (Microsoft Learn).*

## Defender for Identity

The product history is accurate: **Advanced Threat Analytics (ATA)** → **Azure Advanced Threat Protection (Azure ATP)** → **Microsoft Defender for Identity**. ATA was a standalone on-premises product requiring dedicated hardware, while Defender for Identity is cloud-based. 🚨 Minor timing note: ATA's Extended Support ended in **January 2026** — about 6 months before today — so ATA is now fully out of support if anyone is still asking about it on the exam or in a legacy environment.

⚠️ Scope is broader than the transcript states: Defender for Identity now monitors signals from **both on-premises Active Directory and Microsoft Entra ID**, plus select other identity/IAM solutions (e.g., Okta) — not just on-premises AD alone.

The value proposition (behavioral analytics, attack timeline, reduced false-positive fatigue, shared Microsoft security telemetry, and feeding into the broader Microsoft security graph) is confirmed accurate.

### Kill chain categories (confirmed accurate)
Defender for Identity organizes its detections into four phases matching a typical attack kill chain: **Reconnaissance** (e.g., local/domain group enumeration, DNS enumeration), **Compromised Credentials** (e.g., brute-force attempts, suspicious VPN connectivity, honeytoken activity), **Lateral Movement** (e.g., Mimikatz-style credential-dumping behavior), and **Domain Dominance** (e.g., DCSync, DCShadow, Golden Ticket) — this maps directly to Microsoft's own alert categorization.

### Setup (confirmed accurate, with more precise steps)
**Setup steps — add a sensor:**
1. **Microsoft Defender portal → Settings → Identities → On-premises (Deployment) → Sensors tab → Add sensor.**
2. **Download installer** to save the installation package locally.
3. Copy the **Access key** shown and store it securely — it's a **one-time password used only for the sensor's initial registration**; after that, sensor communication uses certificate-based authentication and TLS. Microsoft recommends periodically regenerating this key (doing so doesn't affect already-deployed sensors).
4. Copy the installer to the target domain controller (or a dedicated server with the ability to query Active Directory) and run it, following the prompts — matches the demo's description.

### Alerts and incidents (confirmed accurate)
Defender for Identity alerts appear in the same Microsoft Defender portal alert experience as other Defender products, tagged with detection source **MDI**. Investigation follows the same pivot-based methodology already covered for Defender for Endpoint alerts, and Defender for Identity alerts can be correlated into broader, cross-product Defender XDR incidents alongside other Defender solutions.

### Lateral movement paths (confirmed accurate)
This feature visualizes how an attacker could pivot from a compromised low-privilege account, through nested group memberships and shared server access, to eventually reach a sensitive/domain admin account — helping defenders close off those paths before an attacker achieves domain dominance. The demo's example (a non-admin account with local admin rights on a server where a sensitive domain admin account is also logged in) is a textbook illustration of exactly this risk.

*Verified against: [Microsoft Defender for Identity Overview](https://learn.microsoft.com/en-us/defender-for-identity/what-is), [Understand and investigate Lateral Movement Paths](https://learn.microsoft.com/en-us/defender-for-identity/understand-lateral-movement-paths), and [Install the sensor](https://learn.microsoft.com/en-us/defender-for-identity/deploy/install-sensor) (Microsoft Learn).*

## Working with Cross Domain Incidents

This reinforces the incident structure already covered in earlier demos (Alerts, Devices, Users, Mailboxes, Investigations, Evidence and response tabs), applied here to a cross-domain example spanning email and endpoint.

One useful specific detail confirmed accurate: **email clusters** flagged as malicious (due to malware, high-confidence phishing, malicious files, or malicious URLs) generate a **pending action** — typically a **soft-delete** of the affected messages still sitting in cloud mailboxes (Inbox/Junk Email) — reviewable right on the **Evidence and response** tab, where you can **Approve** or **Reject** it (or jump to **Explorer** / **Go hunt** for more context first). This ties directly into **AIR (Automated Investigation and Response)** for Defender for Office 365, which is what actually generates these email-cluster remediation recommendations.

⚠️ Worth adding: not every email cluster action requires manual approval — AIR **fully automates** remediation for some cluster types (e.g., malicious URL and file clusters), auto-approving the pending action itself rather than waiting on an analyst. So the "approve the action" workflow the transcript walks through applies to clusters that specifically land in **Pending approval** status, not universally to every piece of email evidence.

*Verified against: [Review and manage remediation actions in AIR](https://learn.microsoft.com/en-us/defender-office-365/air-review-approve-pending-completed-actions) and [Investigate incidents in the Microsoft Defender portal](https://learn.microsoft.com/en-us/defender-xdr/investigate-incidents) (Microsoft Learn).*

## Advanced Hunting

Confirmed accurate across the board: advanced hunting is a query-based threat hunting tool built into Microsoft Defender XDR, built on **Kusto Query Language (KQL)**, and covers up to **30 days** of raw data by default (consistent with the data-retention figure already noted earlier in these notes). The two current modes match exactly: **Guided mode** — a visual, dropdown/building-block query builder that doesn't require knowing KQL or the underlying data schema, aimed at hunters of any experience level — and **Advanced mode** — a free-form query editor for those comfortable writing KQL from scratch.

One practical detail worth adding: guided-mode queries can be converted straight to KQL via an **"Edit in KQL"** option, which opens the equivalent query in advanced mode — a useful bridge for learning KQL by first building a query visually and then seeing its KQL equivalent.

*Verified against: [Advanced hunting overview in Microsoft Defender XDR](https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-overview) and [Choose between guided and advanced modes for hunting](https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-modes) (Microsoft Learn).*

## Demo: Working with Advanced Hunting

Navigation (**Hunting → Advanced hunting**) and the **Create new → "Query in builder"** (guided mode) vs. **"Query in editor"** (advanced mode) split are both confirmed exactly as described.

⚠️ Schema category naming has shifted slightly: the current official categories are **Apps and identities**, **Endpoints** (not "Devices"), **Email and collaboration**, **Cloud infrastructure**, and **Exposure management** (not "Threat & Vulnerability Management" — this reflects the same Microsoft Security Exposure Management rebrand already noted earlier in these notes under Vulnerability Management). "Cloud infrastructure" (Defender for Cloud data) is a category the demo doesn't mention at all.

Guided mode's features are confirmed accurate: a **domain** selector tied to the schema, loadable **sample queries** (like the Admin activities example), the ability to adjust sample size and date range, a **Getting started** section with task-oriented starter queries and inline help tips, an **Edit in KQL** option to convert to advanced mode, and the ability to turn a query directly into a **custom detection rule**.

Advanced mode is confirmed accurate: a bare query editor with no built-in guidance (intended for KQL-fluent users), with the same ability to save, set a query time range, and create a detection rule from the query.

The **Queries** area is confirmed accurate, with one small addition: alongside **Shared queries** (Microsoft-authored) and **Community queries** (sourced from Microsoft's public GitHub repository, reviewed before publishing, and organized into folders like Campaigns, Collection, and Defense evasion — matching MITRE ATT&CK-style groupings), there's also a **My queries** section for queries saved just for yourself, which the demo doesn't call out separately.

**"Go hunt"** from the Action Center (or from alerts/investigations generally) is confirmed accurate: it auto-generates a query scoped to the selected entity, defaulting to roughly a **1-hour window** centered on that entity's relevant timestamp — exactly why the demo needed to widen the time range to see meaningful results.

*Verified against: [Data tables in the Microsoft Defender XDR advanced hunting schema](https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-schema-tables), [Use shared queries in Microsoft Defender advanced hunting](https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-shared-queries), and [Get relevant info about an entity with go hunt](https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-go-hunt) (Microsoft Learn).*
