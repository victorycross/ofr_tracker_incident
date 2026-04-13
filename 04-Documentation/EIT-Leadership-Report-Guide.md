# Enterprise Incident Tracker — Leadership Report Guide

**Version:** 1.0
**Date:** February 2026
**Audience:** Executive stakeholders, domain leads, and anyone producing or interpreting enterprise incident reports from the EIT
**Classification:** Internal

---

## Overview

The Enterprise Incident Tracker provides two reporting surfaces for leadership:

1. **Executive Report Screen** — a live, in-app summary view accessible from the EIT Power App, showing current incident counts by domain and severity.
2. **SharePoint List Export** — a manual export of `EIT_Incidents` data from the SharePoint list for use in offline analysis, presentations, or board-level briefings.

This guide covers how to use each surface, how to interpret the data, and how to produce a point-in-time leadership briefing from EIT data.

---

## 1. Executive Report Screen (In-App)

### Accessing the Screen

1. Open the **Enterprise Incident Tracker** Power App from `make.powerapps.com` or from your Microsoft Teams tab.
2. From the **Dashboard**, click **Exec Report** in the header navigation.
3. The screen loads the current state of all active incidents (non-Closed, non-Contained unless otherwise filtered).

> **Note:** The Executive Report Screen is read-only. It contains no edit controls. All data reflects the live state of the `EIT_Incidents` SharePoint list at the moment of page load. Click the screen navigation again to refresh.

### Reading the Domain Summary Table

The table shows one row per enterprise domain:

| Column | Meaning |
|--------|---------|
| **Domain** | Enterprise domain (National Security, NIS, Privacy & Technology, Human Capital, Physical Security) |
| **Open** | Total active incidents in this domain (Status ≠ Closed, Status ≠ Contained) |
| **Critical** | Incidents with Severity = Critical (subset of Open) |
| **Escalated** | Incidents with EscalationThreshold ≠ "Not Escalated" (Watch, Escalate, or Executive Brief) |
| **P&C Active** | Incidents with PrivilegedAndConfidential = Yes that are not Closed (count reflects your access level — may be understated if you are not on all P&C access lists) |
| **Stale** | Incidents with StalenessFlag = Stale (15+ days without update) |

**Color coding:**
- Zero counts display in grey — no action required for that cell.
- Non-zero Critical, Escalated, Stale counts display in red or dark red — these warrant attention.
- Non-zero P&C counts display in dark red.

### Enterprise Totals Row

Below the domain table, five totals are displayed representing the full enterprise picture across all domains. These are the headline figures for a leadership briefing.

### P&C Restricted Notice

The screen includes a note that P&C counts reflect your individual access level. Users who are not on all P&C incident access lists will see understated counts. The EIT administrator can provide a full P&C count on request.

---

## 2. Navigating to Incident Detail from the Report

The Executive Report Screen is read-only, but you can navigate to the **Tracker Screen** to drill into any domain:

1. From the Domain Overview Screen (accessible via "By Domain" button on Dashboard), click any domain card.
2. The Tracker Screen opens, pre-filtered to that domain.
3. Click any incident row to open the Issue Detail Screen.

This allows leadership to drill from the executive summary into the specific incidents driving their numbers.

---

## 3. Producing a Point-in-Time Leadership Briefing

For board-level reports, QBRs, or regulatory submissions, you may need a point-in-time snapshot rather than a live view. The live app always reflects current state. To produce a snapshot:

### Option A — Screenshot / Screen Capture

1. Open the Executive Report Screen in the Power App.
2. Take a screenshot or use your browser's print-to-PDF function.
3. Annotate the report date (visible in the `Report as of: [date]` label on the screen).

### Option B — SharePoint List Export

For more detailed analysis:

1. Navigate to `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker/Lists/EIT_Incidents`
2. Click **Export to Excel** in the list toolbar (or use the **Export** → **Export to CSV** option)
3. Open in Excel
4. Apply filters:
   - Remove Closed and Contained rows for an active-only view
   - Filter by Domain for a per-domain breakdown
   - Filter by Severity for a severity-tier view
5. Build pivot tables or summary charts as needed for your reporting format

> **P&C caution:** Exported spreadsheets may contain incident titles, descriptions, and metadata for incidents you are authorized to see. Handle exports as confidential documents. Do not share exports broadly — they are not restricted by SharePoint item-level permissions once exported.

### Option C — Power BI (Phase 2 Enhancement)

A Power BI dashboard connected to the EIT SharePoint lists is recommended as a Phase 2 enhancement. This would provide automated executive dashboards with trend analysis, without requiring manual export. See `EIT-Phase2-Integration-Roadmap.md`.

---

## 4. Key Metrics Definitions for Leadership Briefings

Use these definitions when presenting EIT data to executive audiences:

| Metric | Definition | Source |
|--------|-----------|--------|
| **Total Open Incidents** | All active incidents (Status ≠ Closed, Status ≠ Contained) across all domains | `CountRows(Filter(EIT_Incidents, Status.Value <> "Closed", Status.Value <> "Contained"))` |
| **Critical Incidents** | Open incidents assessed at Severity = Critical | Filter by Severity = Critical + open |
| **Escalated Incidents** | Open incidents with EscalationThreshold ≠ "Not Escalated" — includes Watch, Escalate, and Executive Brief levels | Filter by EscalationThreshold |
| **Executive Brief** | Open incidents formally escalated to executive sponsor level | Filter by EscalationThreshold = "Executive Brief" |
| **Stale Incidents** | Open incidents with no update in 15+ days — an indicator of management attention deficit | Filter by StalenessFlag = "Stale" |
| **Contained (MTD)** | Incidents contained in the current month — indicates response effectiveness | Filter by Status = "Contained" + DateContained in current month |
| **Closed (MTD)** | Incidents fully resolved and closed in the current month | Filter by Status = "Closed" + LastUpdated in current month |
| **P&C Active** | Active incidents under attorney-client privilege or work product protection | Filter by PrivilegedAndConfidential = Yes + open |

---

## 5. What Counts Are "Normal"

The EIT does not define a target number of incidents — incidents are what they are. However, the following patterns indicate potential management issues that leadership should probe:

| Pattern | Potential Concern |
|---------|------------------|
| High Stale count relative to Open count | Domain leads are not maintaining update cadence. Consider a standing weekly review. |
| Critical incidents with 0 escalation emails sent | Auto-escalation may not be firing; check flow health. Or incidents were updated within threshold (good). |
| Large Open count with no Contained/Closed movement | Incidents are accumulating without resolution. May indicate resourcing or prioritisation issues. |
| P&C count higher than expected | May indicate over-designation. Legal counsel should periodically audit P&C assignments. |
| Zero Open incidents | Could indicate under-reporting, not absence of incidents. Compare with source system data. |

---

## 6. Domain Lead Reporting Responsibilities

Domain leads are expected to:

1. **Review the Tracker Screen weekly** — ensure all incidents in their domain have a current `NextAction` entry and `LastUpdated` within 7 days.
2. **Triage incoming intake items within 24 hours** — pending intake items older than 24 hours in their domain should be promoted, accepted, or dismissed.
3. **Escalate proactively** — do not wait for the daily flow to auto-trigger escalation. If an incident warrants executive attention, trigger it manually (see `EIT-Escalation-Thresholds.md`).
4. **Update severity as incidents evolve** — Severity is not static. If a Moderate incident reveals Critical scope during investigation, update it via the Add Update form.
5. **Set P&C designation correctly** — apply `PrivilegedAndConfidential = Yes` only when legal criteria are met. Misuse of P&C designation can have legal consequences. See `EIT-Access-Control-Guide.md`.

---

## 7. EIT Viewers — Read-Only Leadership Access

Senior leaders who need read-only visibility but do not manage incidents should be added to the `EIT Viewers` M365 group (SharePoint Read permission). They can:

- Access the Power App and view all non-P&C incidents
- Navigate the Dashboard, Tracker, Domain Overview, Kanban, and Executive Report screens
- Cannot submit intake items, update incident records, or promote intake items

P&C incidents are not visible to `EIT Viewers` unless they have been explicitly granted item-level access by the EIT administrator. See `EIT-Access-Control-Guide.md`.
