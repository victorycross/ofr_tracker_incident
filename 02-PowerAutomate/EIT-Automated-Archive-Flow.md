# EIT Automated Archive Flow — Rebuild Guide

> **Phase 2 Integration.** This flow is part of the Phase 2 integration layer. See `04-Documentation/EIT-Phase2-Integration-Roadmap.md`.

> **Prerequisite:** The `EIT_Incidents_Archive` SharePoint list must be created before building this flow. See `01-SharePoint/EIT_Incidents_Archive-schema.json`. Confirm the retention period (default 12 months) with your data governance lead before deploying.

---

## Flow Overview

| Property | Value |
|----------|-------|
| Name | `EIT Monthly Archive` |
| Type | Automated cloud flow (scheduled) |
| Trigger | Recurrence (monthly) |
| Purpose | Moves closed EIT incidents older than 12 months from EIT_Incidents to EIT_Incidents_Archive, then deletes the originals |
| Connections | SharePoint (admin account) |
| Retention period | 12 months from `LastUpdated` date (configurable) |

### What This Flow Does

1. Runs on the 1st of each month at 02:00 UTC
2. Queries `EIT_Incidents` for items where `Status = Closed` AND `LastUpdated` is more than 365 days ago
3. For each qualifying incident:
   a. Creates a copy in `EIT_Incidents_Archive` with `ArchivedDate = today`
   b. Deletes the original from `EIT_Incidents`
4. Logs a summary to `EIT_IncidentHistory` (or sends an admin email)

> **History entries:** This flow does not archive `EIT_IncidentHistory` entries — they remain in place as a permanent audit trail. If you wish to archive history entries, add a secondary loop after Step 3b to move matching `EIT_IncidentHistory` rows to an `EIT_IncidentHistory_Archive` list.

---

## Step-by-Step Build Instructions

### 1. Create the Flow

1. Navigate to `https://make.powerautomate.com`
2. Click **+ Create** → **Scheduled cloud flow**
3. Flow name: `EIT Monthly Archive`
4. Configure the schedule:
   - **Starting:** First day of next month, 02:00 UTC
   - **Repeat every:** `1 Month`
5. Click **Create**

---

### 2. Add Action — Initialize Variable (Archive Cutoff Date)

1. Click **+ New step**
2. Search for **Variables** → select **Initialize variable**
3. Rename to: `Set archive cutoff date`
4. Configure:
   - **Name:** `varArchiveCutoff`
   - **Type:** String
   - **Value:** Expression (see also `flow-expressions/EIT-archive-datefilter.txt`):
     ```
     addDays(utcNow(), -365)
     ```

This produces an ISO 8601 timestamp 365 days in the past. Items with `LastUpdated` before this date are eligible for archiving.

---

### 3. Add Action — Initialize Variable (Archive Count)

1. Click **+ New step**
2. Search for **Variables** → select **Initialize variable**
3. Configure:
   - **Name:** `varArchiveCount`
   - **Type:** Integer
   - **Value:** `0`

This counter tracks how many items were archived in this run for the summary log.

---

### 4. Add Action — Get Items (Closed Incidents Older Than 12 Months)

1. Click **+ New step**
2. Search for **SharePoint** → select **Get items**
3. Rename to: `Get archiveable incidents`
4. Configure:
   - **Site Address:** `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker`
   - **List Name:** `EIT_Incidents`
   - **Filter Query:** Click **Show advanced options** and enter:
     ```
     Status eq 'Closed' and LastUpdated lt '[varArchiveCutoff]'
     ```
     Build using Expression tab to substitute the variable:
     ```
     concat('Status eq ''Closed'' and LastUpdated lt ''', variables('varArchiveCutoff'), '''')
     ```
   - **Top Count:** `500` *(process up to 500 items per run — adjust if archive volume is higher)*
   - **Order By:** `LastUpdated` ascending *(archive oldest first)*

> **OData filter note:** The `lt` (less than) operator on DateTime columns compares ISO 8601 strings. The `addDays(utcNow(), -365)` expression returns a full UTC timestamp; SharePoint OData accepts this format. If you get a filter error, try formatting the date as `yyyy-MM-ddT00:00:00Z` using the `formatDateTime` expression: `formatDateTime(addDays(utcNow(), -365), 'yyyy-MM-ddT00:00:00Z')`.

> **Why 500:** A monthly run on a mature system should not exceed 500 qualifying items. If it does, the flow will process 500 items per run and catch the remainder on the next monthly run. For high-volume scenarios, add a **Do Until** loop that continues until no qualifying items remain.

---

### 5. Add Action — Apply to Each (Process Each Archive Candidate)

1. Click **+ New step**
2. Search for **Control** → select **Apply to each**
3. Rename to: `For each archiveable incident`
4. **Select an output from previous steps:** Dynamic content → `value` from `Get archiveable incidents`

All steps 6–8 are placed inside this Apply to each loop.

---

### 6. Inside Loop — Create Item (Copy to Archive)

1. Click **Add an action** (inside the loop)
2. Search for **SharePoint** → select **Create item**
3. Rename to: `Copy incident to archive`
4. Configure:
   - **Site Address:** `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker`
   - **List Name:** `EIT_Incidents_Archive`

Map all columns from the current incident item to the archive list. The archive list mirrors EIT_Incidents with one additional column (`ArchivedDate`):

| Archive Field | Value Type | Value |
|--------------|-----------|-------|
| **Title** | Expression | `items('For_each_archiveable_incident')?['Title']` |
| **ItemID** | Expression | `items('For_each_archiveable_incident')?['ItemID']` |
| **Owner** | Expression | `items('For_each_archiveable_incident')?['Owner']` |
| **Domain Value** | Expression | `items('For_each_archiveable_incident')?['Domain']?['Value']` |
| **Priority Value** | Expression | `items('For_each_archiveable_incident')?['Priority']?['Value']` |
| **IncidentType Value** | Expression | `items('For_each_archiveable_incident')?['IncidentType']?['Value']` |
| **Severity Value** | Expression | `items('For_each_archiveable_incident')?['Severity']?['Value']` |
| **Status Value** | Static text | `Closed` |
| **EscalationThreshold Value** | Expression | `items('For_each_archiveable_incident')?['EscalationThreshold']?['Value']` |
| **SourceSystem Value** | Expression | `items('For_each_archiveable_incident')?['SourceSystem']?['Value']` |
| **ConfidentialityLevel Value** | Expression | `items('For_each_archiveable_incident')?['ConfidentialityLevel']?['Value']` |
| **RelatedItemIDs** | Expression | `items('For_each_archiveable_incident')?['RelatedItemIDs']` |
| **DateRaised** | Expression | `items('For_each_archiveable_incident')?['DateRaised']` |
| **LastUpdated** | Expression | `items('For_each_archiveable_incident')?['LastUpdated']` |
| **DateContained** | Expression | `items('For_each_archiveable_incident')?['DateContained']` |
| **NextAction** | Expression | `items('For_each_archiveable_incident')?['NextAction']` |
| **DaysSinceUpdate** | Expression | `items('For_each_archiveable_incident')?['DaysSinceUpdate']` |
| **StalenessFlag Value** | Expression | `items('For_each_archiveable_incident')?['StalenessFlag']?['Value']` |
| **PrivilegedAndConfidential** | Expression | `items('For_each_archiveable_incident')?['PrivilegedAndConfidential']` |
| **ArchivedDate** | Expression | `utcNow()` |

> **Choice column mapping note:** Choice columns (Domain, Priority, IncidentType, etc.) are accessed with `?['FieldName']?['Value']` to extract the string value. When setting them in the archive Create item, map to the `FieldName Value` field in the form (the SharePoint Create item action exposes choice columns as `FieldName Value`).

---

### 7. Inside Loop — Increment Archive Count

1. Click **Add an action** (inside the loop, after the Create item)
2. Search for **Variables** → select **Increment variable**
3. Configure:
   - **Name:** `varArchiveCount`
   - **Value:** `1`

---

### 8. Inside Loop — Delete Original Item

After confirming the archive copy was created successfully, delete the original.

1. Click **Add an action** (inside the loop)
2. Search for **SharePoint** → select **Delete item**
3. Rename to: `Delete original incident`
4. Configure:
   - **Site Address:** `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker`
   - **List Name:** `EIT_Incidents`
   - **Id:** Expression → `items('For_each_archiveable_incident')?['ID']`

> **Safety note:** The Delete item action is irreversible. Verify the Create item in Step 6 completed successfully before deleting. To add a safety guard: place the Delete item inside a Condition that checks whether the archive Create item returned an ID (i.e., was successful):
> - Condition: `outputs('Copy_incident_to_archive')?['body/ID']` **is not equal to** *(empty)*
> - If yes: Delete the original
> - If no: Send an error notification (add a Send email action)

---

### 9. Add Action — Send Summary Notification (Post-Loop)

After the Apply to each loop, send an admin notification summarising the archive run.

1. Click **+ New step** (at main flow level, outside the loop)
2. Search for **Outlook** → select **Send an email (V2)**
3. Rename to: `Send archive summary`
4. Configure:
   - **To:** `david@brightpathtechnology.io`
   - **Subject:** Expression →
     ```
     concat('EIT Monthly Archive Complete — ', string(variables('varArchiveCount')), ' incidents archived')
     ```
   - **Body:**
     ```
     EIT Monthly Archive Run — [run date]

     Archive run completed successfully.

     Items archived: [varArchiveCount]
     Archive cutoff date: [varArchiveCutoff] (incidents closed before this date)
     Archive destination: EIT_Incidents_Archive

     History entries (EIT_IncidentHistory) were NOT archived — they remain
     in the live list as a permanent audit trail.

     If the archive count is unexpectedly high or low, verify the EIT_Incidents
     list contents and the flow run history in Power Automate.
     ```

> **Tip:** Use dynamic content tokens for `varArchiveCount` and `varArchiveCutoff` in the email body.

---

### 10. Save and Test

1. Click **Save**
2. **Before running in production:** Test against a copy of the SharePoint site or use sample test items created with a `LastUpdated` date more than 365 days ago (set manually via SharePoint list edit or a test flow)
3. To manually trigger a test run: click **Test** → **Manually** → **Run flow**
4. Verify:
   - Archived items appear in `EIT_Incidents_Archive` with all fields populated and `ArchivedDate = today`
   - Archived items are deleted from `EIT_Incidents`
   - Summary email received by admin account
   - `EIT_IncidentHistory` entries for archived incidents remain in place
5. Run again — verify already-archived items are not re-archived (they no longer exist in `EIT_Incidents`)

> **Test data creation:** To create test items with old LastUpdated dates, use a Power Automate flow or SharePoint list edit to create an item, then manually set `LastUpdated` to a date more than a year ago (SharePoint allows direct date editing in list view).

---

## Configuration Options

| Setting | Default | Where to Change |
|---------|---------|----------------|
| Retention period | 365 days | Step 2: Change `-365` in `addDays(utcNow(), -365)` |
| Items per run | 500 | Step 4: `Top Count` value |
| Archive destination | `EIT_Incidents_Archive` | Step 6: `List Name` |
| Run schedule | 1st of month, 02:00 UTC | Trigger: Recurrence settings |
| Admin notification email | `david@brightpathtechnology.io` | Step 9: `To` field |

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| OData filter error on date | Format the date explicitly: `formatDateTime(addDays(utcNow(),-365),'yyyy-MM-ddT00:00:00Z')` |
| Choice column not mapping in archive | Use `?['FieldName']?['Value']` to extract the string, then map to `FieldName Value` in the archive Create item |
| Items deleted but archive empty | The Create item failed silently. Add a Condition gate (Step 8 safety note) and check flow run history for the create action. |
| `Apply to each` throttled (429 error) | Power Automate SharePoint connector has a rate limit of ~600 operations per minute. Reduce Top Count or add a **Delay** action between loop iterations. |
| Admin email not sent | Outlook connector authentication may have expired. Re-authenticate the connection. |
| Flow doesn't run on schedule | Check that the flow is turned ON (not in draft/disabled state). |

---

## Flow Diagram

```
Recurrence trigger (1st of month, 02:00 UTC)
    |
    v
Initialize: varArchiveCutoff = addDays(utcNow(), -365)
Initialize: varArchiveCount = 0
    |
    v
Get items: EIT_Incidents
  Filter: Status = 'Closed' AND LastUpdated < varArchiveCutoff
  Top: 500, Order: LastUpdated ASC
    |
    v
Apply to each: incident in results
    |
    +-- Create item: EIT_Incidents_Archive
    |     [all EIT_Incidents fields]
    |     ArchivedDate = utcNow()
    |
    +-- Increment variable: varArchiveCount + 1
    |
    +-- Condition: archive created successfully?
          |
          YES ──► Delete item: EIT_Incidents [original]
          |
          NO  ──► Send error email (optional)
    |
    v (after loop)
Send email: archive summary
  To: david@brightpathtechnology.io
  Subject: 'EIT Monthly Archive Complete — [count] incidents archived'
```
