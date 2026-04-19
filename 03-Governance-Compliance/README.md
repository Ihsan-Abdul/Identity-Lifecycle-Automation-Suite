# Part 3: Governance and Security

## Overview

This final phase completes the hybrid setup by shifting the focus from identity creation to identity protection and oversight through Zero Trust access principles.

This phase demonstrates how to:
- Enforce context-aware access using Conditional Access and Intune
- Integrate modern SaaS applications via SSO protocols (SAML/OIDC)
- Maintain audit readiness through automated governance reporting

---

## Table of Contents
- [Phase 1 — Zero Trust Access Enforcement](#phase-1---zero-trust-access-enforcement-conditional-access-and-intune)
  - [Baseline MFA Policy](#baseline-mfa-policy)
  - [RBAC Enforcement (SharePoint Access Control)](#rbac-enforcement-sharepoint-access-control)
  - [Risk-Based Access Policy (SharePoint)](#risk-based-access-policy-sharepoint)
  - [Troubleshooting - Hardware Compatibility & The "Break Glass" Necessity](#troubleshooting---hardware-compatibility--the-break-glass-necessity)
  - [Device Trust (Intune Integration)](#device-trust-intune-integration)
  - [Enforcement Model](#enforcement-model)
  - [Outcome](#outcome)

- [Phase 2 — Modern Authentication (SSO Integrations)](#phase-2--modern-authentication-sso-integrations)
  - [Overview](#overview-1)
  - [SAML Application Integration](#saml-application-integration)
  - [Access Control — Group-Based Assignment](#access-control--group-based-assignment)
  - [SAML Handshake Validation](#saml-handshake-validation)
  - [Troubleshooting — No Backend Application](#troubleshooting--no-backend-application)
  - [Outcome](#outcome-1)

- [Phase 3 — Identity Governance & Risk Review](#phase-3--identity-governance--risk-review)
  - [Overview](#overview-2)
  - [Initial Script — Inactive User Audit](#initial-script--inactive-user-audit)
  - [Limitation — No Access Context](#limitation--no-access-context)
  - [Improved Script — Identity Risk & Privileged Context](#improved-script--identity-risk--privileged-context)
  - [Outcome](#outcome-2)
  - [RBAC Drift Detection — Access Alignment](#rbac-drift-detection--access-alignment)
  - [Limitation](#limitation-1)
  - [Discovery — Permission Creep](#discovery--permission-creep)
  - [Final Enhancement — Access Drift Detection](#final-enhancement--access-drift-detection)
  - [Outcome](#outcome-3)


---


## Phase 1 - Zero Trust Access Enforcement (Conditional Access and Intune)

In a hybrid environment, identity alone is not enough to grant access. This phase focuses on enforcing access controls based on **who** the user is, **what device** they are using, and **what resource** they are trying to access.

---

### Baseline MFA Policy

The first step was implementing a global Conditional Access policy in Microsoft Entra ID to enforce Multi-Factor Authentication across all cloud applications.

- Policy Logic:
  ```
  User = All Users
  App = All Cloud Apps
  Grant Access =  Require MFA
  ```

This establishes the baseline that valid credentials alone are not sufficient for access.

[View MFA Policy Configuration](images/01-mfa-policy.png)

[View MFA User Prompt](images/02-mfa-policy-proof.png)

### Problem — Sensitive Data Requires Stronger Controls

- While MFA protects identity, it does not protect against access from compromised or unmanaged devices.

- In this environment, SharePoint is used as a central location for business data. Allowing access from any device introduces risk.

- This highlighted that MFA alone is insufficient for protecting sensitive resources. Additional controls based on device trust and application context are needed.
  
---
  
### RBAC Enforcement (SharePoint Access Control)

Before applying Conditional Access, access to SharePoint was structured using role-based access control to ensure permissions are assigned by role, not by individual.

**Group Structure:**

- `Finance_Analyst` (Global Group) → Members (edit access)
- `Finance_Manager` (Global Group) → Owners (full control)

This ensures access is determined by role assignment, not direct permission grants mirroring the AGDLP model implemented on-premises.

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

---

### Troubleshooting - Hardware Compatibility & The "Break Glass" Necessity

After enforcing the policy, a real-world failure scenario occurred. Immediately upon attempting to access SharePoint, I was met with a device authentication prompt requiring the Microsoft Intune Company Portal.


[Authenticate device pop-up](images/06a-block-unmanaged-device.png)

When I attempted to register my personal device, the process failed. My physical hardware (macOS) did not meet the Intune requirement. I was locked out of the M365 Admin Center and SharePoint.

[Preview that Mac is not up to date](images/06b-intune-hardware-incompatibility.png)

**The Fix — Break-Glass Group Strategy:**

1. **Group Creation:** I created a Security Group named Admin-Exclude.

2. **Policy Exclusion:** I modified the Conditional Access policy to exclude this group, allowing me to bypass the hardware compliance requirement for administrative tasks.

3. **Identity Governance:** I added my admin account to this group, successfully restoring access to the M365 Admin Center.


[Group exclusion logic inside the CA policy](images/06c-ca-admin-exclusion-fix.png)

 **Note:** *The `Admin-Exclude` group was also applied to the baseline MFA policy as a secondary precaution, ensuring administrative access is never fully blocked by a misconfigured policy.*
 

### What I Learned and Why This Matters**

In the enterprise, "Break-Glass" or Emergency Access Accounts are a non-negotiable security requirement:

- **Resilience:** If a primary identity provider (like an MFA service) goes down or a global policy is misconfigured, these accounts ensure the organization isn't permanently locked out of its own tenant

- **Zero Trust Balance:** This project highlights the delicate balance between high-security enforcement (Intune/MFA) and Business Continuity

---

### Device Trust (Intune Integration)

To support the device compliance requirement, compliance standards were defined in Microsoft Intune. Only devices meeting these conditions are considered trusted.

**Compliance Requirements:**

| Requirement | Purpose |
|-------------|---------|
| Password: Minimum 8 characters | Prevent brute-force attacks |
| Microsoft Defender Antimalware | Prevent unmanaged or non-compliant devices from entering the environment |
| Firewall enabled | Protect the endpoint in untrusted network environments |

[View Intune Compliance Policy](images/07a-intune-policy.png)
[View Intune Policy with Admin Exclusion](images/07b-intune-policy-exclusion.png)

---

### Enforcement Model

Access to SharePoint is only granted when all three conditions are satisfied simultaneously:

```
User Identity (MFA)
+
Device State (Compliant via Intune)
+
Resource Policy (SharePoint Conditional Access)
```

*This completes the Zero Trust access model: ***Verify Identity + Verify Device + Enforce Resource Policy***.*

---

### Outcome

This phase demonstrates the implementation of Zero Trust access controls:

- Identity alone is not trusted — MFA required
- Device health is verified — Intune compliance enforced
- Access is restricted based on resource sensitivity — SharePoint gets stricter controls than general apps
- Access is evaluated per request, not assumed

---


## Phase 2 — Modern Authentication (SSO Integrations)

### Overview

With Zero Trust access controls enforcing MFA and device compliance, the next step was enabling authentication across enterprise applications without introducing additional credentials.

This phase focuses on configuring SAML 2.0 Single Sign-On (SSO) in Microsoft Entra ID to integrate external applications, allowing users to authenticate once and access assigned resources through a centralized identity provider.

---
### SAML Application Integration

To demonstrate SSO, a custom SAML 2.0 application was configured in Microsoft Entra ID.

While modern applications often use OpenID Connect (OIDC), SAML remains widely used for enterprise SaaS integrations and legacy systems.


**Configuration:**

| Field | Value | Purpose |
|-------|-------|---------|
| Identifier (Entity ID) | `https://saml-test-app.com` | Uniquely identifies the application |
| Reply URL (ACS URL) | `https://jwt.ms` | Endpoint where the SAML assertion is sent |


The Reply URL was set to https://jwt.ms to allow validation of the SAML response without requiring a live application backend.

[Basic SAML Configuration](images/08-saml-basic-config.png)



### Access Control — Group-Based Assignment

Following the AGDLP pattern established in Phase 1, access was granted using group membership rather than individual assignment, maintaining the RBAC model.

`Marketing_Staff_GG` was assigned to the application, ensuring that any user synced from the on-premises "Marketing" OU automatically receives SSO access upon migration to the cloud.

This enforces the Principle of Least Privilege. 

[Group Assignment](images/09-user-assignment.png)

---

### SAML Handshake Validation

To verify the handshake, SAML Tracer was used to capture and inspect the SAML response.

**1. User Experience Validation:**

First, I confirmed the application appeared correctly in the user's My Apps portal, proving the Entra ID assignment was active.

[My Apps Portal](images/12-apps-portal.png)



**2. Assertion validation (SAML Response):**

By intercepting the SAML POST request, the assertion issued by Entra ID was inspected.


|Element | Status | Validation |
|--------|--------|------------|
| Issuer | PASS | Matched Entra ID tenant (sts.windows.net). |
| Subject | PASS | Correct user `mvance@IAMCompanylocal.onmicrosoft.com`. |
| AttributeStatement | PASS | Confirmed that `givenname`, `surname`, and `email` were mapped correctly. |


[SAML-tracer Response](images/11-saml-tracer-proof.png)

---

### Troubleshooting — No Backend Application

**Issue:** 

The test application has no actual backend service to receive the SAML response.

**Fix:** 

- The SAML assertion is successfully generated by Entra ID
- The authentication handshake completes successfully
- The failure occurs at the final delivery stage because no application is listening at the Reply URL

- `jwt.ms` is optimized for OIDC/JWT tokens 
- doesn't naturally decode or display the XML packets used by the SAML protocol.

[Token Validation](images/10-saml-token-page.png)

---

### Outcome

This phase demonstrates SAML-based SSO integration within a hybrid identity environment:

***Configuration:*** SAML application configured with correct identifiers and endpoints
***Access Control:*** Role-based assignment aligned with AD group structure
***Validation:*** Successful SAML assertion issuance confirmed through traffic inspection
***Troubleshooting:*** Clear distinction between authentication success and application-layer failure

---


## Phase 3 — Identity Governance & Risk Review

### Overview 

After implementing lifecycle automation and access controls, the next step was adding visibility into how identities behave after provisioning.

At this stage, users were being created, moved, and licensed correctly—but there was no mechanism to verify:

- whether accounts were still active
- whether privileged access was still justified
- whether access still aligned with the user’s role
- whether users had accumulated access over time

This phase focuses on building that visibility and gradually refining it into meaningful governance.

---

### Initial Script — Inactive User Audit

The starting point was a basic inactivity audit using `LastLogonDate`.

The script retrieved all users and calculated inactivity based on defined thresholds.
```
if ($ActualDays -ge $RiskThreshold) {
    "Risk: Account inactive for more than $RiskThreshold Days"
}
```
[Inactive Users Script](scripts/01-InactiveUser.ps1)

This provided immediate visibility into inactive accounts.

[Inactive User Audit](images/08-inactive-user-audit.png)


###  Limitation — No Access Context

While the script identified inactive users, it treated all accounts equally.

It did not distinguish between:

- standard users
- privileged accounts

and did not provide any guidance on what action should be taken.

At this stage, the output was informative, but not actionable.

---

### Improved Script — Identity Risk & Privileged Context

The script was extended to include risk classification and privileged access awareness.

A custom privileged group `IT_Admin_GG` was introduced and checked against each user’s group membership.

```
if ($PrivilegedGroups -contains $GroupName) {
    $GroupName
}

```
[Inactive and Privileged Users Script](scripts/02-inactive_privileged_User.ps1)

This allowed the script to differentiate between:

inactive standard accounts
inactive privileged accounts

and assign different risk levels and actions accordingly.

[Identity Risk Audit](images/09-inactive-privileged-user-audit.png)

***Implementation Note:***
Privileged access in this environment is determined by membership in the IT_Admin_GG group.
Inactivity is evaluated using LastLogonDate, which is sufficient for this lab but may not reflect real-time activity in production environments.

---

### Outcome

This grew the script from:

- simple inactivity tracking to identity risk evaluation.

At this stage, it was now possible to answer:

- which accounts are inactive
- which accounts are risky
- which accounts require immediate attention

---


### RBAC Drift Detection — Access Alignment

With identity risk visibility in place, the next step was validating whether users still had the correct access.

The RBAC drift script was built to compare:

- the user’s Department
- the user’s Title

against their group memberships.
```
if (-not ($Groups | Where-Object { $_ -like "*$Department*" })) {
    $AccessStatus = "Drift"
}
```
[RBAC Drift Script](scripts/03-rbacDrift.ps1)

If both values appeared in the group names, access was considered aligned.

[RBAC Drift](images/10-rbac-drift-audit.png)

--- 

### Limitation

At first glance, this appeared to work correctly.

Most users were returning:

  No Drift — RBAC aligned

However, this introduced another issue.

- Script only validated whether users had the correct access

- Did not check whether users had additional access beyond their role.

---

### Discovery — Permission Creep

This limitation became visible when reviewing specific users.

For example:

- `ppatel` was marked as No Drift in the RBAC output
- their department and role matched correctly
- access appeared valid

However, looking at it, the user still had membership in an additional group that was not aligned with their role.

This exposed a gap:

- The script confirmed correct access
- failed to detect excess access

---


### Final Enhancement — Access Drift Detection

To address this, the script was extended to identify unexpected group memberships.
```
$UnexpectedGroups = $Groups | Where-Object { $_ -notlike "*$Title*" }
```
[Access Drift Script](scipts/04-AccessDrift.ps1)

This introduced a third check:

1. Department alignment
2. Role alignment
3. Unexpected access

If additional groups were found, the user was flagged for review.

[Access Drift](images/11-access-drift-audit.png)


Now, Users previously marked as:

`No Drift` could now be identified as:

- Access Drift — unauthorized additional group found.

This refined the model now also prevents permission creep by going from:

“Does the user have the correct access?”

to:

“Does the user have only the access they need?”

---

### Outcome

This phase completes the transition from:

Identity Provisioning to Identity Governance

The final implementation provides visibility and produces the following into a report file:

- inactive accounts
- privileged risk exposure
- RBAC alignment
- permission creep

---


