# EIT Access Review Reminder Flow — Rebuild Guide

> **Phase 3 Integration.** This flow supports the quarterly access review process described in `EIT-Phase3-Access-Review-Runbook.md`. It is a lightweight notification flow with no data creation — it sends reminder emails to domain leads and the EIT administrator at the start of each quarter.

---

## Flow Overview

| Property | Value |
|----------|-------|
| Name | `EIT Quarterly Access Review Reminder` |
| Type | Scheduled cloud flow |
| Trigger | Recurrence (quarterly) |
| Purpose | Sends access review assignments to domain leads and the EIT administrator at the start of each quarter |
| Connections | Outlook (admin account) + SharePoint (admin account) |

### What This Flow Does

1. Runs on the first business day of each quarter (1 January, 1 April, 1 July, 1 October) at 08:00
2. Reads domain lead email addresses from `EIT_DomainContacts`
3. Sends a personalised access review reminder email to each domain lead with:
   - Review instructions and deadline (10 business days)
   - A link to the access review portal or sign-off template
   - The review scope for their domain
4. Sends a summary notification to the EIT administrator listing all reviews initiated

---

## Step-by-Step Build Instructions

### 1. Create the Flow

1. Navigate to `https://make.powerautomate.com`
2. Click **+ Create** → **Scheduled cloud flow**
3. Flow name: `EIT Quarterly Access Review Reminder`
4. Set schedule:
   - **Starting:** Next 1 January at 08:00 UTC
   - **Repeat every:** `3 Months`
5. Click **Create**

> **Quarterly alignment:** Power Automate's 3-month recurrence starts from the date you set and repeats every 3 months. Set the starting date to the next 1st of January, April, July, or October to align with calendar quarters. If the 1st falls on a weekend, manually adjust to the following Monday, or use a more complex recurrence expression.

---

### 2. Add Action — Get Items (Read Domain Contacts)

1. Click **+ New step**
2. Search for **SharePoint** → select **Get items**
3. Rename to: `Get domain contacts`
4. Configure:
   - **Site Address:** `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker`
   - **List Name:** `EIT_DomainContacts`
   - **Top Count:** `10`

---

### 3. Add Action — Apply to Each (Email Each Domain Lead)

1. Click **+ New step**
2. Search for **Control** → select **Apply to each**
3. Rename to: `For each domain`
4. **Select an output:** Dynamic content → `value` from `Get domain contacts`

Inside the loop:

#### 3a. Send Review Reminder to Domain Lead

1. Click **Add an action**
2. Search for **Outlook** → select **Send an email (V2)**
3. Rename to: `Send review reminder to domain lead`
4. Configure:
   - **To:** Expression → `items('For_each_domain')?['LeadEmail']`
   - **Subject:** Expression →
     ```
     concat('ACTION REQUIRED: EIT Quarterly Access Review — ', items('For_each_domain')?['Title'], ' Domain')
     ```
   - **Body:**

```
Dear [LeadName],

The Enterprise Incident Tracker quarterly access review is due.

As the [Domain] domain lead, you are required to review and certify the
access of all EIT Members assigned to your domain.

REVIEW SCOPE: [Domain] EIT Members
DEADLINE: [10 business days from today]
REVIEW METHOD: [Choose one — see below]

OPTION A — Entra ID Access Reviews (if configured):
1. Navigate to https://myaccess.microsoft.com
2. Click "Access reviews"
3. Find the review assigned to you
4. Approve or deny each member

OPTION B — Manual review:
1. Request the current member list from the EIT administrator
2. Complete the sign-off template (attached to your review assignment email)
3. Return the signed template to david@brightpathtechnology.io

WHAT TO CHECK FOR EACH MEMBER:
- Still employed at the organisation
- Still working in the [Domain] domain
- Has a current business need for EIT access
- Has been active in EIT in the last 90 days (if not, flag as potentially stale)

If you identify any users whose access should be removed, email david@brightpathtechnology.io
immediately — do not wait for the review deadline.

Thank you for completing this review promptly. It is required for our
compliance posture and access governance obligations.

David
EIT Administrator
```

> **Body substitution:** Replace the bracketed placeholders with dynamic content tokens:
> - `[LeadName]` → Expression: `items('For_each_domain')?['LeadName']`
> - `[Domain]` → Expression: `items('For_each_domain')?['Title']`
> - `[10 business days from today]` → Expression: `formatDateTime(addDays(utcNow(), 14), 'dddd, d MMMM yyyy')` *(14 calendar days approximates 10 business days)*

---

### 4. Add Action — Compose (Review Summary for Admin) — Outside Loop

After the Apply to each loop, at the main flow level:

1. Click **+ New step**
2. Search for **Data Operation** → select **Compose**
3. Rename to: `Build admin summary`
4. Expression:
   ```
   concat('EIT Quarterly Access Review initiated for ', string(length(body('Get_domain_contacts')?['value'])), ' domains. Review deadline: ', formatDateTime(addDays(utcNow(), 14), 'dddd, d MMMM yyyy'), '. Domain leads have been notified.')
   ```

---

### 5. Add Action — Send Admin Summary Email

1. Click **+ New step**
2. Search for **Outlook** → select **Send an email (V2)**
3. Rename to: `Send admin summary`
4. Configure:
   - **To:** Static text → `david@brightpathtechnology.io`
   - **CC:** Static text → `[EXEC-ESCALATION-EMAIL]` *(optional — notify CISO or Compliance Officer)*
   - **Subject:** Expression →
     ```
     concat('EIT Quarterly Access Review Initiated — ', formatDateTime(utcNow(), 'MMMM yyyy'))
     ```
   - **Body:**

```
EIT QUARTERLY ACCESS REVIEW — INITIATED

Review period: [QUARTER YEAR]
Initiated: [TODAY'S DATE]
Review deadline: [14 calendar days / 10 business days]

Domain leads notified:
- National Security: [NS-LEAD-NAME] ([NS-LEAD-EMAIL])
- NIS: [NIS-LEAD-NAME] ([NIS-LEAD-EMAIL])
- Privacy & Technology: [PT-LEAD-NAME] ([PT-LEAD-EMAIL])
- Human Capital: [HC-LEAD-NAME] ([HC-LEAD-EMAIL])
- Physical Security: [PS-LEAD-NAME] ([PS-LEAD-EMAIL])

ACTION REQUIRED FROM EIT ADMINISTRATOR:
1. Send the EIT Members list (per domain) to each lead if they request it
2. Monitor for review completions and sign-off returns
3. Process any immediate removal requests within 24 hours
4. After deadline: follow up with any leads who have not completed their review
5. Remove all users identified for removal and document in the compliance record

For the EIT-PAC-Authorized group (reviewed every 6 months):
Check if this quarter is a P&C review quarter and initiate separately
if due. Coordinate with Legal Counsel.

Refer to: EIT-Phase3-Access-Review-Runbook.md for full procedures.
```

> **Tip:** Use Dynamic content from `Build admin summary` Compose for the summary line, or build the body manually with static text and dynamic content tokens.

---

### 6. Save and Test

1. Click **Save**
2. To test: click **Test** → **Manually** → **Run flow**
3. Verify:
   - Each domain lead receives a review reminder email with their domain name in the subject
   - Admin receives the summary email listing all domains
   - All email addresses are correctly populated from `EIT_DomainContacts`
4. Check that the deadline date (14 days from today) appears correctly in the body

---

## P&C Review Variant (Every 6 Months)

This flow sends standard quarterly reviews. For the separate 6-monthly P&C review, create a second flow:

**Flow name:** `EIT P&C Access Review Reminder`
**Schedule:** Every 6 months (1 January and 1 July)
**To:** Legal Counsel email + `david@brightpathtechnology.io`
**Subject:** `ACTION REQUIRED: EIT P&C Access Review — [Month Year]`

Body: Reference `EIT-Phase3-Access-Review-Runbook.md` Section 2 for the review procedure and attach the P&C Access Review Certificate template.

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Domain lead emails missing | Ensure all 5 rows in `EIT_DomainContacts` have valid `LeadEmail` values |
| Flow runs on wrong date | Adjust the recurrence starting date in the trigger settings |
| "Send email" fails | Re-authenticate the Outlook connection |
| Admin summary email shows 0 domains | Check that `Get domain contacts` returns results — verify site URL and list name |

---

## Flow Diagram

```
Recurrence (every 3 months, 08:00 UTC, starting Jan 1)
    |
    v
Get items: EIT_DomainContacts [all 5 rows]
    |
    v
Apply to each: domain contact
    |
    +-- Send email (Outlook V2)
          To: LeadEmail
          Subject: 'ACTION REQUIRED: EIT Quarterly Access Review — [Domain]'
          Body: review instructions + deadline + options
    |
    v (after loop)
Compose: admin summary text
    |
    v
Send email (Outlook V2)
    To: david@brightpathtechnology.io
    Subject: 'EIT Quarterly Access Review Initiated — [Month Year]'
    Body: summary + admin action items
```
