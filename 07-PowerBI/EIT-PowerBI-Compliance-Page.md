# Enterprise Incident Tracker — Power BI Compliance Page Guide

**Version:** 1.0
**Date:** February 2026
**Audience:** Power BI developer, EIT administrator
**Classification:** Internal

---

## Overview

This guide adds a 6th page to the EIT Power BI dashboard: the **Compliance & Audit** page. This page is designed for the Compliance Officer and CISO, providing metrics aligned to regulatory frameworks and board reporting.

**Prerequisite:** Complete the EIT Power BI Build Guide (`EIT-PowerBI-Build-Guide.md`) first. This page extends the existing 5-page report.

---

## New DAX Measures Required

Add the following measures to the `EIT_Incidents` table before building the page. See `EIT-PowerBI-Build-Guide.md` Section 2 for how to create measures.

### Incident Notification Metrics

```dax
Total Incidents YTD =
COUNTROWS(
    FILTER(
        EIT_Incidents,
        EIT_Incidents[DateRaised] >= DATE(YEAR(TODAY()), 1, 1)
    )
)
```

```dax
Critical Incidents YTD =
COUNTROWS(
    FILTER(
        EIT_Incidents,
        EIT_Incidents[Severity] = "Critical" &&
        EIT_Incidents[DateRaised] >= DATE(YEAR(TODAY()), 1, 1)
    )
)
```

```dax
Executive Brief YTD =
COUNTROWS(
    FILTER(
        EIT_Incidents,
        EIT_Incidents[EscalationThreshold] = "Executive Brief" &&
        EIT_Incidents[DateRaised] >= DATE(YEAR(TODAY()), 1, 1)
    )
)
```

```dax
Incidents by Source - ServiceNow =
COUNTROWS(FILTER(EIT_Incidents, EIT_Incidents[SourceSystem] = "ServiceNow"))
```

```dax
Incidents by Source - iSight =
COUNTROWS(FILTER(EIT_Incidents, EIT_Incidents[SourceSystem] = "iSight"))
```

```dax
Incidents by Source - Manual =
COUNTROWS(
    FILTER(
        EIT_Incidents,
        EIT_Incidents[SourceSystem] = "Manual Entry" ||
        EIT_Incidents[SourceSystem] = "Power Apps"
    )
)
```

### Resolution SLA Measures

```dax
Critical Within SLA (5 Days) =
COUNTROWS(
    FILTER(
        EIT_Incidents,
        EIT_Incidents[Severity] = "Critical" &&
        NOT(ISBLANK(EIT_Incidents[DateContained])) &&
        DATEDIFF(EIT_Incidents[DateRaised], EIT_Incidents[DateContained], DAY) <= 5
    )
)
```

```dax
Critical Breached SLA (5 Days) =
COUNTROWS(
    FILTER(
        EIT_Incidents,
        EIT_Incidents[Severity] = "Critical" &&
        NOT(ISBLANK(EIT_Incidents[DateContained])) &&
        DATEDIFF(EIT_Incidents[DateRaised], EIT_Incidents[DateContained], DAY) > 5
    )
)
```

```dax
High Within SLA (14 Days) =
COUNTROWS(
    FILTER(
        EIT_Incidents,
        EIT_Incidents[Severity] = "High" &&
        NOT(ISBLANK(EIT_Incidents[DateContained])) &&
        DATEDIFF(EIT_Incidents[DateRaised], EIT_Incidents[DateContained], DAY) <= 14
    )
)
```

### Audit Trail Completeness

```dax
Incidents With History =
COUNTROWS(
    FILTER(
        EIT_Incidents,
        COUNTROWS(
            RELATEDTABLE(EIT_IncidentHistory)
        ) > 0
    )
)
```

```dax
Audit Trail Completeness % =
DIVIDE(
    [Incidents With History],
    COUNTROWS(EIT_Incidents),
    0
) * 100
```

---

## Page 6: Compliance & Audit

### Add the Page

1. In Power BI Desktop, right-click the last page tab → **New page**
2. Double-click the new tab → rename to `Compliance & Audit`

---

### Section A: YTD KPI Summary (Top Row)

Add 5 **Card** visuals across the top (approx Y=20, height=80, width=195 each):

| Card | Measure | Label |
|------|---------|-------|
| 1 | `Total Incidents YTD` | Incidents YTD |
| 2 | `Critical Incidents YTD` | Critical YTD |
| 3 | `Executive Brief YTD` | Executive Brief YTD |
| 4 | `Mean Time to Contain (Days)` | Avg Days to Contain |
| 5 | `Audit Trail Completeness %` | Audit Completeness |

**Conditional formatting:**
- Card 2 (Critical YTD): if value > 0, font = Red `#E0301E`
- Card 4 (MTTC): if value > 5, font = Red (Critical SLA breach); if 5–14, font = Orange
- Card 5 (Audit %): if value < 100, font = Orange (some incidents lack history entries)

---

### Section B: Monthly Incident Trend (Line Chart)

1. Add a **Line chart**
2. **X-axis:** `DateRaised` — set to Month hierarchy (Year, Month)
3. **Y-axis:** Count of `ItemID` (Count, not Sum)
4. **Secondary Y-axis:** `Mean Time to Contain (Days)` (configure as a second line)
5. **Filter:** DateRaised in last 12 months
6. **Title:** `Monthly Incident Volume & Mean Time to Contain (12 Months)`
7. Format:
   - Primary line (volume): Blue `#415385`
   - Secondary line (MTTC): Orange `#E45C2B`
   - Enable data labels on both lines

---

### Section C: SLA Adherence (Clustered Bar Chart)

Show the proportion of Critical and High incidents contained within SLA.

1. Add a **Clustered bar chart** (or Stacked bar)
2. Build as two bars using calculated measures:
   - Critical: `Critical Within SLA (5 Days)` vs `Critical Breached SLA (5 Days)`
   - High: `High Within SLA (14 Days)` vs remaining High incidents beyond 14 days

> **Implementation note:** Power BI doesn't support two independent stacked bars from measures directly. Create a **summary table** in DAX:
>
> ```dax
> SLA Table =
> DATATABLE(
>     "Severity", STRING,
>     "SLA Target (Days)", INTEGER,
>     {
>         {"Critical", 5},
>         {"High", 14}
>     }
> )
> ```
>
> Then create measures `Within SLA Count` and `Breached SLA Count` using SELECTEDVALUE on this table, and use a matrix or clustered bar with SLA Table[Severity] on the axis.
>
> Alternatively, simplify by using two separate **Card** visuals: "Critical within 5-day SLA: [N]" and "Critical breached SLA: [N]".

3. **Title:** `Containment SLA Adherence`
4. Colors: Within SLA = Green `#328C50`, Breached = Red `#E0301E`
5. Add a **reference line** at 100% for the SLA target

---

### Section D: Incident Source Breakdown (Donut Chart)

1. Add a **Donut chart**
2. **Values:** Count of `ItemID`
3. **Legend:** `SourceSystem`
4. **Title:** `Incidents by Source System`
5. Colors:
   - Manual Entry / Power Apps: Blue `#415385`
   - ServiceNow: Orange `#E45C2B`
   - iSight: Dark Red `#772028`

This chart shows the evolving mix of automated vs manual incident detection as Phase 2 integrations mature.

---

### Section E: Audit Trail Completeness Table

Identifies incidents without any history entries — these represent gaps in the audit trail that may be compliance concerns.

1. Add a **Table** visual
2. Columns: `ItemID`, `Title`, `Domain`, `Severity`, `Status`, `DateRaised`, `Owner`
3. **Visual-level filter:** Apply a filter where the count of related `EIT_IncidentHistory` rows = 0

> **DAX filter approach:** Create a calculated column in `EIT_Incidents`:
> ```dax
> HistoryCount = COUNTROWS(RELATEDTABLE(EIT_IncidentHistory))
> ```
> Then filter the table visual where `HistoryCount = 0`.

4. **Title:** `Incidents Without Audit Entries (Gap Report)`
5. Format the `ItemID` column in Dark Red to draw attention

> **Expected result:** This table should be empty for a well-maintained EIT. Any incident without a history entry was created but never updated — possible system error or process breach.

---

### Section F: P&C Trend (Line Chart)

Track the number of active P&C incidents over time to support legal privilege management reviews.

1. Add a **Line chart**
2. **X-axis:** `DateRaised` — Month hierarchy
3. **Y-axis:** Count of `ItemID` where `PrivilegedAndConfidential = TRUE`
4. **Filter:** `PrivilegedAndConfidential = TRUE`
5. **Title:** `Active P&C Incidents Over Time`

> **RLS note:** This visual will be filtered by RLS for Standard Users — they will see a flat line at 0. Only P&C-authorized users and Power BI workspace admins see actual P&C counts. This is correct and expected behaviour.

---

### Section G: Escalation Response Heatmap (Matrix)

Shows escalation distribution by domain and severity — useful for identifying patterns in which domains or severity tiers generate the most escalations.

1. Add a **Matrix** visual
2. **Rows:** `Domain`
3. **Columns:** `EscalationThreshold`
   - Column order: Watch, Escalate, Executive Brief (use custom sort)
4. **Values:** Count of `ItemID`
5. **Filter:** `EscalationThreshold ≠ Not Escalated` and `Status ≠ Closed`
6. **Conditional formatting:** Background color by value (darker = more escalations)
7. **Title:** `Active Escalations by Domain and Level`

---

## Page Filters

Add the following page-level filters to the Compliance & Audit page:

1. **Date Range Slicer** — allow filtering by DateRaised year or custom date range
2. **Domain Slicer** (optional) — for domain-specific compliance reviews

These allow the Compliance Officer to focus the page on a specific period or domain for regulatory submissions.

---

## RLS Considerations

The Compliance & Audit page contains the P&C trend chart (Section F). Ensure RLS is applied correctly:

- Standard Users (`Standard_Users` RLS role): P&C trend shows 0 — correct
- P&C Authorized (no RLS role): P&C trend shows real counts — correct
- Workspace Admins: See all data — correct

The `Audit Trail Completeness %` and `Gap Report` table (Section E) do not require RLS as they do not surface incident content — only structural metadata (incident IDs and whether history exists).

---

## Publishing

After adding Page 6:

1. Click **File** → **Publish** → **Publish to Power BI** → overwrite the existing report
2. Verify RLS assignments are preserved in Power BI Service (check dataset Security)
3. Confirm the new page is visible to the Compliance Officer user (add their account to the workspace if not already a member)
4. Brief the Compliance Officer on the page layout and how to use the date range slicer for quarterly submissions
