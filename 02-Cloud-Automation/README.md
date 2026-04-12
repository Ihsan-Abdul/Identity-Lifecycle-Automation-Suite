# Part 2 — Hybrid Cloud Integration (Entra ID)

## Technical Implementation

Following the on-premises setup, the next step was extending the local directory to the cloud.

#### Implementation of Microsoft Entra Connect

- Downloaded and installed the Microsoft Entra Connect Sync Agent on the Windows Server 2022 Domain Controller.
- Configured Domain and OU filtering to target the `Corporate` OU specifically.
- This ensures only laboratory users and groups are synchronized, maintaining a clean cloud tenant.

#### Implementation Note
> By selecting "Sync selected domains and OUs," I prevented built-in system accounts and local infrastructure groups from cluttering the Entra ID portal, adhering to standard production best practices.

[View Entra Connect Download](images/cloud-01-entra-connect-download.png)

[View Domain/OU Filtering configuration](images/cloud-02-sync-ou.png)

---

#### Synchronization Troubleshooting (Time Skew)

The first synchronization attempt failed to populate users in the Entra ID portal.

**Issue Identified:**
- The local VM system clock had drifted from the actual time.
- Entra ID authentication tokens require precise time synchronization; the mismatch caused the sync agent to fail authentication with the cloud service.

**Resolution:**
- Reconfigured the Windows Time service to sync from an external NTP source (time.windows.com) instead of the inaccurate Local CMOS clock:

```powershell:

w32tm /config /manualpeerlist:"time.windows.com,0x8" /syncfromflags:manual /update
Restart-Service W32Time
w32tm /resync /force
```

**Key Learning**
- In a hybrid environment, time synchronization is a critical dependency. Even a small drift can invalidate security tokens and break the identity pipeline between on-premises and cloud.

---

***Verification of Cloud Identities***

Once the time sync was resolved, the synchronization cycle completed successfully.

**Results:**
- Verified 18 users found within the Entra ID **All Users** blade.
- Confirmed identities are marked as **Synced from on-premises**, maintaining the local AD as the Source of Authority.
- Identities are created in an **Unlicensed** state, requiring a secondary automation step for service activation.

[View 18 users synchronized in Entra ID](images/cloud-03-synced-users.png)

---

## Phase 1 - Conditional Access: MFA Enforcement

With identities successfully synchronized, the next objective was to secure the cloud tenant. In a hybrid model, passwords alone are insufficient; multi-factor authentication (MFA) is required to protect against credential-based attacks.

**Conditional Access: MFA Enforcement**
- I implemented a Conditional Access Policy to mandate MFA across the entire organization. This ensures that every sign-in attempt to any cloud application must be verified by a secondary factor.

**Implementation Details:**
- Policy Name: "Require MFA - All Users"
- Assignments: Targeted all users within the tenant while explicitly excluding a break-glass administrative account to prevent lockout.
- Target Resources: Applied to "All cloud apps" to ensure no security gaps in the authentication perimeter.
- Grant Control: Set to "Require multi-factor authentication."

**Implementation Note**
- This policy aligns with Zero Trust principles by "verifying explicitly." By applying it to all cloud applications, I established a consistent security baseline that prevents attackers from targeting less-protected legacy apps to gain a foothold.

[View MFA Policy Configuration](images/cloud-04-mfa-policy.png)


**Verification of MFA Policy**
- To validate the configuration, I performed a sign-in test with a synchronized laboratory user.

**Results:**
- Confirmed that the policy correctly interrupts the standard login flow.
- Verified that users are prompted to register for Microsoft Authenticator, preventing access until a secondary identity factor is established.
- This transition successfully moved the tenant from a "Password-Only" state to a secure, modern authentication model.

[View MFA Enrollment Prompt](images/cloud-05-mfa-verification.png)

---

## Phase 2 — Zero Trust Enforcement (Intune & Conditional Access)

---

## Phase 3 — Cloud Lifecycle Automation (Microsoft Graph)

This phase extends the Joiner lifecycle into the cloud, evolving from manual provisioning to a scalable automation model within Microsoft Entra ID.

### Remote Management Infrastructure (Graph Connection)

Before any provisioning could occur, a secure remote bridge was established to the cloud tenant. This represents the shift from local AD management to API-driven cloud administration.

**Implementation**

- **Command:** `Connect-MgGraph -Scopes "User.ReadWrite.All", "Organization.Read.All" -ContextScope Process -UseDeviceAuthentication`
- **Access Model:** Delegated permissions with Least Privilege
- **Security:** Scopes were restricted specifically to User and Directory modifications to ensure a hardened management session

**Key Insight:** Cloud identity management is purely API-driven. Establishing a scoped, authenticated session is the first security boundary in cloud automation.

![Remote Connection](images/RemoteConnection-Mgraph.png)
![Scope Consent](images/Mgraph-permission.jpg)

---

### Terminal Discovery: Inline Licensing & Attribute Friction

Following synchronization, 18 users existed in Entra ID in an unlicensed state. I initially attempted a manual license assignment directly inline in the terminal to establish a baseline.

**Problem**

Initial assignment failed with a `400 BadRequest`.

**Root Cause & Inline Validation**

- **Discovery:** Investigation revealed that Microsoft 365 requires the `UsageLocation` attribute (ISO Country Code) to be defined before a license seat can be legally assigned
- **Manual Fix:** I tested the remediation inline for a single user:
  - Updated attribute: `Update-MgUser -UserId <ID> -UsageLocation "CA"`
  - Re-ran license: `Set-MgUserLicense -UserId <ID> -AddLicenses @{SkuId = $LicenseID}`
- **Result:** Assignment succeeded. This exposed a critical provisioning dependency: **Identity → Attribute Remediation → License Assignment → Service Access**

![Inline Terminal Fix](images/firstLicense-Location.png)

---

### Tactical Scripting (cloud-joiner.ps1)

Once the manual fix was validated, I moved the logic from the terminal into a reusable script to ensure the process was repeatable and included error handling.

**Technical Improvements:**
- **Logic Grouping:** Combined the attribute update and licensing into a single try/catch block
- **Parametrization:** Used `Read-Host` to allow for targeted user provisioning

**The Engineering Bottleneck:**
While functional, this script relied on manual triggers. I identified that asking for a user is not true automation; it is a digital workaround for a manual task that cannot scale for 18 users or an enterprise workforce.

![Tactical Script Execution](images/license-script-proof.png)

---

### Enterprise Scaling: Provisioning Engine (cloud.joiner.loop.ps1)

The final iteration evolved into an autonomous engine that removes the human from the loop by programmatically identifying the workload.

**Key Features:**

- **Automated Discovery:** The script queries the tenant for all users where `assignedLicenses/count -eq 0`, identifying "Staged" users without human input
- **Bulk Remediation:** Programmatically applies the UsageLocation and license across the entire list in a single execution
- **Operational Resiliency:** The foreach loop ensures that a failure on one account (e.g., a naming conflict) does not halt the entire provisioning batch

**PowerShell Logic:**

```powershell
foreach ($User in $TargetUsers) {
    try {
        Update-MgUser -UserID $User.Id -UsageLocation "CA"
        Set-MgUserLicense -UserID $User.Id -AddLicenses @{SkuId = $LicenseID}
    } catch { 
        Write-Host "Failed to license user: $($User.UserprincipalName)" 
    }
}
```

This represents a shift from:

**Single-purpose terminal commands**

to:

**Modular, autonomous lifecycle automation**

![Bulk Engine Execution](images/license-loop-proof.png)

---

### Execution Outcome

A full provisioning cycle was executed across the 18 synchronized laboratory identities.

**Results:**

- **Provisioning Success:** 100% transition from Unlicensed to Active
- **Attribute Integrity:** Confirmed UsageLocation (CA) applied consistently across the tenant
- **Service Readiness:** Enabled immediate access to Exchange Online and SharePoint Online for the incoming workforce

![Initial Staged State](images/00-initial-license.png)
![Final Provisioned State](images/03-license-loop.png)
