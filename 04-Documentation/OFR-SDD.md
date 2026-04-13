# OFR Issue Tracker — System Design Document (SDD)

**Version:** 1.1
**Date:** February 19, 2025
**Classification:** Internal
**Author:** OFR Technology Team

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [System Overview](#2-system-overview)
3. [Architecture](#3-architecture)
4. [Data Layer — SharePoint Online](#4-data-layer--sharepoint-online)
5. [UI Layer — Power Apps Canvas App](#5-ui-layer--power-apps-canvas-app)
6. [Automation Layer — Power Automate](#6-automation-layer--power-automate)
7. [Security & Access Control](#7-security--access-control)
8. [Integration Points](#8-integration-points)
9. [Non-Functional Requirements](#9-non-functional-requirements)
10. [Deployment & Environments](#10-deployment--environments)
11. [Known Limitations & Future Roadmap](#11-known-limitations--future-roadmap)
12. [Appendices](#12-appendices)

---

## 1. Introduction

### 1.1 Purpose

This System Design Document describes the technical architecture, data model, component design, and integration details of the OFR Issue Tracker — an M365-native application for managing cross-firm risk issues through their full lifecycle.

### 1.2 Scope

The system covers:
- Issue intake and triage workflow
- Active issue tracking with staleness indicators
- Update workflow with full audit trail
- Dashboard KPIs and reporting views
- Automated daily staleness calculation

The following features are explicitly out of scope for this release:
- Bilingual (EN/FR) language support
- CSV/PDF export from within the app
- Email notifications for stale issues
- Role-based access control (beyond SharePoint group membership)

### 1.3 Background

The OFR Issue Tracker replaces a previous React/Vite single-page application (SPA) that required Azure Static Web Apps hosting, custom MSAL authentication code, GitHub CI/CD pipelines, and Azure infrastructure management. The M365-native rebuild eliminates all custom infrastructure while preserving the core issue lifecycle functionality.

### 1.4 Design Principles

| Principle | Implementation |
|-----------|----------------|
| **Zero infrastructure** | No Azure subscriptions, custom domains, SSL certificates, or CI/CD pipelines |
| **Native authentication** | Entra ID SSO via Power Platform — no custom MSAL code |
| **Data stays in M365** | All data stored in SharePoint Online lists within the tenant |
| **Low-code maintainability** | Power Apps canvas app and Power Automate flows — editable by non-developers |
| **Auditable** | Full update history with timestamps, status snapshots, and user attribution |

### 1.5 Licensing Requirements

| Component | License Required |
|-----------|-----------------|
| SharePoint Online | M365 Business Standard (included) |
| Power Apps | Power Apps Developer Plan (free) or Power Apps per-user |
| Power Automate | Power Automate Free (included with M365) |

---

## 2. System Overview

### 2.1 High-Level Description

The OFR Issue Tracker is a three-tier application built entirely within the Microsoft 365 ecosystem:

1. **Data Layer** — SharePoint Online lists store all issue data, update history, and intake queue records
2. **UI Layer** — A Power Apps canvas app provides the user interface with six screens (Dashboard, Tracker, Issue Detail, Submit, Group Allocation, Kanban) and two inline side panels
3. **Automation Layer** — Power Automate cloud flows handle scheduled staleness calculations and triggered intake promotion

### 2.2 User Roles

| Role | Capabilities |
|------|-------------|
| **Issue Contributor** | View dashboard and tracker, add updates to existing issues, submit new intake items |
| **Issue Manager** | All contributor capabilities plus: promote/dismiss intake items, change issue priority/status |
| **Administrator** | All manager capabilities plus: modify SharePoint list schemas, edit Power Apps app, manage Power Automate flows, manage site membership |

> **Note:** In the current implementation, access is controlled at the SharePoint site level. All members of the OFR Issue Tracker site have equivalent read/write access. Role differentiation would require Power Apps role-based logic in a future release.

### 2.3 System Context Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                     Microsoft 365 Tenant                        │
│                                                                 │
│  ┌──────────────┐    ┌───────────────────┐    ┌──────────────┐ │
│  │  Entra ID    │    │   SharePoint      │    │   Power      │ │
│  │  (SSO Auth)  │    │   Online          │    │   Automate   │ │
│  └──────┬───────┘    │                   │    │              │ │
│         │            │  ┌─────────────┐  │    │  ┌────────┐  │ │
│         │            │  │ OFR_Issues  │  │◄───┤  │Staleness│ │ │
│         │            │  └─────────────┘  │    │  │  Flow  │  │ │
│         │            │  ┌─────────────┐  │    │  └────────┘  │ │
│  ┌──────▼───────┐    │  │OFR_Update   │  │    │              │ │
│  │  Power Apps  │◄──▶│  │  History    │  │    │  ┌────────┐  │ │
│  │  Canvas App  │    │  └─────────────┘  │◄───┤  │Intake  │  │ │
│  │              │    │  ┌─────────────┐  │    │  │Promotion│ │ │
│  │  - Dashboard │───▶│  │OFR_Intake   │  │    │  │  Flow  │  │ │
│  │  - Tracker   │    │  │  Queue      │  │    │  └────────┘  │ │
│  │  - Detail    │    │  └─────────────┘  │    │              │ │
│  └──────────────┘    └───────────────────┘    └──────────────┘ │
│         ▲                                                       │
│         │                                                       │
└─────────┼───────────────────────────────────────────────────────┘
          │
    ┌─────┴─────┐
    │   Users   │
    │ (Browser/ │
    │  Teams)   │
    └───────────┘
```

---

## 3. Architecture

### 3.1 Component Summary

| Component | Technology | Instance ID |
|-----------|-----------|-------------|
| SharePoint Site | SharePoint Online | https://[TENANT].sharepoint.com/sites/OFRIssueTracker |
| OFR_Issues List | SharePoint List | GUID: `a70da6a6-1f0a-4fd3-bb4e-cf7847e18a99` |
| OFR_UpdateHistory List | SharePoint List | (on same site) |
| OFR_IntakeQueue List | SharePoint List | (on same site) |
| Power Apps Canvas App | Power Apps | App ID: `0fbbc26c-ad71-476a-bcfc-edc0d7989533` |
| Daily Staleness Calculator | Power Automate Cloud Flow | Flow ID: `aefb8de0-35fe-4d5d-a629-ddd8502ee5aa` |
| Intake Promotion | Power Automate Cloud Flow | Flow ID: `1c631640-113f-4602-805e-1d693582de8c` |

### 3.2 Technology Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| Data | SharePoint Online | Current (M365) |
| UI | Power Apps Canvas App | Current (Modern controls enabled) |
| Automation | Power Automate Cloud Flows | Current (New Designer v3) |
| Authentication | Microsoft Entra ID | SSO via Power Platform |
| Hosting | Power Platform / SharePoint | M365 multi-tenant |

### 3.3 Data Flow Diagram

```
User Action                    Power Apps                 SharePoint              Power Automate
───────────                    ──────────                 ──────────              ──────────────

Submit intake item ──────────▶ Patch() to               OFR_IntakeQueue
                               OFR_IntakeQueue            (new row, Pending)

Accept intake (panel) ───────▶ Patch() to               OFR_Issues
                               OFR_Issues                 (new row, Status=New,
                                                           FunctionalGroup mapped)
                               Patch() to               OFR_IntakeQueue
                               OFR_IntakeQueue             (TriageStatus=Accepted)

Reject intake (panel) ───────▶ Patch() to               OFR_IntakeQueue
                               OFR_IntakeQueue             (TriageStatus=Rejected)

Promote intake ──────────────▶ Call PA flow ─────────────────────────────────────▶ Intake Promotion
                                                                                   │
                                                          OFR_IntakeQueue ◄────────┤ TriageStatus=Promoted
                                                          OFR_Issues ◄─────────────┤ New issue created
                                                          OFR_UpdateHistory ◄──────┤ Audit entry created
                               ◄─────────────────────────────────────────────────── NewItemID returned

Add update ──────────────────▶ Patch() to               OFR_UpdateHistory
                               OFR_UpdateHistory          (new row)
                               Patch() to               OFR_Issues
                               OFR_Issues                 (LastUpdated, Status,
                                                           DaysSinceUpdate=0)

(Daily 6 AM) ───────────────────────────────────────────────────────────────────▶ Staleness Calculator
                                                          OFR_Issues ◄──────────── DaysSinceUpdate
                                                                                   StalenessFlag

View dashboard ──────────────▶ Filter() +               OFR_Issues (read)
                               CountRows()              OFR_IntakeQueue (read)

View tracker ────────────────▶ Filter() +               OFR_Issues (read)
                               Search() +
                               SortByColumns()

View issue detail ───────────▶ LookUp() +               OFR_Issues (read)
                               Filter()                 OFR_UpdateHistory (read)

View group allocation ───────▶ Filter() +               OFR_Issues (read)
                               CountRows()              (grouped by FunctionalGroup)

View kanban board ───────────▶ Filter() ×4              OFR_Issues (read)
                               (New, Active,            (one gallery per status)
                                Escalated, Monitoring)
```

---

## 4. Data Layer — SharePoint Online

### 4.1 Site Configuration

| Property | Value |
|----------|-------|
| Site Name | OFR Issue Tracker |
| Site URL | https://[TENANT].sharepoint.com/sites/OFRIssueTracker |
| Site Type | Team site (Private group) |
| Storage | Default SharePoint site quota |

### 4.2 List: OFR_Issues

**Purpose:** Primary issue tracking list. Each row represents one risk issue throughout its lifecycle.

| # | Column Name | Internal Name | Type | Required | Details |
|---|-------------|---------------|------|----------|---------|
| 1 | Title | Title | Single line of text | Yes | Issue topic/description. Max 255 chars. |
| 2 | ItemID | ItemID | Single line of text | No | Unique identifier in OFR-NNN format. Auto-generated during intake promotion via `concat('OFR-', string(ID))`. |
| 3 | Owner | Owner | Single line of text | No | Display name of person responsible. Free text (not a People picker) for simplicity. |
| 4 | Priority | Priority | Choice | No | **Values:** High, Medium, Low. Default: none. |
| 5 | Status | Status | Choice | No | **Values:** New, Active, Monitoring, Escalated, Closed. Default: none. |
| 6 | DateRaised | DateRaised | Date and time | No | When the issue was first identified. Set to `utcNow()` during promotion. Format: ISO 8601. |
| 7 | LastUpdated | LastUpdated | Date and time | No | Timestamp of the most recent update. Reset to `Now()` each time an update is added via Power Apps. |
| 8 | NextAction | NextAction | Multiple lines of text | No | Description of the next steps. Plain text. Populated from intake Description during promotion. |
| 9 | DaysSinceUpdate | DaysSinceUpdate | Number | No | Integer. Number of days since LastUpdated. Calculated daily by Power Automate staleness flow. Reset to 0 when an update is added. |
| 10 | StalenessFlag | StalenessFlag | Choice | No | **Values:** Current, Aging, Stale. Set by Power Automate based on DaysSinceUpdate thresholds: 0–7=Current, 8–14=Aging, 15+=Stale. |
| 11 | FunctionalGroup | FunctionalGroup | Choice | No | **Values:** Risk Management Office, Engagement Risk, Client Risk and KYC, Technology Risk & AI Trust, National Security, OGC General Counsel, OGC Privacy, OGC Contracts, Internal Audit, Independence. Designates which functional group owns this issue. |

**Indexes:** Default SharePoint list indexing. No custom indexes required at current data volumes.

**Relationships:** OFR_Issues.ItemID ← OFR_UpdateHistory.ParentItemID (logical, not enforced by SharePoint).

### 4.3 List: OFR_UpdateHistory

**Purpose:** Immutable audit trail of all updates to issues. Each row represents one update event.

| # | Column Name | Internal Name | Type | Required | Details |
|---|-------------|---------------|------|----------|---------|
| 1 | Title | Title | Single line of text | Yes | Set to ParentItemID for linking. Enables default SharePoint views to show the parent issue. |
| 2 | ParentItemID | ParentItemID | Single line of text | No | Links to OFR_Issues.ItemID. Used to filter history for a specific issue. |
| 3 | UpdateDate | UpdateDate | Date and time | No | When the update was recorded. Set to `Now()` in Power Apps or `utcNow()` in Power Automate. |
| 4 | StatusAtUpdate | StatusAtUpdate | Choice | No | **Values:** New, Active, Monitoring, Escalated, Closed. Captures the issue's status at the time of the update (snapshot). |
| 5 | Notes | Notes | Multiple lines of text | No | Free text commentary describing the update. Plain text format. |
| 6 | UpdatedBy | UpdatedBy | Single line of text | No | Display name of the person who added the update. |

**Design Decision:** This list is append-only by convention. Updates should never be edited or deleted to preserve the integrity of the audit trail.

### 4.4 List: OFR_IntakeQueue

**Purpose:** Triage queue for newly submitted issues before they enter the active tracker.

| # | Column Name | Internal Name | Type | Required | Details |
|---|-------------|---------------|------|----------|---------|
| 1 | Title | Title | Single line of text | Yes | Issue title as submitted by the user. |
| 2 | Owner | Owner | Single line of text | No | Person who raised the issue. |
| 3 | Priority | Priority | Choice | No | **Values:** High, Medium, Low. Initial assessment by the submitter. |
| 4 | Description | Description | Multiple lines of text | No | Detailed issue description. Mapped to NextAction during promotion. |
| 5 | DateSubmitted | DateSubmitted | Date and time | No | When the intake item was created. |
| 6 | TriageStatus | TriageStatus | Choice | No | **Values:** Pending, Promoted, Dismissed, Accepted, Rejected. Default: Pending. Changed to "Promoted" by intake promotion flow, "Dismissed" by Dashboard dismiss button, "Accepted" by Intake Review panel accept button, or "Rejected" by Intake Review panel reject button. |
| 7 | FunctionalGroup | FunctionalGroup | Choice | No | **Values:** Risk Management Office, Engagement Risk, Client Risk and KYC, Technology Risk & AI Trust, National Security, OGC General Counsel, OGC Privacy, OGC Contracts, Internal Audit, Independence. Flows through to OFR_Issues on promotion/acceptance. |

**Retention:** Promoted and dismissed items remain in the list for audit purposes. Only items with TriageStatus = "Pending" are displayed in the Power Apps intake gallery.

### 4.5 Data Model — Entity Relationship Diagram

```
┌────────────────────────┐         ┌────────────────────────┐
│     OFR_IntakeQueue    │         │      OFR_Issues        │
├────────────────────────┤         ├────────────────────────┤
│ ID (auto)              │         │ ID (auto)              │
│ Title                  │ ──────▶ │ Title                  │
│ Owner                  │ promote │ ItemID (OFR-NNN)       │
│ Priority               │         │ Owner                  │
│ Description            │         │ Priority               │
│ DateSubmitted          │         │ Status                 │
│ TriageStatus           │         │ DateRaised             │
│ FunctionalGroup        │         │ FunctionalGroup        │
└────────────────────────┘         │ LastUpdated            │
                                   │ NextAction             │
                                   │ DaysSinceUpdate        │
                                   │ StalenessFlag          │
                                   └───────────┬────────────┘
                                               │
                                               │ 1:N (via ItemID)
                                               │
                                   ┌───────────▼────────────┐
                                   │  OFR_UpdateHistory     │
                                   ├────────────────────────┤
                                   │ ID (auto)              │
                                   │ Title (=ParentItemID)  │
                                   │ ParentItemID           │
                                   │ UpdateDate             │
                                   │ StatusAtUpdate         │
                                   │ Notes                  │
                                   │ UpdatedBy              │
                                   └────────────────────────┘
```

### 4.6 Data Volume Estimates

| Metric | Estimate | Rationale |
|--------|----------|-----------|
| Active issues at any time | 10–50 | Based on current operational load |
| New issues per month | 5–15 | Based on historical intake rates |
| Updates per issue | 3–10 over lifecycle | Weekly updates for ~1–3 month issue lifespan |
| UpdateHistory rows per year | 200–1,500 | Well within SharePoint list limits |
| IntakeQueue rows per year | 60–180 | Most promoted quickly, small accumulation |

**SharePoint List Limits:** 30 million items per list. No concern at projected volumes.

---

## 5. UI Layer — Power Apps Canvas App

### 5.1 App Configuration

| Property | Value |
|----------|-------|
| App Type | Canvas App |
| App ID | `0fbbc26c-ad71-476a-bcfc-edc0d7989533` |
| Screen Count | 6 (DashboardScreen, TrackerScreen, IssueDetailScreen, SubmitScreen, GroupAllocationScreen, KanbanScreen) |
| Data Sources | OFR_Issues, OFR_UpdateHistory, OFR_IntakeQueue (SharePoint connectors) |
| Modern Controls | Enabled |
| Theme | Appkit4 (Blue, Orange, Red, Black) — Primary Blue (#415385), Primary Orange (#D04A02), Primary Red (#E0301E), Neutral Black (#2D2D2D) |

### 5.2 Screen: DashboardScreen

**Purpose:** Landing page with KPI overview and intake triage.

| Component | Type | Data Source | Key Logic |
|-----------|------|-------------|-----------|
| KPI - Open Items | Label | OFR_Issues | `CountRows(Filter(OFR_Issues, Status <> "Closed"))` |
| KPI - Stale | Label | OFR_Issues | `CountRows(Filter(OFR_Issues, StalenessFlag = "Stale", Status <> "Closed"))` |
| KPI - High Priority | Label | OFR_Issues | `CountRows(Filter(OFR_Issues, Priority = "High", Status <> "Closed"))` |
| KPI - Medium | Label | OFR_Issues | `CountRows(Filter(OFR_Issues, Priority = "Medium", Status <> "Closed"))` |
| KPI - Low | Label | OFR_Issues | `CountRows(Filter(OFR_Issues, Priority = "Low", Status <> "Closed"))` |
| Intake Gallery | Gallery | OFR_IntakeQueue | `Filter(OFR_IntakeQueue, TriageStatus = "Pending")` sorted by Priority, DateSubmitted |
| Promote Button | Button | — | Calls OFR Intake Promotion flow via `OFRIntakePromotion.Run(ThisItem.ID)` |
| Dismiss Button | Button | OFR_IntakeQueue | `Patch(OFR_IntakeQueue, ThisItem, {TriageStatus: "Dismissed"})` |
| Intake Gallery OnSelect | — | — | `UpdateContext({showIntakePanel: true, selectedIntake: ThisItem})` |
| **Intake Review Panel** | | | |
| Panel Background | Rectangle | — | Visible: `showIntakePanel`. White panel, right-aligned. |
| Panel Header | Label + Button | — | "Intake Review" heading + "X" close button → `UpdateContext({showIntakePanel: false})` |
| Title Display | Label | OFR_IntakeQueue | `selectedIntake.Title` |
| Description Display | Label | OFR_IntakeQueue | `selectedIntake.Description` |
| Priority Display | Label | OFR_IntakeQueue | `"Priority: " & selectedIntake.Priority.Value` |
| Date Display | Label | OFR_IntakeQueue | `"Submitted: " & Text(selectedIntake.DateSubmitted, "mm/dd/yyyy")` |
| Assign Owner Input | TextInput | — | Free text input for assigning an owner to the issue |
| Accept Button | Button | OFR_Issues, OFR_IntakeQueue | Creates new issue in OFR_Issues via Patch(), marks intake as "Accepted" (see formula below) |
| Reject Button | Button | OFR_IntakeQueue | `Patch(OFR_IntakeQueue, selectedIntake, {TriageStatus: {Value: "Rejected"}})` |
| **Other Controls** | | | |
| + Submit New Issue Button | Button | — | `Navigate(SubmitScreen)` |
| View Tracker Button | Button | — | `Navigate(TrackerScreen)` |
| Group Allocation Button | Button | — | `Navigate(GroupAllocationScreen)` |
| Kanban Board Button | Button | — | `Navigate(KanbanScreen)` |

**Accept Button OnSelect:**
```
Patch(OFR_Issues, Defaults(OFR_Issues), {
    ItemID: "OFR-" & Text(CountRows(OFR_Issues) + 1, "00"),
    Title: selectedIntake.Title,
    Owner: TextInput6.Text,
    Priority: selectedIntake.Priority,
    Status: {Value: "New"},
    DateRaised: selectedIntake.DateSubmitted,
    LastUpdated: Now(),
    DaysSinceUpdate: 0,
    FunctionalGroup: selectedIntake.FunctionalGroup
});
Patch(OFR_IntakeQueue, selectedIntake, {TriageStatus: {Value: "Accepted"}});
Reset(TextInput6);
UpdateContext({showIntakePanel: false});
Notify("Issue accepted into tracker", NotificationType.Success)
```

### 5.3 Screen: TrackerScreen

**Purpose:** Main working view — sortable, filterable issue table.

| Component | Type | Data Source | Key Logic |
|-----------|------|-------------|-----------|
| Filter Toggles | Button group | — | Context variables: `varFilter` (AllOpen/Stale/High/Medium/Low) |
| Search Box | Text input | — | Context variable: `varSearch` |
| Issue Gallery | Gallery (table) | OFR_Issues | Complex filter combining varFilter + varSearch (see formula below) |
| Column Headers | Labels/Buttons | — | Set `varSortColumn` and `varSortAscending` context variables |
| FunctionalGroup Column | Label | OFR_Issues | `ThisItem.FunctionalGroup.Value`. Displayed between Status and Days Stale columns. Truncated to 18 characters with full value in tooltip. |
| Staleness Indicator | Label/Icon | — | Conditional fill: `If(ThisItem.DaysSinceUpdate <= 7, Primary Blue, If(ThisItem.DaysSinceUpdate <= 14, Orange Lighter, Primary Red))` |
| Row OnSelect | — | — | `Navigate(IssueDetailScreen, ScreenTransition.None, {varSelectedIssue: ThisItem})` |
| **Quick-Update Panel** | | | |
| Panel Background | Rectangle | — | Visible: `showQuickUpdate`. White panel, right-aligned. |
| Save Update Button | Button | OFR_UpdateHistory, OFR_Issues | Quick-saves an update note and optional status change from the panel |
| View Full Detail Button | Button | — | `Navigate(IssueDetailScreen, ScreenTransition.None, {varSelectedIssue: selectedIssue})` |
| Back Button | Button | — | `Navigate(DashboardScreen)` |

**Gallery Items Formula (conceptual):**
```
SortByColumns(
  Search(
    Filter(
      OFR_Issues,
      Status <> "Closed",
      Switch(varFilter,
        "Stale", StalenessFlag = "Stale",
        "High", Priority = "High",
        "Medium", Priority = "Medium",
        "Low", Priority = "Low",
        true
      )
    ),
    varSearch,
    "Title", "Owner", "ItemID", "FunctionalGroup"
  ),
  varSortColumn,
  If(varSortAscending, SortOrder.Ascending, SortOrder.Descending)
)
```

### 5.4 Screen: IssueDetailScreen

**Purpose:** Full issue view with update history and add-update form.

| Component | Type | Data Source | Key Logic |
|-----------|------|-------------|-----------|
| Issue Header | Labels | OFR_Issues | Bound to `varSelectedIssue` context variable. Displays ItemID, Title, Status, Priority, Owner, DateRaised, and FunctionalGroup. |
| Update History Gallery | Gallery | OFR_UpdateHistory | `Filter(OFR_UpdateHistory, ParentItemID = varSelectedIssue.ItemID)` sorted descending by UpdateDate |
| Notes Input | Text input | — | Required field for new update |
| Status Dropdown | Dropdown | — | Items: Status choices. Default: `varSelectedIssue.Status` |
| Save Update Button | Button | OFR_UpdateHistory, OFR_Issues | See save logic below |
| Back Button | Button | — | `Navigate(TrackerScreen)` |

**Save Update Logic (conceptual):**
```
// 1. Create update history entry
Patch(OFR_UpdateHistory, Defaults(OFR_UpdateHistory), {
  Title: varSelectedIssue.ItemID,
  ParentItemID: varSelectedIssue.ItemID,
  UpdateDate: Now(),
  StatusAtUpdate: ddStatus.Selected.Value,
  Notes: txtNotes.Text,
  UpdatedBy: User().FullName
});

// 2. Update the parent issue
Patch(OFR_Issues, varSelectedIssue, {
  LastUpdated: Now(),
  Status: ddStatus.Selected,
  DaysSinceUpdate: 0
});

// 3. Reset form
Reset(txtNotes);
```

### 5.5 Screen: SubmitScreen

**Purpose:** Standalone form for submitting new issues into the Intake Queue.

| Component | Type | Data Source | Key Logic |
|-----------|------|-------------|-----------|
| Form Title | Label | — | "Submit New Issue" heading |
| Title Input | TextInput | — | Required. Issue title. |
| Description Input | TextInput | — | Required. Multi-line issue description. |
| Priority Dropdown | Dropdown | — | Items: "High", "Medium", "Low" |
| FunctionalGroup Dropdown | Dropdown | — | Items: 10 functional groups (Risk Management Office, Engagement Risk, Client Risk and KYC, Technology Risk & AI Trust, National Security, OGC General Counsel, OGC Privacy, OGC Contracts, Internal Audit, Independence) |
| Submit Button | Button | OFR_IntakeQueue | `Patch(OFR_IntakeQueue, Defaults(OFR_IntakeQueue), {Title: txtTitle.Text, Description: txtDescription.Text, Priority: ddPriority.Selected, FunctionalGroup: ddNewGroup.Selected, TriageStatus: {Value: "Pending"}, DateSubmitted: Now()})` |
| Back Button | Button | — | `Navigate(DashboardScreen)` |

### 5.6 Navigation Map

```
DashboardScreen
  ├── [View Tracker] ──────▶ TrackerScreen
  │                            ├── [Row tap] ──────▶ IssueDetailScreen
  │                            │                       └── [Back] ──▶ TrackerScreen
  │                            └── [Back] ──────────▶ DashboardScreen
  ├── [+ Submit New Issue] ─▶ SubmitScreen
  │                            └── [Back] ──────────▶ DashboardScreen
  ├── [Group Allocation] ──▶ GroupAllocationScreen
  │                            └── [Back] ──────────▶ DashboardScreen
  ├── [Kanban Board] ──────▶ KanbanScreen
  │                            ├── [Card tap] ─────▶ IssueDetailScreen
  │                            │                       └── [Back] ──▶ TrackerScreen
  │                            └── [Back] ──────────▶ DashboardScreen
  ├── [Intake item tap] ───▶ Opens Intake Review panel (same screen)
  │                            ├── [Accept] ─────────▶ Creates issue + closes panel
  │                            ├── [Reject] ─────────▶ Marks rejected + closes panel
  │                            └── [X Close] ────────▶ Closes panel
  ├── [Promote] ──────────▶ Triggers Power Automate flow
  └── [Dismiss] ──────────▶ Patches intake item
```

### 5.7 Screen: GroupAllocationScreen

**Purpose:** Shows active issue counts per functional group in a card grid, enabling managers to see workload distribution across the ten OFR functional groups.

| Component | Type | Data Source | Key Logic |
|-----------|------|-------------|-----------|
| Header Bar | Rectangle | — | Primary Blue fill. App title and screen label. |
| Back Button | Button | — | `Navigate(DashboardScreen)` |
| Group Cards (×10) | Rectangle + Labels | OFR_Issues | 2×5 card grid (240w × 90h each). Each card displays the group name and active issue count. |
| Card Count Formula | Label | OFR_Issues | `CountRows(Filter(OFR_Issues, FunctionalGroup.Value = "[GroupName]", Status.Value <> "Closed"))` |
| Total Active Count | Label | OFR_Issues | `CountRows(Filter(OFR_Issues, Status.Value <> "Closed"))` |
| Unassigned Count | Label | OFR_Issues | `CountRows(Filter(OFR_Issues, IsBlank(FunctionalGroup.Value), Status.Value <> "Closed"))` — shown in orange as a warning indicator |

**Card Grid Layout:**

```
Row 1 (Y=80):   Risk Mgmt Office  │ Engagement Risk  │ Client Risk & KYC  │ Tech Risk & AI Trust  │ National Security
Row 2 (Y=190):  OGC General Counsel│ OGC Privacy      │ OGC Contracts      │ Internal Audit        │ Independence
```

Each card uses `RGBA(245,245,245,1)` fill with 8px border radius, matching the existing KPI card style. Count labels use Neutral Black `RGBA(45,45,45,1)` and group name labels use `RGBA(100,100,100,1)`.

**Approximate control count:** 30 (3 per card × 10 cards + header bar + title + back button + total label + unassigned label).

### 5.8 Screen: KanbanScreen

**Purpose:** Visual board showing issues as cards arranged in four vertical swim-lanes by status (New, Active, Escalated, Monitoring). Provides a quick overview of issue flow and enables navigation to issue detail.

| Component | Type | Data Source | Key Logic |
|-----------|------|-------------|-----------|
| Header Bar | Rectangle | — | Primary Blue fill. App title and screen label. |
| Back Button | Button | — | `Navigate(DashboardScreen)` |
| Column Headers (×4) | Rectangle + Label | — | Colour-coded headers: New (Primary Blue), Active (Primary Orange), Escalated (Primary Red), Monitoring (Orange Lighter) |
| New Gallery | Vertical Gallery | OFR_Issues | `Filter(OFR_Issues, Status.Value = "New")` sorted by DaysSinceUpdate descending |
| Active Gallery | Vertical Gallery | OFR_Issues | `Filter(OFR_Issues, Status.Value = "Active")` sorted by DaysSinceUpdate descending |
| Escalated Gallery | Vertical Gallery | OFR_Issues | `Filter(OFR_Issues, Status.Value = "Escalated")` sorted by DaysSinceUpdate descending |
| Monitoring Gallery | Vertical Gallery | OFR_Issues | `Filter(OFR_Issues, Status.Value = "Monitoring")` sorted by DaysSinceUpdate descending |
| Card Template | Rectangle + Labels | OFR_Issues | White card with rounded corners containing: ItemID (bold), Priority badge, Title (truncated), Owner, FunctionalGroup (italic), Days since update with staleness colour, and a left-edge staleness accent stripe |
| Card OnSelect | — | — | `Navigate(IssueDetailScreen, ScreenTransition.None, {varSelectedIssue: ThisItem})` |

**Column Layout:**

| Column | Status | X Position | Width | Header Fill |
|--------|--------|------------|-------|-------------|
| 1 | New | 15 | 320 | Primary Blue `RGBA(65,83,133,1)` |
| 2 | Active | 349 | 320 | Primary Orange `RGBA(208,74,2,1)` |
| 3 | Escalated | 683 | 320 | Primary Red `RGBA(224,48,30,1)` |
| 4 | Monitoring | 1017 | 320 | Orange Lighter `RGBA(228,92,43,1)` |

**Card Template Anatomy (TemplateSize: 125px):**
- **Left edge:** 4px-wide accent stripe coloured by staleness (Primary Blue = Current, Orange Lighter = Aging, Primary Red = Stale)
- **Top left:** ItemID in Primary Blue (bold) — e.g., "OFR-4"
- **Top right:** Priority badge (High = Primary Red, Medium = Primary Orange, Low = Primary Blue)
- **Centre:** Title truncated to ~40 characters in Neutral Black
- **Bottom left:** Owner name in secondary text
- **Bottom right:** FunctionalGroup in italic muted text + DaysSinceUpdate in staleness colour

**Note:** Closed issues are excluded from all four galleries. The Kanban view focuses on open/active workflow stages only.

**Approximate control count:** 28 (4 galleries × 7 template controls + 4 column headers + header bar + title + back button).

---

## 6. Automation Layer — Power Automate

### 6.1 Flow: OFR Daily Staleness Calculator

| Property | Value |
|----------|-------|
| Flow ID | `aefb8de0-35fe-4d5d-a629-ddd8502ee5aa` |
| Type | Scheduled cloud flow |
| Trigger | Recurrence — Every 1 day at 06:00 AM UTC |
| Connection | SharePoint ([ADMIN-EMAIL]) |

**Flow Steps:**

```
1. Trigger: Recurrence (daily, 06:00 AM)
     │
2. Get items: SharePoint → OFR_Issues
     │  Site: OFR Issue Tracker
     │  List: OFR_Issues
     │  Filter Query: Status ne 'Closed'
     │
3. Apply to each: body/value
     │
     └─ 4. Update item: SharePoint → OFR_Issues
            Site: OFR Issue Tracker
            List: OFR_Issues
            Id: items('Apply_to_each')?['ID']
            DaysSinceUpdate: [expression A]
            StalenessFlag Value: [expression B]
```

**Expression A — DaysSinceUpdate:**
```
div(
  sub(
    ticks(utcNow()),
    ticks(items('Apply_to_each')?['LastUpdated'])
  ),
  864000000000
)
```
*Calculates difference in ticks between now and LastUpdated, then divides by ticks-per-day (864 billion) to get integer days.*

**Expression B — StalenessFlag:**
```
if(
  lessOrEquals(
    div(sub(ticks(utcNow()),ticks(items('Apply_to_each')?['LastUpdated'])),864000000000),
    7
  ),
  'Current',
  if(
    lessOrEquals(
      div(sub(ticks(utcNow()),ticks(items('Apply_to_each')?['LastUpdated'])),864000000000),
      14
    ),
    'Aging',
    'Stale'
  )
)
```

**Thresholds:**

| DaysSinceUpdate | StalenessFlag |
|-----------------|---------------|
| 0–7 | Current |
| 8–14 | Aging |
| 15+ | Stale |

### 6.2 Flow: OFR Intake Promotion

| Property | Value |
|----------|-------|
| Flow ID | `1c631640-113f-4602-805e-1d693582de8c` |
| Type | Instant cloud flow |
| Trigger | Power Apps V2 |
| Input Parameters | IntakeItemID (Number) |
| Output Parameters | NewItemID (Text) |
| Connection | SharePoint ([ADMIN-EMAIL]) |

**Flow Steps:**

```
1. Trigger: When Power Apps calls a flow (V2)
     │  Input: IntakeItemID (Number)
     │
2. Get item: SharePoint → OFR_IntakeQueue
     │  Site: OFR Issue Tracker
     │  List: OFR_IntakeQueue
     │  Id: IntakeItemID
     │
3. Create item: SharePoint → OFR_Issues
     │  Site: OFR Issue Tracker
     │  List: OFR_Issues
     │  Title: [Get item → Title]
     │  ItemID: concat('OFR-', string(outputs('Get_item')?['body/ID']))
     │  Owner: [Get item → Owner]
     │  Priority Value: [Get item → Priority Value]
     │  Status Value: "New"
     │  DateRaised: utcNow()
     │  LastUpdated: utcNow()
     │  NextAction: [Get item → Description]
     │  DaysSinceUpdate: 0
     │  StalenessFlag Value: "Current"
     │  FunctionalGroup Value: [Get item → FunctionalGroup Value]
     │
4. Create item 1: SharePoint → OFR_UpdateHistory
     │  Site: OFR Issue Tracker
     │  List: OFR_UpdateHistory
     │  Title: [Create item → ItemID]
     │  ParentItemID: [Create item → ItemID]
     │  UpdateDate: utcNow()
     │  StatusAtUpdate Value: "New"
     │  Notes: "Promoted from intake queue"
     │  UpdatedBy: [Get item → Owner]
     │
5. Update item: SharePoint → OFR_IntakeQueue
     │  Site: OFR Issue Tracker
     │  List: OFR_IntakeQueue
     │  Id: IntakeItemID
     │  TriageStatus Value: "Promoted"
     │
6. Respond to a Power App or flow
       Output: NewItemID = [Create item → ItemID]
```

**ItemID Generation Strategy:**

The ItemID is generated using `concat('OFR-', string(ID))` where `ID` is the SharePoint auto-increment ID from the OFR_IntakeQueue item. This means:
- ItemIDs are unique (SharePoint IDs are unique per list)
- The numbering may not be perfectly sequential if intake items are deleted
- The prefix "OFR-" provides namespace identification

---

## 7. Security & Access Control

### 7.1 Authentication

| Mechanism | Details |
|-----------|---------|
| Identity Provider | Microsoft Entra ID (Azure AD) |
| Authentication Flow | SSO via Power Platform — no custom auth code |
| Session Management | Managed by Power Platform runtime |
| MFA | Inherited from tenant Conditional Access policies |

### 7.2 Authorization

| Resource | Access Control Mechanism |
|----------|------------------------|
| SharePoint Site | M365 Group membership (OFR Issue Tracker Members) |
| SharePoint Lists | Inherited from site permissions |
| Power Apps | App sharing — shared with OFR Issue Tracker Members group |
| Power Automate Flows | Run-only access for app users; edit access for flow owner |

### 7.3 Data Protection

| Measure | Details |
|---------|---------|
| Encryption at Rest | SharePoint Online default encryption (Microsoft-managed keys) |
| Encryption in Transit | TLS 1.2 (enforced by M365) |
| Data Residency | Per M365 tenant geo configuration |
| Backup | SharePoint Online built-in versioning and recycle bin (93-day retention) |
| Audit Logging | M365 Unified Audit Log captures SharePoint and Power Platform activity |

---

## 8. Integration Points

### 8.1 Current Integrations

| Integration | Type | Direction | Details |
|-------------|------|-----------|---------|
| Power Apps ↔ SharePoint | Data connector | Bidirectional | Standard SharePoint connector. Read/write to all three lists. |
| Power Apps → Power Automate | Flow trigger | Outbound | Power Apps V2 trigger calls Intake Promotion flow with IntakeItemID parameter. |
| Power Automate → SharePoint | Data connector | Read/Write | Staleness flow reads and updates OFR_Issues. Promotion flow reads/writes all three lists. |
| Microsoft Teams | Tab embed | Display | Power App pinned as a Teams tab for channel access (optional deployment). |

### 8.2 Future Integration Opportunities

| Integration | Purpose | Complexity |
|-------------|---------|------------|
| Power Automate → Outlook | Send email notifications for stale issues | Low |
| Power BI → SharePoint | Advanced reporting dashboards | Medium |
| Power Apps → Microsoft Graph | Use People picker for Owner field | Low |
| Power Automate → Adaptive Cards | Push issue updates to Teams channels | Medium |

---

## 9. Non-Functional Requirements

### 9.1 Performance

| Metric | Target | Notes |
|--------|--------|-------|
| App load time | < 5 seconds | Dependent on SharePoint connector response time |
| Gallery refresh | < 3 seconds | For up to 50 items |
| Flow execution (staleness) | < 2 minutes | For up to 100 open items |
| Flow execution (promotion) | < 30 seconds | Single item operation |

### 9.2 Availability

| Metric | Target | Notes |
|--------|--------|-------|
| System uptime | 99.9% | Governed by Microsoft 365 SLA |
| Planned maintenance | Per M365 schedule | No custom infrastructure to maintain |

### 9.3 Scalability

| Dimension | Current Capacity | Limit |
|-----------|-----------------|-------|
| Concurrent users | ~10 | Power Apps standard connector limits |
| OFR_Issues items | 8 (sample) | 30 million (SharePoint list limit) |
| OFR_UpdateHistory items | 17 (sample) | 30 million |
| Power Automate runs/month | ~30 (daily staleness) | 750 (free license) / 10,000 (per user) |

### 9.4 Maintainability

| Aspect | Approach |
|--------|----------|
| App updates | Edit in Power Apps Studio (make.powerapps.com), publish |
| Flow updates | Edit in Power Automate (make.powerautomate.com) |
| Schema changes | Modify SharePoint list columns directly |
| No deployment pipeline | Changes are live on publish — no CI/CD required |
| Version history | Power Apps maintains version history with rollback capability |

---

## 10. Deployment & Environments

### 10.1 Current Environment

| Property | Value |
|----------|-------|
| Environment | [TENANT-DOMAIN] (default) |
| Tenant | [TENANT-ORG-NAME] ([TENANT]) |
| Region | Default M365 tenant region |

### 10.2 Deployment Steps (New Environment)

1. **Create SharePoint site** — Team site named "OFR Issue Tracker"
2. **Create 3 SharePoint lists** — OFR_Issues, OFR_UpdateHistory, OFR_IntakeQueue with schemas per Section 4
3. **Import or recreate Power Apps** — Canvas app with 6 screens per Section 5
4. **Create Power Automate flows** — Two flows per Section 6
5. **Connect flows to app** — Wire Intake Promotion flow to Dashboard Promote button
6. **Share app** — Share with target M365 group
7. **Optional: Pin to Teams** — Add as Teams channel tab

### 10.3 Rollback Strategy

| Scenario | Action |
|----------|--------|
| Bad Power Apps update | Restore previous version from Power Apps version history |
| Bad flow change | Restore previous version from Power Automate flow history |
| Data issue | Restore items from SharePoint recycle bin (93-day retention) |
| Complete rollback | Remove app sharing, turn off flows. Data remains in SharePoint. |

---

## 11. Known Limitations & Future Roadmap

### 11.1 Known Limitations

| Limitation | Impact | Workaround |
|------------|--------|------------|
| Owner field is free text (not People picker) | No user validation, potential typos | Admin can correct via SharePoint list |
| ItemID uses intake queue SP ID | IDs may have gaps if intake items deleted | Cosmetic only; IDs are unique |
| No bilingual support | English-only interface | Deferred to Phase 2 |
| No in-app export | Cannot download CSV from Power Apps | Use SharePoint list "Export to Excel" |
| No email notifications | Users must check app proactively | Deferred to Phase 2 |
| Single-environment | No dev/test/prod separation | Acceptable for current team size |
| Staleness updates once daily | Colors may lag up to 24 hours after update | DaysSinceUpdate resets immediately; flag catches up next morning |
| No role-based UI | All members see all features | Acceptable for small team; can add role logic later |

### 11.2 Future Roadmap

| Phase | Feature | Description | Effort |
|-------|---------|-------------|--------|
| 2a | Email Notifications | Power Automate flow sends digest of stale items to owners | Low |
| 2b | Bilingual EN/FR | Power Apps language variable with Switch() for all labels | Medium |
| 2c | CSV Export | Power Automate flow generates CSV and emails or stores in SharePoint | Low |
| 2d | Teams Adaptive Cards | Push issue updates/escalations to Teams channels | Medium |
| 3a | People Picker for Owner | Replace text field with M365 People picker control | Low |
| 3b | Power BI Dashboard | Advanced analytics with trend charts and aging curves | Medium |
| 3c | Approval Workflow | Formal approval step for escalated issues | Medium |

---

## 12. Appendices

### Appendix A: Key URLs

| Resource | URL |
|----------|-----|
| SharePoint Site | https://[TENANT].sharepoint.com/sites/OFRIssueTracker |
| OFR_Issues | https://[TENANT].sharepoint.com/sites/OFRIssueTracker/Lists/OFR_Issues |
| OFR_UpdateHistory | https://[TENANT].sharepoint.com/sites/OFRIssueTracker/Lists/OFR_UpdateHistory |
| OFR_IntakeQueue | https://[TENANT].sharepoint.com/sites/OFRIssueTracker/Lists/OFR_IntakeQueue |
| Power Apps Studio | https://make.powerapps.com |
| Power Automate | https://make.powerautomate.com |

### Appendix B: Component IDs

| Component | ID |
|-----------|-----|
| Power Apps App | `0fbbc26c-ad71-476a-bcfc-edc0d7989533` |
| OFR_Issues List GUID | `a70da6a6-1f0a-4fd3-bb4e-cf7847e18a99` |
| Staleness Calculator Flow | `aefb8de0-35fe-4d5d-a629-ddd8502ee5aa` |
| Intake Promotion Flow | `1c631640-113f-4602-805e-1d693582de8c` |

### Appendix C: Related Documents

| Document | Description |
|----------|-------------|
| OFR-User-Guide.md | End-user guidance for the OFR Issue Tracker |
| OFR-Test-Plan.md | 76-case test plan covering all functionality |
| OFR-Completion-Guide.md | Build summary with all technical details |
| OFR-PowerApps-Completion-Guide.md | Detailed Power Apps construction guide |
| hidden-shimmying-moon.md | Original implementation plan |

### Appendix D: Glossary

| Term | Definition |
|------|-----------|
| **Canvas App** | A Power Apps application type where the UI is designed visually on a free-form canvas |
| **Cloud Flow** | A Power Automate workflow that runs in the cloud (not on a desktop) |
| **Entra ID** | Microsoft's cloud identity and access management service (formerly Azure Active Directory) |
| **Intake** | The process of submitting a new potential issue before it is accepted into active tracking |
| **Promotion** | The act of accepting an intake item and creating a corresponding tracked issue |
| **Staleness** | A measure of how long an issue has gone without an update; calculated as days since LastUpdated |
| **Triage** | The process of reviewing intake items and deciding whether to promote or dismiss them |
| **SPA** | Single Page Application — the previous React/Vite implementation that this system replaces |
