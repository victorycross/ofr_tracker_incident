# OFR Issue Tracker — Test Plan

**Version:** 1.1
**Date:** February 19, 2025
**Application:** OFR Issue Tracker (M365-Native)
**Environment:** [TENANT-DOMAIN] (default) — M365 Business Standard + Power Apps Developer + Power Automate Free

---

## 1. Test Objectives

Validate that the M365-native OFR Issue Tracker correctly supports the full issue lifecycle: intake submission, triage, promotion to active tracking, issue updates with audit trail, staleness calculations, dashboard KPIs, and filtering/search. Confirm all three layers (SharePoint data, Power Apps UI, Power Automate automation) work together end-to-end.

---

## 2. Test Environment

| Component | Details |
|-----------|---------|
| SharePoint Site | https://[TENANT].sharepoint.com/sites/OFRIssueTracker |
| Power Apps App ID | `0fbbc26c-ad71-476a-bcfc-edc0d7989533` |
| Staleness Flow ID | `aefb8de0-35fe-4d5d-a629-ddd8502ee5aa` |
| Intake Promotion Flow ID | `1c631640-113f-4602-805e-1d693582de8c` |
| Test User | [ADMIN-EMAIL] (admin) |
| Browser | Microsoft Edge or Google Chrome (latest) |

### Pre-loaded Sample Data

| List | Records | Details |
|------|---------|---------|
| OFR_Issues | 8 items | OFR-1 through OFR-8 with mixed priorities, statuses, staleness |
| OFR_UpdateHistory | 17 entries | 2-3 updates per issue |
| OFR_IntakeQueue | 2 items | Both "Pending" status |

---

## 3. Test Categories

### 3.1 — SharePoint Data Layer Validation

#### TC-SP-01: Verify OFR_Issues list schema
| Field | Value |
|-------|-------|
| **Precondition** | Navigate to OFR_Issues list in SharePoint |
| **Steps** | 1. Open https://[TENANT].sharepoint.com/sites/OFRIssueTracker/Lists/OFR_Issues <br> 2. Verify all 11 columns exist: Title, ItemID, Owner, Priority, Status, DateRaised, LastUpdated, NextAction, DaysSinceUpdate, StalenessFlag, FunctionalGroup <br> 3. Verify column types match spec (Choice fields have correct options) |
| **Expected Result** | All columns present with correct types. Priority has High/Medium/Low. Status has New/Active/Monitoring/Escalated/Closed. StalenessFlag has Current/Aging/Stale. FunctionalGroup has all 10 group choices. |
| **Priority** | High |

#### TC-SP-02: Verify OFR_UpdateHistory list schema
| Field | Value |
|-------|-------|
| **Precondition** | Navigate to OFR_UpdateHistory list |
| **Steps** | 1. Open OFR_UpdateHistory list <br> 2. Verify 6 columns: Title, ParentItemID, UpdateDate, StatusAtUpdate, Notes, UpdatedBy <br> 3. Verify StatusAtUpdate choices match OFR_Issues.Status choices |
| **Expected Result** | All columns present. StatusAtUpdate has New/Active/Monitoring/Escalated/Closed. |
| **Priority** | High |

#### TC-SP-03: Verify OFR_IntakeQueue list schema
| Field | Value |
|-------|-------|
| **Precondition** | Navigate to OFR_IntakeQueue list |
| **Steps** | 1. Open OFR_IntakeQueue list <br> 2. Verify 7 columns: Title, Owner, Priority, Description, DateSubmitted, TriageStatus, FunctionalGroup <br> 3. Verify TriageStatus choices: Pending/Promoted/Dismissed/Accepted/Rejected <br> 4. Verify default value is "Pending" <br> 5. Verify FunctionalGroup has all 10 group choices |
| **Expected Result** | All columns present with correct types. TriageStatus has all 5 choice values. FunctionalGroup has 10 choices. |
| **Priority** | High |

#### TC-SP-04: Verify sample data integrity
| Field | Value |
|-------|-------|
| **Precondition** | Sample data has been loaded |
| **Steps** | 1. Open OFR_Issues — verify 8 items (OFR-1 to OFR-8) <br> 2. Open OFR_UpdateHistory — verify 17 entries <br> 3. Open OFR_IntakeQueue — verify 2 pending items <br> 4. Spot-check: OFR-1 should be High/Active, OFR-3 should be Medium/Active/Stale, OFR-5 should be Low/Monitoring |
| **Expected Result** | All records present with correct field values matching the sample data specification. |
| **Priority** | High |

#### TC-SP-05: Verify FunctionalGroup column on OFR_Issues
| Field | Value |
|-------|-------|
| **Precondition** | Navigate to OFR_Issues list in SharePoint |
| **Steps** | 1. Open OFR_Issues list <br> 2. Verify FunctionalGroup column exists as Choice type <br> 3. Verify all 10 choices: Risk Management Office, Engagement Risk, Client Risk and KYC, Technology Risk & AI Trust, National Security, OGC General Counsel, OGC Privacy, OGC Contracts, Internal Audit, Independence <br> 4. Verify sample data has FunctionalGroup values for all 8 records |
| **Expected Result** | FunctionalGroup column exists with all 10 choices. Sample data has correct group assignments. |
| **Priority** | High |

#### TC-SP-06: Verify FunctionalGroup column on OFR_IntakeQueue
| Field | Value |
|-------|-------|
| **Precondition** | Navigate to OFR_IntakeQueue list in SharePoint |
| **Steps** | 1. Open OFR_IntakeQueue list <br> 2. Verify FunctionalGroup column exists as Choice type <br> 3. Verify all 10 choices match OFR_Issues.FunctionalGroup choices |
| **Expected Result** | FunctionalGroup column exists with identical 10 choices as OFR_Issues. |
| **Priority** | High |

---

### 3.2 — Power Apps: Dashboard Screen

#### TC-PA-D01: Dashboard loads and displays KPIs
| Field | Value |
|-------|-------|
| **Precondition** | Open Power Apps canvas app |
| **Steps** | 1. Launch app from make.powerapps.com or Teams <br> 2. Verify Dashboard Screen loads as the home screen <br> 3. Check KPI card values |
| **Expected Result** | KPI cards show: Open Items = 8, Stale = 2 (OFR-3, OFR-8), High Priority = 4 (OFR-1, OFR-2, OFR-4, OFR-8). Medium and Low counts are correct. |
| **Priority** | High |

#### TC-PA-D02: Intake queue displays pending items
| Field | Value |
|-------|-------|
| **Precondition** | Dashboard Screen loaded, 2 intake items exist |
| **Steps** | 1. Scroll to intake queue section <br> 2. Verify both pending items are displayed <br> 3. Verify each card shows: Title, Owner, Priority badge, DateSubmitted |
| **Expected Result** | "Audit finding on access controls" (High) and "Project staffing shortfall for Q2" (Medium) displayed with correct details. Sorted by Priority then DateSubmitted. |
| **Priority** | High |

#### TC-PA-D03: New Issue submission form
| Field | Value |
|-------|-------|
| **Precondition** | Dashboard Screen loaded |
| **Steps** | 1. Click "+ New Issue" button <br> 2. Verify overlay form appears with fields: Title, Owner, Priority, Description <br> 3. Fill in test data: Title="Test intake item", Owner="Test User", Priority=Medium, Description="Testing intake submission" <br> 4. Submit the form |
| **Expected Result** | Form submits. New item appears in OFR_IntakeQueue with TriageStatus="Pending". Item appears in the intake gallery on Dashboard. |
| **Priority** | High |

#### TC-PA-D04: Dismiss intake item
| Field | Value |
|-------|-------|
| **Precondition** | At least one pending intake item exists |
| **Steps** | 1. Find a pending intake item in the gallery <br> 2. Click "Dismiss" button <br> 3. Verify the item disappears from the gallery |
| **Expected Result** | Item's TriageStatus changes to "Dismissed" in SharePoint. Item no longer appears in the pending intake gallery. KPIs update if applicable. |
| **Priority** | Medium |

#### TC-PA-D05: Navigate to Tracker Screen
| Field | Value |
|-------|-------|
| **Precondition** | Dashboard Screen loaded |
| **Steps** | 1. Click "View Tracker" or navigation button <br> 2. Verify navigation to Tracker Screen |
| **Expected Result** | Tracker Screen loads showing the issue table with all open items. |
| **Priority** | High |

#### TC-PA-D06: Intake Review panel opens on item click
| Field | Value |
|-------|-------|
| **Precondition** | Dashboard Screen loaded, at least one pending intake item exists |
| **Steps** | 1. Click on a pending intake item in the gallery <br> 2. Verify the Intake Review panel opens on the right side <br> 3. Verify panel displays: Title, Description, Priority, Date Submitted <br> 4. Verify "Assign Owner" text input and "Accept into Tracker" / "Reject" buttons are visible |
| **Expected Result** | Panel opens with correct data from the selected intake item. All fields populated. Accept/Reject buttons visible. |
| **Priority** | High |

#### TC-PA-D07: Accept intake item via panel
| Field | Value |
|-------|-------|
| **Precondition** | Intake Review panel is open for a pending item |
| **Steps** | 1. Enter an owner name in the "Assign Owner" field (e.g., "Test User") <br> 2. Click "Accept into Tracker" <br> 3. Verify the panel closes <br> 4. Verify the item disappears from the intake gallery <br> 5. Verify a new issue appears in OFR_Issues with Title, Priority from the intake item, Owner from the text input, Status = "New" <br> 6. Verify the intake item's TriageStatus is now "Accepted" in SharePoint |
| **Expected Result** | New issue created in OFR_Issues with correct data. Intake item marked "Accepted". Panel closes. Success notification displayed. |
| **Priority** | High |

#### TC-PA-D08: Reject intake item via panel
| Field | Value |
|-------|-------|
| **Precondition** | Intake Review panel is open for a pending item |
| **Steps** | 1. Click "Reject" button <br> 2. Verify the panel closes <br> 3. Verify the item disappears from the intake gallery <br> 4. Verify the intake item's TriageStatus is now "Rejected" in SharePoint |
| **Expected Result** | Intake item marked "Rejected". Panel closes. Warning notification displayed. No new issue created. |
| **Priority** | High |

#### TC-PA-D09: Close panel without action
| Field | Value |
|-------|-------|
| **Precondition** | Intake Review panel is open |
| **Steps** | 1. Click the "X" close button <br> 2. Verify the panel closes <br> 3. Verify the intake item remains in the gallery with "Pending" status |
| **Expected Result** | Panel closes. No data changes. Item still pending. |
| **Priority** | Medium |

#### TC-PA-D10: Panel hidden on initial load
| Field | Value |
|-------|-------|
| **Precondition** | Navigate to Dashboard Screen |
| **Steps** | 1. Verify the Intake Review panel is not visible when the Dashboard first loads |
| **Expected Result** | Panel controls are hidden (Visible = false). Only KPIs and intake gallery are visible. |
| **Priority** | Medium |

#### TC-PA-D11: FunctionalGroup flows through acceptance
| Field | Value |
|-------|-------|
| **Precondition** | A pending intake item exists with FunctionalGroup set |
| **Steps** | 1. Click on the intake item to open Intake Review panel <br> 2. Enter an owner name <br> 3. Click "Accept into Tracker" <br> 4. Navigate to Tracker or open OFR_Issues in SharePoint <br> 5. Find the newly created issue <br> 6. Verify FunctionalGroup matches the intake item's FunctionalGroup |
| **Expected Result** | FunctionalGroup value flows from OFR_IntakeQueue to OFR_Issues during acceptance. |
| **Priority** | High |

#### TC-PA-D12: Navigate to Group Allocation Screen
| Field | Value |
|-------|-------|
| **Precondition** | Dashboard Screen loaded |
| **Steps** | 1. Click "Group Allocation" button <br> 2. Verify navigation to GroupAllocationScreen |
| **Expected Result** | GroupAllocationScreen loads showing 10 group cards with active issue counts. |
| **Priority** | High |

#### TC-PA-D13: Navigate to Kanban Board Screen
| Field | Value |
|-------|-------|
| **Precondition** | Dashboard Screen loaded |
| **Steps** | 1. Click "Kanban Board" button <br> 2. Verify navigation to KanbanScreen |
| **Expected Result** | KanbanScreen loads showing 4 status columns with issue cards. |
| **Priority** | High |

---

### 3.2b — Power Apps: Submit Screen

#### TC-PA-S01: Navigate to Submit Screen
| Field | Value |
|-------|-------|
| **Precondition** | Dashboard Screen loaded |
| **Steps** | 1. Click "+ Submit New Issue" button <br> 2. Verify navigation to SubmitScreen |
| **Expected Result** | Submit Screen loads showing the issue submission form with Title, Priority, Description fields. |
| **Priority** | High |

#### TC-PA-S02: Submit a new issue
| Field | Value |
|-------|-------|
| **Precondition** | Submit Screen loaded |
| **Steps** | 1. Enter Title: "Test Submit Screen Issue" <br> 2. Select Priority: High <br> 3. Enter Description: "Testing the submit screen form" <br> 4. Click "Submit" |
| **Expected Result** | New item created in OFR_IntakeQueue with TriageStatus = "Pending", DateSubmitted = now. Success notification displayed. Form resets. |
| **Priority** | High |

#### TC-PA-S03: Back to Dashboard from Submit Screen
| Field | Value |
|-------|-------|
| **Precondition** | Submit Screen loaded |
| **Steps** | 1. Click "Back to Dashboard" button |
| **Expected Result** | Returns to Dashboard Screen. |
| **Priority** | Low |

#### TC-PA-S04: FunctionalGroup dropdown present on Submit Screen
| Field | Value |
|-------|-------|
| **Precondition** | Submit Screen loaded |
| **Steps** | 1. Verify a FunctionalGroup dropdown is visible on the form <br> 2. Open the dropdown <br> 3. Verify all 10 functional groups are listed as options |
| **Expected Result** | Dropdown present with all 10 groups: Risk Management Office, Engagement Risk, Client Risk and KYC, Technology Risk & AI Trust, National Security, OGC General Counsel, OGC Privacy, OGC Contracts, Internal Audit, Independence. |
| **Priority** | High |

#### TC-PA-S05: FunctionalGroup value saved on submit
| Field | Value |
|-------|-------|
| **Precondition** | Submit Screen loaded |
| **Steps** | 1. Fill in Title, Priority, Description <br> 2. Select "OGC Privacy" from the FunctionalGroup dropdown <br> 3. Click Submit <br> 4. Open OFR_IntakeQueue in SharePoint <br> 5. Find the new intake item |
| **Expected Result** | New intake item has FunctionalGroup = "OGC Privacy". |
| **Priority** | High |

---

### 3.3 — Power Apps: Tracker Screen

#### TC-PA-T01: Issue table displays all open items
| Field | Value |
|-------|-------|
| **Precondition** | Tracker Screen loaded |
| **Steps** | 1. Verify gallery shows all open issues <br> 2. Verify columns: ItemID, Title, Owner, Priority badge, Status badge, DaysSinceUpdate <br> 3. Count visible rows |
| **Expected Result** | All 8 open issues displayed with correct data in all columns. |
| **Priority** | High |

#### TC-PA-T02: Staleness color-coding
| Field | Value |
|-------|-------|
| **Precondition** | Tracker Screen loaded with sample data |
| **Steps** | 1. Find OFR-2 (DaysSinceUpdate=2) — should show green/Current <br> 2. Find OFR-1 (DaysSinceUpdate=8) — should show amber/Aging <br> 3. Find OFR-3 (DaysSinceUpdate=19) — should show red/Stale <br> 4. Find OFR-8 (DaysSinceUpdate=21) — should show red/Stale |
| **Expected Result** | Green for 0-7 days, amber for 8-14 days, red for 15+ days. Visual emphasis (bold/highlight) on stale items. |
| **Priority** | High |

#### TC-PA-T03: Filter — All Open
| Field | Value |
|-------|-------|
| **Precondition** | Tracker Screen loaded |
| **Steps** | 1. Click "All Open" filter toggle <br> 2. Count visible items |
| **Expected Result** | All 8 open items visible (none are Closed). |
| **Priority** | Medium |

#### TC-PA-T04: Filter — Stale
| Field | Value |
|-------|-------|
| **Precondition** | Tracker Screen loaded |
| **Steps** | 1. Click "Stale" filter toggle <br> 2. Verify filtered results |
| **Expected Result** | Only OFR-3 and OFR-8 visible (StalenessFlag = "Stale"). |
| **Priority** | High |

#### TC-PA-T05: Filter — High Priority
| Field | Value |
|-------|-------|
| **Precondition** | Tracker Screen loaded |
| **Steps** | 1. Click "High" filter toggle <br> 2. Verify filtered results |
| **Expected Result** | Only OFR-1, OFR-2, OFR-4, OFR-8 visible (Priority = "High"). |
| **Priority** | Medium |

#### TC-PA-T06: Filter — Medium Priority
| Field | Value |
|-------|-------|
| **Precondition** | Tracker Screen loaded |
| **Steps** | 1. Click "Medium" filter toggle <br> 2. Verify filtered results |
| **Expected Result** | Only OFR-3, OFR-6, OFR-7 visible (Priority = "Medium"). |
| **Priority** | Medium |

#### TC-PA-T07: Filter — Low Priority
| Field | Value |
|-------|-------|
| **Precondition** | Tracker Screen loaded |
| **Steps** | 1. Click "Low" filter toggle <br> 2. Verify filtered results |
| **Expected Result** | Only OFR-5 visible (Priority = "Low"). |
| **Priority** | Low |

#### TC-PA-T08: Search by Title
| Field | Value |
|-------|-------|
| **Precondition** | Tracker Screen loaded |
| **Steps** | 1. Type "migration" in search box <br> 2. Verify filtered results |
| **Expected Result** | Only OFR-1 ("Client data migration delays") visible. |
| **Priority** | High |

#### TC-PA-T09: Search by Owner
| Field | Value |
|-------|-------|
| **Precondition** | Tracker Screen loaded |
| **Steps** | 1. Clear search box <br> 2. Type "Sarah" in search box <br> 3. Verify filtered results |
| **Expected Result** | OFR-1 and OFR-5 visible (Owner = "Sarah Chen"). |
| **Priority** | High |

#### TC-PA-T10: Search by ItemID
| Field | Value |
|-------|-------|
| **Precondition** | Tracker Screen loaded |
| **Steps** | 1. Clear search box <br> 2. Type "OFR-4" in search box <br> 3. Verify filtered results |
| **Expected Result** | Only OFR-4 ("Budget reconciliation discrepancies") visible. |
| **Priority** | Medium |

#### TC-PA-T11: Combined filter + search
| Field | Value |
|-------|-------|
| **Precondition** | Tracker Screen loaded |
| **Steps** | 1. Click "High" filter toggle <br> 2. Type "Park" in search box |
| **Expected Result** | Only OFR-4 and OFR-8 visible (High priority AND Owner contains "Park"). |
| **Priority** | Medium |

#### TC-PA-T12: Column sort
| Field | Value |
|-------|-------|
| **Precondition** | Tracker Screen loaded, "All Open" filter active |
| **Steps** | 1. Click Priority column header to sort <br> 2. Verify order changes <br> 3. Click DaysSinceUpdate column header <br> 4. Verify order changes (highest staleness first or last) |
| **Expected Result** | Gallery re-sorts by the clicked column. Clicking again reverses sort direction. |
| **Priority** | Medium |

#### TC-PA-T13: Navigate to Issue Detail
| Field | Value |
|-------|-------|
| **Precondition** | Tracker Screen loaded |
| **Steps** | 1. Tap on OFR-2 row <br> 2. Verify navigation to Issue Detail Screen |
| **Expected Result** | Issue Detail Screen loads showing OFR-2 header info and update history. |
| **Priority** | High |

#### TC-PA-T14: Back to Dashboard navigation
| Field | Value |
|-------|-------|
| **Precondition** | Tracker Screen loaded |
| **Steps** | 1. Click "Back to Dashboard" button |
| **Expected Result** | Returns to Dashboard Screen with KPIs and intake gallery. |
| **Priority** | Low |

#### TC-PA-T15: FunctionalGroup column visible on Tracker
| Field | Value |
|-------|-------|
| **Precondition** | Tracker Screen loaded |
| **Steps** | 1. Verify a "Group" column header exists in the table <br> 2. Verify each row displays the FunctionalGroup value for that issue <br> 3. Verify long group names are truncated (hover for full name) |
| **Expected Result** | Group column visible between Status and Days Stale. All 8 rows show correct FunctionalGroup values. Long names (e.g., "Technology Risk & AI Trust") are truncated with full text in tooltip. |
| **Priority** | High |

#### TC-PA-T16: Search by FunctionalGroup
| Field | Value |
|-------|-------|
| **Precondition** | Tracker Screen loaded |
| **Steps** | 1. Type "Privacy" in the search box <br> 2. Verify filtered results |
| **Expected Result** | Only OFR-3 visible (FunctionalGroup = "OGC Privacy"). |
| **Priority** | Medium |

---

### 3.4 — Power Apps: Issue Detail Screen

#### TC-PA-I01: Issue header displays correctly
| Field | Value |
|-------|-------|
| **Precondition** | Navigated to Issue Detail for OFR-2 |
| **Steps** | 1. Verify header shows: ItemID="OFR-2", Title="Regulatory compliance gap in reporting", Owner="James Wilson", Priority="High", Status="Escalated", DateRaised, LastUpdated |
| **Expected Result** | All header fields match the OFR_Issues record for OFR-2. |
| **Priority** | High |

#### TC-PA-I02: Update history timeline displays correctly
| Field | Value |
|-------|-------|
| **Precondition** | Issue Detail Screen for OFR-2 |
| **Steps** | 1. Verify update history gallery shows 3 entries <br> 2. Verify sorted descending by UpdateDate (newest first) <br> 3. Verify each entry shows: date, status badge, notes, updated by |
| **Expected Result** | 3 entries visible in order: "Escalated to partner level..." (Feb 16), "Drafted remediation plan..." (Feb 5), "Gap found in quarterly reporting..." (Jan 20). |
| **Priority** | High |

#### TC-PA-I03: Update history for issue with many updates
| Field | Value |
|-------|-------|
| **Precondition** | Navigate to Issue Detail for OFR-1 |
| **Steps** | 1. Verify update history shows 3 entries <br> 2. Verify chronological order (newest first) <br> 3. Verify all entries have ParentItemID = "OFR-1" |
| **Expected Result** | 3 entries: "Server allocation pending..." (Feb 10), "Migration tool selected..." (Jan 22), "Issue identified during client..." (Jan 15). |
| **Priority** | Medium |

#### TC-PA-I04: Add update — notes only (no status change)
| Field | Value |
|-------|-------|
| **Precondition** | Issue Detail Screen for OFR-1 (Status = Active) |
| **Steps** | 1. Enter notes: "Follow-up meeting scheduled with IT for server allocation" <br> 2. Leave status dropdown as current ("Active") <br> 3. Click "Save Update" |
| **Expected Result** | New entry appears at top of timeline with today's date, status "Active", the entered notes, and current user. OFR_Issues.LastUpdated updates to now. OFR_Issues.DaysSinceUpdate resets to 0. |
| **Priority** | High |

#### TC-PA-I05: Add update — with status change
| Field | Value |
|-------|-------|
| **Precondition** | Issue Detail Screen for OFR-7 (Status = New) |
| **Steps** | 1. Enter notes: "Initial assessment complete, assigning cross-team leads" <br> 2. Change status dropdown from "New" to "Active" <br> 3. Click "Save Update" |
| **Expected Result** | New entry appears in timeline with status "Active". OFR_Issues.Status changes from "New" to "Active". OFR_Issues.LastUpdated and DaysSinceUpdate update. Header refreshes to show new status. |
| **Priority** | High |

#### TC-PA-I06: Add update — validation (empty notes)
| Field | Value |
|-------|-------|
| **Precondition** | Issue Detail Screen for any issue |
| **Steps** | 1. Leave notes field empty <br> 2. Click "Save Update" |
| **Expected Result** | Form validation prevents submission. Error message or visual indicator on required notes field. No record created. |
| **Priority** | Medium |

#### TC-PA-I07: Update history reflects in SharePoint
| Field | Value |
|-------|-------|
| **Precondition** | An update was added via TC-PA-I04 or TC-PA-I05 |
| **Steps** | 1. Open OFR_UpdateHistory list in SharePoint <br> 2. Find the new entry <br> 3. Verify ParentItemID, UpdateDate, StatusAtUpdate, Notes, UpdatedBy |
| **Expected Result** | SharePoint record matches exactly what was entered in Power Apps. |
| **Priority** | High |

#### TC-PA-I08: Back to Tracker navigation
| Field | Value |
|-------|-------|
| **Precondition** | Issue Detail Screen loaded |
| **Steps** | 1. Click "Back to Tracker" button |
| **Expected Result** | Returns to Tracker Screen with filters preserved. |
| **Priority** | Low |

#### TC-PA-I09: FunctionalGroup displayed on Issue Detail
| Field | Value |
|-------|-------|
| **Precondition** | Navigated to Issue Detail for OFR-1 |
| **Steps** | 1. Verify the header section includes a "FUNCTIONAL GROUP" label <br> 2. Verify the value displayed is "Engagement Risk" (per sample data) |
| **Expected Result** | FunctionalGroup label and value displayed in the issue header. Value matches the OFR_Issues record. |
| **Priority** | Medium |

---

### 3.5 — Power Automate: Daily Staleness Calculator

#### TC-FL-S01: Manual test run
| Field | Value |
|-------|-------|
| **Precondition** | Flow `aefb8de0-35fe-4d5d-a629-ddd8502ee5aa` exists and is turned on |
| **Steps** | 1. Open flow in Power Automate <br> 2. Click "Test" → "Manually" → "Run flow" <br> 3. Wait for flow to complete <br> 4. Check run history for success |
| **Expected Result** | Flow runs successfully. All open issues processed. No errors in run history. |
| **Priority** | High |

#### TC-FL-S02: DaysSinceUpdate calculation accuracy
| Field | Value |
|-------|-------|
| **Precondition** | Staleness flow has been run |
| **Steps** | 1. Open OFR_Issues in SharePoint <br> 2. For each open issue, manually calculate days between LastUpdated and today <br> 3. Compare with DaysSinceUpdate column |
| **Expected Result** | DaysSinceUpdate values match manual calculation (within 1 day tolerance for timezone differences). |
| **Priority** | High |

#### TC-FL-S03: StalenessFlag accuracy
| Field | Value |
|-------|-------|
| **Precondition** | Staleness flow has been run |
| **Steps** | 1. Open OFR_Issues in SharePoint <br> 2. For each item, verify StalenessFlag: <br> — DaysSinceUpdate 0-7 → "Current" <br> — DaysSinceUpdate 8-14 → "Aging" <br> — DaysSinceUpdate 15+ → "Stale" |
| **Expected Result** | All StalenessFlag values match the threshold rules. |
| **Priority** | High |

#### TC-FL-S04: Closed issues excluded
| Field | Value |
|-------|-------|
| **Precondition** | At least one issue has Status = "Closed" |
| **Steps** | 1. Manually close an issue (set Status to "Closed" in SharePoint) <br> 2. Run the staleness flow <br> 3. Check if the closed issue was processed |
| **Expected Result** | Closed issue's DaysSinceUpdate and StalenessFlag are NOT recalculated. Flow's "Get items" filter excludes Closed items. |
| **Priority** | High |

#### TC-FL-S05: Flow after adding an update
| Field | Value |
|-------|-------|
| **Precondition** | An issue was just updated (DaysSinceUpdate = 0) |
| **Steps** | 1. Add an update to OFR-3 (currently Stale) via Power Apps <br> 2. Verify DaysSinceUpdate resets to 0 in Power Apps <br> 3. Run staleness flow <br> 4. Verify OFR-3 now shows "Current" |
| **Expected Result** | After update: DaysSinceUpdate = 0. After staleness flow: StalenessFlag = "Current" (was "Stale"). |
| **Priority** | High |

---

### 3.6 — Power Automate: Intake Promotion

#### TC-FL-P01: Promote intake item via Power Apps
| Field | Value |
|-------|-------|
| **Precondition** | At least one pending intake item exists in OFR_IntakeQueue |
| **Steps** | 1. Open Power Apps Dashboard <br> 2. Find "Audit finding on access controls" in intake gallery <br> 3. Click "Move to Tracker" / "Promote" button <br> 4. Wait for flow to complete |
| **Expected Result** | Flow runs successfully. Notification or visual confirmation in app. |
| **Priority** | High |

#### TC-FL-P02: Verify promoted issue in OFR_Issues
| Field | Value |
|-------|-------|
| **Precondition** | TC-FL-P01 completed |
| **Steps** | 1. Navigate to Tracker Screen or open OFR_Issues in SharePoint <br> 2. Find the newly created issue <br> 3. Verify fields: Title, Owner, Priority match intake item <br> 4. Verify: Status = "New", DaysSinceUpdate = 0, StalenessFlag = "Current" <br> 5. Verify: DateRaised and LastUpdated are set to promotion time <br> 6. Verify: ItemID is in "OFR-N" format <br> 7. Verify: NextAction = Description from intake item |
| **Expected Result** | New issue exists with all fields correctly mapped from intake item. |
| **Priority** | High |

#### TC-FL-P03: Verify audit trail entry
| Field | Value |
|-------|-------|
| **Precondition** | TC-FL-P01 completed |
| **Steps** | 1. Open OFR_UpdateHistory in SharePoint <br> 2. Find entry with ParentItemID matching the new issue's ItemID <br> 3. Verify: StatusAtUpdate = "New", Notes = "Promoted from intake queue", UpdatedBy = Owner from intake |
| **Expected Result** | Audit trail entry exists with correct metadata. |
| **Priority** | High |

#### TC-FL-P04: Verify intake item marked as Promoted
| Field | Value |
|-------|-------|
| **Precondition** | TC-FL-P01 completed |
| **Steps** | 1. Open OFR_IntakeQueue in SharePoint <br> 2. Find the promoted item <br> 3. Verify TriageStatus = "Promoted" |
| **Expected Result** | Intake item's TriageStatus changed from "Pending" to "Promoted". Item no longer shows in the Power Apps intake gallery. |
| **Priority** | High |

#### TC-FL-P05: Verify NewItemID returned
| Field | Value |
|-------|-------|
| **Precondition** | TC-FL-P01 completed |
| **Steps** | 1. Check Power Automate flow run history <br> 2. Verify "Respond to a PowerApp or flow" action output <br> 3. Confirm NewItemID value matches the created issue's ItemID |
| **Expected Result** | Flow returns the correct ItemID (e.g., "OFR-9") to the calling Power App. |
| **Priority** | Medium |

#### TC-FL-P06: KPIs update after promotion
| Field | Value |
|-------|-------|
| **Precondition** | TC-FL-P01 completed, return to Dashboard |
| **Steps** | 1. Navigate back to Dashboard <br> 2. Check Open Items KPI <br> 3. Check intake gallery count |
| **Expected Result** | Open Items increases by 1 (8 → 9). Intake gallery shows one fewer pending item. |
| **Priority** | Medium |

---

### 3.7 — Power Apps: Group Allocation Screen

#### TC-PA-G01: Group Allocation screen loads
| Field | Value |
|-------|-------|
| **Precondition** | Navigate to GroupAllocationScreen from Dashboard |
| **Steps** | 1. Click "Group Allocation" button on Dashboard <br> 2. Verify screen loads with header, 10 group cards, and summary labels |
| **Expected Result** | Screen loads with all 10 group cards arranged in a 2×5 grid. Each card shows a group name and active issue count. |
| **Priority** | High |

#### TC-PA-G02: Group card counts are accurate
| Field | Value |
|-------|-------|
| **Precondition** | GroupAllocationScreen loaded, sample data has 8 issues with FunctionalGroup values |
| **Steps** | 1. Verify each card's count matches the number of non-Closed issues for that group <br> 2. Cross-reference with OFR_Issues SharePoint list <br> 3. Sample check: "Engagement Risk" should show 1 (OFR-1), "Risk Management Office" should show 2 (OFR-2, OFR-8), "Technology Risk & AI Trust" should show 2 (OFR-6, OFR-7) |
| **Expected Result** | All card counts match the actual number of active (non-Closed) issues per group. |
| **Priority** | High |

#### TC-PA-G03: Closed issues excluded from counts
| Field | Value |
|-------|-------|
| **Precondition** | An issue with a FunctionalGroup has been closed |
| **Steps** | 1. Close an issue (change Status to Closed) <br> 2. Return to GroupAllocationScreen <br> 3. Verify the card for that group shows a reduced count |
| **Expected Result** | Closed issues are not counted. Card count decreases by 1. |
| **Priority** | High |

#### TC-PA-G04: Unassigned count displays correctly
| Field | Value |
|-------|-------|
| **Precondition** | GroupAllocationScreen loaded |
| **Steps** | 1. Verify the "Unassigned" count at the bottom <br> 2. Create a test issue with no FunctionalGroup <br> 3. Refresh and verify unassigned count increases |
| **Expected Result** | Unassigned count shows the number of active issues with blank FunctionalGroup. Displayed in orange as a warning. |
| **Priority** | Medium |

#### TC-PA-G05: Total active count matches
| Field | Value |
|-------|-------|
| **Precondition** | GroupAllocationScreen loaded |
| **Steps** | 1. Sum all 10 group card counts plus the unassigned count <br> 2. Compare with the total active count label |
| **Expected Result** | Total active count = sum of all group counts + unassigned count. |
| **Priority** | Medium |

#### TC-PA-G06: Back to Dashboard navigation
| Field | Value |
|-------|-------|
| **Precondition** | GroupAllocationScreen loaded |
| **Steps** | 1. Click "Back to Dashboard" button |
| **Expected Result** | Returns to Dashboard Screen. |
| **Priority** | Low |

---

### 3.8 — Power Apps: Kanban Board Screen

#### TC-PA-K01: Kanban Board screen loads
| Field | Value |
|-------|-------|
| **Precondition** | Navigate to KanbanScreen from Dashboard |
| **Steps** | 1. Click "Kanban Board" button on Dashboard <br> 2. Verify screen loads with 4 colour-coded column headers and galleries |
| **Expected Result** | Screen loads with 4 columns: New (blue header), Active (orange header), Escalated (red header), Monitoring (light orange header). |
| **Priority** | High |

#### TC-PA-K02: Issues appear in correct status columns
| Field | Value |
|-------|-------|
| **Precondition** | KanbanScreen loaded with sample data |
| **Steps** | 1. Verify New column contains OFR-6 (Status=New) <br> 2. Verify Active column contains OFR-1, OFR-2, OFR-5, OFR-7 (Status=Active) <br> 3. Verify Escalated column contains OFR-4 (Status=Escalated) <br> 4. Verify Monitoring column contains OFR-3, OFR-8 (Status=Monitoring) |
| **Expected Result** | Each issue appears in the correct column matching its Status value. |
| **Priority** | High |

#### TC-PA-K03: Card content displays correctly
| Field | Value |
|-------|-------|
| **Precondition** | KanbanScreen loaded |
| **Steps** | 1. Examine a card (e.g., OFR-4 in Escalated) <br> 2. Verify card shows: ItemID (bold blue), Priority badge, Title, Owner, FunctionalGroup (italic), Days since update with staleness colour <br> 3. Verify left-edge staleness accent stripe colour |
| **Expected Result** | All card elements display with correct data and formatting. Staleness stripe matches DaysSinceUpdate thresholds. |
| **Priority** | High |

#### TC-PA-K04: Card tap navigates to Issue Detail
| Field | Value |
|-------|-------|
| **Precondition** | KanbanScreen loaded |
| **Steps** | 1. Tap on OFR-4 card in the Escalated column <br> 2. Verify navigation to IssueDetailScreen |
| **Expected Result** | IssueDetailScreen loads for OFR-4 with correct header and update history. |
| **Priority** | High |

#### TC-PA-K05: Column counts match expected
| Field | Value |
|-------|-------|
| **Precondition** | KanbanScreen loaded with sample data |
| **Steps** | 1. Count cards in each column <br> 2. Verify: New=1, Active=4, Escalated=1, Monitoring=2 |
| **Expected Result** | Card counts per column match the number of issues with each status. Total across all columns = 8 (all open issues). |
| **Priority** | Medium |

#### TC-PA-K06: Staleness colours on cards
| Field | Value |
|-------|-------|
| **Precondition** | KanbanScreen loaded with sample data |
| **Steps** | 1. Find OFR-2 (DaysSinceUpdate=5) — accent stripe should be blue (Current) <br> 2. Find OFR-1 (DaysSinceUpdate=9) — accent stripe should be orange (Aging) <br> 3. Find OFR-3 (DaysSinceUpdate=22) — accent stripe should be red (Stale) |
| **Expected Result** | Staleness accent stripe and days text match staleness thresholds: blue (0-7), orange (8-14), red (15+). |
| **Priority** | Medium |

#### TC-PA-K07: Closed issues excluded from Kanban
| Field | Value |
|-------|-------|
| **Precondition** | At least one issue has Status = Closed |
| **Steps** | 1. Close an issue via IssueDetailScreen <br> 2. Return to KanbanScreen <br> 3. Verify the closed issue does not appear in any column |
| **Expected Result** | Closed issues do not appear on the Kanban board. Total card count decreases. |
| **Priority** | High |

#### TC-PA-K08: Back to Dashboard navigation
| Field | Value |
|-------|-------|
| **Precondition** | KanbanScreen loaded |
| **Steps** | 1. Click "Back to Dashboard" button |
| **Expected Result** | Returns to Dashboard Screen. |
| **Priority** | Low |

---

### 3.9 — End-to-End Lifecycle Test

#### TC-E2E-01: Full lifecycle — new issue from intake to closure
| Field | Value |
|-------|-------|
| **Precondition** | App is running, user is authenticated |
| **Steps** | 1. **Submit:** On Dashboard, click "+ New Issue", fill in: Title="E2E Test Issue", Owner="Test User", Priority=High, Description="End-to-end lifecycle test" <br> 2. **Verify intake:** Confirm item appears in intake gallery as Pending <br> 3. **Promote:** Click "Move to Tracker" on the new intake item <br> 4. **Verify tracker:** Navigate to Tracker, find the new issue, verify Status=New, green staleness <br> 5. **First update:** Tap the issue, add update: "Initial assessment underway", change Status to Active <br> 6. **Verify update:** Confirm timeline shows the update, header shows Active <br> 7. **Second update:** Add another update: "Assessment complete, monitoring required", change Status to Monitoring <br> 8. **Verify:** Timeline shows 3 entries (promotion + 2 updates), header shows Monitoring <br> 9. **Close:** Add final update: "Issue resolved and closed", change Status to Closed <br> 10. **Verify closure:** Issue no longer appears in "All Open" filter on Tracker. Dashboard Open Items count decreases. |
| **Expected Result** | Complete lifecycle works: Submit → Promote → Update (x2) → Close. All data consistent across SharePoint lists, Power Apps screens, and KPIs. |
| **Priority** | Critical |

#### TC-E2E-02: Staleness progression over time
| Field | Value |
|-------|-------|
| **Precondition** | An active issue exists with recent LastUpdated |
| **Steps** | 1. Note an issue's current DaysSinceUpdate and StalenessFlag <br> 2. Wait for staleness flow to run (or trigger manually) <br> 3. Verify DaysSinceUpdate incremented <br> 4. If days cross a threshold (7→8 or 14→15), verify StalenessFlag changes <br> 5. Verify color changes on Tracker Screen |
| **Expected Result** | Staleness progresses: Current (green) → Aging (amber) → Stale (red) as days increase. Colors in Power Apps match the flags in SharePoint. |
| **Priority** | High |

#### TC-E2E-03: Full lifecycle with FunctionalGroup — submit through Kanban and Group Allocation
| Field | Value |
|-------|-------|
| **Precondition** | App is running, user is authenticated |
| **Steps** | 1. **Submit:** Navigate to SubmitScreen, enter Title="E2E FunctionalGroup Test", Priority=High, select FunctionalGroup="Internal Audit", Description="Testing FunctionalGroup flow" <br> 2. **Verify intake:** Open OFR_IntakeQueue in SharePoint, confirm FunctionalGroup = "Internal Audit" <br> 3. **Accept:** On Dashboard, click the intake item, accept it into the tracker <br> 4. **Verify issue:** Open OFR_Issues in SharePoint, confirm FunctionalGroup = "Internal Audit" on the new issue <br> 5. **Verify Tracker:** Navigate to TrackerScreen, verify the Group column shows "Internal Audit" <br> 6. **Verify Detail:** Tap the issue, verify FunctionalGroup displayed in header <br> 7. **Verify Group Allocation:** Navigate to GroupAllocationScreen, verify "Internal Audit" card count increased by 1 <br> 8. **Verify Kanban:** Navigate to KanbanScreen, find the new issue in the "New" column, verify FunctionalGroup shows on the card |
| **Expected Result** | FunctionalGroup flows end-to-end: Submit → IntakeQueue → Accept → OFR_Issues → visible on Tracker, Detail, Group Allocation, and Kanban screens. |
| **Priority** | Critical |

---

### 3.10 — Edge Cases & Negative Tests

#### TC-NEG-01: Search with no results
| Field | Value |
|-------|-------|
| **Precondition** | Tracker Screen loaded |
| **Steps** | 1. Type "ZZZZZ" in search box |
| **Expected Result** | Empty gallery or "No items found" message. No errors. |
| **Priority** | Low |

#### TC-NEG-02: Rapid successive updates
| Field | Value |
|-------|-------|
| **Precondition** | Issue Detail Screen loaded |
| **Steps** | 1. Add update and save <br> 2. Immediately add another update and save |
| **Expected Result** | Both updates saved correctly. Timeline shows both entries. No data corruption or duplicate records. |
| **Priority** | Medium |

#### TC-NEG-03: Long text in notes
| Field | Value |
|-------|-------|
| **Precondition** | Issue Detail Screen loaded |
| **Steps** | 1. Enter a very long note (500+ characters) <br> 2. Save the update |
| **Expected Result** | Note saves completely. Timeline displays the full text (with scrolling if needed). |
| **Priority** | Low |

#### TC-NEG-04: Special characters in text fields
| Field | Value |
|-------|-------|
| **Precondition** | Dashboard or Issue Detail Screen |
| **Steps** | 1. Submit an intake item or update with special characters: é, ñ, &, <, >, "quotes", $dollars |
| **Expected Result** | All characters saved and displayed correctly. No HTML encoding issues. |
| **Priority** | Low |

#### TC-NEG-05: Concurrent access
| Field | Value |
|-------|-------|
| **Precondition** | Two browser sessions open to the same app |
| **Steps** | 1. User A opens OFR-1 detail <br> 2. User B opens OFR-1 detail <br> 3. User A adds an update <br> 4. User B adds a different update |
| **Expected Result** | Both updates saved. No data loss. Both entries appear in timeline (may require refresh). |
| **Priority** | Medium |

---

### 3.11 — Access & Authentication

#### TC-AUTH-01: SSO authentication
| Field | Value |
|-------|-------|
| **Precondition** | User is logged into M365 |
| **Steps** | 1. Open the Power Apps app URL <br> 2. Verify automatic sign-in via Entra ID |
| **Expected Result** | No separate login required. App loads with user context. |
| **Priority** | High |

#### TC-AUTH-02: Unauthorized user access
| Field | Value |
|-------|-------|
| **Precondition** | User NOT in the OFR SharePoint site members group |
| **Steps** | 1. Attempt to open the app |
| **Expected Result** | Access denied or data connection error. User cannot view or modify data. |
| **Priority** | Medium |

---

## 4. Test Execution Summary Template

| Test ID | Description | Status | Tester | Date | Notes |
|---------|-------------|--------|--------|------|-------|
| TC-SP-01 | OFR_Issues schema | | | | |
| TC-SP-02 | OFR_UpdateHistory schema | | | | |
| TC-SP-03 | OFR_IntakeQueue schema | | | | |
| TC-SP-04 | Sample data integrity | | | | |
| TC-PA-D01 | Dashboard KPIs | | | | |
| TC-PA-D02 | Intake queue display | | | | |
| TC-PA-D03 | New Issue form | | | | |
| TC-PA-D04 | Dismiss intake | | | | |
| TC-PA-D05 | Navigate to Tracker | | | | |
| TC-PA-D06 | Intake panel opens on click | | | | |
| TC-PA-D07 | Accept intake via panel | | | | |
| TC-PA-D08 | Reject intake via panel | | | | |
| TC-PA-D09 | Close panel without action | | | | |
| TC-PA-D10 | Panel hidden on load | | | | |
| TC-PA-S01 | Navigate to Submit Screen | | | | |
| TC-PA-S02 | Submit a new issue | | | | |
| TC-PA-S03 | Back from Submit Screen | | | | |
| TC-PA-T01 | Issue table display | | | | |
| TC-PA-T02 | Staleness colors | | | | |
| TC-PA-T03 | Filter — All Open | | | | |
| TC-PA-T04 | Filter — Stale | | | | |
| TC-PA-T05 | Filter — High | | | | |
| TC-PA-T06 | Filter — Medium | | | | |
| TC-PA-T07 | Filter — Low | | | | |
| TC-PA-T08 | Search by Title | | | | |
| TC-PA-T09 | Search by Owner | | | | |
| TC-PA-T10 | Search by ItemID | | | | |
| TC-PA-T11 | Combined filter + search | | | | |
| TC-PA-T12 | Column sort | | | | |
| TC-PA-T13 | Navigate to Detail | | | | |
| TC-PA-T14 | Back to Dashboard | | | | |
| TC-PA-I01 | Issue header display | | | | |
| TC-PA-I02 | Update history timeline | | | | |
| TC-PA-I03 | Multiple update entries | | | | |
| TC-PA-I04 | Add update — notes only | | | | |
| TC-PA-I05 | Add update — status change | | | | |
| TC-PA-I06 | Validation — empty notes | | | | |
| TC-PA-I07 | Update reflects in SP | | | | |
| TC-PA-I08 | Back to Tracker | | | | |
| TC-FL-S01 | Staleness — manual run | | | | |
| TC-FL-S02 | DaysSinceUpdate calc | | | | |
| TC-FL-S03 | StalenessFlag accuracy | | | | |
| TC-FL-S04 | Closed issues excluded | | | | |
| TC-FL-S05 | Flow after update | | | | |
| TC-FL-P01 | Promote intake item | | | | |
| TC-FL-P02 | Verify promoted issue | | | | |
| TC-FL-P03 | Verify audit trail | | | | |
| TC-FL-P04 | Intake marked Promoted | | | | |
| TC-FL-P05 | NewItemID returned | | | | |
| TC-FL-P06 | KPIs update | | | | |
| TC-E2E-01 | Full lifecycle test | | | | |
| TC-E2E-02 | Staleness progression | | | | |
| TC-E2E-03 | FunctionalGroup lifecycle | | | | |
| TC-PA-G01 | Group Allocation loads | | | | |
| TC-PA-G02 | Group card counts | | | | |
| TC-PA-G03 | Closed excluded from groups | | | | |
| TC-PA-G04 | Unassigned count | | | | |
| TC-PA-G05 | Total active count | | | | |
| TC-PA-G06 | Back from Group Allocation | | | | |
| TC-PA-K01 | Kanban Board loads | | | | |
| TC-PA-K02 | Issues in correct columns | | | | |
| TC-PA-K03 | Card content display | | | | |
| TC-PA-K04 | Card tap to detail | | | | |
| TC-PA-K05 | Column counts match | | | | |
| TC-PA-K06 | Staleness colours on cards | | | | |
| TC-PA-K07 | Closed excluded from Kanban | | | | |
| TC-PA-K08 | Back from Kanban | | | | |
| TC-PA-T15 | FunctionalGroup column | | | | |
| TC-PA-T16 | Search by group | | | | |
| TC-PA-I09 | FunctionalGroup on detail | | | | |
| TC-PA-S04 | FunctionalGroup dropdown | | | | |
| TC-PA-S05 | FunctionalGroup saved | | | | |
| TC-PA-D11 | FunctionalGroup acceptance | | | | |
| TC-PA-D12 | Nav to Group Allocation | | | | |
| TC-PA-D13 | Nav to Kanban Board | | | | |
| TC-SP-05 | FunctionalGroup on Issues | | | | |
| TC-SP-06 | FunctionalGroup on Intake | | | | |
| TC-NEG-01 | Search no results | | | | |
| TC-NEG-02 | Rapid updates | | | | |
| TC-NEG-03 | Long text | | | | |
| TC-NEG-04 | Special characters | | | | |
| TC-NEG-05 | Concurrent access | | | | |
| TC-AUTH-01 | SSO authentication | | | | |
| TC-AUTH-02 | Unauthorized access | | | | |

---

## 5. Test Priority Summary

| Priority | Count | Description |
|----------|-------|-------------|
| Critical | 2 | Full end-to-end lifecycle (TC-E2E-01, TC-E2E-03) |
| High | 44 | Core functionality — KPIs, filters, updates, flows, data integrity, intake panel, submit screen, group allocation, kanban board, FunctionalGroup pipeline |
| Medium | 19 | Secondary features — sorting, combined filters, concurrent access, panel states, card counts, staleness colours |
| Low | 11 | Edge cases — empty search, long text, special chars, navigation, back buttons |
| **Total** | **76** | |

### Recommended Test Order
1. **SharePoint data layer** (TC-SP-01 to TC-SP-06) — confirm foundation including FunctionalGroup
2. **Power Automate flows** (TC-FL-S01 to TC-FL-S05, TC-FL-P01 to TC-FL-P06) — confirm automation
3. **Power Apps core screens** (TC-PA-D01 to TC-PA-I09, TC-PA-S01 to TC-PA-S05) — confirm UI including FunctionalGroup
4. **Power Apps new screens** (TC-PA-G01 to TC-PA-G06, TC-PA-K01 to TC-PA-K08) — confirm Group Allocation and Kanban
5. **End-to-end** (TC-E2E-01, TC-E2E-02, TC-E2E-03) — confirm full lifecycle including FunctionalGroup flow
6. **Edge cases & auth** (TC-NEG-01 to TC-AUTH-02) — confirm robustness
