# OFR Issue Tracker — SharePoint Site & List Creation Guide

> **Prerequisite:** Fill in `06-Environment-Config/ENVIRONMENT-VARIABLES.md` with your target tenant values before starting.

---

## Step 1: Create the SharePoint Team Site

1. Navigate to `https://[TENANT].sharepoint.com`
2. Click **+ Create site** (top-left)
3. Select **Team site**
4. Configure:
   - **Site name:** `OFR Issue Tracker`
   - **Group email alias:** `OFRIssueTracker` (auto-suggested)
   - **Site address:** Verify it becomes `https://[TENANT].sharepoint.com/sites/OFRIssueTracker`
   - **Privacy:** Private — only members can access this site
   - **Language:** English
5. Click **Next**, add any initial members, then **Finish**
6. Wait for the site to provision (~30 seconds)

**Verify:** Navigate to `https://[TENANT].sharepoint.com/sites/OFRIssueTracker` and confirm the site loads.

---

## Step 2: Create List — OFR_Issues

This is the primary issue tracking list. Each row represents one risk issue.

1. From the site home page, click **+ New** → **List**
2. Select **Blank list**
3. Name: `OFR_Issues`
4. Click **Create**

### Add Columns

The `Title` column already exists. Add the following 10 columns in order:

| # | Column Name | How to Create |
|---|-------------|--------------|
| 1 | **Title** | *(already exists)* — No changes needed. Max 255 characters. |
| 2 | **ItemID** | Click **+ Add column** → **Single line of text** → Name: `ItemID` → Save |
| 3 | **Owner** | Click **+ Add column** → **Single line of text** → Name: `Owner` → Save |
| 4 | **Priority** | Click **+ Add column** → **Choice** → Name: `Priority` → Choices: `High`, `Medium`, `Low` → Save |
| 5 | **Status** | Click **+ Add column** → **Choice** → Name: `Status` → Choices: `New`, `Active`, `Monitoring`, `Escalated`, `Closed` → Save |
| 6 | **DateRaised** | Click **+ Add column** → **Date and time** → Name: `DateRaised` → Include time: **No** → Save |
| 7 | **LastUpdated** | Click **+ Add column** → **Date and time** → Name: `LastUpdated` → Include time: **No** → Save |
| 8 | **NextAction** | Click **+ Add column** → **Multiple lines of text** → Name: `NextAction` → Type: **Plain text** → Save |
| 9 | **DaysSinceUpdate** | Click **+ Add column** → **Number** → Name: `DaysSinceUpdate` → Decimal places: **0** → Save |
| 10 | **StalenessFlag** | Click **+ Add column** → **Choice** → Name: `StalenessFlag` → Choices: `Current`, `Aging`, `Stale` → Save |
| 11 | **FunctionalGroup** | Click **+ Add column** → **Choice** → Name: `FunctionalGroup` → Choices: `Risk Management Office`, `Engagement Risk`, `Client Risk and KYC`, `Technology Risk & AI Trust`, `National Security`, `OGC General Counsel`, `OGC Privacy`, `OGC Contracts`, `Internal Audit`, `Independence` → Save |

### Verify OFR_Issues

Click through each column header to confirm all 11 columns exist with correct types. The list should have these columns in the default view: Title, ItemID, Owner, Priority, Status, DateRaised, LastUpdated, NextAction, DaysSinceUpdate, StalenessFlag, FunctionalGroup.

**Reference schema:** See `OFR_Issues-schema.json` for the definitive column specification.

---

## Step 3: Create List — OFR_UpdateHistory

This is an append-only audit trail for all issue updates.

1. From the site home page, click **+ New** → **List**
2. Select **Blank list**
3. Name: `OFR_UpdateHistory`
4. Click **Create**

### Add Columns

| # | Column Name | How to Create |
|---|-------------|--------------|
| 1 | **Title** | *(already exists)* — Used to store the ParentItemID for default view display. |
| 2 | **ParentItemID** | Click **+ Add column** → **Single line of text** → Name: `ParentItemID` → Save |
| 3 | **UpdateDate** | Click **+ Add column** → **Date and time** → Name: `UpdateDate` → Include time: **Yes** → Save |
| 4 | **StatusAtUpdate** | Click **+ Add column** → **Choice** → Name: `StatusAtUpdate` → Choices: `New`, `Active`, `Monitoring`, `Escalated`, `Closed` → Save |
| 5 | **Notes** | Click **+ Add column** → **Multiple lines of text** → Name: `Notes` → Type: **Plain text** → Save |
| 6 | **UpdatedBy** | Click **+ Add column** → **Single line of text** → Name: `UpdatedBy` → Save |

### Verify OFR_UpdateHistory

Confirm 6 columns exist. This list is append-only by convention — updates should never be edited or deleted.

**Reference schema:** See `OFR_UpdateHistory-schema.json`.

---

## Step 4: Create List — OFR_IntakeQueue

This is the triage queue for newly submitted issues.

1. From the site home page, click **+ New** → **List**
2. Select **Blank list**
3. Name: `OFR_IntakeQueue`
4. Click **Create**

### Add Columns

| # | Column Name | How to Create |
|---|-------------|--------------|
| 1 | **Title** | *(already exists)* — Issue title as submitted. |
| 2 | **Owner** | Click **+ Add column** → **Single line of text** → Name: `Owner` → Save |
| 3 | **Priority** | Click **+ Add column** → **Choice** → Name: `Priority` → Choices: `High`, `Medium`, `Low` → Save |
| 4 | **Description** | Click **+ Add column** → **Multiple lines of text** → Name: `Description` → Type: **Plain text** → Save |
| 5 | **DateSubmitted** | Click **+ Add column** → **Date and time** → Name: `DateSubmitted` → Include time: **No** → Save |
| 6 | **TriageStatus** | Click **+ Add column** → **Choice** → Name: `TriageStatus` → Choices: `Pending`, `Promoted`, `Dismissed`, `Accepted`, `Rejected` → Default value: `Pending` → Save |
| 7 | **FunctionalGroup** | Click **+ Add column** → **Choice** → Name: `FunctionalGroup` → Choices: `Risk Management Office`, `Engagement Risk`, `Client Risk and KYC`, `Technology Risk & AI Trust`, `National Security`, `OGC General Counsel`, `OGC Privacy`, `OGC Contracts`, `Internal Audit`, `Independence` → Save |

> **Important:** The TriageStatus column needs **all 5 choice values**:
> - `Pending` — default, set on submission
> - `Promoted` — set by the Power Automate Intake Promotion flow
> - `Dismissed` — set by the Power Apps Dashboard dismiss action
> - `Accepted` — set by the Power Apps Intake Review panel Accept button
> - `Rejected` — set by the Power Apps Intake Review panel Reject button

### Verify OFR_IntakeQueue

Confirm 7 columns exist with TriageStatus having all 5 choices and defaulting to "Pending", and FunctionalGroup having all 10 group choices.

**Reference schema:** See `OFR_IntakeQueue-schema.json`.

---

## Step 5: Load Sample Data (Optional)

Sample CSV files are provided in `sample-data/` for testing:

- `OFR_Issues-sample.csv` — 8 records with varied priorities, statuses, and staleness levels
- `OFR_UpdateHistory-sample.csv` — 17 update history entries across all issues
- `OFR_IntakeQueue-sample.csv` — 2 pending intake items for triage testing

### To Import:

**Option A — SharePoint Quick Edit:**
1. Open each list
2. Click **Edit in grid view** (or Quick Edit)
3. Copy-paste data from CSV rows into the grid
4. Click **Exit grid view** to save

**Option B — Manual Entry:**
1. Open each list
2. Click **+ New** for each row
3. Fill in field values from the CSV

> **Note:** SharePoint auto-generates the `ID` column. Do not attempt to set it manually. The `ItemID` field in OFR_Issues (e.g., `OFR-1`) is a text column that you set explicitly.

---

## Step 6: Final Verification

After creating all three lists and optionally loading sample data:

1. Navigate to `https://[TENANT].sharepoint.com/sites/OFRIssueTracker`
2. Click **Site contents** (left nav or gear icon)
3. Verify all three lists appear:
   - OFR_Issues
   - OFR_UpdateHistory
   - OFR_IntakeQueue
4. Open each list and confirm column count and types match the schema JSONs
5. If sample data was loaded, verify record counts: 8 issues, 17 updates, 2 intake items

**SharePoint setup is complete.** Proceed to `02-PowerAutomate/` to build the automation flows.
