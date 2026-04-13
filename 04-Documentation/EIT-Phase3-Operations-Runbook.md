# Enterprise Incident Tracker — Operations Runbook

**Version:** 1.0
**Date:** February 2026
**Audience:** EIT administrator, M365 platform team
**Classification:** Internal — Restricted

---

## Overview

This runbook is the single reference for operating the Enterprise Incident Tracker in production. It covers:

- System component inventory and health check procedures
- Scheduled maintenance tasks
- Failure detection and response procedures
- Manual fallback procedures when the system is unavailable
- Escalation contacts for system failures
- Known failure modes and remediation steps

---

## Section 1: System Component Inventory

### 1.1 All EIT Components

| Component | Type | Platform | Admin URL | Health Indicator |
|-----------|------|----------|-----------|-----------------|
| `EIT_Incidents` | SharePoint list | SharePoint Online | `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker` | List accessible, item count ≥ 0 |
| `EIT_IncidentHistory` | SharePoint list | SharePoint Online | Same site | List accessible |
| `EIT_IntakeQueue` | SharePoint list | SharePoint Online | Same site | List accessible |
| `EIT_DomainContacts` | SharePoint list | SharePoint Online | Same site | 5 rows present |
| `EIT_Incidents_Archive` | SharePoint list | SharePoint Online | Same site | List accessible |
| `Enterprise Incident Tracker` | Power App | Power Apps | `https://make.powerapps.com` | App opens without error |
| `EIT Daily Staleness & Escalation Calculator` | Flow | Power Automate | `https://make.powerautomate.com` | Last run = yesterday, Succeeded |
| `EIT Intake Promotion` | Flow | Power Automate | Same | On, no errors |
| `EIT Domain Escalation Notifier` | Flow | Power Automate | Same | On, no errors |
| `EIT ServiceNow Ingest` | Flow | Power Automate | Same | Last run within 24h, Succeeded |
| `EIT iSight Alert Polling` | Flow | Power Automate | Same | Last run ≤ 15 min ago, Succeeded |
| `EIT Monthly Archive` | Flow | Power Automate | Same | Last run ≤ 35 days, Succeeded |
| `EIT Monthly Compliance Report` | Flow | Power Automate | Same | Last run ≤ 35 days, Succeeded |
| `EIT Quarterly Access Review Reminder` | Flow | Power Automate | Same | Last run ≤ 95 days, Succeeded |
| `EIT-Dashboard` | Power BI report | Power BI Service | `https://app.powerbi.com` | Last refresh ≤ 24h, Succeeded |

### 1.2 Escalation Contacts

| Role | Contact | Escalation Condition |
|------|---------|---------------------|
| EIT Administrator | `david@brightpathtechnology.io` | All EIT system issues |
| M365 Platform Team | `[platform-team@TENANT-DOMAIN]` | SharePoint site outage, tenant-level issues |
| Microsoft Support | `https://admin.microsoft.com` → Support | Microsoft service outage affecting SharePoint/Power Automate/Power BI |
| Legal Counsel | *(from EIT_DomainContacts)* | P&C access or breach concerns |
| ServiceNow Administrator | *(your ServiceNow team contact)* | ServiceNow webhook not firing |
| iSight Account Manager | *(iSight contact)* | iSight API key issues, API changes |

---

## Section 2: Health Check Procedures

### 2.1 Daily Health Check (~10 minutes)

Run every business day morning.

**Step 1: Power Automate — Daily Staleness Flow**

1. Navigate to `https://make.powerautomate.com` → `My flows`
2. Open `EIT Daily Staleness & Escalation Calculator`
3. Click **28 day run history**
4. Verify: yesterday's run shows status = **Succeeded**
5. If **Failed**: see Section 3.1

**Step 2: Power Automate — iSight Polling (if deployed)**

1. Open `EIT iSight Alert Polling`
2. Verify: last run within 15 minutes, status = **Succeeded** (or no alerts — this is normal)
3. If repeated failures: see Section 3.5

**Step 3: SharePoint Lists — Quick Sanity Check**

1. Open `EIT_Incidents` in SharePoint
2. Verify: list loads, items visible, `DaysSinceUpdate` column shows recent values (not all zeros or blank)
3. If `DaysSinceUpdate` is 0 for all items: staleness flow may not have updated — check Step 1

**Step 4: Power App — Connectivity Check**

1. Open the Enterprise Incident Tracker Power App from `make.powerapps.com`
2. Verify: Dashboard loads, KPI cards show non-zero values (if incidents exist), no connection error banners
3. If app shows "Connection failed" or similar: see Section 3.6

**Daily health check complete. Estimated time: 5–10 minutes.**

### 2.2 Weekly Health Check (~20 minutes)

Run every Monday morning, in addition to the daily check.

**Step 1: All flows — Run history review**

For each flow, check the last 7 days of run history:
- Any failed runs?
- Any flows with unexpectedly long run times (>10 minutes for most; >30 minutes for archive flow)?

**Step 2: EIT_Incidents — Data integrity check**

1. In SharePoint, filter `EIT_Incidents`:
   - Any items with ItemID = `EIT-PENDING` → these were not fully created; see Section 3.3
   - Any items with blank Domain or blank Severity → data quality issue; update manually

**Step 3: Power BI — Refresh check**

1. In Power BI Service → EIT-Dashboard dataset → Refresh history
2. Verify: last two refreshes (06:00 and 18:00) succeeded within the last 24 hours
3. If failed: see Section 3.7

**Step 4: ServiceNow webhook (if deployed) — Spot check**

1. In ServiceNow: check the outbound notification history (ServiceNow admin view)
2. Verify: at least one notification was sent in the last 7 days (if any P1/P2 incidents occurred)
3. Cross-reference: verify the corresponding EIT_Incidents items exist for recent ServiceNow incidents

**Step 5: Stale incidents — Domain lead notification**

1. In EIT_Incidents, filter `StalenessFlag = Stale`
2. If stale count > 0: send a brief email to the relevant domain leads noting stale incident IDs and requesting an update
3. This is a process reminder, not a system failure

### 2.3 Monthly Health Check (~30 minutes)

Run on the 1st of each month after the archive and compliance report flows have run.

1. Verify compliance report email was received
2. Verify archive flow ran and correct number of items were archived
3. Check SharePoint connection credential expiry dates in Power Automate and Power BI
4. Review Entra ID sign-in logs for any EIT connection accounts — check for failed authentications
5. Verify `EIT_DomainContacts` — all 5 rows have valid email addresses; confirm with domain leads if contact details have changed
6. Check Power App version — is the published app the latest version? (Open app in edit mode, confirm no unpublished changes)

---

## Section 3: Failure Response Procedures

### 3.1 Daily Staleness Flow — Failed Run

**Symptoms:** `EIT Daily Staleness & Escalation Calculator` shows failed run; `DaysSinceUpdate` values not updated overnight.

**Impact:** DaysSinceUpdate values will be 1 day stale. StalenessFlag may be incorrect. Auto-escalation emails may not have fired for newly-stale incidents.

**Response:**

1. Open the failed flow run → click **Details** → find the failing step
2. Common causes and fixes:

| Failing Step | Likely Cause | Fix |
|-------------|-------------|-----|
| Get items (EIT_Incidents) | SharePoint connection expired | Re-authenticate connection in Power Automate → Connections |
| Apply to each | SharePoint throttling (429 error) | Reduce Top Count; add Delay action; or wait 30 min and re-run |
| Update item | Item-level permissions blocking update on P&C incident | Ensure flow connection account has Full Control on the site |
| Send escalation email | Outlook connection expired | Re-authenticate Outlook connection |

3. After fixing the root cause: click **Resubmit** on the failed run (Power Automate → failed run → Resubmit)
4. Verify the resubmitted run succeeds
5. If the fix takes > 24 hours: manually update `DaysSinceUpdate` for any Critical or High incidents via SharePoint list edit (to prevent incorrect auto-escalation on the next day's run)

### 3.2 ServiceNow Webhook — Not Firing

**Symptoms:** P1/P2 incidents exist in ServiceNow but are not appearing in EIT_Incidents.

**Diagnosis:**
1. In Power Automate → `EIT ServiceNow Ingest` → run history: are there recent runs?
2. If no runs: the webhook is not being sent by ServiceNow → contact ServiceNow administrator
3. If runs exist but failed: inspect the failure reason in the run details

**Response:**
1. Contact ServiceNow administrator and provide:
   - The HTTP trigger URL from the flow
   - Expected payload format (from `EIT-ServiceNow-Ingest-Flow.md`)
   - ServiceNow outbound notification configuration requirements
2. While awaiting fix: affected incidents must be manually entered into EIT_IntakeQueue or EIT_Incidents
3. After fix: test with a synthetic POST (see `EIT-ServiceNow-Ingest-Flow.md` Step 11)

**Manual entry procedure during outage:**
1. Domain lead (NIS or PT) identifies the ServiceNow incident
2. Domain lead submits via EIT Submit overlay in Power App
3. Domain lead notes in the intake description: "Manual entry — ServiceNow [number] — service disruption"
4. Triage and promote normally

### 3.3 Incidents Stuck with ItemID = EIT-PENDING

**Symptoms:** One or more items in `EIT_Incidents` show `ItemID = EIT-PENDING`.

**Cause:** The Create item step succeeded but the subsequent Update item (set real ItemID) failed. The flow may have timed out, been throttled, or encountered a transient error.

**Response:**
1. Identify all `EIT-PENDING` items in EIT_Incidents
2. For each:
   a. Note the SharePoint integer `ID` from the list (visible in the ID column — add to view if hidden)
   b. Compose the correct ItemID: `EIT-[ID]` (e.g., SharePoint ID = 47 → ItemID = `EIT-47`)
   c. Edit the item directly in SharePoint list view: update `ItemID` from `EIT-PENDING` to `EIT-47`
3. Also update `EIT_IncidentHistory` entries for this incident: they may have `Title` and `ParentItemID` set to `EIT-PENDING` — update these to the real ItemID
4. Investigate the root cause in the flow run history to prevent recurrence

### 3.4 iSight Polling — API Authentication Failure (HTTP 401)

**Symptoms:** `EIT iSight Alert Polling` fails with HTTP 401 error on the HTTP action.

**Cause:** iSight API key expired or revoked.

**Response:**
1. Contact NIS team or iSight account manager to obtain a new API key
2. Update the key in the flow (see `EIT-Phase2-Admin-Guide.md` Section 2.2)
3. Test the flow manually
4. Check for missed alerts during the outage: review iSight portal directly for any HIGH/CRITICAL alerts during the downtime window and manually create EIT incidents if needed

### 3.5 iSight Polling — Repeated Timeouts

**Symptoms:** Flow runs succeed but take > 5 minutes; Apply to each loop times out on large alert batches.

**Response:**
1. Add a Filter array step before the Apply to each to cap at the top 20 alerts
2. Implement the state tracking pattern (`EIT_FlowState`) so alerts are not re-queried on every run
3. Consider switching to a webhook model if iSight supports outbound webhooks (eliminates polling overhead entirely)

### 3.6 Power App — Connection Error

**Symptoms:** Power App displays "Connection failed" or data sources show error banners.

**Cause options:**
- SharePoint site temporarily unavailable
- Power Apps data source connection expired
- Power App needs a republish (stale connector)

**Response:**
1. Check Microsoft 365 service health (`https://admin.microsoft.com` → Service health) — is SharePoint Online listed as degraded?
2. If Microsoft outage: wait for service restoration; use manual fallback (Section 4)
3. If not a Microsoft outage: open the app in edit mode (`make.powerapps.com`) → Data → check each data source connection → re-authenticate if showing error
4. After re-authentication: **Publish** the app (even if no code changes — this refreshes the connection tokens)
5. If the app is inaccessible entirely: users can access EIT_Incidents directly via SharePoint list view as a fallback

### 3.7 Power BI — Scheduled Refresh Failure

**Symptoms:** Power BI refresh history shows failed refresh.

**Common causes and fixes:**

| Error | Fix |
|-------|-----|
| Credential expired | Re-authenticate SharePoint connection in dataset settings |
| Privacy level mismatch | Set Privacy level to Organizational for the SharePoint data source |
| Gateway required | If on-premises gateway was accidentally selected, switch to none (SharePoint Online doesn't need a gateway) |
| Timeout | Data volume too high; optimize Power Query steps or contact Power BI support |

**Impact while failed:** Dashboard shows last successful refresh data. Add a "Last Refreshed" card to the dashboard so users can see the data age. Temporary workaround: manually refresh in Power BI Desktop and re-publish.

### 3.8 SharePoint Site Outage

**Symptoms:** SharePoint site (`EnterpriseIncidentTracker`) is inaccessible; all EIT components fail.

**Response:**
1. Check Microsoft 365 service health — confirm SharePoint Online status
2. If Microsoft outage: activate Manual Fallback (Section 4)
3. Communicate to all EIT users: EIT is temporarily unavailable; use email-based incident reporting as fallback
4. Monitor service health; restore normal operations once SharePoint is restored
5. After restoration: run the staleness flow manually (Test → Manually) to recalculate any missed staleness updates

---

## Section 4: Manual Fallback Procedures

When the EIT is unavailable, the following manual procedures maintain incident tracking continuity.

### 4.1 Manual Incident Reporting During Outage

**For submitters (all staff):**

Send incident reports by email to the appropriate domain lead:
- National Security: `[NS-LEAD-EMAIL]`
- NIS: `[NIS-LEAD-EMAIL]`
- Privacy & Technology: `[PT-LEAD-EMAIL]`
- Human Capital: `[HC-LEAD-EMAIL]`
- Physical Security: `[PS-LEAD-EMAIL]`

Email subject format: `[EIT INCIDENT REPORT] [Brief description] — [Severity if known]`

Email body: include Description, Incident Type, Who is affected, Date/time of discovery, Suggested severity.

**For domain leads during outage:**

Maintain a temporary incident log in an Excel file on the domain lead's OneDrive:
- Columns: Date, Title, Description, Severity, Status, Owner, Action Taken
- Date/time stamp all entries
- When EIT is restored: enter all manual log items into EIT (via intake queue) and note "Entered retroactively due to EIT outage on [date]" in the NextAction field

### 4.2 Restoring After Outage

After EIT is restored:

1. Enter all manual log items into `EIT_IntakeQueue` via Power App submit overlay
2. Triage and promote all backlog items
3. For any incidents that progressed significantly during the outage: add history entries capturing the outage period's actions
4. Run the daily staleness flow manually to reset `DaysSinceUpdate` to accurate values
5. Notify all domain leads that EIT is restored and backlog has been entered

---

## Section 5: Scheduled Maintenance Calendar

| Task | Frequency | Typical Date | Who |
|------|-----------|-------------|-----|
| Daily health check | Daily | Every business day AM | EIT Admin |
| Weekly health check | Weekly | Monday AM | EIT Admin |
| Monthly health check + archive review | Monthly | 1st business day | EIT Admin |
| Power BI SharePoint credential refresh | Every 90 days | Rolling | EIT Admin |
| Power Automate connection audit | Quarterly | Start of quarter | EIT Admin |
| Domain contacts update | Quarterly | Start of quarter | EIT Admin + Domain Leads |
| Quarterly access review | Quarterly | Jan 1, Apr 1, Jul 1, Oct 1 | EIT Admin + Domain Leads |
| P&C access review | Every 6 months | Jan 1, Jul 1 | Legal Counsel + EIT Admin |
| Annual DR test | Annually | Agreed date | EIT Admin + M365 team |
| iSight API key rotation | Per security policy | Per policy | EIT Admin + NIS team |
| Power App version review | Quarterly | Start of quarter | EIT Admin |

---

## Section 6: Change Management

All changes to EIT components must follow this procedure:

1. **Document the change:** What is changing, why, expected impact
2. **Test in non-production first:** Use a test SharePoint site or test Power App environment if available
3. **Notify users:** For changes affecting the Power App UI or SharePoint list schema, notify domain leads 48 hours in advance
4. **Implement outside business hours:** For disruptive changes (schema updates, app republish), implement after 17:00 local time
5. **Verify after change:** Run the daily health check immediately after implementing
6. **Record the change:** Update the EIT changelog (maintain a simple SharePoint list or OneNote page)

**Changes requiring full DR backup before implementation:**
- SharePoint list schema changes (adding/removing/renaming columns)
- Power App major version updates (new screens, data source changes)
- Power Automate flow restructuring (not just expression updates)
- Any change to `EIT_DomainContacts` email addresses (could affect live escalation routing)
