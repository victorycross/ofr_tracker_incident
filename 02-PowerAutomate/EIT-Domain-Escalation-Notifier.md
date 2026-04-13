# EIT Domain Escalation Notifier — Flow Rebuild Guide

> **Prerequisite:** The SharePoint site, all 4 lists, and `EIT_DomainContacts` rows with real email addresses must exist before creating this flow. See `01-SharePoint/EIT-CREATE-SITE.md`.

> **License note:** This flow uses the Outlook **Send an email (V2)** connector, which requires Power Automate Standard. This is included with Microsoft 365 Business Standard and above via the seeded plan. Verify connector availability on your tenant before building.

---

## Flow Overview

| Property | Value |
|----------|-------|
| Name | `EIT Domain Escalation Notifier` |
| Type | Instant cloud flow |
| Trigger | Power Apps (V2) |
| Input | `IncidentItemID` (Number) — SharePoint ID of the incident |
| Input | `EscalationLevel` (Text) — `Escalate` or `Executive Brief` |
| Output | `Result` (Text) — Confirmation message |
| Purpose | Sends an escalation email to the domain lead or executive contact when an incident is escalated |
| Connections | SharePoint (admin account) + Outlook (admin account or shared mailbox) |

### What This Flow Does

1. Receives the incident ID and escalation level from Power Apps
2. Reads the incident details from `EIT_Incidents`
3. Gets all rows from `EIT_DomainContacts` and filters to find the matching domain
4. Sends an escalation email:
   - `Escalate` → email to `LeadEmail`
   - `Executive Brief` → email to `EscalationEmail`
5. Updates `EscalationThreshold` on the incident to confirm the escalation
6. Returns a confirmation message to Power Apps

> **No OFR analog.** This is a net-new flow with no OFR predecessor.

---

## Step-by-Step Build Instructions

### 1. Create the Flow

1. Navigate to `https://make.powerautomate.com`
2. Click **+ Create** → **Instant cloud flow**
3. Flow name: `EIT Domain Escalation Notifier`
4. Select trigger: **PowerApps (V2)**
5. Click **Create**

### 2. Configure the Trigger — Power Apps (V2)

1. Click the trigger to expand it
2. Add **two inputs**:

   **Input 1:**
   - Click **+ Add an input** → **Number**
   - Name: `IncidentItemID`
   - Description: `SharePoint ID of the incident being escalated`

   **Input 2:**
   - Click **+ Add an input** → **Text**
   - Name: `EscalationLevel`
   - Description: `Escalate or Executive Brief`

### 3. Add Action — Get Item (Read Incident)

1. Click **+ New step**
2. Search for **SharePoint** → select **Get item** (singular)
3. Rename this action to: `Get incident` (click the action title)
4. Configure:
   - **Site Address:** `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker`
   - **List Name:** `EIT_Incidents`
   - **Id:** Select `IncidentItemID` dynamic content from the trigger

### 4. Add Action — Get Items (Read Domain Contacts)

1. Click **+ New step**
2. Search for **SharePoint** → select **Get items**
3. Rename this action to: `Get domain contacts`
4. Configure:
   - **Site Address:** `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker`
   - **List Name:** `EIT_DomainContacts`
   - **Top Count:** `10` *(list only has 5 rows — 10 is a safe ceiling)*

> **Why Get items (not Get item):** `EIT_DomainContacts` is looked up by the incident's `Domain` field value, not by a known integer ID. Retrieving all 5 rows and filtering in the next step is the most reliable approach for small reference lists.

### 5. Add Action — Filter Array (Find Matching Domain Contact)

1. Click **+ New step**
2. Search for **Data Operation** → select **Filter array**
3. Rename this action to: `Find domain contact`
4. Configure:
   - **From:** Dynamic content → `body/value` from Get domain contacts
   - **Condition:**
     - Left side: Click the field → switch to **Expression** tab → enter:
       ```
       item()?['Title']
       ```
       Click **OK**
     - Operator: **is equal to**
     - Right side: Dynamic content → `Domain Value` from Get incident

> **Case sensitivity:** The `item()?['Title']` in EIT_DomainContacts must match the `Domain Value` from the incident exactly. Both fields use the same choice values (e.g., `Privacy & Technology`). If the filter returns an empty array, check for trailing spaces or encoding differences in the DomainContacts rows.

### 6. Add Action — Compose (Extract First Match)

1. Click **+ New step**
2. Search for **Data Operation** → select **Compose**
3. Rename this action to: `Extract contact record`
4. In the **Inputs** field, switch to **Expression** tab and enter:
   ```
   first(body('Find_domain_contact'))
   ```
   Click **OK**

> **Why Compose:** The Filter array returns an array. `first()` safely extracts the single matching record. This Compose output is used in subsequent steps to reference the contact's email fields.

### 7. Add Action — Condition (Determine Escalation Level)

1. Click **+ New step**
2. Search for **Control** → select **Condition**
3. Rename this action to: `Is Executive Brief?`
4. Configure the condition:
   - Left side: Dynamic content → `EscalationLevel` from trigger
   - Operator: **is equal to**
   - Right side: Type `Executive Brief`

This creates two branches: **If yes** and **If no**.

### 8. Add Action (If yes branch) — Send Executive Brief Email

Inside the **If yes** branch:

1. Click **Add an action**
2. Search for **Outlook** → select **Send an email (V2)**
3. Configure:
   - **To:** Expression tab →
     ```
     outputs('Extract_contact_record')?['EscalationEmail']
     ```
   - **Subject:** Expression tab →
     ```
     concat(if(equals(body('Get_incident')?['PrivilegedAndConfidential'], true), '[PRIVILEGED & CONFIDENTIAL] ', ''), '[EXECUTIVE BRIEF] ', body('Get_incident')?['Title'], ' — ', body('Get_incident')?['Domain']?['Value'], ' — ', body('Get_incident')?['Severity']?['Value'])
     ```
   - **Body:** Build using a mix of static text and dynamic content from Get incident:

```
⚠ PRIVILEGED AND CONFIDENTIAL — ATTORNEY-CLIENT PRIVILEGE (if flagged P&C)
   Omit this line if PrivilegedAndConfidential = No

ENTERPRISE INCIDENT TRACKER — EXECUTIVE BRIEF

Incident: [ItemID from Get incident]
Title: [Title from Get incident]
Domain: [Domain Value from Get incident]
Severity: [Severity Value from Get incident]
Status: [Status Value from Get incident]
Escalation Level: Executive Brief
Privileged & Confidential: [PrivilegedAndConfidential from Get incident]
Days Since Last Update: [DaysSinceUpdate from Get incident]

Owner: [Owner from Get incident]
Date Raised: [DateRaised from Get incident]
Last Updated: [LastUpdated from Get incident]

Next Action / Current Status:
[NextAction from Get incident]

---
This notification was generated automatically by the Enterprise Incident Tracker.
To view and update this incident, open the EIT Power App.
If this incident is marked Privileged & Confidential, do not forward this email
or discuss its contents with personnel not on the authorized access list.
```

> **Tip for the body:** Type the static labels manually, then click into each bracket placeholder position and select the matching dynamic content from the **Get incident** step. Do not use actual square brackets in the sent email — remove them and replace with the dynamic content tokens.

### 9. Add Action (If no branch) — Send Domain Lead Email

Inside the **If no** branch (EscalationLevel = 'Escalate'):

1. Click **Add an action**
2. Search for **Outlook** → select **Send an email (V2)**
3. Configure:
   - **To:** Expression tab →
     ```
     outputs('Extract_contact_record')?['LeadEmail']
     ```
   - **Subject:** Expression tab →
     ```
     concat(if(equals(body('Get_incident')?['PrivilegedAndConfidential'], true), '[PRIVILEGED & CONFIDENTIAL] ', ''), '[ESCALATION] ', body('Get_incident')?['Title'], ' — ', body('Get_incident')?['Domain']?['Value'], ' — ', body('Get_incident')?['Severity']?['Value'])
     ```
   - **Body:** Same template as Step 8, replacing `EXECUTIVE BRIEF` with `ESCALATION ALERT` in the header and subject. Include the P&C disclaimer line in the footer if `PrivilegedAndConfidential = Yes`.

### 10. Add Action — Update Item (Confirm Escalation on Incident)

After the Condition block (outside both branches — at the main flow level):

1. Click **+ New step**
2. Search for **SharePoint** → select **Update item**
3. Rename to: `Update incident escalation`
4. Configure:
   - **Site Address:** `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker`
   - **List Name:** `EIT_Incidents`
   - **Id:** Select `IncidentItemID` from the trigger
   - **Title:** Dynamic content → `Title` from Get incident
   - **EscalationThreshold Value:** Dynamic content → `EscalationLevel` from trigger

> **Why update here:** This writes the EscalationLevel back to the incident record, confirming the escalation is formally recorded in the data. Power Apps will refresh after the flow responds, reflecting the updated EscalationThreshold in the UI.

### 11. Add Action — Respond to a PowerApp or Flow

1. Click **+ New step**
2. Search for **Respond** → select **Respond to a PowerApp or flow**
3. Click **+ Add an output**
4. Select **Text**
5. Set:
   - **Title:** `Result`
   - **Value:** Expression tab →
     ```
     concat('Escalation notification sent to ', outputs('Extract_contact_record')?['LeadEmail'])
     ```

### 12. Save and Test

1. Click **Save**
2. Verify `EIT_DomainContacts` has real email addresses in `LeadEmail` and `EscalationEmail` for at least one domain
3. Click **Test** → **Manually** → **Test**
4. Enter:
   - `IncidentItemID`: SharePoint integer ID of any incident in `EIT_Incidents` (check the ID column)
   - `EscalationLevel`: `Escalate`
5. Click **Run flow**
6. Verify:
   - Email received at the domain lead's address
   - `EscalationThreshold` on the incident updated to `Escalate`
   - Repeat with `EscalationLevel = Executive Brief` to verify the alternate branch

### 13. Grant Run-Only Access

1. Go to the flow details page
2. Click **Edit** on the **Run only users** section
3. Add the M365 group that will use the Power App
4. Under **Connections Used**, configure:
   - SharePoint: **Use this connection**
   - Outlook: **Use this connection** (emails will be sent from the admin account's mailbox — consider using a shared mailbox such as `eit-notifications@brightpathtechnology.io` for a clean sender identity)
5. Click **Save**

---

## Connecting to Power Apps

After creating the flow, wire it to the Save Update button in the Issue Detail screen when an escalation is triggered:

```
// In the Save Update button OnSelect (IssueDetailScreen), after the Patch calls:
If(
    ddUpdateType.Selected.Value = "Escalation",
    EITDomainEscalationNotifier.Run(
        varSelectedIncident.ID,
        ddEscalationThreshold.Selected.Value
    )
);
```

If Power Apps doesn't recognise the flow name, go to **Action** menu → **Power Automate** → **Add flow** → select `EIT Domain Escalation Notifier`.

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Filter array returns empty | Check that `EIT_DomainContacts.Title` values exactly match the `Domain` choice values in `EIT_Incidents` — case-sensitive, including the ampersand in "Privacy & Technology" |
| `first()` returns null | If no domain contact row matches, `first()` on an empty array returns null. Add a Condition before the email steps to check `length(body('Find_domain_contact')) > 0` |
| Email not delivered | Verify the `LeadEmail` and `EscalationEmail` values in `EIT_DomainContacts` are valid M365 mailboxes accessible to the Outlook connector account |
| "Access denied" when sending email | The Outlook connector must be authenticated with an account that has a valid Exchange Online mailbox. Re-authenticate the connection if needed. |
| `Domain Value` not available in dynamic content | Use the expression `body('Get_incident')?['Domain']?['Value']` directly in the Filter array condition |
| EscalationThreshold not updating | Verify the Update item action runs after the Condition block (at main flow level, not inside a branch) and that `EscalationLevel` from the trigger is mapped to `EscalationThreshold Value` |
| "Flow not found" in Power Apps | Re-add via Action → Power Automate → Add flow. Ensure you have saved the flow first. |

---

## Flow Diagram

```
Power Apps (V2) trigger
  Input: IncidentItemID (Number)
  Input: EscalationLevel (Text)
    |
    v
Get item: EIT_Incidents [by IncidentItemID]
  → incident details (Title, Domain, Severity, Status, etc.)
    |
    v
Get items: EIT_DomainContacts [all 5 rows]
    |
    v
Filter array: find row where Title = incident.Domain.Value
    |
    v
Compose: first(filtered array) → contact record
    |
    v
Condition: EscalationLevel = 'Executive Brief'?
    |
    +-- YES ──► Send email (Outlook V2)
    |               To: contact.EscalationEmail
    |               Subject: [EXECUTIVE BRIEF] Title — Domain — Severity
    |               Body: full incident detail
    |
    +-- NO ───► Send email (Outlook V2)
                    To: contact.LeadEmail
                    Subject: [ESCALATION] Title — Domain — Severity
                    Body: full incident detail
    |
    v
Update item: EIT_Incidents
  Id: IncidentItemID
  EscalationThreshold: EscalationLevel
    |
    v
Respond to PowerApp
  Output: Result = 'Escalation notification sent to [email]'
```
