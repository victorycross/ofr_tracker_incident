# EIT Compliance Report Flow — Rebuild Guide

> **Phase 3 Integration.** This flow generates a monthly compliance metrics report from EIT data and distributes it to the compliance officer, CISO, and domain leads. See `EIT-Phase3-Compliance-Guide.md` for how to use this report in regulatory submissions.

---

## Flow Overview

| Property | Value |
|----------|-------|
| Name | `EIT Monthly Compliance Report` |
| Type | Scheduled cloud flow |
| Trigger | Recurrence (monthly, 1st of month, 07:00 UTC) |
| Purpose | Calculates monthly compliance metrics from EIT_Incidents + EIT_IncidentHistory and emails a formatted report to the compliance distribution list |
| Connections | SharePoint (admin account) + Outlook (admin account) |

### What This Flow Does

1. Runs on the 1st of each month at 07:00 UTC
2. Queries `EIT_Incidents` and `EIT_IncidentHistory` for the previous calendar month's data
3. Calculates 8 compliance metrics (see Metrics section)
4. Formats a structured report email
5. Sends to: Compliance Officer, CISO (`[EXEC-ESCALATION-EMAIL]`), and EIT administrator
6. Optionally: saves the report as a text file to a SharePoint Compliance Records library

---

## Metrics Calculated

| Metric | Definition | Source |
|--------|-----------|--------|
| **Total Incidents Raised (MTD)** | New incidents created in the previous month | EIT_Incidents.DateRaised in previous month |
| **Total Incidents Resolved (MTD)** | Incidents moved to Closed in the previous month | EIT_Incidents.Status = Closed, LastUpdated in previous month |
| **Total Incidents Contained (MTD)** | Incidents moved to Contained in the previous month | EIT_Incidents.DateContained in previous month |
| **Critical Incidents (MTD)** | Incidents with Severity = Critical raised in the previous month | EIT_Incidents.Severity = Critical, DateRaised in previous month |
| **Escalated Incidents (MTD)** | Incidents with any EscalationThreshold ≠ Not Escalated at month-end | EIT_Incidents active with escalation |
| **P&C Incidents Active** | Incidents with PrivilegedAndConfidential = Yes that are not Closed | EIT_Incidents.PrivilegedAndConfidential = Yes, Status ≠ Closed |
| **Stale Incidents (Month-End)** | Open incidents with StalenessFlag = Stale at end of month | EIT_Incidents.StalenessFlag = Stale, Status ≠ Closed |
| **Mean Time to Contain (Days)** | Average days from DateRaised to DateContained for incidents contained in the month | EIT_Incidents with DateContained in previous month |

---

## Step-by-Step Build Instructions

### 1. Create the Flow

1. Navigate to `https://make.powerautomate.com`
2. Click **+ Create** → **Scheduled cloud flow**
3. Flow name: `EIT Monthly Compliance Report`
4. Schedule:
   - **Starting:** 1st of next month at 07:00 UTC
   - **Repeat every:** `1 Month`
5. Click **Create**

---

### 2. Initialize Date Variables

Add the following Initialize variable actions in sequence (not in parallel — each must complete before the next):

#### 2a. Month Start Date

1. **+ New step** → **Variables** → **Initialize variable**
2. Name: `varMonthStart`, Type: String
3. Value: Expression →
   ```
   formatDateTime(startOfMonth(addDays(utcNow(), -1)), 'yyyy-MM-ddT00:00:00Z')
   ```

> **Why `-1` day:** The flow runs on the 1st of the month. `addDays(utcNow(), -1)` steps back into the previous month, then `startOfMonth()` returns the 1st of that month. This reliably captures the previous month's start regardless of run time.

#### 2b. Month End Date

1. **+ New step** → **Variables** → **Initialize variable**
2. Name: `varMonthEnd`, Type: String
3. Value: Expression →
   ```
   formatDateTime(addDays(startOfMonth(utcNow()), -1), 'yyyy-MM-ddT23:59:59Z')
   ```

#### 2c. Report Month Label

1. **+ New step** → **Variables** → **Initialize variable**
2. Name: `varReportMonth`, Type: String
3. Value: Expression →
   ```
   formatDateTime(addDays(startOfMonth(utcNow()), -1), 'MMMM yyyy')
   ```
   This produces a human-readable label like `January 2026`.

---

### 3. Get All Incidents (for Month-End Snapshot)

1. **+ New step** → **SharePoint** → **Get items**
2. Rename to: `Get all incidents`
3. Configure:
   - **Site Address:** `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker`
   - **List Name:** `EIT_Incidents`
   - **Top Count:** `5000`
   - No filter — retrieve all items (both open and recently closed)

> **Why no filter:** Computing multiple metrics across different filter combinations is simpler when working from a full in-memory dataset. With expected incident volumes under 5,000 total, this is performant. If volume exceeds 5,000, add targeted Get items actions for each metric instead.

---

### 4. Calculate Metrics Using Filter Array Actions

For each metric, add a **Filter array** action on the `value` output from Step 3. Each filter produces an array of matching incidents; the count of that array is the metric value.

#### 4a. Total Incidents Raised (MTD)

1. **+ New step** → **Data Operation** → **Filter array**
2. Rename: `Incidents raised MTD`
3. **From:** `value` from Get all incidents
4. **Condition (Advanced mode):**
   ```
   @and(greaterOrEquals(item()?['DateRaised'], variables('varMonthStart')), lessOrEquals(item()?['DateRaised'], variables('varMonthEnd')))
   ```

#### 4b. Incidents Resolved (Closed, MTD)

1. **Filter array** → rename: `Incidents closed MTD`
2. **Condition (Advanced mode):**
   ```
   @and(equals(item()?['Status']?['Value'], 'Closed'), greaterOrEquals(item()?['LastUpdated'], variables('varMonthStart')), lessOrEquals(item()?['LastUpdated'], variables('varMonthEnd')))
   ```

#### 4c. Incidents Contained (MTD)

1. **Filter array** → rename: `Incidents contained MTD`
2. **Condition (Advanced mode):**
   ```
   @and(not(equals(item()?['DateContained'], null)), greaterOrEquals(item()?['DateContained'], variables('varMonthStart')), lessOrEquals(item()?['DateContained'], variables('varMonthEnd')))
   ```

#### 4d. Critical Incidents Raised (MTD)

1. **Filter array** → rename: `Critical incidents MTD`
2. **Condition (Advanced mode):**
   ```
   @and(equals(item()?['Severity']?['Value'], 'Critical'), greaterOrEquals(item()?['DateRaised'], variables('varMonthStart')), lessOrEquals(item()?['DateRaised'], variables('varMonthEnd')))
   ```

#### 4e. Escalated Incidents (Month-End Active)

1. **Filter array** → rename: `Escalated active`
2. **Condition (Advanced mode):**
   ```
   @and(not(equals(item()?['EscalationThreshold']?['Value'], 'Not Escalated')), not(equals(item()?['Status']?['Value'], 'Closed')))
   ```

#### 4f. P&C Active Incidents

1. **Filter array** → rename: `PAC active`
2. **Condition (Advanced mode):**
   ```
   @and(equals(item()?['PrivilegedAndConfidential'], true), not(equals(item()?['Status']?['Value'], 'Closed')))
   ```

#### 4g. Stale Incidents (Month-End)

1. **Filter array** → rename: `Stale active`
2. **Condition (Advanced mode):**
   ```
   @and(equals(item()?['StalenessFlag']?['Value'], 'Stale'), not(equals(item()?['Status']?['Value'], 'Closed')))
   ```

---

### 5. Calculate Mean Time to Contain

The mean time to contain requires averaging across contained incidents. Power Automate does not natively average an array — use a loop with a running total.

#### 5a. Initialize Variables

Add two Initialize variable actions:
- `varMTTCTotal` — Integer — value `0` (running sum of days to contain)
- `varMTTCCount` — Integer — value `0` (count of contained incidents with date data)

#### 5b. Apply to Each — Contained Incidents

1. **+ New step** → **Control** → **Apply to each**
2. Rename: `Calculate MTTC`
3. **Select output:** `Body` from `Incidents contained MTD` (Filter array 4c)

Inside the loop:

**Condition:** Check that DateContained and DateRaised are both not null:
```
@and(not(equals(items('Calculate_MTTC')?['DateContained'], null)), not(equals(items('Calculate_MTTC')?['DateRaised'], null)))
```

**If yes:**
1. **Compose** → rename: `Days to contain this incident`
   Expression:
   ```
   div(sub(ticks(items('Calculate_MTTC')?['DateContained']), ticks(items('Calculate_MTTC')?['DateRaised'])), 864000000000)
   ```
   *(Divides tick difference by 864000000000 to convert ticks to days)*

2. **Increment variable** → `varMTTCTotal` by `outputs('Days_to_contain_this_incident')`
3. **Increment variable** → `varMTTCCount` by `1`

#### 5c. Compose — Final MTTC

After the Apply to each loop:

1. **+ New step** → **Data Operation** → **Compose**
2. Rename: `Compute MTTC`
3. Expression:
   ```
   if(greater(variables('varMTTCCount'), 0), string(div(variables('varMTTCTotal'), variables('varMTTCCount'))), 'N/A')
   ```

---

### 6. Compose — Full Report Body

1. **+ New step** → **Data Operation** → **Compose**
2. Rename: `Build report body`
3. Build the report text using `concat()` with all metric counts:

```
concat(
'ENTERPRISE INCIDENT TRACKER — MONTHLY COMPLIANCE REPORT',
'\n', 'Report period: ', variables('varReportMonth'),
'\n', 'Generated: ', formatDateTime(utcNow(), 'dddd d MMMM yyyy HH:mm UTC'),
'\n\n',
'━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━',
'\n', 'INCIDENT ACTIVITY (', variables('varReportMonth'), ')',
'\n', '━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━',
'\n', 'Total incidents raised:        ', string(length(body('Incidents_raised_MTD'))),
'\n', 'Total incidents contained:     ', string(length(body('Incidents_contained_MTD'))),
'\n', 'Total incidents resolved:      ', string(length(body('Incidents_closed_MTD'))),
'\n', 'Critical incidents raised:     ', string(length(body('Critical_incidents_MTD'))),
'\n\n',
'━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━',
'\n', 'MONTH-END SNAPSHOT',
'\n', '━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━',
'\n', 'Escalated incidents (active):  ', string(length(body('Escalated_active'))),
'\n', 'Privileged & Confidential (active): ', string(length(body('PAC_active'))),
'\n', 'Stale incidents (15+ days):    ', string(length(body('Stale_active'))),
'\n\n',
'━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━',
'\n', 'RESPONSE EFFECTIVENESS',
'\n', '━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━',
'\n', 'Mean time to contain (days):   ', outputs('Compute_MTTC'),
'\n\n',
'━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━',
'\n', 'NOTES',
'\n', '━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━',
'\n', '• P&C incident count reflects incidents visible to the EIT administrator.',
'\n', '  Domain leads should verify P&C counts with Legal Counsel independently.',
'\n', '• This report is generated automatically from EIT SharePoint data.',
'\n', '  For detailed incident lists, export EIT_Incidents from SharePoint.',
'\n', '• Mean time to contain is calculated only for incidents with DateContained populated.'
)
```

---

### 7. Send Report Email

1. **+ New step** → **Outlook** → **Send an email (V2)**
2. Configure:
   - **To:** `david@brightpathtechnology.io`
   - **CC:** `[EXEC-ESCALATION-EMAIL]` *(Compliance Officer / CISO)*
   - **Subject:** Expression →
     ```
     concat('EIT Monthly Compliance Report — ', variables('varReportMonth'))
     ```
   - **Body:** Dynamic content → `Outputs` from `Build report body`
   - **Importance:** Normal

---

### 8. (Optional) Save Report to SharePoint

To create a searchable archive of monthly compliance reports:

1. **+ New step** → **SharePoint** → **Create file**
2. Configure:
   - **Site Address:** `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker`
   - **Folder Path:** `/Compliance Reports` *(create this document library first)*
   - **File Name:** Expression →
     ```
     concat('EIT-Compliance-', formatDateTime(addDays(startOfMonth(utcNow()), -1), 'yyyy-MM'), '.txt')
     ```
   - **File Content:** Dynamic content → `Outputs` from `Build report body`

---

### 9. Save and Test

1. Click **Save**
2. Click **Test** → **Manually** → **Run flow**
3. Verify:
   - Compliance report email received with all 8 metrics populated
   - Metric values cross-checked manually against SharePoint list filters
   - Report month label is correct (previous month)
   - If SP file creation enabled: file appears in `/Compliance Reports` library
4. For month-end testing: create test incidents with `DateRaised`, `DateContained` in a prior month and verify they appear in the report

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Report month is wrong | Verify `addDays(utcNow(), -1)` correctly steps back. Test on the 1st of month only — if testing on any other day the logic will give unexpected results. |
| Filter array returns unexpected counts | Check date filter expressions — SharePoint date strings must match ISO 8601 format. Use a Compose to inspect the `varMonthStart` and `varMonthEnd` values. |
| MTTC shows N/A | No incidents were contained in the previous month. This is correct if `varMTTCCount = 0`. |
| P&C count lower than expected | The flow reads all EIT_Incidents including those with item-level restrictions. If the admin account lacks item-level access to some P&C incidents, those won't be counted. Ensure the flow's SharePoint connection account has Full Control on the site. |
| Email formatting lost | Switch the email body to HTML format and add `<br>` tags for line breaks. The Outlook V2 connector supports HTML bodies. |
