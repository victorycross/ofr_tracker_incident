# EIT iSight Polling Flow — Rebuild Guide

> **Phase 2 Integration.** This flow is part of the Phase 2 integration layer. Phase 1 must be fully deployed and tested before building this flow. See `04-Documentation/EIT-Phase2-Integration-Roadmap.md`.

> **Prerequisite:** iSight API access credentials (API key or OAuth client credentials) must be obtained from the NIS team before building this flow. The NIS domain lead must also confirm the severity threshold above which alerts should flow to EIT.

---

## Flow Overview

| Property | Value |
|----------|-------|
| Name | `EIT iSight Alert Polling` |
| Type | Automated cloud flow (scheduled) |
| Trigger | Recurrence (every 15 minutes) |
| Purpose | Polls iSight for new threat alerts above a defined severity threshold and creates corresponding incidents in EIT_Incidents for NIS domain review |
| Connections | SharePoint (admin account) + HTTP connector (iSight API) |
| Domain | NIS (all iSight alerts are routed to the NIS domain by default) |

### What This Flow Does

1. Runs every 15 minutes
2. Calculates a lookback window (alerts since 15 minutes ago)
3. Calls the iSight REST API to retrieve alerts within that window
4. Filters for alerts at or above the configured severity threshold
5. For each qualifying alert: checks for an existing EIT record (deduplication), and if new, creates an item in `EIT_Incidents` and an opening entry in `EIT_IncidentHistory`

### Recommended Flow to EIT_IntakeQueue vs EIT_Incidents

The Phase 2 roadmap recommends routing iSight alerts to `EIT_IntakeQueue` first, allowing the NIS domain lead to triage before the alert becomes a formal incident. This guide documents the direct-to-`EIT_Incidents` pattern as an alternative. Choose based on your NIS team's preference:

| Pattern | Pros | Cons |
|---------|------|------|
| **Direct to EIT_Incidents** *(this guide)* | Immediately visible to all EIT users; no triage delay | Alert may not warrant formal incident status; clutters tracker |
| **Via EIT_IntakeQueue** | NIS lead triages before incident is created; more control | Requires NIS lead to review intake queue promptly |

To use the intake queue pattern: replace Steps 9a–9c with a Create item to `EIT_IntakeQueue` instead of `EIT_Incidents`, and omit the ItemID generation steps.

---

## iSight API Reference

iSight (Recorded Future i/o, formerly iSIGHT Partners) exposes a REST API for retrieving threat intelligence reports and alerts.

| Property | Value |
|----------|-------|
| Base URL | `https://api.isightpartners.com` *(confirm with your iSight account manager)* |
| Authentication | API key — passed as HTTP header `X-Auth-Token: [ISIGHT-API-KEY]` |
| Alerts endpoint | `GET /alerts` |
| Date filter parameter | `startDate` (ISO 8601 format: `YYYY-MM-DDTHH:MM:SSZ`) |
| Response format | JSON array of alert objects |

> **API version note:** iSight API versions vary. Confirm the endpoint path, authentication header name, and response schema with your NIS team and iSight account manager. The examples in this guide use the most common iSight API pattern.

### iSight Alert Object Fields Used

| iSight Field | Type | Description |
|-------------|------|-------------|
| `alertId` | String | Unique alert identifier — used for deduplication |
| `title` | String | Alert title → EIT incident Title |
| `description` | String | Alert description → EIT NextAction |
| `severity` | String | `CRITICAL`, `HIGH`, `MEDIUM`, `LOW` |
| `publishDate` | String | Alert publish timestamp (ISO 8601) |
| `reportLink` | String | Link to full iSight report |

---

## Step-by-Step Build Instructions

### 1. Create the Flow

1. Navigate to `https://make.powerautomate.com`
2. Click **+ Create** → **Scheduled cloud flow**
3. Flow name: `EIT iSight Alert Polling`
4. Configure the schedule:
   - **Starting:** Today
   - **Repeat every:** `15 Minutes`
5. Click **Create**

---

### 2. Add Action — Initialize Variable (Lookback Timestamp)

1. Click **+ New step**
2. Search for **Variables** → select **Initialize variable**
3. Rename to: `Set lookback window`
4. Configure:
   - **Name:** `varStartDate`
   - **Type:** String
   - **Value:** Expression →
     ```
     addMinutes(utcNow(), -15)
     ```

> **Reliability note:** This lookback window approach is simple but has a gap risk: if a flow run fails, alerts from that 15-minute window may be missed on the next run. For production hardening, store the last successful run timestamp in a SharePoint list (`EIT_FlowState`) and use that as the lookback start. See `04-Documentation/EIT-Phase2-Admin-Guide.md` for the hardened pattern.

---

### 3. Add Action — HTTP (Call iSight API)

1. Click **+ New step**
2. Search for **HTTP** → select **HTTP**

> **Connector note:** The HTTP connector may require a Premium license (Power Automate Premium or per-flow plan). Confirm licensing before building. Alternatively, if your organisation has a Logic Apps subscription, this step can be built in Logic Apps with the same configuration.

3. Rename to: `Get iSight alerts`
4. Configure:
   - **Method:** `GET`
   - **URI:** Expression →
     ```
     concat('https://api.isightpartners.com/alerts?startDate=', variables('varStartDate'))
     ```
   - **Headers:** Click **+ Add new item** for each:
     - `X-Auth-Token` : `[ISIGHT-API-KEY]` *(store this in an environment variable or Azure Key Vault reference — do not hardcode)*
     - `Accept` : `application/json`
   - **Authentication:** None *(API key passed via header — do not set authentication type)*

> **Security note:** Store `[ISIGHT-API-KEY]` in a Power Automate Environment Variable (secure string type) or Azure Key Vault, not as a hardcoded string in the flow. Reference it using: `@parameters('[ISIGHT-API-KEY]')` or the Key Vault Get secret action.

---

### 4. Add Action — Parse JSON (Parse iSight Response)

1. Click **+ New step**
2. Search for **Data Operation** → select **Parse JSON**
3. Rename to: `Parse iSight response`
4. Configure:
   - **Content:** Dynamic content → `Body` from `Get iSight alerts`
   - **Schema:** Click **Generate from sample** and paste a representative iSight response:

```json
[
  {
    "alertId": "12345-abcde",
    "title": "New Ransomware Variant Targeting Financial Sector",
    "description": "Threat actors have deployed a new variant of LockBit targeting financial institutions in the APAC region.",
    "severity": "HIGH",
    "publishDate": "2026-02-24T08:00:00Z",
    "reportLink": "https://isightpartners.com/report/12345"
  }
]
```

Click **Done** to generate the schema.

> **Schema adjustment:** If iSight returns the alerts array nested inside a wrapper object (e.g., `{"alerts": [...], "total": 5}`), update the Parse JSON content to target `body('Get_iSight_alerts')?['alerts']` and adjust the schema accordingly. Confirm the exact response structure from your NIS team.

---

### 5. Add Action — Filter Array (Severity Threshold)

Filter to only process alerts at or above the agreed severity threshold.

1. Click **+ New step**
2. Search for **Data Operation** → select **Filter array**
3. Rename to: `Filter by severity threshold`
4. Configure:
   - **From:** Dynamic content → `Body` from `Parse iSight response`
   - **Condition:**
     - Left side: Expression → `item()?['severity']`
     - Operator: **is equal to**
     - Right side: `HIGH`

> **Threshold configuration:** The example above passes `HIGH` and (implicitly) `CRITICAL` alerts to EIT. To include both, change the condition to use a custom expression. Replace the simple condition with an **Advanced mode** expression:
> ```
> @or(equals(item()?['severity'], 'CRITICAL'), equals(item()?['severity'], 'HIGH'))
> ```
> Click the three-dot menu on the Filter array condition and select **Edit in advanced mode** to enter this expression directly. Agree the threshold with the NIS domain lead before deployment.

---

### 6. Add Action — Apply to Each (Process Each Alert)

1. Click **+ New step**
2. Search for **Control** → select **Apply to each**
3. Rename to: `For each qualifying alert`
4. **Select an output from previous steps:** Dynamic content → `Body` from `Filter by severity threshold`

All steps 7–10 are placed inside this Apply to each loop.

---

### 7. Inside Loop — Initialize Variable (Severity Mapping)

1. Click **Add an action** (inside the loop)
2. Search for **Variables** → select **Initialize variable**

> **Note:** Power Automate does not allow Initialize variable inside Apply to each. Use **Set variable** instead. Initialize the `varAlertSeverity` variable **before** the Apply to each (add it after Step 5 and before Step 6), then use **Set variable** inside the loop.

**Before the Apply to each (at flow level):**

1. Add **Initialize variable**: Name = `varAlertSeverity`, Type = String, Value = `Moderate`

**Inside the Apply to each loop:**

1. Add **Switch**: Map iSight severity to EIT Severity
   - **On:** Expression → `items('For_each_qualifying_alert')?['severity']`
   - Cases:

| Case Value | Set varAlertSeverity |
|------------|---------------------|
| `CRITICAL` | `Critical` |
| `HIGH` | `High` |
| `MEDIUM` | `Moderate` |
| `LOW` | `Low` |

Default case: `Set variable varAlertSeverity = Moderate`

---

### 8. Inside Loop — Get Items (Deduplication Check)

1. Click **Add an action** (inside the loop, after the Switch)
2. Search for **SharePoint** → select **Get items**
3. Rename to: `Check for existing alert record`
4. Configure:
   - **Site Address:** `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker`
   - **List Name:** `EIT_Incidents`
   - **Filter Query:** OData filter to find existing record by alert ID:
     ```
     RelatedItemIDs eq '[alertId]'
     ```
     Build using Expression:
     ```
     concat('RelatedItemIDs eq ''', items('For_each_qualifying_alert')?['alertId'], '''')
     ```
   - **Top Count:** `1`

---

### 9. Inside Loop — Condition (Is New Alert?)

1. Click **Add an action**
2. Search for **Control** → select **Condition**
3. Rename to: `Is new alert?`
4. Configure:
   - Left side: Expression →
     ```
     length(body('Check_for_existing_alert_record')?['value'])
     ```
   - Operator: **is equal to**
   - Right side: `0`

**If yes** (no existing record — this is a new alert):

#### 9a. Create Item — EIT_Incidents

1. Click **Add an action** (inside If yes)
2. Search for **SharePoint** → select **Create item**
3. Rename to: `Create iSight incident`
4. Configure:
   - **Site Address:** `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker`
   - **List Name:** `EIT_Incidents`

| Field | Value Type | Value |
|-------|-----------|-------|
| **Title** | Expression | `items('For_each_qualifying_alert')?['title']` |
| **ItemID** | Static text | `EIT-PENDING` *(updated in 9c)* |
| **Owner** | Static text | `[NIS-LEAD-NAME]` *(iSight alerts route to NIS lead by default)* |
| **Domain Value** | Static text | `NIS` |
| **Priority Value** | Static text | `High` |
| **IncidentType Value** | Static text | `Threat` |
| **Severity Value** | Dynamic content | `varAlertSeverity` |
| **Status Value** | Static text | `New` |
| **EscalationThreshold Value** | Static text | `Not Escalated` |
| **SourceSystem Value** | Static text | `iSight` |
| **ConfidentialityLevel Value** | Static text | `Internal` |
| **RelatedItemIDs** | Expression | `items('For_each_qualifying_alert')?['alertId']` |
| **DateRaised** | Expression | `utcNow()` |
| **LastUpdated** | Expression | `utcNow()` |
| **NextAction** | Expression | `items('For_each_qualifying_alert')?['description']` |
| **DaysSinceUpdate** | Static number | `0` |
| **StalenessFlag Value** | Static text | `Current` |
| **PrivilegedAndConfidential** | Static | `false` |

#### 9b. Compose — Generate EIT ItemID

1. Click **Add an action**
2. Search for **Data Operation** → select **Compose**
3. Rename to: `Generate iSight ItemID`
4. Expression (see also `flow-expressions/EIT-servicenow-itemid.txt` — same pattern):
   ```
   concat('EIT-', string(outputs('Create_iSight_incident')?['body/ID']))
   ```

#### 9c. Update Item — Set Real ItemID

1. Click **Add an action**
2. Search for **SharePoint** → select **Update item**
3. Rename to: `Set ItemID on iSight incident`
4. Configure:
   - **Site Address:** `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker`
   - **List Name:** `EIT_Incidents`
   - **Id:** Expression → `outputs('Create_iSight_incident')?['body/ID']`
   - **Title:** Expression → `items('For_each_qualifying_alert')?['title']`
   - **ItemID:** Dynamic content → `Outputs` from `Generate iSight ItemID`

#### 9d. Create Item — Opening History Entry

1. Click **Add an action**
2. Search for **SharePoint** → select **Create item**
3. Rename to: `Create iSight history entry`
4. Configure:
   - **Site Address:** `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker`
   - **List Name:** `EIT_IncidentHistory`
   - **Title:** Dynamic content → `Outputs` from `Generate iSight ItemID`
   - **ParentItemID:** Dynamic content → `Outputs` from `Generate iSight ItemID`
   - **UpdateDate:** Expression → `utcNow()`
   - **StatusAtUpdate Value:** Static text → `New`
   - **SeverityAtUpdate Value:** Dynamic content → `varAlertSeverity`
   - **UpdateType Value:** Static text → `Initial Entry`
   - **Notes:** Expression →
     ```
     concat('Auto-imported from iSight alert ', items('For_each_qualifying_alert')?['alertId'], '. Report: ', items('For_each_qualifying_alert')?['reportLink'])
     ```
   - **UpdatedBy:** Static text → `EIT iSight Alert Polling (automated)`
   - **PrivilegedAndConfidential:** Static → `false`

**If no** (alert already in EIT): Leave the If no branch empty (no action required — deduplication is successful).

---

### 10. Save and Test

1. Click **Save**
2. To test without a live iSight connection:
   - Temporarily replace the HTTP action (Step 3) with a **Compose** action returning a static JSON array:
     ```json
     [{"alertId":"TEST-001","title":"Test iSight Alert","description":"Test alert from iSight polling flow validation.","severity":"HIGH","publishDate":"2026-02-24T12:00:00Z","reportLink":"https://example.com/report/TEST-001"}]
     ```
   - Set Parse JSON content to `outputs('Compose')` instead of the HTTP body
   - Run the flow manually via **Test** → **Manually**
3. Verify:
   - New item in `EIT_Incidents` with Domain=NIS, IncidentType=Threat, SourceSystem=iSight, Severity=High
   - RelatedItemIDs = `TEST-001`
   - ItemID set to `EIT-NNN` (not `EIT-PENDING`)
   - Opening history entry in `EIT_IncidentHistory`
4. Run again — verify no duplicate is created (deduplication working)
5. Restore the real HTTP action after testing

---

## Hardened Pattern: State Tracking via SharePoint

For production resilience, replace the fixed 15-minute lookback (Step 2) with a SharePoint-based last-run tracker:

1. Create a single-row SharePoint list `EIT_FlowState`:
   - Column: `Title` (text) — value: `iSight_LastRun`
   - Column: `LastRunTime` (DateTime)

2. At the start of the flow, add:
   - **Get items**: `EIT_FlowState`, Filter: `Title eq 'iSight_LastRun'`
   - **Compose**: `first(body('Get_items')?['value'])?['LastRunTime']` → `varStartDate`

3. At the end of the flow (after Apply to each), add:
   - **Update item**: `EIT_FlowState`, set `LastRunTime = utcNow()`

This ensures the next run picks up exactly where the last successful run left off, even if a run fails.

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| HTTP 401 from iSight | Verify the `X-Auth-Token` header value. API keys expire — rotate as needed per iSight API key policy. |
| HTTP 403 from iSight | Your iSight subscription may not include API access or the `/alerts` endpoint. Contact your iSight account manager. |
| Empty alert array | No alerts in the last 15 minutes above threshold — this is expected. Check the iSight portal manually to confirm. |
| Parse JSON fails | The iSight response structure doesn't match the schema. Use a **Compose** action before Parse JSON to inspect the raw `body` and update the schema to match. |
| Duplicates being created | The `RelatedItemIDs` filter is case-sensitive. Ensure iSight `alertId` values are stored in consistent case. |
| Apply to each times out | The iSight API returned a large number of alerts in one batch. Add a Filter array step before the loop to cap at the top 50 highest-severity alerts. |
| `varAlertSeverity` not updating inside loop | Check that you used **Set variable** (not Initialize variable) inside the loop. |

---

## Flow Diagram

```
Recurrence trigger (every 15 minutes)
    |
    v
Initialize variable: varStartDate = addMinutes(utcNow(), -15)
Initialize variable: varAlertSeverity = 'Moderate'
    |
    v
HTTP GET: iSight /alerts?startDate=[varStartDate]
  Headers: X-Auth-Token: [ISIGHT-API-KEY]
    |
    v
Parse JSON: iSight response array
    |
    v
Filter array: severity = 'HIGH' OR 'CRITICAL'
    |
    v
Apply to each: qualifying alert
    |
    +-- Switch: iSight severity → varAlertSeverity
    |     CRITICAL → Critical
    |     HIGH → High
    |     MEDIUM → Moderate
    |
    +-- Get items: EIT_Incidents
    |     Filter: RelatedItemIDs = alertId
    |     Top: 1
    |
    +-- Condition: length(results) = 0?
          |
          YES (new alert):
            Create item: EIT_Incidents
              Domain = NIS, IncidentType = Threat
              SourceSystem = iSight
              Severity = varAlertSeverity
              RelatedItemIDs = alertId
            Compose: EIT-NNN
            Update item: EIT_Incidents (set real ItemID)
            Create item: EIT_IncidentHistory
              Notes = 'Auto-imported from iSight [alertId]'
          |
          NO (duplicate): (no action)
```
