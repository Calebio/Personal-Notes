# Azure Security Engineer (AZ-500) Notes

---

# 1. Manage Identity and Access

## Identity Security Principles
- Secure identity and authentication systems as a high priority.
- Common controls:
  - Restrict privileged roles and accounts
  - Require strong authentication
  - Monitor and audit high-risk activities

---

## Strong Authentication

Password-only authentication is considered **legacy and insecure**.

Recommended authentication methods:

- Multi-Factor Authentication (MFA)
- Passwordless authentication

Deployment strategy:
1. Administrators
2. Privileged users
3. All users

---

## Single Sign-On (SSO)

Allows users to authenticate once and access multiple services.

Benefits:
- Better user experience
- Reduced credential management
- Works across:
  - Cloud services
  - On-prem systems

---

## Zero Trust Model

Access decisions must validate multiple signals:

- Strong authentication
- User behavior analytics
- Device compliance
- Location
- Group membership

---

## Managed Identities

Best practice: **Use managed identities instead of storing credentials.**

Benefits:

- Eliminates credential storage
- Automatic credential rotation
- Built-in Azure integration

Fallback option:

Use **Microsoft Entra service principals**

Recommended authentication:

- Certificate credentials
- Client secrets (less secure)

---

## Group Types

### Security Groups

Purpose: Access management.

Members:
- Users
- Devices
- Service principals
- Other groups

Owners:
- Users
- Service principals

Important:

Nested group members **do not inherit the parent group assignment automatically.**

---

### Microsoft 365 Groups

Purpose: Collaboration.

Provides:

- Shared mailbox
- Calendar
- SharePoint
- File storage

Members:
- Users only

External users supported.

---

## Entra ID Protection

Provides:

- Identity risk detection
- Automated remediation
- Risk investigation tools
- Export risk data to external tools

---

## Hybrid Identity (Azure AD Connect)

Synchronizes on-prem AD with Entra ID.

### Authentication Methods

**Password Hash Sync**
- Password hashes stored in cloud.

**Pass-through Authentication**
- Password validated on-prem.

**Federated Authentication**
- Uses external identity provider (e.g., ADFS).

---

## Entra Roles Example

### Cloud Application Administrator

Permissions:

- Manage enterprise applications
- Assign users/groups to applications

# 2. Implement Platform Protection

---

## Network Segmentation

Virtual networks should align with enterprise segmentation strategy.

High-risk workloads should be placed in isolated virtual networks.

---

## Network Security Groups (NSG)

Rules can be applied at:

- Subnet level
- Network Interface level

Important rule:

DENY rules always override ALLOW rules.

Example:

Subnet rule:
DENY port 80

NIC rule:
ALLOW port 80

Result:
Traffic is blocked.

---

## User Defined Routes (UDR)

Used to override Azure default routes.

Limits:

- Maximum 1000 routes per route table
- Each subnet can have only one route table

Next Hop Types:

- Virtual Appliance
- Virtual Network Gateway
- Virtual Network
- Internet
- None (drop traffic)

---

## Service Endpoints

Allow subnets to access Azure services directly.

Characteristics:

- Service-type specific
- Not resource specific

Example:
Access to all storage accounts within the service scope.

---

## Private Endpoints

Provide private connectivity to specific resources.

Example:

Access to one storage account's blob service.

Benefits:

- Reduced attack surface
- Resource-specific access

Preferred for high-security workloads.

---

## Just-in-Time VM Access (JIT)

Reduces attack surface by closing management ports.

Workflow:

1. Ports closed
2. Access request submitted
3. Temporary NSG rule created
4. Rule removed after time expires

Requirements:

- NSG attached to VM
- Defender for Servers Plan 2

---

## Azure DDoS Protection

Azure DDoS Network Protection protects resources with public IPs.

Features:

- Protects Basic SKU public IPs
- Protection applied at network level

# 3. Secure Data and Applications

---

## Data Protection in Transit

Use encryption to protect data from interception.

Primary protocol:

Transport Layer Security (TLS)

Client verifies:

- Server certificate
- Trusted certificate authority

---

## Azure Key Vault (General Concept)

Stores and manages:

- Secrets
- Keys
- Certificates

Security benefits:

- Centralized secret management
- RBAC or access policy authorization
- Managed identity integration

---

## Blob Storage Security

Access control options:

- Container-based access conditions
- Blob Index Tags (attribute-based access control)

Blob index tags allow more flexible policies.

---

## SQL Managed Instance Networking

Requirements:

- Dedicated subnet
- Subnet must not contain other resources (like VMs)

Allowed configurations:

- Service endpoints
- Delegated subnet

---

## App Service Networking

Only **Isolated pricing tier** can be deployed directly inside a VNet subnet.

Other tiers:

- Inbound traffic uses public endpoint
- Outbound traffic can access VNets

---

## Application Security Components

### Azure Front Door

Role: Global Layer 7 load balancer and CDN.

Features:

- Global routing
- WAF at edge
- Private Link (Premium)

---

### Azure Application Gateway

Role: Regional Layer 7 load balancer.

Features:

- Web Application Firewall (WAF)
- Protection against OWASP Top 10
- SSL termination
- End-to-end TLS

Requires:

Dedicated subnet.

---

### Azure Firewall

Managed Layer 3–7 network firewall.

Standard features:

- Network filtering
- Application rules
- Threat intelligence

Premium features:

- TLS inspection
- Intrusion Detection and Prevention (IDPS)
- URL filtering

Rule processing order:

1. DNAT rules
2. Network rules
3. Application rules

# 4. Manage Security Operations

---

# Microsoft Defender for Cloud

Cloud Native Application Protection Platform (CNAPP).

Combines:

- Cloud Security Posture Management (CSPM)
- Cloud Workload Protection (CWP)

---

## Governance

Defender for Cloud relies on Azure Policy.

Default initiative:

Microsoft Cloud Security Benchmark (MCSB)

---

## Secure Score

Measures overall security posture.

Score improves when recommendations are remediated.

---

## Regulatory Compliance Dashboard

Provides compliance reporting for frameworks such as:

- NIST
- PCI DSS
- ISO 27001
- SOC 2

Used for audit reporting.

---

## Defender Plans

Examples include:

- Defender for Servers
- Defender for SQL
- Defender for Storage
- Defender for Key Vault

---

## Vulnerability Assessment

SQL vulnerability scans run automatically:

Once per week.

Frequency cannot be changed.

---

## Microsoft Defender External Attack Surface Management

Requirements:

- At least one discovery group
- Must include a seed asset

Examples:

- Domain
- ASN
- IP range

Approved Inventory assets:

- Always shown in dashboard
- Scanned daily

---

# Microsoft Sentinel

Cloud-native SIEM and SOAR platform.

Capabilities:

- Log aggregation
- Threat detection
- Incident investigation
- Automated response

---

## Data Connectors

Examples:

Service connectors:
- Entra ID
- Defender for Cloud
- Office 365

Syslog / CEF connectors:
- Linux servers
- Network appliances

Threat intelligence via TAXII feeds.

---

## Analytics Rules

Used to detect threats using KQL queries.

Types:

- Microsoft Security
- Fusion
- Scheduled
- Near Real-Time (NRT)

---

## Automation

Uses Azure Logic Apps.

### Playbooks

Triggered by:

- Alerts
- Incidents

Example actions:

- Block IP
- Reset password
- Send alert notification

---

## Automation Rules

Used to trigger playbooks based on conditions.

Example:

If Incident Severity = High → run playbook.

---

## Sentinel Roles

Sentinel Responder:
- Manage incidents
- Cannot query logs

Sentinel Contributor:
- Create analytics rules
- Configure automation


# The 35 AZ-500 Concepts That Appear Most Often

These concepts appear repeatedly in AZ-500 exam scenarios.

---

# Identity & Access

1. Conditional Access policies  
2. Multi-Factor Authentication (MFA)  
3. Passwordless authentication  
4. Privileged Identity Management (PIM)  
5. Managed Identities  
6. Service Principals  
7. Microsoft Entra ID roles vs Azure RBAC  
8. Hybrid identity (Azure AD Connect)  
9. Pass-through Authentication vs Password Hash Sync  
10. Microsoft Entra ID Identity Protection risk policies  

---

# Network Security

11. Network Security Groups (NSG)  
12. NSG rule evaluation order  
13. Azure Firewall rule processing order  
14. Azure Firewall Standard vs Premium  
15. Azure DDoS Protection  
16. Service Endpoints  
17. Private Endpoints  
18. Hub-and-Spoke network architecture  
19. User Defined Routes (UDR)  
20. Just-in-Time (JIT) VM Access  

---

# Application Security

21. Azure Front Door vs Application Gateway  
22. Web Application Firewall (WAF)  
23. TLS encryption  
24. End-to-End TLS vs SSL Termination  
25. Managed Identities for application authentication  

---

# Data Protection

26. Azure Key Vault access policies  
27. Azure Key Vault RBAC model  
28. Storage Account security controls  
29. SQL vulnerability assessment  
30. Data encryption in transit  

---

# Security Operations

31. Microsoft Defender for Cloud Secure Score  
32. Defender plans (Servers, SQL, Storage)  
33. Microsoft Sentinel analytics rules  
34. Microsoft Sentinel playbooks (Logic Apps automation)  
35. Azure Policy security governance
