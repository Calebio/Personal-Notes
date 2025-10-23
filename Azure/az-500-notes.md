# Azure Security Engineer AZ-500 Notes

**Security principle:** Secure your identity and authentication system as a high priority in your organization's cloud security practice. Common security controls include:

- Restrict privileged roles and accounts
- Require strong authentication for all privileged access
- Monitor and audit high-risk activities

**Security principle:** Use managed application identities instead of creating human accounts for applications to access resources and execute code. Managed application identities provide benefits such as reducing the exposure of credentials. Automate the rotation of credentials to ensure the security of the identities.
For services that don't support managed identities, use Microsoft Entra ID to create a service principal with restricted permissions at the resource level. It is recommended to configure service principals with certificate credentials and fall back to client secrets for authentication.

**Security principle:** Authenticate remote servers and services from your client side to ensure you are connecting to trusted server and services. The most common server authentication protocol is Transport Layer Security (TLS), where the client-side (often a browser or client device) verifies the server by verifying the server’s certificate was issued by a trusted certificate authority.

