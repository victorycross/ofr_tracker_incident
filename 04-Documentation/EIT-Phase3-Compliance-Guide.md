# Enterprise Incident Tracker — Phase 3 Compliance & Regulatory Reporting Guide

**Version:** 1.0
**Date:** February 2026
**Audience:** Compliance Officer, Legal Counsel, EIT administrator, domain leads preparing regulatory submissions
**Classification:** Internal — Restricted

---

## Overview

The Enterprise Incident Tracker generates a structured audit trail that supports compliance with multiple regulatory and governance frameworks. This guide covers:

1. How EIT data maps to common regulatory frameworks (ISO 27001, NIST SP 800-61, APRA CPS 234)
2. How to produce quarterly board and executive compliance reports from EIT data
3. Regulatory submission guidance — what to include, what to exclude
4. Using the monthly automated compliance report (from `EIT-Compliance-Report-Flow.md`)
5. Evidence package assembly for regulators and auditors

---

## Section 1: Regulatory Framework Mapping

### 1.1 ISO 27001:2022 — Information Security Management

| ISO 27001 Control | EIT Support |
|------------------|------------|
| **A.5.24** — Information security incident management planning and preparation | EIT provides the operational platform for incident management. Existence of EIT demonstrates the control is operationalised. |
| **A.5.25** — Assessment and decision on information security events | `EIT_IntakeQueue` triage process documents the assessment step. `TriageStatus` shows the decision (Promoted/Dismissed/Accepted/Rejected). |
| **A.5.26** — Response to information security incidents | `EIT_Incidents.NextAction` and `EIT_IncidentHistory` document response actions and progress. |
| **A.5.27** — Learning from information security incidents | EIT_IncidentHistory append-only audit trail provides the lessons-learned record. Post-incident review updates are captured as history entries. |
| **A.5.28** — Collection of evidence | `EIT_IncidentHistory` with `UpdatedBy`, `UpdateDate`, and append-only structure provides tamper-evident evidence. P&C flag protects legally privileged evidence. |
| **A.8.15** — Logging | `EIT_IncidentHistory` is the incident management log. `DateRaised`, `LastUpdated`, `DaysSinceUpdate`, and history timestamps provide a complete timeline. |
| **A.8.16** — Monitoring activities | Staleness monitoring (daily flow) and escalation automation demonstrate active monitoring. Stale incident reports are the monitoring output. |

**Evidence package for ISO 27001 audit:**
- Export of `EIT_Incidents` (last 12 months) showing incident lifecycle (Status transitions)
- Export of `EIT_IncidentHistory` showing response actions
- Screenshot or export of the Executive Report Screen showing domain/severity breakdown
- The monthly compliance report (automated, from the flow)
- Evidence that the daily staleness flow ran (Power Automate run history)

### 1.2 NIST SP 800-61 — Computer Security Incident Handling Guide

NIST defines four incident response phases: Preparation, Detection & Analysis, Containment/Eradication/Recovery, Post-Incident Activity.

| NIST Phase | EIT Mapping |
|-----------|------------|
| **Preparation** | EIT itself is the preparation artifact. `EIT_DomainContacts` holds the escalation contacts. The taxonomy (severity, escalation thresholds) defines the response framework. |
| **Detection & Analysis** | `EIT_IntakeQueue` is the detection capture mechanism. Severity assessment on promotion is the analysis step. iSight integration (Phase 2) automates threat detection. |
| **Containment** | `Status = Contained` and `DateContained` document the containment action and timing. Mean time to contain is the primary KPI for this phase. |
| **Eradication & Recovery** | `Status = Closed` documents recovery. `NextAction` entries during this phase document eradication steps. |
| **Post-Incident Activity** | Append-only `EIT_IncidentHistory` provides the post-incident record. Notes captured as `UpdateType = Lessons Learned` (if this value is added to the taxonomy) support the post-incident review. |

**Key NIST metrics from EIT:**
- Mean time to detect (MTD) — `DateRaised` minus the actual event date (if captured in NextAction)
- Mean time to contain — `DateContained - DateRaised`
- Mean time to resolve — `LastUpdated (Closed) - DateRaised`
- Number of incidents per severity tier per quarter

### 1.3 APRA CPS 234 — Information Security (Australian context)

| CPS 234 Requirement | EIT Support |
|--------------------|------------|
| **Para 36** — Notify APRA within 72 hours of material information security incident | EIT's escalation automation (Executive Brief level) triggers within the detection-to-notification window. The EscalationThreshold and timestamp provide evidence of timely escalation. |
| **Para 37** — Annual independent testing | EIT test plan (Phase 1 and Phase 2) provides documented evidence of testing. Export test results as the testing record. |
| **Para 38** — Incident management capability | EIT is the incident management capability. Its existence, active use, and the audit trail it generates demonstrate the capability. |

> **Sector-specific note:** Substitute your applicable sector regulation for APRA CPS 234. The EIT data structure supports most incident notification and record-keeping requirements across financial services, health, government, and critical infrastructure sectors. Coordinate with Legal Counsel on the specific notification obligations applicable to your organisation.

---

## Section 2: Board and Executive Compliance Reports

### 2.1 Quarterly Board Report Format

The following format is recommended for board-level incident reporting using EIT data. Produce this report using the Power BI Executive Dashboard (Phase 2), the automated compliance report email (Phase 3), and a one-page summary narrative.

**Recommended Structure:**

```
ENTERPRISE INCIDENT TRACKER — QUARTERLY BOARD REPORT
[Quarter] [Year] | As of [date]

1. EXECUTIVE SUMMARY
   In [Quarter], the Enterprise Incident Tracker recorded [N] incidents across
   [N] enterprise domains. [N] incidents were escalated to Executive Brief
   level. [N] incidents were contained; [N] were fully resolved.

2. INCIDENT ACTIVITY SUMMARY
   ┌─────────────────────────┬────────────┬────────────┬────────────┐
   │ Metric                  │ This Qtr   │ Prior Qtr  │ YTD        │
   ├─────────────────────────┼────────────┼────────────┼────────────┤
   │ Incidents Raised        │            │            │            │
   │ Incidents Contained     │            │            │            │
   │ Incidents Resolved      │            │            │            │
   │ Critical Incidents      │            │            │            │
   │ Executive Brief         │            │            │            │
   │ Mean Time to Contain    │            │            │            │
   │ Stale Incidents (EoQ)   │            │            │            │
   └─────────────────────────┴────────────┴────────────┴────────────┘

3. DOMAIN BREAKDOWN
   [Bar chart or table from Power BI — open incidents by domain]

4. SEVERITY PROFILE
   [Donut chart or table from Power BI — open incidents by severity]

5. NOTABLE INCIDENTS
   [Narrative of 2-3 significant incidents, without P&C-protected details]

6. RISK INDICATORS
   ┌──────────────────────────────────┬──────────────────────────────┐
   │ Indicator                        │ Status                       │
   ├──────────────────────────────────┼──────────────────────────────┤
   │ Stale incident ratio             │ [N]% of open incidents stale │
   │ Domains with critical incidents  │ [list domains]               │
   │ Escalation rate                  │ [N]% of incidents escalated  │
   │ P&C incidents active             │ [available to authorised     │
   │                                  │  board members only]         │
   └──────────────────────────────────┴──────────────────────────────┘

7. MANAGEMENT RESPONSE
   [Domain leads provide brief narrative on significant incidents]
```

### 2.2 Producing the Quarterly Board Report

1. Run the monthly compliance reports for each month of the quarter (automated, from the flow)
2. Open Power BI dashboard — navigate to **Executive Overview** and **Resolution Metrics** pages
3. Export the relevant Power BI visuals (click visual → Export → Export data or use Print function)
4. In Excel: aggregate the three monthly compliance reports to produce quarterly totals
5. Insert the Power BI charts and quarterly totals into the board report template
6. Add the risk indicators narrative — these require human judgement on context
7. Submit to board at least 48 hours before the board meeting to allow reading time

> **P&C handling in board reports:** Do not include P&C incident titles, descriptions, or owners in the board report unless all board recipients have P&C access clearance. Use the `P&C Active` count only (no detail). If the board includes external non-executive directors without P&C clearance, prepare a separate P&C-redacted version.

---

## Section 3: Regulatory Submission Evidence Package

### 3.1 What Regulators Typically Request

| Regulatory Request | EIT Evidence Source |
|------------------|-------------------|
| "List all information security incidents in the past 12 months" | Export `EIT_Incidents` filtered: IncidentType = Cyber, DateRaised in last 12 months |
| "Show your incident response capability" | EIT system documentation (SDD), test plan and results, flow screenshots |
| "Evidence of timely escalation for critical incidents" | Filter `EIT_IncidentHistory` for EscalationThreshold changes; timestamps show escalation timing |
| "Board awareness of material incidents" | Board report distribution record; Executive Brief email records; escalation flow run history |
| "Incident notification timeline" | `DateRaised` (detection), EscalationThreshold change date (internal escalation), regulatory notification date (note manually) |
| "Post-incident review evidence" | `EIT_IncidentHistory` entries with UpdateType = Lessons Learned; or closure notes in NextAction |

### 3.2 Export Procedure for Regulators

**Standard export (non-P&C incidents):**

1. Navigate to `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker/Lists/EIT_Incidents`
2. Apply filters:
   - Remove Status = Closed if active-only requested
   - Filter PrivilegedAndConfidential = No (unless regulator has P&C access)
   - Filter by date range as required
3. Click **Export to Excel** in the list toolbar
4. In Excel: remove columns not relevant to the submission (e.g., internal operational fields)
5. Add a cover sheet noting: export date, filter criteria applied, EIT version

**P&C incident export (requires Legal Counsel sign-off):**

1. Before exporting P&C data: obtain written Legal Counsel approval for the specific regulatory disclosure
2. The P&C designation may or may not be disclosable depending on the regulatory context
3. In some cases, P&C incidents are disclosed with the description and NextAction fields redacted
4. Legal Counsel determines the disclosure approach for each submission

> **Never export raw P&C incident data to a regulator without Legal Counsel review.** The attorney-client privilege protecting P&C incidents may be waived by disclosure. See `EIT-Access-Control-Guide.md`.

### 3.3 Incident Notification Tracking

The EIT does not natively track regulatory notification actions (e.g., "Notified [Regulator] at 14:00 on 24 February 2026"). Capture this in the `EIT_IncidentHistory` as a dedicated update:

- **UpdateType:** Compliance Action *(add this to the UpdateType taxonomy if not present)*
- **Notes:** "Regulatory notification submitted to [Regulator name] at [time] by [Name]. Reference: [regulator reference number]"
- **UpdatedBy:** the person who made the notification

This creates a timestamped, auditable record of notification within the EIT system.

---

## Section 4: Using the Automated Monthly Compliance Report

The `EIT Monthly Compliance Report` flow (see `EIT-Compliance-Report-Flow.md`) generates and emails a structured metrics report on the 1st of each month.

### 4.1 Report Distribution

Recipients (configured in the flow):
- **EIT Administrator** — receives every report; responsible for compliance records
- **Compliance Officer / CISO** (`[EXEC-ESCALATION-EMAIL]`) — receives every report for compliance monitoring

Add additional recipients as CC in the flow if required (General Counsel, Board Risk Committee Chair).

### 4.2 Reading the Monthly Report

| Metric | Healthy range | Action if outside range |
|--------|--------------|------------------------|
| Incidents raised vs. resolved | Resolved ≥ Raised (no accumulation) | Investigate if backlog grows month-over-month |
| Critical incidents | Should be few; any sustained high count warrants executive attention | Escalate to board if Critical count > [threshold to be agreed] |
| Stale incidents | Target: 0 stale at month-end | Initiate domain lead review; check staleness flow health |
| P&C active | Inform Legal Counsel; monitor trend | Significant increase may indicate over-designation |
| Mean time to contain | Target: ≤ 5 days for Critical; ≤ 14 days for High | If above threshold: process review; consider resourcing |

### 4.3 Archiving Compliance Reports

Each monthly report email should be saved to a permanent compliance record. The flow optionally saves to a SharePoint library. Additionally:

1. Each compliance officer should save incoming compliance report emails to a dedicated compliance mailbox folder
2. Compliance reports are retained for 3 years minimum (per standard records retention)
3. Reports for any month in which a Critical or Executive Brief incident occurred should be retained for 7 years

---

## Section 5: Compliance Metrics Reference Card

For quick reference during regulatory interviews or board presentations:

| Metric | EIT Source | Formula |
|--------|-----------|---------|
| Total open incidents | EIT_Incidents | Count where Status ≠ Closed and Status ≠ Contained |
| Critical open incidents | EIT_Incidents | Count where Severity = Critical and Status ≠ Closed |
| Incidents by domain | EIT_Incidents | Group by Domain, count open |
| Mean time to contain | EIT_Incidents | Avg(DateContained − DateRaised) for Contained items |
| Mean time to close | EIT_Incidents | Avg(LastUpdated − DateRaised) for Closed items |
| Escalation rate | EIT_Incidents | Count(EscalationThreshold ≠ Not Escalated) / Count(open) |
| P&C incidents active | EIT_Incidents | Count where P&C=Yes and Status ≠ Closed |
| Stale rate | EIT_Incidents | Count(StalenessFlag=Stale) / Count(open) |
| Incidents from ServiceNow | EIT_Incidents | Count where SourceSystem=ServiceNow |
| Incidents from iSight | EIT_Incidents | Count where SourceSystem=iSight |

---

## Section 6: Taxonomy Alignment for Reporting

### 6.1 Severity to Regulatory Materiality Mapping

Regulators often use different severity/materiality terminology. The following mapping helps translate EIT severity to regulatory language:

| EIT Severity | Regulatory Equivalent | Notification Trigger (typical) |
|-------------|----------------------|-------------------------------|
| Critical | Material / High Impact | Yes — likely triggers regulatory notification |
| High | Significant / Medium-High Impact | Possibly — assess on case-by-case basis |
| Moderate | Minor / Medium Impact | No — monitor and document |
| Low | Negligible / Low Impact | No |
| Informational | Informational / No immediate impact | No |

> **Note:** Regulatory materiality thresholds vary by sector and jurisdiction. Legal Counsel must determine notification obligations for each incident; EIT severity alone is not the determining factor.

### 6.2 EIT Domain to Organisational Function Mapping

For submissions using functional terminology rather than EIT domain names:

| EIT Domain | Organisational Function |
|-----------|------------------------|
| National Security | National Security / Government Relations |
| NIS | Network & Information Security / Cybersecurity |
| Privacy & Technology | Privacy / Data Protection / Technology Risk |
| Human Capital | Human Resources / People Risk |
| Physical Security | Physical Security / Facilities |
