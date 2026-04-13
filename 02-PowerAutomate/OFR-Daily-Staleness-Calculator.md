# OFR Daily Staleness Calculator — Flow Rebuild Guide

> **Prerequisite:** The SharePoint site and all 3 lists must exist before creating this flow. See `01-SharePoint/CREATE-SITE.md`.

---

## Flow Overview

| Property | Value |
|----------|-------|
| Name | `OFR Daily Staleness Calculator` |
| Type | Scheduled cloud flow |
| Trigger | Recurrence — Every 1 day at 06:00 AM UTC |
| Purpose | Calculates `DaysSinceUpdate` and sets `StalenessFlag` for all non-closed issues |
| Connection | SharePoint (your admin account) |

### Staleness Thresholds

| DaysSinceUpdate | StalenessFlag |
|-----------------|---------------|
| 0 - 7 | Current |
| 8 - 14 | Aging |
| 15+ | Stale |

---

## Step-by-Step Build Instructions

### 1. Create the Flow

1. Navigate to `https://make.powerautomate.com`
2. Click **+ Create** → **Scheduled cloud flow**
3. Flow name: `OFR Daily Staleness Calculator`
4. Starting: **Today's date**
5. Repeat every: **1 Day**
6. Click **Create**

### 2. Configure the Trigger — Recurrence

The trigger is auto-created. Modify it:

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
   - **Site Address:** `https://[TENANT].sharepoint.com/sites/OFRIssueTracker`
   - **List Name:** `OFR_Issues`
   - Click **Show advanced options**
   - **Filter Query:** `Status ne 'Closed'`
   - **Top Count:** `100` *(handles up to 100 open issues; increase if needed)*

### 4. Add Action — Apply to Each

1. Click **+ New step**
2. Search for **Control** → select **Apply to each**
3. In the **Select an output from previous steps** field, select: `body/value` (from the Get items action)

### 5. Add Action (Inside Loop) — Update Item

1. Inside the Apply to each loop, click **Add an action**
2. Search for **SharePoint** → select **Update item**
3. Configure:
   - **Site Address:** `https://[TENANT].sharepoint.com/sites/OFRIssueTracker`
   - **List Name:** `OFR_Issues`
   - **Id:** Click the field, then switch to **Expression** tab and enter:
     ```
     items('Apply_to_each')?['ID']
     ```
     Click **OK**.
   - **Title:** Click Dynamic content → select `Title` from the Apply to each context

### 6. Set DaysSinceUpdate Expression

In the Update item action:

1. Click the **DaysSinceUpdate** field
2. Switch to the **Expression** tab
3. Paste this expression exactly (also available in `flow-expressions/staleness-DaysSinceUpdate.txt`):

```
div(sub(ticks(utcNow()),ticks(items('Apply_to_each')?['LastUpdated'])),864000000000)
```

4. Click **OK**

**How it works:**
- `ticks(utcNow())` — current time in ticks (100-nanosecond intervals since epoch)
- `ticks(items('Apply_to_each')?['LastUpdated'])` — LastUpdated time in ticks
- `sub(...)` — difference in ticks
- `div(..., 864000000000)` — convert ticks to days (864 billion ticks = 1 day)

### 7. Set StalenessFlag Expression

In the Update item action:

1. Click the **StalenessFlag Value** field
2. Switch to the **Expression** tab
3. Paste this expression exactly (also available in `flow-expressions/staleness-StalenessFlag.txt`):

```
if(lessOrEquals(div(sub(ticks(utcNow()),ticks(items('Apply_to_each')?['LastUpdated'])),864000000000),7),'Current',if(lessOrEquals(div(sub(ticks(utcNow()),ticks(items('Apply_to_each')?['LastUpdated'])),864000000000),14),'Aging','Stale'))
```

4. Click **OK**

**How it works:**
- If days since update <= 7 → `Current`
- Else if days since update <= 14 → `Aging`
- Else → `Stale`

### 8. Save and Test

1. Click **Save** (top-right)
2. Click **Test** → **Manually** → **Test**
3. Wait for the flow to complete (~30-60 seconds for 8 sample issues)
4. Verify results:
   - Open `OFR_Issues` list in SharePoint
   - Check that `DaysSinceUpdate` values are correct integers
   - Check that `StalenessFlag` values match the thresholds above

### 9. Turn On the Flow

1. After successful test, the flow will run automatically daily at 06:00 UTC
2. Click the flow name breadcrumb to go back to the flow details page
3. Verify **Status** is **On**

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| "Item not found" errors | Verify the SharePoint site address and list name match exactly |
| Expression errors | Ensure `LastUpdated` column has values in all rows; empty dates will cause tick conversion failures |
| Flow runs but values don't update | Check that the column internal names match (`DaysSinceUpdate`, `StalenessFlag`). SharePoint may use different internal names if columns were renamed. |
| Timeout on large lists | Increase **Top Count** in Get items. For 100+ items, consider pagination settings. |

---

## Flow Diagram

```
Recurrence (Daily 06:00 UTC)
    |
    v
Get items: OFR_Issues (Status ne 'Closed')
    |
    v
Apply to each (body/value)
    |
    +---> Update item: OFR_Issues
              ID: items('Apply_to_each')?['ID']
              DaysSinceUpdate: [Expression A - days calculation]
              StalenessFlag: [Expression B - threshold logic]
```
