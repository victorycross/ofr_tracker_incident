# Enterprise Incident Tracker — Phase 3 Test Plan

**Version:** 1.0
**Date:** February 2026
**Classification:** Internal

---

## Overview

This test plan covers acceptance testing for the three Phase 3 workstreams:

1. Entra ID Advanced Access Control (dynamic groups, PIM, conditional access)
2. Compliance & Audit Reporting (compliance report flow, Power BI compliance page)
3. Operational Resilience (operations runbook procedures, disaster recovery)

**Prerequisites:** Phase 1 and Phase 2 must pass their respective go/no-go criteria before executing Phase 3 tests. Phase 3a requires Entra ID P1 or P2 licensing.

---

## Category 1: Dynamic Security Groups

### TC-DG-01: Dynamic Group Includes New Eligible User

**Priority:** High

**Precondition:**
- `EIT Members (Dynamic)` group created with a department-based membership rule
- A test user account exists that does NOT currently match the dynamic rule

**Steps:**
1. Note the current member count of `EIT Members (Dynamic)` in Entra ID
2. Update the test user's `department` attribute in Entra ID to match the dynamic rule (e.g., set to `NIS`)
3. Wait up to 30 minutes for dynamic group processing
4. Check group membership

**Expected Result:**
- Test user now appears in `EIT Members (Dynamic)` member list
- Test user can access the EIT SharePoint site with Contribute permission
- Test user can open the EIT Power App successfully

---

### TC-DG-02: Dynamic Group Removes User When Attribute Changes

**Priority:** High

**Precondition:** TC-DG-01 completed — test user is a member of `EIT Members (Dynamic)`

**Steps:**
1. Update the test user's `department` attribute to a value that does NOT match the dynamic rule
2. Wait up to 30 minutes
3. Check group membership and SharePoint access

**Expected Result:**
- Test user removed from `EIT Members (Dynamic)`
- Test user can no longer access the EIT SharePoint site (receives "Access denied")
- Test user can no longer open the EIT Power App

---

### TC-DG-03: EIT Owners Remain Unaffected by Dynamic Group Changes

**Priority:** Medium

**Steps:**
1. Confirm `EIT Owners` is a static group (not dynamic)
2. Update the EIT admin test account's department attribute to a value outside the dynamic rule
3. Verify the EIT admin still has Full Control access to the EIT site

**Expected Result:**
- EIT admin retains access (they are in the static `EIT Owners` group, not dependent on the dynamic rule)

---

## Category 2: Privileged Identity Management (PIM)

### TC-PIM-01: Eligible User Cannot Access P&C Incidents Without Activation

**Priority:** Critical

**Precondition:**
- PIM enabled for `EIT-PAC-Authorized`
- A test user has an **eligible** (not permanent) assignment
- At least 1 P&C incident exists in EIT_Incidents

**Steps:**
1. Sign in to EIT Power App as the test user
2. Observe the Dashboard — check the P&C Active KPI card
3. Navigate to TrackerScreen — check whether P&C incidents are visible

**Expected Result:**
- P&C Active count = 0 (user does not have active PIM access)
- P&C incidents are not visible in the tracker (filtered by `varUserHasPACAccess = false`)
- No P&C incident rows appear in any gallery

---

### TC-PIM-02: Eligible User Activates PIM and Gains P&C Access

**Priority:** Critical

**Precondition:** TC-PIM-01 — test user has eligible but inactive PIM assignment

**Steps:**
1. Test user navigates to `https://myaccess.microsoft.com` → Privileged access groups
2. Finds `EIT-PAC-Authorized` → clicks **Activate**
3. Completes MFA
4. Enters justification: `Testing PIM access for EIT Phase 3 acceptance test`
5. Sets duration: 2 hours
6. Clicks **Activate** (and approves if approval required)
7. Opens or refreshes the EIT Power App

**Expected Result:**
- `varUserHasPACAccess = true` (verified by P&C Active count > 0 on Dashboard if P&C incidents exist)
- P&C incidents are visible in the TrackerScreen (with amber P&C banner when opened)
- EIT functions normally with P&C access active

---

### TC-PIM-03: PIM Access Expires Automatically

**Priority:** High

**Precondition:** TC-PIM-02 — test user has active PIM access set to expire in 2 hours

**Steps:**
1. Wait for the 2-hour activation to expire (or manually deactivate via My Access → Active assignments → Deactivate)
2. Refresh the EIT Power App (close and reopen, or navigate to a new screen)
3. Check P&C incident visibility

**Expected Result:**
- After expiry/deactivation: P&C incidents no longer visible
- P&C Active count returns to 0
- No error message — the app silently reverts to the non-P&C view

---

### TC-PIM-04: PIM Activation Appears in Audit Log

**Priority:** High

**Precondition:** TC-PIM-02 completed

**Steps:**
1. Navigate to Entra ID → Identity Governance → PIM → Groups → `EIT-PAC-Authorized` → Audit history
2. Find the activation record from TC-PIM-02

**Expected Result:**
- Activation record shows: user identity, timestamp, justification text, duration, activation status (approved/completed)
- The audit record is complete and accurate

---

## Category 3: Conditional Access

### TC-CA-01: EIT User Prompted for MFA on First Access

**Priority:** High

**Precondition:**
- Conditional access policy `EIT - Standard Access - Require MFA` is enabled (not Report-only)
- A test user account that has not recently performed MFA

**Steps:**
1. Sign in to the EIT SharePoint site (`https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker`) in a private/incognito browser as the test user
2. Observe the sign-in flow

**Expected Result:**
- After entering credentials, the user is prompted to complete MFA before accessing the site
- After completing MFA: access to the site is granted

---

### TC-CA-02: Conditional Access Policy in Report-Only Generates Expected Logs

**Priority:** Medium

**Precondition:** Policy in Report-only mode (pre-enforcement verification)

**Steps:**
1. Navigate to Entra ID → Conditional Access → Insights and reporting
2. Review the sign-in events for EIT users over the past 7 days
3. Check the Report-only results for the EIT policy

**Expected Result:**
- Policy would have applied to EIT user sign-ins
- All EIT users would have been prompted for MFA under this policy
- No unexpected exclusions (e.g., admin accounts appearing as not-targeted when they should be)

---

## Category 4: Quarterly Access Review Flow

### TC-AR-FLOW-01: Review Reminder Emails Sent to All Domain Leads

**Priority:** High

**Precondition:**
- `EIT Quarterly Access Review Reminder` flow is created and connected
- All 5 `EIT_DomainContacts` rows have valid `LeadEmail` values

**Steps:**
1. Trigger the flow manually (Test → Manually)
2. Wait for completion
3. Check each domain lead's inbox (or use test email addresses in `EIT_DomainContacts`)

**Expected Result:**
- 5 reminder emails received, one per domain
- Each email subject includes the domain name
- Admin summary email received at `david@brightpathtechnology.io`
- Deadline date in email body is approximately 14 days from today

---

### TC-AR-FLOW-02: Admin Summary Contains All Domains

**Priority:** Medium

**Precondition:** TC-AR-FLOW-01

**Steps:**
1. Inspect the admin summary email content

**Expected Result:**
- Email lists all 5 domains
- Domain lead names and emails are correctly sourced from `EIT_DomainContacts`
- Admin action checklist is present in the email body

---

## Category 5: Compliance Report Flow

### TC-CR-01: Compliance Report Email Generated on Schedule

**Priority:** High

**Precondition:**
- `EIT Monthly Compliance Report` flow is created and connected
- Test incidents exist with `DateRaised` in the previous month

**Steps:**
1. Trigger the flow manually
2. Check `david@brightpathtechnology.io` inbox

**Expected Result:**
- Email received with subject `EIT Monthly Compliance Report — [Month Year]`
- Report body contains all 8 metric rows with numeric values
- Report month label matches the previous calendar month

---

### TC-CR-02: Incident Counts Match SharePoint Source

**Priority:** Critical

**Steps:**
1. Before running the flow: manually count in SharePoint:
   - Items in `EIT_Incidents` where `DateRaised` is in the previous month (total raised)
   - Items where `Status = Closed` and `LastUpdated` in previous month (resolved)
2. Run the compliance report flow
3. Compare manual counts against report values

**Expected Result:**
- Flow counts match the manual counts (±0)

---

### TC-CR-03: MTTC Calculates Correctly

**Priority:** High

**Precondition:** At least 2 incidents exist with both `DateRaised` and `DateContained` in the previous month

**Steps:**
1. Manually calculate MTTC for these incidents: `(DateContained - DateRaised)` in days, averaged
2. Run the compliance report flow
3. Compare MTTC in the report

**Expected Result:**
- Flow MTTC matches manual calculation (±1 day for rounding)

---

### TC-CR-04: Report Saved to SharePoint Compliance Library

**Priority:** Medium

**Precondition:** SharePoint Compliance Reports library exists; "Save report to SharePoint" step is enabled in the flow

**Steps:**
1. Run the flow
2. Navigate to `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker/Compliance Reports`

**Expected Result:**
- New `.txt` file exists with naming convention `EIT-Compliance-[YYYY-MM].txt`
- File content matches the email body

---

## Category 6: Power BI Compliance Page

### TC-PBI-C-01: Compliance Page Loads with All Visuals

**Priority:** High

**Steps:**
1. Open the EIT Power BI dashboard
2. Navigate to the `Compliance & Audit` page (Page 6)

**Expected Result:**
- All 5 KPI cards load with numeric values
- Monthly trend line chart renders without errors
- Audit Trail Completeness % is between 0% and 100%
- No visual shows "Data is not available" or similar error

---

### TC-PBI-C-02: Audit Trail Gap Report Identifies Incidents Without History

**Priority:** High

**Precondition:** Create a test incident in `EIT_Incidents` without adding any `EIT_IncidentHistory` entries for it

**Steps:**
1. Refresh the Power BI dataset
2. Navigate to Compliance & Audit → Audit Trail Gap Report table

**Expected Result:**
- The test incident appears in the gap report table
- `Audit Trail Completeness %` card shows < 100%

---

### TC-PBI-C-03: SLA Adherence Visuals Reflect Actual Data

**Priority:** High

**Precondition:** At least 1 Critical incident exists with `DateContained - DateRaised ≤ 5 days` (within SLA) and at least 1 with `> 5 days` (breached)

**Steps:**
1. Refresh Power BI
2. Inspect the SLA adherence section on the Compliance page

**Expected Result:**
- Within-SLA count matches expected value from manual count
- Breached count matches expected value
- Visual colors distinguish within-SLA (green) from breached (red)

---

## Category 7: Operational Resilience

### TC-OR-01: Manual Staleness Flow Re-run After Simulated Failure

**Priority:** High

**Steps:**
1. Simulate a staleness flow failure: in the flow edit, temporarily introduce a syntax error in an expression (e.g., delete a closing bracket)
2. Wait for the scheduled run to fail
3. Fix the expression
4. Manually re-run the flow (Test → Manually)
5. Verify `DaysSinceUpdate` values are correctly updated in `EIT_Incidents`

**Expected Result:**
- After manual re-run: `DaysSinceUpdate` reflects the correct day count
- `StalenessFlag` values are correctly updated (Current/Aging/Stale per threshold)
- Flow run history shows the manual run as Succeeded

---

### TC-OR-02: Manual Fallback — Email Incident Report During Simulated Outage

**Priority:** High

**Steps:**
1. Simulate an EIT outage: temporarily remove the test user's access to the EIT SharePoint site
2. Test user attempts to access EIT Power App → receives access denied
3. Test user sends an incident report email to the domain lead as per the fallback procedure (`EIT-Phase3-Operations-Runbook.md` Section 4.1)
4. Domain lead receives the email and manually logs it

**Expected Result:**
- Incident report email received by domain lead with correct subject format
- Domain lead can complete the manual log (Excel template or equivalent)
- No EIT system interaction required for the fallback to function

---

### TC-OR-03: CSV Restore from Backup

**Priority:** High

**Precondition:** A recent `EIT_Incidents` CSV backup exists; a test SharePoint site (`EnterpriseIncidentTrackerDR`) is available

**Steps:**
1. Create the `EIT_Incidents` list on the DR test site using the JSON schema
2. Import the CSV backup using Microsoft Lists import or Power Automate
3. Verify row count matches the source backup

**Expected Result:**
- All rows imported successfully
- ItemID, Title, Domain, Severity, Status fields all populated correctly
- No import errors

---

### TC-OR-04: Power App Package Restoration

**Priority:** High

**Precondition:** A Power App backup `.zip` file exists

**Steps:**
1. Navigate to `https://make.powerapps.com`
2. Import the backup package (new app, not overwrite)
3. Reconnect the app to the DR test site SharePoint lists
4. Open the restored app and navigate to the Dashboard

**Expected Result:**
- App opens without errors
- Dashboard screen loads and shows data from the DR test site
- No missing controls or broken screens

---

## Test Execution Summary

| Test Case | Category | Priority | Pass/Fail | Notes |
|-----------|----------|----------|-----------|-------|
| TC-DG-01 | Dynamic Groups | High | | |
| TC-DG-02 | Dynamic Groups | High | | |
| TC-DG-03 | Dynamic Groups | Medium | | |
| TC-PIM-01 | PIM | Critical | | |
| TC-PIM-02 | PIM | Critical | | |
| TC-PIM-03 | PIM | High | | |
| TC-PIM-04 | PIM | High | | |
| TC-CA-01 | Conditional Access | High | | |
| TC-CA-02 | Conditional Access | Medium | | |
| TC-AR-FLOW-01 | Access Review Flow | High | | |
| TC-AR-FLOW-02 | Access Review Flow | Medium | | |
| TC-CR-01 | Compliance Report | High | | |
| TC-CR-02 | Compliance Report | Critical | | |
| TC-CR-03 | Compliance Report | High | | |
| TC-CR-04 | Compliance Report | Medium | | |
| TC-PBI-C-01 | PBI Compliance Page | High | | |
| TC-PBI-C-02 | PBI Compliance Page | High | | |
| TC-PBI-C-03 | PBI Compliance Page | High | | |
| TC-OR-01 | Operational Resilience | High | | |
| TC-OR-02 | Operational Resilience | High | | |
| TC-OR-03 | Operational Resilience | High | | |
| TC-OR-04 | Operational Resilience | High | | |

---

## Phase 3 Go / No-Go Criteria

| Workstream | Go Criteria |
|-----------|-------------|
| Entra ID Access Control | TC-PIM-01, TC-PIM-02 both Pass (PIM blocking non-active users is a security control) |
| Compliance Reporting | TC-CR-02 Pass (report counts must match source data) |
| Operational Resilience | TC-OR-03, TC-OR-04 Pass (backup restore must be validated before relying on it) |
| **Overall Phase 3** | All Critical-priority test cases Pass; TC-PIM-01 is an absolute blocker |

**Blocking condition:** TC-PIM-01 (eligible user cannot access P&C without activation) is an absolute blocker. If PIM-eligible users can still see P&C incidents without activating, the PIM deployment is ineffective and must be corrected before Phase 3 is considered complete.

---

## Phase 3 Licensing Prerequisites

| Feature | Licence Required | Notes |
|---------|----------------|-------|
| Entra ID Dynamic Groups | Entra ID P1 (M365 E3/E5) | Required for TC-DG-01, TC-DG-02 |
| Privileged Identity Management | Entra ID P2 (M365 E5 or add-on) | Required for all TC-PIM tests |
| Conditional Access | Entra ID P1 | Required for TC-CA-01 |
| Entra ID Access Reviews | Entra ID P2 | Required for automated access review configuration |

If P2 licensing is not available, PIM tests (TC-PIM-01 through TC-PIM-04) cannot be executed and PIM is not deployable. The alternative (manual periodic access review only) must be documented as the accepted control.
