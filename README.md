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

This system was not designed fully in advance. It evolved through iterative improvements across both on-prem and cloud environments.


## On-Prem Foundation

- Built structured OU hierarchy
- Implemented RBAC using AGDLP
- Assigned access through groups instead of direct permissions

Initial scripts worked but were repetitive, hardcoded, and not reusable.

## Lifecycle Automation (JML)

The next step was handling real identity changes:

Joiner → creating new users
Mover → updating roles and departments
Leaver → removing access and disabling accounts


## Improvement — Structured Automation

Scripts were rewritten to use:

- variables and arrays
- loop through user data
- standardize how users and groups are created

This evolved into full ***Joiner/Mover/Leaver (JML) Automation.***


## Identity Integrity Challenges

During lifecycle changes issues appeared:

- Moving users changed their Distinguished Name (DN)
- Scripts that depended on the old DN broke
- Manager relationships could become invalid

## Fix — Execution Order & Data Handling

To resolve this:

- reordered operations (update relationships before moving objects)
- added validation checks
- ensured scripts didn’t rely on outdated references

This was the first point where the system had to be thought through, and not just scripted.

---
*Full in depth On-Premise creation and automation found here: [Part 1: On-Prem Infrastructure](01-On-Prem-Infrastructure/README.md)*

---

## Cloud Integration (Entra ID)

After stabilizing the on-prem side, identities were synced into Microsoft Entra ID.

At this point:

- users existed in the cloud but had no access to services
- An initial attempt to assign licenses failed with a 400 BadRequest.


### Root Cause

Cloud provisioning required  `UsageLocation`. Without it the license assignment fails.

### Fix — Attribute Before Access

Testing showed that:

`Update-MgUser -UsageLocation "CA"` had to be applied before assigning a license.

Identity → Attributes → License → Service Access


## Moving from Manual to Automation (Cloud)

At first the fix was applied manually. Then, into a script that combined attribute update + license assignment and added error handling.

### Automation Progression

Provisioning evolved through stages:

- manual terminal fixes: depended on manual execution.
- single-user scripts: required selecting users one at a time.
- loop-based automation engine: find all unlicensed users, loop through them automatically and apply fixes and assign licenses.

This changed the workflow from manual provisioning to automated lifecycle provisioning.

---
***View full in depth Cloud intergration and provisioning here: [Part 2: Hybrid Cloud Integration](02-Cloud-Integration/README.md)***

---

## Access Control & Security

Once users had access, control became the focus.

- Implemented Conditional Access in Microsoft Entra ID
- Enforced MFA across all users
- Applied stricter controls to SharePoint

## Device-Based Access (Intune)

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
- Application-specific policies applied to sensitive resources

Access is granted only after verifying:
User Identity + Device State + Access Context

---
***View full device and access controls here: [Part 3: Governance & Security](03-Governance-Compliance/README.md)***

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
