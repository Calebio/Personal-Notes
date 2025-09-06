# Azure Administrator AZ-104 Notes

**Practice Exam:** https://certpreps.com/exams/az104/practice/free-practice-exam-1copy-8/

## Azure CloudShell
- Times out after 20mins
- The files in cloudshell can be persisted and accessed in another session.
- Doesn't permit multiple sessions at a time

**When should you use Azure CloudShell?**
- When you need a secure cloud bases CLI. 
- When you don't have access to your own computer and you need to do something using CLI.

**Bash**

**PowerShell**

**Deploy Azure infrastructure by using JSON ARM templates**

## Manage identities and governance in Azure

- **EntraID**
Microsoft Entra ID is a cloud-based identity and access management service provided by Microsoft.

| **Feature**                            | **Microsoft Entra ID**                                                                                      | **Active Directory Domain Services (AD DS)**                       |
|----------------------------------------|-------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------|
| **Schema & Object Types**             | Fewer object types; no `computer` class, uses `device` class; schema is easily extensible & reversible.     | Rich schema with `computer` class; extensions are rigid & harder to roll back. |
| **Device/Computer Joining**           | Devices join as `devices` (modern join); no traditional domain membership.                                 | Computers join domain as `computers` (traditional join).           |
| **Management Model**                  | No GPO support; relies on modern cloud-based management (e.g., Intune, MDM).                               | Strong GPO support; centralized, on-premises management of users, computers, and settings. |
| **Organizational Structure**          | No Organizational Units (OUs); uses groups to organize & delegate.                                         | Supports hierarchical OUs for delegation & GPO scoping.            |
| **Applications & Service Principals** | Applications (`Application` class) are separate from instances (`servicePrincipal`); supports multi-tenant apps. | No equivalent application/service principal distinction; not cloud-oriented. |
| **Primary Strengths**                 | Cloud-native; authenticates/authorizes users, devices, and apps for services like Microsoft 365; scales globally. | On-premises; manages local resources, computers, and users in a domain-controlled environment. |


**Compare Microsoft Entra ID and Active Directory Domain Services**

| **Feature / Characteristic**           | **Microsoft Entra ID**                                                                                       | **Active Directory Domain Services (AD DS)**                            |
|-----------------------------------------|---------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------|
| **Deployment**                         | Cloud-based, multi-tenant identity solution.                                                                  | On-premises (or in Azure VM), Windows Server-based directory service.    |
| **Primary Purpose**                    | Identity & access management for cloud/internet apps.                                                         | Full-fledged directory service for on-premises network resources.        |
| **Protocol Support**                   | Uses HTTP/HTTPS (SAML, WS-Federation, OpenID Connect for auth, OAuth for authorization).                      | Uses Kerberos (primarily) & LDAP for authentication & directory queries. |
| **Directory Structure**                | Flat structure — no OUs, no GPOs. Objects organized using groups.                                             | Hierarchical X.500-based structure with OUs & GPOs.                      |
| **Schema & Object Types**             | Fewer object types; no `computer` class, uses `device` class; extensible & reversible.                        | Rich schema including `computer` class; rigid & harder to extend.        |
| **Device/Computer Joining**           | Devices join as `devices` (modern join), no traditional domain join.                                          | Computers join as `computer` objects into the domain (traditional).      |
| **Management Model**                  | Modern management via Intune/MDM; no GPO support.                                                             | Centralized, on-premises management using GPOs & delegation.             |
| **Organizational Structure**          | No OUs; objects grouped by group memberships for delegation/policy.                                           | Supports OUs & domain trusts for delegation & scoping.                   |
| **Applications & Federation**         | Applications (`Application` & `servicePrincipal`) separated; supports federation with many 3rd-party services. | No equivalent; federation handled separately via AD FS if needed.        |
| **Query Methods**                     | Queried via REST APIs over HTTP/HTTPS.                                                                        | Queried via LDAP.                                                         |
| **DNS Usage**                         | Does not depend on DNS internally.                                                                            | Uses DNS heavily for locating domain controllers & resources.            |
| **Authentication Protocols**         | Modern protocols: SAML, OpenID Connect, WS-Fed over HTTP/HTTPS.                                              | Kerberos + NTLM for authentication.                                      |
| **Scalability & Availability**        | Built for global internet scale with high availability.                                                       | Can be scaled in Azure VMs, but requires careful design & extra disks.   |
| **Federation & Trusts**               | Federated with many external identity providers (e.g., Facebook).                                             | Trusts between domains & forests for delegation.                         |
| **Installation/Storage**             | No server to install — provided as SaaS.                                                                      | Requires server(s), disks, and proper configuration of storage (DB, logs).|


**Compare Microsoft Entra ID P1 and P2 plans**

| **Feature / Functionality**                     | **Microsoft Entra ID P1**                                                                                         | **Microsoft Entra ID P2**                                                                                     |
|--------------------------------------------------|-------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------|
| **License Model**                               | Paid add-on or included in Enterprise Mobility + Security (EMS) suites.                                          | Paid add-on or included in EMS E5 suite.                                                                      |
| **Trial Availability**                          | Available as part of free trial (via P2 trial, which also covers P1 features).                                   | Available as part of free trial.                                                                              |
| **Self-Service Group Management**               | ✅ Users can create, manage, and request to join groups; owners can approve requests and manage memberships.     | ✅ Same as P1.                                                                                                 |
| **Advanced Security Reports & Alerts**         | ✅ Machine-learning based reports showing anomalies and inconsistent access patterns.                            | ✅ Same as P1.                                                                                                 |
| **Multi-Factor Authentication (MFA)**           | ✅ Full MFA support for Azure, M365, Dynamics 365, VPN/RADIUS, and gallery apps (not for non-browser apps).      | ✅ Same as P1.                                                                                                 |
| **Microsoft Identity Manager (MIM) Licensing** | ✅ Included for hybrid identity scenarios integrating on-premises stores with cloud.                             | ✅ Same as P1.                                                                                                 |
| **Service Level Agreement (SLA)**              | ✅ 99.9% uptime SLA.                                                                                             | ✅ Same as P1.                                                                                                 |
| **Password Reset with Writeback**              | ✅ Self-service password reset, respecting on-premises AD password policies.                                     | ✅ Same as P1.                                                                                                 |
| **Cloud App Discovery**                        | ✅ Discover frequently used cloud-based apps in your environment.                                               | ✅ Same as P1.                                                                                                 |
| **Conditional Access (Device/Group/Location)** | ✅ Policy-based access controls based on device, user group, or location.                                       | ✅ Same as P1.                                                                                                 |
| **Microsoft Entra Connect Health**            | ✅ Operational insights with alerts, performance, usage, and configuration monitoring.                           | ✅ Same as P1.                                                                                                 |
| **Microsoft Entra ID Protection**             | ❌ Not included.                                                                                                 | ✅ Advanced risk-based policies, risky user & sign-in detection, and behavior analysis.                       |
| **Privileged Identity Management (PIM)**      | ❌ Not included.                                                                                                 | ✅ Manage privileged accounts with just-in-time access, approval workflows, and time-limited admin roles.     |


**Custom Security Attribute**
Custom security attributes in Microsoft Entra ID are business-specific attributes (key-value pairs) that you can define and assign to Microsoft Entra objects. These attributes can be used to store information, categorize objects, or enforce fine-grained access control over specific Azure resources.

***Features of custom security attributes***
- Available tenant-wide
- Include a description
- Support different data types: Boolean, integer, string
- Support single value or multiple values
- Support user-defined free-form values or predefined values
- Assign custom security attributes to directory synced users from an on-premises Active Director

**Notes on licensing errors:**
'MutuallyExclusiveViolation' ==> whats the meaning?
'DependencyViolation' ==> what's the meaning?
'LicenseAssignmentAttributeConcurrencyException' what's the meaning?


**Azure Policy design**
Azure Policy enforces organizational standards and assesses compliance at scale. It evaluates Azure resources and actions by comparing their properties to business rules, providing an aggregated view of the environment's overall state. This policy allows for detailed analysis down to each resource and policy level with granularity. Six policy resources are available in Azure, they are

Definitions, Initiatives, Assignments, Exemptions, Attestations, Remediations.


- ***Definitions***
Azure Policy definitions describe resource compliance conditions and the effect to take if a condition is met. Several settings determine which resources are evaluated by any Azure Policy. You explore these settings in the next unit, Azure Policy definitions. The primary concept to which these settings can be applied is scope.



- ***Initiatives***
Azure Policy initiatives, also known as a policy set, allow you to group several policy definitions to simplify assignments and management because you work with the initiatives as a single item. Initiatives offer a streamlined and automated approach to governance, allowing organizations to manage and monitor compliance at scale.


- ***Assignments***
Policy assignments define which resources are evaluated by a policy definition or initiative. Policy assignments can be done in the portal, an API call, or through the command line interface.


- ***Exemptions***
Use the Policy exemptions feature to exempt a resource hierarchy or an individual resource from evaluation of initiatives or definitions. Resources that are exempt count toward overall compliance but can't be evaluated or have a temporary waiver. They're created as a child object on the resource hierarchy, or the individual resource granted the exemption.


- ***Attestations***
Policy attestations are used by Azure Policy to set compliance states of resources or scopes targeted by manual policies. Each applicable resource requires one attestation for each manual policy assignment. For ease of management, manual policies should be designed to target the scope that defines the boundary of resources whose compliance state needs to be attested.


- ***Remediations***
The policy remediation task feature is used to bring resources into compliance based on a definition and assignment. Resources that are noncompliant to a modify or `deployIfNotExists` definition assignment can be brought into compliance by using a remediation task. Resources that are newly created or updated that are applicable to a `deployIfNotExists` or modify definition assignment are automatically remediated.

**Azure Storage**

![](@attachment/Clipboard_2025-07-13-19-27-23.png)

- ***Azure Blob Storage***
Azure Blob Storage is Microsoft's object storage solution for the cloud. Blob Storage is optimized for storing massive amounts of unstructured or nonrelational data, such as text, audio, video or binary data.


- ***Azure Files***
Azure Files enables you to set up highly available network file shares. Shares can be accessed by using the Server Message Block (SMB) protocol and the Network File System (NFS) protocol. Multiple virtual machines can share the same files with both read and write access.


- ***Azure Queue Storage***
Azure Queue Storage is used to store and retrieve messages. Queue messages can be up to 64 KB in size, and a queue can contain millions of messages. Queues are used to store lists of messages to be processed asynchronously.


- ***Azure Table Storage***
Azure Table storage is a service that stores nonrelational structured data (also known as structured NoSQL data) in the cloud, providing a key/attribute store with a schemaless design. Because Table storage is schemaless, it's easy to adapt your data as the needs of your application evolve.


**Azure Backup**
The Azure Backup service provides simple, secure, and cost-effective solutions to back up your data and recover it from the Microsoft Azure cloud.

**Azure DNS**
Azure DNS is a hosting service for Domain Name System (DNS) domains that provides name resolution by using Microsoft Azure infrastructure.

Azure DNS allows you to host and manage your domains by using a globally distributed name-server infrastructure. It allows you to manage all of your domains by using your existing Azure credentials.

Azure DNS acts as the SOA for the domain.

You can't use Azure DNS to register a domain name; you need to register it by using a third-party domain registrar.

**DNS record types**
Configuration information for your DNS server is stored as a file within a zone on your DNS server. Each file is called a record. The following record types are the most commonly created and used:

- ***A*** is the host record, and is the most common type of DNS record. It maps the domain or host name to the IP address.
- ***CNAME*** is a Canonical Name record that's used to create an alias from one domain name to another domain name. If you had different domain names that all accessed the same website, you'd use CNAME.
- ***MX*** is the mail exchange record. It maps mail requests to your mail server, whether hosted on-premises or in the cloud.
- ***TXT*** is the text record. It's used to associate text strings with a domain name. Azure and Microsoft 365 use TXT records to verify domain ownership.

Additionally, there are the following record types:

- Wildcards
- CAA (certificate authority)
- NS (name server)
- SOA (start of authority)
- SPF (sender policy framework)
- SRV (server locations)

***Security features***
Azure DNS provides the following security features:

- Role-based access control, which gives you fine-grained control over users' access to Azure resources. You can monitor their usage and control the resources and services to which they have access.
- Activity logs, which let you track changes to a resource and pinpoint where faults occurred.
- Resource locking, which gives you a greater level of control to restrict or remove access to resource groups, subscriptions, or any Azure resources.


git clone https://github.com/MicrosoftDocs/mslearn-host-domain-azure-dns.git


## Extra notes

- Anything like azure runbooks?
- Azure Backups (read up)
- "Virtual Machine Contributor" role alone does not grant login permissions. The "Virtual Machine User Login" role is required to allow the user to sign in to the virtual machine.
- "Virtual Machine Contributor" role allows users to manage virtual machines and attached disks but does not grant permissions to manage disk snapshots. Managing disk snapshots requires specific permissions, such as those granted by the "Disk Snapshot Contributor" role.

| Storage Component   | Min Backup Frequency via Azure Backup | Notes                                      |
| ------------------- | ------------------------------------- | ------------------------------------------ |
| **Blob Storage**    | Once per day                          | Full account-level backups only            |
| **File Shares**     | Every 4 hours                         | Up to 6 times per day                      |
| **Tables / Queues** | N/A (custom only)                     | Frequency depends on custom implementation |


- Global Admins do not have permissions to assign custom security attributes to Entra objects.


- Azure Storage accounts allow a maximum of 5 stored access policies per container. 

- In Azure Blob Storage, you can create up to 2 immutable blob stored access policies per container.

- Encryption scopes in Azure are designed to apply to Blob storage, specifically to containers and individual blobs within a storage account.

- Custom security attributes in Microsoft Entra ID can only be assigned to users and service principals (enterprise applications). Security groups are not supported objects for custom security attributes.

- Custom security attributes cannot be assigned to Microsoft 365 groups and Security groups.


- Azure supports Azure role assignment conditions for Blob Storage containers and Queue Storage. 

- cloud-init.txt is used to customize a Linux VM as it boots for the first time. You can use cloud-init to install packages(e.g trusted root certificate authority CA) and write files, or to configure users and security.

- Action groups in Azure are collections of notification preferences and actions that are triggered when an alert is generated by Azure Monitor, Azure Service Health, or Azure Advisor. They are used to manage alerts by sending notifications (such as voice calls, SMS, push notifications, or emails) and executing automated actions in response to alerts.


- The "Virtual Machine User Login" role is required to allow the user to sign in to the virtual machine, even if the user has "Virtual Machine Contributor" role they still require the "Virtual Machine User Login" role.


- "Virtual Machine Contributor" role allows users to manage virtual machines and attached disks but does not grant permissions to manage disk snapshots.

- Application security groups allow you to group together the network interfaces from multiple virtual machines, and then use the group as the source or destination in an NSG rule. The network interfaces must be in the same virtual network.


- Azure Network Watcher is a regional service that allows you to monitor and diagnose conditions at a network scenario level in, to, and from Azure.

- Azure Object Replication is a feature of Azure Blob Storage, and it works at the container level. To replicate blobs from one storage account to another, you must configure object replication policies between source and destination containers. Object replication is supported by both **general-purpose v2 storage accounts (StorageV2)** and **premium block blob storage accounts (BlobStorage)**.

- Microsoft Entra ID tenant, you can only assign licenses to individual users like. Group-based licensing requires a paid version of Microsoft Entra ID (such as P1 or P2).

- If a user has Contributor role in an RG for a specific resource (e.g Storage Account), they can create that servcie(In this case storage account) in that RG.

- Virtual network peering allows you to peer virtual networks globally across different regions and subscriptions, even when they are linked to different Microsoft Entra tenants.

- Public IP addresses can be associated with a load balancer or a network interface to enable external communication with resources like virtual machines or services behind the load balancer.

- By using service chaining and configuring user-defined routes (UDRs), you can route traffic from the on-premises network, through the VPN gateway on VNet1, to VNet2. This setup leverages the existing peering between VNet1 and VNet2 without requiring additional costly infrastructure like Azure Firewall or ExpressRoute, thereby minimizing costs. UDRs will ensure that the traffic is directed through the correct paths.

- To ensure that the access keys for a storage account rotate automatically, you should store the keys in Azure Key Vault and enable key rotation policies. Key Vault allows for the secure storage of secrets, including storage account access keys, and provides the ability to configure automatic key rotation.

- According to Microsoft Entra documentation, a group cannot be deleted if it has licenses directly assigned, considering that the tenant needs p1 or p2 license to be ables to license a group.

- Azure Backup requires that the storage account used for diagnostic settings (like Azure Backup reports) be in the same region as the Recovery Services vault.

- Even if self-service password reset (SSPR) is enabled for all users, the regular SSPR policy does not apply to users in administrative roles, such as Security Administrator. Administrators must go through a two-gate policy to reset their password, which differs from the standard SSPR process configured for non-admin users. 

- Microsoft Entra group-based licensing does not support nested group inheritance.

- When a license is assigned to a group, the licensing options are inherited by all members of the group, and individual users cannot modify or enable specific license features that were disabled at the group level, e.g if a license does not allow microsoft cloud apps, the user can't modify it individually.

- Azure resource locks such as Read-only and Delete are designed to prevent deletion or modification of resources, but they do not block move operations.

- When setting up Log Analytics workspace to store backup reporting data, the location and subscription where the Log Analytics workspace can be created is independent of the location and subscription of where the vault exists.


- Object replication can be used to replicate blobs between storage accounts. Before configuring object replication, you must enable blob versioning for both storage accounts, and you must enable the change feed for the source account.

- `netstat -an` will list the ports that the server is listening on.

- An internal load balancer is designed to distribute network traffic within a virtual network (VNet) across virtual machines (VMs) without exposing them to the internet


- A private endpoint maps the storage account to a private IP address in the virtual network, preventing traffic from traversing the internet and providing a secure connection.


- Azure App service Contributor role provides the necessary permissions to manage the web app, including deploying content using their Microsoft Entra credentials to deploy content to it.

- When a private DNS zone with auto-registration enabled is added a vnet, resources like vm in the vnet will be automatically added as an A record to the fabrikam.com private DNS zone.

- SSPR can be directly assigned only to a group of users or to ALL users but not to individual users


- While it is generally recommended to use a Key Vault in the same region as the resource for performance and latency considerations, keyvault can be accessed globally.

- In Azure Storage, the routing preference setting allows you to specify how user traffic reaches your storage account—either through the Microsoft global network or over the public internet. 

- When you move a resource like a web app on App Service to a different resource group, the region of the resource does not change—it stays in the same physical location unless explicitly redeployed.

- `SignedVersion (sv)`, `SignedPermissions (sp)`, `Signature (sig)`, `SignedServices (ss)` and `SignedResourceTypes (srt)` are required to create a SAS token


- To restore a VM backup follow these steps in this sequence:
  - Create a Recovery Services vault.
  - Set Replication to Zone-redundant storage (ZRS).
  - Create a backup policy and configure the backup for the VM.






- Storage Blob Data Contributor vs Storage Account Contributor (Research).

**Side Note:** In practice test 2, check question 3 for flagging.


**What are the areas I find difficult at the moment**

- Licensing/IAM
- Implement and manage virtual networking
- Deploy and manage Azure compute resources

