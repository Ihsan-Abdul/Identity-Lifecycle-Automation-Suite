# Hybrid Identity & Governance Framework

## End-to-End JML Automation with Active Directory and Microsoft Entra ID

### Project Overview
This project documents the process of building a hybrid identity environment from the ground up, starting with on-premises Active Directory and extending into Microsoft Entra ID.

The goal was to understand how identity works across systems—not just creating users, but managing their lifecycle, controlling access, and enforcing security as they move through an organization.

Instead of building a complete system upfront, the project was developed step-by-step:

- starting with manual AD setup
- moving into PowerShell automation
- extending identities into the cloud
- then adding access control and security

Each phase reflects a limitation that had to be identified and solved before moving forward.

## Engineering Approach & Evolution

This system was not designed fully in advance. It evolved through iterative improvements across both on-prem and cloud environments, with each stage building on the last.

## On-Prem Foundation

The project began with building a structured Active Directory environment:

- Created OU hierarchy for departments
- Implemented RBAC using AGDLP
- Assigned access through groups instead of direct permissions

At this stage, initial scripts worked but were repetitive, hardcoded, and not reusable.

## Lifecycle Automation (JML)

To move beyond static setup, the next step was handling real identity changes:

Joiner → creating new users
Mover → updating roles and departments
Leaver → removing access and disabling accounts


## Improvement — Structured Automation

To support these lifecycle changes at scale, scripts were rewritten to:

- use variables and arrays
- loop through user data
- standardize provisioning logic

This marked the transition from one-off scripts to reusable automation and led into full ***Joiner / Mover / Leaver (JML) automation.***


## Identity Integrity Challenges

As lifecycle automation expanded, issues began to appear:

- Moving users changed their Distinguished Name (DN)
- Scripts depending on the old DN broke
- Manager relationships could become invalid

### Fix — Execution Order & Data Handling

To resolve this:

- reordered operations (update relationships before moving objects)
- added validation checks
- ensured scripts did not rely on outdated references

This was the point where the system had to be designed carefully, not just scripted.

---
*Full in depth On-Premise creation and automation found here: [Part 1: On-Prem Infrastructure](01-On-Prem-Infrastructure/README.md)*

---

## Cloud Integration (Entra ID)

After stabilizing the on-prem side, identities were synced into Microsoft Entra ID.

At this point:

- users existed in the cloud
- but had no access to services

An initial attempt to assign licenses failed with a 400 BadRequest.


## Root Cause

Cloud provisioning required  `UsageLocation`. Without it the license assignment fails.

### Fix — Attribute Before Access

Testing showed that:

`Update-MgUser -UsageLocation "CA"` had to be applied before assigning a license.

This introduced the key concept of:

Identity → Attributes → License → Service Access


## Moving from Manual to Automation (Cloud)

Initially, the fix was applied manually. 
Then it evolved into:
- a script that combined attribute update + license assignment
-  structured  error handling for reliability

### Automation Progression

Provisioning evolved through three stages:

- ***manual fixes:*** required direct intervention.
- ***single-user scripts:*** reduced repition but stil manual.
- ***loop-based automation engine:*** automatically find all unlicensed users, loop through them, apply fixes and assign licenses.

This transitioned the workflow from manual provisioning to automated lifecycle provisioning.

---
*Full in depth Cloud intergration and provisioning here: [Part 2: Hybrid Cloud Integration](02-Cloud-Integration/README.md)*

---

## Access Control & Security

Once users had access, control became the focus.

- Implemented Conditional Access in Microsoft Entra ID
- Enforced MFA across all users
- Applied stricter controls to SharePoint

### Device-Based Access (Intune)

Using Microsoft Intune:

- Defined device compliance policies
- Required compliant devices for accessing SharePoint

## Security & Access Model

This project applies layered access control rather than relying on identity alone.

### RBAC (Role-Based Access Control)

- Access assigned through groups
- No direct user-to-resource permissions
- Role changes automatically update access

### Least Privilege

- Users receive only required access
- Permissions are adjusted during lifecycle changes
- Prevents permission creep

### Zero Trust Access Controls

Access decisions require more than valid credentials.
- MFA enforced through Conditional Access
- Device compliance enforced through Intune
- Application specific policies applied to sensitive resources

Access is granted only after verifying:
- User Identity + Device State + Access Context

---
*Full access control and device security here: [Part 3: Governance & Security](03-Governance-Compliance/README.md)*

---

## Identity Governance

To maintain visibility over access:

- built scripts to review user permissions
- identified inactive accounts
- ensured access aligned with current roles

| Environment | Components|
|-------------|-----------|
| On-Premises |	Windows Server 2022, Active Directory, PowerShell |
| Cloud	| Microsoft Entra ID, Microsoft Graph |
| Access Control	| Conditional Access, MFA |
| Device Security |	Microsoft Intune, Compliance Policies |


## Project Structure
Each section of the project focuses on a specific stage of the identity lifecycle:

[Part 1: On-Prem Infrastructure](01-On-Prem-Infrastructure/README.md): AD build, RBAC (AGDLP), JML automation 
[Part 2: Hybrid Cloud Integration](02-Cloud-Integration/README.md): Entra ID sync, Microsoft Graph, provisioning automation 
[Part 3: Governance & Security](03-Governance-Compliance/README.md): Conditional Access, MFA, Intune, device compliance 


## What This Project Demonstrates

This project shows the progression from:

`manual user management`

to:

`automated identity lifecycle with controlled access across hybrid systems`

It reflects how identity systems are actually built:

- starting simple
- encountering limitations
- fixing issues
- and improving toward automation and control

---

*Note: All scripts have been tested in an isolated lab environment with screenshot documentation available.*

---
