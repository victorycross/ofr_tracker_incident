# Enterprise Incident Tracker — System Design Document (SDD)

**Version:** 1.0
**Date:** February 2026
**Classification:** Internal

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

This System Design Document describes the technical architecture, data model, component design, and integration details of the Enterprise Incident Tracker (EIT) — an M365-native platform for centralized enterprise incident management across five enterprise domains.

### 1.2 Scope

The EIT covers:
- Incident intake and triage workflow
- Active incident tracking with domain classification, ERM-aligned severity, and escalation thresholds
- Update workflow with append-only audit trail
- Automated daily staleness calculation and escalation threshold automation
- Domain lead and executive escalation notification emails
- Privileged & Confidential (P&C) incident designation and item-level access restriction
- Dashboard KPIs, domain overview, Kanban board, and executive report views

The following are explicitly out of scope for Phase 1:
- Automated ingest from ServiceNow, iSight, or other source systems
- Power BI dashboard (manual export only)
- Microsoft Teams channel notifications
- Automated incident archiving
- Bilingual (EN/FR) support
- CSV/PDF export from within the app

### 1.3 Background

The organization manages incidents across five domains (National Security, NIS, Privacy & Technology, Human Capital, Physical Security) using fragmented tools — ServiceNow, iSight, manual HC processes, and ad hoc tracking. The EIT replaces fragmented tracking with a single enterprise-wide view while preserving domain-level ownership and escalation paths.

The EIT is built on the M365-native pattern established by the OFR Issue Tracker, extended to support multiple domains, ERM-aligned severity, escalation automation, P&C designation, and a net-new executive reporting view.

### 1.4 Assumptions

- The organization has M365 Business Standard or higher licensing for all users.
- Power Automate Standard connectors (Outlook Send email V2) are available via the seeded plan.
- The deploying administrator has SharePoint Site Owner, Power Apps, and Power Automate access.
- All domain leads have M365 mailboxes accessible via the Outlook connector.

---

## 2. System Overview

The EIT is a three-layer M365-native application:

```
Entra ID (SSO) ──► Power Apps Canvas App ◄──► SharePoint Online (4 Lists)
                        │                              ▲
                        ▼                              │
               Power Automate (3 Cloud Flows) ─────────┘
                        │
                        ▼
               Outlook (Escalation Emails)
```

| Layer | Technology | Purpose |
|-------|-----------|---------|
| Data | SharePoint Online | Persistent storage for all incident records, history, intake queue, and domain contacts |
| UI | Power Apps Canvas App | User interface for incident submission, triage, tracking, updating, and reporting |
| Automation | Power Automate Cloud Flows | Scheduled staleness calculation + escalation threshold automation; instant intake promotion; instant escalation notifications |
| Notification | Outlook (via Power Automate) | Email escalation notifications to domain leads and executive sponsors |
| Identity | Entra ID (M365) | Single sign-on, P&C group membership check |

---

## 3. Architecture

### 3.1 Component Diagram

```
┌──────────────────────────────────────────────────────────────┐
│                    SharePoint Online                         │
│                                                              │
│  ┌─────────────────┐  ┌──────────────────┐                  │
│  │  EIT_Incidents  │  │EIT_IncidentHistory│                  │
│  │  (20 columns)   │  │  (9 columns)      │                  │
│  └────────┬────────┘  └────────┬──────────┘                  │
│           │ ParentItemID link  │                             │
│  ┌────────┴────────┐  ┌────────┴──────────┐                  │
│  │  EIT_IntakeQueue│  │EIT_DomainContacts │                  │
│  │  (10 columns)   │  │  (5 columns, 5 rows)│                 │
│  └─────────────────┘  └───────────────────┘                  │
└──────────────────────────────────────────────────────────────┘
            ▲                          ▲
            │ Read/Write               │ Read
            ▼                          ▼
┌──────────────────────┐    ┌──────────────────────────┐
│  Power Apps Canvas   │    │   Power Automate Flows   │
│  Enterprise Incident │    │                          │
│  Tracker             │    │  1. Daily Staleness +    │
│                      │    │     Escalation Calc      │
│  7 Screens:          │    │     (Scheduled, 07:00)   │
│  - DashboardScreen   │    │                          │
│  - TrackerScreen     │    │  2. Intake Promotion     │
│  - IssueDetailScreen │    │     (Instant, from app)  │
│  - DomainOverview    │    │                          │
│  - KanbanScreen      │    │  3. Domain Escalation    │
│  - ExecutiveReport   │    │     Notifier             │
│                      │    │     (Instant, from app)  │
│  3 Flows Connected:  │    │         │                │
│  - EIT Intake Promo  │    │         ▼                │
│  - EIT Escalation    │    │    Outlook (Email)       │
│    Notifier          │    │                          │
└──────────────────────┘    └──────────────────────────┘
            ▲
            │ Entra ID SSO
            │ Office365Groups.IsMemberOfGroup()
            ▼
┌──────────────────────┐
│     Entra ID         │
│  - EIT Members       │
│  - EIT Viewers       │
│  - EIT Owners        │
│  - EIT-PAC-Authorized│
└──────────────────────┘
```

### 3.2 Data Flow — Incident Lifecycle

```
1. SUBMISSION
   User fills "+ New Incident" overlay in Power App
        │
        ▼
   Patch to EIT_IntakeQueue (TriageStatus = "Pending")

2. TRIAGE
   Domain lead reviews intake queue in DashboardScreen
        │
        ├── Clicks "Move to Tracker" button
        │       │
        │       ▼
        │   EIT Intake Promotion flow runs:
        │     - Creates EIT_Incidents row (EIT-NNN)
        │     - Creates EIT_IncidentHistory opening entry
        │     - Updates EIT_IntakeQueue row (TriageStatus = "Promoted")
        │     - Returns new ItemID to app
        │
        ├── Clicks "Accept" (direct, no flow) → TriageStatus = "Accepted"
        ├── Clicks "Reject" → TriageStatus = "Rejected"
        └── Clicks "Dismiss" → TriageStatus = "Dismissed"

3. ACTIVE MANAGEMENT
   Domain lead/owner opens incident via TrackerScreen or KanbanScreen
        │
        ▼
   IssueDetailScreen: Add Update form
     - Patch to EIT_IncidentHistory (new entry)
     - Patch to EIT_Incidents (Status, Severity, EscalationThreshold, DaysSinceUpdate=0)
     - If UpdateType = "Escalation": EIT Domain Escalation Notifier flow fires → email sent

4. DAILY AUTOMATION (07:00)
   Daily Staleness & Escalation Calculator flow runs
     - For each open incident:
         DaysSinceUpdate = days(utcNow() - LastUpdated)
         StalenessFlag = Current/Aging/Stale
         EscalationThreshold = auto-escalate if Critical>14d or High>7d

5. CLOSURE
   Domain lead sets Status = "Contained" (DateContained recorded)
   Then Status = "Closed" (incident archived from active views)
```

---

## 4. Data Layer — SharePoint Online

### 4.1 Site

| Property | Value |
|----------|-------|
| Site Name | `Enterprise Incident Tracker` |
| Site URL | `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker` |
| Site Type | Team site (M365 group-connected) |
| Privacy | Private |

### 4.2 EIT_Incidents — Primary Incident List

**Purpose:** One row per enterprise incident. The primary tracking list.

**Column count:** 20

| # | Column | Type | Notes |
|---|--------|------|-------|
| 1 | Title | Single line | Incident title. Required. Max 255 chars. |
| 2 | ItemID | Single line | `EIT-NNN`. Set by Intake Promotion flow. Logical primary key for app relationships. |
| 3 | Owner | Single line | Accountable person. Free text. |
| 4 | Domain | Choice | National Security, NIS, Privacy & Technology, Human Capital, Physical Security |
| 5 | SubDomain | Single line | Optional free-text sub-classification within domain |
| 6 | Severity | Choice | Critical, High, Moderate (default), Low, Informational |
| 7 | Priority | Choice | High, Medium, Low. Operational triage; independent of Severity. |
| 8 | Status | Choice | New, Active, Monitoring, Escalated, Contained, Closed |
| 9 | IncidentType | Choice | Breach, Vulnerability, Policy Violation, Threat, Operational, Personnel, Physical |
| 10 | SourceSystem | Choice | Power Apps (default), ServiceNow, iSight, Manual Entry |
| 11 | DateRaised | Date | Date the incident entered the tracker |
| 12 | DateContained | Date | Date the incident was contained. Set by app when Status=Contained. |
| 13 | LastUpdated | Date | Updated by Power Apps on every Patch call. Used for staleness calculation. |
| 14 | NextAction | Multiple lines | Free text. Current status summary and next steps. |
| 15 | EscalationThreshold | Choice | Not Escalated (default), Watch, Escalate, Executive Brief |
| 16 | DaysSinceUpdate | Number | Calculated daily by flow. Days since LastUpdated. |
| 17 | StalenessFlag | Choice | Current (0-7d), Aging (8-14d), Stale (15+d) |
| 18 | ConfidentialityLevel | Choice | Internal (default), Restricted, Confidential |
| 19 | RelatedItemIDs | Single line | Comma-separated EIT ItemIDs or external ticket numbers (free text) |
| 20 | PrivilegedAndConfidential | Yes/No | Default: No. Triggers item-level permission restriction process. |

**Key constraints:**
- `Domain` choices must exactly match `EIT_DomainContacts.Title` values (case-sensitive) for escalation flow lookup.
- `Severity` choices must exactly match those referenced in the Daily Staleness flow expression.
- `PrivilegedAndConfidential = Yes` triggers the P&C access restriction process (Layer 2 item-level permissions).

### 4.3 EIT_IncidentHistory — Audit Trail

**Purpose:** Append-only audit trail. One row per update event. Records never edited or deleted after creation.

**Column count:** 9

| # | Column | Type | Notes |
|---|--------|------|-------|
| 1 | Title | Single line | Set to the ParentItemID value (e.g., `EIT-001`). Used for SharePoint list display. |
| 2 | ParentItemID | Single line | EIT_Incidents.ItemID of the parent incident. Logical foreign key. |
| 3 | UpdateDate | Date (incl. time) | Timestamp of the update. Set to `utcNow()` by flow or `Now()` by app. |
| 4 | StatusAtUpdate | Choice | Same choices as EIT_Incidents.Status. Records status at time of this update. |
| 5 | Notes | Multiple lines | Free-text update notes. Required before Save Update is enabled. |
| 6 | UpdatedBy | Single line | `User().FullName` from Power Apps. |
| 7 | SeverityAtUpdate | Choice | Same choices as EIT_Incidents.Severity. Records severity at time of this update. |
| 8 | UpdateType | Choice | Status Change (default), Evidence Added, Escalation, Containment, Closure |
| 9 | PrivilegedAndConfidential | Yes/No | Default: No. Set to match parent incident's P&C flag on entry creation. |

**Append-only enforcement:** SharePoint list settings should be configured so that `EIT Members` have Contribute access only for creating new items, not editing or deleting existing ones. Configure via List Settings → Advanced Settings → "Create items and edit items that were created by the user" (restrict to own items, or EIT Owners only for edit/delete).

### 4.4 EIT_IntakeQueue — Triage Queue

**Purpose:** Holds newly submitted incidents pending domain lead triage. Items are transient — they move to `Promoted`, `Accepted`, `Rejected`, or `Dismissed` status and are no longer shown in the active queue.

**Column count:** 10

| # | Column | Type | Notes |
|---|--------|------|-------|
| 1 | Title | Single line | Incident title as submitted |
| 2 | Owner | Single line | Submitter-assigned owner |
| 3 | Priority | Choice | High, Medium, Low (submitter's operational priority) |
| 4 | Description | Multiple lines | Submitter's description of the incident |
| 5 | Domain | Choice | Same choices as EIT_Incidents.Domain |
| 6 | IncidentType | Choice | Same choices as EIT_Incidents.IncidentType |
| 7 | DateSubmitted | Date | Submission timestamp |
| 8 | TriageStatus | Choice | Pending (default), Promoted, Dismissed, Accepted, Rejected |
| 9 | SubmissionSource | Choice | Power Apps (default), Email Forward, Manual Entry |
| 10 | PrivilegedAndConfidential | Yes/No | Default: No. Submitter's suggested P&C flag — must be confirmed by domain lead at triage. |

### 4.5 EIT_DomainContacts — Reference Data

**Purpose:** Static reference list. Exactly 5 rows — one per domain. Powers escalation email routing.

**Column count:** 5

| Column | Type | Notes |
|--------|------|-------|
| Title | Single line | Domain name. Must exactly match Domain choice values in EIT_Incidents (case-sensitive). |
| LeadName | Single line | Domain lead display name |
| LeadEmail | Single line | Domain lead email. Used as `To` on `Escalate` level notifications. |
| EscalationEmail | Single line | Executive escalation email. Used as `To` on `Executive Brief` notifications. |
| ActiveIncidentCount | Number | Optional counter. Not used by flows — informational only. |

**Critical:** The Power Automate Domain Escalation Notifier flow performs a case-sensitive Filter Array match on `Title = incident.Domain.Value`. The 5 `Title` values must match the 5 Domain choice values in `EIT_Incidents` exactly, including the ampersand in "Privacy & Technology".

### 4.6 Data Relationships

```
EIT_Incidents.ItemID (text)
    ──────────────────────────────► EIT_IncidentHistory.ParentItemID (text)
                                    (one-to-many logical relationship)
                                    (not enforced by SharePoint — enforced by app)

EIT_Incidents.Domain.Value (choice text)
    ──────────────────────────────► EIT_DomainContacts.Title (text)
                                    (used by escalation flow Filter Array)
                                    (case-sensitive match)

EIT_IntakeQueue.ID (SharePoint integer)
    ──────────────────────────────► Intake Promotion flow input parameter
                                    (flow creates EIT_Incidents row from intake data)
```

---

## 5. UI Layer — Power Apps Canvas App

### 5.1 App Overview

| Property | Value |
|----------|-------|
| App Name | Enterprise Incident Tracker |
| Type | Canvas App |
| Format | Tablet (landscape) — 1366 × 768 |
| Data Sources | EIT_Incidents, EIT_IncidentHistory, EIT_IntakeQueue, EIT_DomainContacts |
| Flows Connected | EIT Intake Promotion, EIT Domain Escalation Notifier |
| Connectors | SharePoint, Office365Groups |

### 5.2 Screens

| Screen | Purpose | Key Formulas |
|--------|---------|--------------|
| **DashboardScreen** | KPI cards (Open, Critical, Stale, Escalated, P&C), Intake Queue gallery, submit overlay, navigation | `CountRows(Filter(...))` for KPIs; `EITIntakePromotion.Run()` on Promote button |
| **TrackerScreen** | Domain filter chips, search, incident table gallery | `SortByColumns(Filter(EIT_Incidents, ..., Or(Not(PrivilegedAndConfidential), varUserHasPACAccess)))` |
| **IssueDetailScreen** | Full incident view, P&C banner, update history, add update form | Dual-Patch to EIT_IncidentHistory + EIT_Incidents; `EITDomainEscalationNotifier.Run()` on Escalation save |
| **DomainOverviewScreen** | 5 domain cards with active counts, click to filter tracker | `CountRows(Filter(EIT_Incidents, Domain.Value = "...", ...))` per card |
| **KanbanScreen** | 5 swim-lanes (New, Active, Escalated, Monitoring, Contained) | `Filter(EIT_Incidents, Status.Value = "...")` per gallery |
| **ExecutiveReportScreen** | Read-only summary by domain and severity | `AddColumns(["NS","NIS",...], "OpenCount", CountRows(...))` |

### 5.3 Key App-Level Variables

| Variable | Type | Set In | Purpose |
|----------|------|--------|---------|
| `varUserHasPACAccess` | Boolean | App.OnStart | True if current user is in the EIT-PAC-Authorized Entra group |
| `varCurrentUser` | Text | App.OnStart | `User().FullName` — cached for efficiency |
| `varCurrentUserEmail` | Text | App.OnStart | `User().Email` — cached for efficiency |
| `varSelectedIncident` | Record | Navigate to IssueDetailScreen | Currently selected incident record |
| `varDomainFilter` | Text | TrackerScreen.OnVisible, filter buttons | Active domain filter value |
| `varSearch` | Text | txtSearch.OnChange | Search string applied to incident gallery |
| `varShowContained` | Boolean | togShowContained | Toggle Contained incidents in tracker |
| `varShowNewIncident` | Boolean | btnNewIncident.OnSelect | Controls submit overlay visibility |
| `showIntakePanel` | Boolean | galIntakeQueue.OnSelect | Controls intake review panel visibility |
| `selectedIntake` | Record | galIntakeQueue.OnSelect | Currently selected intake item in review panel |

### 5.4 P&C Access Gate

```
// App.OnStart:
Set(
    varUserHasPACAccess,
    Office365Groups.IsMemberOfGroup("[EIT-PAC-GROUP-ID]", User().Email).value
);

// TrackerScreen gallery Items (partial):
Filter(
    EIT_Incidents,
    ...,
    Or(
        Not(PrivilegedAndConfidential),
        varUserHasPACAccess
    )
)
```

> **Important:** This is a UX-layer control only. The enforceable security control for P&C incidents is SharePoint item-level permissions (Layer 2). The Power Apps gate improves usability by not showing P&C records to unauthorized users, but does not prevent direct SharePoint list access. See Section 7.

### 5.5 Design System

The app uses the Appkit4 colour theme. Key semantic colors:

| Element | RGBA |
|---------|------|
| Header / Nav | `RGBA(65,83,133,1)` — Appkit4 Blue |
| Primary action buttons | `RGBA(208,74,2,1)` — Appkit4 Orange |
| Critical / Stale alerts | `RGBA(224,48,30,1)` — Appkit4 Red |
| Critical Severity (darkest) | `RGBA(119,40,32,1)` — Dark Red |
| Contained status | `RGBA(50,140,80,1)` — Green |
| Screen background | `RGBA(245,247,250,1)` |

---

## 6. Automation Layer — Power Automate

### 6.1 Flow 1: EIT Daily Staleness & Escalation Calculator

| Property | Value |
|----------|-------|
| Type | Scheduled cloud flow |
| Trigger | Recurrence — daily at 07:00 UTC |
| Connection | SharePoint (admin account) |
| Purpose | Recalculates DaysSinceUpdate, StalenessFlag, and auto-escalation threshold for every open incident |

**Logic:**
1. Get items from `EIT_Incidents` where `Status ne 'Closed' and Status ne 'Contained'`
2. For each incident:
   - Calculate `DaysSinceUpdate` from ticks difference between `utcNow()` and `LastUpdated`
   - Set `StalenessFlag`: 0-7d = Current, 8-14d = Aging, 15+d = Stale
   - Set `EscalationThreshold`: Critical+>14d = Executive Brief; High+>7d = Escalate; else preserve current value
3. Update item in `EIT_Incidents`

**Key expression (`EscalationThreshold`):**
```
if(and(greater(div(sub(ticks(utcNow()),ticks(items('Apply_to_each')?['LastUpdated'])),864000000000),14),equals(items('Apply_to_each')?['Severity']?['Value'],'Critical')),'Executive Brief',if(and(greater(div(sub(ticks(utcNow()),ticks(items('Apply_to_each')?['LastUpdated'])),864000000000),7),equals(items('Apply_to_each')?['Severity']?['Value'],'High')),'Escalate',items('Apply_to_each')?['EscalationThreshold']?['Value']))
```

### 6.2 Flow 2: EIT Intake Promotion

| Property | Value |
|----------|-------|
| Type | Instant cloud flow |
| Trigger | Power Apps (V2) |
| Input | `IntakeItemID` (Number) — SharePoint ID of EIT_IntakeQueue item |
| Output | `NewItemID` (Text) — generated `EIT-NNN` identifier |
| Connection | SharePoint (admin account) |
| Purpose | Promotes a pending intake item into EIT_Incidents |

**Steps:**
1. Get item from `EIT_IntakeQueue` by ID
2. Create item in `EIT_Incidents` (maps intake fields; sets defaults: Severity=Moderate, Status=New, EscalationThreshold=Not Escalated, SourceSystem=Power Apps, ConfidentialityLevel=Internal)
3. ItemID formula: `concat('EIT-', string(outputs('Get_item')?['body/ID']))`
4. Create opening entry in `EIT_IncidentHistory`
5. Update `EIT_IntakeQueue` row: TriageStatus=Promoted
6. Return `NewItemID` to Power Apps

### 6.3 Flow 3: EIT Domain Escalation Notifier

| Property | Value |
|----------|-------|
| Type | Instant cloud flow |
| Trigger | Power Apps (V2) |
| Inputs | `IncidentItemID` (Number), `EscalationLevel` (Text: "Escalate" or "Executive Brief") |
| Output | `Result` (Text) — confirmation message |
| Connections | SharePoint (admin account), Outlook (admin account or shared mailbox) |
| Purpose | Sends escalation notification email and updates EscalationThreshold on the incident |

**Steps:**
1. Get item from `EIT_Incidents` by ID
2. Get all items from `EIT_DomainContacts` (Top 10)
3. Filter array: find row where `Title = incident.Domain.Value`
4. Compose: `first(filtered array)` — contact record
5. Condition: `EscalationLevel = "Executive Brief"`?
   - Yes: Send email to `contact.EscalationEmail`, subject `[EXECUTIVE BRIEF]...`
   - No: Send email to `contact.LeadEmail`, subject `[ESCALATION]...`
   - Both: Prepend `[PRIVILEGED & CONFIDENTIAL]` if `PrivilegedAndConfidential = true`
6. Update `EIT_Incidents`: set `EscalationThreshold = EscalationLevel`
7. Respond to Power Apps with `Result` confirmation

---

## 7. Security & Access Control

### 7.1 Two-Layer Access Control Model

**Layer 1 — SharePoint Permission Groups (list-level):**

| Group | SharePoint Role | Access |
|-------|----------------|--------|
| `EIT Members` | Contribute | Read/write all non-P&C incident records |
| `EIT Viewers` | Read | Read-only access to all non-P&C incident records |
| `EIT Owners` | Full Control | Full administrative access |

**Layer 2 — Item-Level Permissions (row-level, P&C incidents only):**

When `PrivilegedAndConfidential = Yes` is set on an `EIT_Incidents` row:
1. Stop inheriting permissions on that row
2. Remove `EIT Members` and `EIT Viewers`
3. Add only the authorized individuals (incident owner, domain lead, legal counsel, EIT admin)
4. Document the access grant in `EIT_IncidentHistory`

### 7.2 Power Apps P&C Gate

The `varUserHasPACAccess` variable (set via `Office365Groups.IsMemberOfGroup()` on app start) gates the display of P&C incidents in the Tracker Screen gallery. This is a UX control — not a security enforcement mechanism.

**Effective access model:**
- A user with `EIT Members` group membership can access the SharePoint list directly.
- Item-level permissions (Layer 2) prevent unauthorized users from reading P&C incident rows even via direct SharePoint access.
- The Power Apps gallery filter provides a cleaner user experience by not showing P&C records to users without access.

### 7.3 Audit Trail

All SharePoint item access events are logged to the Microsoft Purview Unified Audit Log (retention: 90 days for Business Standard, 1 year for E3/E5). Audit log review for P&C incidents is described in `EIT-Access-Control-Guide.md`.

### 7.4 P&C Designation

The legal definition, application rules, and operational procedures for the `PrivilegedAndConfidential` flag are fully documented in `EIT-Access-Control-Guide.md`. Key principles:
- P&C is a legal designation — not a sensitivity classification.
- Misuse of P&C designation can expose the organization to adverse legal inference.
- Only domain leads, legal counsel, or EIT administrators may set or clear the P&C flag.

---

## 8. Integration Points

### 8.1 Phase 1 Integrations

| Integration | Type | Description |
|-------------|------|-------------|
| SharePoint ↔ Power Apps | Native | Power Apps reads/writes SharePoint lists via connector |
| SharePoint ↔ Power Automate | Native | Flows read/write SharePoint lists via connector |
| Power Apps → Power Automate | Native | App calls flows via `.Run()` method |
| Outlook → Recipients | Native | Power Automate sends escalation emails via Outlook connector |
| Entra ID → Power Apps | Native | `Office365Groups.IsMemberOfGroup()` checks PAC group membership on app start |

### 8.2 Phase 2 Integration Points (Planned)

See `EIT-Phase2-Integration-Roadmap.md` for full architecture. In summary:
- ServiceNow → EIT_Incidents (via REST API + Power Automate HTTP connector)
- iSight → EIT_Incidents (via API polling)
- EIT_Incidents → Power BI (via SharePoint connector)
- EIT escalation flow → Teams channel (adaptive card notification)

---

## 9. Non-Functional Requirements

### 9.1 Performance

| Requirement | Target |
|-------------|--------|
| App load time | < 5 seconds on standard M365 tenant |
| Gallery render (100 incidents) | < 3 seconds |
| Patch (save update) response | < 2 seconds |
| Flow execution (Intake Promotion) | < 10 seconds end-to-end |
| Flow execution (Escalation Notifier) | < 15 seconds including email delivery |
| Daily staleness flow (200 incidents) | < 5 minutes total run time |

**Delegation:** Power Apps Filter and SortByColumns are delegable for SharePoint. At Phase 1 scale (< 500 active incidents), delegation limits are not a concern. See `EIT-PowerApps-Completion-Guide.md` Appendix B for delegation considerations.

### 9.2 Availability

The EIT depends entirely on M365 service availability. No additional infrastructure is required. Service Level Objectives are governed by Microsoft's M365 SLA (99.9% uptime for SharePoint Online, Power Apps, and Power Automate).

### 9.3 Scalability

| Limit | Threshold | Action Required |
|-------|-----------|----------------|
| SharePoint list items | 30M items total | Not a concern for Phase 1 |
| SharePoint list view threshold | 5,000 items in a single view | If EIT_Incidents exceeds 5,000 rows, configure indexed columns (Domain, Status) |
| Power Apps delegation | 500 rows default | Increase data row limit to 2,000 in Power Apps Settings if needed |
| Daily flow execution | 200 incidents | Negligible processing time; scales to thousands before concern |

### 9.4 Data Retention

| Data | Retention Policy | Notes |
|------|-----------------|-------|
| EIT_Incidents (Closed) | Indefinite in Phase 1 | Phase 2 archiving flow will handle 12-month cleanup |
| EIT_IncidentHistory | Indefinite (append-only) | Do not delete |
| EIT_IntakeQueue | Indefinite (TriageStatus filters active view) | Promoted/Dismissed/Rejected items remain for record-keeping |
| Audit Log (Purview) | 90 days (Business Std) / 1 year (E3/E5) | Configure retention policy for regulatory compliance |

---

## 10. Deployment & Environments

### 10.1 Environment

| Component | Environment |
|-----------|------------|
| SharePoint Site | `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker` |
| Power Apps | Default environment (or dedicated EIT environment if available) |
| Power Automate | Default environment |

### 10.2 Deployment Order

1. Fill in `06-Environment-Config/EIT-ENVIRONMENT-VARIABLES.md`
2. Create M365 groups: EIT Members, EIT Viewers, EIT Owners, EIT-PAC-Authorized
3. Run `EIT-find-and-replace-checklist.md` to substitute placeholders
4. Create SharePoint site → 4 lists → configure permissions → load sample data
5. Configure `EIT_DomainContacts` rows with real contact data
6. Build 3 Power Automate flows; test each individually
7. Build Power Apps canvas app; connect all data sources and flows; set `App.OnStart`
8. Test end-to-end lifecycle (see `EIT-Test-Plan.md`)
9. Share app with `EIT Members` group; optionally add `EIT Viewers`
10. Publish and optionally pin to Microsoft Teams

### 10.3 Version Control

The EIT does not have a traditional codebase. Documentation is managed in this repository. Track changes to formulas and flow expressions in git commit messages when modifying guide files.

---

## 11. Known Limitations & Future Roadmap

### 11.1 Known Phase 1 Limitations

| Limitation | Impact | Mitigation |
|-----------|--------|-----------|
| Power Apps is a UX layer only — does not enforce SharePoint permissions | P&C incidents require manual Layer 2 permission setup for each row | Documented procedure in EIT-Access-Control-Guide.md; EIT administrator responsible |
| ExecutiveReportScreen uses AddColumns — not delegable | Report accuracy limited to top 500 incidents per domain | At Phase 1 scale, acceptable; Phase 2 moves to Power BI with server-side aggregation |
| Daily flow must be manually monitored for failures | Flow failure means DaysSinceUpdate is not updated | Configure flow run alert in Power Automate; domain leads will notice stale data |
| No bilingual (EN/FR) support | App and notifications are English only | Phase 2 enhancement |
| Manual P&C access restriction process | Risk of human error in access setup | Documented checklist in EIT-Access-Control-Guide.md; EIT administrator must follow rigorously |
| No automated notification for new intake submissions | Domain leads must check the dashboard regularly | Phase 2: add Power Automate flow triggered on new IntakeQueue item |

### 11.2 Future Roadmap

See `EIT-Phase2-Integration-Roadmap.md` for full details on:
- ServiceNow → EIT automated ingest
- iSight → EIT automated ingest
- Power BI executive dashboard
- Teams channel notifications
- Automated incident archiving (12-month cycle)

---

## 12. Appendices

### Appendix A: File Inventory

| Directory | File | Purpose |
|-----------|------|---------|
| `01-SharePoint/` | `EIT-CREATE-SITE.md` | Step-by-step SharePoint setup |
| `01-SharePoint/` | `EIT_Incidents-schema.json` | Column specifications |
| `01-SharePoint/` | `EIT_IncidentHistory-schema.json` | Column specifications |
| `01-SharePoint/` | `EIT_IntakeQueue-schema.json` | Column specifications |
| `01-SharePoint/` | `EIT_DomainContacts-schema.json` | Column specifications + 5 static rows |
| `01-SharePoint/sample-data/` | `EIT_Incidents-sample.csv` | 15 sample records |
| `01-SharePoint/sample-data/` | `EIT_IncidentHistory-sample.csv` | 38 history entries |
| `01-SharePoint/sample-data/` | `EIT_IntakeQueue-sample.csv` | 5 pending items |
| `01-SharePoint/sample-data/` | `EIT_DomainContacts-sample.csv` | 5 placeholder rows |
| `02-PowerAutomate/` | `EIT-Daily-Staleness-Escalation-Calculator.md` | Flow rebuild guide |
| `02-PowerAutomate/` | `EIT-Intake-Promotion.md` | Flow rebuild guide |
| `02-PowerAutomate/` | `EIT-Domain-Escalation-Notifier.md` | Flow rebuild guide |
| `02-PowerAutomate/flow-expressions/` | `EIT-promotion-ItemID.txt` | Copy-paste expression |
| `02-PowerAutomate/flow-expressions/` | `EIT-escalation-Threshold.txt` | Copy-paste expression |
| `03-PowerApps/` | `EIT-REBUILD-GUIDE.md` | High-level app build guide |
| `03-PowerApps/` | `EIT-PowerApps-Completion-Guide.md` | Full screen-by-screen build guide |
| `04-Documentation/` | `EIT-SDD.md` | This document |
| `04-Documentation/` | `EIT-Test-Plan.md` | Test plan (76+ test cases) |
| `04-Documentation/` | `EIT-User-Guide.md` | End-user guide |
| `04-Documentation/` | `EIT-Access-Control-Guide.md` | P&C and access control guide |
| `04-Documentation/` | `EIT-Enterprise-Taxonomy.md` | Taxonomy definitions |
| `04-Documentation/` | `EIT-Escalation-Thresholds.md` | Escalation matrix and procedures |
| `04-Documentation/` | `EIT-Leadership-Report-Guide.md` | Executive reporting guide |
| `04-Documentation/` | `EIT-Phase2-Integration-Roadmap.md` | Phase 2 integration plan |
| `06-Environment-Config/` | `EIT-ENVIRONMENT-VARIABLES.md` | Deployment environment variables |
| `06-Environment-Config/` | `EIT-find-and-replace-checklist.md` | Placeholder replacement checklist |

### Appendix B: Domain Contacts Data Model

```json
{
  "Title": "National Security",
  "LeadName": "[NS-LEAD-NAME]",
  "LeadEmail": "[NS-LEAD-EMAIL]",
  "EscalationEmail": "[EXEC-ESCALATION-EMAIL]",
  "ActiveIncidentCount": 0
}
```
Repeat for: NIS, Privacy & Technology, Human Capital, Physical Security.

### Appendix C: Escalation Expression Reference

**ItemID generation (Intake Promotion flow):**
```
concat('EIT-', string(outputs('Get_item')?['body/ID']))
```

**EscalationThreshold calculation (Daily Staleness flow):**
```
if(and(greater(div(sub(ticks(utcNow()),ticks(items('Apply_to_each')?['LastUpdated'])),864000000000),14),equals(items('Apply_to_each')?['Severity']?['Value'],'Critical')),'Executive Brief',if(and(greater(div(sub(ticks(utcNow()),ticks(items('Apply_to_each')?['LastUpdated'])),864000000000),7),equals(items('Apply_to_each')?['Severity']?['Value'],'High')),'Escalate',items('Apply_to_each')?['EscalationThreshold']?['Value']))
```

**DaysSinceUpdate calculation (Daily Staleness flow):**
```
div(sub(ticks(utcNow()),ticks(items('Apply_to_each')?['LastUpdated'])),864000000000)
```
*(864,000,000,000 ticks = 1 day in 100-nanosecond intervals)*
