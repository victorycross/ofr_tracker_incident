# Enterprise Incident Tracker — Test Plan

**Version:** 1.0
**Date:** February 2026
**Application:** Enterprise Incident Tracker (M365-Native)
**Environment:** brightpathtechnology.io — M365 Business Standard + Power Automate Standard

---

## 1. Test Objectives

Validate that the EIT correctly supports the full incident lifecycle: intake submission, triage, promotion to active tracking, incident updates with audit trail, automated staleness calculation, escalation threshold automation, escalation notification emails, P&C designation and access restriction, dashboard KPIs, domain filtering, and all reporting views.

Confirm all three layers (SharePoint data, Power Apps UI, Power Automate automation) work together end-to-end. Confirm P&C access controls function correctly at both the SharePoint item-level and the Power Apps UI level.

---

## 2. Test Environment

| Component | Details |
|-----------|---------|
| SharePoint Site | `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker` |
| Power Apps App | Enterprise Incident Tracker (record App ID after creation) |
| Power Automate | EIT Daily Staleness & Escalation Calculator, EIT Intake Promotion, EIT Domain Escalation Notifier |
| Test User (Admin) | `david@brightpathtechnology.io` — member of EIT Members and EIT Owners |
| Test User (Viewer) | A second account in EIT Viewers only — for access control tests |
| Test User (Non-Member) | A third account not in any EIT group — for negative access tests |
| Browser | Microsoft Edge or Google Chrome (latest) |

### Pre-loaded Sample Data

| List | Records |
|------|---------|
| EIT_Incidents | 15 items (EIT-001 to EIT-015) — varied domains, severities, statuses, staleness, P&C flags |
| EIT_IncidentHistory | 38 entries across 15 incidents |
| EIT_IntakeQueue | 5 pending items (one per domain) |
| EIT_DomainContacts | 5 rows with real email addresses for at least one domain (for flow testing) |

---

## 3. Test Categories

### 3.1 — SharePoint Data Layer

#### TC-SP-01: EIT_Incidents list schema
| Field | Value |
|-------|-------|
| **Precondition** | Navigate to EIT_Incidents list in SharePoint |
| **Steps** | 1. Open the list 2. Verify 20 columns exist with correct types 3. Verify choice values for Domain (5 options), Severity (5), Status (6, incl. Contained), EscalationThreshold (4), ConfidentialityLevel (3), IncidentType (7), SourceSystem (4) 4. Verify defaults: Severity=Moderate, SourceSystem=Power Apps, EscalationThreshold=Not Escalated, ConfidentialityLevel=Internal, PrivilegedAndConfidential=No |
| **Expected** | All 20 columns present. Choice fields have correct options. Defaults verified. |
| **Priority** | High |

#### TC-SP-02: EIT_IncidentHistory list schema
| Field | Value |
|-------|-------|
| **Steps** | 1. Open EIT_IncidentHistory 2. Verify 9 columns 3. Verify StatusAtUpdate choices include Contained 4. Verify UpdateType choices include all 5 values with default Status Change 5. Verify PrivilegedAndConfidential column exists, defaults to No |
| **Expected** | All 9 columns present with correct types and defaults. |
| **Priority** | High |

#### TC-SP-03: EIT_IntakeQueue list schema
| Field | Value |
|-------|-------|
| **Steps** | 1. Open EIT_IntakeQueue 2. Verify 10 columns 3. Verify TriageStatus has all 5 choices, defaults to Pending 4. Verify Domain choices match EIT_Incidents.Domain exactly 5. Verify PrivilegedAndConfidential defaults to No |
| **Expected** | All 10 columns present. TriageStatus has 5 choices, defaults to Pending. |
| **Priority** | High |

#### TC-SP-04: EIT_DomainContacts setup
| Field | Value |
|-------|-------|
| **Steps** | 1. Open EIT_DomainContacts 2. Verify exactly 5 rows 3. Verify Title values match Domain choices exactly: "National Security", "NIS", "Privacy & Technology", "Human Capital", "Physical Security" 4. Verify LeadEmail and EscalationEmail are filled in for at least one domain |
| **Expected** | 5 rows with correct Title values. At least one domain has valid email addresses. |
| **Priority** | High |

#### TC-SP-05: Sample data integrity
| Field | Value |
|-------|-------|
| **Steps** | 1. Open EIT_Incidents — verify 15 records, 3 per domain 2. Open EIT_IncidentHistory — verify 38 entries 3. Open EIT_IntakeQueue — verify 5 pending items 4. Spot-check: EIT-001 (NS/Critical/Active), EIT-005 (HC/Stale), EIT-004 (HC/P&C=Yes) |
| **Expected** | All records present with correct field values matching sample data specification. |
| **Priority** | High |

---

### 3.2 — Power Apps: DashboardScreen

#### TC-DA-01: KPI cards — Open Incidents
| Field | Value |
|-------|-------|
| **Steps** | 1. Open the app 2. Count incidents in EIT_Incidents where Status ≠ Closed and Status ≠ Contained 3. Verify `lblOpenCount` matches |
| **Expected** | Count matches SharePoint filtered count. |
| **Priority** | High |

#### TC-DA-02: KPI cards — Critical
| Field | Value |
|-------|-------|
| **Steps** | Count incidents where Severity=Critical, Status≠Closed, Status≠Contained. Verify `lblCritCount`. |
| **Expected** | Count matches. Label displays in dark red. |
| **Priority** | High |

#### TC-DA-03: KPI cards — Stale
| Field | Value |
|-------|-------|
| **Steps** | Count incidents where StalenessFlag=Stale, Status≠Closed, Status≠Contained. Verify `lblStaleCount`. |
| **Expected** | Count matches. Label displays in red. |
| **Priority** | High |

#### TC-DA-04: KPI cards — Escalated
| Field | Value |
|-------|-------|
| **Steps** | Count incidents where EscalationThreshold≠"Not Escalated", Status≠Closed, Status≠Contained. Verify `lblEscCount`. |
| **Expected** | Count matches. |
| **Priority** | High |

#### TC-DA-05: KPI cards — P&C Active
| Field | Value |
|-------|-------|
| **Steps** | Count incidents where PrivilegedAndConfidential=Yes, Status≠Closed. Verify `lblPACCount`. |
| **Expected** | Count matches (for users with PAC access). |
| **Priority** | High |

#### TC-DA-06: Intake queue shows pending items only
| Field | Value |
|-------|-------|
| **Steps** | 1. Verify galIntakeQueue shows only items with TriageStatus=Pending 2. Promote one item and verify it disappears from queue |
| **Expected** | Queue shows pending items only. Disappears on promotion. |
| **Priority** | High |

#### TC-DA-07: Intake queue — Priority sort and P&C badge
| Field | Value |
|-------|-------|
| **Steps** | 1. Verify items sorted by priority (High first) then date 2. Verify P&C intake item shows red "P&C" badge 3. Verify non-P&C item shows no badge |
| **Expected** | Sort order correct. P&C badge visible on P&C=Yes items only. |
| **Priority** | Medium |

#### TC-DA-08: "Move to Tracker" (Promote) button — EIT Intake Promotion flow
| Field | Value |
|-------|-------|
| **Steps** | 1. Click "Move to Tracker" on a Pending intake item 2. Observe notification message 3. Verify new EIT-NNN item appears in EIT_Incidents 4. Verify EIT_IncidentHistory has opening entry with ParentItemID = new ItemID 5. Verify intake item TriageStatus = Promoted 6. Verify intake item no longer in queue |
| **Expected** | New incident created. History entry created. Intake item promoted. EIT-NNN format correct. Severity=Moderate, Status=New, EscalationThreshold=Not Escalated. |
| **Priority** | High |

#### TC-DA-09: Intake Review Panel — opens and closes
| Field | Value |
|-------|-------|
| **Steps** | 1. Click a pending intake item 2. Verify panel opens on right side 3. Verify title, description, domain, incident type, priority, date, P&C flag displayed correctly 4. Click X to close 5. Verify panel closes |
| **Expected** | Panel opens with correct data. Closes on X click. |
| **Priority** | High |

#### TC-DA-10: Submit New Incident overlay
| Field | Value |
|-------|-------|
| **Steps** | 1. Click "+ New Incident" 2. Verify overlay appears 3. Attempt submit with empty title — verify button disabled 4. Fill in all required fields (Title, Domain, IncidentType) 5. Toggle P&C on 6. Click Submit 7. Verify overlay closes 8. Verify new item appears in EIT_IntakeQueue with TriageStatus=Pending and PrivilegedAndConfidential=Yes |
| **Expected** | Overlay opens. Validation prevents blank submit. Successful submit creates intake item. P&C value persisted. |
| **Priority** | High |

#### TC-DA-11: Navigation buttons
| Field | Value |
|-------|-------|
| **Steps** | Click each navigation button: Tracker, By Domain, Kanban, Exec Report. Verify each navigates to correct screen. |
| **Expected** | All navigation buttons reach the correct screen. |
| **Priority** | Medium |

---

### 3.3 — Power Apps: TrackerScreen

#### TC-TR-01: Domain filter chips
| Field | Value |
|-------|-------|
| **Steps** | 1. Click each domain filter chip (NS, NIS, PT, HC, PS) 2. Verify gallery shows only incidents of that domain 3. Click "All Open" — verify all non-Closed, non-Contained incidents shown 4. Click "Critical" — verify only Severity=Critical shown |
| **Expected** | Each chip filters correctly. All Open resets to full non-closed list. Active chip shows as filled/white. |
| **Priority** | High |

#### TC-TR-02: Show Contained toggle
| Field | Value |
|-------|-------|
| **Steps** | 1. Default state — verify Contained incidents not shown 2. Toggle "Incl. Contained" on 3. Verify Contained incidents appear in gallery |
| **Expected** | Toggle correctly adds/removes Contained incidents from view. |
| **Priority** | Medium |

#### TC-TR-03: Search box
| Field | Value |
|-------|-------|
| **Steps** | 1. Type an incident title fragment — verify matching results 2. Type an ItemID (e.g., "EIT-003") — verify that item shown 3. Type a domain name — verify matching results 4. Clear search — verify all results return |
| **Expected** | Search filters by Title, Owner, ItemID, and Domain. Clears correctly. |
| **Priority** | High |

#### TC-TR-04: Severity badge colors (5-tier)
| Field | Value |
|-------|-------|
| **Steps** | Verify each severity badge in the gallery displays the correct background color: Critical=dark red, High=red, Moderate=orange, Low=blue, Informational=grey |
| **Expected** | All 5 severity colors correct. |
| **Priority** | Medium |

#### TC-TR-05: Status badge — Contained
| Field | Value |
|-------|-------|
| **Steps** | Enable "Incl. Contained" toggle. Verify Contained incidents show a green status badge. |
| **Expected** | Contained status badge is green `RGBA(50,140,80,1)`. |
| **Priority** | Medium |

#### TC-TR-06: P&C left-stripe indicator
| Field | Value |
|-------|-------|
| **Steps** | 1. Verify P&C incidents show a dark red left-stripe on their row 2. Verify non-P&C incidents have no stripe |
| **Expected** | P&C stripe visible only on P&C=Yes incidents. |
| **Priority** | Medium |

#### TC-TR-07: P&C visibility gate
| Field | Value |
|-------|-------|
| **Precondition** | Test with a user NOT in EIT-PAC-Authorized group |
| **Steps** | 1. Log in as non-PAC user 2. Open Tracker screen 3. Verify P&C incidents are not visible in gallery |
| **Expected** | P&C incidents hidden from non-authorized users in the app UI. |
| **Priority** | High |

#### TC-TR-08: Row click → IssueDetailScreen
| Field | Value |
|-------|-------|
| **Steps** | Click any incident row. Verify navigation to IssueDetailScreen with correct incident data populated. |
| **Expected** | IssueDetailScreen opens with correct incident header values. |
| **Priority** | High |

#### TC-TR-09: Result count label
| Field | Value |
|-------|-------|
| **Steps** | Verify the results count label updates correctly when filters are applied. |
| **Expected** | Count reflects filtered gallery item count. |
| **Priority** | Low |

---

### 3.4 — Power Apps: IssueDetailScreen

#### TC-ID-01: P&C warning banner
| Field | Value |
|-------|-------|
| **Steps** | 1. Navigate to a P&C incident — verify amber warning banner appears at top 2. Navigate to a non-P&C incident — verify banner is not visible |
| **Expected** | Banner visible only on P&C=Yes incidents. Contains required warning text. |
| **Priority** | High |

#### TC-ID-02: Incident header — all fields
| Field | Value |
|-------|-------|
| **Steps** | Verify header card shows: ItemID, Title, Owner, Domain, IncidentType, Severity badge (correct color), Status badge (correct color), Escalation badge, DateRaised, LastUpdated, Days Stale (correct staleness color), ConfidentialityLevel, NextAction |
| **Expected** | All fields present and displaying correct values from EIT_Incidents. Badges show correct colors. |
| **Priority** | High |

#### TC-ID-03: Update history gallery — sort order and content
| Field | Value |
|-------|-------|
| **Steps** | 1. Verify history entries show most recent first 2. Verify each entry shows: date, UpdateType badge, StatusAtUpdate badge, SeverityAtUpdate badge, UpdatedBy, Notes 3. Verify P&C history entries show P&C badge and italic notes |
| **Expected** | Descending date order. All fields displayed. P&C entries visually distinguished. |
| **Priority** | High |

#### TC-ID-04: Save Update — Status Change
| Field | Value |
|-------|-------|
| **Steps** | 1. Set Status=Active, Severity=High, UpdateType=Status Change 2. Enter notes 3. Click Save Update 4. Verify: new history entry in galUpdateHistory with correct StatusAtUpdate, SeverityAtUpdate, UpdateType, Notes, UpdatedBy 5. Verify EIT_Incidents header updates: new Status and Severity badges, DaysSinceUpdate=0 (StalenessFlag=Current) |
| **Expected** | History entry created. Incident header refreshed. Staleness reset to Current. |
| **Priority** | High |

#### TC-ID-05: Save Update — disabled when Notes empty
| Field | Value |
|-------|-------|
| **Steps** | Leave Notes field blank. Verify Save Update button is disabled (greyed out). |
| **Expected** | Button disabled. Cannot save without notes. |
| **Priority** | Medium |

#### TC-ID-06: Save Update — Escalation (triggers flow)
| Field | Value |
|-------|-------|
| **Precondition** | EIT_DomainContacts has real email for the test incident's domain |
| **Steps** | 1. Set UpdateType=Escalation 2. Verify EscalationThreshold dropdown appears 3. Set EscalationThreshold=Escalate 4. Add notes 5. Click Save Update 6. Verify: EIT Domain Escalation Notifier flow triggered 7. Verify escalation email received at domain lead email 8. Verify EscalationThreshold on incident updated to Escalate 9. Verify history entry with UpdateType=Escalation |
| **Expected** | Flow triggered. Email delivered with correct subject and body. EscalationThreshold updated. History entry created. |
| **Priority** | High |

#### TC-ID-07: Save Update — Containment (sets DateContained)
| Field | Value |
|-------|-------|
| **Steps** | 1. Set Status=Contained, UpdateType=Containment 2. Add notes 3. Click Save Update 4. Verify DateContained is set on EIT_Incidents row to today's date |
| **Expected** | DateContained populated. Status badge green (Contained). |
| **Priority** | High |

#### TC-ID-08: Save Update — EscalationThreshold dropdown only visible for Escalation
| Field | Value |
|-------|-------|
| **Steps** | Set UpdateType to each value: Status Change, Evidence Added, Containment, Closure. Verify EscalationThreshold dropdown is hidden for all. Then set UpdateType=Escalation — verify dropdown appears. |
| **Expected** | Dropdown visible only when UpdateType=Escalation. |
| **Priority** | Medium |

---

### 3.5 — Power Apps: DomainOverviewScreen

#### TC-DO-01: Domain card counts
| Field | Value |
|-------|-------|
| **Steps** | For each of the 5 domain cards, manually count matching incidents in EIT_Incidents (Status≠Closed, Status≠Contained). Compare against displayed count. |
| **Expected** | All 5 domain card counts correct. |
| **Priority** | High |

#### TC-DO-02: Domain card — Critical sub-count
| Field | Value |
|-------|-------|
| **Steps** | Verify each domain card's Critical sub-count label matches actual Critical incidents in that domain. |
| **Expected** | Critical counts correct per domain. |
| **Priority** | Medium |

#### TC-DO-03: Domain card click → Tracker filtered
| Field | Value |
|-------|-------|
| **Steps** | Click each domain card. Verify TrackerScreen opens with the domain filter chip pre-selected for that domain. |
| **Expected** | TrackerScreen opens filtered to clicked domain. |
| **Priority** | Medium |

---

### 3.6 — Power Apps: KanbanScreen

#### TC-KA-01: Five swim-lanes visible
| Field | Value |
|-------|-------|
| **Steps** | Verify five column headers appear: New, Active, Escalated, Monitoring, Contained. |
| **Expected** | All 5 swim-lanes visible. Column headers display correct count for each status. |
| **Priority** | High |

#### TC-KA-02: Kanban card content
| Field | Value |
|-------|-------|
| **Steps** | Verify each card shows: ItemID, Severity badge, Title (truncated), Domain, Owner, Days count |
| **Expected** | All card fields present and correct. |
| **Priority** | Medium |

#### TC-KA-03: Staleness stripe colors
| Field | Value |
|-------|-------|
| **Steps** | Verify staleness stripe on left of each card: blue (0-7d), orange (8-14d), red (15+d) |
| **Expected** | Three stripe colors match staleness thresholds. |
| **Priority** | Medium |

#### TC-KA-04: P&C right-stripe
| Field | Value |
|-------|-------|
| **Steps** | Verify P&C incidents show dark red stripe on right edge of card. Non-P&C show transparent right edge. |
| **Expected** | P&C stripe visible only on P&C=Yes cards. |
| **Priority** | Medium |

#### TC-KA-05: Card click → IssueDetailScreen
| Field | Value |
|-------|-------|
| **Steps** | Click a card in each swim-lane. Verify navigation to IssueDetailScreen with correct incident. |
| **Expected** | All swim-lane cards navigate correctly. |
| **Priority** | High |

---

### 3.7 — Power Apps: ExecutiveReportScreen

#### TC-EX-01: Domain summary table accuracy
| Field | Value |
|-------|-------|
| **Steps** | For each domain row, manually calculate expected counts from SharePoint (Open, Critical, Escalated, P&C, Stale). Compare against displayed values. |
| **Expected** | All values in the table match manually calculated counts. |
| **Priority** | High |

#### TC-EX-02: Zero counts display in grey
| Field | Value |
|-------|-------|
| **Steps** | Verify that any column showing 0 displays in grey color, not red. |
| **Expected** | Zero counts are grey. Non-zero red/dark-red values attract attention. |
| **Priority** | Low |

#### TC-EX-03: Enterprise totals match domain sum
| Field | Value |
|-------|-------|
| **Steps** | Manually sum each column across all 5 domain rows. Verify enterprise totals match. |
| **Expected** | Enterprise totals are accurate sums of domain rows. |
| **Priority** | Medium |

#### TC-EX-04: Report date label
| Field | Value |
|-------|-------|
| **Steps** | Verify "Report as of: [date]" label shows today's date. |
| **Expected** | Date matches current date. |
| **Priority** | Low |

---

### 3.8 — Power Automate: Daily Staleness & Escalation Calculator

#### TC-FL-01: Manual flow test — DaysSinceUpdate
| Field | Value |
|-------|-------|
| **Precondition** | At least one incident with LastUpdated set to > 7 days ago |
| **Steps** | 1. Run the flow manually (Test → Manually) 2. After completion, check EIT_Incidents for that incident 3. Verify DaysSinceUpdate matches calculated days |
| **Expected** | DaysSinceUpdate = correct integer days elapsed since LastUpdated. |
| **Priority** | High |

#### TC-FL-02: StalenessFlag calculation
| Field | Value |
|-------|-------|
| **Steps** | After flow run, verify: incidents with 0-7 DaysSinceUpdate = Current; 8-14 = Aging; 15+ = Stale. |
| **Expected** | All three StalenessFlag tiers assigned correctly. |
| **Priority** | High |

#### TC-FL-03: Auto-escalation — High severity > 7 days
| Field | Value |
|-------|-------|
| **Precondition** | An incident with Severity=High and LastUpdated > 7 days ago exists |
| **Steps** | Run flow. Verify EscalationThreshold on that incident updates to "Escalate". |
| **Expected** | EscalationThreshold = Escalate. |
| **Priority** | High |

#### TC-FL-04: Auto-escalation — Critical severity > 14 days
| Field | Value |
|-------|-------|
| **Precondition** | An incident with Severity=Critical and LastUpdated > 14 days ago exists |
| **Steps** | Run flow. Verify EscalationThreshold updates to "Executive Brief". |
| **Expected** | EscalationThreshold = Executive Brief. |
| **Priority** | High |

#### TC-FL-05: Auto-escalation does not downgrade
| Field | Value |
|-------|-------|
| **Precondition** | An incident already at EscalationThreshold=Executive Brief but now updated (DaysSinceUpdate=0) |
| **Steps** | Run flow. Verify EscalationThreshold remains Executive Brief (not reset to Not Escalated by flow). |
| **Expected** | Flow does not downgrade EscalationThreshold. Only manual action can de-escalate. |
| **Priority** | High |

#### TC-FL-06: Closed and Contained incidents excluded
| Field | Value |
|-------|-------|
| **Steps** | Verify flow does not update DaysSinceUpdate or StalenessFlag for incidents with Status=Closed or Status=Contained. |
| **Expected** | Closed and Contained incidents unchanged by flow run. |
| **Priority** | Medium |

---

### 3.9 — Power Automate: EIT Intake Promotion

#### TC-IP-01: Flow promotion — field mapping
| Field | Value |
|-------|-------|
| **Steps** | 1. Promote a pending intake item via the app "Move to Tracker" button 2. Open the new EIT_Incidents record in SharePoint 3. Verify: ItemID=EIT-NNN format; Title, Owner, Domain, IncidentType, PrivilegedAndConfidential match intake item; Severity=Moderate; Status=New; EscalationThreshold=Not Escalated; SourceSystem=Power Apps; ConfidentialityLevel=Internal; DateRaised set to today |
| **Expected** | All field mappings correct. Defaults applied for Severity, Status, EscalationThreshold, SourceSystem, ConfidentialityLevel. |
| **Priority** | High |

#### TC-IP-02: History entry creation
| Field | Value |
|-------|-------|
| **Steps** | After promotion, verify EIT_IncidentHistory has an entry with: ParentItemID = new ItemID; StatusAtUpdate=New; SeverityAtUpdate=Moderate; UpdateType=Status Change; Notes="Promoted from intake queue"; UpdatedBy = intake owner |
| **Expected** | History entry present with all correct values. |
| **Priority** | High |

#### TC-IP-03: Intake item marked Promoted
| Field | Value |
|-------|-------|
| **Steps** | After promotion, verify the original EIT_IntakeQueue item has TriageStatus=Promoted. Verify it no longer appears in galIntakeQueue. |
| **Expected** | TriageStatus=Promoted. Item removed from dashboard queue. |
| **Priority** | High |

---

### 3.10 — Power Automate: EIT Domain Escalation Notifier

#### TC-EN-01: Escalate level — email to domain lead
| Field | Value |
|-------|-------|
| **Precondition** | EIT_DomainContacts has valid LeadEmail for the test incident's domain |
| **Steps** | 1. From IssueDetailScreen, set UpdateType=Escalation, EscalationThreshold=Escalate, add notes, save 2. Check email inbox of domain lead 3. Verify email received with subject `[ESCALATION] [Title] — [Domain] — [Severity]` 4. Verify email body contains correct incident details 5. Verify EIT_Incidents.EscalationThreshold = Escalate |
| **Expected** | Email received at LeadEmail. Subject format correct. Body contains accurate incident data. Threshold updated. |
| **Priority** | High |

#### TC-EN-02: Executive Brief level — email to exec escalation address
| Field | Value |
|-------|-------|
| **Precondition** | EIT_DomainContacts has valid EscalationEmail |
| **Steps** | Repeat TC-EN-01 but set EscalationThreshold=Executive Brief. Verify email to EscalationEmail with `[EXECUTIVE BRIEF]` in subject. |
| **Expected** | Email received at EscalationEmail with correct subject prefix. |
| **Priority** | High |

#### TC-EN-03: P&C incident — email subject and disclaimer
| Field | Value |
|-------|-------|
| **Precondition** | Test incident has PrivilegedAndConfidential=Yes |
| **Steps** | Trigger escalation on a P&C incident. Verify: 1. Email subject has `[PRIVILEGED & CONFIDENTIAL]` prepended 2. Email footer contains P&C non-disclosure disclaimer |
| **Expected** | P&C prefix in subject. Disclaimer in footer. |
| **Priority** | High |

#### TC-EN-04: Flow returns Result to Power Apps
| Field | Value |
|-------|-------|
| **Steps** | After escalation save, verify no error appears in the app. Check flow run history — verify Respond action returned a Result string. |
| **Expected** | Flow completes without error. Result confirmation returned. |
| **Priority** | Medium |

---

### 3.11 — Access Control

#### TC-AC-01: EIT Viewers — read-only enforcement
| Field | Value |
|-------|-------|
| **Precondition** | Test account in EIT Viewers only (not EIT Members) |
| **Steps** | 1. Log into Power App as Viewer 2. Navigate to TrackerScreen 3. Verify incidents display (read access) 4. Navigate to IssueDetailScreen — verify Save Update is inaccessible or disabled 5. Navigate to DashboardScreen — verify "+ New Incident" button is inaccessible |
| **Expected** | Viewer can see incidents but cannot submit, update, or promote. |
| **Priority** | High |

#### TC-AC-02: P&C item-level permissions — SharePoint layer
| Field | Value |
|-------|-------|
| **Precondition** | A P&C incident has item-level permissions applied (inherited permissions stopped; EIT Members removed; only authorized individuals added) |
| **Steps** | 1. Log into SharePoint directly as a user in EIT Members but NOT on the P&C incident's authorized list 2. Navigate to the EIT_Incidents list 3. Attempt to open the P&C incident row 4. Verify access denied |
| **Expected** | SharePoint denies access. Item-level permissions are enforced independently of Power Apps. |
| **Priority** | High |

#### TC-AC-03: P&C incidents hidden in Power Apps (non-PAC user)
| Field | Value |
|-------|-------|
| **Precondition** | Test account NOT in EIT-PAC-Authorized Entra group |
| **Steps** | 1. Log into app 2. Navigate to TrackerScreen 3. Verify P&C incidents are not visible in gallery 4. Navigate to KanbanScreen — verify P&C cards not visible |
| **Expected** | P&C incidents hidden from non-PAC users across all app screens. |
| **Priority** | High |

#### TC-AC-04: P&C incidents visible to PAC-authorized user
| Field | Value |
|-------|-------|
| **Precondition** | Test account IS in EIT-PAC-Authorized Entra group AND on item-level access list for P&C incident |
| **Steps** | 1. Log into app 2. Verify P&C incidents visible in TrackerScreen (with P&C badge and dark red stripe) 3. Open a P&C incident — verify amber warning banner displayed 4. Navigate to history — verify P&C history entries show P&C badge |
| **Expected** | P&C incidents visible. Banner and badges displayed. |
| **Priority** | High |

---

### 3.12 — Cross-Screen Data Refresh

#### TC-RF-01: Save update → header refresh
| Field | Value |
|-------|-------|
| **Steps** | Save an update (status change). Verify IssueDetailScreen header immediately reflects new Status badge and DaysSinceUpdate=0 without leaving the screen. |
| **Expected** | Header updates immediately after save. |
| **Priority** | High |

#### TC-RF-02: Promote → Dashboard queue refresh
| Field | Value |
|-------|-------|
| **Steps** | Promote an intake item. Verify intake queue gallery immediately removes the promoted item without navigating away. |
| **Expected** | Queue refreshes in place. Promoted item gone. |
| **Priority** | High |

#### TC-RF-03: Submit intake → queue count update
| Field | Value |
|-------|-------|
| **Steps** | Submit a new intake item. Return to Dashboard. Verify `lblIntakeHeader` count has incremented. |
| **Expected** | Count increments on new submission. |
| **Priority** | Medium |

---

## 4. Test Execution Summary Template

| TC ID | Description | Status | Notes |
|-------|-------------|--------|-------|
| TC-SP-01 | EIT_Incidents schema | [ ] Pass / [ ] Fail | |
| TC-SP-02 | EIT_IncidentHistory schema | [ ] Pass / [ ] Fail | |
| TC-SP-03 | EIT_IntakeQueue schema | [ ] Pass / [ ] Fail | |
| TC-SP-04 | EIT_DomainContacts setup | [ ] Pass / [ ] Fail | |
| TC-SP-05 | Sample data integrity | [ ] Pass / [ ] Fail | |
| TC-DA-01 to 11 | DashboardScreen | [ ] Pass / [ ] Fail | |
| TC-TR-01 to 09 | TrackerScreen | [ ] Pass / [ ] Fail | |
| TC-ID-01 to 08 | IssueDetailScreen | [ ] Pass / [ ] Fail | |
| TC-DO-01 to 03 | DomainOverviewScreen | [ ] Pass / [ ] Fail | |
| TC-KA-01 to 05 | KanbanScreen | [ ] Pass / [ ] Fail | |
| TC-EX-01 to 04 | ExecutiveReportScreen | [ ] Pass / [ ] Fail | |
| TC-FL-01 to 06 | Daily Staleness Flow | [ ] Pass / [ ] Fail | |
| TC-IP-01 to 03 | Intake Promotion Flow | [ ] Pass / [ ] Fail | |
| TC-EN-01 to 04 | Domain Escalation Notifier | [ ] Pass / [ ] Fail | |
| TC-AC-01 to 04 | Access Control | [ ] Pass / [ ] Fail | |
| TC-RF-01 to 03 | Data Refresh | [ ] Pass / [ ] Fail | |

**Total test cases: 57**

---

## 5. Defect Logging

Log defects in the format: `[TC-ID] — [Brief Description] — [Steps to Reproduce] — [Actual vs Expected] — [Severity: Critical/High/Medium/Low]`

**Critical defects** (block go-live): any TC-SP, TC-IP, TC-FL-01/02/03/04, TC-AC-01/02/03, TC-ID-06, TC-EN-01/02/03 failures.

**High defects** (require fix before go-live): TC-DA-08, TC-TR-01, TC-ID-01/02/04/07, TC-KA-05.

**Medium/Low** (can be addressed in post-go-live sprint): cosmetic issues, count label mismatch, badge color variations.

---

## 6. Go/No-Go Criteria

| Criterion | Threshold |
|-----------|-----------|
| Critical test cases passing | 100% |
| High priority test cases passing | ≥ 95% |
| Access control tests (TC-AC-01 to 04) | 100% |
| Escalation flow tests (TC-EN-01, TC-EN-03) | 100% — no go-live with broken escalation |
| P&C item-level permission test (TC-AC-02) | 100% — no go-live with broken P&C enforcement |
