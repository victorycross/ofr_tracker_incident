# EIT ServiceNow Ingest Flow — Rebuild Guide

> **Phase 2 Integration.** This flow is part of the Phase 2 integration layer. Phase 1 must be fully deployed and tested before building this flow. See `04-Documentation/EIT-Phase2-Integration-Roadmap.md`.

> **Prerequisite:** ServiceNow admin cooperation is required to configure the outbound REST notification from ServiceNow. Coordinate with your ServiceNow team to define the trigger conditions and provide them the Power Automate HTTP trigger URL after completing Step 1.

---

## Flow Overview

| Property | Value |
|----------|-------|
| Name | `EIT ServiceNow Ingest` |
| Type | Automated cloud flow |
| Trigger | When an HTTP request is received |
| Purpose | Receives a webhook from ServiceNow when a P1/P2 incident is created or updated; creates or updates the corresponding record in EIT_Incidents |
| Connections | SharePoint (admin account) |
| ServiceNow side | Outbound REST notification rule, triggered on Priority P1/P2 + Category = Security Incident |

### What This Flow Does

1. Receives an HTTP POST from ServiceNow carrying incident fields
2. Checks whether the ServiceNow ticket already exists in EIT_Incidents (deduplication)
3. **If new:** Creates a new incident in `EIT_Incidents`, creates an opening `EIT_IncidentHistory` entry, then updates the incident with the generated `EIT-NNN` ItemID
4. **If existing:** Updates the existing EIT incident's Severity and Status to reflect the latest ServiceNow state
5. Responds HTTP 200 to ServiceNow with the EIT ItemID (for ServiceNow to store as a cross-reference)

---

## Step-by-Step Build Instructions

### 1. Create the Flow and HTTP Trigger

1. Navigate to `https://make.powerautomate.com`
2. Click **+ Create** → **Automated cloud flow**
3. Flow name: `EIT ServiceNow Ingest`
4. Search for trigger: **When an HTTP request is received**
5. Click **Create**

The trigger generates a unique **HTTP POST URL** after you save the flow. You will provide this URL to your ServiceNow administrator to configure as the outbound REST endpoint.

#### 1a. Set the Request Body JSON Schema

In the trigger card, click **Use sample payload to generate schema**, and paste the following ServiceNow payload template:

```json
{
  "sys_id": "abc123def456",
  "number": "INC0012345",
  "priority": "1 - Critical",
  "category": "Security",
  "assignment_group": "Security Operations",
  "short_description": "Ransomware detected on endpoint cluster",
  "description": "Full incident description and initial findings from analyst.",
  "caller_id": "John Smith",
  "opened_at": "2026-02-24T08:30:00Z",
  "state": "In Progress",
  "severity": "1 - High"
}
```

Click **Done**. Power Automate generates the JSON schema from the sample — the trigger fields become available as dynamic content in subsequent steps.

> **Coordinate with ServiceNow admin:** The exact field names in the webhook payload depend on your ServiceNow instance configuration. Confirm the field names used above match your outbound REST notification body. The mapping table in Step 5 references these names.

---

### 2. Add Action — Initialize Variable (Domain Mapping)

ServiceNow assignment groups do not map directly to EIT domains. Use a Switch block to perform the mapping. First, initialize a variable to hold the result.

1. Click **+ New step**
2. Search for **Variables** → select **Initialize variable**
3. Rename to: `Initialize domain variable`
4. Configure:
   - **Name:** `varDomain`
   - **Type:** String
   - **Value:** `NIS` *(safe default — update the Switch in the next step to override for each group)*

---

### 3. Add Action — Switch (Map Assignment Group to EIT Domain)

1. Click **+ New step**
2. Search for **Control** → select **Switch**
3. Rename to: `Map assignment group to domain`
4. **On:** Dynamic content → `assignment_group` from the trigger

Add a **Case** for each ServiceNow assignment group that should map to an EIT domain:

| Case Value (ServiceNow Assignment Group) | Action |
|------------------------------------------|--------|
| `Security Operations` | Set variable `varDomain` = `NIS` |
| `Privacy & Compliance` | Set variable `varDomain` = `Privacy & Technology` |
| `HR Security` | Set variable `varDomain` = `Human Capital` |
| `Physical Security` | Set variable `varDomain` = `Physical Security` |
| `National Security Ops` | Set variable `varDomain` = `National Security` |

> **Customise this mapping:** The case values above are placeholders. Replace them with the actual ServiceNow assignment group names used in your environment. Obtain the complete list from your ServiceNow administrator during the field mapping workshop.

For each Case:
1. Click **Add a case**
2. Set the **Equals** value to the ServiceNow group name
3. Inside the case: search **Variables** → **Set variable**
   - **Name:** `varDomain`
   - **Value:** the corresponding EIT domain string

Add a **Default** case with `Set variable varDomain = NIS` to handle unmapped groups.

---

### 4. Add Action — Initialize Variable (Severity Mapping)

1. Click **+ New step**
2. Search for **Variables** → select **Initialize variable**
3. Rename to: `Initialize severity variable`
4. Configure:
   - **Name:** `varSeverity`
   - **Type:** String
   - **Value:** `Moderate`

---

### 5. Add Action — Switch (Map ServiceNow Priority to EIT Severity)

1. Click **+ New step**
2. Search for **Control** → select **Switch**
3. Rename to: `Map priority to severity`
4. **On:** Dynamic content → `priority` from the trigger

Add Cases:

| Case Value (ServiceNow Priority) | Set varSeverity |
|----------------------------------|-----------------|
| `1 - Critical` | `Critical` |
| `2 - High` | `High` |
| `3 - Moderate` | `Moderate` |
| `4 - Low` | `Low` |
| `5 - Planning` | `Informational` |

Default case: `Set variable varSeverity = Moderate`

> **Priority string format:** ServiceNow displays priority as `1 - Critical`, `2 - High`, etc. Confirm the exact string format in your ServiceNow instance's outbound payload. The priority field may be a numeric string (`"1"`) or a display label. Adjust case values accordingly.

---

### 6. Add Action — Get Items (Deduplication Check)

Before creating a new incident, check whether the ServiceNow ticket already exists in EIT_Incidents.

1. Click **+ New step**
2. Search for **SharePoint** → select **Get items**
3. Rename to: `Check for existing EIT record`
4. Configure:
   - **Site Address:** `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker`
   - **List Name:** `EIT_Incidents`
   - **Filter Query:** Click **Show advanced options** →
     ```
     RelatedItemIDs eq '[number from trigger]'
     ```
     Use the Expression tab to build this:
     ```
     concat('RelatedItemIDs eq ''', triggerBody()?['number'], '''')
     ```
     Or type the OData filter directly as: `RelatedItemIDs eq 'INC0012345'` and substitute the dynamic content `number` field.
   - **Top Count:** `1`

> **OData filter note:** The `RelatedItemIDs` column is a single-line text field. SharePoint OData equality filter on text is: `RelatedItemIDs eq 'value'`. If the column stores multiple IDs (comma-separated), use `substringof` instead: `substringof('INC0012345', RelatedItemIDs) eq true`.

---

### 7. Add Action — Condition (Is This a Duplicate?)

1. Click **+ New step**
2. Search for **Control** → select **Condition**
3. Rename to: `Is duplicate?`
4. Configure:
   - Left side: Expression tab →
     ```
     length(body('Check_for_existing_EIT_record')?['value'])
     ```
   - Operator: **is greater than**
   - Right side: `0`

This creates two branches: **If yes** (duplicate) and **If no** (new incident).

---

### 8. If Yes Branch — Update Existing Incident

Inside the **If yes** branch, the ServiceNow incident already exists in EIT. Update it to reflect the latest state.

#### 8a. Compose — Extract Existing Record

1. Click **Add an action** (inside If yes)
2. Search for **Data Operation** → select **Compose**
3. Rename to: `Extract existing record`
4. Expression:
   ```
   first(body('Check_for_existing_EIT_record')?['value'])
   ```

#### 8b. Update Item — Sync Severity and Status

1. Click **Add an action**
2. Search for **SharePoint** → select **Update item**
3. Rename to: `Update existing EIT incident`
4. Configure:
   - **Site Address:** `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker`
   - **List Name:** `EIT_Incidents`
   - **Id:** Expression →
     ```
     outputs('Extract_existing_record')?['ID']
     ```
   - **Title:** Expression →
     ```
     outputs('Extract_existing_record')?['Title']
     ```
   - **Severity Value:** Dynamic content → `varSeverity`
   - **LastUpdated:** Expression → `utcNow()`

#### 8c. Create Item — History Note for Update

1. Click **Add an action**
2. Search for **SharePoint** → select **Create item**
3. Rename to: `Create history note (update)`
4. Configure:
   - **Site Address:** `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker`
   - **List Name:** `EIT_IncidentHistory`
   - **Title:** Expression → `outputs('Extract_existing_record')?['ItemID']`
   - **ParentItemID:** Expression → `outputs('Extract_existing_record')?['ItemID']`
   - **UpdateDate:** Expression → `utcNow()`
   - **StatusAtUpdate Value:** Static text → `Active`
   - **SeverityAtUpdate Value:** Dynamic content → `varSeverity`
   - **UpdateType Value:** Static text → `Status Change`
   - **Notes:** Expression →
     ```
     concat('ServiceNow status update received for ', triggerBody()?['number'], '. Priority: ', triggerBody()?['priority'])
     ```
   - **UpdatedBy:** Static text → `EIT ServiceNow Ingest (automated)`
   - **PrivilegedAndConfidential:** Static → `false`

---

### 9. If No Branch — Create New Incident

Inside the **If no** branch, this is a new ServiceNow ticket that has never been seen in EIT.

#### 9a. Create Item — New EIT_Incidents Record

1. Click **Add an action** (inside If no)
2. Search for **SharePoint** → select **Create item**
3. Rename to: `Create new EIT incident`
4. Configure:
   - **Site Address:** `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker`
   - **List Name:** `EIT_Incidents`

| Field | Value Type | Value |
|-------|-----------|-------|
| **Title** | Dynamic content | `short_description` from trigger |
| **ItemID** | Static text | `EIT-PENDING` *(will be updated in Step 9c)* |
| **Owner** | Dynamic content | `caller_id` from trigger |
| **Domain Value** | Dynamic content | `varDomain` |
| **Priority Value** | Static text | `High` *(EIT Priority ≠ Severity; set default)* |
| **IncidentType Value** | Static text | `Cyber` *(adjust per ServiceNow category mapping)* |
| **Severity Value** | Dynamic content | `varSeverity` |
| **Status Value** | Static text | `New` |
| **EscalationThreshold Value** | Static text | `Not Escalated` |
| **SourceSystem Value** | Static text | `ServiceNow` |
| **ConfidentialityLevel Value** | Static text | `Internal` |
| **RelatedItemIDs** | Dynamic content | `number` from trigger *(ServiceNow ticket number)* |
| **DateRaised** | Expression | `utcNow()` |
| **LastUpdated** | Expression | `utcNow()` |
| **NextAction** | Dynamic content | `description` from trigger |
| **DaysSinceUpdate** | Static number | `0` |
| **StalenessFlag Value** | Static text | `Current` |
| **PrivilegedAndConfidential** | Static | `false` |

> **IncidentType mapping:** ServiceNow category values vary by instance. Build a Switch block (as in Steps 3–5) to map ServiceNow `category` → EIT `IncidentType`. Common mappings: "Security" → "Cyber", "Privacy" → "Privacy", "HR" → "Human Capital Issue". Default to "Cyber" for security-category tickets flowing from ServiceNow.

#### 9b. Compose — Generate EIT ItemID

1. Click **Add an action**
2. Search for **Data Operation** → select **Compose**
3. Rename to: `Generate ItemID`
4. Expression (see also `flow-expressions/EIT-servicenow-itemid.txt`):
   ```
   concat('EIT-', string(outputs('Create_new_EIT_incident')?['body/ID']))
   ```

#### 9c. Update Item — Set Real ItemID

1. Click **Add an action**
2. Search for **SharePoint** → select **Update item**
3. Rename to: `Set ItemID on new incident`
4. Configure:
   - **Site Address:** `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker`
   - **List Name:** `EIT_Incidents`
   - **Id:** Expression →
     ```
     outputs('Create_new_EIT_incident')?['body/ID']
     ```
   - **Title:** Dynamic content → `short_description` from trigger
   - **ItemID:** Dynamic content → `Outputs` from `Generate ItemID`

#### 9d. Create Item — Opening History Entry

1. Click **Add an action**
2. Search for **SharePoint** → select **Create item**
3. Rename to: `Create opening history entry`
4. Configure:
   - **Site Address:** `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker`
   - **List Name:** `EIT_IncidentHistory`
   - **Title:** Dynamic content → `Outputs` from `Generate ItemID`
   - **ParentItemID:** Dynamic content → `Outputs` from `Generate ItemID`
   - **UpdateDate:** Expression → `utcNow()`
   - **StatusAtUpdate Value:** Static text → `New`
   - **SeverityAtUpdate Value:** Dynamic content → `varSeverity`
   - **UpdateType Value:** Static text → `Initial Entry`
   - **Notes:** Expression →
     ```
     concat('Auto-imported from ServiceNow ', triggerBody()?['number'], '. Assignment group: ', triggerBody()?['assignment_group'])
     ```
   - **UpdatedBy:** Static text → `EIT ServiceNow Ingest (automated)`
   - **PrivilegedAndConfidential:** Static → `false`

---

### 10. Add Action — Response (HTTP 200 to ServiceNow)

After the Condition block (at main flow level, outside both branches):

1. Click **+ New step**
2. Search for **Request** → select **Response**
3. Rename to: `Respond to ServiceNow`
4. Configure:
   - **Status Code:** `200`
   - **Headers:** Click **+ Add new parameter** → **Headers**
     - Key: `Content-Type`
     - Value: `application/json`
   - **Body:**
     ```json
     {
       "status": "accepted",
       "eit_item_id": "@{if(greater(length(body('Check_for_existing_EIT_record')?['value']), 0), outputs('Extract_existing_record')?['ItemID'], outputs('Generate_ItemID'))}"
     }
     ```

> **Response note:** This returns the EIT ItemID to ServiceNow so the ServiceNow admin can optionally store it in a cross-reference field on the ServiceNow incident. This enables two-way traceability between systems.

---

### 11. Save and Test

1. Click **Save**
2. Copy the **HTTP POST URL** from the trigger card — you will need this for the ServiceNow configuration
3. To test without ServiceNow: use Postman or curl to POST the sample JSON payload from Step 1a to the trigger URL:
   ```bash
   curl -X POST "[TRIGGER-URL]" \
     -H "Content-Type: application/json" \
     -d '{"sys_id":"test001","number":"INC0099999","priority":"1 - Critical","category":"Security","assignment_group":"Security Operations","short_description":"Test incident from curl","description":"Test description","caller_id":"Test User","opened_at":"2026-02-24T10:00:00Z","state":"New","severity":"1 - High"}'
   ```
4. Verify in SharePoint:
   - New item in `EIT_Incidents` with correct Severity (Critical), Domain (NIS), SourceSystem (ServiceNow), RelatedItemIDs (INC0099999)
   - ItemID set to `EIT-NNN` (not `EIT-PENDING`)
   - Opening entry in `EIT_IncidentHistory` referencing the ServiceNow number
5. Re-send the same payload and verify:
   - No duplicate created — existing item updated instead
   - New history entry added to the existing record

---

## ServiceNow Configuration

Provide the following to your ServiceNow administrator:

### Outbound REST Configuration

| Setting | Value |
|---------|-------|
| **Endpoint URL** | The HTTP POST URL from Step 1 |
| **Method** | POST |
| **Content-Type** | application/json |
| **Authentication** | None (Power Automate HTTP trigger does not require auth by default — consider adding an API key header if your security policy requires it) |
| **Trigger condition** | Priority = 1 - Critical OR Priority = 2 - High; Category = Security |
| **Trigger events** | On insert (new incident) + On update (priority change, state change) |

### Payload Template

```json
{
  "sys_id": "${sys_id}",
  "number": "${number}",
  "priority": "${priority_label}",
  "category": "${category_label}",
  "assignment_group": "${assignment_group.name}",
  "short_description": "${short_description}",
  "description": "${description}",
  "caller_id": "${caller_id.name}",
  "opened_at": "${opened_at}",
  "state": "${state_label}",
  "severity": "${severity_label}"
}
```

> **`${field.name}` notation:** These are ServiceNow script include reference fields. Your ServiceNow administrator will adapt this to the actual outbound notification script syntax used in your instance.

---

## Field Mapping Reference

| ServiceNow Field | EIT Column | Mapping Logic |
|-----------------|------------|---------------|
| `short_description` | `Title` | Direct |
| `number` | `RelatedItemIDs` | Direct (stored for cross-reference) |
| `priority` | `Severity` | Switch: 1→Critical, 2→High, 3→Moderate, 4→Low, 5→Informational |
| `assignment_group` | `Domain` | Switch (customise per Step 3) |
| `description` | `NextAction` | Direct |
| `caller_id` | `Owner` | Direct (ServiceNow display name) |
| `category` | `IncidentType` | Switch: Security→Cyber, Privacy→Privacy, etc. |
| *(auto)* | `SourceSystem` | Static: `ServiceNow` |
| *(auto)* | `Status` | Static: `New` (on create) |
| *(auto)* | `EscalationThreshold` | Static: `Not Escalated` |
| *(auto)* | `ConfidentialityLevel` | Static: `Internal` (P&C not inherited from ServiceNow) |
| *(auto)* | `DateRaised` | `utcNow()` |
| *(auto)* | `ItemID` | Generated: `EIT-` + SharePoint list item ID |

> **P&C designation note:** Privileged & Confidential status is not inherited from ServiceNow. EIT operators must manually apply the P&C flag if an auto-ingested incident meets the legal criteria. See `EIT-Access-Control-Guide.md`.

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Duplicate records created | Check that the `RelatedItemIDs` filter in Step 6 uses the exact ServiceNow `number` value. The filter is case-sensitive. |
| ItemID remains `EIT-PENDING` | The Update item in Step 9c failed. Check the `ID` expression references the correct action name `Create_new_EIT_incident`. |
| Domain maps to wrong value | Confirm the `assignment_group` values in the Switch cases match exactly what ServiceNow sends. Add a Compose action to inspect the raw trigger body. |
| HTTP 400 from Power Automate | The incoming payload doesn't match the JSON schema. Re-generate the schema using the exact ServiceNow payload format. |
| Severity not mapping | Check that the `priority` string format matches your Switch cases. ServiceNow may send `"1"` (numeric) rather than `"1 - Critical"` (label). Inspect the raw body and adjust cases. |
| Flow not triggering | Confirm the ServiceNow outbound REST notification is pointing to the correct trigger URL and that the HTTP method is POST with Content-Type: application/json. |
| "Access denied" to SharePoint | Re-authenticate the SharePoint connection. The connection must use an account with EIT site contributor access. |

---

## Flow Diagram

```
HTTP Trigger (POST from ServiceNow)
  Payload: sys_id, number, priority, category,
           assignment_group, short_description,
           description, caller_id, opened_at, state
    |
    v
Switch: Map assignment_group → varDomain
  Security Operations → NIS
  Privacy & Compliance → Privacy & Technology
  ... (customise per tenant)
    |
    v
Switch: Map priority → varSeverity
  1 - Critical → Critical
  2 - High → High
  3 - Moderate → Moderate
  ...
    |
    v
Get items: EIT_Incidents
  Filter: RelatedItemIDs = trigger.number
  Top: 1
    |
    v
Condition: length(results) > 0?
    |
    +-- YES (duplicate) ──► Update item: EIT_Incidents
    |                            Severity = varSeverity
    |                            LastUpdated = utcNow()
    |                       Create item: EIT_IncidentHistory
    |                            Notes = 'ServiceNow status update'
    |
    +-- NO (new) ──────► Create item: EIT_Incidents
                              ItemID = 'EIT-PENDING' (temp)
                              SourceSystem = 'ServiceNow'
                              RelatedItemIDs = trigger.number
                              [all mapped fields]
                         Compose: EIT-NNN = concat('EIT-', created.ID)
                         Update item: EIT_Incidents
                              ItemID = composed EIT-NNN
                         Create item: EIT_IncidentHistory
                              Notes = 'Auto-imported from ServiceNow [number]'
    |
    v
Response: HTTP 200
  Body: { status: "accepted", eit_item_id: "[EIT-NNN]" }
```
