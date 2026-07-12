# SC-200 Tool & Keyword Identification Index

Scenario says X → it's testing Y. Trigger phrases in **bold**, gotchas marked ⚠️/🚨.

---

## Defender XDR / Cross-Product

| Tool | Trigger Keywords | Gotcha |
|---|---|---|
| **Defender XDR** | pre/post-breach, cross-domain correlation, unified incident queue, self-healing | Sentinel sits *above* XDR, not the same thing |
| **SOC tiers (escalation)** | Tier 1 triage/high-volume; Tier 2 investigation; Tier 3 hunt/hypothesis-driven | ⚠️ Different from geographic Tier 1/2/3 (local/regional/global RBAC scope) — same names |
| **Attack disruption** | blast radius, in-progress attack, contain device/user/IP, isolate device, ~99% confidence, reversible | AIR = investigate/remediate; disruption = contain |
| **AIR** | verdict (Malicious/Suspicious/No threat), Action Center approve/reject, automation levels, 7-day timeout | 🚨 Manual trigger retires Sept 1, 2026 |
| **Threat Analytics** | Latest/High-impact/Highest-exposure threats, analyst report, IOCs | Reports (curated) vs. Hunting (you query) |
| **Advanced Hunting** | KQL, Guided/Advanced mode, 30-day window, Go hunt (~1hr) | Endpoint-only tables = DeviceInfo, DeviceProcessEvents, etc. (10 total) |
| **Incidents/Alerts** | High/Med/Low/Informational, "hack-tools" (not ACT), Mailboxes tab | Mailboxes tab needs Defender for O365 Plan 2 |

## Defender for Endpoint

| Tool | Trigger Keywords | Gotcha |
|---|---|---|
| **Advanced features** | EDR block mode, tamper protection, "Save preferences" | Settings page, not per-device actions |
| **Custom Indicators** | file hash/IP/URL/cert, 15,000 cap, Allow>Warn>Block | Certs = Allow/Block-and-remediate only |
| **Web content filtering** | 5 categories, SmartScreen (Edge)/Network Protection (other), audit-only policy | Endpoint-level, works off-network |
| **Device Discovery** | Basic (passive)/Standard (active, ~50KB) scan | 🚨 Windows authenticated scan dead (Dec 2025) |
| **Network device scan** | SNMP v2/v3, up to 40 scanners | Different from Device Discovery |
| **Vulnerability Mgmt** | Exposure score, "Open ticket in Intune" (Entra-joined only) | Baseline assessment = CIS/STIG specifically |
| **RBAC (classic)** | Endpoints→Roles, permission categories | 🚨 New tenants (post Feb 2025) default to URBAC |
| **Device groups** | matching conditions, highest-rank wins, Ungrouped devices | Device joins ONE group even if multiple match |
| **Notification rules** | Alert (no toggle) vs. Vulnerability (toggleable) | Classic on/off distinction trap |
| **ASR rules** | audit→ring rollout, "signed drivers" (not devices) | Warn unsupported on 3 specific rules |
| **Response actions** | Isolate (keeps cloud connection), Restrict app execution | VPN split-tunnel needed or isolation breaks cloud link |
| **Live Response** | getfile/findfile/run = 30min; else 10min | — |
| **Onboarding** | local script (≤10), GPO, Intune, ConfigMgr, deployment tool | MMA dependency gone for old Windows (May 2026) |

## Defender for Cloud

| Tool | Trigger Keywords | Gotcha |
|---|---|---|
| **Azure Arc** | azcmagent, needed for Servers Plan 2 parity | 🚨 Not the retired MMA workspace-key flow |
| **Direct onboarding** | toggle + subscription, 24hr appear | Plan 1 full; Plan 2 still needs Arc |
| **AWS/GCP connectors** | Mgmt account/Single account (AWS); Org/Project (GCP); CIEM needs Security Admin | AWS not on Gov clouds |
| **Security alerts** | Take action tab, Sample alerts generator | Sample alerts validate real pipeline, not cosmetic |

## Microsoft Purview

| Tool | Trigger Keywords | Gotcha |
|---|---|---|
| **SITs** | primary/supporting element, confidence 65/75/85, proximity | Confidence ≠ severity |
| **Sensitivity labels** | Personal/Public/General/Confidential/Highly Confidential | Labels classify; DLP acts — don't conflate |
| **DLP** | locations, conditions+actions, Activity explorer | Inline web traffic (preview) = unmanaged AI/cloud apps |
| **Insider Risk Mgmt** | HR connector (CSV, not native SAP), Policies→Alerts→Triage→Investigate→Action | Escalate → eDiscovery (Premium), not generic "investigate more" |
| **Safe Attachments** | Off/Monitor/Block/Dynamic Delivery | 🚨 "Replace" is deprecated |
| **Safe Links** | mail-flow + time-of-click | Teams: click-check only, no URL rewrite |
| **Anti-phishing** | impersonation (off by default) vs spoof intel (on by default) | — |

## Entra ID Protection / Conditional Access

| Tool | Trigger Keywords | Gotcha |
|---|---|---|
| **CA signals** | user/group, location, device platform, app, risk | Session controls apply only after grant controls |
| **User/sign-in risk** | leaked creds, Entra threat intelligence, atypical travel, verified threat actor IP | — |
| **Legacy risk policies** | configured directly in Identity Protection | 🚨 Retiring Oct 1, 2026 → risk-based CA. MFA reg. policy unaffected |
| **CA grant controls** | Require risk remediation/password change, Block access | "Risk remediation" auto-picks the right flow |
| **Confirm compromised** | sets risk to High | Doesn't block by itself — needs a policy already in place |
| **Identity Secure Score** | 24hr recalc (not 48) | One of 5 categories under Microsoft Secure Score |

## Defender for Cloud Apps

| Tool | Trigger Keywords | Gotcha |
|---|---|---|
| **Cloud Discovery** | Sanctioned/Unsanctioned apps | Now in unified Defender portal |
| **Unsanctioned blocking** | needs Defender for Endpoint integration first | ~3hr latency, not instant |
| **Policies** | Threat detection/Info protection/Conditional access/Compliance/Shadow IT | Templates can overwrite existing settings |

## Defender for Identity

| Tool | Trigger Keywords | Gotcha |
|---|---|---|
| **History** | ATA→Azure ATP→Defender for Identity | 🚨 ATA support ended Jan 2026 |
| **Scope** | on-prem AD **and** Entra ID | Not AD-only |
| **Kill chain** | Recon/Compromised Creds/Lateral Movement/Domain Dominance | DCSync/DCShadow/Golden Ticket = Domain Dominance |
| **Sensor** | one-time Access key, then cert-based auth | — |

## Sentinel — Platform & Workspace

| Tool | Trigger Keywords | Gotcha |
|---|---|---|
| **Sentinel overview** | cloud-native SIEM, region = key setting | 🚨 Azure portal retires Mar 31, 2027 |
| **Workspace models** | Model 1 single; Model 2 regional; Model 3 Lighthouse multi-tenant | Same-tenant-only connectors force 1 workspace/tenant often |
| **Retention** | 90 days free, 2yr analytics/12yr archive max | Export beyond max = data export rules/ADX/`Invoke-AzOperationalInsightsQuery` (NOT "Continuous Export") |
| **Sentinel roles** | Reader/Responder/Contributor/Playbook Operator/Automation Contributor | 🚨 Data lake uses separate Entra-role RBAC |
| **Workspace Manager** | Direct-link/Co-management/N-Tier, 2,000 op cap | Playbooks NOT centrally publishable; preview, Azure-portal only |

## Sentinel — Data Ingestion

| Tool | Trigger Keywords | Gotcha |
|---|---|---|
| **Content Hub** | connector = whole packaged solution | 🚨 HTTP Data Collector API retires Sept 14, 2026 |
| **Syslog/CEF via AMA** | RFC 3164/5424, DCRs | Syslog→`Syslog` table; CEF→`CommonSecurityLog` |
| **Windows Events via AMA** | DCR, Collect tab (All/Common/Minimal/Custom) | Replaces legacy OMS agent entirely |
| **Threat intel connector** | STIX, upload API (Preview) | 🚨 Old TIP connector deprecated June 2026 |
| **Custom tables** | `_CL` suffix, 500 tables/500 cols cap | Legacy = omsadmin.sh/MMA; current = DCR-based |
| **Defender XDR connector** | Connect incidents/entities/events | Auto-enabled if Sentinel is on Defender portal |
| **Defender for Cloud sync** | bi-directional = closing Sentinel incident closes DfC alert | One-way sync automatic; bi-directional needs Contributor/Security Admin |

## Sentinel — Detection & Response

| Tool | Trigger Keywords | Gotcha |
|---|---|---|
| **Analytics rules** | Scheduled/NRT/Fusion/Microsoft security/ML Behavior/Threat Intel/Anomaly (7 types) | Fusion = cross-product; Microsoft security = single-product auto-incident |
| **Scheduled rule config** | run interval/lookback (5min–14d), 5-min ingestion delay, 24hr suppression | Widen lookback for source latency, don't confuse with interval |
| **Incident grouping** | up to 150 alerts/incident, overflow = new incident | Defender-portal Sentinel: XDR engine decides instead |
| **Playbook-on-rule (legacy)** | attaching playbook directly to analytics rule | 🚨 Deprecated Mar 2026 → use automation rules |
| **Investigate incidents** | New/Active/Closed, `Get/New-AzSentinelIncident` | Dashboard defaults to last 24hrs |
| **Multi-workspace view** | read+write on every workspace, up to 100 workspaces | Incidents only, not other Sentinel features |
| **UEBA** | baseline profiles, blast radius | Needs Security Admin + Owner/Contributor — NOT Global Admin |
| **Notebooks** | MSTICPy, MSTIC, external sources (VirusTotal), Azure ML workspace | Needs BOTH Sentinel roles AND AML roles |
| **Hunting queries** | KQL, entity mapping, promote to analytics rule | Same KQL as analytics rules |
| **Bookmarks** | preserve query+results+notes, escalate to incident | Livestream 🚨 deprecated Mar 2026 |
| **Workbooks** | My workbooks vs. gallery template (read-only until saved) | Native data only — Notebooks pull EXTERNAL data (key distinction) |
| **Playbooks (SOAR)** | Logic Apps, alert trigger vs. incident trigger (must match) | Logic App Contributor ≠ Operator ≠ Sentinel Playbook Operator ≠ Automation Contributor |

## Retirement Dates — Fast Lookup

| Date | Retiring |
|---|---|
| Aug 2024 | Legacy Log Analytics agent (MMA) → AMA only |
| Feb 2025 | URBAC default for new Defender tenants |
| Dec 2025 | Windows authenticated scan (Device Discovery) |
| Jan 2026 | ATA Extended Support |
| Apr 2026 | SC-200 outline restructured (3 domains: 40-45/35-40/20-25%) |
| Mar 2026 | Playbook-on-analytics-rule; Sentinel Livestream |
| May 2026 | MMA dependency removed (old Windows agents) |
| Jun 2026 | Legacy TIP data connector |
| Sept 2026 | AIR manual trigger (Sept 1); HTTP Data Collector API (Sept 14) |
| Oct 2026 | Legacy User/Sign-in Risk Policy |
| Mar 2027 | Sentinel in Azure portal |

---
*From `sc-200-notes.md`. Verify time-sensitive items against current Learn docs near exam date.*
