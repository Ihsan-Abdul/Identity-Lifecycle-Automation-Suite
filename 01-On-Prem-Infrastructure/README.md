# Part 1 — On-Premises Infrastructure & Lifecycle

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

[View script here](scripts/1-NewOU-Basic.ps1)

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
` powershell
foreach ($User in $UserToCreate) {
    New-ADUser @UserParams
}`
This transformed the script from:

**Manual per-user creation**

into:

**Scalable bulk provisioning**

[View script here](scripts/2-NewOU-Iterative.ps1)

---

## Phase 3 — Joiner Automation (Reusable Provisioning)

### First Joiner Script

This was the transition from environment setup → lifecycle automation.

**Key additions:**
- User existence validation
- Try/Catch error handling
- Standardized user creation process

At this stage, the script was still partially static but introduced control logic.

[View script here](scripts/3a-static-Joiner.ps1)

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

**Implementation Note**
- Instead of hardcoding user data, this script uses a param() block to accept dynamic input at runtime, allowing the same script to be used for any department. Also includes validation to prevent duplicate account creation.

Powershell: 

``param(``

``[string]$Dept = "Marketing",``

``[string]$FirstName = "Elena",``

``[string]$LastName = "Mark",``

``[string]$UserID = "emark",``

``[string]$Title = "$($Dept) Staff"``

``)``

[View Joiner script here](scripts/3b-joiner.ps1)

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

[View role mover script](scripts/4a-role-mover.ps1)

### Department Mover

A second script handled cross-department moves, introducing:
- OU relocation
- Manager reassignment
- Role updates

This expanded the logic beyond simple role changes.

[View department mover](scripts/4b-department-mover.ps1)

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

**Implementation Note**
- To prevent Permission Creep, the Mover script identifies the user's old role-based group and removes it in the same operation. This ensures the user only has the access required for their current role.

Powershell: 

``Add-ADGroupMember -Identity $NewGroup -Members $UserID -ErrorAction Stop``

``Remove-ADGroupMember -Identity $OldGroup -Members $UserID -Confirm:$false -ErrorAction Stop``

``Write-Host "$UserId has from $OldGroup to $NewGroup" ``

[View comprehensive Mover script here](scripts/4c-mover.ps1)

---

## Phase 5 — Leaver Automation (Offboarding & Security)

The leaver script represents the most complete stage of the lifecycle.

### Key Capabilities

**1. Pre-check Validation**
- Detects if the user is already disabled or moved
- Prevents duplicate execution

**2. Security Scrubbing**
- Removes all group memberships except Domain Users
- `Get-ADPrincipalGroupMembership | Remove-ADGroupMember`
- This ensures a zero-access state

**3. Referential Integrity**
- Identifies users reporting to the departing employee
- Reassigns them to a new manager

**4. Account Decommissioning**
- Disables the account
- Moves it to a secured Disabled Users OU
- This avoids deletion while maintaining audit visibility

**Implementation Note**
- Before disabling a manager's account, the script identifies and reassigns all subordinates to prevent "orphaned" reporting lines:

Powershell: 

``$Subordinate = Get-ADUser -Filter "Manager -eq '$($User.DistinguishedName)'"  ``

``if($Subordinate){
$Subordinate | Set-ADUser -Manager $NewManager 
Write-Host "Re-assigned $($Subordinate.Count) to $NewManager"``

[View Leaver script here](scripts/5-Leaver.ps1)

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
**Execution and Results**

- To validate the automation suite a full lifecycle test was done in the lab environment. The screenshot demonstrates:

1. Joiner: Successful provisioning of `emark` into the Marketing OU.

2. Mover: A seamless transition of `emark` to the Finance department, including a role-based group swap.

3. Leaver: Secure offboarding of `asmith` including permission removal and relocation of the User object to the Disabled Users OU.

![Image of JML automated process](images/jml-script-screenshot.png)

---

