# Azure Security Engineer AZ-500 Notes

**Security principle:** Secure your identity and authentication system as a high priority in your organization's cloud security practice. Common security controls include:

- Restrict privileged roles and accounts
- Require strong authentication for all privileged access
- Monitor and audit high-risk activities

**Security principle:** Use managed application identities instead of creating human accounts for applications to access resources and execute code. Managed application identities provide benefits such as reducing the exposure of credentials. Automate the rotation of credentials to ensure the security of the identities.
For services that don't support managed identities, use Microsoft Entra ID to create a service principal with restricted permissions at the resource level. It is recommended to configure service principals with certificate credentials and fall back to client secrets for authentication.

**Security principle:** Authenticate remote servers and services from your client side to ensure you are connecting to trusted server and services. The most common server authentication protocol is Transport Layer Security (TLS), where the client-side (often a browser or client device) verifies the server by verifying the server’s certificate was issued by a trusted certificate authority.

**Security principle:** Use single sign-on (SSO) to simplify the user experience for authenticating to resources including applications and data across cloud services and on-premises environments.

**Security principle:** Enforce strong authentication controls (strong passwordless authentication or multi-factor authentication) with your centralized identity and authentication management system for all access to resources. Authentication based on password credentials alone is considered legacy, as it is insecure and does not stand up to popular attack methods.

When deploying strong authentication, configure administrators and privileged users first, to ensure the highest level of the strong authentication method, quickly followed by rolling out the appropriate strong authentication policy to all users.

**Security principle:** Explicitly validate trusted signals to allow or deny user access to resources, as part of a zero-trust access model. Signals to validate should include strong authentication of user account, behavioral analytics of user account, device trustworthiness, user or group membership, locations and so on.

##Group Types

- **Security:** Used to manage user and computer access to shared resources.
Members of a security group can include users, devices, service principals, and other groups (also known as nested groups), which define access policy and permissions. Owners of a security group can include users and service principals.

**Note**
When nesting an existing security group to another security group, only members in the parent group will have access to shared resources and applications. Nested group members don't have the same assigned membership as the parent group members.

**Microsoft 365:** Provides collaboration opportunities by giving group members access to a shared mailbox, calendar, files, SharePoint sites, and more.
This option also lets you give people outside of your organization access to the group. Members of a Microsoft 365 group can only include users. Owners of a Microsoft 365 group can include users and service principals.



##Implement Entra ID protection
- Automate the detection and remediation of identity-based risks.

- Investigate risks using data in the portal.

- Export risk detection data to third-party utilities for further analysis


- Entra connect is a way to sync on-prem and cloud identities



**Authentication options:**

- Password Hash sync  ==> this makes sure that password from the cloud is synced with the on-prem

- Pass-through   ==> this does not sync passwords between cloud and on-prem.

- Federated Authentication  ==> Requires an external system for authentication (like an authentication server)


**Security principle:** Protect the data in transit against 'out of band' attacks (such as traffic capture) using encryption to ensure that attackers can't easily read or modify the data. Set the network boundary and service scope where data in transit encryption is mandatory inside and outside of the network. While this is optional for traffic on private networks, this is critical for traffic on external and public networks.

**Security principle:** Ensure that your virtual network deployment aligns to your enterprise segmentation strategy defined in the GS-2 security control. Any workload that could incur higher risk for the organization should be in isolated virtual networks.

- You can create custom, or user-defined (static), routes in Azure to override Azure's default system routes, or to add more routes to a subnet's route table. In Azure, you create a route table, then associate the route table to zero or more virtual network subnets. Each subnet can have zero or one route table associated to it. If there are conflicting route assignments, user-defined routes override the default routes. You are limited to 1,000 routes per route table.

**You can specify the following next hop types when creating a user-defined route:**

- **Virtual Appliance:** A virtual appliance is a virtual machine that typically runs a network application, such as a firewall. When using a Virtual appliance your next hop can be a Private Ip address of a network interface attached to a virtual machine OR the private ip address of an Azure internal load balancer.

- **Virtual network gateway:** Specify when you want traffic destined for specific address prefixes routed to a virtual network gateway. The virtual network gateway must be created with type VPN. 

- **None:** This is used when you want to drop the traffic or have not fully configured UDR.

- **Virtual network:** Specify the Virtual network option when you want to override the default routing within a virtual network.

- Internet: Specify the Internet option when you want to explicitly route traffic destined to an address prefix to the Internet, or if you want traffic destined for Azure services with public IP addresses kept within the Azure backbone network.



**Service Endpoint** gives access from subnets in a VNET to a range of multiple services e.g. A range of Storage Accounts. The downside of this option is, the resources from the subnet can access all the Storage accounts within this scope, which can increase blast radius in case of a security breach. The logic behind this is that the service endpoint creates a network card that links to the Storage service in the selected subnet and gives it a private ip within the subnet.


**Private Endpoint** gives access from subnet of a VNET to a specific service e.g. One specific Storage Account and the specific storage type like blob or file share. This is more preferred from a security POV. The logic behind this is that the service endpoint creates a network card that links to the Storage service in the selected subnet and gives it a private ip within the subnet.

- A virtual hub is a Microsoft-managed virtual network that serves as a central point for connecting other virtual networks and on-premises networks.

- Azure Firewall Manager allows centralized management and configuration of multiple Azure Firewall instances across different regions, enabling the creation and management of secured virtual hubs.

- On Microsoft Defender External Attack Surface Management (Defender EASM) Dashboard the “Approved Inventory” assets are always represented in dashboard charts and are scanned daily to ensure data recency.

- In Azure Blob storage, while container names are utilized in access conditions, blob index tags offer a more customizable attribute-based approach when configuring ABAC.

- When creating a new resource for VNET Integration, it requires an unused subnet that's an IPv4 /26 block or larger. Also, the subnet can't have service endpoint policies enabled.

- Firewall can be deployed to must be named AzureFirewallSubnet. Hence we can only use AzureFirewallSubnet and not any other

- If service endpoint is enabled from a vm to access a resource (e.g VM to db) Then even if the VM has a public ip address allowed it must still use the service endpoint.

- Microsoft Defender for Cloud limits the count of alert notifications that is sent out. 4 high severity alerts are allowed while 2 and 1 medium and low severity alerts respectively are allowed per day.

- Microsoft Defender for Cloud only allows assignment of Initiatives (which is a collection of policies) to your subscriptions.

- Service endpoints are service-type specific and not resource-specific (use Private endpoints for specific resource).

- Standard tier of Azure firewall supports the below list of features -

  - Built-in high availability

  - Availability Zones

  - Unrestricted cloud scalability

  - Application FQDN filtering rules

  - Network traffic filtering rules

  - FQDN tags Service tags

  - Threat intelligence

  - DNS proxy

  - Custom DNS FQDN in network rules

  - Deployment without public IP address in Forced Tunnel Mode

  - Outbound SNAT support

  - Inbound DNAT support

  - Multiple public IP addresses

  - Azure Monitor logging

  - Forced tunneling

  - Web categories

  - Certifications
 
- In Azure AD, Synchronization Rules Editor allows you to define and manage synchronization rules for Azure AD Connect. By creating an attribute-based filtering rule, you can exclude users based on specific attributes,

- JIT feature needs an NSG to be attached to a VM before it enables the temporary access. JIT adds a temporary rule in NSG to enable the access. 

- Cloud Application Administrator role to Admin1, because this role grants the ability to manage enterprise applications, including assigning users and groups to apps like App1. 

- Using certificate credentials for Microsoft Entra app registration ensures that the credentials are not transmitted during authentication, that they are stored securely, and that the credential usage follows the principle of least privilege.

- To run workloads that are compliant with GDPR, custom initiatives should be to be created. GDPR compliance initiatives are not yet available in Azure.

- Defender EASM requires at least one discovery group with a seed asset (such as a domain, ASN, or IP block) before it can begin external asset discovery.

- Periodic recurring scans on Microsoft Defender for SQL vulnerability assessment is triggered once a week. This value cannot be changed and is set by Microsoft.

- Azure Arc-enabled Kubernetes is the only configuration that includes Kubernetes and can be deployed to AWS.

- You can deploy an SQL managed instance to a dedicated virtual network subnet that does not have any resource connected. The subnet can have a service endpoint or can be delegated for a different service. You cannot deploy an SQL managed instance to a subnet with a connected virtual machine.

- For App Service Only the Isolated pricing plan (tier) can be deployed to a virtual network subnet. With other pricing plans, inbound traffic is always routed to the public IP address of the web app, while web app outbound traffic can reach the endpoints on a virtual network.

- You can deploy an SQL managed instance to a dedicated virtual network subnet that does not have any resource connected. The subnet can have a service endpoint or can be delegated for a different service.

- Azure DDoS Network Protection offers comprehensive DDoS protection for Azure resources, including those with Basic SKU public IPs. It includes the capabilities to protect against DDoS attacks, regardless of the public IP SKU, by applying the protection at the network level.

**Azure Front Door**
**Role:** Global Layer 7 Load Balancer & CDN. Scope: Global (Not deployed into a VNet; it sits at the "Edge" of Microsoft's network).

**Key Features for AZ-500**
- **Global Load Balancing:** Routes traffic to the closest backend (region) using Anycast protocol.

- **WAF at the Edge:** Blocks attacks before they reach your VNet.

- **Split TCP:** Optimizes connection speed by terminating the TCP connection at the nearest edge POP.

- **Front Door Premium (Exam Critical):**

  - **Private Link:** Support for connecting privately to your backend origins (App Services, Storage) avoiding public internet exposure.

  - **Managed Bot Protection:** Advanced bot rules.

- **WAF Policy:** Global WAF policies can be associated with Front Door.

**Azure Application Gateway**<br/>
**Role:** Regional Layer 7 (HTTP/HTTPS) Load Balancer & WAF. Scope: Regional (deployed into a Virtual Network subnet).

**Key Features for AZ-500**
- **Web Application Firewall (WAF):** Protects against OWASP Top 10 attacks (SQL Injection, XSS).

  - **Tiers:** WAF_v2 is the standard for exams (supports auto-scaling and zone redundancy).

  - **Modes:** Detection (logs only) vs. Prevention (blocks attacks).

  - **Rules:** Uses Core Rule Sets (CRS) 3.x. You can create Custom Rules to block specific IPs or Geographies.

- **Encryption:** Supports End-to-End TLS (encrypts traffic from gateway to backend) and SSL Termination (offloads encryption at the gateway).

- **Networking:**

  - Requires a dedicated subnet.

  - **NSGs:** If you use an NSG on the App Gateway subnet, you must allow traffic on ports 65200-65535 for Azure infrastructure communication (Gateway Manager service tag).

**Exam Tips & "Gotchas"**
**Scenario:** "You need to protect a web app hosted in East US from SQL injection." -> **Answer:** Application Gateway with WAF.

**Scenario:** "You need session affinity (sticky sessions) based on cookies." -> **Answer:** Application Gateway (Cookie-based affinity).

**Connection Draining:** Know that this feature allows you to gracefully remove backend pool members (updates/maintenance) without dropping active connections.


### Azure Firewall
**Role:** Managed, Cloud-based Network Security Service (Layer 3-7).
**Scope:** Regional (often placed in a Hub VNet).

#### Key Features for AZ-500
* **Standard vs. Premium (HIGH YIELD):**
    * **Standard:** L3-L7 filtering, Threat Intelligence (Alert/Deny known malicious IPs), FQDN Tags.
    * **Premium:** Adds **TLS Inspection** (decrypts HTTPS to inspect packet), **IDPS** (Intrusion Detection/Prevention System), and **URL Filtering** (can block `google.com/news` specifically, whereas Standard only sees `google.com`).
* **Rule Processing Order (Must Memorize):**
    1. **DNAT Rules** (Inbound traffic translation)
    2. **Network Rules** (IPs, Ports, Protocols - L3/L4)
    3. **Application Rules** (FQDNs, HTTP/S - L7)
    * *Note: If a Network rule allows traffic, Application rules are NOT processed.*
* **Management:** Controlled via **Azure Firewall Policy** (preferred for exams) or Classic Rules.

* **Deployment:** When deploying an Azure Firewall with the Firewall Management NIC enabled, the management interface must be placed in a dedicated subnet named AzureFirewallManagementSubnet. This subnet must follow a strict naming convention and meet Azure’s requirements. According to the latest Azure guidance, the minimum subnet size for 'AzureFirewallManagementSubnet' is /26,

#### Exam Tips & "Gotchas"
* **Scenario:** "You need to prevent internal VMs from accessing malicious websites and inspect the content of encrypted HTTPS traffic." -> **Answer:** Azure Firewall **Premium** (because of TLS Inspection).
* **Scenario:** "You need to allow Windows Update traffic for all VMs." -> **Answer:** Use an **Application Rule** with the `WindowsUpdate` FQDN Tag.
* **DNS Proxy:** Azure Firewall can act as a DNS proxy to forward DNS queries from VMs to a custom DNS server or Azure DNS.

---

### Comparison Summary Table

| Feature | **Application Gateway** | **Azure Front Door** | **Azure Firewall** |
| :--- | :--- | :--- | :--- |
| **OSI Layer** | Layer 7 (Web) | Layer 7 (Web) | Layer 3-7 (Network + App) |
| **Scope** | Regional | Global | Regional |
| **Primary Use** | Regional Web Apps, WAF | Global Web Apps, CDN | Network Security, Outbound Control |
| **WAF Support** | Yes (Regional) | Yes (Global) | No (Uses IDPS/Threat Intel instead) |
| **Backend Connectivity** | Direct IP / VNet Integration | Public IP or Private Link (Premium) | VNet Peering (Hub & Spoke) |
| **TLS/SSL** | Termination & End-to-End | Termination (Edge) | **Inspection** (Forward Proxy - Premium) |


## Microsoft Defender for Cloud (MDC)
**Role:** Cloud Native Application Protection Platform (CNAPP). Focuses on **CSPM** (Posture Management) and **CWP** (Workload Protection).

### 1. Implement and Manage Cloud Governance
MDC relies heavily on **Azure Policy** to drive governance.

* **Microsoft Cloud Security Benchmark (MCSB):** The default policy initiative assigned to every subscription when you enable MDC. It replaces the old "Azure Security Benchmark."
* **Regulatory Compliance Dashboard:**
    * Used to audit against standards like **NIST 800-53, PCI DSS, ISO 27001, and SOC 2**.
    * **Action:** You add these standards as "Compliance Policies" in the Environment Settings.
    * **Exam Logic:** If a question asks how to generate a report for an auditor proving PCI compliance, the answer is the **Regulatory Compliance Dashboard** in MDC.
* **Exemptions:** If a recommendation is not applicable (e.g., "MFA should be enabled" on a break-glass account), you create an **Exemption** (either for a specific resource or a subscription) to prevent it from lowering your Secure Score.

### 2. Manage Security Posture (CSPM)
**Primary Metric:** **Secure Score**.

* **Secure Score:** A percentage measurement of your security stance.
    * **Remediation:** Recommendations are grouped into "Controls." You must fix *all* recommendations in a Control to get the points (or use "Quick Fix").
    * **Quick Fix:** A button available for certain recommendations (usually configuration changes) that automatically applies the fix (e.g., "Secure transfer required" for Storage Accounts).
* **Inventory:** Provides a view of security state across all resources (Azure, AWS, GCP, and on-prem connected via Azure Arc).

### 3. Configure Threat Protection (CWP)
This requires enabling paid plans (Defender plans).

* **Just-in-Time (JIT) VM Access:**
    * **Goal:** Reduces attack surface by closing management ports (RDP 3389, SSH 22) until valid access is requested.
    * **Workflow:** User requests access -> Approval (optional) -> NSG rule creates a temporary "Allow" rule for the specific source IP -> Rule is removed after time window expires.
    * **Requirement:** Requires **Defender for Servers Plan 2**.
* **Adaptive Application Controls:**
    * **Goal:**Prevents malware by defining a strict "Allow List" of applications that can run on a VM.
    * **Mechanism:** Uses Machine Learning to analyze behavior and recommend legitimate apps to allow.
* **File Integrity Monitoring (FIM):**
    * **Goal:** Monitors Windows Registry and critical OS files for changes (a sign of compromise).
    * **Requirement:** Requires the Azure Monitor Agent (AMA) or Log Analytics Agent.
* **Defender for specific resources:**
    * **Defender for SQL:** Scans for vulnerabilities (SQL Injection, unusual locations).
    * **Defender for Storage:** Detects unusual uploads/downloads or potential malware hosting.
    * **Defender for Key Vault:** Detects unusual access patterns or Tor exit node access.
 
* Microsoft Defender for Cloud limits the count of alert notifications that is sent out. 4 high severity alerts are allowed while 2 medium and 1 low severity alerts respectively are allowed per day.

* The OWASP (Open Web Application Security Project) Top 10 dashboard within Defender EASM specifically focuses on the top ten most critical web application security risks. It provides insights and assessments related to common vulnerabilities that impact web applications. Utilizing this dashboard allows you to identify vulnerabilities in your inventory assets that align with these critical risks.

---

## Microsoft Sentinel
**Role:** Cloud-native **SIEM** (Security Information and Event Management) and **SOAR** (Security Orchestration, Automation, and Response).

### 1. Data Connectors (Getting Data In)
Sentinel is useless without data. You must configure **Data Connectors** first.
* **Service-to-Service:** Connects Azure Activity, Entra ID (AAD), Defender for Cloud, Office 365. (Usually free ingestion).
* **Syslog/CEF:** For Linux machines and network appliances (firewalls) using a log forwarder.
* **Threat Intelligence:** Connects TAXII servers to import external threat indicators.

### 2. Analytics Rules (Detecting Threats)
Rules query the data to generate **Alerts** and **Incidents**.
* **KQL (Kusto Query Language):** The language used to write rules.
* **Rule Types:**
    * **Microsoft Security (Incident creation):** Automatically creates Sentinel incidents from MDC or Entra Identity Protection alerts.
    * **Fusion:** Uses ML to correlate low-fidelity signals into high-fidelity incidents (enabled by default).
    * **Scheduled:** Standard custom query rules (e.g., "Detect 5 failed logins in 10 minutes").
    * **NRT (Near Real-Time):** Runs every minute for fast detection.

### 3. Automation (SOAR)
Both MDC and Sentinel use **Azure Logic Apps** for automation, but the terminology differs slightly.

| Feature | Context | Triggered By | Use Case |
| :--- | :--- | :--- | :--- |
| **Workflow Automation** | Defender for Cloud | **Alerts** or **Recommendations** | Auto-remediate a config issue (e.g., enable logs) or email the CISO when a high-severity alert fires. |
| **Playbooks** | Microsoft Sentinel | **Incidents** or **Alerts** | Block an IP in Azure Firewall, reset a user's password in Entra ID, post a message to Teams. |

* **Automation Rules (Sentinel):** Used to manage Playbooks. You create a rule (e.g., "If Incident Severity = High") to automatically run a Playbook.
* **Notebooks:** You can use ML to run note books on Sentinel to automate processes


---

## Exam "Gotchas" & Comparison

* **MDC vs. Sentinel:**
    * If the question asks about **Compliance**, **Secure Score**, or **Protecting Workloads** (Servers/SQL), the answer is **Defender for Cloud**.
    * If the question asks about **Aggregating Logs** from multiple clouds/on-prem, **Correlating events**, **Hunting**, or **Investigating** a breach timeline, the answer is **Microsoft Sentinel**.
* **Permissions:**
    * To run a Playbook/Logic App, the Logic App must have a **Managed Identity** with permissions to perform the action (e.g., "User Access Administrator" to reset passwords).
    * **Sentinel Responder** role: Can manage incidents (assign, close) but cannot query logs.
    * **Sentinel Contributor** role: Can create analytics rules and automation.
* **KQL Keyword:** Watch for `SecurityEvent` (Windows logs), `SigninLogs` (Entra ID), and `Heartbeat` (Agent health).

## References

- [Azure subscriptions quota and service limits](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/azure-subscription-service-limits?toc=%2Fazure%2Fvirtual-network%2Ftoc.json#networking-limits)

- [Routing Examples for UDR's](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview#routing-example)

- [Virtual Network Integration & Limitations](https://learn.microsoft.com/en-us/azure/app-service/overview-vnet-integration#limitations)

- [Azure Firewall](https://learn.microsoft.com/en-us/azure/firewall/features-by-sku)

- [Azure Key Vault HSM](https://learn.microsoft.com/azure/key-vault/managed-hsm/overview)







