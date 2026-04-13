# OFR Intake Promotion — Flow Rebuild Guide

> **Prerequisite:** The SharePoint site and all 3 lists must exist before creating this flow. See `01-SharePoint/CREATE-SITE.md`.

---

## Flow Overview

| Property | Value |
|----------|-------|
| Name | `OFR Intake Promotion` |
| Type | Instant cloud flow |
| Trigger | Power Apps (V2) |
| Input | `IntakeItemID` (Number) — SharePoint ID of the intake queue item |
| Output | `NewItemID` (Text) — Generated OFR-NNN identifier |
| Purpose | Promotes a pending intake item into the active issue tracker |
| Connection | SharePoint (your admin account) |

### What This Flow Does

1. Reads the intake queue item by ID
2. Creates a new issue in OFR_Issues with data from the intake item
3. Creates an audit trail entry in OFR_UpdateHistory
4. Marks the intake item as "Promoted"
5. Returns the new ItemID to Power Apps

---

## Step-by-Step Build Instructions

### 1. Create the Flow

1. Navigate to `https://make.powerautomate.com`
2. Click **+ Create** → **Instant cloud flow**
3. Flow name: `OFR Intake Promotion`
4. Select trigger: **PowerApps (V2)**
5. Click **Create**

### 2. Configure the Trigger — Power Apps (V2)

1. Click the trigger to expand it
2. Click **+ Add an input**
3. Select **Number**
4. Set the input name to: `IntakeItemID`
5. Description (optional): `SharePoint ID of the intake queue item to promote`

### 3. Add Action — Get Item (Read Intake Queue)

1. Click **+ New step**
2. Search for **SharePoint** → select **Get item** (singular, not "Get items")
3. Configure:
   - **Site Address:** `https://[TENANT].sharepoint.com/sites/OFRIssueTracker`
   - **List Name:** `OFR_IntakeQueue`
   - **Id:** Select the `IntakeItemID` dynamic content from the trigger

### 4. Add Action — Create Item (New Issue in OFR_Issues)

1. Click **+ New step**
2. Search for **SharePoint** → select **Create item**
3. Configure:
   - **Site Address:** `https://[TENANT].sharepoint.com/sites/OFRIssueTracker`
   - **List Name:** `OFR_Issues`

Now set each field:

| Field | Value Type | Value |
|-------|-----------|-------|
| **Title** | Dynamic content | `Title` from Get item |
| **ItemID** | Expression | See below |
| **Owner** | Dynamic content | `Owner` from Get item |
| **Priority Value** | Dynamic content | `Priority Value` from Get item |
| **Status Value** | Static text | `New` |
| **DateRaised** | Expression | `utcNow()` |
| **LastUpdated** | Expression | `utcNow()` |
| **NextAction** | Dynamic content | `Description` from Get item |
| **DaysSinceUpdate** | Static number | `0` |
| **StalenessFlag Value** | Static text | `Current` |

**ItemID Expression** (also in `flow-expressions/promotion-ItemID.txt`):

Click the **ItemID** field → switch to **Expression** tab → paste:

```
concat('OFR-', string(outputs('Get_item')?['body/ID']))
```

Click **OK**.

**How it works:** Uses the SharePoint auto-increment ID from the OFR_IntakeQueue item to create a unique `OFR-NNN` identifier.

**DateRaised and LastUpdated:** Click each field → Expression tab → type `utcNow()` → OK.

### 5. Add Action — Create Item (Audit Trail in OFR_UpdateHistory)

1. Click **+ New step**
2. Search for **SharePoint** → select **Create item**
3. **Rename this action** to `Create item 1` (Power Automate may auto-name it)
4. Configure:
   - **Site Address:** `https://[TENANT].sharepoint.com/sites/OFRIssueTracker`
   - **List Name:** `OFR_UpdateHistory`

| Field | Value Type | Value |
|-------|-----------|-------|
| **Title** | Dynamic content | `ItemID` from Create item (Step 4) |
| **ParentItemID** | Dynamic content | `ItemID` from Create item (Step 4) |
| **UpdateDate** | Expression | `utcNow()` |
| **StatusAtUpdate Value** | Static text | `New` |
| **Notes** | Static text | `Promoted from intake queue` |
| **UpdatedBy** | Dynamic content | `Owner` from Get item (Step 3) |

### 6. Add Action — Update Item (Mark Intake as Promoted)

1. Click **+ New step**
2. Search for **SharePoint** → select **Update item**
3. Configure:
   - **Site Address:** `https://[TENANT].sharepoint.com/sites/OFRIssueTracker`
   - **List Name:** `OFR_IntakeQueue`
   - **Id:** Select `IntakeItemID` from the trigger
   - **Title:** Dynamic content → `Title` from Get item (required by SharePoint update)
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
   - Click **Test** → **Manually** → **Test**
   - Enter a valid IntakeItemID (the SharePoint ID of a pending intake item — check OFR_IntakeQueue list for the ID column)
   - Click **Run flow**
3. Verify:
   - New item appears in OFR_Issues with correct data
   - New audit entry appears in OFR_UpdateHistory
   - Intake item's TriageStatus changed to "Promoted"
   - Flow output shows the new ItemID

### 9. Grant Run-Only Access

After testing, Power Apps users need permission to trigger this flow:

1. Go to the flow details page
2. Click **Edit** on the **Run only users** section
3. Add the M365 group that will use the Power App
4. Under **Connections Used**, select **Use this connection** for the SharePoint connection
5. Click **Save**

---

## Connecting to Power Apps

After the flow is created, you must wire it to the Power Apps Promote button:

1. Open the Power Apps canvas app in edit mode
2. On the DashboardScreen, select the **Promote** button inside the Intake Gallery
3. Set its **OnSelect** to:
   ```
   OFRIntakePromotion.Run(ThisItem.ID);
   Refresh(OFR_IntakeQueue);
   Refresh(OFR_Issues);
   Notify("Issue promoted to tracker", NotificationType.Success)
   ```
4. If Power Apps doesn't recognize the flow name, go to **Action** menu → **Power Automate** → **Add flow** → select `OFR Intake Promotion`

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| "Flow not found" in Power Apps | Re-add the flow via Action → Power Automate → Add flow |
| "Item not found" error | Verify the IntakeItemID matches an existing row in OFR_IntakeQueue |
| Priority not mapping | Ensure OFR_Issues Priority choices exactly match OFR_IntakeQueue choices (High, Medium, Low) |
| "Access denied" for users | Check Run-only permissions and ensure SharePoint connection is shared |

---

## Flow Diagram

```
Power Apps (V2) trigger
  Input: IntakeItemID (Number)
    |
    v
Get item: OFR_IntakeQueue [by IntakeItemID]
    |
    v
Create item: OFR_Issues
  Title = intake.Title
  ItemID = concat('OFR-', string(intake.ID))
  Owner = intake.Owner
  Priority = intake.Priority
  Status = 'New'
  DateRaised = utcNow()
  LastUpdated = utcNow()
  NextAction = intake.Description
  DaysSinceUpdate = 0
  StalenessFlag = 'Current'
    |
    v
Create item: OFR_UpdateHistory
  Title = new.ItemID
  ParentItemID = new.ItemID
  UpdateDate = utcNow()
  StatusAtUpdate = 'New'
  Notes = 'Promoted from intake queue'
  UpdatedBy = intake.Owner
    |
    v
Update item: OFR_IntakeQueue
  Id = IntakeItemID
  TriageStatus = 'Promoted'
    |
    v
Respond to PowerApp
  Output: NewItemID = new.ItemID
```
