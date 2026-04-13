# Enterprise Incident Tracker — Power Apps Completion Guide

> **Prerequisites:** SharePoint site + 4 lists created, all 3 Power Automate flows built and tested, sample data loaded. Canvas app created (Tablet landscape, 1366×768). All 4 lists and both flows connected as data sources. See `EIT-REBUILD-GUIDE.md` for setup steps.

---

## Pre-Requisites Checklist

- [ ] Canvas app created: `Enterprise Incident Tracker`, Tablet format (1366×768)
- [ ] Data sources connected: `EIT_Incidents`, `EIT_IncidentHistory`, `EIT_IntakeQueue`, `EIT_DomainContacts`
- [ ] Flows added: `EIT Intake Promotion` (`EITIntakePromotion.Run()`), `EIT Domain Escalation Notifier` (`EITDomainEscalationNotifier.Run()`)
- [ ] Seven screens created: `DashboardScreen`, `TrackerScreen`, `IssueDetailScreen`, `DomainOverviewScreen`, `KanbanScreen`, `ExecutiveReportScreen`
- [ ] App.OnStart set (see EIT-REBUILD-GUIDE.md Step 4)

---

## 1. DashboardScreen

The command centre: enterprise KPI row, pending intake queue, submit overlay, and navigation to all screens.

### 1.1 Header Bar

Insert a **Rectangle**. Rename `rectDashHeader`.

| Property | Value |
|----------|-------|
| X | `0` |
| Y | `0` |
| Width | `1366` |
| Height | `60` |
| Fill | `RGBA(65,83,133,1)` |

Insert a **Label**. Rename `lblDashTitle`.

| Property | Value |
|----------|-------|
| X | `20` |
| Y | `10` |
| Width | `500` |
| Height | `40` |
| Text | `"Enterprise Incident Tracker"` |
| Size | `18` |
| FontWeight | `FontWeight.Bold` |
| Color | `Color.White` |

### 1.2 KPI Card Layout — Overview

Five KPI cards in a horizontal row below the header bar:

| Card | X | Count Label | Subtitle Label |
|------|---|-------------|----------------|
| Open Items | 20 | `lblOpenCount` | `lblOpenSub` |
| Critical | 272 | `lblCritCount` | `lblCritSub` |
| Stale | 524 | `lblStaleCount` | `lblStaleSub` |
| Escalated | 776 | `lblEscCount` | `lblEscSub` |
| P&C Active | 1028 | `lblPACCount` | `lblPACSub` |

### 1.3 KPI Card — Shared Properties

#### Background Rectangle (e.g., `rectOpen`, `rectCrit`, `rectStale`, `rectEsc`, `rectPAC`)

| Property | Value |
|----------|-------|
| Y | `75` |
| Width | `240` |
| Height | `90` |
| Fill | `RGBA(245,245,245,1)` |
| BorderColor | `RGBA(225,225,225,1)` |
| BorderThickness | `1` |
| RadiusTopLeft | `8` |
| RadiusTopRight | `8` |
| RadiusBottomLeft | `8` |
| RadiusBottomRight | `8` |

#### Count Label (e.g., `lblOpenCount`)

| Property | Value |
|----------|-------|
| Y | `78` |
| Width | `230` |
| Height | `45` |
| Size | `24` |
| FontWeight | `FontWeight.Bold` |
| Align | `Align.Center` |
| Color | `RGBA(65,83,133,1)` |

#### Subtitle Label (e.g., `lblOpenSub`)

| Property | Value |
|----------|-------|
| Y | `120` |
| Width | `230` |
| Height | `30` |
| Size | `11` |
| Align | `Align.Center` |
| Color | `RGBA(130,130,130,1)` |

### 1.4 KPI Card Formulas

#### Open Items Card (X=20)

**lblOpenCount.Text:**
```
CountRows(Filter(EIT_Incidents, Status.Value <> "Closed", Status.Value <> "Contained"))
```

**lblOpenSub.Text:**
```
"Open Incidents"
```

#### Critical Card (X=272) — `rectCrit` Fill: `RGBA(119,40,32,0.08)`, `BorderColor: RGBA(119,40,32,0.3)`

**lblCritCount.Text:**
```
CountRows(Filter(EIT_Incidents, Severity.Value = "Critical", Status.Value <> "Closed", Status.Value <> "Contained"))
```

**lblCritCount.Color:** `RGBA(119,40,32,1)`

**lblCritSub.Text:**
```
"Critical Severity"
```

#### Stale Card (X=524) — `rectStale` Fill: `RGBA(224,48,30,0.06)`, `BorderColor: RGBA(224,48,30,0.2)`

**lblStaleCount.Text:**
```
CountRows(Filter(EIT_Incidents, StalenessFlag.Value = "Stale", Status.Value <> "Closed", Status.Value <> "Contained"))
```

**lblStaleCount.Color:** `RGBA(224,48,30,1)`

**lblStaleSub.Text:**
```
"Stale (15+ days)"
```

#### Escalated Card (X=776) — `rectEsc` Fill: `RGBA(228,92,43,0.06)`

**lblEscCount.Text:**
```
CountRows(Filter(EIT_Incidents, EscalationThreshold.Value <> "Not Escalated", Status.Value <> "Closed", Status.Value <> "Contained"))
```

**lblEscCount.Color:** `RGBA(228,92,43,1)`

**lblEscSub.Text:**
```
"Escalated / Exec Brief"
```

#### P&C Active Card (X=1028)

**lblPACCount.Text:**
```
CountRows(Filter(EIT_Incidents, PrivilegedAndConfidential = true, Status.Value <> "Closed", Status.Value <> "Contained"))
```

**lblPACCount.Color:** `RGBA(65,83,133,1)`

**lblPACSub.Text:**
```
"Privileged & Confidential"
```

### 1.5 Navigation Buttons (in header bar)

Insert four **Buttons** in the right portion of the header rectangle. All share:
- `Y = 12`, `Height = 36`, `Color = Color.White`, `Fill = Color.Transparent`, `HoverFill = RGBA(98,113,154,1)`, `BorderColor = Color.White`, `BorderThickness = 1`, `FontWeight = FontWeight.Semibold`, `Size = 12`, `RadiusTopLeft/TopRight/BottomLeft/BottomRight = 6`

| Control | X | Width | Text | OnSelect |
|---------|---|-------|------|----------|
| `btnNavTracker` | 800 | 120 | `"Tracker"` | `Navigate(TrackerScreen, ScreenTransition.None)` |
| `btnNavDomains` | 930 | 140 | `"By Domain"` | `Navigate(DomainOverviewScreen, ScreenTransition.None)` |
| `btnNavKanban` | 1080 | 120 | `"Kanban"` | `Navigate(KanbanScreen, ScreenTransition.None)` |
| `btnNavExec` | 1208 | 140 | `"Exec Report"` | `Navigate(ExecutiveReportScreen, ScreenTransition.None)` |

### 1.6 Intake Queue Gallery

#### Section Header Label — `lblIntakeHeader`

| Property | Value |
|----------|-------|
| X | `20` |
| Y | `175` |
| Width | `700` |
| Height | `30` |
| Text | `"Intake Queue (" & CountRows(Filter(EIT_IntakeQueue, TriageStatus.Value = "Pending")) & " pending)"` |
| Size | `14` |
| FontWeight | `FontWeight.Semibold` |
| Color | `RGBA(65,83,133,1)` |

#### Gallery — `galIntakeQueue`

| Property | Value |
|----------|-------|
| X | `20` |
| Y | `205` |
| Width | `1326` |
| Height | `465` |
| TemplatePadding | `4` |
| TemplateSize | `75` |
| ShowScrollbar | `true` |

**galIntakeQueue.Items:**
```
Sort(
    Filter(
        EIT_IntakeQueue,
        TriageStatus.Value = "Pending"
    ),
    Switch(Priority.Value, "High", 1, "Medium", 2, "Low", 3, 4),
    SortOrder.Ascending,
    DateSubmitted,
    SortOrder.Ascending
)
```

**galIntakeQueue.OnSelect:**
```
UpdateContext({showIntakePanel: true, selectedIntake: ThisItem})
```

#### Controls Inside the Gallery Template

**a) Title — `lblIntakeTitle`**

| Property | Value |
|----------|-------|
| X | `10` |
| Y | `6` |
| Width | `420` |
| Height | `28` |
| Text | `If(Len(ThisItem.Title) > 55, Left(ThisItem.Title, 55) & "...", ThisItem.Title)` |
| Size | `13` |
| FontWeight | `FontWeight.Semibold` |
| Color | `RGBA(65,83,133,1)` |
| Tooltip | `ThisItem.Title` |

**b) Owner — `lblIntakeOwner`**

| Property | Value |
|----------|-------|
| X | `10` |
| Y | `38` |
| Width | `180` |
| Height | `24` |
| Text | `ThisItem.Owner` |
| Size | `11` |
| Color | `RGBA(100,100,100,1)` |

**c) Date — `lblIntakeDate`**

| Property | Value |
|----------|-------|
| X | `200` |
| Y | `38` |
| Width | `180` |
| Height | `24` |
| Text | `"Submitted: " & Text(ThisItem.DateSubmitted, "mm/dd/yyyy")` |
| Size | `11` |
| Color | `RGBA(100,100,100,1)` |

**d) Domain Badge — `lblIntakeDomain`**

| Property | Value |
|----------|-------|
| X | `430` |
| Y | `38` |
| Width | `150` |
| Height | `24` |
| Text | `ThisItem.Domain.Value` |
| Size | `10` |
| Color | `RGBA(65,83,133,1)` |
| FontWeight | `FontWeight.Semibold` |

**e) IncidentType — `lblIntakeType`**

| Property | Value |
|----------|-------|
| X | `590` |
| Y | `38` |
| Width | `120` |
| Height | `24` |
| Text | `ThisItem.IncidentType.Value` |
| Size | `10` |
| Color | `RGBA(130,130,130,1)` |

**f) Priority Badge — `lblIntakePriority`**

| Property | Value |
|----------|-------|
| X | `740` |
| Y | `12` |
| Width | `80` |
| Height | `25` |
| Text | `ThisItem.Priority.Value` |
| Size | `11` |
| FontWeight | `FontWeight.Bold` |
| Align | `Align.Center` |
| Color | `Color.White` |
| Fill | `Switch(ThisItem.Priority.Value, "High", RGBA(224,48,30,1), "Medium", RGBA(228,92,43,1), "Low", RGBA(65,83,133,1), RGBA(180,180,180,1))` |
| RadiusTopLeft | `4` |
| RadiusTopRight | `4` |
| RadiusBottomLeft | `4` |
| RadiusBottomRight | `4` |

**g) P&C Indicator — `lblIntakePAC`**

| Property | Value |
|----------|-------|
| X | `830` |
| Y | `12` |
| Width | `70` |
| Height | `25` |
| Text | `"P&C"` |
| Size | `10` |
| FontWeight | `FontWeight.Bold` |
| Align | `Align.Center` |
| Color | `Color.White` |
| Fill | `RGBA(119,40,32,1)` |
| Visible | `ThisItem.PrivilegedAndConfidential` |
| RadiusTopLeft | `4` |
| RadiusTopRight | `4` |
| RadiusBottomLeft | `4` |
| RadiusBottomRight | `4` |

**h) Promote Button — `btnIntakePromote`**

| Property | Value |
|----------|-------|
| X | `1010` |
| Y | `15` |
| Width | `160` |
| Height | `36` |
| Text | `"Move to Tracker"` |
| Size | `11` |
| FontWeight | `FontWeight.Semibold` |
| Color | `Color.White` |
| Fill | `RGBA(208,74,2,1)` |
| HoverFill | `RGBA(195,76,47,1)` |
| RadiusTopLeft | `6` |
| RadiusTopRight | `6` |
| RadiusBottomLeft | `6` |
| RadiusBottomRight | `6` |

**btnIntakePromote.OnSelect:**
```
Set(
    varPromoteResult,
    EITIntakePromotion.Run(Text(ThisItem.ID))
);
Notify(
    "Incident promoted as " & varPromoteResult.newitemid,
    NotificationType.Success
);
Refresh(EIT_IntakeQueue);
Refresh(EIT_Incidents)
```

**i) Dismiss Button — `btnIntakeDismiss`**

| Property | Value |
|----------|-------|
| X | `1180` |
| Y | `15` |
| Width | `90` |
| Height | `36` |
| Text | `"Dismiss"` |
| Size | `11` |
| Color | `RGBA(100,100,100,1)` |
| Fill | `RGBA(240,240,240,1)` |
| HoverFill | `RGBA(220,220,220,1)` |
| RadiusTopLeft | `6` |
| RadiusTopRight | `6` |
| RadiusBottomLeft | `6` |
| RadiusBottomRight | `6` |

**btnIntakeDismiss.OnSelect:**
```
Patch(
    EIT_IntakeQueue,
    ThisItem,
    {TriageStatus: {Value: "Dismissed"}}
);
Notify("Item dismissed", NotificationType.Warning);
Refresh(EIT_IntakeQueue)
```

### 1.7 Submit New Incident Overlay

The "+ New Incident" button opens a modal overlay on DashboardScreen.

#### "+ New Incident" Button — `btnNewIncident`

| Property | Value |
|----------|-------|
| X | `1100` |
| Y | `170` |
| Width | `160` |
| Height | `38` |
| Text | `"+ New Incident"` |
| Size | `13` |
| FontWeight | `FontWeight.Semibold` |
| Color | `Color.White` |
| Fill | `RGBA(208,74,2,1)` |
| HoverFill | `RGBA(195,76,47,1)` |
| RadiusTopLeft | `6` |
| RadiusTopRight | `6` |
| RadiusBottomLeft | `6` |
| RadiusBottomRight | `6` |

**btnNewIncident.OnSelect:**
```
UpdateContext({varShowNewIncident: true})
```

#### Overlay Background — `rectOverlayBg`

| Property | Value |
|----------|-------|
| X | `0` |
| Y | `0` |
| Width | `1366` |
| Height | `768` |
| Fill | `RGBA(0,0,0,0.5)` |
| Visible | `varShowNewIncident` |

#### Overlay Card — `rectOverlayCard`

| Property | Value |
|----------|-------|
| X | `333` |
| Y | `90` |
| Width | `700` |
| Height | `575` |
| Fill | `Color.White` |
| RadiusTopLeft | `12` |
| RadiusTopRight | `12` |
| RadiusBottomLeft | `12` |
| RadiusBottomRight | `12` |
| Visible | `varShowNewIncident` |

All overlay controls below have `Visible = varShowNewIncident`.

#### Overlay Header — `lblOverlayTitle`

| Property | Value |
|----------|-------|
| X | `353` |
| Y | `105` |
| Width | `500` |
| Height | `40` |
| Text | `"Submit New Incident"` |
| Size | `18` |
| FontWeight | `FontWeight.Bold` |
| Color | `RGBA(65,83,133,1)` |

#### Close Button — `btnOverlayClose`

| Property | Value |
|----------|-------|
| X | `993` |
| Y | `105` |
| Width | `40` |
| Height | `40` |
| Text | `"X"` |
| Size | `16` |
| FontWeight | `FontWeight.Bold` |
| Color | `RGBA(100,100,100,1)` |
| Fill | `Color.Transparent` |

**btnOverlayClose.OnSelect:**
```
UpdateContext({varShowNewIncident: false});
Reset(txtNewTitle);
Reset(txtNewOwner);
Reset(ddNewPriority);
Reset(ddNewDomain);
Reset(ddNewIncidentType);
Reset(txtNewDescription)
```

#### Title Field — `txtNewTitle`

| Property | Value |
|----------|-------|
| X | `353` |
| Y | `165` |
| Width | `660` |
| Height | `40` |
| HintText | `"Incident title (required)"` |
| Mode | `TextMode.SingleLine` |
| BorderColor | `RGBA(200,200,200,1)` |
| FocusedBorderColor | `RGBA(208,74,2,1)` |

Label above (`lblNewTitleLabel`): X=353, Y=150, Text=`"Title *"`, Size=11, FontWeight=Bold, Color=`RGBA(65,83,133,1)`

#### Owner Field — `txtNewOwner`

| Property | Value |
|----------|-------|
| X | `353` |
| Y | `235` |
| Width | `660` |
| Height | `40` |
| HintText | `"Incident owner"` |
| Default | `User().FullName` |
| BorderColor | `RGBA(200,200,200,1)` |
| FocusedBorderColor | `RGBA(208,74,2,1)` |

Label above (`lblNewOwnerLabel`): X=353, Y=220, Text=`"Owner"`, Size=11, FontWeight=Bold, Color=`RGBA(65,83,133,1)`

#### Domain Dropdown — `ddNewDomain`

| Property | Value |
|----------|-------|
| X | `353` |
| Y | `305` |
| Width | `315` |
| Height | `40` |
| Items | `["National Security", "NIS", "Privacy & Technology", "Human Capital", "Physical Security"]` |
| BorderColor | `RGBA(200,200,200,1)` |
| ChevronBackground | `RGBA(65,83,133,1)` |

Label above (`lblNewDomainLabel`): X=353, Y=290, Text=`"Domain *"`, Size=11, FontWeight=Bold, Color=`RGBA(65,83,133,1)`

#### IncidentType Dropdown — `ddNewIncidentType`

| Property | Value |
|----------|-------|
| X | `688` |
| Y | `305` |
| Width | `325` |
| Height | `40` |
| Items | `["Breach", "Vulnerability", "Policy Violation", "Threat", "Operational", "Personnel", "Physical"]` |
| BorderColor | `RGBA(200,200,200,1)` |
| ChevronBackground | `RGBA(65,83,133,1)` |

Label above (`lblNewTypeLabel`): X=688, Y=290, Text=`"Incident Type *"`, Size=11, FontWeight=Bold, Color=`RGBA(65,83,133,1)`

#### Priority Dropdown — `ddNewPriority`

| Property | Value |
|----------|-------|
| X | `353` |
| Y | `375` |
| Width | `200` |
| Height | `40` |
| Items | `["High", "Medium", "Low"]` |
| Default | `"Medium"` |
| BorderColor | `RGBA(200,200,200,1)` |
| ChevronBackground | `RGBA(208,74,2,1)` |

Label above (`lblNewPriorityLabel`): X=353, Y=360, Text=`"Priority *"`, Size=11, FontWeight=Bold, Color=`RGBA(65,83,133,1)`

#### P&C Toggle — `togNewPAC`

Insert a **Toggle** control. Rename `togNewPAC`.

| Property | Value |
|----------|-------|
| X | `600` |
| Y | `375` |
| Width | `200` |
| Height | `40` |
| Default | `false` |
| OnText | `"P&C: Yes"` |
| OffText | `"P&C: No"` |
| FocusedBorderColor | `RGBA(208,74,2,1)` |

Label above (`lblNewPACLabel`): X=600, Y=360, Text=`"Privileged & Confidential"`, Size=11, FontWeight=Bold, Color=`RGBA(65,83,133,1)`

> **Note:** Staff submitting incidents may suggest P&C designation here. Domain lead or legal counsel must confirm the designation during triage. See `EIT-Access-Control-Guide.md`.

#### Description Field — `txtNewDescription`

| Property | Value |
|----------|-------|
| X | `353` |
| Y | `445` |
| Width | `660` |
| Height | `110` |
| HintText | `"Describe the incident..."` |
| Mode | `TextMode.MultiLine` |
| BorderColor | `RGBA(200,200,200,1)` |
| FocusedBorderColor | `RGBA(208,74,2,1)` |

Label above (`lblNewDescLabel`): X=353, Y=430, Text=`"Description"`, Size=11, FontWeight=Bold, Color=`RGBA(65,83,133,1)`

#### Submit Button — `btnSubmitNewIncident`

| Property | Value |
|----------|-------|
| X | `353` |
| Y | `575` |
| Width | `160` |
| Height | `40` |
| Text | `"Submit"` |
| Size | `14` |
| FontWeight | `FontWeight.Semibold` |
| Color | `Color.White` |
| Fill | `RGBA(208,74,2,1)` |
| HoverFill | `RGBA(195,76,47,1)` |
| DisabledFill | `RGBA(200,200,200,1)` |
| DisplayMode | `If(IsBlank(txtNewTitle.Text) \|\| IsBlank(ddNewDomain.Selected.Value), DisplayMode.Disabled, DisplayMode.Edit)` |
| RadiusTopLeft | `6` |
| RadiusTopRight | `6` |
| RadiusBottomLeft | `6` |
| RadiusBottomRight | `6` |

**btnSubmitNewIncident.OnSelect:**
```
Patch(
    EIT_IntakeQueue,
    Defaults(EIT_IntakeQueue),
    {
        Title: txtNewTitle.Text,
        Owner: txtNewOwner.Text,
        Priority: {Value: ddNewPriority.Selected.Value},
        Domain: {Value: ddNewDomain.Selected.Value},
        IncidentType: {Value: ddNewIncidentType.Selected.Value},
        Description: txtNewDescription.Text,
        DateSubmitted: Now(),
        TriageStatus: {Value: "Pending"},
        SubmissionSource: {Value: "Power Apps"},
        PrivilegedAndConfidential: togNewPAC.Value
    }
);
Reset(txtNewTitle);
Reset(txtNewOwner);
Reset(ddNewPriority);
Reset(ddNewDomain);
Reset(ddNewIncidentType);
Reset(txtNewDescription);
UpdateContext({varShowNewIncident: false});
Notify("Incident submitted to intake queue", NotificationType.Success);
Refresh(EIT_IntakeQueue)
```

#### Cancel Button — `btnCancelNewIncident`

| Property | Value |
|----------|-------|
| X | `523` |
| Y | `575` |
| Width | `120` |
| Height | `40` |
| Text | `"Cancel"` |
| Size | `14` |
| Color | `RGBA(100,100,100,1)` |
| Fill | `RGBA(240,240,240,1)` |

**btnCancelNewIncident.OnSelect:**
```
UpdateContext({varShowNewIncident: false});
Reset(txtNewTitle);
Reset(txtNewOwner);
Reset(ddNewPriority);
Reset(ddNewDomain);
Reset(ddNewIncidentType);
Reset(txtNewDescription)
```

### 1.8 Intake Review Panel (Side Panel)

See `EIT-REBUILD-GUIDE.md` Section 6 for the full panel control list and formulas. Position the panel at X=886, Y=0, Width=480, Height=768 (right-side slide-in).

Key panel controls summary:

| Control | Rename | Key Property |
|---------|--------|-------------|
| Rectangle (backdrop) | `rectIntakePanel` | `Visible: showIntakePanel`, Fill=White, Width=480 |
| Close button | `btnIntakePanelClose` | `OnSelect: UpdateContext({showIntakePanel: false})` |
| P&C warning label | `lblPanelPAC` | `Visible: selectedIntake.PrivilegedAndConfidential` |
| Promote via Flow | `btnPanelPromote` | `OnSelect: EITIntakePromotion.Run(...)` |
| Accept | `btnPanelAccept` | `OnSelect: Patch(EIT_IntakeQueue, selectedIntake, {TriageStatus: {Value: "Accepted"}})` |
| Reject | `btnPanelReject` | `OnSelect: Patch(EIT_IntakeQueue, selectedIntake, {TriageStatus: {Value: "Rejected"}})` |

### 1.9 DashboardScreen.OnVisible

```
UpdateContext({varShowNewIncident: false, showIntakePanel: false});
Refresh(EIT_Incidents);
Refresh(EIT_IntakeQueue)
```

---

## 2. TrackerScreen (Main Incident Table)

### 2.1 Navy Header Bar

Insert a **Rectangle**. Rename `rectTrackerHeader`.

| Property | Value |
|----------|-------|
| X | `0` |
| Y | `0` |
| Width | `1366` |
| Height | `60` |
| Fill | `RGBA(65,83,133,1)` |

Insert a **Label**. Rename `lblTrackerTitle`.

| Property | Value |
|----------|-------|
| X | `20` |
| Y | `10` |
| Width | `400` |
| Height | `40` |
| Text | `"Incident Tracker"` |
| Size | `18` |
| FontWeight | `FontWeight.Bold` |
| Color | `Color.White` |

Insert a **back button**. Rename `btnTrackerBack`.

| Property | Value |
|----------|-------|
| X | `1200` |
| Y | `12` |
| Width | `140` |
| Height | `36` |
| Text | `"< Dashboard"` |
| Color | `Color.White` |
| Fill | `Color.Transparent` |
| HoverFill | `RGBA(98,113,154,1)` |
| BorderColor | `Color.White` |
| BorderThickness | `1` |
| FontWeight | `FontWeight.Semibold` |
| OnSelect | `Navigate(DashboardScreen, ScreenTransition.None)` |
| RadiusTopLeft/TopRight/BottomLeft/BottomRight | `6` |

### 2.2 TrackerScreen.OnVisible

```
UpdateContext({varDomainFilter: "All", varSearch: "", varShowContained: false});
Refresh(EIT_Incidents)
```

### 2.3 Domain Filter Chips

A row of filter buttons below the header acting as a toggle bar. One is active at a time, controlled by `varDomainFilter`.

Create **7 buttons** in a horizontal row:

| Control | X | Width | Text | varDomainFilter value |
|---------|---|-------|------|-----------------------|
| `btnFilterAll` | 20 | 90 | `"All Open"` | `"All"` |
| `btnFilterNS` | 118 | 155 | `"Nat'l Security"` | `"NS"` |
| `btnFilterNIS` | 281 | 75 | `"NIS"` | `"NIS"` |
| `btnFilterPT` | 364 | 130 | `"Privacy & Tech"` | `"PT"` |
| `btnFilterHC` | 502 | 125 | `"Human Capital"` | `"HC"` |
| `btnFilterPS` | 635 | 135 | `"Physical Sec"` | `"PS"` |
| `btnFilterCrit` | 778 | 100 | `"Critical"` | `"Critical"` |

Shared properties:
- `Y = 68`, `Height = 34`, `Size = 12`, `FontWeight = FontWeight.Semibold`
- `RadiusTopLeft/TopRight/BottomLeft/BottomRight = 6`
- Active state: `Color = Color.White`, `Fill = RGBA(65,83,133,1)`
- Inactive state: `Color = RGBA(65,83,133,1)`, `Fill = RGBA(240,240,240,1)`

**Color formula pattern (e.g., for btnFilterNS):**

```
Color: If(varDomainFilter = "NS", Color.White, RGBA(65,83,133,1))
Fill: If(varDomainFilter = "NS", RGBA(65,83,133,1), RGBA(240,240,240,1))
```

**btnFilterCrit colors (uses red):**
```
Color: If(varDomainFilter = "Critical", Color.White, RGBA(119,40,32,1))
Fill: If(varDomainFilter = "Critical", RGBA(119,40,32,1), RGBA(240,240,240,1))
```

**OnSelect for each button:**
```
UpdateContext({varDomainFilter: "[value]"})
```
Replace `[value]` with the varDomainFilter value for each button (e.g., `"NS"`, `"NIS"`, etc.)

#### Show Contained Toggle — `togShowContained`

Insert a **Toggle** to the right of the filter chips:

| Property | Value |
|----------|-------|
| X | `900` |
| Y | `70` |
| Width | `170` |
| Height | `30` |
| Default | `false` |
| OnText | `"Incl. Contained"` |
| OffText | `"Hide Contained"` |
| OnChange | `UpdateContext({varShowContained: Self.Value})` |

### 2.4 Search Box — `txtSearch`

| Property | Value |
|----------|-------|
| X | `1080` |
| Y | `68` |
| Width | `250` |
| Height | `34` |
| HintText | `"Search by Title, Owner, ID, Domain..."` |
| BorderColor | `RGBA(200,200,200,1)` |
| FocusedBorderColor | `RGBA(208,74,2,1)` |
| Size | `12` |

**txtSearch.OnChange:**
```
UpdateContext({varSearch: Self.Text})
```

### 2.5 Column Headers Row

All headers: `Y = 108`, `Height = 28`, `Size = 11`, `FontWeight = FontWeight.Bold`, `Color = RGBA(65,83,133,1)`, `Fill = RGBA(235,235,235,1)`.

| Control | X | Width | Text |
|---------|---|-------|------|
| `lblColID` | 20 | 80 | `"Item ID"` |
| `lblColTitle` | 105 | 220 | `"Title"` |
| `lblColOwner` | 330 | 130 | `"Owner"` |
| `lblColDomain` | 465 | 130 | `"Domain"` |
| `lblColSeverity` | 600 | 90 | `"Severity"` |
| `lblColStatus` | 695 | 90 | `"Status"` |
| `lblColEsc` | 790 | 90 | `"Escalation"` |
| `lblColDays` | 885 | 75 | `"Days Stale"` |
| `lblColNextAction` | 965 | 361 | `"Next Action"` |

### 2.6 Incident Table Gallery — `galIncidentTable`

| Property | Value |
|----------|-------|
| X | `20` |
| Y | `138` |
| Width | `1326` |
| Height | `580` |
| TemplatePadding | `2` |
| TemplateSize | `48` |
| ShowScrollbar | `true` |

**galIncidentTable.Items:**
```
SortByColumns(
    Filter(
        EIT_Incidents,
        Status.Value <> "Closed",
        If(varShowContained, true, Status.Value <> "Contained"),
        If(
            varDomainFilter = "NS", Domain.Value = "National Security",
            varDomainFilter = "NIS", Domain.Value = "NIS",
            varDomainFilter = "PT", Domain.Value = "Privacy & Technology",
            varDomainFilter = "HC", Domain.Value = "Human Capital",
            varDomainFilter = "PS", Domain.Value = "Physical Security",
            varDomainFilter = "Critical", Severity.Value = "Critical",
            varDomainFilter = "Stale", StalenessFlag.Value = "Stale",
            true
        ),
        Or(
            IsBlank(varSearch),
            varSearch in Title,
            varSearch in Owner,
            varSearch in ItemID,
            varSearch in Domain.Value,
            varSearch in IncidentType.Value
        ),
        Or(
            Not(PrivilegedAndConfidential),
            varUserHasPACAccess
        )
    ),
    "DaysSinceUpdate",
    SortOrder.Descending
)
```

**galIncidentTable.OnSelect:**
```
Navigate(
    IssueDetailScreen,
    ScreenTransition.None,
    {varSelectedIncident: ThisItem}
)
```

**galIncidentTable.TemplateFill:**
```
If(ThisItem.IsSelected, RGBA(210,215,226,1), Color.Transparent)
```

#### Controls Inside Gallery Template

**a) Item ID — `lblRowID`**

| Property | Value |
|----------|-------|
| X | `0` |
| Y | `6` |
| Width | `80` |
| Height | `35` |
| Text | `ThisItem.ItemID` |
| Size | `11` |
| Color | `RGBA(65,83,133,1)` |
| FontWeight | `FontWeight.Semibold` |

**b) Title — `lblRowTitle`**

| Property | Value |
|----------|-------|
| X | `105` |
| Y | `6` |
| Width | `220` |
| Height | `35` |
| Text | `If(Len(ThisItem.Title) > 35, Left(ThisItem.Title, 35) & "...", ThisItem.Title)` |
| Size | `11` |
| Color | `RGBA(45,45,45,1)` |
| Tooltip | `ThisItem.Title` |

**c) Owner — `lblRowOwner`**

| Property | Value |
|----------|-------|
| X | `330` |
| Y | `6` |
| Width | `130` |
| Height | `35` |
| Text | `ThisItem.Owner` |
| Size | `11` |
| Color | `RGBA(80,80,80,1)` |

**d) Domain — `lblRowDomain`**

| Property | Value |
|----------|-------|
| X | `465` |
| Y | `6` |
| Width | `130` |
| Height | `35` |
| Text | `If(IsBlank(ThisItem.Domain.Value), "-", If(Len(ThisItem.Domain.Value) > 16, Left(ThisItem.Domain.Value, 16) & "...", ThisItem.Domain.Value))` |
| Size | `10` |
| Color | `RGBA(65,83,133,1)` |
| Tooltip | `ThisItem.Domain.Value` |

**e) Severity Badge — `lblRowSeverity`**

| Property | Value |
|----------|-------|
| X | `600` |
| Y | `10` |
| Width | `90` |
| Height | `25` |
| Text | `ThisItem.Severity.Value` |
| Size | `9` |
| FontWeight | `FontWeight.Bold` |
| Align | `Align.Center` |
| Color | `Color.White` |
| Fill | `Switch(ThisItem.Severity.Value, "Critical", RGBA(119,40,32,1), "High", RGBA(224,48,30,1), "Moderate", RGBA(228,92,43,1), "Low", RGBA(65,83,133,1), RGBA(150,150,150,1))` |
| RadiusTopLeft | `4` |
| RadiusTopRight | `4` |
| RadiusBottomLeft | `4` |
| RadiusBottomRight | `4` |

**f) Status Badge — `lblRowStatus`**

| Property | Value |
|----------|-------|
| X | `695` |
| Y | `10` |
| Width | `90` |
| Height | `25` |
| Text | `ThisItem.Status.Value` |
| Size | `9` |
| Align | `Align.Center` |
| Color | `Color.White` |
| FontWeight | `FontWeight.Semibold` |
| Fill | `Switch(ThisItem.Status.Value, "New", RGBA(65,83,133,1), "Active", RGBA(208,74,2,1), "Monitoring", RGBA(228,92,43,1), "Escalated", RGBA(224,48,30,1), "Contained", RGBA(50,140,80,1), RGBA(140,140,140,1))` |
| RadiusTopLeft | `4` |
| RadiusTopRight | `4` |
| RadiusBottomLeft | `4` |
| RadiusBottomRight | `4` |

**g) Escalation Badge — `lblRowEsc`**

| Property | Value |
|----------|-------|
| X | `790` |
| Y | `10` |
| Width | `90` |
| Height | `25` |
| Text | `ThisItem.EscalationThreshold.Value` |
| Size | `9` |
| Align | `Align.Center` |
| Color | `If(ThisItem.EscalationThreshold.Value = "Not Escalated", RGBA(130,130,130,1), Color.White)` |
| Fill | `Switch(ThisItem.EscalationThreshold.Value, "Executive Brief", RGBA(119,40,32,1), "Escalate", RGBA(224,48,30,1), "Watch", RGBA(228,92,43,1), RGBA(235,235,235,1))` |
| RadiusTopLeft | `4` |
| RadiusTopRight | `4` |
| RadiusBottomLeft | `4` |
| RadiusBottomRight | `4` |

**h) Days Stale — `lblRowDays`**

| Property | Value |
|----------|-------|
| X | `885` |
| Y | `6` |
| Width | `75` |
| Height | `35` |
| Text | `Text(ThisItem.DaysSinceUpdate) & " days"` |
| Size | `11` |
| FontWeight | `FontWeight.Bold` |
| Align | `Align.Center` |
| Color | `If(ThisItem.DaysSinceUpdate <= 7, RGBA(65,83,133,1), ThisItem.DaysSinceUpdate <= 14, RGBA(228,92,43,1), RGBA(224,48,30,1))` |

**i) Next Action — `lblRowNextAction`**

| Property | Value |
|----------|-------|
| X | `965` |
| Y | `6` |
| Width | `361` |
| Height | `35` |
| Text | `If(Len(ThisItem.NextAction) > 50, Left(ThisItem.NextAction, 50) & "...", ThisItem.NextAction)` |
| Size | `11` |
| Color | `RGBA(80,80,80,1)` |
| Tooltip | `ThisItem.NextAction` |

**j) P&C Row Indicator — `lblRowPAC`**

| Property | Value |
|----------|-------|
| X | `0` |
| Y | `0` |
| Width | `5` |
| Height | `48` |
| Text | `""` |
| Fill | `If(ThisItem.PrivilegedAndConfidential, RGBA(119,40,32,1), Color.Transparent)` |

**k) Row Separator — `rectRowSep`**

| Property | Value |
|----------|-------|
| X | `0` |
| Y | `45` |
| Width | `1326` |
| Height | `1` |
| Fill | `RGBA(225,225,225,1)` |

### 2.7 Results Count Label — `lblResultCount`

| Property | Value |
|----------|-------|
| X | `886` |
| Y | `70` |
| Width | `185` |
| Height | `30` |
| Text | `CountRows(galIncidentTable.AllItems) & " incidents"` |
| Size | `11` |
| Color | `RGBA(130,130,130,1)` |
| Align | `Align.Right` |

---

## 3. IssueDetailScreen

### 3.1 Navy Header Bar

Insert **Rectangle** `rectDetailHeader` — same as TrackerScreen (X=0, Y=0, W=1366, H=60, Fill navy).

**lblDetailTitle.Text:**
```
"Incident Detail: " & varSelectedIncident.ItemID
```

| Property | Value |
|----------|-------|
| X | `20` |
| Y | `10` |
| Width | `600` |
| Height | `40` |
| Size | `18` |
| FontWeight | `FontWeight.Bold` |
| Color | `Color.White` |

**btnBackToTracker:**
- X=1200, Y=12, W=140, H=36, Text=`"< Back"`, transparent style
- `OnSelect: Navigate(TrackerScreen, ScreenTransition.None)`

### 3.2 P&C Warning Banner

This banner appears only when the incident is marked Privileged & Confidential. Insert a **Rectangle** and a **Label** stacked together.

**`rectPACBanner`:**

| Property | Value |
|----------|-------|
| X | `20` |
| Y | `68` |
| Width | `1326` |
| Height | `36` |
| Fill | `RGBA(228,92,43,0.15)` |
| BorderColor | `RGBA(228,92,43,1)` |
| BorderThickness | `1` |
| RadiusTopLeft | `4` |
| RadiusTopRight | `4` |
| RadiusBottomLeft | `4` |
| RadiusBottomRight | `4` |
| Visible | `varSelectedIncident.PrivilegedAndConfidential` |

**`lblPACBanner`:**

| Property | Value |
|----------|-------|
| X | `35` |
| Y | `75` |
| Width | `1300` |
| Height | `22` |
| Text | `"⚠ PRIVILEGED & CONFIDENTIAL — This incident is restricted. Access is limited to the authorized list. Do not discuss or forward outside the authorized list."` |
| Size | `11` |
| FontWeight | `FontWeight.Semibold` |
| Color | `RGBA(119,40,32,1)` |
| Visible | `varSelectedIncident.PrivilegedAndConfidential` |

> **When the banner is visible**, the incident header card below shifts down. Set `rectIncidentCard.Y = If(varSelectedIncident.PrivilegedAndConfidential, 112, 70)`.

### 3.3 Incident Header Card

#### Background Card — `rectIncidentCard`

| Property | Value |
|----------|-------|
| X | `20` |
| Y | `If(varSelectedIncident.PrivilegedAndConfidential, 112, 70)` |
| Width | `1326` |
| Height | `150` |
| Fill | `RGBA(245,245,245,1)` |
| RadiusTopLeft | `8` |
| RadiusTopRight | `8` |
| RadiusBottomLeft | `8` |
| RadiusBottomRight | `8` |

#### Field Labels and Values

Field label properties: `Size = 10`, `FontWeight = FontWeight.Bold`, `Color = RGBA(130,130,130,1)`, `Height = 18`.
Value label properties: `Size = 13`, `FontWeight = FontWeight.Semibold`, `Height = 25`.

All Y coordinates below are relative to the card Y (use `rectIncidentCard.Y +` offset in your formulas for dynamic positioning, or use fixed values calculated for P&C=No).

**Row 1 (labels at rectIncidentCard.Y+8, values at rectIncidentCard.Y+28):**

| Field | X | Width | Label Text | Value Formula |
|-------|---|-------|-----------|---------------|
| Item ID | 35 | 80 | `"ITEM ID"` | `varSelectedIncident.ItemID` |
| Title | 130 | 380 | `"TITLE"` | `varSelectedIncident.Title` |
| Owner | 525 | 180 | `"OWNER"` | `varSelectedIncident.Owner` |
| Domain | 720 | 160 | `"DOMAIN"` | `varSelectedIncident.Domain.Value` |
| IncidentType | 895 | 160 | `"INCIDENT TYPE"` | `varSelectedIncident.IncidentType.Value` |

**Row 2 (labels at rectIncidentCard.Y+68, values at rectIncidentCard.Y+88):**

| Field | X | Width | Label Text | Value |
|-------|---|-------|-----------|-------|
| Severity | 35 | 90 | `"SEVERITY"` | Badge (see below) |
| Status | 140 | 100 | `"STATUS"` | Badge (see below) |
| Escalation | 255 | 130 | `"ESCALATION"` | Badge (see below) |
| Date Raised | 400 | 150 | `"DATE RAISED"` | `Text(varSelectedIncident.DateRaised, "mmm dd, yyyy")` |
| Last Updated | 565 | 150 | `"LAST UPDATED"` | `Text(varSelectedIncident.LastUpdated, "mmm dd, yyyy")` |
| Days Stale | 730 | 120 | `"DAYS STALE"` | see lblDetailDays below |
| Confidentiality | 865 | 130 | `"CLASSIFICATION"` | `varSelectedIncident.ConfidentialityLevel.Value` |

**Severity Badge — `lblDetailSeverity`:**
```
// Fill:
Switch(varSelectedIncident.Severity.Value,
    "Critical", RGBA(119,40,32,1),
    "High", RGBA(224,48,30,1),
    "Moderate", RGBA(228,92,43,1),
    "Low", RGBA(65,83,133,1),
    RGBA(150,150,150,1)
)
```
Width=90, Height=24, Color=White, FontWeight=Bold, Align=Center, Radius=4

**Status Badge — `lblDetailStatus`:**
```
// Fill:
Switch(varSelectedIncident.Status.Value,
    "New", RGBA(65,83,133,1),
    "Active", RGBA(208,74,2,1),
    "Monitoring", RGBA(228,92,43,1),
    "Escalated", RGBA(224,48,30,1),
    "Contained", RGBA(50,140,80,1),
    RGBA(140,140,140,1)
)
```
Width=100, Height=24, Color=White, FontWeight=Semibold, Align=Center, Radius=4

**Escalation Badge — `lblDetailEscalation`:**
```
// Fill:
Switch(varSelectedIncident.EscalationThreshold.Value,
    "Executive Brief", RGBA(119,40,32,1),
    "Escalate", RGBA(224,48,30,1),
    "Watch", RGBA(228,92,43,1),
    RGBA(220,220,220,1)
)
// Color:
If(varSelectedIncident.EscalationThreshold.Value = "Not Escalated",
    RGBA(100,100,100,1), Color.White)
```
Width=130, Height=24, FontWeight=Semibold, Align=Center, Radius=4

**Days Stale — `lblDetailDays`:**

```
// Text:
Text(varSelectedIncident.DaysSinceUpdate) & " days"
// Color:
If(varSelectedIncident.DaysSinceUpdate <= 7, RGBA(65,83,133,1),
    varSelectedIncident.DaysSinceUpdate <= 14, RGBA(228,92,43,1),
    RGBA(224,48,30,1))
```
Width=120, Height=24, Size=13, FontWeight=Bold, Align=Center

#### Next Action — `lblDetailNextAction`

| Property | Value |
|----------|-------|
| X | `35` |
| Y | `rectIncidentCard.Y + 125` |
| Width | `1280` |
| Height | `25` |
| Text | `"Next Action: " & varSelectedIncident.NextAction` |
| Size | `12` |
| Color | `RGBA(45,45,45,1)` |
| Overflow | `Overflow.Scroll` |

### 3.4 Update History Timeline

#### Section Header — `lblHistoryHeader`

| Property | Value |
|----------|-------|
| X | `20` |
| Y | `280` |
| Width | `300` |
| Height | `30` |
| Text | `"Update History"` |
| Size | `14` |
| FontWeight | `FontWeight.Bold` |
| Color | `RGBA(65,83,133,1)` |

#### History Gallery — `galUpdateHistory`

| Property | Value |
|----------|-------|
| X | `20` |
| Y | `315` |
| Width | `860` |
| Height | `415` |
| TemplatePadding | `4` |
| TemplateSize | `100` |
| ShowScrollbar | `true` |

**galUpdateHistory.Items:**
```
SortByColumns(
    Filter(
        EIT_IncidentHistory,
        ParentItemID = varSelectedIncident.ItemID
    ),
    "UpdateDate",
    SortOrder.Descending
)
```

#### Controls Inside History Gallery Template

**a) Timeline Dot — `rectHistDot`:**
X=0, Y=10, W=12, H=12, Fill=`RGBA(208,74,2,1)`, Radius=6

**b) Timeline Line — `rectHistLine`:**
X=5, Y=22, W=2, H=78, Fill=`RGBA(200,200,200,1)`

**c) Date — `lblHistDate`:**
X=22, Y=4, W=200, H=22
```
Text(ThisItem.UpdateDate, "mmm dd, yyyy - hh:mm AM/PM")
```
Size=11, FontWeight=Semibold, Color=`RGBA(65,83,133,1)`

**d) UpdateType Badge — `lblHistType`:**
X=232, Y=4, W=100, H=22, Size=9, FontWeight=Bold, Align=Center, Color=White
```
// Text:
ThisItem.UpdateType.Value
// Fill:
Switch(ThisItem.UpdateType.Value,
    "Escalation", RGBA(224,48,30,1),
    "Closure", RGBA(50,140,80,1),
    "Containment", RGBA(50,140,80,1),
    "Evidence Added", RGBA(65,83,133,1),
    RGBA(140,140,140,1)
)
```
Radius=4

**e) Status Badge — `lblHistStatus`:**
X=342, Y=4, W=90, H=22, same Fill formula as status badges, Size=9, FontWeight=Bold, Align=Center, Color=White, Radius=4

**f) Severity Badge — `lblHistSeverity`:**
X=442, Y=4, W=90, H=22, same Severity Fill formula, Size=9, FontWeight=Bold, Align=Center, Color=White, Radius=4

**g) Updated By — `lblHistBy`:**
X=543, Y=4, W=200, H=22
```
"by " & ThisItem.UpdatedBy
```
Size=10, Color=`RGBA(130,130,130,1)`

**h) P&C Indicator — `lblHistPAC`:**
X=753, Y=4, W=40, H=22
Text=`"P&C"`, Visible=`ThisItem.PrivilegedAndConfidential`, Fill=`RGBA(119,40,32,1)`, Color=White, Size=9, Bold, Align=Center, Radius=4

**i) Notes — `lblHistNotes`:**
X=22, Y=30, W=820, H=65
```
ThisItem.Notes
```
Size=11, Color=`RGBA(45,45,45,1)`, Overflow=Overflow.Scroll, Italic=`ThisItem.PrivilegedAndConfidential`

### 3.5 Add Update Form

#### Form Card — `rectAddUpdateCard`

| Property | Value |
|----------|-------|
| X | `900` |
| Y | `280` |
| Width | `446` |
| Height | `465` |
| Fill | `RGBA(245,245,245,1)` |
| RadiusTopLeft | `8` |
| RadiusTopRight | `8` |
| RadiusBottomLeft | `8` |
| RadiusBottomRight | `8` |

#### Form Header — `lblAddUpdateHeader`

X=920, Y=290, W=300, H=30, Text=`"Add Update"`, Size=14, FontWeight=Bold, Color=`RGBA(65,83,133,1)`

#### Update Type Dropdown — `ddUpdateType`

| Property | Value |
|----------|-------|
| X | `920` |
| Y | `335` |
| Width | `406` |
| Height | `40` |
| Items | `["Status Change", "Evidence Added", "Escalation", "Containment", "Closure"]` |
| Default | `"Status Change"` |
| BorderColor | `RGBA(200,200,200,1)` |
| ChevronBackground | `RGBA(208,74,2,1)` |

Label above (`lblUpdateTypeLabel`): X=920, Y=318, Text=`"Update Type"`, Size=11, FontWeight=Bold, Color=`RGBA(65,83,133,1)`

#### Status Dropdown — `ddStatus`

| Property | Value |
|----------|-------|
| X | `920` |
| Y | `405` |
| Width | `200` |
| Height | `40` |
| Items | `["New", "Active", "Monitoring", "Escalated", "Contained", "Closed"]` |
| Default | `varSelectedIncident.Status.Value` |
| BorderColor | `RGBA(200,200,200,1)` |
| ChevronBackground | `RGBA(208,74,2,1)` |

Label above (`lblStatusLabel`): X=920, Y=388, Text=`"Status"`, Size=11, FontWeight=Bold, Color=`RGBA(65,83,133,1)`

#### Severity Dropdown — `ddSeverity`

| Property | Value |
|----------|-------|
| X | `1130` |
| Y | `405` |
| Width | `196` |
| Height | `40` |
| Items | `["Critical", "High", "Moderate", "Low", "Informational"]` |
| Default | `varSelectedIncident.Severity.Value` |
| BorderColor | `RGBA(200,200,200,1)` |
| ChevronBackground | `RGBA(208,74,2,1)` |

Label above (`lblSeverityLabel`): X=1130, Y=388, Text=`"Severity"`, Size=11, FontWeight=Bold, Color=`RGBA(65,83,133,1)`

#### Escalation Threshold Dropdown — `ddEscalationThreshold`

Visible only when UpdateType = "Escalation":

| Property | Value |
|----------|-------|
| X | `920` |
| Y | `475` |
| Width | `406` |
| Height | `40` |
| Items | `["Not Escalated", "Watch", "Escalate", "Executive Brief"]` |
| Default | `varSelectedIncident.EscalationThreshold.Value` |
| Visible | `ddUpdateType.Selected.Value = "Escalation"` |
| BorderColor | `RGBA(200,200,200,1)` |
| ChevronBackground | `RGBA(224,48,30,1)` |

Label above (`lblEscLabel`): X=920, Y=458, Text=`"Escalation Level"`, Size=11, FontWeight=Bold, Color=`RGBA(224,48,30,1)`, Visible=same condition

#### Notes Field — `txtNotes`

| Property | Value |
|----------|-------|
| X | `920` |
| Y | `535` |
| Width | `406` |
| Height | `150` |
| HintText | `"Enter update notes (required)..."` |
| Mode | `TextMode.MultiLine` |
| BorderColor | `RGBA(200,200,200,1)` |
| FocusedBorderColor | `RGBA(208,74,2,1)` |

Label above (`lblNotesLabel`): X=920, Y=518, Text=`"Notes *"`, Size=11, FontWeight=Bold, Color=`RGBA(65,83,133,1)`

#### Save Update Button — `btnSaveUpdate`

| Property | Value |
|----------|-------|
| X | `920` |
| Y | `700` |
| Width | `200` |
| Height | `42` |
| Text | `"Save Update"` |
| Size | `14` |
| FontWeight | `FontWeight.Semibold` |
| Color | `Color.White` |
| Fill | `RGBA(208,74,2,1)` |
| HoverFill | `RGBA(195,76,47,1)` |
| DisabledFill | `RGBA(200,200,200,1)` |
| DisplayMode | `If(IsBlank(txtNotes.Text), DisplayMode.Disabled, DisplayMode.Edit)` |
| RadiusTopLeft | `6` |
| RadiusTopRight | `6` |
| RadiusBottomLeft | `6` |
| RadiusBottomRight | `6` |

**btnSaveUpdate.OnSelect:**
```
// 1. Write history entry
Patch(
    EIT_IncidentHistory,
    Defaults(EIT_IncidentHistory),
    {
        Title: varSelectedIncident.ItemID,
        ParentItemID: varSelectedIncident.ItemID,
        UpdateDate: Now(),
        StatusAtUpdate: {Value: ddStatus.Selected.Value},
        SeverityAtUpdate: {Value: ddSeverity.Selected.Value},
        UpdateType: {Value: ddUpdateType.Selected.Value},
        Notes: txtNotes.Text,
        UpdatedBy: User().FullName,
        PrivilegedAndConfidential: varSelectedIncident.PrivilegedAndConfidential
    }
);
// 2. Update incident record
Patch(
    EIT_Incidents,
    LookUp(EIT_Incidents, ItemID = varSelectedIncident.ItemID),
    {
        LastUpdated: Now(),
        Status: {Value: ddStatus.Selected.Value},
        Severity: {Value: ddSeverity.Selected.Value},
        EscalationThreshold: {Value: If(
            ddUpdateType.Selected.Value = "Escalation",
            ddEscalationThreshold.Selected.Value,
            varSelectedIncident.EscalationThreshold.Value
        )},
        DaysSinceUpdate: 0,
        StalenessFlag: {Value: "Current"},
        DateContained: If(
            ddStatus.Selected.Value = "Contained",
            Now(),
            varSelectedIncident.DateContained
        )
    }
);
// 3. Trigger escalation flow if applicable
If(
    ddUpdateType.Selected.Value = "Escalation" &&
    ddEscalationThreshold.Selected.Value <> "Not Escalated" &&
    ddEscalationThreshold.Selected.Value <> "Watch",
    EITDomainEscalationNotifier.Run(
        varSelectedIncident.ID,
        ddEscalationThreshold.Selected.Value
    )
);
// 4. Refresh UI
Set(
    varSelectedIncident,
    LookUp(EIT_Incidents, ItemID = varSelectedIncident.ItemID)
);
Reset(txtNotes);
Notify("Update saved successfully", NotificationType.Success);
Refresh(EIT_IncidentHistory)
```

### 3.6 IssueDetailScreen.OnVisible

```
Refresh(EIT_IncidentHistory);
Reset(txtNotes)
```

---

## 4. DomainOverviewScreen

This screen shows active incident counts per enterprise domain in a single row of 5 cards.

### 4.1 Header Bar

Insert **Rectangle** `rectDomainHeader` — same navy style (X=0, Y=0, W=1366, H=55, Fill navy).

**Title label** `lblDomainTitle`: X=20, Y=10, W=600, H=35, Text=`"Active Incidents by Domain"`, Size=20, Bold, White.

**Back button** `btnBackFromDomain`: X=1180, Y=12, W=160, H=32, Text=`"Back to Dashboard"`, OnSelect=`Navigate(DashboardScreen)`, transparent white style.

### 4.2 Domain Cards Layout

5 cards in a horizontal row:

| Domain | Card Rect | X | Count Label | Name Label |
|--------|-----------|---|-------------|------------|
| National Security | `rectDomainNS` | 20 | `lblNSCount` | `lblNSName` |
| NIS | `rectDomainNIS` | 284 | `lblNISCount` | `lblNISName` |
| Privacy & Technology | `rectDomainPT` | 548 | `lblPTCount` | `lblPTName` |
| Human Capital | `rectDomainHC` | 812 | `lblHCCount` | `lblHCName` |
| Physical Security | `rectDomainPS` | 1076 | `lblPSCount` | `lblPSName` |

### 4.3 Card Template (repeat for all 5 domains)

**Background Rectangle:**

| Property | Value |
|----------|-------|
| Y | `80` |
| Width | `252` |
| Height | `150` |
| Fill | `RGBA(245,245,245,1)` |
| BorderColor | `RGBA(225,225,225,1)` |
| BorderThickness | `1` |
| RadiusTopLeft | `8` |
| RadiusTopRight | `8` |
| RadiusBottomLeft | `8` |
| RadiusBottomRight | `8` |

**Domain Color Accent (top stripe):**

| Property | Value |
|----------|-------|
| Y | `80` |
| Width | `252` |
| Height | `6` |
| RadiusTopLeft | `8` |
| RadiusTopRight | `8` |
| Fill | *(per domain — see below)* |

Domain accent colors:
- National Security: `RGBA(119,40,32,1)` (dark red)
- NIS: `RGBA(224,48,30,1)` (red)
- Privacy & Technology: `RGBA(65,83,133,1)` (blue)
- Human Capital: `RGBA(228,92,43,1)` (orange)
- Physical Security: `RGBA(50,120,70,1)` (green)

**Count Label (e.g., `lblNSCount`):**

| Property | Value |
|----------|-------|
| Y | `96` |
| Width | `242` |
| Height | `55` |
| Text | `CountRows(Filter(EIT_Incidents, Domain.Value = "National Security", Status.Value <> "Closed", Status.Value <> "Contained"))` |
| Size | `36` |
| FontWeight | `FontWeight.Bold` |
| Color | `RGBA(45,45,45,1)` |
| Align | `Align.Center` |

> Replace `"National Security"` with each domain's exact value for each card.

**Domain Name Label (e.g., `lblNSName`):**

| Property | Value |
|----------|-------|
| Y | `152` |
| Width | `242` |
| Height | `30` |
| Text | `"National Security"` |
| Size | `13` |
| Color | `RGBA(65,83,133,1)` |
| FontWeight | `FontWeight.Semibold` |
| Align | `Align.Center` |

**Critical Count Sub-label (e.g., `lblNSCrit`):**

| Property | Value |
|----------|-------|
| Y | `186` |
| Width | `242` |
| Height | `22` |
| Text | `CountRows(Filter(EIT_Incidents, Domain.Value = "National Security", Severity.Value = "Critical", Status.Value <> "Closed", Status.Value <> "Contained")) & " Critical"` |
| Size | `11` |
| Color | `RGBA(119,40,32,1)` |
| Align | `Align.Center` |

**Card OnSelect** (navigate to Tracker filtered to that domain):
```
// e.g., for NS card:
UpdateContext({varDomainFilter: "NS"});
Navigate(TrackerScreen, ScreenTransition.None)
```

### 4.4 Summary Row

**Total Active — `lblDomainTotal`:**

| Property | Value |
|----------|-------|
| X | `20` |
| Y | `260` |
| Width | `500` |
| Height | `30` |
| Text | `"Total Active: " & CountRows(Filter(EIT_Incidents, Status.Value <> "Closed", Status.Value <> "Contained")) & " incidents across all domains"` |
| Size | `14` |
| FontWeight | `FontWeight.Semibold` |
| Color | `RGBA(45,45,45,1)` |

**P&C Active — `lblDomainPAC`:**

| Property | Value |
|----------|-------|
| X | `20` |
| Y | `295` |
| Width | `500` |
| Height | `25` |
| Text | `CountRows(Filter(EIT_Incidents, PrivilegedAndConfidential = true, Status.Value <> "Closed")) & " Privileged & Confidential incidents active"` |
| Size | `12` |
| Color | `RGBA(119,40,32,1)` |

### 4.5 DomainOverviewScreen.OnVisible

```
Refresh(EIT_Incidents)
```

---

## 5. KanbanScreen

Visual workflow board with 5 status swim-lanes. Closed incidents excluded from all columns.

### 5.1 Header Bar

Insert **Rectangle** `rectKanbanHeader` — navy (X=0, Y=0, W=1366, H=55, Fill=`RGBA(65,83,133,1)`).

**Title**: `lblKanbanTitle` — X=20, Y=10, W=400, H=35, Text=`"Incident Kanban Board"`, Size=20, Bold, White.

**Back button** `btnKanbanBack`: X=1180, Y=12, W=160, H=32, Text=`"Back to Dashboard"`, OnSelect=`Navigate(DashboardScreen)`, transparent white style.

### 5.2 Column Headers

Five column header labels:

| Control | X | Width | Text | Fill |
|---------|---|-------|------|------|
| `lblKanbanNewHdr` | 10 | 258 | `"New (" & CountRows(Filter(EIT_Incidents, Status.Value = "New")) & ")"` | `RGBA(65,83,133,1)` |
| `lblKanbanActiveHdr` | 275 | 258 | `"Active (" & CountRows(Filter(EIT_Incidents, Status.Value = "Active")) & ")"` | `RGBA(208,74,2,1)` |
| `lblKanbanEscHdr` | 540 | 258 | `"Escalated (" & CountRows(Filter(EIT_Incidents, Status.Value = "Escalated")) & ")"` | `RGBA(224,48,30,1)` |
| `lblKanbanMonHdr` | 805 | 258 | `"Monitoring (" & CountRows(Filter(EIT_Incidents, Status.Value = "Monitoring")) & ")"` | `RGBA(228,92,43,1)` |
| `lblKanbanConHdr` | 1070 | 258 | `"Contained (" & CountRows(Filter(EIT_Incidents, Status.Value = "Contained")) & ")"` | `RGBA(50,140,80,1)` |

Shared header properties: Y=65, Height=35, Size=13, FontWeight=Bold, Color=White, Align=Center, RadiusTopLeft=6, RadiusTopRight=6.

### 5.3 Status Galleries

Five vertical galleries, one per status:

| Gallery | X | Status Filter |
|---------|---|---------------|
| `galKanbanNew` | 10 | `Status.Value = "New"` |
| `galKanbanActive` | 275 | `Status.Value = "Active"` |
| `galKanbanEscalated` | 540 | `Status.Value = "Escalated"` |
| `galKanbanMonitoring` | 805 | `Status.Value = "Monitoring"` |
| `galKanbanContained` | 1070 | `Status.Value = "Contained"` |

Shared gallery properties:
- Y=100, Width=258, Height=640
- TemplateSize=130, TemplatePadding=5
- `Items: SortByColumns(Filter(EIT_Incidents, [STATUS FILTER]), "DaysSinceUpdate", SortOrder.Descending)`
- `Fill: RGBA(240,240,240,1)`, `BorderColor: RGBA(225,225,225,1)`, `BorderThickness: 1`

### 5.4 Card Template (inside each gallery)

**Card Background — `rectKanbanCard`:**
X=5, Y=2, W=244, H=123, Fill=White, BorderColor=`RGBA(225,225,225,1)`, BorderThickness=1, Radius=6

**Staleness Stripe — `rectKanbanStripe`:**
X=5, Y=2, W=4, H=123, Radius top-left/bottom-left=6
```
Fill: If(ThisItem.DaysSinceUpdate <= 7, RGBA(65,83,133,1),
         If(ThisItem.DaysSinceUpdate <= 14, RGBA(228,92,43,1),
            RGBA(224,48,30,1)))
```

**P&C Stripe — `rectKanbanPAC`:**
X=239, Y=2, W=4, H=123, Radius top-right/bottom-right=6
```
Fill: If(ThisItem.PrivilegedAndConfidential, RGBA(119,40,32,1), Color.Transparent)
```

**Item ID — `lblKanbanID`:** X=14, Y=6, W=70, H=20, Text=`ThisItem.ItemID`, Size=10, Bold, Color=`RGBA(65,83,133,1)`

**Severity Badge — `lblKanbanSeverity`:** X=185, Y=6, W=60, H=20, Text=`ThisItem.Severity.Value`, Size=8, Bold, Center, Color=White, Fill=Severity switch formula, Radius=4

**Title — `lblKanbanTitle`:** X=14, Y=28, W=230, H=28, Text=`If(Len(ThisItem.Title) > 38, Left(ThisItem.Title, 38) & "...", ThisItem.Title)`, Size=11, Bold, Color=`RGBA(45,45,45,1)`, Tooltip=`ThisItem.Title`

**Domain — `lblKanbanDomain`:** X=14, Y=58, W=180, H=18, Text=`ThisItem.Domain.Value`, Size=9, Color=`RGBA(65,83,133,1)`, Italic=true

**Owner — `lblKanbanOwner`:** X=14, Y=78, W=150, H=18, Text=`ThisItem.Owner`, Size=10, Color=`RGBA(80,80,80,1)`

**Days — `lblKanbanDays`:** X=175, Y=78, W=65, H=18, Text=`Text(ThisItem.DaysSinceUpdate) & "d"`, Size=10, Bold, Align=Right
```
Color: If(ThisItem.DaysSinceUpdate <= 7, RGBA(65,83,133,1),
          If(ThisItem.DaysSinceUpdate <= 14, RGBA(228,92,43,1), RGBA(224,48,30,1)))
```

**Escalation Label — `lblKanbanEsc`:** X=14, Y=100, W=200, H=18, Text=`ThisItem.EscalationThreshold.Value`, Size=9, Color=`RGBA(228,92,43,1)`, Visible=`ThisItem.EscalationThreshold.Value <> "Not Escalated"`

### 5.5 Card OnSelect (all 5 galleries)

```
Navigate(IssueDetailScreen, ScreenTransition.None, {varSelectedIncident: ThisItem})
```

### 5.6 KanbanScreen.OnVisible

```
Refresh(EIT_Incidents)
```

---

## 6. ExecutiveReportScreen

Read-only leadership summary. No edit controls.

### 6.1 Header Bar

Insert **Rectangle** `rectExecHeader` — same navy style (X=0, Y=0, W=1366, H=60).

**Title** `lblExecTitle`: X=20, Y=10, W=600, H=40, Text=`"Executive Incident Summary"`, Size=20, Bold, White.

**Back button**: X=1200, Y=12, W=140, H=36, Text=`"< Dashboard"`, OnSelect=`Navigate(DashboardScreen)`, transparent white style.

**Report date** `lblExecDate`: X=800, Y=20, W=350, H=22, Text=`"Report as of: " & Text(Now(), "mmmm dd, yyyy")`, Size=12, Color=White, Align=Right.

### 6.2 Domain Summary Table

Insert a **Gallery > Blank vertical**. Rename `galExecDomainSummary`.

| Property | Value |
|----------|-------|
| X | `20` |
| Y | `80` |
| Width | `1326` |
| Height | `300` |
| TemplateSize | `50` |
| TemplatePadding | `0` |
| ShowScrollbar | `false` |

**galExecDomainSummary.Items:**
```
AddColumns(
    ["National Security", "NIS", "Privacy & Technology", "Human Capital", "Physical Security"],
    "DomainName", Value,
    "OpenCount", CountRows(Filter(EIT_Incidents, Domain.Value = Value, Status.Value <> "Closed", Status.Value <> "Contained")),
    "CriticalCount", CountRows(Filter(EIT_Incidents, Domain.Value = Value, Severity.Value = "Critical", Status.Value <> "Closed", Status.Value <> "Contained")),
    "EscalatedCount", CountRows(Filter(EIT_Incidents, Domain.Value = Value, EscalationThreshold.Value <> "Not Escalated", Status.Value <> "Closed", Status.Value <> "Contained")),
    "PACCount", CountRows(Filter(EIT_Incidents, Domain.Value = Value, PrivilegedAndConfidential = true, Status.Value <> "Closed")),
    "StaleCount", CountRows(Filter(EIT_Incidents, Domain.Value = Value, StalenessFlag.Value = "Stale", Status.Value <> "Closed", Status.Value <> "Contained"))
)
```

> **Note:** Power Apps AddColumns with Filter may hit delegation limits on large datasets (>500 rows). For EIT at Phase 1 scale (expected <200 rows per domain) this is acceptable. Phase 2 should consider a scheduled Power Automate flow that pre-calculates these counts into a summary SharePoint list.

#### Column Headers for Executive Table

Insert a row of header labels above the gallery (Y=65):

| Label | X | Width | Text |
|-------|---|-------|------|
| `lblExecColDomain` | 20 | 300 | `"Domain"` |
| `lblExecColOpen` | 325 | 100 | `"Open"` |
| `lblExecColCrit` | 430 | 100 | `"Critical"` |
| `lblExecColEsc` | 535 | 120 | `"Escalated"` |
| `lblExecColPAC` | 660 | 100 | `"P&C Active"` |
| `lblExecColStale` | 765 | 100 | `"Stale"` |

All: Height=28, Size=11, FontWeight=Bold, Color=`RGBA(65,83,133,1)`, Fill=`RGBA(235,235,235,1)`

#### Controls Inside the Gallery Template

**Domain Name — `lblExecDomain`:**
X=20, Y=10, W=300, H=30, Text=`ThisItem.DomainName`, Size=14, FontWeight=Bold, Color=`RGBA(45,45,45,1)`

**Open Count — `lblExecOpen`:**
X=325, Y=10, W=100, H=30, Text=`Text(ThisItem.OpenCount)`, Size=20, Bold, Align=Center, Color=`RGBA(65,83,133,1)`

**Critical Count — `lblExecCrit`:**
X=430, Y=10, W=100, H=30, Text=`Text(ThisItem.CriticalCount)`, Size=20, Bold, Align=Center
```
Color: If(ThisItem.CriticalCount > 0, RGBA(119,40,32,1), RGBA(180,180,180,1))
```

**Escalated Count — `lblExecEsc`:**
X=535, Y=10, W=120, H=30, Text=`Text(ThisItem.EscalatedCount)`, Size=20, Bold, Align=Center
```
Color: If(ThisItem.EscalatedCount > 0, RGBA(224,48,30,1), RGBA(180,180,180,1))
```

**P&C Count — `lblExecPAC`:**
X=660, Y=10, W=100, H=30, Text=`Text(ThisItem.PACCount)`, Size=20, Bold, Align=Center
```
Color: If(ThisItem.PACCount > 0, RGBA(119,40,32,1), RGBA(180,180,180,1))
```

**Stale Count — `lblExecStale`:**
X=765, Y=10, W=100, H=30, Text=`Text(ThisItem.StaleCount)`, Size=20, Bold, Align=Center
```
Color: If(ThisItem.StaleCount > 0, RGBA(224,48,30,1), RGBA(180,180,180,1))
```

**Row Separator:**
X=20, Y=48, W=1286, H=1, Fill=`RGBA(225,225,225,1)`

### 6.3 Enterprise Totals Row

Below the domain summary gallery (Y=400):

**`lblExecTotalHeader`:** X=20, Y=395, W=400, H=25, Text=`"ENTERPRISE TOTALS"`, Size=11, Bold, Color=`RGBA(130,130,130,1)`

| Metric Label | X | Value Formula |
|-------------|---|---------------|
| `lblExecTotalOpen` | 20 | `CountRows(Filter(EIT_Incidents, Status.Value <> "Closed", Status.Value <> "Contained")) & " Total Open"` |
| `lblExecTotalCrit` | 250 | `CountRows(Filter(EIT_Incidents, Severity.Value = "Critical", Status.Value <> "Closed")) & " Critical"` |
| `lblExecTotalEsc` | 480 | `CountRows(Filter(EIT_Incidents, EscalationThreshold.Value <> "Not Escalated", Status.Value <> "Closed")) & " Escalated"` |
| `lblExecTotalPAC` | 710 | `CountRows(Filter(EIT_Incidents, PrivilegedAndConfidential = true, Status.Value <> "Closed")) & " P&C"` |
| `lblExecTotalStale` | 940 | `CountRows(Filter(EIT_Incidents, StalenessFlag.Value = "Stale", Status.Value <> "Closed")) & " Stale"` |

All: Y=420, Height=40, Size=18, FontWeight=Bold, Color=`RGBA(65,83,133,1)`

### 6.4 P&C Restricted Notice

Insert a label at Y=490:

**`lblExecPACNote`:**

| Property | Value |
|----------|-------|
| X | `20` |
| Y | `490` |
| Width | `1000` |
| Height | `40` |
| Text | `"Note: P&C count reflects incidents visible to you. Access to full P&C incident details requires item-level SharePoint permissions. Contact the EIT administrator."` |
| Size | `11` |
| Color | `RGBA(130,130,130,1)` |
| Italic | `true` |

### 6.5 ExecutiveReportScreen.OnVisible

```
Refresh(EIT_Incidents)
```

---

## 7. App-Level Control Naming Reference

| Screen | Control Type | Name Pattern |
|--------|-------------|--------------|
| All screens | Header rectangle | `rectXxxHeader` |
| All screens | Header title label | `lblXxxTitle` |
| All screens | Back button | `btnXxxBack` or `btnBackFromXxx` |
| DashboardScreen | KPI rectangles | `rectOpen`, `rectCrit`, `rectStale`, `rectEsc`, `rectPAC` |
| DashboardScreen | KPI count labels | `lblOpenCount`, `lblCritCount`, etc. |
| DashboardScreen | Intake gallery | `galIntakeQueue` |
| DashboardScreen | Intake panel | `rectIntakePanel`, `btnIntakePanelClose` |
| DashboardScreen | Submit overlay | `rectOverlayBg`, `rectOverlayCard`, `btnSubmitNewIncident` |
| TrackerScreen | Filter buttons | `btnFilterAll`, `btnFilterNS`, `btnFilterNIS`, `btnFilterPT`, `btnFilterHC`, `btnFilterPS`, `btnFilterCrit` |
| TrackerScreen | Incident gallery | `galIncidentTable` |
| IssueDetailScreen | P&C banner | `rectPACBanner`, `lblPACBanner` |
| IssueDetailScreen | Incident header card | `rectIncidentCard` |
| IssueDetailScreen | History gallery | `galUpdateHistory` |
| IssueDetailScreen | Update form | `ddUpdateType`, `ddStatus`, `ddSeverity`, `ddEscalationThreshold`, `txtNotes`, `btnSaveUpdate` |
| DomainOverviewScreen | Domain cards | `rectDomainNS`, `rectDomainNIS`, `rectDomainPT`, `rectDomainHC`, `rectDomainPS` |
| KanbanScreen | Swim-lane galleries | `galKanbanNew`, `galKanbanActive`, `galKanbanEscalated`, `galKanbanMonitoring`, `galKanbanContained` |
| ExecutiveReportScreen | Domain summary gallery | `galExecDomainSummary` |

---

## 8. Context Variables Reference

| Variable | Set In | Purpose |
|----------|--------|---------|
| `varUserHasPACAccess` | App.OnStart | Boolean — true if user is in P&C access group |
| `varCurrentUser` | App.OnStart | `User().FullName` |
| `varCurrentUserEmail` | App.OnStart | `User().Email` |
| `varSelectedIncident` | Navigate to IssueDetailScreen, or galIncidentTable.OnSelect | The current incident record |
| `varDomainFilter` | TrackerScreen.OnVisible, filter buttons | `"All"`, `"NS"`, `"NIS"`, `"PT"`, `"HC"`, `"PS"`, `"Critical"`, `"Stale"` |
| `varSearch` | txtSearch.OnChange | Search string |
| `varShowContained` | togShowContained.OnChange | Boolean — show Contained incidents in tracker |
| `varShowNewIncident` | btnNewIncident.OnSelect | Boolean — show submit overlay |
| `showIntakePanel` | galIntakeQueue.OnSelect | Boolean — show intake review panel |
| `selectedIntake` | galIntakeQueue.OnSelect | The intake queue item being reviewed |
| `varPromoteResult` | btnIntakePromote.OnSelect | Flow response containing `newitemid` |

---

## 9. Testing Checklist

### Dashboard
- [ ] 5 KPI cards show correct counts
- [ ] Intake queue shows Pending items only, sorted by Priority then date
- [ ] Clicking intake item opens review panel with correct details
- [ ] P&C items show P&C badge in gallery and P&C warning in panel
- [ ] Promote button runs flow and shows new EIT-NNN in notification
- [ ] Dismiss sets TriageStatus=Dismissed, removes item from queue
- [ ] "+ New Incident" opens overlay; all required fields validated before submit
- [ ] Submit creates item in EIT_IntakeQueue and closes overlay
- [ ] Navigation buttons reach all screens

### Tracker Screen
- [ ] Domain filter chips filter correctly per domain
- [ ] Critical filter shows only Severity=Critical items
- [ ] Search filters by Title, Owner, ItemID, Domain
- [ ] Severity badge displays correct 5-tier colors
- [ ] Status badge includes Contained with green color
- [ ] Escalation badge shows correct 4-tier colors
- [ ] Red left-stripe appears on P&C items only
- [ ] P&C items hidden from users where varUserHasPACAccess=false
- [ ] Clicking row navigates to IssueDetailScreen with correct item

### Issue Detail Screen
- [ ] P&C amber warning banner visible on P&C incidents, hidden on others
- [ ] Header card shows all fields: ItemID, Title, Owner, Domain, IncidentType, Severity, Status, Escalation, Dates, Days, Confidentiality
- [ ] Severity badge 5-tier colors correct
- [ ] Status badge 6-value colors correct including Contained
- [ ] Update history shows items in descending date order
- [ ] History badges show UpdateType, Status, Severity
- [ ] P&C history entries show italic style and P&C badge
- [ ] Save Update disabled when Notes is empty
- [ ] EscalationThreshold dropdown visible only when UpdateType=Escalation
- [ ] Saving Escalation update type triggers EITDomainEscalationNotifier flow
- [ ] Saving Contained status sets DateContained on incident record
- [ ] After save: incident header refreshes with new values, history entry appears

### Domain Overview Screen
- [ ] 5 domain cards show correct active counts
- [ ] Critical sub-counts correct
- [ ] Clicking a domain card navigates to Tracker filtered to that domain

### Kanban Screen
- [ ] 5 swim-lanes visible: New, Active, Escalated, Monitoring, Contained
- [ ] Column headers show correct counts
- [ ] Staleness stripe colors correct (blue/orange/red)
- [ ] P&C right stripe (dark red) visible on P&C cards only
- [ ] Clicking card navigates to IssueDetailScreen

### Executive Report Screen
- [ ] Domain summary table shows all 5 domains
- [ ] Open, Critical, Escalated, P&C, Stale counts accurate
- [ ] Zero counts shown in grey, non-zero in appropriate color
- [ ] Enterprise totals match sum of domain rows
- [ ] P&C restricted notice visible at bottom

---

## Appendix A: Appkit4 Color Quick Reference

| Name | RGBA |
|------|------|
| Primary Blue | `RGBA(65,83,133,1)` |
| Primary Orange | `RGBA(208,74,2,1)` |
| Primary Red | `RGBA(224,48,30,1)` |
| Orange-Red | `RGBA(228,92,43,1)` |
| Dark Charcoal | `RGBA(45,45,45,1)` |
| Navy Header | `RGBA(65,83,133,1)` |
| Screen Background | `RGBA(245,247,250,1)` |
| Card Background | `RGBA(245,245,245,1)` |
| Muted Text | `RGBA(130,130,130,1)` |
| Hover Blue | `RGBA(98,113,154,1)` |
| Critical Dark Red | `RGBA(119,40,32,1)` |
| Contained Green | `RGBA(50,140,80,1)` |

## Appendix B: Delegation Considerations

Power Apps delegation to SharePoint is supported for the following functions in EIT formulas:
- `Filter` with supported column types and operators: `=`, `<>`, `in` (on text fields), `=true`/`=false` (on Yes/No fields)
- `SortByColumns` on single columns
- `CountRows` with a Filter expression

**Non-delegable patterns to watch for:**
- `Or()` with multiple conditions across different column types — may truncate at 500 rows
- `AddColumns()` with `Filter` inside — not delegable; returns at most 500 rows from the base table
- `Len()`, `Left()`, `in` on multi-value fields — not delegable

At EIT Phase 1 scale (expected <200 active incidents), the 500-row delegation limit is not a concern. If incident volume grows toward 500+, add an OData filter in the data source connection or pre-aggregate in Power Automate flows.
