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






