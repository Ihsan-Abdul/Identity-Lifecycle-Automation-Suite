# Identity-Lifecycle-Automation-Suite

# PowerShell-Based User Management for Regional Enterprise Infrastructure

This project documents the development of a modular, error-resilient automation suite designed to manage the full Joiner-Mover-Leaver (JLM) lifecycle within an Active Directory environment. Developed with a focus on regional business operations in Windsor, ON, this suite demonstrates a progression from manual task execution to advanced, logic-driven automation.

## Technical Evolution

### 1. Provisioning & Initial Joiner (Phase: Standardization)
**The Problem:** Inconsistent user creation and "copy-paste" errors from manual GUI entry.

**The Solution:** Transitioned from manual entry to a functional Marketing User Script.

**Key Features:** Initial implementation of `New-ADUser` with hardcoded attributes to ensure same standards for all new hires.

**Takeaway:** Established the baseline for attribute consistency and naming conventions.

### 2. Comprehensive Mover (Phase: Modular Design)
**The Problem:** Hardcoded scripts cannot scale for different movement types (Promotions vs. Departmental Transfers).

**The Solution:** A robust, Param-based Mover Script that handles multiple scenarios.

**Key Features:**
- Utilizes Mandatory Parameters for `UserID`, `NewDept`, `NewGroup`, and `OldGroup`
- Introduction of Variable Interpolation for dynamic Title generation (e.g., `$($Dept) Staff`)
- Enforced "Clean-Swap" logic for security groups to prevent permissions creep

**Takeaway:** Mastery of user input handling and conditional logic to manage internal transitions.

### 3. Advanced Leaver (Phase: Robust Automation)
**The Problem:** Manual offboarding leaves security gaps (orphaned accounts, active memberships, and broken reporting lines).

**The Solution:** An automated, global cleanup script that serves as the "Cap-Stone" of the suite.

**Key Features:**
- **Recursive Cleanup:** Automatically identifies and strips all group memberships via the pipeline
- **Orphaned Account Prevention:** Queries for direct reports before account disabling to re-assign subordinates to a new manager
- **Lifecycle Compliance:** Disables accounts, updates descriptions with timestamps, and migrates objects to a dedicated "Disabled Users" OU for auditing

**Takeaway:** Demonstrated high-level PowerShell proficiency including Error Handling (`Try/Catch`), Splatting, and Pipeline-based bulk modifications.

## Technical Skills Demonstrated
- **Scripting:** PowerShell 5.1/7.x, Active Directory Module
- **Logic:** Parameter Validation, Sub-expressions, Variable Interpolation
- **Security:** Principle of Least Privilege (Group Swapping), Identity Governance
- **Troubleshooting:** `ErrorAction Stop`, Return-on-Failure logic

## Future Roadmap
- **CSV Integration:** Transitioning from manual parameters to bulk processing for large-scale hire events
- **Hybrid Sync:** Extending logic to support Entra ID (Azure AD) synchronization for cloud-based resources
- **Automated Logging:** Implementing transcript logging for all lifecycle events to support compliance audits
