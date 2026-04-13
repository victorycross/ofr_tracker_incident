# Enterprise Incident Tracker — Quarterly Access Review Runbook

**Version:** 1.0
**Date:** February 2026
**Audience:** EIT administrator, domain leads, Legal Counsel
**Classification:** Internal — Restricted

---

## Overview

This runbook governs the quarterly access certification process for the Enterprise Incident Tracker. Access reviews ensure that only current, authorised personnel retain EIT access and that no stale or inappropriate access persists.

The EIT operates a **layered access review** model:

| Review | Frequency | Reviewer | Scope |
|--------|-----------|----------|-------|
| EIT Members | Quarterly | Domain leads (per domain) | All active EIT users with read-write access |
| EIT Viewers | Quarterly | EIT administrator | Read-only leadership stakeholders |
| EIT-PAC-Authorized | Every 6 months | Legal Counsel + EIT admin | P&C access eligible/permanent members |
| PIM Activation Audit | Monthly | EIT administrator | Who activated P&C access, justification review |

---

## Section 1: Quarterly EIT Members Review

### 1.1 Trigger

The `EIT Access Review Reminder` Power Automate flow (see `02-PowerAutomate/EIT-Access-Review-Reminder-Flow.md`) sends review assignments to domain leads on the first business day of each quarter (January, April, July, October).

Alternatively, if using Entra ID Access Reviews (see `EIT-Phase3-Entra-Guide.md` Section 4), Entra sends review requests automatically.

### 1.2 Reviewer Assignments

| Domain | Primary Reviewer | Secondary Reviewer |
|--------|-----------------|-------------------|
| National Security | `[NS-LEAD-NAME]` | EIT Administrator |
| NIS | `[NIS-LEAD-NAME]` | EIT Administrator |
| Privacy & Technology | `[PT-LEAD-NAME]` | EIT Administrator |
| Human Capital | `[HC-LEAD-NAME]` | EIT Administrator |
| Physical Security | `[PS-LEAD-NAME]` | EIT Administrator |

The EIT administrator reviews the `EIT Viewers` group and serves as secondary reviewer for all domain groups.

### 1.3 Review Procedure — Domain Leads

**Deadline:** Complete within 10 business days of receiving the review request.

**Step 1: Obtain the current member list**

Option A — via Entra ID Access Reviews (if configured):
1. Navigate to `https://myaccess.microsoft.com` → **Access reviews**
2. Find the review assigned to you and click **Start review**
3. The review shows each member in your domain group

Option B — via manual group export:
1. Navigate to `https://entra.microsoft.com` → **Groups** → find `EIT Members`
2. Click **Members** → **Download members** (CSV)
3. Filter the CSV for users in your domain (by department or manager)

**Step 2: Assess each member**

For each user in your domain, confirm:

| Check | Action if YES | Action if NO |
|-------|--------------|-------------|
| User is still employed at the organisation | Approve | Flag for removal |
| User is still assigned to your domain | Approve | Flag — transfer or remove |
| User has used EIT in the last 90 days (check audit log) | Approve | Flag as potentially stale |
| User has a legitimate ongoing need for EIT access | Approve | Flag for removal |

> **Audit log access:** Navigate to the EIT SharePoint site → **Site Settings** → **Audit log reports** to confirm user activity. Alternatively, check `EIT_IncidentHistory` for entries with the user's name in `UpdatedBy` within the last 90 days.

**Step 3: Submit decisions**

- Option A (Entra Access Reviews): Select Approve or Deny for each user in the review portal
- Option B (manual): Complete the Access Review Sign-Off Template (see Section 1.5) and email to the EIT administrator

**Step 4: Handle departures and role changes**

If a user has left or changed roles:
1. Email EIT administrator immediately (do not wait for review completion)
2. EIT administrator removes the user from the relevant group within 24 hours
3. If user is in `EIT-PAC-Authorized`: Legal Counsel must be notified

### 1.4 Review Procedure — EIT Administrator

**For EIT Viewers group:**

1. Export the `EIT Viewers` member list from Entra ID
2. Confirm with executive assistants or HR that each viewer is still in a leadership role requiring incident visibility
3. Remove any viewers who have left or changed roles

**For stale access (users who haven't logged in):**

1. In Entra ID → **Sign-in logs**, filter for each EIT group member → check last sign-in date to SharePoint or Power Apps
2. Flag any user with no EIT activity in 90+ days for domain lead review
3. Do not auto-remove without domain lead confirmation — some users may have legitimate periodic access needs

**Final reconciliation:**

After all domain leads submit their reviews:
1. Process all "deny" decisions — remove users from groups
2. Update the Access Review Sign-Off Record (Section 1.5)
3. Send completion confirmation to each domain lead and Legal Counsel

### 1.5 Access Review Sign-Off Template

Complete and retain for each quarterly review. File in the compliance records folder.

```
EIT QUARTERLY ACCESS REVIEW — SIGN-OFF RECORD

Review period: Q[N] [YEAR]
Review completed: [DATE]
Reviewed by: [REVIEWER NAME, TITLE]

DOMAIN: [Domain name]

Members reviewed: [COUNT]
Members approved: [COUNT]
Members removed: [COUNT]

Removed members:
  - [NAME] — Reason: [departed / role change / no longer needs access]
  - [NAME] — Reason: ...

Certification statement:
I confirm that all members of the EIT [Domain] group have been reviewed
and that only users with a current, legitimate business need retain access.

Signature: _____________________________ Date: __________
```

---

## Section 2: P&C Access Review (Every 6 Months)

### 2.1 Scope

The `EIT-PAC-Authorized` group (or PIM eligible assignment list) is reviewed every 6 months. This review is more rigorous than the standard quarterly review because P&C access carries legal privilege implications.

### 2.2 Reviewers

**Primary:** Legal Counsel (the attorney responsible for P&C designation decisions)
**Secondary:** EIT Administrator

Both reviewers must independently assess each member. Disagreements are escalated to General Counsel.

### 2.3 P&C Review Procedure

**Step 1: Generate PIM activation audit (if PIM deployed)**

1. Navigate to `https://entra.microsoft.com` → **Identity Governance** → **Privileged Identity Management** → **Groups** → `EIT-PAC-Authorized` → **Audit history**
2. Export the last 6 months of activation records
3. For each user, note: number of activations, justifications provided, incidents accessed

**Step 2: Assess each eligible member**

For each person in `EIT-PAC-Authorized` (eligible or permanent):

| Check | Action if YES | Action if NO |
|-------|--------------|-------------|
| User currently represents the organisation in a legal or privileged capacity | Retain | Remove from group |
| User has a current matter that may require P&C incident access | Retain | Consider removing |
| User's activations in the last 6 months were justified (appropriate incidents) | No action | Investigate and document |
| User activated access more than 10 times in 6 months | Review justifications | Flag for Legal Counsel discussion |

**Step 3: Submit review outcome**

Legal Counsel completes and signs the P&C Access Review Certificate:

```
EIT P&C ACCESS REVIEW CERTIFICATE

Review period: [FROM DATE] to [TO DATE]
Completed: [DATE]
Primary reviewer: [Legal Counsel name, title]
Secondary reviewer: [EIT Administrator name]

Current eligible members: [COUNT]
Members retained: [COUNT]
Members removed: [COUNT]
Activation incidents reviewed: [COUNT]

Removed members:
  - [NAME] — Reason: ...

Certification:
I certify that all members of the EIT-PAC-Authorized group have been reviewed
and that access is restricted to individuals with a current, legitimate need
for attorney-client privileged or work-product protected incident information.

Legal Counsel: _____________________________ Date: __________
EIT Admin: _____________________________ Date: __________
```

---

## Section 3: Monthly PIM Activation Audit

### 3.1 Purpose

Even with quarterly reviews, the EIT administrator should perform a monthly spot-check on PIM activations to detect anomalies.

### 3.2 Procedure

**Run time:** First business day of each month, ~15 minutes.

1. Navigate to `https://entra.microsoft.com` → **Identity Governance** → **PIM** → **Groups** → `EIT-PAC-Authorized` → **Audit history**
2. Filter to the previous calendar month
3. Review each activation record for:
   - Was the justification specific and plausible? (e.g., "Reviewing EIT-142 for legal session" is good; "review" alone is not)
   - Was the duration appropriate? (extended durations without re-activation warrant investigation)
   - Did the user actually access the incident referenced in the justification? (check EIT_IncidentHistory for that user's updates on that item)
4. Flag any suspicious activations to Legal Counsel immediately

**Red flags:**
- User activated P&C access but made no updates to any P&C incident in EIT
- User repeatedly activates for maximum duration (8 hours) every day
- Activation justification references an incident that does not exist in EIT
- Activation from an unexpected location (check Entra sign-in logs for the activation timestamp)

### 3.3 Activation Anomaly Response

If an anomalous activation is identified:

1. Contact the user directly and ask for clarification
2. If the response is unsatisfactory: suspend the user's eligible PIM assignment immediately
3. Notify Legal Counsel within 24 hours
4. Document the incident in the EIT compliance records

---

## Section 4: Access Review Calendar

| Review | Q1 (Jan) | Q2 (Apr) | Q3 (Jul) | Q4 (Oct) |
|--------|----------|----------|----------|----------|
| EIT Members — All Domains | ✓ | ✓ | ✓ | ✓ |
| EIT Viewers | ✓ | ✓ | ✓ | ✓ |
| EIT-PAC-Authorized | ✓ | — | ✓ | — |
| PIM Activation Audit | Every month | Every month | Every month | Every month |

---

## Section 5: Remediation Procedures

### 5.1 Removing a User from EIT Groups

**Static group:**
1. Navigate to `https://entra.microsoft.com` → **Groups** → select the group → **Members**
2. Find the user → click the ellipsis → **Remove from group**
3. Confirm removal
4. If using dynamic groups: the user must be excluded from the dynamic rule if they should not be included (modify the rule in the group settings)

**PIM eligible assignment:**
1. Navigate to PIM → **Groups** → `EIT-PAC-Authorized` → **Assignments**
2. Find the user in the **Eligible** tab
3. Click **Remove** → confirm

**SharePoint item-level permissions (for P&C incidents):**
After removing from `EIT-PAC-Authorized`, also remove any direct item-level permissions:
1. In the EIT SharePoint site, navigate to each P&C incident the user was granted direct access to
2. Item permissions → check the user's name → remove

> **Finding P&C incidents the user accessed:** Filter `EIT_IncidentHistory` for rows where `UpdatedBy = [user name]` AND `PrivilegedAndConfidential = Yes`. These are the incidents the user interacted with — verify item-level permissions are removed for each.

### 5.2 Emergency Access Revocation

For immediate revocation (e.g., employee termination, security incident):

1. Disable the user's Entra ID account immediately (HR/IT action)
2. Revoke all active sessions: Entra ID → Users → select user → **Revoke sessions**
3. Remove from all EIT groups
4. If user had P&C access: notify Legal Counsel within 1 hour

Emergency revocation takes effect immediately for Entra ID account disable (session revocation). Group removal propagates within minutes to SharePoint.

---

## Section 6: Record Retention

All access review records must be retained for **3 years** (or per applicable regulatory requirement, whichever is longer):

| Record | Retention | Storage Location |
|--------|-----------|-----------------|
| Quarterly review sign-off forms | 3 years | Compliance SharePoint library (restricted) |
| P&C access review certificates | 7 years | Legal records system |
| PIM activation audit exports | 3 years | Compliance SharePoint library |
| Anomaly investigation records | 7 years | Legal records system |
