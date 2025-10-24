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
When nesting an existing security group to another security group, only members in the parent group will have access to shared resouces and applications. Nested group members don't have the same assigned membership as the parent group members.

**Microsoft 365:** Provides collaboration opportunities by giving group members access to a shared mailbox, calendar, files, SharePoint sites, and more.
This option also lets you give people outside of your organization access to the group. Members of a Microsoft 365 group can only include users. Owners of a Microsoft 365 group can include users and service principals.



