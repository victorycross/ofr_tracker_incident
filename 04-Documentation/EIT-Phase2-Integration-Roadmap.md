# Enterprise Incident Tracker — Phase 2 Integration Roadmap

**Version:** 1.0
**Date:** February 2026
**Classification:** Internal

---

## Overview

Phase 1 of the Enterprise Incident Tracker delivers a fully operational M365-native tracking platform. Phase 2 is the integration layer — connecting EIT to existing source systems so that incidents flow into the tracker automatically and data is not duplicated across silos.

This document outlines the Phase 2 integration architecture, recommended integration patterns, and the preparatory design decisions embedded in Phase 1 to facilitate Phase 2.

---

## Phase 1 → Phase 2 Transition Summary

| Capability | Phase 1 (Current) | Phase 2 (Target) |
|-----------|------------------|------------------|
| Incident source | Manual submission via Power App or email forward | Automated ingest from ServiceNow, iSight, Manual Entry |
| Reporting | Live app view + SharePoint export | Power BI dashboard with trend analysis |
| Notifications | Escalation emails (domain lead, exec) | Configurable notification rules, Teams channel alerts |
| Access control | SharePoint groups + item-level permissions | Entra ID dynamic groups, conditional access |
| Staleness calculation | Daily Power Automate scheduled flow | Same or migrated to Logic Apps for higher reliability |
| Data retention | SharePoint default (no archiving) | Automated archiving for Closed incidents older than 12 months |

---

## Phase 2 Integration 1: ServiceNow → EIT_Incidents

### Business Need

The NIS and Privacy & Technology domains currently manage incidents in ServiceNow. High-severity incidents identified in ServiceNow should automatically appear in the EIT without manual re-entry.

### Recommended Pattern

**Trigger:** ServiceNow record reaches a defined priority/category threshold (e.g., Priority = P1 or P2, Category = Security Incident).

**Mechanism:** ServiceNow REST API → Power Automate HTTP connector (or Logic Apps HTTP trigger) → SharePoint Create Item action on `EIT_Incidents`.

**Architecture:**

```
ServiceNow (Incident Management)
    │
    │  REST webhook / outbound notification
    │  (on P1/P2 create or status change)
    ▼
Power Automate / Logic Apps
    │
    ├── Map ServiceNow fields → EIT_Incidents columns:
    │     ServiceNow Priority P1/P2 → Severity High/Critical
    │     ServiceNow Category → IncidentType
    │     ServiceNow Assignment Group → Domain (lookup table)
    │     ServiceNow Description → NextAction (initial entry)
    │
    ├── Create item in EIT_Incidents
    │     SourceSystem = "ServiceNow"
    │     ItemID = "EIT-" + sequence number
    │
    ├── Create opening entry in EIT_IncidentHistory
    │     Notes = "Auto-imported from ServiceNow #[ticket number]"
    │
    └── Optionally: update ServiceNow record with EIT ItemID for cross-reference
```

**Phase 1 Preparation Already in Place:**
- `SourceSystem` column in `EIT_Incidents` has `ServiceNow` as a choice value
- `RelatedItemIDs` column can store the ServiceNow ticket number as a cross-reference
- `EIT_DomainContacts` provides the domain-to-lead mapping needed for routing

**Key Design Decisions for Phase 2 Build:**
1. Define the priority-to-severity mapping table (ServiceNow P1 → Critical, P2 → High, P3 → Moderate, etc.)
2. Define the ServiceNow assignment group → EIT Domain mapping table
3. Determine whether ServiceNow incidents should flow to EIT_IntakeQueue (for triage) or directly to EIT_Incidents (bypassing triage for high-severity automated alerts)
4. Establish a deduplication check: if the ServiceNow ticket is already in EIT (by RelatedItemIDs match), update the existing record rather than creating a duplicate

---

## Phase 2 Integration 2: iSight → EIT_Incidents

### Business Need

The NIS domain uses iSight for threat intelligence. Critical threat indicators should be automatically surfaced in the EIT as new incidents for assessment and tracking.

### Recommended Pattern

**Trigger:** iSight alert reaching a defined severity threshold.

**Mechanism:** iSight API → Power Automate HTTP connector → SharePoint Create Item on `EIT_Incidents`.

**Architecture:**

```
iSight (Threat Intelligence)
    │
    │  API polling (scheduled) or webhook (if supported by iSight version)
    ▼
Power Automate Scheduled Flow
    │
    ├── GET iSight alerts (since last run)
    ├── Filter for severity >= threshold
    │
    ├── For each new alert:
    │     Create item in EIT_Incidents
    │       Domain = "NIS"
    │       IncidentType = "Threat"
    │       Severity = mapped from iSight severity
    │       SourceSystem = "iSight"
    │       Title = iSight alert title
    │       NextAction = iSight alert description
    │
    └── Create history entry noting iSight source and alert ID
```

**Phase 1 Preparation Already in Place:**
- `SourceSystem` column has `iSight` as a choice value
- `IncidentType = "Threat"` defined in taxonomy

**Key Design Decisions:**
1. iSight API authentication method (API key, OAuth)
2. Polling interval (recommend 15-minute scheduled flow)
3. iSight alert severity → EIT Severity mapping
4. Whether iSight alerts go to intake queue or direct to tracker (recommend intake queue to allow NIS domain lead triage)

---

## Phase 2 Integration 3: Power BI Executive Dashboard

### Business Need

The Executive Report Screen in Phase 1 provides a live view but requires opening the Power App. Senior leadership and the board need a persistent, auto-refreshing dashboard accessible without opening Power Apps — ideally embedded in Teams or SharePoint.

### Recommended Pattern

**Mechanism:** Power BI Desktop/Service → SharePoint Online connector → `EIT_Incidents`, `EIT_IncidentHistory` → published Power BI report → embedded in Teams tab or SharePoint page.

**Dashboard Components:**

| Page | Content |
|------|---------|
| **Executive Overview** | Open incidents by domain (bar chart), severity distribution (donut), escalation funnel (funnel chart), trending (line chart: incidents opened vs. closed per week) |
| **Staleness View** | Aging heatmap by domain and severity, incidents approaching staleness threshold |
| **Escalation Drill-Down** | Table of all Escalated and Executive Brief incidents with owner and days since update |
| **Domain Detail** | Per-domain filter; incident list with status, severity, escalation level, owner |
| **Resolution Metrics** | MTD contained and closed counts; mean time to contain (days from DateRaised to DateContained); mean time to close |

**Refresh cadence:** Scheduled dataset refresh at 06:00 and 18:00 daily via Power BI Service.

**Phase 1 Preparation Already in Place:**
- `DateContained` column enables mean time to contain calculation
- `SourceSystem` enables breakdown by origin
- `PrivilegedAndConfidential` column allows P&C incidents to be excluded from shared dashboards (using row-level security in Power BI)

**Key Design Decisions:**
1. Row-level security: P&C incidents should be excluded from broadly shared Power BI reports unless RLS is configured
2. Licensing: Power BI Pro required for publishing and sharing reports (included with M365 E3/E5; add-on for Business Standard)
3. Embedding in Teams: use Teams Power BI tab (native) or SharePoint webpart for dashboard embedding

---

## Phase 2 Integration 4: Teams Channel Notifications

### Business Need

Domain leads and executive stakeholders want incident notifications in Microsoft Teams channels rather than (or in addition to) email.

### Recommended Pattern

**Mechanism:** Extend existing Power Automate escalation flow to add a Teams adaptive card notification alongside the email.

**Architecture:**

```
EIT Domain Escalation Notifier (existing flow)
    │
    ├── [Existing] Send email (Outlook V2)
    │
    └── [New] Post adaptive card to Teams channel
          Channel: "EIT-[Domain]-Escalations" (or a unified "EIT-Escalations" channel)
          Card content: Incident ID, Title, Domain, Severity, Escalation Level, Owner, Days Stale
          Action button: "Open in EIT" (deep link to Power App with selected incident)
```

**Phase 1 Preparation:** The escalation flow architecture is modular — adding a Teams action inside the existing Condition branches (If yes / If no) is straightforward.

---

## Phase 2 Integration 5: Automated Archiving

### Business Need

Closed incidents accumulate in `EIT_Incidents` over time. Items older than 12 months add noise to the list and may eventually approach SharePoint list item limits (30M items — unlikely but possible).

### Recommended Pattern

**Mechanism:** Monthly Power Automate scheduled flow:
1. Get items from `EIT_Incidents` where `Status = Closed` AND `LastUpdated < addDays(utcNow(), -365)`
2. For each item: copy to an archive SharePoint list (`EIT_Incidents_Archive`) or SharePoint library (as a JSON file)
3. Copy associated `EIT_IncidentHistory` entries
4. Delete original items from `EIT_Incidents` and `EIT_IncidentHistory`

**Phase 1 Preparation:** No structural changes needed. `DateContained` and `LastUpdated` columns support the date-based archive filter.

---

## Phase 2 Decision Framework

| Integration | Complexity | Estimated Build Effort | Prerequisites |
|-------------|-----------|----------------------|---------------|
| ServiceNow → EIT | Medium | 2–3 weeks | ServiceNow API access, ServiceNow admin cooperation, field mapping workshop |
| iSight → EIT | Medium | 1–2 weeks | iSight API access or webhook capability, NIS team lead agreement on threshold mapping |
| Power BI Dashboard | Low-Medium | 1–2 weeks | Power BI Pro licenses, data model design, executive sign-off on KPI definitions |
| Teams Notifications | Low | 2–3 days | Teams channel structure agreed, EIT escalation flow access |
| Automated Archiving | Low | 1 week | Retention policy agreed (12 months default), archive list created |

**Recommended Phase 2 sequence:**
1. Power BI Dashboard (highest leadership value, lowest dependency)
2. Teams Notifications (quick win, extends existing flow)
3. ServiceNow integration (highest volume source, most complex field mapping)
4. iSight integration (NIS-specific, can proceed in parallel with ServiceNow)
5. Automated Archiving (operational hygiene, low urgency)

---

## Phase 1 Design Decisions Supporting Phase 2

The following Phase 1 design choices were made explicitly to enable Phase 2 integration:

| Decision | Phase 2 Benefit |
|----------|----------------|
| `SourceSystem` column with ServiceNow, iSight, Manual Entry choices | Enables source-of-record tracking and deduplication in Phase 2 flows |
| `RelatedItemIDs` text column | Stores ServiceNow/iSight ticket numbers for cross-reference |
| `EIT_DomainContacts` as a reference list | Powers domain routing in both Phase 1 escalation and Phase 2 ingest flows |
| `DateContained` column | Enables mean time to contain metric in Power BI |
| `ConfidentialityLevel` and `PrivilegedAndConfidential` columns | Supports Power BI row-level security for P&C incident exclusion |
| `EIT-` prefix ItemID format | Stable identifier for cross-system reference |
| Append-only `EIT_IncidentHistory` | Clean audit trail for Power BI trend analysis and regulatory reporting |
| Modular Power Automate flow architecture | Each flow is independently extensible — Teams step added to escalation flow without restructuring |
