# Enterprise Incident Tracker — User Guide

**Version:** 1.0
**Last Updated:** February 2026
**Classification:** Internal

---

## Welcome

The Enterprise Incident Tracker (EIT) is your firm's central hub for tracking and managing incidents across all five enterprise domains: National Security, NIS, Privacy & Technology, Human Capital, and Physical Security.

The EIT runs entirely within Microsoft 365. You sign in with your existing M365 account — no separate password required.

This guide covers everything you need to do in the app. If you have questions about whether your access level allows a particular action, contact your EIT administrator.

---

## Getting Started

### How to Open the App

**Option A — From Power Apps:**
1. Go to [make.powerapps.com](https://make.powerapps.com)
2. Find **Enterprise Incident Tracker** in your Apps list
3. Click to open

**Option B — From Microsoft Teams (if your admin has pinned it):**
1. Open Microsoft Teams
2. Navigate to the channel where the app tab appears
3. Click the **Enterprise Incident Tracker** tab

You will be signed in automatically. The app loads your access level in the background — this includes checking whether you can view Privileged & Confidential (P&C) incidents.

### Access Levels

Your access level determines what you can see and do:

| Role | What You Can Do |
|------|----------------|
| **EIT Member** (Contribute) | Submit incidents, view all non-P&C incidents, update incident records, promote intake items |
| **EIT Viewer** (Read) | View all non-P&C incidents — read only. Cannot submit or update. |
| **Domain Lead** | All EIT Member actions plus: confirm P&C designation, set EscalationThreshold, approve access for P&C incidents |
| **EIT Administrator** | Full access including SharePoint list management and P&C item-level permission management |

If you believe your access is incorrect, contact the EIT administrator.

---

## Screen 1: Dashboard

The Dashboard is your home screen. It shows a real-time snapshot of enterprise incident status and lets you manage incoming submissions.

### KPI Summary Cards

Five cards at the top of the screen show key metrics across all domains:

| Card | What It Shows |
|------|--------------|
| **Open Incidents** | Total active incidents (Status ≠ Closed, Status ≠ Contained) |
| **Critical** | Open incidents assessed at Critical severity — highest ERM risk tier |
| **Stale** | Open incidents with no update in 15 or more days — these need urgent attention |
| **Escalated / Exec Brief** | Incidents that have been formally escalated (to domain lead or executive) |
| **Privileged & Confidential** | Active incidents with P&C designation (count reflects your access level) |

These numbers refresh each time you return to the Dashboard.

### Intake Queue

Below the KPIs you will see the **Intake Queue** — newly submitted incidents waiting for domain lead triage. Each row shows the incident title, who submitted it, the date, domain, incident type, priority badge, and a P&C badge (for P&C-flagged submissions).

**What you can do with each intake item:**

| Action | What Happens |
|--------|-------------|
| **Click the item** | Opens the Intake Review Panel on the right, showing full details. From here you can promote, accept, or reject it. |
| **Move to Tracker** | Runs the Intake Promotion flow. A new EIT-NNN incident is created in the tracker with triage defaults (Severity=Moderate, Status=New). The intake item is marked Promoted and disappears from the queue. |
| **Dismiss** | Marks the item as Dismissed and removes it from the queue. The item is retained in the SharePoint list for audit purposes. |

### Intake Review Panel

Click any pending intake item to open the review panel on the right side of the screen. The panel shows:
- Incident title, description, domain, incident type, priority, date submitted
- P&C indicator (amber, if the submitter suggested P&C designation)

From the panel, you can:
1. **Promote via Flow** — Creates the incident in the tracker (recommended — creates audit entry automatically)
2. **Accept** — Marks the intake item as Accepted (direct, no flow — use when manual processing is needed)
3. **Reject** — Marks the intake item as Rejected and closes the panel
4. **X** — Closes the panel without action

> **Note on P&C at intake:** When a P&C item is promoted, the domain lead must confirm the P&C designation and apply SharePoint item-level permissions before the incident is visible only to authorized individuals. See the P&C section below.

### Submitting a New Incident

1. Click the **+ New Incident** button on the Dashboard
2. Fill in the required fields:
   - **Title** (required): Brief descriptive title for the incident
   - **Domain** (required): Which enterprise domain owns this incident
   - **Incident Type** (required): Category of incident (Breach, Threat, Personnel, etc.)
   - **Owner**: The accountable person (defaults to your name)
   - **Priority**: Operational urgency (High/Medium/Low) — defaults to Medium
   - **Privileged & Confidential**: Toggle on if you believe this incident may involve attorney-client privilege or litigation. Domain lead will confirm at triage.
   - **Description**: Describe the incident — include what you know, when it was discovered, and what immediate action (if any) has been taken
3. Click **Submit**

The incident enters the Intake Queue as Pending. Domain leads will triage it within 24 hours.

### Navigation

From the header bar, click the navigation buttons to move between screens:
- **Tracker** — Full incident table with domain filters and search
- **By Domain** — Active incident counts per domain (overview cards)
- **Kanban** — Visual workflow board (5 status swim-lanes)
- **Exec Report** — Executive summary by domain and severity (read-only)

---

## Screen 2: Tracker

The Tracker shows all active incidents in a table. Use it to find specific incidents and drill into the detail.

### Domain Filter Chips

A row of filter chips below the header lets you narrow the incident list:

| Chip | Shows |
|------|-------|
| **All Open** | All non-Closed incidents (default) |
| **Nat'l Security** | National Security domain only |
| **NIS** | NIS domain only |
| **Privacy & Tech** | Privacy & Technology domain only |
| **Human Capital** | Human Capital domain only |
| **Physical Sec** | Physical Security domain only |
| **Critical** | Incidents with Severity = Critical (any domain) |

Only one chip is active at a time. Click a chip to activate it — it fills with color to show it's selected.

### Show Contained Toggle

Use the **Incl. Contained** toggle to include Contained incidents in the view. By default, Contained incidents are hidden (they've been stabilised) so the table focuses on active management priorities.

### Search

Type in the **search box** (top-right) to filter by:
- Incident title
- Owner name
- Item ID (e.g., "EIT-003")
- Domain name

The search applies in addition to any active domain filter chip.

### Reading the Incident Table

Each row shows:

| Column | What It Means |
|--------|--------------|
| **Item ID** | Unique EIT identifier (EIT-NNN) |
| **Title** | Incident title (hover to see full text if truncated) |
| **Owner** | Accountable person |
| **Domain** | Enterprise domain |
| **Severity** | ERM severity tier — colored badge (dark red=Critical, red=High, orange=Moderate, blue=Low, grey=Informational) |
| **Status** | Current lifecycle status — colored badge |
| **Escalation** | Current escalation level (dark red=Executive Brief, red=Escalate, orange=Watch, grey=Not Escalated) |
| **Days Stale** | Days since last update — colored: blue (0-7d, current), orange (8-14d, aging), red (15+d, stale) |
| **Next Action** | Current next steps or status summary (hover for full text) |

**P&C indicator:** A dark red stripe on the left edge of a row indicates the incident is Privileged & Confidential.

### Opening an Incident

Click any row to open the full **Issue Detail Screen** for that incident.

---

## Screen 3: Issue Detail

The Issue Detail screen is the full view of a single incident. You will use this screen for most day-to-day incident management work.

### P&C Warning Banner

If the incident is designated **Privileged & Confidential**, an amber warning banner appears at the top of the screen:

> *"⚠ PRIVILEGED & CONFIDENTIAL — This incident is restricted. Access is limited to the authorized list. Do not discuss or forward outside the authorized list."*

This banner means the incident is legally protected. Be careful about what you discuss verbally and in writing. Do not forward details to anyone not explicitly authorized.

### Incident Header

The header card shows all key incident metadata:

| Field | Description |
|-------|-------------|
| **ITEM ID** | Unique identifier |
| **TITLE** | Incident title |
| **OWNER** | Accountable person |
| **DOMAIN** | Enterprise domain |
| **INCIDENT TYPE** | Category of incident |
| **SEVERITY** | ERM severity badge |
| **STATUS** | Current status badge |
| **ESCALATION** | Escalation threshold badge |
| **DATE RAISED** | Date promoted to tracker |
| **LAST UPDATED** | Date of last update |
| **DAYS STALE** | Days since last update (color-coded) |
| **CLASSIFICATION** | Confidentiality level |
| **Next Action** | Current next steps or status summary |

### Update History Timeline

The left panel shows the full update history for this incident, most recent first. Each entry shows:
- Date and time of update
- Update type badge (Status Change, Escalation, Containment, etc.)
- Status at the time of the update
- Severity at the time of the update
- Who made the update
- Notes

P&C history entries are displayed in italic with a small P&C badge.

### Adding an Update

Use the **Add Update** form on the right side of the screen to record a new update:

1. **Update Type** — Select the category:
   - *Status Change*: The incident's status is changing
   - *Evidence Added*: New information or evidence is being documented
   - *Escalation*: You are formally escalating this incident (see below)
   - *Containment*: The incident has been contained (use with Status=Contained)
   - *Closure*: You are closing the incident (use with Status=Closed)

2. **Status** — Select the new status (or keep the current one if status is unchanged)

3. **Severity** — Select the current severity assessment (update if it has changed)

4. **Notes** (required) — Write a concise summary of: what has happened since the last update, what actions were taken, and what the next steps are. Notes cannot be empty — Save Update is disabled until notes are entered.

5. Click **Save Update**

The header card updates immediately to reflect the new status, severity, and staleness reset. A new entry appears in the history timeline.

### Escalating an Incident

To formally escalate:

1. Set **Update Type = Escalation**
2. An **Escalation Level** dropdown appears — select:
   - *Escalate*: Notify the domain lead
   - *Executive Brief*: Notify the executive sponsor
3. Enter notes explaining the escalation rationale
4. Click **Save Update**

An escalation email is automatically sent to the appropriate contact. The email includes full incident details. If the incident is P&C, the email subject is prepended with `[PRIVILEGED & CONFIDENTIAL]` and a non-disclosure disclaimer is added to the footer.

> **Important:** For P&C incidents, verify the escalation email recipient is on the authorized access list before escalating. Contact the EIT administrator if unsure.

### Containing an Incident

When the immediate risk has been neutralised:

1. Set **Update Type = Containment**
2. Set **Status = Contained**
3. Enter containment notes (what was done, what controls are in place, any residual risk)
4. Click **Save Update**

The `DateContained` timestamp is set automatically. The incident moves to the Contained swim-lane on the Kanban board.

### Closing an Incident

When all work is complete, documentation is finalised, and no further action is required:

1. Set **Update Type = Closure**
2. Set **Status = Closed**
3. Enter closure notes (summary of outcome, lessons learned, any post-incident recommendations)
4. Click **Save Update**

Closed incidents are excluded from all active views and KPI counts. **Closed incidents are not reopened.** If the same issue recurs, create a new incident and link it via the `RelatedItemIDs` field.

---

## Screen 4: Domain Overview

The Domain Overview screen shows at a glance how many active incidents each domain is managing.

### Domain Cards

Five cards — one per enterprise domain — show:
- Total open incidents (Status ≠ Closed, Status ≠ Contained)
- Critical severity sub-count
- A colour-coded top stripe identifying the domain

**Click any domain card** to open the Tracker Screen pre-filtered to that domain.

---

## Screen 5: Kanban Board

The Kanban board shows incidents as cards in five swim-lanes based on their current status.

### Swim-Lanes

| Lane | Status |
|------|--------|
| **New** | Just promoted from intake — awaiting active management |
| **Active** | Actively being investigated or remediated |
| **Escalated** | Formally escalated — domain lead or executive engaged |
| **Monitoring** | Immediate response complete — in observation period |
| **Contained** | Risk neutralised — documentation and review in progress |

### Reading Kanban Cards

Each card shows:
- **Item ID** and **Severity badge** (top)
- **Title** (middle)
- **Domain** (italic)
- **Owner** and **days since last update** (bottom)

**Staleness stripe** (left edge of card): Blue = current (0-7 days), Orange = aging (8-14 days), Red = stale (15+ days). Cards with a red stripe need a status update.

**P&C stripe** (right edge of card): A dark red stripe indicates the incident is Privileged & Confidential.

Click any card to open the full **Issue Detail Screen** for that incident.

---

## Screen 6: Executive Report

The Executive Report is a read-only summary of all active incidents by domain and severity. It is designed for leadership briefings and regular review.

### Domain Summary Table

The table shows one row per enterprise domain with the following columns:

| Column | Meaning |
|--------|---------|
| **Domain** | Enterprise domain |
| **Open** | Total active incidents (non-Closed, non-Contained) |
| **Critical** | Incidents at Critical severity |
| **Escalated** | Incidents with any escalation threshold (Watch, Escalate, or Executive Brief) |
| **P&C Active** | Incidents with P&C designation (visible to you based on your access) |
| **Stale** | Incidents with no update in 15+ days |

Non-zero values for Critical, Escalated, Stale, and P&C are highlighted in red to attract attention.

### Enterprise Totals

Below the table, enterprise-wide totals for each metric are displayed.

### Refreshing the Data

The Executive Report reflects the live state of the EIT at the moment you open the screen. To refresh, navigate away and return to the screen — or use the Dashboard navigation buttons.

---

## Understanding Privileged & Confidential (P&C) Incidents

### What P&C Means

A Privileged & Confidential designation means the incident involves **attorney-client privilege** and/or the **work product doctrine**. This is a legal protection — it means the incident details may be shielded from disclosure in regulatory proceedings or litigation, provided they are handled correctly.

**You must:**
- Treat P&C incident details as strictly confidential
- Not discuss P&C incident details with anyone not on the authorized access list
- Not forward P&C escalation emails to unauthorized persons
- Not share screenshots, exports, or summaries of P&C incidents outside the authorized group

**The amber warning banner** on the Issue Detail Screen is your reminder that an incident is P&C-restricted.

### Who Can View P&C Incidents

Access to P&C incidents is restricted at two levels:
1. **SharePoint item-level permissions:** The EIT administrator grants explicit access to specific individuals. If you don't have item-level access, you cannot see the incident even in SharePoint directly.
2. **Power Apps visibility:** The app hides P&C incidents from users not in the P&C-authorized group.

If you believe you should have access to a P&C incident for legitimate business reasons, contact the domain lead or EIT administrator. Access will be documented in the incident history.

### Who Sets the P&C Flag

The submitter may suggest P&C designation when submitting a new incident. The **domain lead** confirms or clears the designation at triage. P&C should only be set when the incident genuinely involves:
- Legal advice from the Office of General Counsel (OGC) or external counsel
- Anticipated or ongoing litigation, arbitration, or regulatory enforcement
- A legal hold order

P&C is **not** appropriate for incidents that are merely sensitive, embarrassing, or involve executives (seniority alone does not confer privilege). Misapplying P&C designation can have legal consequences.

---

## Understanding Severity and Escalation

### Severity Tiers

Severity reflects the enterprise risk management (ERM) classification — not just how urgently you need to act on it.

| Severity | Color | What It Means |
|----------|-------|--------------|
| **Critical** | Dark Red | Enterprise-threatening impact. Requires immediate response and executive awareness. |
| **High** | Red | Significant impact on operations, clients, or regulatory standing. Urgent attention required. |
| **Moderate** | Orange | Meaningful risk with defined scope. Default on promotion — triage reviewer must confirm. |
| **Low** | Blue | Limited, contained impact. Normal operational timelines apply. |
| **Informational** | Grey | No immediate risk. For awareness or record-keeping. |

### Automatic Escalation

The EIT runs a daily check at 07:00 that automatically upgrades escalation thresholds:
- **High severity** incidents with no update for **more than 7 days** → automatically set to `Escalate`
- **Critical severity** incidents with no update for **more than 14 days** → automatically set to `Executive Brief`

**The best way to avoid auto-escalation is to keep your incidents updated.** Any update (notes entry, status change) resets the staleness counter.

### Days Stale Color Coding

The Days Stale indicator on incident rows uses color to communicate urgency:
- **Blue (0-7 days):** Current — incident is being actively managed
- **Orange (8-14 days):** Aging — update is due soon; review and add an update
- **Red (15+ days):** Stale — escalation thresholds may trigger; update required immediately

---

## Frequently Asked Questions

**Q: I can't see some incidents that I know exist. Why?**
A: If the incidents are P&C-restricted and you are not on the authorized access list, they will not appear in the app or in SharePoint. Contact the domain lead or EIT administrator if you need access for a legitimate reason.

**Q: I submitted an incident but it's not in the Tracker yet. Where is it?**
A: Submitted incidents go into the Intake Queue first (TriageStatus=Pending). A domain lead must triage and promote (or accept) the item before it appears in the Tracker. You can see your submission in the Dashboard Intake Queue.

**Q: Can I edit an incident after it's been closed?**
A: Closed incidents are read-only in the tracker. If you have new information or a recurrence, create a new incident and use the RelatedItemIDs field to link it to the closed incident.

**Q: Can I edit the Notes on a previous history entry?**
A: No. The EIT_IncidentHistory list is append-only. History entries cannot be edited or deleted. If a history entry contains an error, add a new entry with a correction note.

**Q: I accidentally submitted a P&C suggestion but the incident isn't actually P&C. How do I fix it?**
A: During triage, the domain lead can confirm or clear the P&C flag. If the incident was already promoted with P&C=Yes, the domain lead or EIT administrator must clear the flag and restore inherited SharePoint permissions. Contact your domain lead or EIT administrator.

**Q: The escalation email went to the wrong person. What do I do?**
A: Check that EIT_DomainContacts has the correct LeadEmail and EscalationEmail for the relevant domain. If the data is incorrect, the EIT administrator must update the DomainContacts list. Contact the EIT administrator.

**Q: The "Save Update" button is greyed out. Why?**
A: The Save Update button is disabled until the Notes field has content. Enter update notes and the button will activate.

**Q: The app is slow to load or showing old data. What should I do?**
A: Try refreshing the screen (navigate away and back). If the issue persists, the M365 service may be experiencing a delay. Check the Microsoft 365 Service Health dashboard in your admin portal.

---

## Quick Reference — Key Actions

| I want to... | Where to go |
|-------------|-------------|
| Submit a new incident | Dashboard → "+ New Incident" button |
| Review pending intake items | Dashboard → Intake Queue gallery |
| Promote an intake item to the tracker | Dashboard → click item → "Move to Tracker" (or Promote via Flow button) |
| Find a specific incident | Tracker → search box or domain filter chips |
| Update an incident's status or notes | Tracker → click incident → Issue Detail → Add Update form |
| Escalate an incident | Issue Detail → UpdateType=Escalation → select level → Save Update |
| Mark an incident as Contained | Issue Detail → Status=Contained, UpdateType=Containment → Save Update |
| Close an incident | Issue Detail → Status=Closed, UpdateType=Closure → Save Update |
| See all incidents by domain | By Domain (navigation) or Domain Overview screen |
| See incidents as a visual board | Kanban (navigation) |
| See leadership summary | Exec Report (navigation) |
| View P&C incident (if authorized) | Tracker → P&C incidents shown with dark red left-stripe |

---

## Support

For issues with the app, contact your **EIT Administrator**.

For questions about whether an incident should be designated P&C, contact your **domain lead** or the **Office of General Counsel**.

For access requests, contact your **EIT Administrator** and specify the incident you need access to and your business reason.
