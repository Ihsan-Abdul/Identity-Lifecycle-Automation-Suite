# Hybrid Identity Lifecycle Automation Lab

## Project Overview
This project demonstrates the implementation of a comprehensive Hybrid Identity & Governance Framework, bridging an on-premises Active Directory environment with a cloud-native Microsoft Entra ID tenant.

The framework is built on a foundation of Identity Lifecycle Management (JML), automating the "Joiner, Mover, Leaver" process while enforcing enterprise-grade security standards. Rather than a static implementation, the project follows an iterative engineering path—evolving from ad-hoc scripting to scalable, API-driven automation and Zero Trust enforcement.

### Core Principles & Governance Logic:

*Defense in Depth:* Implementing security at every layer, from AGDLP group nesting on-premises to Conditional Access and MFA in the cloud.

*Least Privilege & RBAC:* Ensuring users receive only the access required for their current role, with automated "swaps" during department transfers to eliminate permission creep.

*Zero Trust Architecture:* Moving beyond perimeter security by verifying the user, the device (Intune), and the context before granting access.

*Identity Governance (IGA):* Maintaining audit readiness through automated reporting, tracking user entitlements, and identifying inactive account risks.


### Hybrid Environment Scope:

| Environment | Components & Protocols |
|-------------|------------------------|
| On-Premises | Windows Server 2022, Active Directory DS, PowerShell Automation Suite (JML) |
| Cloud (Entra ID) | Microsoft Graph API, Hybrid Sync, Enterprise Application Integration |
| Authentication | SAML 2.0, OpenID Connect (OIDC), OAuth 2.0, Multi-Factor Authentication |
| Endpoint & Security | Microsoft Intune (MDM), Conditional Access Policies, Device Compliance |


The focus was to:

- *Implement RBAC:* Standardizing access via AGDLP design and Entra ID security groups.

- *Automate Lifecycle:* Bridging the Joiner-Mover-Leaver process from local AD to Cloud Apps.

- *Enforce Zero Trust:* Moving beyond simple passwords to device compliance and MFA.

- *Ensure Governance:* Using PowerShell to audit entitlements and flag inactive identities.


### Project Roadmap

| Section | Focus | Key Technologies |
|---------|-------|------------------|
| [Part 1: On-Prem Infrastructure](01-On-Prem-Infrastructure/README.md) | AD Forest Build & JML Automation | PowerShell, AD DS, AGDLP |
| [Part 2: Hybrid Cloud Integration](02-Cloud-Integration/README.md) | Graph API & Enterprise SSO | Entra ID, SAML, OIDC, Graph |
| [Part 3: Governance & Security](03-Governance-Compliance/README.md) | Zero Trust & Compliance | Intune, Conditional Access |



All scripts have been tested in an isolated lab environment with screenshot documentation available.

---
