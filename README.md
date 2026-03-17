# Identity Lifecycle Automation Lab
## PowerShell-Based Joiner / Mover / Leaver (JLM) Suite

### Project Overview
This project demonstrates the development of an Identity Lifecycle Management (JLM) automation framework using PowerShell and Active Directory.

Rather than starting with a complete solution, this project was built iteratively—from basic scripting to a structured automation suite—reflecting how real-world IAM environments evolve over time.

The focus was to:
- Implement RBAC using AGDLP design
- Automate Joiner, Mover, and Leaver processes
- Reduce manual errors and permission creep
- Maintain referential integrity (manager/subordinate relationships)

All scripts have been tested in an isolated lab environment with screenshot documentation available.

---

## 📍 Table of Contents
- [Technical Evolution](#technical-evolution)
  - [Phase 1 — Foundational Scripting](#phase-1--foundational-scripting-manual--hardcoded-automation)
  - [Phase 2 — Structured Automation](#phase-2--structured-automation-betternewou)
  - [Phase 3 — Joiner Automation](#phase-3--joiner-automation-reusable-provisioning)
  - [Phase 4 — Mover Automation](#phase-4--mover-automation-access-control--permission-integrity)
  - [Phase 5 — Leaver Automation](#phase-5--leaver-automation-offboarding--security)
- [Troubleshooting Case Study](#troubleshooting-case-study)
- [Technical Skills Demonstrated](#technical-skills-demonstrated)
- [Project Outcome](#project-outcome)
- [Next Phase](#next-phase)

---

## Technical Evolution

### Phase 1 — Foundational Scripting (Manual → Hardcoded Automation)

**Initial Script: OU, Groups, and Users (NewOU)**

The first script created:
- Department OU structure
- Global and Domain Local groups
- Group nesting (RBAC model)
- Users with assigned roles

This version was fully hardcoded, including:
- OU paths
- User attributes
- Group names

While functional, it had several limitations:
- Not reusable across departments
- Required manual updates for every change
- Repeated commands for each user

---

### Phase 2 — Structured Automation (BetterNewOU)

The second iteration focused on reducing repetition and improving scalability.

**Improvements introduced:**
- Centralized environment variables (Domain, OU paths)
- Dynamic naming using `$Dept`
- Use of arrays to store user data
- Use of `foreach` loops for bulk user creation
- Continued use of splatting for readability

**Example improvement:**
 powershell
foreach ($User in $UserToCreate) {
    New-ADUser @UserParams
}
This transformed the script from:

**Manual per-user creation**

into:

**Scalable bulk provisioning**

---

## Phase 3 — Joiner Automation (Reusable Provisioning)

### First Joiner Script

This was the transition from environment setup → lifecycle automation.

**Key additions:**
- User existence validation
- Try/Catch error handling
- Standardized user creation process

At this stage, the script was still partially static but introduced control logic.

### Parameterized Joiner Script

The joiner script was redesigned to support dynamic input using a `param()` block.

**Key improvements:**
- Accepts user input at runtime:
  - Name
  - UserID
  - Department
  - Role
- Removes need to modify script code
- Enforces consistent provisioning logic

This marked the shift to:

**Reusable automation for help desk or IT operations**

---

## Phase 4 — Mover Automation (Access Control & Permission Integrity)

### Problem

User transitions (role or department changes) are a primary cause of permission creep.

### Role-Based Mover

The first mover script handled role changes within a department.

**Key logic:**
- Check if user already has the target role
- Update user attributes
- Perform a group membership swap
  - Remove old role group
  - Add new role group

This ensured RBAC enforcement during promotions.

### Department Mover

A second script handled cross-department moves, introducing:
- OU relocation
- Manager reassignment
- Role updates

This expanded the logic beyond simple role changes.

### Master Mover Script (Consolidation)

The final mover script combined both scenarios into a single parameter-driven tool.

**Key features:**
- Supports:
  - Role changes
  - Department transfers
- Uses conditional logic to handle optional inputs:
  - `ReportsToID`
  - `ReportsFromID`
- Updates:
  - Title
  - Department
  - Manager relationships
  - OU location
- Maintains referential integrity for reporting structures
- Enforces clean access transitions:
  - Remove old group
  - Add new group

This represents a shift from:

**Single-purpose scripts**

to:

**Modular lifecycle automation**

---

## Phase 5 — Leaver Automation (Offboarding & Security)

The leaver script represents the most complete stage of the lifecycle.

### Key Capabilities

**1. Pre-check Validation**
- Detects if the user is already disabled or moved
- Prevents duplicate execution

**2. Security Scrubbing**
- Removes all group memberships except Domain Users
 'Get-ADPrincipalGroupMembership | Remove-ADGroupMember'
- This ensures a zero-access state

**3. Referential Integrity**
- Identifies users reporting to the departing employee
- Reassigns them to a new manager

**4. Account Decommissioning**
- Disables the account
- Moves it to a secured Disabled Users OU
- This avoids deletion while maintaining audit visibility

---

## Troubleshooting Case Study

**Issue**

During development, an error occurred when reassigning subordinates after moving a user.

**Root Cause**

Moving the user object changed its Distinguished Name (DN) immediately, making the stored reference invalid.

**Resolution**

The execution order was corrected:
1. Reassign subordinates
2. Remove access
3. Disable account
4. Move object

**Key Learning**
- Active Directory attributes update in real time
- Execution order is critical in automation workflows

---

## Technical Skills Demonstrated

**Active Directory & IAM**
- OU design and organization
- RBAC implementation (AGDLP)
- Group-based access control
- Identity lifecycle management (JLM)

**PowerShell**
- Parameterized scripting (`param`)
- Splatting
- Try/Catch error handling
- Pipeline processing
- Arrays and loops (`foreach`)
- Conditional logic

**Security & Governance**
- Permission creep prevention
- Referential integrity management
- Secure offboarding practices
- Least privilege enforcement

**Testing & Documentation**
- Isolated lab environment testing
- Screenshot documentation of script execution
- Troubleshooting methodology documentation

---

## Project Outcome

This project demonstrates the transition from:

**Manual AD administration**

to:

**Automated identity lifecycle management**

It reflects real IAM responsibilities, including:
- Standardized onboarding
- Controlled access transitions
- Secure and auditable offboarding

---

## Next Phase

**Hybrid Identity Integration**
- Extending on-premises automation to Microsoft Entra ID
- Implementing Azure AD Connect sync logic
- PowerShell-based cloud license assignment
- Cloud-only group management
