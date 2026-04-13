# Enterprise Incident Tracker — Escalation Thresholds Reference

**Version:** 1.0
**Date:** February 2026
**Classification:** Internal

---

## Overview

The EIT uses a four-tier escalation threshold system. Escalation thresholds serve two purposes:

1. **Automated alert:** Triggers email notification to the domain lead (`Escalate`) or executive sponsor (`Executive Brief`) via the EIT Domain Escalation Notifier flow.
2. **Leadership visibility:** Drives the Escalated KPI card on the Dashboard and the Escalated column in the Executive Report.

Escalation thresholds are set by two mechanisms:
- **Automatically** by the Daily Staleness & Escalation Calculator (scheduled Power Automate flow, runs daily at 07:00)
- **Manually** by a domain lead or EIT administrator via the IssueDetailScreen Add Update form (`UpdateType = Escalation`)

---

## Escalation Matrix

### Automatic Escalation Rules (Daily Flow)

| Severity | DaysSinceUpdate | Auto-Set Threshold | Notification Sent To |
|----------|-----------------|-------------------|----------------------|
| **Critical** | > 14 days | `Executive Brief` | `EIT_DomainContacts.EscalationEmail` (exec sponsor) |
| **High** | > 7 days | `Escalate` | `EIT_DomainContacts.LeadEmail` (domain lead) |
| Any other | Any | *(no change)* | — |

> **Logic:** The flow evaluates every open (non-Closed, non-Contained) incident daily. It only upgrades the threshold — it does not downgrade it. Once an incident reaches `Escalate` or `Executive Brief`, it stays at that level until a domain lead or administrator manually resets it.

### Manual Escalation Override

Domain leads may escalate or de-escalate at any time via the IssueDetailScreen:

1. Open the incident → Add Update form
2. Set `UpdateType = Escalation`
3. Select the desired `EscalationThreshold` value
4. Enter notes explaining the escalation decision
5. Click **Save Update** — this triggers the Domain Escalation Notifier flow immediately

**De-escalation** (setting threshold back to `Not Escalated` or `Watch`) also uses this path. De-escalation does not send a notification email — it is a record-keeping action only.

---

## Escalation Threshold Definitions

### Not Escalated
- **Meaning:** No escalation has been triggered. The incident is being actively managed within normal operational parameters.
- **Set by:** Default on promotion from intake queue.
- **UI color:** Grey (no badge fill)
- **Notification:** None

### Watch
- **Meaning:** The domain lead has flagged the incident as approaching escalation thresholds. Heightened attention required but formal escalation not yet warranted.
- **Set by:** Domain lead or EIT administrator (manual only — not auto-set by the daily flow).
- **UI color:** Grey (same as Not Escalated visually — distinguished by text)
- **Notification:** None

### Escalate
- **Meaning:** Formally escalated to the domain lead. The domain lead is expected to review, provide direction, and ensure the incident is receiving adequate attention.
- **Set by:** Automatic (Severity=High, >7 days) or manual.
- **UI color:** Red `RGBA(224,48,30,1)`
- **Notification:** Escalation email to `EIT_DomainContacts.LeadEmail` for the incident's domain. Subject line: `[ESCALATION] [Incident Title] — [Domain] — [Severity]`. If P&C, prepends `[PRIVILEGED & CONFIDENTIAL]`.

### Executive Brief
- **Meaning:** Escalated to executive sponsor. The executive recipient is expected to be briefed and may need to make decisions or provide resources. This is the highest escalation level.
- **Set by:** Automatic (Severity=Critical, >14 days) or manual by domain lead/EIT admin.
- **UI color:** Dark Red `RGBA(119,40,32,1)`
- **Notification:** Executive Brief email to `EIT_DomainContacts.EscalationEmail`. Subject line: `[EXECUTIVE BRIEF] [Incident Title] — [Domain] — [Severity]`. If P&C, prepends `[PRIVILEGED & CONFIDENTIAL]`.

> **P&C note:** If `PrivilegedAndConfidential = Yes` on the incident, the escalation email subject is prepended with `[PRIVILEGED & CONFIDENTIAL]` and the email footer includes a non-disclosure disclaimer. The email recipient (domain lead or exec sponsor) must be on the authorized P&C access list before the email is sent. Verify SharePoint item-level permissions before triggering escalation on a P&C incident. See `EIT-Access-Control-Guide.md`.

---

## Escalation Notification Email — Field Reference

The Domain Escalation Notifier flow sends the following content:

| Field | Source |
|-------|--------|
| To | `EIT_DomainContacts.LeadEmail` (Escalate) or `EIT_DomainContacts.EscalationEmail` (Executive Brief) |
| Subject | `[ESCALATION]` or `[EXECUTIVE BRIEF]` + Title + Domain + Severity (+ `[PRIVILEGED & CONFIDENTIAL]` if P&C) |
| Incident ID | `EIT_Incidents.ItemID` |
| Title | `EIT_Incidents.Title` |
| Domain | `EIT_Incidents.Domain.Value` |
| Severity | `EIT_Incidents.Severity.Value` |
| Status | `EIT_Incidents.Status.Value` |
| Escalation Level | Input from trigger (`EscalationLevel`) |
| Owner | `EIT_Incidents.Owner` |
| Date Raised | `EIT_Incidents.DateRaised` |
| Last Updated | `EIT_Incidents.LastUpdated` |
| Days Since Update | `EIT_Incidents.DaysSinceUpdate` |
| Next Action | `EIT_Incidents.NextAction` |
| P&C Flag | `EIT_Incidents.PrivilegedAndConfidential` |

---

## Escalation Responsibility Matrix

| Escalation Level | Who Acts | Expected Response Time | Response |
|-----------------|----------|----------------------|---------|
| Watch | Domain Lead | Within 48 hours | Review incident; determine if full escalation warranted |
| Escalate | Domain Lead | Within 24 hours | Direct active response; provide guidance to incident owner; update NextAction |
| Executive Brief | Executive Sponsor + Domain Lead | Within 4 hours of notification | Acknowledge receipt; confirm response plan; escalate externally if required |

---

## When to Escalate Manually (Before Auto-Threshold)

Domain leads should trigger a manual escalation immediately (without waiting for the daily flow) when:

1. **Regulatory notification deadline is approaching** — e.g., PIPEDA 72-hour breach notification window
2. **External parties are involved** — police, regulatory body, media inquiry
3. **Litigation or legal proceedings are anticipated** — OGC should be looped in; P&C designation may be required
4. **The incident scope has materially expanded** — what appeared to be Low/Moderate has revealed Critical impact
5. **Named executive is involved** — even if Severity has not yet reached Critical threshold

To manually escalate: Open the incident → Add Update → UpdateType=Escalation → select threshold → add notes documenting the escalation rationale → Save Update.

---

## Escalation Status and P&C Incidents

For incidents marked `PrivilegedAndConfidential = Yes`:

1. **Before triggering escalation**, verify the email recipient (domain lead or exec sponsor) is on the item-level SharePoint permission list for this incident.
2. The escalation email will contain incident details — if the recipient does not have SharePoint access to the incident record, grant it first (see `EIT-Access-Control-Guide.md`, Layer 2).
3. Document the access grant in `EIT_IncidentHistory` before triggering the escalation.
4. The exec sponsor must be added to the SharePoint access list before an Executive Brief escalation is triggered if `EscalationThreshold = Executive Brief` is imminent.

---

## Staleness Thresholds

Staleness is calculated daily based on `DaysSinceUpdate = days elapsed since LastUpdated`.

| StalenessFlag | DaysSinceUpdate | UI Color |
|--------------|-----------------|----------|
| **Current** | 0–7 days | Blue `RGBA(65,83,133,1)` |
| **Aging** | 8–14 days | Orange `RGBA(228,92,43,1)` |
| **Stale** | 15+ days | Red `RGBA(224,48,30,1)` |

An incident becomes **Stale** at 15+ days. Any update (status change, notes entry) resets `DaysSinceUpdate = 0` and `StalenessFlag = Current`. The reset is applied in the `btnSaveUpdate.OnSelect` formula in the IssueDetailScreen.

**Relationship between Staleness and Escalation:** Staleness alone does not trigger escalation. It is the combination of Staleness AND Severity that triggers the auto-escalation thresholds (High/>7 days → Escalate; Critical/>14 days → Executive Brief).
