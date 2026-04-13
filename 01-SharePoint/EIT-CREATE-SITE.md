# Enterprise Incident Tracker — SharePoint Site & List Creation Guide

> **Prerequisite:** Fill in `06-Environment-Config/EIT-ENVIRONMENT-VARIABLES.md` with your target tenant values before starting. Replace all `[PLACEHOLDER]` values using the `EIT-find-and-replace-checklist.md`.

---

## Step 1: Create the SharePoint Team Site

1. Navigate to `https://papercutscafe.sharepoint.com`
2. Click **+ Create site** (top-left)
3. Select **Team site**
4. Configure:
   - **Site name:** `Enterprise Incident Tracker`
   - **Group email alias:** `EnterpriseIncidentTracker` (auto-suggested)
   - **Site address:** Verify it becomes `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker`
   - **Privacy:** Private — only members can access this site
   - **Language:** English
5. Click **Next**, add initial members (domain leads + EIT admins), then **Finish**
6. Wait for the site to provision (~30 seconds)

**Verify:** Navigate to `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker` and confirm the site loads.

---

## Step 2: Create List — EIT_Incidents

This is the primary incident tracking list. Each row represents one enterprise incident.

1. From the site home page, click **+ New** → **List**
2. Select **Blank list**
3. Name: `EIT_Incidents`
4. Click **Create**

### Add Columns

The `Title` column already exists. Add the following 19 columns in order:

| # | Column Name | How to Create |
|---|-------------|--------------|
| 1 | **Title** | *(already exists)* — No changes needed. Max 255 characters. |
| 2 | **ItemID** | Click **+ Add column** → **Single line of text** → Name: `ItemID` → Save |
| 3 | **Owner** | Click **+ Add column** → **Single line of text** → Name: `Owner` → Save |
| 4 | **Domain** | Click **+ Add column** → **Choice** → Name: `Domain` → Choices: `National Security`, `NIS`, `Privacy & Technology`, `Human Capital`, `Physical Security` → Save |
| 5 | **SubDomain** | Click **+ Add column** → **Single line of text** → Name: `SubDomain` → Save |
| 6 | **Severity** | Click **+ Add column** → **Choice** → Name: `Severity` → Choices: `Critical`, `High`, `Moderate`, `Low`, `Informational` → Default value: `Moderate` → Save |
| 7 | **Priority** | Click **+ Add column** → **Choice** → Name: `Priority` → Choices: `High`, `Medium`, `Low` → Save |
| 8 | **Status** | Click **+ Add column** → **Choice** → Name: `Status` → Choices: `New`, `Active`, `Monitoring`, `Escalated`, `Contained`, `Closed` → Save |
| 9 | **IncidentType** | Click **+ Add column** → **Choice** → Name: `IncidentType` → Choices: `Breach`, `Vulnerability`, `Policy Violation`, `Threat`, `Operational`, `Personnel`, `Physical` → Save |
| 10 | **SourceSystem** | Click **+ Add column** → **Choice** → Name: `SourceSystem` → Choices: `Power Apps`, `ServiceNow`, `iSight`, `Manual Entry` → Default value: `Power Apps` → Save |
| 11 | **DateRaised** | Click **+ Add column** → **Date and time** → Name: `DateRaised` → Include time: **No** → Save |
| 12 | **DateContained** | Click **+ Add column** → **Date and time** → Name: `DateContained` → Include time: **No** → Save |
| 13 | **LastUpdated** | Click **+ Add column** → **Date and time** → Name: `LastUpdated` → Include time: **No** → Save |
| 14 | **NextAction** | Click **+ Add column** → **Multiple lines of text** → Name: `NextAction` → Type: **Plain text** → Save |
| 15 | **EscalationThreshold** | Click **+ Add column** → **Choice** → Name: `EscalationThreshold` → Choices: `Not Escalated`, `Watch`, `Escalate`, `Executive Brief` → Default value: `Not Escalated` → Save |
| 16 | **DaysSinceUpdate** | Click **+ Add column** → **Number** → Name: `DaysSinceUpdate` → Decimal places: **0** → Save |
| 17 | **StalenessFlag** | Click **+ Add column** → **Choice** → Name: `StalenessFlag` → Choices: `Current`, `Aging`, `Stale` → Save |
| 18 | **ConfidentialityLevel** | Click **+ Add column** → **Choice** → Name: `ConfidentialityLevel` → Choices: `Internal`, `Restricted`, `Confidential` → Default value: `Internal` → Save |
| 19 | **RelatedItemIDs** | Click **+ Add column** → **Single line of text** → Name: `RelatedItemIDs` → Save |
| 20 | **PrivilegedAndConfidential** | Click **+ Add column** → **Yes/No** → Name: `PrivilegedAndConfidential` → Default value: **No** → Save |

> **Critical:** The `Domain` choice values must be entered exactly as shown (case-sensitive, ampersand in "Privacy & Technology"). These values are matched by the Power Automate escalation flow against `EIT_DomainContacts.Title`.

> **Critical:** `Severity` and `Priority` are two separate columns serving different purposes. `Severity` is the ERM-aligned classification (5 tiers). `Priority` is the operational triage field (3 tiers). Both are required.

### Verify EIT_Incidents

Click through each column header to confirm all 20 columns exist with correct types. Default values must be set on: `Severity` (Moderate), `SourceSystem` (Power Apps), `EscalationThreshold` (Not Escalated), `ConfidentialityLevel` (Internal), `PrivilegedAndConfidential` (No).

**Reference schema:** See `EIT_Incidents-schema.json` for the full column specification.

---

## Step 3: Create List — EIT_IncidentHistory

This is an append-only audit trail for all incident updates. Records must never be edited or deleted after creation.

1. From the site home page, click **+ New** → **List**
2. Select **Blank list**
3. Name: `EIT_IncidentHistory`
4. Click **Create**

### Add Columns

| # | Column Name | How to Create |
|---|-------------|--------------|
| 1 | **Title** | *(already exists)* — Used to store the ParentItemID. |
| 2 | **ParentItemID** | Click **+ Add column** → **Single line of text** → Name: `ParentItemID` → Save |
| 3 | **UpdateDate** | Click **+ Add column** → **Date and time** → Name: `UpdateDate` → Include time: **Yes** → Save |
| 4 | **StatusAtUpdate** | Click **+ Add column** → **Choice** → Name: `StatusAtUpdate` → Choices: `New`, `Active`, `Monitoring`, `Escalated`, `Contained`, `Closed` → Save |
| 5 | **Notes** | Click **+ Add column** → **Multiple lines of text** → Name: `Notes` → Type: **Plain text** → Save |
| 6 | **UpdatedBy** | Click **+ Add column** → **Single line of text** → Name: `UpdatedBy` → Save |
| 7 | **SeverityAtUpdate** | Click **+ Add column** → **Choice** → Name: `SeverityAtUpdate` → Choices: `Critical`, `High`, `Moderate`, `Low`, `Informational` → Save |
| 8 | **UpdateType** | Click **+ Add column** → **Choice** → Name: `UpdateType` → Choices: `Status Change`, `Evidence Added`, `Escalation`, `Containment`, `Closure` → Default value: `Status Change` → Save |
| 9 | **PrivilegedAndConfidential** | Click **+ Add column** → **Yes/No** → Name: `PrivilegedAndConfidential` → Default value: **No** → Save |

> **Note:** `StatusAtUpdate` choices must include `Contained` to match EIT_Incidents.Status — this is a change from OFR_UpdateHistory.

### Verify EIT_IncidentHistory

Confirm 9 columns exist. `UpdateType` must default to `Status Change`, `PrivilegedAndConfidential` must default to `No`. This list is append-only — do not configure any delete or edit permissions for standard users.

**Reference schema:** See `EIT_IncidentHistory-schema.json`.

---

## Step 4: Create List — EIT_IntakeQueue

This is the triage queue for newly submitted incidents.

1. From the site home page, click **+ New** → **List**
2. Select **Blank list**
3. Name: `EIT_IntakeQueue`
4. Click **Create**

### Add Columns

| # | Column Name | How to Create |
|---|-------------|--------------|
| 1 | **Title** | *(already exists)* — Incident title as submitted. |
| 2 | **Owner** | Click **+ Add column** → **Single line of text** → Name: `Owner` → Save |
| 3 | **Priority** | Click **+ Add column** → **Choice** → Name: `Priority` → Choices: `High`, `Medium`, `Low` → Save |
| 4 | **Description** | Click **+ Add column** → **Multiple lines of text** → Name: `Description` → Type: **Plain text** → Save |
| 5 | **Domain** | Click **+ Add column** → **Choice** → Name: `Domain` → Choices: `National Security`, `NIS`, `Privacy & Technology`, `Human Capital`, `Physical Security` → Save |
| 6 | **IncidentType** | Click **+ Add column** → **Choice** → Name: `IncidentType` → Choices: `Breach`, `Vulnerability`, `Policy Violation`, `Threat`, `Operational`, `Personnel`, `Physical` → Save |
| 7 | **DateSubmitted** | Click **+ Add column** → **Date and time** → Name: `DateSubmitted` → Include time: **No** → Save |
| 8 | **TriageStatus** | Click **+ Add column** → **Choice** → Name: `TriageStatus` → Choices: `Pending`, `Promoted`, `Dismissed`, `Accepted`, `Rejected` → Default value: `Pending` → Save |
| 9 | **SubmissionSource** | Click **+ Add column** → **Choice** → Name: `SubmissionSource` → Choices: `Power Apps`, `Email Forward`, `Manual Entry` → Default value: `Power Apps` → Save |
| 10 | **PrivilegedAndConfidential** | Click **+ Add column** → **Yes/No** → Name: `PrivilegedAndConfidential` → Default value: **No** → Save |

> **Important:** `TriageStatus` must have all 5 choice values and default to `Pending`:
> - `Pending` — default on submission
> - `Promoted` — set by the Power Automate Intake Promotion flow
> - `Dismissed` — set by the Power Apps Dashboard dismiss action
> - `Accepted` — set by the Power Apps Intake Review panel Accept button
> - `Rejected` — set by the Power Apps Intake Review panel Reject button

> **Important:** `Domain` choice values must match `EIT_Incidents.Domain` exactly (case-sensitive).

### Verify EIT_IntakeQueue

Confirm 10 columns exist. `TriageStatus` must default to `Pending` and have all 5 choices. `PrivilegedAndConfidential` must default to `No`.

**Reference schema:** See `EIT_IntakeQueue-schema.json`.

---

## Step 5: Create List — EIT_DomainContacts

This is a static reference list with exactly 5 rows — one per enterprise domain. It powers the escalation notification flow.

1. From the site home page, click **+ New** → **List**
2. Select **Blank list**
3. Name: `EIT_DomainContacts`
4. Click **Create**

### Add Columns

| # | Column Name | How to Create |
|---|-------------|--------------|
| 1 | **Title** | *(already exists)* — Domain name. Must match Domain choice values exactly. |
| 2 | **LeadName** | Click **+ Add column** → **Single line of text** → Name: `LeadName` → Save |
| 3 | **LeadEmail** | Click **+ Add column** → **Single line of text** → Name: `LeadEmail` → Save |
| 4 | **EscalationEmail** | Click **+ Add column** → **Single line of text** → Name: `EscalationEmail` → Save |
| 5 | **ActiveIncidentCount** | Click **+ Add column** → **Number** → Name: `ActiveIncidentCount` → Decimal places: **0** → Save |

### Add the 5 Domain Rows

After creating the columns, immediately add the 5 static rows. Click **+ New** for each:

| Title | LeadName | LeadEmail | EscalationEmail | ActiveIncidentCount |
|---|---|---|---|---|
| `National Security` | [NS-LEAD-NAME] | [NS-LEAD-EMAIL] | [EXEC-ESCALATION-EMAIL] | 0 |
| `NIS` | [NIS-LEAD-NAME] | [NIS-LEAD-EMAIL] | [EXEC-ESCALATION-EMAIL] | 0 |
| `Privacy & Technology` | [PT-LEAD-NAME] | [PT-LEAD-EMAIL] | [EXEC-ESCALATION-EMAIL] | 0 |
| `Human Capital` | [HC-LEAD-NAME] | [HC-LEAD-EMAIL] | [EXEC-ESCALATION-EMAIL] | 0 |
| `Physical Security` | [PS-LEAD-NAME] | [PS-LEAD-EMAIL] | [EXEC-ESCALATION-EMAIL] | 0 |

> **Critical:** Replace all `[PLACEHOLDER]` values before saving rows. The Power Automate escalation flow performs a case-sensitive filter on the `Title` field — "Privacy & Technology" must be entered with the ampersand, not "Privacy and Technology".

### Verify EIT_DomainContacts

Confirm 5 columns and exactly 5 rows. Verify `Title` values match the Domain choices in `EIT_Incidents` and `EIT_IntakeQueue` exactly.

**Reference schema:** See `EIT_DomainContacts-schema.json`.

---

## Step 6: Load Sample Data (Optional but Recommended)

Sample CSV files are provided in `sample-data/` for testing the full lifecycle before going live. Load at least 15 records (3 per domain) to test all dashboard KPIs and filter combinations.

- `EIT_Incidents-sample.csv` — 15 records with varied domains, severities, statuses, and staleness
- `EIT_IncidentHistory-sample.csv` — update history entries across incidents
- `EIT_IntakeQueue-sample.csv` — 5 pending intake items (one per domain) for triage testing
- `EIT_DomainContacts-sample.csv` — 5 placeholder rows (replace with real contact data)

### To Import:

**Option A — SharePoint Quick Edit (recommended for bulk):**
1. Open each list
2. Click **Edit in grid view**
3. Copy-paste data from CSV rows into the grid
4. Click **Exit grid view** to save

**Option B — Manual Entry:**
1. Open each list
2. Click **+ New** for each row
3. Fill in field values from the CSV

> **Note:** SharePoint auto-generates the `ID` column (integer). Do not attempt to set it manually. The `ItemID` field in `EIT_Incidents` (e.g., `EIT-001`) is a separate text column that you set explicitly when loading sample data.

---

## Step 7: Final Verification

After creating all four lists and optionally loading sample data:

1. Navigate to `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker`
2. Click **Site contents** (left nav or gear icon)
3. Verify all four lists appear:
   - `EIT_Incidents`
   - `EIT_IncidentHistory`
   - `EIT_IntakeQueue`
   - `EIT_DomainContacts`
4. Open each list and confirm column count matches the schema JSONs:
   - `EIT_Incidents` — 20 columns
   - `EIT_IncidentHistory` — 9 columns
   - `EIT_IntakeQueue` — 10 columns
   - `EIT_DomainContacts` — 5 columns, 5 rows
5. Verify default values are set correctly on choice columns with defaults
6. Verify `EIT_DomainContacts` has exactly 5 rows with `Title` values matching the Domain choices

**SharePoint setup is complete.** Proceed to `02-PowerAutomate/` to build the automation flows.

---

## Checklist Summary

- [ ] SharePoint site created at `https://papercutscafe.sharepoint.com/sites/enterpriseincidentracker`
- [ ] `EIT_Incidents` created — 19 columns, default values set on 4 columns
- [ ] `EIT_IncidentHistory` created — 8 columns
- [ ] `EIT_IntakeQueue` created — 9 columns, `TriageStatus` defaults to Pending
- [ ] `EIT_DomainContacts` created — 5 columns, 5 rows entered with real contact data
- [ ] Sample data loaded (optional)
- [ ] All `[PLACEHOLDER]` values replaced in `EIT_DomainContacts` rows
- [ ] Site contents verified — 4 lists visible
