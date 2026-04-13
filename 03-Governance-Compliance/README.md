# Part 3: Governance and Security

## Overview
This final phase completes the "Hybrid: setup by shifting the focus from identity creation to identity protection and oversight by implementing Zero Trust access principles.

This phase demonstrates how to enforce context-aware access using Conditional Access and Intune, integrate modern SaaS applications via SSO protocols (SAML/OIDC), and maintain audit readiness through automated governance reporting.

## Table of Contents



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

[View MFA Policy Configuration](images/)

[View MFA User Prompt](images/)

### Problem — Sensitive Data Requires Stronger Controls

- While MFA protects identity, it does not protect against access from compromised or unmanaged devices.

- In this environment, SharePoint is used as a central location for business data. Allowing access from any device introduces risk.

- This introduced: 

### RBAC Enforcement (SharePoint Access Control)

Before applying Conditional Access, access to SharePoint was structured using role-based access control.

Using Microsoft SharePoint:

- Finance_Analyst (Global Group) → Members (edit access)
- Finance_Manager (Global Group) → Owners (full control)

This ensures:

- Access is determined by role, not individual assignment

[Finance Portal Members](images/Finance_Analyst)
[Finance Portal Owners](images/Finance_Manager)


### Risk-Based Access Policy (SharePoint)

To secure this resource, a stricter Conditional Access policy was created specifically for SharePoint Online.

[SharePoint Conditional Access Policy](images/)

Policy Logic 
```
User = All  
Application = SharePoint Online  
Grant Access = Require:
    - MFA  
    - Device marked as compliant

```

### Device Trust (Intune Integration)

To support this policy, device compliance was defined using Microsoft Intune.

***Compliance Requirements:***

- Minimum OS version
- Firewall enabled
- BitLocker encryption

Only devices meeting these conditions are considered trusted.

[Intune Compliance Policy](images/)
[Compliant Device State](images/)


### Enforcement Model

At this stage, access to SharePoint is only granted when all conditions are satisfied:

```
User Identity (MFA)  
+  
Device State (Compliant)  
+  
Resource Policy (SharePoint CA Policy)
```

### Key Outcome

This phase demonstrates the implementation of Zero Trust access controls:

- Identity alone is not trusted
- Device health is verified
- Access is restricted based on resource sensitivity
- Access is evaluated per request, not assumed




- The Problem: SharePoint contains sensitive corporate documentation. Access from an unmanaged or "unhealthy" device poses a data exfiltration risk.

- The Solution: Integrated Microsoft Intune to define device compliance (Minimum OS version, active Firewall, and BitLocker encryption).

[View SharePoint Policy Configuration](images/)

Grant Control - Access to SharePoint is only granted if:

- The User satisfies MFA.

- The Device is marked as Compliant in Intune.

[View Intune Compliance Policy & Enrolled Device State](images/)


*Implementation Note: This completes the Zero Trust triangle: **Verify Identity (MFA) + Verify Device (Intune) + Verify Context (SharePoint Policy)**.*









## Phase 2 — Modern Authentication (SSO Integrations)






















## Phase 3 - Identity Governance (Access Review Reporting)





















