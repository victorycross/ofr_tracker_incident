# Enterprise Incident Tracker — Phase 3 Entra ID Advanced Access Control Guide

**Version:** 1.0
**Date:** February 2026
**Audience:** EIT administrator, Entra ID / M365 platform administrator
**Classification:** Internal

---

## Overview

Phase 1 established EIT access control using static SharePoint groups (`EIT Members`, `EIT Viewers`, `EIT Owners`) and SharePoint item-level permissions for P&C incidents. Phase 3 hardens this model with three Entra ID-native capabilities:

1. **Dynamic security groups** — group membership driven by user attributes (department, job title, location), not manual management
2. **Privileged Identity Management (PIM)** — just-in-time, time-limited elevation for P&C incident access, replacing permanent membership in `EIT-PAC-Authorized`
3. **Conditional access policy** — enforce MFA and compliant-device requirements for EIT resources

These controls reduce administrative overhead, improve auditability, and limit the blast radius of compromised accounts.

---

## Section 1: Dynamic Security Groups

### 1.1 Why Dynamic Groups

Phase 1 groups (`EIT Members`, `EIT Viewers`) require manual membership updates when users join or leave the organisation, change roles, or move between domains. Dynamic groups automatically add and remove members based on Entra ID user attribute rules.

> **Licence requirement:** Dynamic group membership requires Entra ID P1 or P2 (included with M365 E3/E5). Verify your tenant licence before proceeding.

### 1.2 Dynamic Group — EIT Members

**Purpose:** All active employees in the five enterprise domains who need read-write access to the EIT tracker.

**Replace:** Static `EIT Members` group → dynamic `EIT Members (Dynamic)` group.

**Setup:**

1. Navigate to `https://entra.microsoft.com` → **Groups** → **+ New group**
2. Configure:
   - **Group type:** Security
   - **Group name:** `EIT Members (Dynamic)`
   - **Membership type:** Dynamic User
3. Click **Add dynamic query** and enter the rule. Example for department-based membership:
   ```
   (user.department -in ["National Security","NIS","Privacy & Technology","Human Capital","Physical Security"]) -and (user.accountEnabled -eq true)
   ```
   Adapt the department values to match your Entra ID user attribute schema. If a `department` attribute is not reliably populated, use `jobTitle`, a custom extension attribute, or a group-based rule.
4. Click **Validate rules** → add test users to verify the rule matches expected members
5. Click **Create**

> **Rule validation:** Entra ID evaluates dynamic rules within minutes to hours of attribute changes. Use the **Validate rules** → **Get validation results** feature to test rule accuracy before activating.

### 1.3 Dynamic Group — EIT Viewers

**Purpose:** Senior leaders and read-only stakeholders (not domain staff).

**Setup:** Same process as 1.2, with a rule based on job title or a custom "EIT Stakeholder" attribute:

```
(user.jobTitle -in ["Chief Risk Officer","General Counsel","Chief Information Security Officer","Director","Managing Director"]) -and (user.accountEnabled -eq true)
```

> **Alternative:** If leadership titles are not standardised in Entra ID, maintain `EIT Viewers` as a static group but automate membership via a provisioning workflow (e.g., Entra ID Lifecycle Workflows) triggered by job changes.

### 1.4 Updating SharePoint to Use Dynamic Groups

After creating dynamic groups:

1. Navigate to the EIT SharePoint site → **Site Settings** → **Site permissions**
2. Remove the old static groups from each permission level
3. Add the new dynamic groups:
   - `EIT Members (Dynamic)` → **Contribute** permission
   - `EIT Viewers (Dynamic)` → **Read** permission
   - `EIT Owners` (keep as static — a small set of admins)
4. Test access with a newly-eligible user (verify they gain access within 1 hour of group processing)

### 1.5 Dynamic Group — Domain-Specific Groups (Optional)

For domain leads who should only see incidents in their domain via SharePoint views or Power Apps context:

Create five groups:
- `EIT-NS-Members` — National Security domain staff
- `EIT-NIS-Members` — NIS domain staff
- `EIT-PT-Members` — Privacy & Technology domain staff
- `EIT-HC-Members` — Human Capital domain staff
- `EIT-PS-Members` — Physical Security domain staff

These can be used in Phase 3 Power Automate flows to automatically route domain-lead review tasks, or in future Power Apps releases to pre-filter the app to the user's domain.

---

## Section 2: Privileged Identity Management for P&C Access

### 2.1 Why PIM for P&C

Phase 1 uses a static `EIT-PAC-Authorized` group for P&C incident access. Permanent group membership means users retain P&C access indefinitely, even when they no longer have a business need. PIM replaces permanent access with:

- **Eligible assignment** — users can activate P&C access when needed, with justification
- **Time-limited activation** — access expires automatically (e.g., after 8 hours)
- **MFA requirement on activation** — user must re-authenticate at elevation time
- **Audit log** — every activation is recorded, including justification, approver (if required), and duration

> **Licence requirement:** Entra ID PIM requires Entra ID P2 (included with M365 E5; add-on for E3). Confirm licensing before proceeding.

### 2.2 Configure PIM for the EIT-PAC-Authorized Group

**Prerequisites:**
- `EIT-PAC-Authorized` must be a security group in Entra ID (not a Microsoft 365 group)
- The group must be enabled for PIM management

**Step 1: Enable PIM for the group**

1. Navigate to `https://entra.microsoft.com` → **Identity Governance** → **Privileged Identity Management** → **Groups**
2. Click **Discover groups**
3. Find `EIT-PAC-Authorized` → click **Manage**
4. The group is now PIM-managed. Any existing permanent members remain; new access will be granted as eligible assignments.

**Step 2: Configure role settings**

1. In PIM Groups → `EIT-PAC-Authorized` → **Settings**
2. Configure the **Member** role settings:
   - **Activation maximum duration:** 8 hours *(adjust to operational need — legal reviews may need full-day access)*
   - **On activation, require:** Multi-factor authentication + Justification
   - **Require approval to activate:** Yes (or No if the EIT admin is the approver and availability is a concern)
   - **Approver:** EIT administrator or Legal Counsel
   - **Require ticket information on activation:** Yes (link to incident being reviewed)
   - **Assignment expiration:** Set eligible assignments to expire after 12 months *(requires annual recertification)*

**Step 3: Convert permanent members to eligible**

For existing permanent members of `EIT-PAC-Authorized`:

1. In PIM Groups → `EIT-PAC-Authorized` → **Assignments**
2. For each existing permanent member:
   - Note their assignment
   - Remove permanent assignment
   - Add as **Eligible** assignment with 12-month expiry
3. Communicate the change to affected users — they must now activate access before viewing P&C incidents

> **Transition window:** Give users 1 week's notice and training on how to activate PIM access before removing permanent assignments. See Section 2.3 for the activation procedure.

**Step 4: Update Power Apps P&C gate**

The Phase 1 Power Apps P&C check uses:
```
Set(varUserHasPACAccess, Office365Groups.IsMemberOfGroup("[EIT-PAC-GROUP-ID]", User().Email).value)
```

With PIM, this check still works — `IsMemberOfGroup` returns true only when the user has activated their PIM membership (active assignment). Users who are only **eligible** (not yet activated) will return false. No change to the Power Apps formula is required.

### 2.3 P&C Access Activation — User Procedure

Share this procedure with all P&C eligible users:

**When you need to access P&C incidents:**

1. Navigate to `https://myaccess.microsoft.com` (or `https://entra.microsoft.com` → Identity Governance → My Access)
2. Click **Privileged access groups**
3. Find `EIT-PAC-Authorized` → click **Activate**
4. Complete MFA if prompted
5. Enter justification: e.g., `Reviewing EIT-142 for legal counsel session on 2026-02-24`
6. Enter ticket reference: the relevant EIT ItemID (e.g., `EIT-142`)
7. Set duration: hours needed (max as configured; default 8 hours)
8. Click **Activate**
9. If approval is required: the EIT administrator receives an approval request email. Activation completes once approved.
10. Open the EIT Power App — P&C incidents are now visible for the duration of your activation

**When your session ends:**
- Access expires automatically at the configured duration
- You can self-deactivate early via My Access → Active assignments → Deactivate

### 2.4 PIM Audit Log

All P&C access activations are logged in:
- Entra ID → **Identity Governance** → **Privileged Identity Management** → **Groups** → `EIT-PAC-Authorized` → **Audit history**

This log shows: who activated, when, justification provided, approver, duration, whether access was approved or denied. This log supports regulatory audit requirements and attorney-client privilege documentation.

---

## Section 3: Conditional Access Policy

### 3.1 Policy Design

The EIT conditional access policy enforces security requirements when users access EIT resources (SharePoint site, Power Apps, Power BI). Two policy tiers are recommended:

| Policy | Applies to | Conditions | Controls |
|--------|-----------|-----------|---------|
| **EIT Standard Access** | All EIT users | Any location, any device | Require MFA |
| **EIT P&C Access** | EIT-PAC-Authorized (when active) | Any location | Require MFA + Require compliant device |

> **Licence requirement:** Conditional access requires Entra ID P1 or P2.

### 3.2 Policy 1: EIT Standard Access (Require MFA)

1. Navigate to `https://entra.microsoft.com` → **Protection** → **Conditional Access** → **+ New policy**
2. **Name:** `EIT - Standard Access - Require MFA`
3. **Assignments:**
   - **Users:** Include groups: `EIT Members (Dynamic)`, `EIT Viewers (Dynamic)`
   - **Target resources:**
     - **Cloud apps:** Select → search `SharePoint` (Microsoft SharePoint Online) + `Power Apps` (Microsoft Power Apps) + `Power BI Service`
     - Or use **All cloud apps** with Exclude for non-EIT apps if simpler
4. **Conditions:**
   - **Locations:** Any location (or exclude named trusted locations if you want MFA only outside the office)
5. **Access controls → Grant:**
   - ✅ Require multi-factor authentication
   - Grant access
6. **Session:** No custom session controls required
7. **Enable policy:** On (after testing in Report-only mode first)

> **Report-only mode:** Set the policy to **Report only** for 1 week before enabling. Check the Conditional access insights workbook to confirm the policy would apply as expected before enforcing.

### 3.3 Policy 2: EIT P&C Access (Require Compliant Device)

For users accessing P&C incidents, require a compliant (Intune-managed) device in addition to MFA:

1. **Name:** `EIT - PAC Access - Require Compliant Device`
2. **Assignments:**
   - **Users:** Include group: `EIT-PAC-Authorized`
   - **Target resources:** SharePoint (EnterpriseIncidentTracker site) + Power Apps
3. **Conditions:** None (apply universally)
4. **Access controls → Grant:**
   - ✅ Require multi-factor authentication
   - ✅ Require device to be marked as compliant
   - Require all selected controls
5. **Enable policy:** On

> **Intune prerequisite:** The compliant device control requires Microsoft Intune device management. If Intune is not deployed, replace `Require compliant device` with `Require hybrid Azure AD joined device` (for domain-joined corporate devices) or remove this control and rely on MFA only.

### 3.4 Named Location — Trusted Corporate Networks (Optional)

If users should be able to access EIT without MFA from trusted corporate networks:

1. Navigate to **Conditional Access** → **Named locations** → **+ IP ranges location**
2. **Name:** `Corporate Network`
3. Add your corporate IP ranges (obtain from IT infrastructure team)
4. Check **Mark as trusted location**
5. In Policy 1 (Standard Access): under **Conditions** → **Locations** → **Exclude** → select `Corporate Network`

This allows MFA-free access from the office while requiring MFA from all other locations.

---

## Section 4: Entra ID Access Reviews

> Entra ID access reviews are covered in depth in `EIT-Phase3-Access-Review-Runbook.md`. This section summarises the Entra configuration.

### 4.1 Automated Access Review — EIT Members

1. Navigate to **Identity Governance** → **Access Reviews** → **+ New access review**
2. **Review type:** Teams + Groups
3. **Group:** `EIT Members (Dynamic)` *(or static `EIT Members` if not using dynamic)*
4. **Scope:** All users
5. **Reviewers:** Group owners (domain leads review their own team members), or specific users
6. **Duration:** 14 days
7. **Recurrence:** Quarterly
8. **Upon completion:** Auto-apply results — remove access for users whose access was denied

### 4.2 Access Review — EIT-PAC-Authorized

1. Same as 4.1, but for `EIT-PAC-Authorized` group
2. **Reviewers:** Legal Counsel + EIT administrator (dual review for P&C access)
3. **Recurrence:** Every 6 months *(more frequent for privileged access)*
4. Reviewers must confirm each user still has a legitimate business need for P&C access

---

## Configuration Checklist

| Item | Status | Notes |
|------|--------|-------|
| Dynamic group `EIT Members (Dynamic)` created | | Verify rule matches all domain staff |
| Dynamic group `EIT Viewers (Dynamic)` created | | Verify rule matches leadership stakeholders |
| SharePoint permissions updated to use dynamic groups | | Test with new user after group processing |
| PIM enabled for `EIT-PAC-Authorized` | | Requires Entra ID P2 |
| PIM role settings configured (8h max, MFA, justification) | | Confirm with legal counsel |
| Existing permanent P&C members converted to eligible | | Notify users 1 week before |
| Conditional access policy 1 created (Standard, MFA) | | Start in Report-only mode |
| Conditional access policy 2 created (PAC, compliant device) | | Start in Report-only mode |
| Access review for EIT Members configured (quarterly) | | |
| Access review for EIT-PAC-Authorized configured (6-monthly) | | |
| User training on PIM activation provided | | |
