# EIT Teams Notification Extension — Flow Modification Guide

> **Phase 2 Integration.** This guide extends the existing `EIT Domain Escalation Notifier` flow (Phase 1) to post adaptive card notifications to a Microsoft Teams channel alongside the existing escalation email. Phase 1 must be fully deployed before making these changes. See `02-PowerAutomate/EIT-Domain-Escalation-Notifier.md`.

> **Prerequisite:** The Teams channel(s) for escalation notifications must be created and the flow service account must be a member of those channels before the Teams actions will work.

---

## Overview

### What Changes

The `EIT Domain Escalation Notifier` flow currently sends an email (Outlook V2) to the domain lead or executive contact when an escalation is triggered. This extension adds a **Post adaptive card in a chat or channel** action inside each email branch to send a parallel Teams notification.

No existing steps are modified or removed. Two new actions are added — one per branch (Escalate and Executive Brief).

### Teams Channel Structure

Choose one of the following channel patterns before building:

| Pattern | Description | Recommended for |
|---------|-------------|----------------|
| **Unified channel** | Single `EIT-Escalations` channel in the EIT Teams team. All escalation alerts appear in one place. | Smaller teams; centralised oversight |
| **Per-domain channels** | Separate `EIT-NS-Escalations`, `EIT-NIS-Escalations`, etc. channels. Domain-specific notifications. | Larger teams; domain-specific visibility |

This guide uses the **unified channel** pattern. Adapt the Team/Channel configuration in Steps 3 and 4 for per-domain channels if needed.

---

## Step 1: Create the Teams Channel

Before modifying the flow:

1. Open Microsoft Teams
2. Navigate to or create the **Enterprise Incident Tracker** Team
3. Click **+** to add a channel
4. Name: `EIT-Escalations`
5. Description: `Automated escalation notifications from the Enterprise Incident Tracker`
6. Privacy: **Standard** (visible to all team members) or **Private** (restricted) — coordinate with your security team
7. Click **Create**
8. Add the flow service account (the account used for the Outlook connector) as a member of the channel

> **Per-domain variant:** Create five channels: `EIT-NS-Escalations`, `EIT-NIS-Escalations`, `EIT-PT-Escalations`, `EIT-HC-Escalations`, `EIT-PS-Escalations`. In Steps 3 and 4, use a Switch block inside each branch to select the correct channel based on `Domain Value` from the Get incident step.

---

## Step 2: Open the Existing Flow

1. Navigate to `https://make.powerautomate.com`
2. Find and open `EIT Domain Escalation Notifier`
3. Click **Edit**

---

## Step 3: Add Teams Action — If Yes Branch (Executive Brief)

Inside the **If yes** branch (EscalationLevel = 'Executive Brief'), after the existing Send email action:

1. Click **Add an action** (inside the If yes branch, below the existing Send email)
2. Search for **Microsoft Teams** → select **Post adaptive card in a chat or channel**
3. Rename to: `Post Executive Brief card to Teams`
4. Configure:
   - **Post as:** `Flow bot`
   - **Post in:** `Channel`
   - **Team:** Select `Enterprise Incident Tracker` *(or type the team name)*
   - **Channel:** Select `EIT-Escalations`
   - **Adaptive Card:** Paste the following JSON, then substitute dynamic content tokens for the bracketed placeholders:

```json
{
  "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
  "type": "AdaptiveCard",
  "version": "1.4",
  "body": [
    {
      "type": "Container",
      "style": "attention",
      "items": [
        {
          "type": "TextBlock",
          "text": "⚠ EXECUTIVE BRIEF",
          "weight": "Bolder",
          "size": "Medium",
          "color": "Attention"
        }
      ]
    },
    {
      "type": "FactSet",
      "facts": [
        { "title": "Incident", "value": "${ItemID}" },
        { "title": "Title", "value": "${Title}" },
        { "title": "Domain", "value": "${Domain}" },
        { "title": "Severity", "value": "${Severity}" },
        { "title": "Escalation", "value": "Executive Brief" },
        { "title": "Owner", "value": "${Owner}" },
        { "title": "Days Since Update", "value": "${DaysSinceUpdate}" }
      ]
    },
    {
      "type": "TextBlock",
      "text": "This incident has been escalated to Executive Brief level and requires executive sponsor attention.",
      "wrap": true,
      "isSubtle": true
    }
  ],
  "actions": [
    {
      "type": "Action.OpenUrl",
      "title": "Open in EIT",
      "url": "https://make.powerapps.com/environments/[AUTO-GENERATED-ENV-ID]/apps/[AUTO-GENERATED-APP-ID]"
    }
  ]
}
```

**Substitute dynamic content for each `${placeholder}`:**

To substitute dynamic content into the adaptive card JSON, you must use Power Automate expressions directly in the JSON string. Replace each `${placeholder}` with a dynamic content reference using this pattern:

Click into the **Adaptive Card** field, then switch to the **Expression** view or build the JSON as a single expression using `concat()`:

```
concat('{\"$schema\":\"http://adaptivecards.io/schemas/adaptive-card.json\",\"type\":\"AdaptiveCard\",\"version\":\"1.4\",\"body\":[{\"type\":\"Container\",\"style\":\"attention\",\"items\":[{\"type\":\"TextBlock\",\"text\":\"⚠ EXECUTIVE BRIEF\",\"weight\":\"Bolder\",\"size\":\"Medium\",\"color\":\"Attention\"}]},{\"type\":\"FactSet\",\"facts\":[{\"title\":\"Incident\",\"value\":\"', body('Get_incident')?['ItemID'], '\"},{\"title\":\"Title\",\"value\":\"', body('Get_incident')?['Title'], '\"},{\"title\":\"Domain\",\"value\":\"', body('Get_incident')?['Domain']?['Value'], '\"},{\"title\":\"Severity\",\"value\":\"', body('Get_incident')?['Severity']?['Value'], '\"},{\"title\":\"Escalation\",\"value\":\"Executive Brief\"},{\"title\":\"Owner\",\"value\":\"', body('Get_incident')?['Owner'], '\"},{\"title\":\"Days Since Update\",\"value\":\"', string(body('Get_incident')?['DaysSinceUpdate']), '\"}]},{\"type\":\"TextBlock\",\"text\":\"This incident has been escalated to Executive Brief level.\",\"wrap\":true,\"isSubtle\":true}],\"actions\":[{\"type\":\"Action.OpenUrl\",\"title\":\"Open in EIT\",\"url\":\"https://make.powerapps.com/environments/[AUTO-GENERATED-ENV-ID]/apps/[AUTO-GENERATED-APP-ID]\"}]}')
```

> **Simpler alternative:** If the concat expression is unwieldy, paste the static card JSON into the field and manually insert dynamic content tokens by clicking within the JSON string at each placeholder position and selecting the matching dynamic content. Power Automate will insert `@{body('Get_incident')?['ItemID']}` inline.

> **Power App URL:** Replace `[AUTO-GENERATED-ENV-ID]` and `[AUTO-GENERATED-APP-ID]` with the real values from `06-Environment-Config/EIT-ENVIRONMENT-VARIABLES.md` after app publication.

---

## Step 4: Add Teams Action — If No Branch (Escalate)

Inside the **If no** branch (EscalationLevel = 'Escalate'), after the existing Send email action:

1. Click **Add an action** (inside the If no branch, below the existing Send email)
2. Search for **Microsoft Teams** → select **Post adaptive card in a chat or channel**
3. Rename to: `Post Escalation card to Teams`
4. Configure identically to Step 3, with the following differences in the card JSON:
   - Change the header text from `⚠ EXECUTIVE BRIEF` to `⚠ ESCALATION ALERT`
   - Change the `"Escalation"` fact value from `"Executive Brief"` to `"Escalate"`
   - Change the footer text to: `"This incident has been escalated to the domain lead for immediate review."`

---

## Step 5: Save and Test

1. Click **Save**
2. Navigate to a test incident in `EIT_Incidents` with a known domain
3. Trigger the flow manually: from Power Apps IssueDetailScreen, submit an update with UpdateType = Escalation and EscalationThreshold = Escalate
4. Verify:
   - Escalation email received (existing behaviour unchanged)
   - Adaptive card appears in the `EIT-Escalations` Teams channel
   - Card shows correct Incident ID, Title, Domain, Severity, Owner, Days Since Update
   - "Open in EIT" button navigates to the Power App (confirm the URL is correct)
5. Test Executive Brief branch:
   - Update EscalationThreshold to `Executive Brief`
   - Verify the Executive Brief card appears with `⚠ EXECUTIVE BRIEF` header
   - Verify both email and Teams card are sent

---

## Adaptive Card — Field Reference

| Card Field | Source | Expression |
|-----------|--------|-----------|
| Incident | EIT_Incidents.ItemID | `body('Get_incident')?['ItemID']` |
| Title | EIT_Incidents.Title | `body('Get_incident')?['Title']` |
| Domain | EIT_Incidents.Domain | `body('Get_incident')?['Domain']?['Value']` |
| Severity | EIT_Incidents.Severity | `body('Get_incident')?['Severity']?['Value']` |
| Escalation Level | Trigger input | `triggerBody()['EscalationLevel']` |
| Owner | EIT_Incidents.Owner | `body('Get_incident')?['Owner']` |
| Days Since Update | EIT_Incidents.DaysSinceUpdate | `body('Get_incident')?['DaysSinceUpdate']` |

---

## Per-Domain Channel Variant

To post to domain-specific channels instead of a unified channel, add a Switch block inside each branch to select the channel name:

**Before the Post adaptive card action (inside each branch):**

1. Add **Initialize variable** (before the branches, at flow level): `varTeamsChannel` (String, default `EIT-Escalations`)
2. Inside each branch, before the Post card action, add **Switch**:
   - **On:** Expression → `body('Get_incident')?['Domain']?['Value']`
   - Cases:
     - `National Security` → Set `varTeamsChannel` = `EIT-NS-Escalations`
     - `NIS` → Set `varTeamsChannel` = `EIT-NIS-Escalations`
     - `Privacy & Technology` → Set `varTeamsChannel` = `EIT-PT-Escalations`
     - `Human Capital` → Set `varTeamsChannel` = `EIT-HC-Escalations`
     - `Physical Security` → Set `varTeamsChannel` = `EIT-PS-Escalations`
3. In the Post adaptive card action, set **Channel** to Dynamic content → `varTeamsChannel`

> **Channel dynamic content limitation:** The Teams connector's Channel field may require a static selection at design time. If dynamic channel selection is not supported, duplicate the Post adaptive card action for each domain (five actions) and use a Switch to route to the correct one. This is more verbose but reliable.

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Teams action not visible | Microsoft Teams connector must be added to the flow connections. Click the Teams action and sign in with the admin account or flow service account. |
| Card not appearing in channel | Verify the service account is a member of the Teams channel. Flow Bot must have channel posting permission. |
| Card renders with raw JSON | Ensure the adaptive card JSON is valid. Use the [Adaptive Card Designer](https://adaptivecards.io/designer/) to validate and preview. |
| "Open in EIT" button URL broken | Replace the placeholder URL with the real Power App URL from `06-Environment-Config/EIT-ENVIRONMENT-VARIABLES.md`. |
| Dynamic content not substituting | If typing in the card JSON field, use `@{expression}` syntax inline. If using the full concat expression, ensure all JSON is properly escaped (double-quote `"` becomes `\"`). |
| Existing email not sending | The Teams action was placed at the wrong level. Ensure both email and Teams actions are inside the same branch, not replacing each other. |

---

## Updated Flow Diagram

```
Power Apps (V2) trigger
  Input: IncidentItemID, EscalationLevel
    |
    v
Get item: EIT_Incidents [by IncidentItemID]
    |
    v
Get items: EIT_DomainContacts
    |
    v
Filter array: find domain contact
    |
    v
Compose: first(filtered array)
    |
    v
Condition: EscalationLevel = 'Executive Brief'?
    |
    +-- YES ──► Send email (Outlook V2)          [EXISTING]
    |               To: EscalationEmail
    |           Post adaptive card (Teams)         [NEW]
    |               Channel: EIT-Escalations
    |               Card: ⚠ EXECUTIVE BRIEF
    |
    +-- NO ───► Send email (Outlook V2)           [EXISTING]
    |               To: LeadEmail
    |           Post adaptive card (Teams)          [NEW]
    |               Channel: EIT-Escalations
    |               Card: ⚠ ESCALATION ALERT
    |
    v
Update item: EIT_Incidents [EscalationThreshold]  [EXISTING]
    |
    v
Respond to PowerApp                               [EXISTING]
```
