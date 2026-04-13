# EIT Intake Promotion — Flow Rebuild Guide

> **Prerequisite:** The SharePoint site and all 4 lists must exist before creating this flow. See `01-SharePoint/EIT-CREATE-SITE.md`.

---

## Flow Overview

| Property | Value |
|----------|-------|
| Name | `EIT Intake Promotion` |
| Type | Instant cloud flow |
| Trigger | Power Apps (V2) |
| Input | `IntakeItemID` (Number) — SharePoint ID of the intake queue item |
| Output | `NewItemID` (Text) — Generated EIT-NNN identifier |
| Purpose | Promotes a pending intake item into the active incident tracker |
| Connection | SharePoint (your admin account) |

### Changes from OFR Intake Promotion

| Change | Detail |
|--------|--------|
| List names | `OFR_IntakeQueue` → `EIT_IntakeQueue`; `OFR_Issues` → `EIT_Incidents`; `OFR_UpdateHistory` → `EIT_IncidentHistory` |
| Site URL | `OFRIssueTracker` → `EnterpriseIncidentTracker` |
| ItemID prefix | `OFR-` → `EIT-` |
| New mapped fields | `Domain`, `IncidentType` (from intake); `Severity`, `EscalationThreshold`, `SourceSystem`, `ConfidentialityLevel` (defaults) |
| History entry | Adds `SeverityAtUpdate` and `UpdateType` fields to match EIT_IncidentHistory schema |

### What This Flow Does

1. Reads the intake queue item by ID
2. Creates a new incident in `EIT_Incidents` with data from the intake item
3. Creates an opening audit entry in `EIT_IncidentHistory`
4. Marks the intake item as "Promoted" in `EIT_IntakeQueue`
5. Returns the new ItemID to Power Apps

---

## Step-by-Step Build Instructions

### 1. Create the Flow

1. Navigate to `https://make.powerautomate.com`
2. Click **+ Create** → **Instant cloud flow**
3. Flow name: `EIT Intake Promotion`
4. Select trigger: **PowerApps (V2)**
5. Click **Create**

### 2. Configure the Trigger — Power Apps (V2)

1. Click the trigger to expand it
2. Click **+ Add an input**
3. Select **Number**
4. Set the input name to: `IntakeItemID`
5. Description (optional): `SharePoint ID of the EIT_IntakeQueue item to promote`

### 3. Add Action — Get Item (Read Intake Queue)

1. Click **+ New step**
2. Search for **SharePoint** → select **Get item** (singular, not "Get items")
3. Configure:
   - **Site Address:** `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker`
   - **List Name:** `EIT_IntakeQueue`
   - **Id:** Select the `IntakeItemID` dynamic content from the trigger

### 4. Add Action — Create Item (New Incident in EIT_Incidents)

1. Click **+ New step**
2. Search for **SharePoint** → select **Create item**
3. Configure:
   - **Site Address:** `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker`
   - **List Name:** `EIT_Incidents`

Set each field as follows:

| Field | Value Type | Value |
|-------|-----------|-------|
| **Title** | Dynamic content | `Title` from Get item |
| **ItemID** | Expression | See below |
| **Owner** | Dynamic content | `Owner` from Get item |
| **Domain Value** | Dynamic content | `Domain Value` from Get item |
| **Priority Value** | Dynamic content | `Priority Value` from Get item |
| **IncidentType Value** | Dynamic content | `IncidentType Value` from Get item |
| **Severity Value** | Static text | `Moderate` |
| **Status Value** | Static text | `New` |
| **SourceSystem Value** | Static text | `Power Apps` |
| **EscalationThreshold Value** | Static text | `Not Escalated` |
| **ConfidentialityLevel Value** | Static text | `Internal` |
| **DateRaised** | Expression | `utcNow()` |
| **LastUpdated** | Expression | `utcNow()` |
| **NextAction** | Dynamic content | `Description` from Get item |
| **DaysSinceUpdate** | Static number | `0` |
| **StalenessFlag Value** | Static text | `Current` |
| **PrivilegedAndConfidential** | Dynamic content | `PrivilegedAndConfidential` from Get item |

**ItemID Expression** (also in `flow-expressions/EIT-promotion-ItemID.txt`):

Click the **ItemID** field → switch to **Expression** tab → paste:

```
concat('EIT-', string(outputs('Get_item')?['body/ID']))
```

Click **OK**.

> **How it works:** Uses the SharePoint auto-increment ID from the `EIT_IntakeQueue` item to create a unique `EIT-NNN` identifier.

**For `DateRaised` and `LastUpdated`:** Click each field → Expression tab → type `utcNow()` → OK.

> **Severity default:** New incidents are promoted with Severity = `Moderate` as a safe default. The triage reviewer must update this in the Issue Detail screen after reviewing the full incident context. This is by design — severity assignment requires human judgement and should not be inherited blindly from the submitter's Priority field.

> **Domain mapping note:** `Domain Value` in the Create item field maps to `Domain Value` from the intake Get item step. Both lists use identical Domain choice values — the mapping is direct. If you cannot find `Domain Value` in the dynamic content picker, expand the Get item step in the panel or type "Domain" to search.

### 5. Add Action — Create Item (Audit Entry in EIT_IncidentHistory)

1. Click **+ New step**
2. Search for **SharePoint** → select **Create item**
3. Power Automate will name this `Create item 2` — rename it `Create history entry` for clarity (click the action title)
4. Configure:
   - **Site Address:** `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker`
   - **List Name:** `EIT_IncidentHistory`

| Field | Value Type | Value |
|-------|-----------|-------|
| **Title** | Dynamic content | `ItemID` from Create item (Step 4) |
| **ParentItemID** | Dynamic content | `ItemID` from Create item (Step 4) |
| **UpdateDate** | Expression | `utcNow()` |
| **StatusAtUpdate Value** | Static text | `New` |
| **Notes** | Static text | `Promoted from intake queue` |
| **UpdatedBy** | Dynamic content | `Owner` from Get item (Step 3) |
| **SeverityAtUpdate Value** | Static text | `Moderate` |
| **UpdateType Value** | Static text | `Status Change` |

> **Note:** `SeverityAtUpdate` is set to `Moderate` here to match the default Severity set on the new incident in Step 4. Both values will be updated by users as the incident progresses.

### 6. Add Action — Update Item (Mark Intake as Promoted)

1. Click **+ New step**
2. Search for **SharePoint** → select **Update item**
3. Configure:
   - **Site Address:** `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker`
   - **List Name:** `EIT_IntakeQueue`
   - **Id:** Select `IntakeItemID` from the trigger
   - **Title:** Dynamic content → `Title` from Get item (required by SharePoint Update item)
   - **TriageStatus Value:** Static text → `Promoted`

### 7. Add Action — Respond to a PowerApp or Flow

1. Click **+ New step**
2. Search for **Respond** → select **Respond to a PowerApp or flow**
3. Click **+ Add an output**
4. Select **Text**
5. Set:
   - **Title:** `NewItemID`
   - **Value:** Dynamic content → `ItemID` from Create item (Step 4)

### 8. Save and Test

1. Click **Save** (top-right)
2. To test manually:
   - Ensure at least one `Pending` item exists in `EIT_IntakeQueue` (load `sample-data/EIT_IntakeQueue-sample.csv` if needed)
   - Note the SharePoint `ID` (integer) of the item — visible in the list's `ID` column (add it to the view if not shown)
   - Click **Test** → **Manually** → **Test**
   - Enter that integer as `IntakeItemID`
   - Click **Run flow**
3. Verify:
   - New item appears in `EIT_Incidents` with `ItemID` in `EIT-NNN` format
   - `Domain` matches the intake item's Domain
   - `Severity` is `Moderate`, `EscalationThreshold` is `Not Escalated`
   - New audit entry appears in `EIT_IncidentHistory` with `ParentItemID` matching the new ItemID
   - Intake item's `TriageStatus` changed to `Promoted`
   - Flow output shows the new `EIT-NNN` identifier

### 9. Grant Run-Only Access

1. Go to the flow details page
2. Click **Edit** on the **Run only users** section
3. Add the M365 group that will use the Power App (e.g., `Enterprise Incident Tracker Members`)
4. Under **Connections Used**, select **Use this connection** for the SharePoint connection
5. Click **Save**

---

## Connecting to Power Apps

After creating the flow, wire it to the Promote button in the Power Apps canvas app:

1. Open the EIT canvas app in edit mode
2. On the DashboardScreen, select the **Promote** button inside the Intake Gallery
3. Set its **OnSelect** to:
   ```
   EITIntakePromotion.Run(ThisItem.ID);
   Refresh(EIT_IntakeQueue);
   Refresh(EIT_Incidents);
   Notify("Incident promoted to tracker", NotificationType.Success)
   ```
4. If Power Apps doesn't recognise the flow name, go to **Action** menu → **Power Automate** → **Add flow** → select `EIT Intake Promotion`

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| "Flow not found" in Power Apps | Re-add the flow via Action → Power Automate → Add flow |
| "Item not found" error | Verify the `IntakeItemID` matches an existing row in `EIT_IntakeQueue` |
| `Domain` not mapping | Check that `Domain Value` appears in the dynamic content panel from the Get item step. If not, use an expression: `outputs('Get_item')?['body/Domain/Value']` |
| `Priority` not mapping | Ensure `EIT_Incidents.Priority` choices exactly match `EIT_IntakeQueue.Priority` choices (High, Medium, Low) |
| "Access denied" for app users | Check Run-only permissions and ensure the SharePoint connection is configured as "Use this connection" |
| ItemID shows wrong format | Confirm the expression uses `outputs('Get_item')?['body/ID']` — the `Get_item` action name must match exactly. If you renamed the action, update the expression accordingly. |

---

## Flow Diagram

```
Power Apps (V2) trigger
  Input: IntakeItemID (Number)
    |
    v
Get item: EIT_IntakeQueue [by IntakeItemID]
    |
    v
Create item: EIT_Incidents
  Title               = intake.Title
  ItemID              = concat('EIT-', string(intake.ID))
  Owner               = intake.Owner
  Domain              = intake.Domain
  Priority            = intake.Priority
  IncidentType        = intake.IncidentType
  Severity            = 'Moderate'            ← default, reviewer updates
  Status              = 'New'
  SourceSystem        = 'Power Apps'
  EscalationThreshold = 'Not Escalated'
  ConfidentialityLevel= 'Internal'
  DateRaised          = utcNow()
  LastUpdated         = utcNow()
  NextAction               = intake.Description
  DaysSinceUpdate          = 0
  StalenessFlag            = 'Current'
  PrivilegedAndConfidential= intake.PrivilegedAndConfidential
    |
    v
Create item: EIT_IncidentHistory
  Title            = new.ItemID
  ParentItemID     = new.ItemID
  UpdateDate       = utcNow()
  StatusAtUpdate   = 'New'
  SeverityAtUpdate = 'Moderate'
  UpdateType       = 'Status Change'
  Notes            = 'Promoted from intake queue'
  UpdatedBy        = intake.Owner
    |
    v
Update item: EIT_IntakeQueue
  Id            = IntakeItemID
  TriageStatus  = 'Promoted'
    |
    v
Respond to PowerApp
  Output: NewItemID = new.ItemID
```
