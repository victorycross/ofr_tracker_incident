# Enterprise Incident Tracker — Phase 2 Integration Test Plan

**Version:** 1.0
**Date:** February 2026
**Classification:** Internal

---

## Overview

This test plan covers acceptance testing for the five Phase 2 integrations:

1. ServiceNow → EIT ingest flow
2. iSight → EIT polling flow
3. Power BI Executive Dashboard
4. Teams Channel Notifications (escalation flow extension)
5. Automated Archiving

**Prerequisites:** Phase 1 must pass all go/no-go criteria (see `EIT-Test-Plan.md`) before executing Phase 2 tests.

---

## Test Environment Requirements

| Component | Requirement |
|-----------|-------------|
| ServiceNow | Test instance with outbound REST notification configured to Phase 2 HTTP trigger URL |
| iSight | API key with access to test alerts, or synthetic payload capability |
| Power BI Desktop | Installed and connected to EIT SharePoint site |
| Power BI Service | Workspace created; Power BI Pro licenses allocated |
| Microsoft Teams | EIT-Escalations channel created; flow service account is channel member |
| EIT_Incidents_Archive | List created per `EIT_Incidents_Archive-schema.json` |
| Test incidents | At least 5 closed EIT_Incidents items with `LastUpdated` set to > 365 days ago |

---

## Category 1: ServiceNow Ingest Flow

### TC-SN-01: New P1 Incident Creates EIT Record

**Priority:** Critical

**Precondition:**
- `EIT ServiceNow Ingest` flow is enabled and connected
- `EIT_Incidents` does not contain an item with `RelatedItemIDs = INC0099001`

**Steps:**
1. POST the following payload to the flow HTTP trigger URL:
   ```json
   {"sys_id":"test-sn-01","number":"INC0099001","priority":"1 - Critical","category":"Security","assignment_group":"Security Operations","short_description":"Test P1 Critical Incident","description":"Test incident raised by Phase 2 acceptance testing.","caller_id":"Test Tester","opened_at":"2026-02-24T10:00:00Z","state":"In Progress","severity":"1 - High"}
   ```
2. Wait 30 seconds for the flow to complete
3. Open `EIT_Incidents` in SharePoint

**Expected Result:**
- New item exists with:
  - Title: `Test P1 Critical Incident`
  - Severity: `Critical`
  - Domain: `NIS` (mapped from Security Operations)
  - SourceSystem: `ServiceNow`
  - RelatedItemIDs: `INC0099001`
  - ItemID: `EIT-NNN` format (not `EIT-PENDING`)
  - Status: `New`
  - EscalationThreshold: `Not Escalated`
- New opening entry in `EIT_IncidentHistory` for this ItemID, Notes containing `INC0099001`
- HTTP response from flow: 200 OK with `eit_item_id` in body

---

### TC-SN-02: New P2 Incident Creates EIT Record with High Severity

**Priority:** High

**Precondition:** `EIT_Incidents` does not contain `RelatedItemIDs = INC0099002`

**Steps:**
1. POST payload with `"priority": "2 - High"`, `"number": "INC0099002"`, `"assignment_group": "Privacy & Compliance"`
2. Wait 30 seconds and check `EIT_Incidents`

**Expected Result:**
- New item with Severity = `High`, Domain = `Privacy & Technology`, RelatedItemIDs = `INC0099002`

---

### TC-SN-03: Duplicate ServiceNow Ticket Does Not Create Second Record

**Priority:** Critical

**Precondition:** TC-SN-01 completed — `INC0099001` exists in `EIT_Incidents`

**Steps:**
1. POST the same payload as TC-SN-01 (same `number: INC0099001`)
2. Wait 30 seconds
3. Check `EIT_Incidents` — count items with `RelatedItemIDs = INC0099001`

**Expected Result:**
- Only 1 item with `RelatedItemIDs = INC0099001` (no duplicate)
- Existing item's `Severity` and `LastUpdated` updated to reflect the new payload values
- New `EIT_IncidentHistory` entry added noting the ServiceNow update

---

### TC-SN-04: ServiceNow Status Update Syncs to EIT

**Priority:** High

**Precondition:** TC-SN-01 completed

**Steps:**
1. POST a payload for `INC0099001` with `"priority": "2 - High"` (downgraded)
2. Wait 30 seconds
3. Check the existing EIT item for `INC0099001`

**Expected Result:**
- Existing item's `Severity` changed from `Critical` to `High`
- `LastUpdated` reflects current timestamp
- New history entry documenting the update

---

### TC-SN-05: Unmapped Assignment Group Uses Default Domain

**Priority:** Medium

**Precondition:** The assignment group `Unknown Group XYZ` is not in the flow's Switch mapping

**Steps:**
1. POST payload with `"assignment_group": "Unknown Group XYZ"`, `"number": "INC0099003"`
2. Wait 30 seconds and check `EIT_Incidents`

**Expected Result:**
- Item created with the default domain (e.g., `NIS`) — as configured in the Switch default case
- No flow failure; flow completes with run status = Succeeded

---

### TC-SN-06: Flow Responds HTTP 200 with EIT ItemID

**Priority:** High

**Steps:**
1. POST a new unique payload and capture the HTTP response body
2. Inspect the response

**Expected Result:**
- HTTP status: `200 OK`
- Response body contains `{"status": "accepted", "eit_item_id": "EIT-NNN"}`
- The returned `EIT-NNN` matches the ItemID created in `EIT_Incidents`

---

## Category 2: iSight Polling Flow

### TC-IS-01: New Alert Creates EIT Incident

**Priority:** Critical

**Precondition:**
- Flow's HTTP action temporarily replaced with a Compose returning a static JSON array (see `EIT-iSight-Polling-Flow.md` Step 10)
- Static payload: alertId = `TEST-ISIGHT-001`, severity = `HIGH`
- `EIT_Incidents` does not contain `RelatedItemIDs = TEST-ISIGHT-001`

**Steps:**
1. Run the flow manually (Test → Manually)
2. Wait for completion
3. Check `EIT_Incidents`

**Expected Result:**
- New item: Domain = `NIS`, IncidentType = `Threat`, Severity = `High`, SourceSystem = `iSight`, RelatedItemIDs = `TEST-ISIGHT-001`
- ItemID set to `EIT-NNN` format
- Opening history entry with Notes containing `TEST-ISIGHT-001`

---

### TC-IS-02: Critical Alert Maps to Critical Severity

**Priority:** High

**Precondition:** Static payload with `"severity": "CRITICAL"`, `"alertId": "TEST-ISIGHT-002"`

**Steps:**
1. Run flow manually with critical severity payload
2. Check EIT_Incidents

**Expected Result:**
- New item with `Severity = Critical`

---

### TC-IS-03: Below-Threshold Alert Is Not Ingested

**Priority:** High

**Precondition:** Static payload with `"severity": "LOW"`, `"alertId": "TEST-ISIGHT-003"`

**Steps:**
1. Run flow manually
2. Check `EIT_Incidents`

**Expected Result:**
- No item created with `RelatedItemIDs = TEST-ISIGHT-003`
- Flow run succeeds (no error); Filter array step returns 0 items; Apply to each does not execute

---

### TC-IS-04: Duplicate Alert Not Ingested Twice

**Priority:** Critical

**Precondition:** TC-IS-01 completed — `TEST-ISIGHT-001` exists in EIT_Incidents

**Steps:**
1. Run flow again with same `alertId = TEST-ISIGHT-001`
2. Check `EIT_Incidents` for duplicate

**Expected Result:**
- No duplicate item — only 1 item with `RelatedItemIDs = TEST-ISIGHT-001`
- If no action: the If no branch (empty) ran for this alert — verify in flow run history

---

### TC-IS-05: Scheduled Flow Runs Every 15 Minutes

**Priority:** Medium

**Steps:**
1. Enable the flow
2. Wait 30 minutes
3. Check flow run history

**Expected Result:**
- At least 2 run entries visible in run history, approximately 15 minutes apart
- All runs show status = Succeeded (or no qualifying alerts)

---

## Category 3: Power BI Executive Dashboard

### TC-PBI-01: EIT_Incidents Data Loads Without Error

**Priority:** Critical

**Steps:**
1. Open `EIT-Dashboard.pbix` in Power BI Desktop
2. Click Refresh

**Expected Result:**
- No data load errors
- `EIT_Incidents` table shows correct row count (matching SharePoint list count)
- `EIT_IncidentHistory` table loads correctly
- Relationship between the two tables is intact (check Model view)

---

### TC-PBI-02: Executive Overview Page Counts Match Power App

**Priority:** Critical

**Precondition:** EIT Power App Executive Report Screen is open showing current counts

**Steps:**
1. Note the Open, Critical, Escalated, Stale counts from the Power App Executive Report Screen
2. Open Power BI Desktop, refresh data
3. Navigate to the Executive Overview page
4. Note the KPI card values

**Expected Result:**
- Power BI Open count = Power App Total Open (±0, same moment in time)
- Power BI Critical count = Power App Total Critical
- Power BI Escalated count = Power App Total Escalated
- Power BI Stale count = Power App Stale count

> **Timing note:** Power BI uses the last dataset refresh timestamp, not live data. Minor discrepancies are expected if incidents were added/updated between the last Power BI refresh and the Power App snapshot. For this test, compare immediately after a Power BI data refresh.

---

### TC-PBI-03: P&C Incidents Excluded for Standard Users (RLS)

**Priority:** Critical

**Precondition:**
- At least 1 incident with `PrivilegedAndConfidential = Yes` exists in `EIT_Incidents`
- RLS `Standard_Users` role is configured per `EIT-PowerBI-RLS-Guide.md`
- A test Standard User account is available (member of EIT Members group, assigned to `Standard_Users` RLS role)

**Steps:**
1. In Power BI Service, use **Test as role** → `Standard_Users`
2. Navigate to all 5 report pages
3. Note total incident counts
4. Remove the role test and check counts as admin

**Expected Result:**
- As `Standard_Users`: P&C incidents are NOT visible in any table or counted in any KPI
- As admin: P&C incidents ARE counted
- The difference in total counts equals the number of P&C incidents

---

### TC-PBI-04: P&C Authorized Users See All Incidents

**Priority:** Critical

**Precondition:**
- A test user who is a member of `EIT-PAC-Authorized` M365 group and is NOT assigned to `Standard_Users` RLS role
- At least 1 P&C incident exists

**Steps:**
1. Sign into Power BI Service as the P&C authorized test user
2. Open the EIT dashboard
3. Check total incident counts

**Expected Result:**
- P&C incidents are visible and counted
- Counts match the admin view

---

### TC-PBI-05: Scheduled Refresh Runs Successfully

**Priority:** High

**Steps:**
1. After publishing to Power BI Service, wait for the next scheduled refresh (06:00 or 18:00)
2. Check Power BI Service → EIT-Dashboard dataset → Refresh history

**Expected Result:**
- Last refresh shows `Completed` status
- Refresh timestamp is within the expected window
- No credential errors

---

### TC-PBI-06: Mean Time to Contain Calculates Correctly

**Priority:** High

**Precondition:** At least 3 incidents exist with `Status = Contained` and `DateContained` populated

**Steps:**
1. Manually calculate the expected MTTC:
   - For each contained incident: `DateContained - DateRaised` in days
   - Average across all contained incidents
2. Open Power BI Resolution Metrics page
3. Note the `Mean Time to Contain (Days)` card value

**Expected Result:**
- Power BI value matches manually calculated average (±1 day for rounding)

---

### TC-PBI-07: Teams Embed Renders Correctly

**Priority:** Medium

**Steps:**
1. Add the Power BI report as a tab in the EIT Teams channel (or SharePoint webpart)
2. Open the Teams tab
3. Navigate all 5 pages

**Expected Result:**
- Report renders without error
- All pages are navigable
- Slicers and filters function (Domain slicer on Page 4 filters correctly)
- No "Access denied" or license error messages

---

## Category 4: Teams Channel Notifications

### TC-TN-01: Escalation Alert Posts to Teams Channel

**Priority:** Critical

**Precondition:**
- `EIT Domain Escalation Notifier` flow updated with Teams actions
- `EIT-Escalations` Teams channel exists and flow service account is a member
- A test incident exists in `EIT_Incidents`

**Steps:**
1. Open the EIT Power App
2. Navigate to the test incident's Issue Detail screen
3. Select UpdateType = Escalation, EscalationThreshold = Escalate
4. Add a note and click Save Update
5. Check the `EIT-Escalations` Teams channel

**Expected Result:**
- Adaptive card appears in `EIT-Escalations` within 60 seconds
- Card shows: ⚠ ESCALATION ALERT header, correct Incident ID, Title, Domain, Severity, Escalation (Escalate), Owner, Days Since Update
- Existing escalation email is also sent (original behaviour preserved)
- `EIT_IncidentHistory` entry created for the update

---

### TC-TN-02: Executive Brief Posts Correct Card

**Priority:** Critical

**Steps:**
1. Trigger an update with EscalationThreshold = Executive Brief on a test incident
2. Check the Teams channel

**Expected Result:**
- Card shows ⚠ EXECUTIVE BRIEF header (not ESCALATION ALERT)
- Email sent to EscalationEmail (not LeadEmail)

---

### TC-TN-03: "Open in EIT" Button Navigates to Power App

**Priority:** High

**Steps:**
1. Click the "Open in EIT" action button on an adaptive card in Teams
2. Observe the destination

**Expected Result:**
- Power App opens in browser (or Teams app if configured)
- No "page not found" or authentication error

---

### TC-TN-04: Card Does Not Appear for Non-Escalation Updates

**Priority:** Medium

**Steps:**
1. Update a test incident with UpdateType = Progress Update (not Escalation)
2. Check the Teams channel

**Expected Result:**
- No new Teams card posted (Teams notification only triggers on Escalation update type)
- History entry is created, email is NOT sent (non-escalation updates do not trigger email or Teams)

---

### TC-TN-05: P&C Escalation Card Does Not Expose Sensitive Content

**Priority:** Critical

**Precondition:** A P&C incident exists (`PrivilegedAndConfidential = Yes`)

**Steps:**
1. Escalate the P&C incident via Issue Detail screen
2. Check the Teams channel card

**Expected Result:**
- Adaptive card posts to channel
- Card does NOT display the incident title or any free-text content — it shows only the Incident ID, Domain, Severity, and Escalation Level
- The escalation email subject includes `[PRIVILEGED & CONFIDENTIAL]` prefix

> **Design note:** The adaptive card in Phase 2 posts the `Title` field, which for P&C incidents may contain sensitive information. If the Teams channel is broadly accessible, consider masking the title for P&C incidents. Update the adaptive card JSON to show `[P&C — RESTRICTED]` instead of the real title when `PrivilegedAndConfidential = true`. This is a configuration decision for the EIT administrator.

---

## Category 5: Automated Archiving

### TC-AR-01: Closed Old Incident Is Archived

**Priority:** Critical

**Precondition:**
- `EIT_Incidents_Archive` list exists with correct schema
- At least 1 item in `EIT_Incidents` with Status = Closed AND `LastUpdated` date manually set to > 365 days ago
- Note the ItemID of this test item

**Steps:**
1. Trigger `EIT Monthly Archive` manually (Test → Manually)
2. Wait for flow completion (may take several minutes if processing many items)
3. Check `EIT_Incidents_Archive`
4. Check `EIT_Incidents`

**Expected Result:**
- Test item appears in `EIT_Incidents_Archive` with all fields correctly copied
- `ArchivedDate` set to today's date
- Test item no longer exists in `EIT_Incidents`

---

### TC-AR-02: Active Incidents Are Not Archived

**Priority:** Critical

**Precondition:** Active incidents (Status ≠ Closed) exist in `EIT_Incidents`

**Steps:**
1. Run the archive flow manually
2. Check that active incidents remain in `EIT_Incidents`

**Expected Result:**
- No Active, New, Monitoring, Escalated, or Contained incidents appear in `EIT_Incidents_Archive`
- These incidents remain in `EIT_Incidents` unchanged

---

### TC-AR-03: Recently Closed Incidents Are Not Archived

**Priority:** Critical

**Precondition:** An incident with Status = Closed and `LastUpdated` within the last 12 months exists

**Steps:**
1. Run the archive flow manually
2. Check whether this recently-closed incident is archived

**Expected Result:**
- Item is NOT archived — it remains in `EIT_Incidents`
- `LastUpdated` filter correctly excludes items within the 365-day window

---

### TC-AR-04: EIT_IncidentHistory Entries Preserved After Archive

**Priority:** High

**Precondition:** TC-AR-01 completed — the test item was archived

**Steps:**
1. Check `EIT_IncidentHistory` for entries with `ParentItemID` matching the archived incident's ItemID

**Expected Result:**
- `EIT_IncidentHistory` entries for the archived incident still exist (history is not deleted)
- History entries remain available for audit trail access

---

### TC-AR-05: Admin Summary Email Received

**Priority:** Medium

**Steps:**
1. Run the archive flow manually
2. Check the `david@brightpathtechnology.io` inbox

**Expected Result:**
- Email received with subject containing "EIT Monthly Archive Complete"
- Email body shows the correct archive count
- Archive count matches the number of items moved during this run

---

## Category 6: Row-Level Security

### TC-RLS-01: Standard User Cannot See P&C Incidents in Power BI

*See TC-PBI-03 above — this is the same test, critical-priority duplicate included for RLS tracking.*

---

### TC-RLS-02: P&C Incident Count Difference Is Correct

**Priority:** High

**Precondition:**
- Exact count of P&C incidents is known (from EIT admin or Power App Exec Report Screen)
- Admin Power BI view count and Standard User Power BI view count are both available

**Steps:**
1. As admin: note Total Open count in Power BI
2. As Standard User (using Test as role): note Total Open count
3. Calculate: Admin count − Standard User count

**Expected Result:**
- Difference = exact number of P&C active incidents

---

### TC-RLS-03: RLS Persists After Report Republication

**Priority:** High

**Steps:**
1. Make a minor change to the PBIX (e.g., update a chart title)
2. Re-publish to Power BI Service (overwrite existing)
3. After publishing, check dataset Security settings in Power BI Service

**Expected Result:**
- `Standard_Users` RLS role still has `EIT Members` and `EIT Viewers` groups assigned
- P&C filtering still functions (verify with TC-RLS-01)

---

## Test Execution Summary

| Test Case | Category | Priority | Pass/Fail | Notes |
|-----------|----------|----------|-----------|-------|
| TC-SN-01 | ServiceNow Ingest | Critical | | |
| TC-SN-02 | ServiceNow Ingest | High | | |
| TC-SN-03 | ServiceNow Ingest | Critical | | |
| TC-SN-04 | ServiceNow Ingest | High | | |
| TC-SN-05 | ServiceNow Ingest | Medium | | |
| TC-SN-06 | ServiceNow Ingest | High | | |
| TC-IS-01 | iSight Polling | Critical | | |
| TC-IS-02 | iSight Polling | High | | |
| TC-IS-03 | iSight Polling | High | | |
| TC-IS-04 | iSight Polling | Critical | | |
| TC-IS-05 | iSight Polling | Medium | | |
| TC-PBI-01 | Power BI | Critical | | |
| TC-PBI-02 | Power BI | Critical | | |
| TC-PBI-03 | Power BI | Critical | | |
| TC-PBI-04 | Power BI | Critical | | |
| TC-PBI-05 | Power BI | High | | |
| TC-PBI-06 | Power BI | High | | |
| TC-PBI-07 | Power BI | Medium | | |
| TC-TN-01 | Teams Notifications | Critical | | |
| TC-TN-02 | Teams Notifications | Critical | | |
| TC-TN-03 | Teams Notifications | High | | |
| TC-TN-04 | Teams Notifications | Medium | | |
| TC-TN-05 | Teams Notifications | Critical | | |
| TC-AR-01 | Archiving | Critical | | |
| TC-AR-02 | Archiving | Critical | | |
| TC-AR-03 | Archiving | Critical | | |
| TC-AR-04 | Archiving | High | | |
| TC-AR-05 | Archiving | Medium | | |
| TC-RLS-01 | RLS | Critical | | |
| TC-RLS-02 | RLS | High | | |
| TC-RLS-03 | RLS | High | | |

---

## Phase 2 Go / No-Go Criteria

Phase 2 integrations must meet the following criteria before each integration is considered production-ready:

| Integration | Go Criteria |
|-------------|-------------|
| ServiceNow Ingest | TC-SN-01, TC-SN-03, TC-SN-06 all Pass |
| iSight Polling | TC-IS-01, TC-IS-04 Pass |
| Power BI Dashboard | TC-PBI-01, TC-PBI-02, TC-PBI-03, TC-PBI-04 all Pass |
| Teams Notifications | TC-TN-01, TC-TN-02, TC-TN-05 all Pass |
| Automated Archiving | TC-AR-01, TC-AR-02, TC-AR-03 all Pass |
| **Overall Phase 2** | All Critical-priority test cases Pass; all RLS tests Pass |

**Blocking condition:** TC-PBI-03 and TC-PBI-04 (P&C RLS tests) are absolute blockers. The Power BI dashboard must not go live if P&C incident exclusion cannot be verified for Standard Users.

---

## Defect Severity Classification

| Severity | Definition | Example |
|----------|-----------|---------|
| **Critical** | Integration fails completely, data loss risk, or P&C data exposure | Duplicate incidents created; P&C data visible to Standard Users |
| **High** | Integration partially works but key fields are wrong | Severity mapped incorrectly; Teams card missing key fields |
| **Medium** | Minor functional issue with workaround | Admin email not sent; scheduled refresh not running on time |
| **Low** | Cosmetic or non-functional issue | Card formatting; chart colour incorrect |
