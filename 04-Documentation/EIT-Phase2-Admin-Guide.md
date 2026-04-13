# Enterprise Incident Tracker — Phase 2 Integration Administration Guide

**Version:** 1.0
**Date:** February 2026
**Audience:** EIT administrator, M365 platform administrator
**Classification:** Internal

---

## Overview

Phase 2 adds five new automated integrations to the Enterprise Incident Tracker:

1. **ServiceNow → EIT** — Ingest flow triggered by ServiceNow webhook
2. **iSight → EIT** — Scheduled polling flow (every 15 minutes)
3. **Power BI Dashboard** — Scheduled dataset refresh (twice daily)
4. **Teams Notifications** — Extension to the Domain Escalation Notifier flow
5. **Automated Archiving** — Monthly archive flow

This guide covers the ongoing administration, monitoring, and troubleshooting of all Phase 2 components. For initial build instructions, see the individual guides in `02-PowerAutomate/` and `07-PowerBI/`.

---

## 1. ServiceNow Integration Administration

### 1.1 Flow Health Monitoring

| Check | How | Frequency |
|-------|-----|-----------|
| Flow run history | Power Automate → `EIT ServiceNow Ingest` → Run history | Weekly |
| Failed runs | Look for red (failed) run entries. Click to inspect the error. | Weekly |
| Trigger URL validity | Confirm with ServiceNow team that webhooks are reaching the URL. Check ServiceNow outbound notification history. | Monthly |
| Deduplication integrity | Spot-check EIT_Incidents for duplicate ServiceNow numbers in `RelatedItemIDs` | Monthly |

### 1.2 Connection Expiry

The SharePoint connection used by the flow may require re-authentication periodically (typically every 90 days for OAuth connections, or on password change).

To re-authenticate:
1. Navigate to `https://make.powerautomate.com`
2. Go to **Connections** (left nav)
3. Find the SharePoint connection used by `EIT ServiceNow Ingest`
4. Click the three-dot menu → **Fix connection** → sign in again
5. Return to the flow — it resumes processing automatically once the connection is valid

### 1.3 Field Mapping Changes

If ServiceNow adds new priority levels, assignment groups, or categories:

1. Open `EIT ServiceNow Ingest` in edit mode
2. Find the relevant Switch block (Map priority to severity, Map assignment group to domain, or Map category to incident type)
3. Add the new case value
4. Save and test with a synthetic POST payload (see `EIT-ServiceNow-Ingest-Flow.md` Step 11)

### 1.4 Trigger URL Rotation

If the Power Automate HTTP trigger URL needs to be rotated (for security reasons):

1. Open `EIT ServiceNow Ingest` → click the HTTP trigger
2. Click **Generate new URL** (this invalidates the old URL immediately)
3. Copy the new URL
4. Provide the new URL to the ServiceNow administrator immediately — ServiceNow will fail to deliver webhooks until the URL is updated
5. Coordinate the changeover during a low-activity window to minimise missed alerts

> **Warning:** URL rotation causes a gap in incident ingestion while ServiceNow is reconfigured. Any ServiceNow incidents raised during this window must be manually checked and entered into EIT if they meet the P1/P2 threshold.

### 1.5 ServiceNow API Payload Changes

If the ServiceNow team changes the outbound notification payload schema:

1. In the HTTP trigger, click **Use sample payload to generate schema**
2. Paste the updated payload and click **Done**
3. Review all downstream steps that reference the changed fields — update dynamic content references where field names have changed
4. Test before saving

---

## 2. iSight Integration Administration

### 2.1 Flow Health Monitoring

| Check | How | Frequency |
|-------|-----|-----------|
| Flow run history | Power Automate → `EIT iSight Alert Polling` → Run history | Daily |
| Failed runs | Check for HTTP 4xx/5xx errors from iSight API. Click failed run for details. | Daily |
| Alert volume | Review EIT_Incidents for new iSight-sourced items — confirm plausible volume | Weekly |
| Duplicate check | Verify no duplicate iSight alerts (same `RelatedItemIDs` on two items) | Weekly |

### 2.2 iSight API Key Rotation

iSight API keys should be rotated per your security policy. When rotating:

1. Obtain the new API key from iSight (via your account manager or iSight portal)
2. Open `EIT iSight Alert Polling` in edit mode
3. Find the HTTP action (`Get iSight alerts`)
4. Update the `X-Auth-Token` header value to the new API key
5. If using a Power Automate Environment Variable: update the variable value in **Settings** → **Environment variables** instead
6. Save and test the flow (see `EIT-iSight-Polling-Flow.md` Step 10)
7. Revoke the old API key in iSight

### 2.3 Severity Threshold Changes

If the NIS team requests a change to the severity threshold (e.g., also include MEDIUM severity):

1. Open `EIT iSight Alert Polling` in edit mode
2. Find the `Filter by severity threshold` Filter array action
3. Update the condition from `severity = HIGH` to the new threshold
4. For an OR condition (HIGH or CRITICAL): switch to Advanced mode, enter:
   ```
   @or(equals(item()?['severity'], 'CRITICAL'), equals(item()?['severity'], 'HIGH'), equals(item()?['severity'], 'MEDIUM'))
   ```
5. Save and inform the NIS domain lead — they should expect increased intake volume

### 2.4 State Tracking Hardening (Optional)

For production environments where alert continuity is critical, implement the state-tracking pattern:

1. Create a SharePoint list `EIT_FlowState` with columns: `Title` (text), `LastRunTime` (DateTime)
2. Create one item: Title = `iSight_LastRun`, LastRunTime = [current time]
3. In the flow:
   - At start: Get items from `EIT_FlowState` (filter Title = iSight_LastRun), set `varStartDate` from `LastRunTime`
   - At end (after Apply to each): Update item `EIT_FlowState`, set `LastRunTime = utcNow()`
4. This ensures the next run picks up exactly where the previous run ended, even if a run fails

---

## 3. Power BI Dashboard Administration

### 3.1 Scheduled Refresh Monitoring

| Check | How | Frequency |
|-------|-----|-----------|
| Refresh history | Power BI Service → EIT-Dashboard dataset → Settings → Refresh history | Daily |
| Failed refreshes | Receive email notification (enable in dataset Settings → Data source failures → Send email) | On failure |
| Credential expiry | Check if the SharePoint connection credential has expired (90-day OAuth tokens) | Monthly |
| Row count accuracy | Compare Power BI total open count against EIT Power App Executive Report Screen | Weekly |

### 3.2 Re-authenticating the SharePoint Connection

Power BI Service uses a stored credential to connect to SharePoint for scheduled refresh. If this expires:

1. In Power BI Service, navigate to the **EIT-Dashboard** dataset settings
2. Under **Data source credentials** → click the SharePoint data source → **Edit credentials**
3. Sign in with the admin M365 account
4. Click **Sign in** and complete MFA if prompted
5. Next scheduled refresh will use the new credential

### 3.3 Updating the PBIX Model

When EIT_Incidents columns are added or renamed (e.g., from a Phase 2 schema change):

1. Open `EIT-Dashboard.pbix` in Power BI Desktop
2. Click **Transform Data** → **Data source settings** (if URL changed) or directly **Transform Data** (if schema changed)
3. In Power Query, refresh the EIT_Incidents query to pick up new columns
4. Add new columns to relevant report visuals
5. Update DAX measures if new fields are involved
6. Re-publish (overwrite the existing report in the workspace)
7. Re-apply RLS role assignments after publishing (verify they are preserved — they usually are, but confirm)

### 3.4 RLS Maintenance

See `EIT-PowerBI-RLS-Guide.md` for full RLS administration. Summary:

| Event | Action |
|-------|--------|
| New EIT Member added | No action — M365 group membership update is automatic |
| New P&C authorized user | Add to `EIT-PAC-Authorized` M365 group; do NOT add to `Standard_Users` RLS role |
| EIT model republished | Verify RLS role assignments are still in place in dataset Security settings |
| P&C incident count discrepancy | Check that `PrivilegedAndConfidential` column is correctly imported as boolean in Power Query |

---

## 4. Teams Notification Administration

### 4.1 Channel Management

| Task | How |
|------|-----|
| Add new domain lead to channel | Add them as a Teams channel member in the `EIT-Escalations` channel |
| Create per-domain channels | Follow the per-domain variant instructions in `EIT-Teams-Notification-Extension.md` |
| Change from unified to per-domain | Modify the flow to add a Switch block selecting channel by domain (see Extension guide) |
| Remove a user from notifications | Remove them from the Teams channel; the flow will still post, but they won't see it |

### 4.2 Adaptive Card Updates

To update the adaptive card content or format:

1. Open `EIT Domain Escalation Notifier` in edit mode
2. Find the `Post Executive Brief card to Teams` and `Post Escalation card to Teams` actions
3. Update the JSON in the Adaptive Card field
4. Validate the JSON at [adaptivecards.io/designer](https://adaptivecards.io/designer/) before saving
5. Save and test

### 4.3 Teams Connector Re-authentication

If Teams actions fail with authentication errors:

1. Navigate to Power Automate → **Connections**
2. Find the Microsoft Teams connection
3. Click three-dot menu → **Fix connection** → re-authenticate
4. The flow resumes automatically

---

## 5. Automated Archiving Administration

### 5.1 Monthly Run Monitoring

| Check | How | Frequency |
|-------|-----|-----------|
| Flow run completion | Power Automate → `EIT Monthly Archive` → Run history (check 1st of each month) | Monthly |
| Archive count email | Review admin summary email received on 1st of each month | Monthly |
| Archive list growth | Check item count in `EIT_Incidents_Archive` list year over year | Quarterly |
| Deleted items verification | Confirm archived items no longer appear in `EIT_Incidents` | Monthly (post-run) |

### 5.2 Manual Archive Trigger

To archive on demand (outside the scheduled run):

1. Open `EIT Monthly Archive` in Power Automate
2. Click **Run** → **Manually run**
3. This runs the same logic — verify you intend to archive before triggering

### 5.3 Retention Period Adjustment

To change the retention period from 12 months to a different period:

1. Open `EIT Monthly Archive` in edit mode
2. Find the `Set archive cutoff date` Initialize variable action
3. Change `-365` to the desired number of days (e.g., `-548` for 18 months, `-730` for 2 years)
4. Confirm the change with your data governance lead
5. Save the flow

> **Regulatory note:** If the organisation is subject to data retention regulations (e.g., GDPR, sector-specific rules), the retention period must comply with applicable requirements. Legal counsel should review any reduction in retention period before implementation.

### 5.4 Restoring an Accidentally Archived Item

If an item was archived prematurely:

1. Navigate to `EIT_Incidents_Archive` in SharePoint
2. Find the item by ItemID or Title
3. Note all field values
4. Manually create a new item in `EIT_Incidents` with those values (set Status back to the correct pre-archive status)
5. Delete the archive copy
6. Alert the domain lead to review and re-triage the item

> **SharePoint Recycle Bin:** Deleted items from `EIT_Incidents` are in the SharePoint site Recycle Bin for 93 days. If an item was deleted in error during the archive run, it can be restored via the Recycle Bin before manually re-entering. Navigate to Site Settings → Recycle Bin.

---

## 6. Common Cross-Integration Issues

### 6.1 Duplicate Incidents

**Symptoms:** Same ServiceNow number or iSight alertId appears in multiple EIT_Incidents rows.

**Cause:** Deduplication check failed — most commonly because the `RelatedItemIDs` field value doesn't match the stored value (case sensitivity, whitespace, or format mismatch).

**Resolution:**
1. Identify duplicates by filtering EIT_Incidents by `SourceSystem` and checking for repeated `RelatedItemIDs`
2. Merge the duplicates: update the correct record with the most current information, then delete the duplicate
3. Update the flow deduplication filter to match the exact string format returned by the source system

### 6.2 SharePoint Throttling

**Symptoms:** Flows fail with HTTP 429 (Too Many Requests) errors during Apply to each loops.

**Cause:** Power Automate SharePoint connector has a rate limit (~600 operations/minute). Loops processing many items can exceed this.

**Resolution:**
1. Add a **Delay** action inside Apply to each loops (1-second delay between iterations)
2. Reduce the `Top Count` in Get items actions to process fewer items per run
3. Consider migrating high-volume flows to Logic Apps, which has higher SharePoint throughput limits

### 6.3 Flow Connection Drift

**Symptoms:** Flows fail with "connection is not authorized" or similar errors.

**Cause:** The M365 account used for connections has changed its password, MFA tokens expired, or the account was disabled/modified.

**Resolution:**
1. Audit all flow connections in Power Automate → Connections
2. Re-authenticate any connections showing error status
3. Consider using a dedicated service account for flow connections, not a personal user account — this reduces connection drift from personal account lifecycle events

### 6.4 P&C Incidents Leaking to Power BI

**Symptoms:** Standard users report seeing P&C incidents in the Power BI dashboard.

**Cause:** Most likely the user was not assigned to the `Standard_Users` RLS role, or was added as a workspace member without a role assignment (which bypasses RLS).

**Resolution:**
1. Check the user's role assignment in Power BI Service → EIT-Dashboard dataset → Security
2. If they are listed as a workspace **Admin** or **Member** without a role: they bypass RLS by design
3. Change their workspace role to **Viewer** (not Member) — Viewers are subject to RLS
4. If they need edit access for other reasons: they must be a Member/Admin and therefore will see P&C data — grant this access only to those with a genuine need and appropriate clearance

---

## 7. Phase 2 Flow Inventory

| Flow Name | Type | Trigger | Owner |
|-----------|------|---------|-------|
| `EIT ServiceNow Ingest` | Automated | HTTP POST (from ServiceNow) | EIT Admin |
| `EIT iSight Alert Polling` | Scheduled | Every 15 minutes | EIT Admin |
| `EIT Monthly Archive` | Scheduled | 1st of month, 02:00 UTC | EIT Admin |
| `EIT Domain Escalation Notifier` | Instant | Power Apps | EIT Admin |

**Phase 1 flows (unchanged in Phase 2):**

| Flow Name | Type | Trigger |
|-----------|------|---------|
| `EIT Daily Staleness & Escalation Calculator` | Scheduled | Daily, 01:00 UTC |
| `EIT Intake Promotion` | Instant | Power Apps |

---

## 8. Administrative Checklist — Phase 2 Quarterly Review

Run this checklist at the start of each quarter:

- [ ] Review all Phase 2 flow run histories for recurring failures
- [ ] Verify SharePoint connections for all flows are authenticated
- [ ] Check iSight API key expiry date — rotate if within 30 days
- [ ] Confirm Power BI scheduled refresh ran successfully in the last 7 days
- [ ] Verify RLS role assignments in Power BI are current
- [ ] Review `EIT_Incidents_Archive` item count — notify data governance if unexpectedly high
- [ ] Confirm ServiceNow webhook is active and pointing to the correct trigger URL
- [ ] Test Teams adaptive card rendering (trigger a test escalation and verify the card appears)
- [ ] Review and confirm the domain-to-group mapping table in `EIT ServiceNow Ingest` is current
- [ ] Check iSight severity threshold with NIS domain lead — confirm it is still appropriate
