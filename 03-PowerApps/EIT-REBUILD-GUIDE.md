# Enterprise Incident Tracker — Power Apps Canvas App Rebuild Guide

> **Prerequisites before starting this step:**
> 1. SharePoint site and all 4 lists created (see `01-SharePoint/EIT-CREATE-SITE.md`)
> 2. All 3 Power Automate flows created and tested (see `02-PowerAutomate/`)
> 3. Sample data loaded (optional but recommended for testing)

---

## Overview

The Enterprise Incident Tracker Power App is a Canvas App with **7 screens**:

| Screen | Purpose |
|--------|---------|
| **DashboardScreen** | Enterprise KPI overview, Intake Queue gallery, Submit New Incident overlay, navigation to all screens |
| **TrackerScreen** | Domain-filtered, searchable incident table with Severity/Status badges and quick-update side panel |
| **IssueDetailScreen** | Full incident view with P&C warning banner, update history timeline, and add-update form (triggers escalation flow) |
| **DomainOverviewScreen** | Active incident counts per domain in a 1×5 card row |
| **KanbanScreen** | Visual board with 5 vertical swim-lanes by status (New, Active, Escalated, Monitoring, Contained) |
| **ExecutiveReportScreen** | Read-only leadership summary — incident counts and escalation status by domain and severity |

> **Changes from OFR:** 6 screens → 7 screens. GroupAllocationScreen renamed/redesigned as DomainOverviewScreen (5 cards, not 10). ExecutiveReportScreen is net-new. Submit form is an overlay on DashboardScreen (not a standalone screen). Kanban adds Contained swim-lane.

---

## Step 1: Create a New Canvas App

1. Navigate to `https://make.powerapps.com`
2. Ensure you're in the correct environment (check top-right environment picker)
3. Click **+ Create** → **Blank app** → **Blank canvas app**
4. Name: `Enterprise Incident Tracker`
5. Format: **Tablet** (landscape)
6. Click **Create**

---

## Step 2: Add Data Sources

1. In the Power Apps Studio left panel, click the **Data** icon (cylinder)
2. Click **+ Add data**
3. Search for **SharePoint**
4. Select your SharePoint connection (or create one with your admin account)
5. Enter the site URL: `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker`
6. Select all 4 lists:
   - `EIT_Incidents`
   - `EIT_IncidentHistory`
   - `EIT_IntakeQueue`
   - `EIT_DomainContacts`
7. Click **Connect**

---

## Step 3: Add the Power Automate Flows

1. Click the **Action** menu (top toolbar)
2. Select **Power Automate**
3. Click **Add flow**
4. Add `EIT Intake Promotion` — available in formulas as `EITIntakePromotion.Run()`
5. Click **Add flow** again
6. Add `EIT Domain Escalation Notifier` — available in formulas as `EITDomainEscalationNotifier.Run()`

---

## Step 4: Set App-Level Variables (App.OnStart)

These variables initialize on app launch and are available across all screens.

Click the **App** object in the tree → set **OnStart**:

```
Set(
    varUserHasPACAccess,
    Office365Groups.IsMemberOfGroup("[EIT-PAC-GROUP-ID]", User().Email).value
);
Set(varCurrentUser, User().FullName);
Set(varCurrentUserEmail, User().Email)
```

> **Replace `[EIT-PAC-GROUP-ID]`** with the Entra group object ID for your P&C-authorized access group. This variable gates visibility of P&C incidents in galleries. See `04-Documentation/EIT-Access-Control-Guide.md` for the full P&C access model.

---

## Step 5: Build the Screens

Follow the detailed construction guide in **`EIT-PowerApps-Completion-Guide.md`** (included in this folder). It provides:

- Screen-by-screen, control-by-control build instructions
- Exact formulas for every property (OnSelect, Items, Text, Fill, Visible, etc.)
- Coordinates (X, Y, Width, Height) for every control
- Color values matching the Appkit4 design system
- Navigation wiring between screens
- EIT-specific formulas for Domain filtering, Severity badges, and P&C warning banners

### Build Order

1. **DashboardScreen** — 5 KPI cards, Intake Gallery with promote/dismiss, Submit New Incident overlay, 4 navigation buttons
2. **TrackerScreen** — 7 Domain filter chips, search, incident table gallery with Domain/Severity columns, result count
3. **IssueDetailScreen** — Incident header card with P&C banner, update history gallery, add-update form with escalation notifier trigger
4. **DomainOverviewScreen** — 5 domain cards in a single row showing active incident counts
5. **KanbanScreen** — 5 vertical galleries (New, Active, Escalated, Monitoring, Contained) with incident cards
6. **ExecutiveReportScreen** — Read-only summary gallery built with AddColumns formula, no edit controls

---

## Step 6: Intake Review Panel (DashboardScreen)

The Intake Review side panel opens when a user clicks a pending intake item. It includes:

| Control | Type | Key Property |
|---------|------|-------------|
| `rectIntakePanel` | Rectangle (panel background) | `Visible: showIntakePanel` |
| `lblIntakePanelTitle` | Label ("Intake Review" heading) | `Visible: showIntakePanel` |
| `btnIntakePanelClose` | Button ("X" close) | `OnSelect: UpdateContext({showIntakePanel: false})` |
| `lblPanelTitle` | Label (incident title) | `Text: selectedIntake.Title` |
| `lblPanelDesc` | Label (description) | `Text: selectedIntake.Description` |
| `lblPanelDomain` | Label (domain) | `Text: "Domain: " & selectedIntake.Domain.Value` |
| `lblPanelType` | Label (incident type) | `Text: "Type: " & selectedIntake.IncidentType.Value` |
| `lblPanelPriority` | Label (priority) | `Text: "Priority: " & selectedIntake.Priority.Value` |
| `lblPanelDate` | Label (date submitted) | `Text: "Submitted: " & Text(selectedIntake.DateSubmitted, "mm/dd/yyyy")` |
| `lblPanelPAC` | Label (P&C warning) | `Text: "⚠ Privileged & Confidential"`, `Visible: selectedIntake.PrivilegedAndConfidential` |
| `lblPanelOwnerPrompt` | Label ("Assign Owner") | `Visible: showIntakePanel` |
| `txtPanelOwner` | Text input (owner entry) | `Default: User().FullName` |
| `btnPanelAccept` | Button ("Accept into Tracker") | See formula below |
| `btnPanelReject` | Button ("Reject") | See formula below |
| `btnPanelPromote` | Button ("Promote via Flow") | See formula below |

**galIntakeQueue.OnSelect** (Intake Queue gallery):
```
UpdateContext({showIntakePanel: true, selectedIntake: ThisItem})
```

**btnPanelPromote.OnSelect** (Promote via Flow — preferred path):
```
Set(
    varPromoteResult,
    EITIntakePromotion.Run(Text(selectedIntake.ID))
);
Notify(
    "Incident promoted as " & varPromoteResult.newitemid,
    NotificationType.Success
);
Refresh(EIT_IntakeQueue);
Refresh(EIT_Incidents);
UpdateContext({showIntakePanel: false})
```

**btnPanelAccept.OnSelect** (Direct accept — fallback if flow is unavailable):
```
Patch(
    EIT_IntakeQueue,
    selectedIntake,
    {TriageStatus: {Value: "Accepted"}}
);
Reset(txtPanelOwner);
UpdateContext({showIntakePanel: false});
Notify("Item accepted", NotificationType.Success)
```

**btnPanelReject.OnSelect**:
```
Patch(
    EIT_IntakeQueue,
    selectedIntake,
    {TriageStatus: {Value: "Rejected"}}
);
UpdateContext({showIntakePanel: false});
Notify("Item rejected", NotificationType.Warning)
```

---

## Step 7: Design System Reference

| Element | Value |
|---------|-------|
| Screen background | `RGBA(245,247,250,1)` |
| Panel/card background | `RGBA(255,255,255,1)` |
| Header/nav bar | `RGBA(65,83,133,1)` (Appkit4 Blue) |
| Primary accent / buttons | `RGBA(208,74,2,1)` (Appkit4 Orange) |
| Critical / Stale / High | `RGBA(224,48,30,1)` (Appkit4 Red) |
| Heading text | `RGBA(55,71,79,1)` (dark charcoal) |
| Body text | `RGBA(45,45,45,1)` |
| Muted / metadata text | `RGBA(100,100,100,1)` |
| Font | Segoe UI |
| Canvas size | 1366 × 768 (Tablet landscape) |

### Severity Badge Colors

| Severity | Fill |
|----------|------|
| Critical | `RGBA(119,40,32,1)` (dark red) |
| High | `RGBA(224,48,30,1)` |
| Moderate | `RGBA(228,92,43,1)` |
| Low | `RGBA(65,83,133,1)` |
| Informational | `RGBA(150,150,150,1)` |

**Severity Switch formula (reusable):**
```
Switch(
    [SeverityField].Value,
    "Critical", RGBA(119,40,32,1),
    "High", RGBA(224,48,30,1),
    "Moderate", RGBA(228,92,43,1),
    "Low", RGBA(65,83,133,1),
    RGBA(150,150,150,1)
)
```

### Status Badge Colors (EIT — includes Contained)

```
Switch(
    [StatusField].Value,
    "New", RGBA(65,83,133,1),
    "Active", RGBA(208,74,2,1),
    "Monitoring", RGBA(228,92,43,1),
    "Escalated", RGBA(224,48,30,1),
    "Contained", RGBA(50,140,80,1),
    RGBA(140,140,140,1)
)
```

---

## Step 8: Save, Test, and Publish

1. **Save:** File → Save (or Ctrl+S)
2. **Preview:** Click the Play button (top-right) to test
3. Test checklist:
   - Dashboard KPIs show correct counts for each metric
   - Intake queue shows pending items; clicking opens review panel
   - Promote button runs the flow and creates an EIT-NNN incident
   - Tracker screen: domain filter chips filter correctly
   - Tracker screen: search filters by Title, Owner, ItemID, Domain, IncidentType
   - Issue detail shows all fields including Severity, EscalationThreshold, P&C banner
   - Adding an update saves history entry and refreshes incident header
   - When Escalation update type is selected and saved, escalation notifier flow fires
   - Domain Overview shows correct active counts per domain
   - Kanban shows issues in correct swim-lanes; Contained lane populated
   - Executive Report shows read-only summary table by domain
4. **Publish:** Click the Publish icon → Publish this version

---

## Step 9: Share the App

1. Go to `https://make.powerapps.com` → Apps
2. Find `Enterprise Incident Tracker` → click the three dots (...) → **Share**
3. Add the M365 group `EIT Members` for Contribute-level app users
4. Optionally add `EIT Viewers` with **User** permission (read-only in app)
5. Click **Share**

> **Note:** SharePoint item-level permissions for P&C incidents are enforced at the SharePoint layer, not in Power Apps. The `varUserHasPACAccess` variable gates the P&C gallery display in the app as a UX control only. See `04-Documentation/EIT-Access-Control-Guide.md`.

---

## Step 10: Optional — Pin to Microsoft Teams

1. Open Microsoft Teams
2. Navigate to the target channel
3. Click **+** (Add a tab)
4. Search for **Power Apps**
5. Select `Enterprise Incident Tracker`
6. Click **Save**

---

## Notes

- The full construction guide (`EIT-PowerApps-Completion-Guide.md`) contains exact coordinates, every control property, and detailed formulas for all 7 screens. Use it as the primary reference.
- This EIT-REBUILD-GUIDE.md provides the high-level sequence and key panel formulas.
- All SharePoint Choice columns require the `{Value: "text"}` syntax in Power Apps Patch formulas.
- `EIT_IncidentHistory` is append-only — the app never Patches existing history rows, only creates new ones.
- The `varUserHasPACAccess` variable should be checked wherever P&C incidents might be displayed. Use `If(varUserHasPACAccess, ..., ...)` in gallery Items formulas to hide restricted records.
