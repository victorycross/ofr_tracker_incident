# OFR Issue Tracker — Power Apps Canvas App Rebuild Guide

> **Prerequisites before starting this step:**
> 1. SharePoint site and all 3 lists created (see `01-SharePoint/CREATE-SITE.md`)
> 2. Both Power Automate flows created and tested (see `02-PowerAutomate/`)
> 3. Sample data loaded (optional but recommended for testing)

---

## Overview

The OFR Issue Tracker Power App is a Canvas App with **6 screens** and **2 side panels**:

| Screen | Purpose |
|--------|---------|
| **DashboardScreen** | Executive KPI overview, Intake Queue gallery, Intake Review panel, Submit New Issue button, navigation to all screens |
| **TrackerScreen** | Sortable/filterable issue table with FunctionalGroup column and quick-update side panel |
| **IssueDetailScreen** | Full issue view with FunctionalGroup, update history timeline and add-update form |
| **SubmitScreen** | New issue submission form with FunctionalGroup dropdown (writes to OFR_IntakeQueue) |
| **GroupAllocationScreen** | Active issue counts per functional group in 2×5 card grid |
| **KanbanScreen** | Visual board with 4 vertical swim-lanes by status (New, Active, Escalated, Monitoring) |

---

## Step 1: Create a New Canvas App

1. Navigate to `https://make.powerapps.com`
2. Ensure you're in the correct environment (check top-right environment picker)
3. Click **+ Create** → **Blank app** → **Blank canvas app**
4. Name: `OFR Issue Tracker`
5. Format: **Tablet** (landscape)
6. Click **Create**

## Step 2: Add Data Sources

1. In the Power Apps Studio left panel, click the **Data** icon (cylinder)
2. Click **+ Add data**
3. Search for **SharePoint**
4. Select your SharePoint connection (or create one with your admin account)
5. Enter the site URL: `https://[TENANT].sharepoint.com/sites/OFRIssueTracker`
6. Select all 3 lists:
   - `OFR_Issues`
   - `OFR_UpdateHistory`
   - `OFR_IntakeQueue`
7. Click **Connect**

## Step 3: Add the Power Automate Flow

1. Click the **Action** menu (top toolbar)
2. Select **Power Automate**
3. Click **Add flow**
4. Select `OFR Intake Promotion` from the list
5. The flow is now available as `OFRIntakePromotion.Run()` in formulas

## Step 4: Build the Screens

Follow the detailed construction guide in **`OFR-PowerApps-Completion-Guide.md`** (included in this folder). It provides:

- Screen-by-screen, control-by-control build instructions
- Exact formulas for every property (OnSelect, Items, Text, Fill, Visible, etc.)
- Coordinates (X, Y, Width, Height) for every control
- Color values matching the design system
- Navigation wiring between screens

### Build Order

1. **DashboardScreen** — KPI cards, Intake Gallery, navigation buttons (Tracker, Group Allocation, Kanban)
2. **TrackerScreen** — Filter toggles, search, issue gallery with FunctionalGroup column, column headers, quick-update panel
3. **IssueDetailScreen** — Issue header with FunctionalGroup, update history gallery, add-update form
4. **SubmitScreen** — New issue submission form with FunctionalGroup dropdown
5. **GroupAllocationScreen** — 10 group cards in 2×5 grid showing active issue counts per functional group
6. **KanbanScreen** — 4 vertical galleries (New, Active, Escalated, Monitoring) with issue cards

### Intake Review Panel (DashboardScreen)

The Intake Review panel is a side panel on the DashboardScreen that opens when a user clicks a pending intake item. It includes:

| Control | Type | Key Property |
|---------|------|-------------|
| Rectangle2 | Rectangle (panel background) | Visible: `showIntakePanel` |
| Label1 | Label ("Intake Review" heading) | Visible: `showIntakePanel` |
| Button1 | Button ("X" close) | OnSelect: `UpdateContext({showIntakePanel: false})` |
| Label2 | Label (Title display) | Text: `selectedIntake.Title` |
| Label3 | Label (Description display) | Text: `selectedIntake.Description` |
| Label4 | Label (Priority display) | Text: `"Priority: " & selectedIntake.Priority.Value` |
| Label5 | Label (Date display) | Text: `"Submitted: " & Text(selectedIntake.DateSubmitted, "mm/dd/yyyy")` |
| Label22 | Label ("Assign Owner") | Visible: `showIntakePanel` |
| TextInput6 | Text input (owner entry) | Visible: `showIntakePanel` |
| Button8 | Button ("Accept into Tracker") | See formula below |
| Button12 | Button ("Reject") | See formula below |

**Gallery1.OnSelect** (Intake Queue gallery):
```
UpdateContext({showIntakePanel: true, selectedIntake: ThisItem})
```

**Button8 OnSelect** (Accept into Tracker):
```
Patch(
    OFR_Issues,
    Defaults(OFR_Issues),
    {
        ItemID: "OFR-" & Text(CountRows(OFR_Issues) + 1, "00"),
        Title: selectedIntake.Title,
        Owner: TextInput6.Text,
        Priority: selectedIntake.Priority,
        Status: {Value: "New"},
        DateRaised: selectedIntake.DateSubmitted,
        LastUpdated: Now(),
        DaysSinceUpdate: 0,
        FunctionalGroup: selectedIntake.FunctionalGroup
    }
);
Patch(
    OFR_IntakeQueue,
    selectedIntake,
    {
        TriageStatus: {Value: "Accepted"}
    }
);
Reset(TextInput6);
UpdateContext({showIntakePanel: false});
Notify("Issue accepted into tracker", NotificationType.Success)
```

**Button12 OnSelect** (Reject):
```
Patch(
    OFR_IntakeQueue,
    selectedIntake,
    {
        TriageStatus: {Value: "Rejected"}
    }
);
UpdateContext({showIntakePanel: false});
Notify("Issue rejected", NotificationType.Warning)
```

## Step 5: Design System Reference

| Element | Value |
|---------|-------|
| Screen background | `RGBA(245, 247, 250, 1)` |
| Panel background | `RGBA(255, 255, 255, 1)` (white) |
| Heading text color | `RGBA(55, 71, 79, 1)` (dark charcoal) |
| Accept button fill | `RGBA(202, 80, 16, 1)` (orange) |
| Accept button text | `RGBA(255, 255, 255, 1)` (white) |
| Reject button fill | `RGBA(180, 180, 180, 1)` (gray) |
| Font | Segoe UI |
| Header/nav bar | `RGBA(13, 31, 60, 1)` (navy) |
| Primary accent | `RGBA(208, 74, 2, 1)` (orange) |

## Step 6: Save, Test, and Publish

1. **Save:** File → Save (or Ctrl+S)
2. **Preview:** Click the Play button (top-right) to test
3. Test checklist:
   - Dashboard KPIs show correct counts
   - Intake queue shows pending items
   - Clicking intake item opens review panel
   - Accept/Reject work correctly
   - Tracker screen shows all issues with correct filtering
   - Issue detail shows update history
   - Adding an update saves correctly
   - Submit screen creates new intake items
4. **Publish:** Click the Publish icon → Publish this version

## Step 7: Share the App

1. Go to `https://make.powerapps.com` → Apps
2. Find `OFR Issue Tracker` → click the three dots (...)  → **Share**
3. Add the M365 group (e.g., `OFR Issue Tracker Members`)
4. Set permission level: **User** (can use but not edit)
5. Click **Share**

## Step 8: Optional — Pin to Microsoft Teams

1. Open Microsoft Teams
2. Navigate to the target channel
3. Click **+** (Add a tab)
4. Search for **Power Apps**
5. Select `OFR Issue Tracker`
6. Click **Save**

---

## Notes

- The full construction guide (`OFR-PowerApps-Completion-Guide.md`) contains exact coordinates, every control property, and detailed formulas. Use it as the primary reference.
- This REBUILD-GUIDE.md provides the high-level sequence and key formulas for the Intake Review panel (which was built after the main completion guide was written).
- All SharePoint Choice columns require the `{Value: "text"}` syntax in Power Apps Patch formulas.
