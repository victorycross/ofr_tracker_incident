# Enterprise Incident Tracker — Disaster Recovery Guide

**Version:** 1.0
**Date:** February 2026
**Audience:** EIT administrator, M365 platform administrator
**Classification:** Internal — Restricted

---

## Overview

This guide covers backup, restoration, and disaster recovery procedures for all Enterprise Incident Tracker components. The EIT is a cloud-native system hosted entirely in Microsoft 365; its primary resilience depends on Microsoft's platform SLAs. This guide addresses the additional steps the EIT administrator controls.

### Recovery Objectives

| Scenario | Recovery Time Objective (RTO) | Recovery Point Objective (RPO) |
|----------|------------------------------|-------------------------------|
| Single flow failure | 2 hours | 0 (reprocess from source) |
| Power App unavailable | 4 hours | 0 (data in SharePoint is unaffected) |
| SharePoint list data corruption | 8 hours | 24 hours (last backup) |
| SharePoint site deleted | 24 hours | 24 hours (last backup) |
| Full M365 tenant outage | Per Microsoft SLA (99.9% uptime) | Per Microsoft SLA |

---

## Section 1: Backup Procedures

### 1.1 SharePoint Lists — Manual CSV Export

**Frequency:** Weekly (every Friday afternoon, or before any major EIT change)

**Procedure:**

For each of the 5 EIT lists, export to CSV:

1. Navigate to the list in SharePoint
2. Click **Export to Excel** in the list toolbar (or Export → Export to CSV)
3. Save the file with the naming convention: `[ListName]-backup-[YYYY-MM-DD].csv`
   - Example: `EIT_Incidents-backup-2026-02-21.csv`
4. Save to: `[Admin OneDrive]/EIT-Backups/[YYYY-MM]/`

**Lists to back up:**
- `EIT_Incidents`
- `EIT_IncidentHistory`
- `EIT_IntakeQueue`
- `EIT_DomainContacts`
- `EIT_Incidents_Archive`

> **P&C note:** CSV exports contain all incident data including P&C incidents. Store backup files in a restricted location accessible only to the EIT administrator and EIT Owners. Do not share backup files broadly.

### 1.2 SharePoint Lists — Automated Export (Optional Enhancement)

For automated weekly backups, create a Power Automate flow:

**Flow: EIT Weekly Backup**
- **Trigger:** Recurrence (weekly, Friday 16:00)
- **For each list:**
  - Get items (all items)
  - Compose → JSON array
  - Create file in SharePoint document library (`/EIT-Backups/[list-name]-[date].json`)
- **After all lists:** Send admin notification confirming backup completion

### 1.3 Power Apps — Export the App Package

**Frequency:** After every significant app update; minimum monthly

**Procedure:**

1. Navigate to `https://make.powerapps.com`
2. Select the correct environment (top-right dropdown)
3. Find **Enterprise Incident Tracker** in the Apps list
4. Click the three-dot menu → **Export package**
5. Configure:
   - **Name:** `EIT-PowerApp-[YYYY-MM-DD]`
   - **Export content:** App only (not connections)
   - For each connection resource: select **Create as new**
6. Click **Export** → save the `.zip` file
7. Store in: `[Admin OneDrive]/EIT-Backups/PowerApp/`

> **What the package contains:** The `.zip` contains the full Power Apps canvas app definition — all screens, controls, formulas, media, and layout. It does not contain SharePoint data or flow definitions.

### 1.4 Power Automate Flows — Export Flow Packages

**Frequency:** After every flow modification; minimum monthly

**Procedure:**

For each EIT flow:

1. Navigate to `https://make.powerautomate.com` → **My flows**
2. Open the flow
3. Click the three-dot menu → **Export** → **Package (.zip)**
4. Configure:
   - **Name:** `[FlowName]-[YYYY-MM-DD]`
   - For connections: **Create as new** (connections are re-established on import)
5. Click **Export** → save the `.zip`
6. Store in: `[Admin OneDrive]/EIT-Backups/Flows/`

**Flows to back up:**
- `EIT Daily Staleness & Escalation Calculator`
- `EIT Intake Promotion`
- `EIT Domain Escalation Notifier`
- `EIT ServiceNow Ingest`
- `EIT iSight Alert Polling`
- `EIT Monthly Archive`
- `EIT Monthly Compliance Report`
- `EIT Quarterly Access Review Reminder`

### 1.5 Power BI Report — Download PBIX

**Frequency:** After every report update; minimum monthly

**Procedure:**

1. Navigate to `https://app.powerbi.com`
2. Open the **Enterprise Incident Tracker** workspace
3. Find the **EIT-Dashboard** report
4. Click the three-dot menu → **Download this file**
5. Save the `.pbix` file to: `[Admin OneDrive]/EIT-Backups/PowerBI/EIT-Dashboard-[YYYY-MM-DD].pbix`

> **What the PBIX contains:** The full Power BI report including data model, DAX measures, all page layouts, and RLS role definitions. It does not contain the live SharePoint data — the report reconnects to SharePoint on first refresh.

### 1.6 Backup Retention Schedule

| Backup Type | Retention | Notes |
|-------------|-----------|-------|
| CSV exports (weekly) | 12 months rolling | Keep 4 weeks at daily frequency during high-change periods |
| Power App packages | Last 5 versions | Keep all versions from the last 6 months |
| Flow packages | Last 3 versions per flow | Keep all versions from the last 3 months |
| Power BI PBIX | Last 5 versions | Keep all versions from the last 6 months |
| Compliance report emails | 3 years | Saved in compliance mailbox folder |

---

## Section 2: Restoration Procedures

### 2.1 Restore SharePoint List Data from CSV

Use when: individual items were incorrectly deleted or corrupted; full list restoration is needed after accidental deletion.

**Option A — SharePoint Recycle Bin (preferred for recent deletions)**

Deleted SharePoint list items are retained in the Recycle Bin for 93 days:

1. Navigate to the EIT SharePoint site
2. Click the **Recycle Bin** gear icon (Site Settings → Recycle Bin, or click the wastebasket icon in the left nav)
3. Find the deleted items (sort by Deleted Date)
4. Select the items → **Restore**
5. Items are restored to their original location

**Option B — CSV Re-import (for older data or full restore)**

1. Locate the most recent CSV backup
2. Open the CSV in Excel and verify the data
3. In SharePoint: navigate to the list → **Edit in grid view**
4. Paste data row by row (SharePoint doesn't support bulk CSV import natively — use the Microsoft Lists import feature or a Power Automate flow)
5. For bulk import via Power Automate: create a temporary flow that reads the CSV from SharePoint document library and creates items in the target list

> **Alternative bulk import:** Use **Microsoft Lists** → **Import from CSV** feature (available in the Microsoft Lists app at `lists.microsoft.com`). This imports up to 15,000 rows from a CSV file directly into a new or existing list.

### 2.2 Restore the Power App

Use when: the published app is broken or needs to be rolled back to a previous version.

**From an exported package:**

1. Navigate to `https://make.powerapps.com`
2. Click **+ New app** → **Import canvas app**
3. Click **Upload** → select the backup `.zip` file
4. Configure each resource:
   - For the app: **Create new** or **Update** existing (choose Update to replace the broken version)
   - For connections: **Select during import** (reconnect to existing SharePoint/Power Automate connections)
5. Click **Import**
6. After import: test the app thoroughly before sharing
7. If the restored app is an earlier version: apply any changes made since that version

> **Version history:** Power Apps does not natively maintain version history beyond the current published and saved states. This is why external backup packages are essential for rollback.

### 2.3 Restore a Power Automate Flow

Use when: a flow was accidentally deleted, or the current version is broken and needs rollback.

**From an exported package:**

1. Navigate to `https://make.powerautomate.com`
2. Click **+ Create** → **Import**
3. Click **Upload** → select the backup `.zip`
4. Configure each resource:
   - For the flow: **Create new** (to restore a deleted flow) or **Update** (to overwrite broken)
   - For connections: **Select during import** → choose existing authenticated connections
5. Click **Import**
6. After import: turn the flow **On** (imported flows default to Off)
7. Test the flow manually before leaving it in scheduled/automated operation

### 2.4 Restore Power BI Report

**From a PBIX backup:**

1. Navigate to Power BI Service workspace
2. Click **+ Upload** → **From this computer** → select the backup `.pbix`
3. If overwriting: Power BI will ask to update the existing report and dataset
4. After upload: reconfigure the scheduled refresh credentials (dataset settings → Data source credentials)
5. Re-verify RLS role assignments (dataset → Security)
6. Run a manual refresh to confirm connectivity

---

## Section 3: Full DR Scenario — Site Deletion or Corruption

### 3.1 Scenario: EIT SharePoint Site Accidentally Deleted

**Detection:** Users report the EIT Power App shows "Connection failed" for all data sources; SharePoint site URL returns 404.

**SharePoint site recovery window:** Deleted sites are retained for 93 days in the SharePoint admin centre Recycle Bin.

**Response:**

1. Navigate to `https://admin.sharepoint.com` → **Active sites**
2. If the site is not listed: go to **Deleted sites**
3. Find `EnterpriseIncidentTracker` → select it → **Restore**
4. Wait for restoration (typically 15–60 minutes)
5. After restoration: verify all 5 lists are intact with their data
6. Run the Power App — confirm data connections re-established automatically
7. Run a manual Power BI refresh

**If site is not in the admin Recycle Bin (>93 days since deletion):**

This is a full restoration scenario:

1. Create a new SharePoint site with the same URL slug (`EnterpriseIncidentTracker`)
2. Create all 5 lists using the JSON schemas in `01-SharePoint/`
3. Import data from the most recent CSV backups (see 2.1 Option B)
4. Re-share the site with the EIT groups
5. Update all Power Automate flows (SharePoint connection uses the site URL — connections should auto-reconnect if the URL is unchanged)
6. Test all flows manually
7. Republish the Power App (open in edit mode → Publish)
8. Reconfigure Power BI scheduled refresh (re-authenticate credentials)

**Estimated RTO:** 8–24 hours depending on data volume and whether the site URL is preserved.

### 3.2 Scenario: Accidental Bulk Delete of EIT_Incidents

**Detection:** EIT administrator or user notices item count dropped dramatically.

**Immediate response (within 93 days of deletion):**

1. Check SharePoint site Recycle Bin for deleted items
2. If found: Restore all deleted items
3. Verify `DaysSinceUpdate` values are still accurate (run the staleness flow manually after restoration)

**If Recycle Bin is empty (bulk delete > 93 days ago):**

1. Restore from the most recent weekly CSV backup
2. Import items to `EIT_Incidents` using Microsoft Lists import or Power Automate import flow
3. Note: restored items will have new SharePoint integer IDs. The `ItemID` (EIT-NNN) field is preserved from the CSV; however the relationship between `EIT_IncidentHistory.ParentItemID` and `EIT_Incidents.ItemID` is logical (text match) — the relationship is maintained through the ItemID text field, not the SharePoint integer ID. History entries should still be associated correctly.

---

## Section 4: Annual DR Test

The EIT administrator must conduct a DR test annually to verify that backup and restoration procedures work and that RTOs are achievable.

### 4.1 DR Test Scope

Each annual test should cover at least:

1. **CSV restore test:** Import a backup CSV into a test SharePoint list and verify row count and field accuracy
2. **Flow restore test:** Import a flow backup package into a test environment and verify it configures correctly
3. **App restore test:** Import a Power App backup package and verify it opens without errors
4. **PBIX restore test:** Upload a PBIX backup to a test Power BI workspace and verify it connects to a test data source

### 4.2 DR Test Procedure

**Preparation (1 day before):**
- Notify the M365 platform team of the planned test
- Create a test SharePoint site (`EnterpriseIncidentTrackerDR`)
- Locate the most recent backup files for all components

**Test execution (estimated 3 hours):**

1. Restore `EIT_Incidents` CSV to the test site — verify row count matches source
2. Restore `EIT_IncidentHistory` CSV to the test site — verify history entries reference correct ItemIDs
3. Import the `EIT Intake Promotion` flow package → reconfigure to point to test site → test manually
4. Import the Power App package → reconnect to test site data sources → verify Dashboard screen loads with test data
5. Upload PBIX to test Power BI workspace → connect to test SharePoint → verify data loads
6. Record RTO: time from test start to fully functional test environment

**Post-test:**
- Document actual RTO vs target RTO
- Note any gaps or failures in the restoration procedures
- Update this guide with any procedure improvements identified
- Delete the test site and test flow

### 4.3 DR Test Sign-Off Record

```
EIT ANNUAL DISASTER RECOVERY TEST — SIGN-OFF

Test date: [DATE]
Tester: [NAME, TITLE]
Environment: [Test site URL]

Component                 | Restored? | Time    | Issues
--------------------------|-----------|---------|------------------
EIT_Incidents (CSV)       |           |         |
EIT_IncidentHistory (CSV) |           |         |
EIT Intake Promotion flow |           |         |
Power App                 |           |         |
Power BI PBIX             |           |         |

Overall RTO achieved: [HH:MM]
Target RTO: 8 hours

Passed / Failed: _____________

Actions required:
  - [List any procedure gaps or improvements needed]

Sign-off: _____________________________ Date: __________
```

---

## Section 5: Microsoft Platform Resilience

The EIT relies on Microsoft 365 platform availability. EIT-specific DR procedures cannot address Microsoft service outages — these are governed by Microsoft's own resilience architecture.

### 5.1 Microsoft Service Health Monitoring

Monitor Microsoft 365 service health for any degradation affecting EIT:

- **Service health dashboard:** `https://admin.microsoft.com` → Service health
- **Services to monitor:** SharePoint Online, Power Apps, Power Automate, Power BI Service
- **Subscribe to email alerts:** Service health → Preferences → Email notifications

### 5.2 Microsoft SLAs

| Service | Microsoft SLA (uptime) |
|---------|----------------------|
| SharePoint Online | 99.9% |
| Power Apps | 99.9% |
| Power Automate | 99.9% |
| Power BI Service | 99.9% |

All services include geo-redundancy and automated failover within Microsoft's data centres. No additional EIT-specific geo-redundancy configuration is required.

### 5.3 Support Escalation

For Microsoft service outages affecting EIT:

1. Log a Microsoft 365 Admin support ticket: `https://admin.microsoft.com` → Support → New service request
2. Reference the specific service (e.g., SharePoint Online) and impact (EIT system unavailable)
3. Activate manual fallback while awaiting resolution (see `EIT-Phase3-Operations-Runbook.md` Section 4)
