# Part 3: Governance and Security

## Overview
This final phase completes the Hybrid setup by shifting the focus from identity creation to identity protection and oversight by implementing Zero Trust access principles.

This phase demonstrates how to enforce context-aware access using Conditional Access and Intune, integrate modern SaaS applications via SSO protocols (SAML/OIDC), and maintain audit readiness through automated governance reporting.

---

## Table of Contents
- [Phase 1 — Zero Trust Access Enforcement](#phase-1--zero-trust-access-enforcement-conditional-access-and-intune)
  - [Baseline MFA Policy](#baseline-mfa-policy)
  - [RBAC Enforcement (SharePoint Access Control)](#rbac-enforcement-sharepoint-access-control)
  - [Risk-Based Access Policy (SharePoint)](#risk-based-access-policy-sharepoint)
  - [Device Trust (Intune Integration)](#device-trust-intune-integration)
  - [Enforcement Model](#enforcement-model)
- [Phase 2 — Modern Authentication (SSO Integrations)](#phase-2--modern-authentication-sso-integrations)
- [Phase 3 — Identity Governance (Access Review Reporting)](#phase-3--identity-governance-access-review-reporting)
- [Technical Skills Demonstrated](#technical-skills-demonstrated)
- [Outcome](#outcome)

---


## Phase 1 - Zero Trust Access Enforcement (Conditional Access and Intune)

In a hybrid environment, identity alone is not enough to grant access. This phase focuses on enforcing access controls based on who the user is, what device they are using, and what resource they are trying to access.


### Baseline MFA Policy

The first step was implementing a global Conditional Access policy in Microsoft Entra ID to enforce Multi-Factor Authentication across all cloud applications.

- Policy Logic:
  ```
  User = Any
  App = All Cloud Apps
  Grant Access =  Require MFA
  ```

This establishes the baseline that valid credentials alone are not sufficient for access.

[View MFA Policy Configuration](images/01-mfa-policy.png)

[View MFA User Prompt](images/02-mfa-policy-proof.png)

### Problem — Sensitive Data Requires Stronger Controls

- While MFA protects identity, it does not protect against access from compromised or unmanaged devices.

- In this environment, SharePoint is used as a central location for business data. Allowing access from any device introduces risk.

- This introduced the need for stricter, resource-specific controls beyond MFA alone.

  
### RBAC Enforcement (SharePoint Access Control)

Before applying Conditional Access, access to SharePoint was structured using role-based access control to ensure permissions are assigned by role, not by individual.

**Group Structure:**
- `Finance_Analyst` (Global Group) → Members (edit access)
- `Finance_Manager` (Global Group) → Owners (full control)

This ensures access is determined by role assignment, not direct permission grants — mirroring the AGDLP model implemented on-premises.

[View Finance Portal Members](images/04-sharepoint-finance-members.png)

[View Finance Portal Owners](images/05-sharepoint-finance-Owners.png)


### Risk-Based Access Policy (SharePoint)

A stricter Conditional Access policy was created specifically for SharePoint Online, requiring both MFA and a compliant device before access is granted.

**Policy Logic:**
```
User     = All Users
App      = SharePoint Online
Grant    = Require MFA
          + Require device marked as compliant
```

[View SharePoint Conditional Access Policy](images/03-SharePoint-policy.png)


### Troubleshooting - Hardware Compatibility & The "Break Glass" Necessity

After enforcing the SharePoint Compliance policy, I attempted to verify access. Immediately upon attempting to access SharePoint, I was met with a device authentication prompt requiring the Microsoft Intune Company Portal.

[Authenticate device pop-up](images/06a-block-unmanaged-device.png)

When I attempted to register my personal device, the process failed. My physical hardware (macOS) did not meet the Intune requirement+. I was effectively locked out of the M365 Admin Center and SharePoint.

[Preview that Mac is not up to date](images/06b-intune-hardware-incompatibility.png)

To restore access and prevent future lockouts, I implemented a Break-Glass Group Strategy:

Group Creation: I created a Security Group named Admin-Exclude.

Policy Exclusion: I modified the Conditional Access policy to exclude this group, allowing me to bypass the hardware compliance requirement for administrative tasks.

Identity Governance: I added my admin account to this group, successfully restoring access to the M365 Admin Center.


[Group exclusion logic inside the CA policy](images/06c-ca-admin-exclusion-fix.png)

*The Admin-Exclude group was also applied to the baseline MFA policy as a secondary precaution, ensuring administrative access is never fully blocked by a misconfigured policy.*

**What I Learned and Why This Matters**

In the enterprise, "Break-Glass" or Emergency Access Accounts are a non-negotiable security requirement.

Resilience: If a primary identity provider (like an MFA service) goes down or a global policy is misconfigured, these accounts ensure the organization isn't permanently locked out of its own tenant.

Zero Trust Balance: This project highlights the delicate balance between high-security enforcement (Intune/MFA) and Business Continuity.


### Device Trust (Intune Integration)

To support the device compliance requirement, compliance standards were defined in Microsoft Intune. Only devices meeting these conditions are considered trusted.

**Compliance Requirements:**

- Password Requirement: Minimum 8 characters to prevent brute-force attacks.
- Microsoft Defender Antimalware: Required to be active to prevent "dirty" devices from entering the environment.
- Firewall: Required to protect the endpoint in untrusted network environments.

[View Intune Compliance Policy](images/07a-intune-policy.png)
[View Intune Policy with Admin Exclusion](images/07b-intune-policy-exclusion.png)


### Enforcement Model

Access to SharePoint is only granted when all three conditions are satisfied simultaneously:

```
User Identity (MFA)
+
Device State (Compliant via Intune)
+
Resource Policy (SharePoint Conditional Access)
```

*This completes the Zero Trust triangle: **Verify Identity + Verify Device + Verify Context***


### Key Outcome

This phase demonstrates the implementation of Zero Trust access controls:

- Identity alone is not trusted — MFA required
- Device health is verified — Intune compliance enforced
- Access is restricted based on resource sensitivity — SharePoint gets stricter controls than general apps
- Access is evaluated per request, not assumed

---


## Phase 2 — Modern Authentication (SSO Integrations)





















---

## Phase 3 - Identity Governance (Access Review Reporting)





















