# EIT Daily Staleness and Escalation Calculator — Flow Rebuild Guide

> **Prerequisite:** The SharePoint site and all 4 lists must exist before creating this flow. See `01-SharePoint/EIT-CREATE-SITE.md`.

---

## Flow Overview

| Property | Value |
|----------|-------|
| Name | `EIT Daily Staleness and Escalation Calculator` |
| Type | Scheduled cloud flow |
| Trigger | Recurrence — Every 1 day at 06:00 AM UTC |
| Purpose | Calculates `DaysSinceUpdate`, sets `StalenessFlag`, and updates `EscalationThreshold` for all open incidents |
| Connection | SharePoint (your admin account) |

### Changes from OFR Daily Staleness Calculator

| Change | Detail |
|--------|--------|
| List name | `OFR_Issues` → `EIT_Incidents` |
| Site URL | `OFRIssueTracker` → `EnterpriseIncidentTracker` |
| Filter query | `Status ne 'Closed'` → excludes both `Closed` and `Contained` |
| New field | `EscalationThreshold` — calculated from Severity + DaysSinceUpdate combination |
| Expressions reused verbatim | `staleness-DaysSinceUpdate.txt`, `staleness-StalenessFlag.txt` |

### Staleness Thresholds (unchanged from OFR)

| DaysSinceUpdate | StalenessFlag |
|-----------------|---------------|
| 0–7 | Current |
| 8–14 | Aging |
| 15+ | Stale |

### Escalation Threshold Logic (new)

| Condition | EscalationThreshold set to |
|-----------|---------------------------|
| Severity = Critical AND DaysSinceUpdate > 14 | `Executive Brief` |
| Severity = High AND DaysSinceUpdate > 7 | `Escalate` |
| All other combinations | Existing value preserved (no change) |

> **Important:** This flow only auto-escalates based on age. Manual escalation set by users in the Power App is preserved unless the age+severity rule is triggered. The flow will never auto-downgrade an escalation threshold.

---

## Step-by-Step Build Instructions

### 1. Create the Flow

1. Navigate to `https://make.powerautomate.com`
2. Click **+ Create** → **Scheduled cloud flow**
3. Flow name: `EIT Daily Staleness and Escalation Calculator`
4. Starting: **Today's date**
5. Repeat every: **1 Day**
6. Click **Create**

### 2. Configure the Trigger — Recurrence

1. Click the **Recurrence** trigger
2. Set:
   - **Interval:** `1`
   - **Frequency:** `Day`
   - **Time zone:** `(UTC) Coordinated Universal Time`
   - **At these hours:** `6`
   - **At these minutes:** `0`

### 3. Add Action — Get Items

1. Click **+ New step**
2. Search for **SharePoint** → select **Get items**
3. Configure:
   - **Site Address:** `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker`
   - **List Name:** `EIT_Incidents`
   - Click **Show advanced options**
   - **Filter Query:** `Status ne 'Closed' and Status ne 'Contained'`
   - **Top Count:** `500` *(enterprise volume — adjust higher if needed)*

> **Note:** The filter excludes both `Closed` and `Contained` incidents, as these are terminal states where staleness tracking is no longer meaningful. This is an extension of the OFR pattern which only excluded `Closed`.

### 4. Add Action — Apply to Each

1. Click **+ New step**
2. Search for **Control** → select **Apply to each**
3. In the **Select an output from previous steps** field, select: `body/value` (from the Get items action)

### 5. Add Action (Inside Loop) — Update Item

1. Inside the Apply to each loop, click **Add an action**
2. Search for **SharePoint** → select **Update item**
3. Configure:
   - **Site Address:** `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker`
   - **List Name:** `EIT_Incidents`
   - **Id:** Click the field → switch to **Expression** tab → enter:
     ```
     items('Apply_to_each')?['ID']
     ```
     Click **OK**
   - **Title:** Dynamic content → select `Title` from the Apply to each context

### 6. Set DaysSinceUpdate Expression

In the Update item action:

1. Click the **DaysSinceUpdate** field
2. Switch to the **Expression** tab
3. Paste this expression exactly (also available in `flow-expressions/staleness-DaysSinceUpdate.txt` — reused verbatim from OFR):

```
div(sub(ticks(utcNow()),ticks(items('Apply_to_each')?['LastUpdated'])),864000000000)
```

4. Click **OK**

### 7. Set StalenessFlag Expression

In the Update item action:

1. Click the **StalenessFlag Value** field
2. Switch to the **Expression** tab
3. Paste this expression exactly (also available in `flow-expressions/staleness-StalenessFlag.txt` — reused verbatim from OFR):

```
if(lessOrEquals(div(sub(ticks(utcNow()),ticks(items('Apply_to_each')?['LastUpdated'])),864000000000),7),'Current',if(lessOrEquals(div(sub(ticks(utcNow()),ticks(items('Apply_to_each')?['LastUpdated'])),864000000000),14),'Aging','Stale'))
```

4. Click **OK**

### 8. Set EscalationThreshold Expression (new — not in OFR)

In the Update item action:

1. Click the **EscalationThreshold Value** field
2. Switch to the **Expression** tab
3. Paste this expression exactly (also available in `flow-expressions/EIT-escalation-Threshold.txt`):

```
if(and(greater(div(sub(ticks(utcNow()),ticks(items('Apply_to_each')?['LastUpdated'])),864000000000),14),equals(items('Apply_to_each')?['Severity']?['Value'],'Critical')),'Executive Brief',if(and(greater(div(sub(ticks(utcNow()),ticks(items('Apply_to_each')?['LastUpdated'])),864000000000),7),equals(items('Apply_to_each')?['Severity']?['Value'],'High')),'Escalate',items('Apply_to_each')?['EscalationThreshold']?['Value']))
```

4. Click **OK**

**How it works:**
- If `DaysSinceUpdate > 14` AND `Severity = Critical` → set `Executive Brief`
- Else if `DaysSinceUpdate > 7` AND `Severity = High` → set `Escalate`
- Else → preserve the existing `EscalationThreshold` value (no change)

> **Troubleshooting note:** If the Severity comparison fails, try replacing `items('Apply_to_each')?['Severity']?['Value']` with `items('Apply_to_each')?['Severity']` (without `?['Value']`). The nested property path depends on how your SharePoint connector returns Choice fields. Test with the sample data to confirm.

### 9. Save and Test

1. Click **Save** (top-right)
2. Click **Test** → **Manually** → **Test**
3. Wait for the flow to complete (~60 seconds for 15 sample incidents)
4. Verify results in the `EIT_Incidents` SharePoint list:
   - `DaysSinceUpdate` values are correct integers for each incident
   - `StalenessFlag` values match the threshold table above
   - `EscalationThreshold` for EIT-003 (Critical, 3 days) remains `Executive Brief` (preserved — not overwritten because days ≤ 14)
   - `EscalationThreshold` for EIT-002 (Moderate, 16 days stale) remains `Watch` (preserved — Moderate is not in the auto-escalation rules)
   - Re-run after manually setting an incident's `LastUpdated` to 20 days ago with `Severity = Critical` — confirm it becomes `Executive Brief`

### 10. Turn On the Flow

1. After successful test, the flow runs automatically daily at 06:00 UTC
2. Click the flow name breadcrumb to return to the flow details page
3. Verify **Status** is **On**

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| "Item not found" errors | Verify site address is `EnterpriseIncidentTracker` (not `OFRIssueTracker`) and list name is `EIT_Incidents` |
| Expression errors on LastUpdated | Ensure `LastUpdated` has values in all rows — empty dates cause tick conversion failures |
| EscalationThreshold not updating | Verify the `Severity` column internal name is exactly `Severity` (case-sensitive). If the column was renamed after creation, the internal name may differ. |
| `Severity` comparison always false | Remove the `?['Value']` suffix from the Severity reference in the expression — try `items('Apply_to_each')?['Severity']` instead |
| Timeout on large lists | Increase **Top Count** in Get items and enable pagination (Settings → Pagination → On, Threshold: 5000) |
| Flow runs but EscalationThreshold reverts to old value | Check the expression is returning the existing value correctly in the else branch — the `?['EscalationThreshold']?['Value']` reference must match the column's internal name exactly |

---

## Flow Diagram

```
Recurrence (Daily 06:00 UTC)
    |
    v
Get items: EIT_Incidents
  Filter: Status ne 'Closed' and Status ne 'Contained'
  Top Count: 500
    |
    v
Apply to each (body/value)
    |
    +---> Update item: EIT_Incidents
              ID:                  items('Apply_to_each')?['ID']
              DaysSinceUpdate:     [Expression A — days calculation]
              StalenessFlag Value: [Expression B — Current/Aging/Stale]
              EscalationThreshold: [Expression C — Executive Brief/Escalate/preserve]
```
