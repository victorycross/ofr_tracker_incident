# Enterprise Incident Tracker — Power BI Dashboard Build Guide

**Version:** 1.0
**Date:** February 2026
**Audience:** Power BI developer / EIT administrator
**Classification:** Internal

---

## Overview

This guide covers building the EIT Power BI executive dashboard — a 5-page report connected to the EIT SharePoint lists that provides persistent, auto-refreshing leadership visibility without requiring users to open the Power App.

### What You Will Build

| Page | Purpose |
|------|---------|
| **1. Executive Overview** | Open incidents by domain, severity distribution, escalation funnel, weekly trend |
| **2. Staleness View** | Aging heatmap by domain and severity; incidents approaching staleness threshold |
| **3. Escalation Drill-Down** | Table of all Escalated and Executive Brief incidents with owner and age |
| **4. Domain Detail** | Per-domain filter; full incident list for domain leads |
| **5. Resolution Metrics** | MTD contained/closed; mean time to contain; mean time to close |

### Prerequisites

| Requirement | Notes |
|-------------|-------|
| Power BI Desktop | Free download from Microsoft. Install before starting. |
| Power BI Pro license | Required for publishing and sharing. Included with M365 E3/E5; add-on for Business Standard. |
| SharePoint site deployed | `EIT_Incidents` and `EIT_IncidentHistory` must have live data before connecting. |
| P&C RLS policy decision | Decide RLS approach before publishing. See `EIT-PowerBI-RLS-Guide.md`. |
| Executive approval on KPIs | Confirm the metric definitions in Section 4 with leadership before building. |

---

## Section 1: Connect to SharePoint Data

### 1.1 Get Data — EIT_Incidents

1. Open **Power BI Desktop**
2. Click **Get Data** → **SharePoint Online List**
3. Enter Site URL: `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker`
4. Click **OK** → sign in with your M365 account
5. In the Navigator, check **EIT_Incidents**
6. Click **Transform Data** *(do not click Load directly — you need to shape the data first)*

### 1.2 Shape EIT_Incidents in Power Query

In the Power Query Editor:

1. Remove unnecessary system columns — right-click and delete: `_UIVersionString`, `Attachments`, `AuthorId`, `EditorId`, `OData__UIVersion`, `@odata.etag`, `ItemInternalId`, `ContentTypeId`, `OwshiddenVersion`, `FileSystemObjectType`, `ServerRedirectedEmbedUri`, `ServerRedirectedEmbedUrl`, `ID0` *(keep `ID`)*

2. Expand Choice columns — SharePoint choice columns import as records. Expand each:
   - `Domain` → expand, rename the `.Value` column to `Domain`
   - `Priority` → expand, rename to `Priority`
   - `IncidentType` → expand, rename to `IncidentType`
   - `Severity` → expand, rename to `Severity`
   - `Status` → expand, rename to `Status`
   - `EscalationThreshold` → expand, rename to `EscalationThreshold`
   - `SourceSystem` → expand, rename to `SourceSystem`
   - `ConfidentialityLevel` → expand, rename to `ConfidentialityLevel`
   - `StalenessFlag` → expand, rename to `StalenessFlag`

3. Set data types:
   - `DateRaised`, `LastUpdated`, `DateContained` → **Date/Time**
   - `DaysSinceUpdate` → **Whole Number**
   - `PrivilegedAndConfidential` → **True/False**
   - `ID` → **Whole Number**
   - All other retained columns → **Text**

4. Rename the query: right-click the query in the Queries panel → Rename → `EIT_Incidents`

5. Click **Close & Apply**

### 1.3 Get Data — EIT_IncidentHistory

1. Click **Get Data** → **SharePoint Online List** (same site URL)
2. Check **EIT_IncidentHistory**
3. Click **Transform Data**

In Power Query:

1. Remove system columns (same as above)
2. Expand Choice columns:
   - `StatusAtUpdate` → expand, rename to `StatusAtUpdate`
   - `SeverityAtUpdate` → expand, rename to `SeverityAtUpdate`
   - `UpdateType` → expand, rename to `UpdateType`
3. Set data types:
   - `UpdateDate` → **Date/Time**
   - `PrivilegedAndConfidential` → **True/False**
4. Rename the query: `EIT_IncidentHistory`
5. Click **Close & Apply**

### 1.4 Create the Relationship

1. Click the **Model** view (relationship diagram icon in left rail)
2. Drag `EIT_IncidentHistory[ParentItemID]` onto `EIT_Incidents[ItemID]`
3. Relationship configuration:
   - **Cardinality:** Many to One (many history entries per one incident)
   - **Cross filter direction:** Single (EIT_Incidents filters EIT_IncidentHistory)
4. Click **OK**

---

## Section 2: Create DAX Measures

All measures are created in the **EIT_Incidents** table. Click the **Data** view, select `EIT_Incidents`, then use **New measure** for each.

### 2.1 Core Incident Counts

```dax
Total Open =
COUNTROWS(
    FILTER(
        EIT_Incidents,
        EIT_Incidents[Status] <> "Closed" &&
        EIT_Incidents[Status] <> "Contained"
    )
)
```

```dax
Total Critical =
COUNTROWS(
    FILTER(
        EIT_Incidents,
        EIT_Incidents[Severity] = "Critical" &&
        EIT_Incidents[Status] <> "Closed"
    )
)
```

```dax
Total Escalated =
COUNTROWS(
    FILTER(
        EIT_Incidents,
        EIT_Incidents[EscalationThreshold] <> "Not Escalated" &&
        EIT_Incidents[Status] <> "Closed"
    )
)
```

```dax
Total Executive Brief =
COUNTROWS(
    FILTER(
        EIT_Incidents,
        EIT_Incidents[EscalationThreshold] = "Executive Brief" &&
        EIT_Incidents[Status] <> "Closed"
    )
)
```

```dax
Total Stale =
COUNTROWS(
    FILTER(
        EIT_Incidents,
        EIT_Incidents[StalenessFlag] = "Stale" &&
        EIT_Incidents[Status] <> "Closed"
    )
)
```

```dax
Total PAC Active =
COUNTROWS(
    FILTER(
        EIT_Incidents,
        EIT_Incidents[PrivilegedAndConfidential] = TRUE() &&
        EIT_Incidents[Status] <> "Closed"
    )
)
```

### 2.2 MTD Resolution Metrics

```dax
MTD Contained =
COUNTROWS(
    FILTER(
        EIT_Incidents,
        EIT_Incidents[Status] = "Contained" &&
        EIT_Incidents[DateContained] >= DATE(YEAR(TODAY()), MONTH(TODAY()), 1)
    )
)
```

```dax
MTD Closed =
COUNTROWS(
    FILTER(
        EIT_Incidents,
        EIT_Incidents[Status] = "Closed" &&
        EIT_Incidents[LastUpdated] >= DATE(YEAR(TODAY()), MONTH(TODAY()), 1)
    )
)
```

### 2.3 Mean Time Metrics

```dax
Mean Time to Contain (Days) =
AVERAGEX(
    FILTER(
        EIT_Incidents,
        NOT(ISBLANK(EIT_Incidents[DateContained]))
    ),
    DATEDIFF(EIT_Incidents[DateRaised], EIT_Incidents[DateContained], DAY)
)
```

```dax
Mean Time to Close (Days) =
AVERAGEX(
    FILTER(
        EIT_Incidents,
        EIT_Incidents[Status] = "Closed"
    ),
    DATEDIFF(EIT_Incidents[DateRaised], EIT_Incidents[LastUpdated], DAY)
)
```

### 2.4 Trend Metric (Weekly Incidents Opened)

```dax
Incidents Opened This Week =
COUNTROWS(
    FILTER(
        EIT_Incidents,
        EIT_Incidents[DateRaised] >= (TODAY() - WEEKDAY(TODAY(), 2) + 1)
    )
)
```

---

## Section 3: Build the Report Pages

### Page 1: Executive Overview

**Rename the page:** Double-click the default page tab → type `Executive Overview`.

**Visuals to add:**

#### A. KPI Cards (top row)

Add 5 **Card** visuals across the top (approx Y=20, height=80, width=180 each):

| Card | Measure | Label |
|------|---------|-------|
| 1 | `Total Open` | Open Incidents |
| 2 | `Total Critical` | Critical |
| 3 | `Total Escalated` | Escalated |
| 4 | `Total Stale` | Stale (15+ Days) |
| 5 | `MTD Contained` | Contained (MTD) |

Format each card:
- **Data label** font: Segoe UI, 28pt, Bold
- **Category label** font: Segoe UI, 10pt
- **Background:** White with thin border
- **Conditional formatting** on `Total Critical`: if value > 0, font color = Red (`#E0301E`)
- **Conditional formatting** on `Total Stale`: if value > 0, font color = Dark Red (`#772028`)

#### B. Open Incidents by Domain (Clustered Bar Chart)

1. Add a **Clustered bar chart**
2. **Y-axis:** `Domain` (from EIT_Incidents)
3. **X-axis:** `Total Open` measure
4. **Legend:** `Severity` (from EIT_Incidents)
5. Format:
   - Title: `Open Incidents by Domain`
   - Data colors:
     - Critical: `#772028` (dark red)
     - High: `#E0301E` (red)
     - Moderate: `#E45C2B` (orange)
     - Low: `#415385` (blue)
     - Informational: `#969696` (grey)
   - Filter: Add visual-level filter — `Status is not Closed` and `Status is not Contained`

#### C. Severity Distribution (Donut Chart)

1. Add a **Donut chart**
2. **Values:** `Total Open` measure
3. **Legend:** `Severity`
4. Format:
   - Title: `Severity Distribution (Open)`
   - Same color scheme as chart B
   - Detail labels: on, show percentage

#### D. Escalation Funnel (Funnel Chart)

1. Add a **Funnel chart**
2. **Group:** Create a calculated column or use a manually ordered legend:
   - Layer 1: Total Open → `Total Open`
   - Layer 2: Watch → count where EscalationThreshold = "Watch"
   - Layer 3: Escalate → count where EscalationThreshold = "Escalate"
   - Layer 4: Executive Brief → `Total Executive Brief`

> **Funnel tip:** Power BI's funnel chart uses a single measure across ordered categories. The cleanest approach is to create a summary table in DAX:
>
> ```dax
> Escalation Funnel =
> DATATABLE(
>     "Level", STRING,
>     "SortOrder", INTEGER,
>     {
>         {"Open", 1},
>         {"Watch", 2},
>         {"Escalate", 3},
>         {"Executive Brief", 4}
>     }
> )
> ```
>
> Then create a measure `Funnel Count` that uses SELECTEDVALUE on this table to return the appropriate count. Alternatively, use a **Stacked bar chart** with EscalationThreshold on the axis and Total Open as the value.

#### E. Weekly Trend (Line Chart)

1. Add a **Line chart**
2. **X-axis:** `DateRaised` (set to hierarchy → Week)
3. **Y-axis (Line 1):** Count of incidents opened per week (use `DateRaised` count)
4. **Y-axis (Line 2):** Count of incidents closed per week (use `LastUpdated` count, filter Status = Closed)
5. Format:
   - Title: `Incidents Opened vs. Closed (Weekly)`
   - Line 1 (Opened): Blue `#415385`
   - Line 2 (Closed): Green `#328C50`
   - Data labels: on

---

### Page 2: Staleness View

**Rename page:** `Staleness View`

#### A. Staleness Heatmap (Matrix)

1. Add a **Matrix** visual
2. **Rows:** `Domain`
3. **Columns:** `StalenessFlag` (ordered: Current, Aging, Stale)
4. **Values:** Count of rows (drag `ItemID` into values → set to Count)
5. **Filter:** Status not Closed
6. Format — Conditional formatting on values:
   - Background color by rules:
     - `Stale` column: if value > 0 → Dark Red `#772028`
     - `Aging` column: if value > 0 → Orange `#E45C2B`
     - `Current` column: if value > 0 → Blue `#415385`
   - All zero values: Grey `#EFEFEF`

#### B. Incidents Approaching Staleness (Table)

Show incidents that are in the "Aging" band (8–14 days) — these are at risk of becoming Stale before the next review.

1. Add a **Table** visual
2. Columns: `ItemID`, `Title`, `Domain`, `Severity`, `Owner`, `DaysSinceUpdate`
3. **Visual-level filter:**
   - `StalenessFlag = Aging`
   - `Status` is not `Closed`
4. **Sort:** `DaysSinceUpdate` descending (highest days first)
5. **Conditional formatting** on `DaysSinceUpdate`: data bars, orange color
6. Title: `Incidents Approaching Staleness (8–14 Days)`

#### C. Stale Incidents by Severity (Stacked Bar)

1. Add a **Clustered bar chart**
2. **Y-axis:** `Domain`
3. **X-axis:** Count of stale incidents
4. **Legend:** `Severity`
5. **Filter:** `StalenessFlag = Stale`
6. Title: `Stale Incidents by Domain and Severity`

---

### Page 3: Escalation Drill-Down

**Rename page:** `Escalation Drill-Down`

#### A. Escalated Incidents Table

1. Add a **Table** visual (large, filling most of the page)
2. Columns:
   - `ItemID` (hyperlink if possible — see tip below)
   - `Title`
   - `Domain`
   - `Severity`
   - `EscalationThreshold`
   - `Owner`
   - `DaysSinceUpdate`
   - `LastUpdated`
3. **Visual-level filter:**
   - `EscalationThreshold` is not `Not Escalated`
   - `Status` is not `Closed`
4. **Sort:** `EscalationThreshold` (Executive Brief first, then Escalate, then Watch), then `DaysSinceUpdate` descending
5. **Conditional formatting:**
   - `EscalationThreshold`: background color — Executive Brief = Dark Red, Escalate = Red, Watch = Orange
   - `DaysSinceUpdate`: data bars, red gradient
   - `Severity`: font color — Critical = Dark Red, High = Red, Moderate = Orange

> **Hyperlink tip:** To make `ItemID` a clickable link to the EIT Power App, create a calculated column:
> ```dax
> ItemURL =
> "https://make.powerapps.com/environments/[AUTO-GENERATED-ENV-ID]/apps/[AUTO-GENERATED-APP-ID]"
> ```
> Then set the `ItemID` column URL to `ItemURL`. This creates a deep-link to the Power App (but does not deep-link to a specific incident, since Power Apps URLs don't support that natively at Phase 2 stage).

#### B. Summary KPI Cards (top of page)

Add 3 small cards above the table:
- `Total Escalated` — "Total Escalated"
- `Total Executive Brief` — "Executive Brief"
- `Total Stale` — "Stale"

---

### Page 4: Domain Detail

**Rename page:** `Domain Detail`

#### A. Domain Slicer

1. Add a **Slicer** visual
2. **Field:** `Domain`
3. **Style:** Dropdown or vertical list
4. Position in top-left corner

#### B. Domain Incident Table

1. Add a **Table** visual
2. Columns: `ItemID`, `Title`, `IncidentType`, `Severity`, `Status`, `EscalationThreshold`, `Owner`, `DaysSinceUpdate`, `LastUpdated`, `SourceSystem`
3. **Filter:** `Status` is not `Closed` (unless a "Show Closed" toggle is added)
4. **Sort:** `DaysSinceUpdate` descending
5. All conditional formatting from Page 3 applies

#### C. Domain KPI Summary Cards

Add 4 cards below the slicer (left column):
- Open incidents in selected domain
- Critical in selected domain
- Stale in selected domain
- Escalated in selected domain

These should react to the Domain slicer automatically since they use the same EIT_Incidents table.

#### D. Status Distribution (Donut Chart)

1. Add a **Donut chart**
2. **Values:** Count of ItemID
3. **Legend:** `Status`
4. Reacts to Domain slicer
5. Status colors: New=Blue, Active=Orange, Monitoring=Orange (lighter), Escalated=Red, Contained=Green

---

### Page 5: Resolution Metrics

**Rename page:** `Resolution Metrics`

#### A. MTD KPI Cards (top row)

4 cards across the top:
- `MTD Contained` — "Contained (MTD)"
- `MTD Closed` — "Closed (MTD)"
- `Mean Time to Contain (Days)` — "Avg Days to Contain"
- `Mean Time to Close (Days)` — "Avg Days to Close"

#### B. Monthly Trend — Contained and Closed (Grouped Bar)

1. Add a **Clustered column chart**
2. **X-axis:** `DateContained` (for Contained) / `LastUpdated` (for Closed) — use month hierarchy
3. **Y-axis:** Count of incidents
4. **Legend:** `Status` (filtered to Contained and Closed only)
5. Title: `Monthly Resolution Activity`

> **Two-metric approach:** Since Contained uses `DateContained` and Closed uses `LastUpdated`, you may need two separate measures on the Y-axis with the month on the X-axis using DateRaised as a shared date dimension. Alternatively, create a calculated column `ResolutionDate = IF(Status = "Contained", DateContained, IF(Status = "Closed", LastUpdated, BLANK()))` and use it as the X-axis date field.

#### C. Mean Time to Contain by Domain (Bar Chart)

1. Add a **Clustered bar chart**
2. **Y-axis:** `Domain`
3. **X-axis:** `Mean Time to Contain (Days)` measure
4. **Filter:** Status = "Contained" or "Closed" (incidents that have been contained)
5. Title: `Average Days to Contain by Domain`

#### D. Source System Breakdown (Donut Chart)

1. Add a **Donut chart**
2. **Values:** Count of ItemID
3. **Legend:** `SourceSystem`
4. Title: `Incidents by Source System`
5. Useful for tracking what proportion of incidents originated from ServiceNow vs iSight vs manual entry

---

## Section 4: Publish to Power BI Service

### 4.1 Publish the Report

1. In Power BI Desktop, click **File** → **Publish** → **Publish to Power BI**
2. Select or create the **Enterprise Incident Tracker** workspace
3. Click **Select**

### 4.2 Configure Scheduled Data Refresh

1. In Power BI Service (`https://app.powerbi.com`), navigate to the **Enterprise Incident Tracker** workspace
2. Find the **EIT-Dashboard** dataset
3. Click **Settings**
4. Under **Data source credentials**: click **Edit credentials** for SharePoint → sign in with the admin M365 account
5. Under **Scheduled refresh**:
   - Toggle **On**
   - Add two refresh times: `06:00` and `18:00` (local time or UTC — confirm with your admin)
   - Timezone: set to your organisation's timezone
6. Click **Apply**

> **Refresh license note:** Scheduled refresh requires Power BI Pro or Premium. The maximum refresh frequency on Power BI Pro is 8 times per day; twice-daily is well within this limit.

### 4.3 Share the Report

1. Open the **EIT-Dashboard** report in Power BI Service
2. Click **Share**
3. Add the `EIT Members` and `EIT Viewers` M365 groups
4. Permission: **Can view** (do not grant edit access)
5. Uncheck "Allow recipients to share this report" for EIT Viewers
6. Click **Send**

---

## Section 5: Embed in Microsoft Teams

### Option A — Teams Power BI Tab

1. Open the **Enterprise Incident Tracker** Teams team
2. Navigate to the `General` channel (or create a dedicated `EIT-Reports` channel)
3. Click **+** (Add a tab)
4. Select **Power BI**
5. Select the **EIT-Dashboard** report from the **Enterprise Incident Tracker** workspace
6. Click **Save**

The report is now accessible as a tab in Teams without leaving the Teams client.

### Option B — SharePoint Webpart

1. Navigate to the EIT SharePoint site home page
2. Click **Edit** → **Add a new web part** → **Power BI**
3. Select the EIT-Dashboard report
4. Resize and position the webpart
5. Click **Publish**

> **Licence requirement for embedding:** All users viewing the embedded report must have a Power BI Pro license, or the workspace must be in a Premium capacity. Verify your licensing before embedding.

---

## Section 6: Report Maintenance

| Task | Frequency | Who |
|------|-----------|-----|
| Review data refresh logs for failures | Weekly | EIT admin |
| Re-authenticate SharePoint connection if it expires | As needed (90-day token) | EIT admin |
| Add new domain (if EIT expands) | On demand | Power BI developer |
| Update KPI measure definitions | On demand (requires PBIX edit + re-publish) | Power BI developer |
| Re-apply RLS after model changes | After each publish | EIT admin |
| Review P&C exclusion accuracy | Monthly | EIT admin + legal |

---

## Appendix: DAX Quick Reference

| Measure | Purpose |
|---------|---------|
| `Total Open` | Active incident count (excludes Closed and Contained) |
| `Total Critical` | Critical open incidents |
| `Total Escalated` | Open incidents with any escalation level |
| `Total Executive Brief` | Open incidents at Executive Brief level |
| `Total Stale` | Open incidents with StalenessFlag = Stale |
| `Total PAC Active` | Open P&C-flagged incidents |
| `MTD Contained` | Incidents contained in current month |
| `MTD Closed` | Incidents closed in current month |
| `Mean Time to Contain (Days)` | Average days from DateRaised to DateContained |
| `Mean Time to Close (Days)` | Average days from DateRaised to LastUpdated (Closed only) |
