# OFR Issue Tracker — M365-Native Build Completion Guide

## Build Summary

The One Firm Risk (OFR) Issue Tracker has been rebuilt as a fully M365-native solution using SharePoint Online, Power Apps, and Power Automate. Zero custom code, zero Azure infrastructure.

---

## What Was Built

### SharePoint Site & Lists (Data Layer)

**Site:** https://[TENANT].sharepoint.com/sites/OFRIssueTracker

| List | Purpose | Columns |
|------|---------|---------|
| OFR_Issues | Main issue tracker | Title, ItemID, Owner, Priority (High/Medium/Low), Status (New/Active/Monitoring/Escalated/Closed), DateRaised, LastUpdated, NextAction, DaysSinceUpdate, StalenessFlag (Current/Aging/Stale), FunctionalGroup (10 groups) |
| OFR_UpdateHistory | Audit trail for updates | Title, ParentItemID, UpdateDate, StatusAtUpdate, Notes, UpdatedBy |
| OFR_IntakeQueue | Triage queue for new issues | Title, Owner, Priority, Description, DateSubmitted, TriageStatus (Pending/Promoted/Dismissed/Accepted/Rejected), FunctionalGroup (10 groups) |

### Power Apps Canvas App (UI Layer)

**App ID:** `0fbbc26c-ad71-476a-bcfc-edc0d7989533`

**6 Screens + 2 Side Panels:**

1. **Dashboard Screen**
   - KPI summary cards: Open Items, Stale, High Priority, Medium, Low
   - Intake Queue gallery with Promote/Dismiss buttons
   - **Intake Review panel** — click a pending item to open a right-side panel showing full details with Accept/Reject buttons and owner assignment
   - "+ Submit New Issue" button → navigates to SubmitScreen
   - Navigation to Tracker Screen, Group Allocation Screen, and Kanban Board Screen

2. **Tracker Screen**
   - Sortable/filterable issue gallery (table layout)
   - Filter toggles: All Open, Stale, High, Medium, Low
   - Search box (Title, Owner, ItemID, FunctionalGroup)
   - FunctionalGroup column showing group allocation per issue
   - Staleness color-coding: Primary Blue (0-7 days), Orange Lighter (8-14), Primary Red (15+) — Appkit4 palette
   - **Quick-update side panel** — click an issue to open a right-side panel for rapid note entry and status change
   - Row tap → Issue Detail Screen (via "View Full Detail" button in panel)

3. **Issue Detail Screen**
   - Full issue header with all metadata including FunctionalGroup
   - Update history timeline (sorted descending)
   - Add Update form with notes + status change
   - Auto-patches OFR_Issues.LastUpdated and DaysSinceUpdate on save

4. **Submit Screen**
   - Dedicated form for submitting new issues to the Intake Queue
   - Fields: Title, Priority (dropdown), FunctionalGroup (dropdown — 10 groups), Description
   - Submits with TriageStatus = "Pending" and DateSubmitted = Now()
   - Back button returns to Dashboard

5. **Group Allocation Screen**
   - 10 group cards in a 2×5 grid showing active issue counts per functional group
   - Groups: Risk Management Office, Engagement Risk, Client Risk and KYC, Technology Risk & AI Trust, National Security, OGC General Counsel, OGC Privacy, OGC Contracts, Internal Audit, Independence
   - Total active count and unassigned count (orange warning)
   - Back button returns to Dashboard

6. **Kanban Board Screen**
   - 4 vertical swim-lane galleries by status: New (blue), Active (orange), Escalated (red), Monitoring (light orange)
   - Issue cards showing ItemID, Priority badge, Title, Owner, FunctionalGroup, and staleness indicator
   - Left-edge accent stripe on each card colour-coded by staleness
   - Card tap → navigates to Issue Detail Screen
   - Back button returns to Dashboard

### Power Automate Flows (Automation Layer)

| Flow | ID | Trigger | Purpose |
|------|----|---------|---------|
| OFR Daily Staleness Calculator | `aefb8de0-35fe-4d5d-a629-ddd8502ee5aa` | Recurrence (daily 6 AM) | Calculates DaysSinceUpdate and sets StalenessFlag for all open issues |
| OFR Intake Promotion | `1c631640-113f-4602-805e-1d693582de8c` | Power Apps V2 trigger | Promotes intake item → creates OFR_Issues entry + UpdateHistory audit trail → marks intake as Promoted → returns new ItemID |

### Sample Data Loaded

- **8 issues** in OFR_Issues (OFR-1 through OFR-8) with varied priorities, statuses, and staleness levels
- **17 update history entries** across all issues
- **2 intake queue items** (Pending) ready for triage testing

---

## Flow Details

### Flow 1: OFR Daily Staleness Calculator

```
Trigger: Recurrence → Every 1 Day at 06:00 AM
  ↓
Get items: OFR_Issues where Status ≠ 'Closed'
  ↓
Apply to each:
  ↓
  Update item:
    DaysSinceUpdate = div(sub(ticks(utcNow()), ticks(LastUpdated)), 864000000000)
    StalenessFlag = if(days ≤ 7, 'Current', if(days ≤ 14, 'Aging', 'Stale'))
```

### Flow 2: OFR Intake Promotion

```
Trigger: Power Apps V2 → IntakeItemID (Number)
  ↓
Get item: OFR_IntakeQueue by IntakeItemID
  ↓
Create item: OFR_Issues
  - Title, Owner, Priority from intake item
  - ItemID = concat('OFR-', string(ID))
  - Status = 'New', DateRaised = utcNow(), LastUpdated = utcNow()
  - DaysSinceUpdate = 0, StalenessFlag = 'Current'
  - NextAction = Description from intake
  - FunctionalGroup = FunctionalGroup from intake
  ↓
Create item: OFR_UpdateHistory
  - ParentItemID = new ItemID
  - StatusAtUpdate = 'New'
  - Notes = 'Promoted from intake queue'
  - UpdatedBy = Owner from intake
  ↓
Update item: OFR_IntakeQueue
  - TriageStatus = 'Promoted'
  ↓
Respond to Power App: NewItemID
```

---

## Testing Checklist

### Basic Verification
- [ ] Open the Power Apps canvas app
- [ ] Dashboard shows correct KPI counts (8 open, 2 stale, 4 high priority)
- [ ] Intake queue shows 2 pending items

### Issue Lifecycle Test
- [ ] Promote an intake item → verify it appears in OFR_Issues with correct data
- [ ] Navigate to Tracker screen → verify all 8+ issues display
- [ ] Filter by "Stale" → should show OFR-3 and OFR-8
- [ ] Filter by "High" → should show OFR-1, OFR-2, OFR-4, OFR-8
- [ ] Search for "Sarah" → should show OFR-1 and OFR-5
- [ ] Tap an issue → verify detail screen shows correct header + update history
- [ ] Add an update with notes → verify it appears in timeline
- [ ] Change status during update → verify status badge updates

### Staleness Flow Test
- [ ] Manually trigger or wait for daily staleness flow
- [ ] Verify DaysSinceUpdate recalculates correctly
- [ ] Verify StalenessFlag transitions (Current → Aging → Stale)

### Intake Test
- [ ] Submit a new issue via SubmitScreen
- [ ] Verify it appears in intake queue as "Pending"
- [ ] Click the pending item → Intake Review panel opens with full details
- [ ] Enter an owner and click "Accept into Tracker" → verify new issue in OFR_Issues, intake marked "Accepted"
- [ ] Submit another item → click it → click "Reject" → verify intake marked "Rejected"
- [ ] Promote an intake item via Promote button → verify flow runs and issue appears in tracker
- [ ] Dismiss an intake item → verify TriageStatus changes to "Dismissed"

---

## Next Steps (Deferred Features)

| Feature | Approach | Effort |
|---------|----------|--------|
| Bilingual EN/FR | Power Apps variables + Switch() for labels | Medium |
| CSV Export | SharePoint list "Export to Excel" or Power Automate file creation | Low |
| Email Notifications | Power Automate flow for stale items → Office 365 Outlook send | Low |
| Teams Integration | Pin Power App as tab in Teams channel | Low |

---

## Key URLs

| Resource | URL |
|----------|-----|
| SharePoint Site | https://[TENANT].sharepoint.com/sites/OFRIssueTracker |
| OFR_Issues List | https://[TENANT].sharepoint.com/sites/OFRIssueTracker/Lists/OFR_Issues |
| OFR_UpdateHistory List | https://[TENANT].sharepoint.com/sites/OFRIssueTracker/Lists/OFR_UpdateHistory |
| OFR_IntakeQueue List | https://[TENANT].sharepoint.com/sites/OFRIssueTracker/Lists/OFR_IntakeQueue |
| Power Apps Studio | https://make.powerapps.com |
| Power Automate | https://make.powerautomate.com |

---

## Architecture Advantage

**Zero infrastructure.** No Azure subscriptions, no GitHub repos, no CI/CD pipelines, no custom domains, no SSL certificates. Everything runs within M365. Users authenticate automatically via Entra ID. Data stays in SharePoint. The app lives in Power Apps and can be pinned to Teams. IT has nothing new to manage.
